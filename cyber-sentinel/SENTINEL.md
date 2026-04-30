# Sentinel — l'EDR LLM-first

> Detection par raisonnement semantique, pas par signature.

---

## Le probleme

Les RAT modernes (Sliver, Mythic, AsyncRAT) generent un hash unique a chaque build. Les bases de signatures sont mortes. Le ML sur features extraites (entropie binaire, structure des sections, tables d'import) capte les ressemblances avec du malware connu mais rate les binaires neufs en provenance de frameworks adversariaux.

Mesures comparatives sur le scenario de reference (un RAT installe via une application 3D legitime trojanisee) :

| Outil | Detection probable |
|-------|-------------------|
| Antivirus signatures | ~0% — hash inconnu |
| Defender for Endpoint macOS | ~25-35% |
| CrowdStrike Falcon | ~50-60% |
| SentinelOne Singularity | ~55-65% |
| **Sentinel** | **>90%** (POC mesure) |

Les EDR commerciaux integrent des assistants IA (Charlotte AI, Purple AI, Security Copilot) — mais ce sont des couches de tri **post-detection**. Si le sensor de base ne flagge pas, l'IA ne voit rien.

---

## L'hypothese

Et si le LLM faisait la detection primaire ?

Un modele comme Claude Haiku 4.5 peut lire le contexte complet d'un evenement (binaire, signatures de code, arbre de processus, connexions reseau, source d'installation, correlation OSINT) et raisonner sur l'incoherence comportementale. Pas un score numerique : un verdict explicite avec justification.

Exemple de verdict produit pendant le POC, sur un faux LaunchAgent simulant un RAT :

> Le binaire s'appelle `com.system.update.helper` mais n'est pas signe Apple. Il a ete installe par un processus telecharge via curl depuis un domaine non-whitelisted. Il ouvre une socket vers une IP Vultr connue pour heberger du C2. La combinaison nom-imitant-Apple + non-signe + connexion-sortante-non-sollicitee + provenance-curl-direct est un pattern de RAT par persistance LaunchAgent. Verdict : RAT, confiance 99%. Action recommandee : kill + quarantaine.

C'est une inference qui combine 5-6 signaux faibles en une conclusion forte. Aucun outil traditionnel ne le ferait.

---

## L'architecture

```
                ┌─────────────────┐
                │  Mac M3 / M2 /  │  Endpoints proteges
                │   Spark Linux   │
                └────────┬────────┘
                         │
                ┌────────▼────────┐
                │   osquery       │  Couche 1 : observation
                │  (Apache 2.0)   │  <2% CPU, <50 MB RAM
                └────────┬────────┘
                         │ JSON events
                         ▼
                ┌─────────────────┐
                │     CORTEX      │  Couche 2 : analyse
                │  Bun TypeScript │
                │                 │
                │  ┌───────────┐  │
                │  │  triage   │  │  Haiku — 95% des events
                │  │  Haiku    │  │  $0.0015/event, 2.64s
                │  └─────┬─────┘  │
                │        │        │
                │        │ suspect│
                │        ▼        │
                │  ┌───────────┐  │
                │  │ deep dive │  │  Sonnet — 5% escalation
                │  │  Sonnet   │  │  $0.008/event, 8.31s
                │  └─────┬─────┘  │
                │        │        │
                │  ┌─────▼─────┐  │
                │  │  threat   │  │  Enrichissement IOC
                │  │   intel   │  │  16 APIs gratuites
                │  └───────────┘  │
                └────────┬────────┘
                         │ verdicts + niveau menace
                         ▼
                ┌─────────────────┐
                │    RESPONSE     │  Couche 3 : actions
                │                 │
                │  log → alerte   │
                │  isolate proc   │
                │  kill + quaran  │
                │  block IP       │
                │  isolate net    │
                └─────────────────┘
```

---

## Resultats POC quantifies

Test sur 100 evenements representatifs (90 benins + 10 menaces) :

| Metrique | Cible PRD | Haiku 4.5 | Sonnet 4.6 |
|----------|-----------|-----------|------------|
| Confiance detection | >80% | 97.6% | 98.0% |
| Latence par evenement | <5s | 2.64s | 8.31s |
| Cout/jour 1000 events | <$5 | $1.52 | $7.97 |
| Recall menaces | >80% | 100% | 100% |
| F1 score corrige | — | ~95% | ~95% |

Les 10 vecteurs de menace testes : RAT LaunchAgent, cryptominer xmrig, reverse shell, info stealer (keychain/browser), backdoor, dropper curl|bash, brute force SSH, exfiltration DNS, escalade SUID, persistance crontab C2.

100% recall sur les 10 vecteurs avec les deux modeles.

---

## Limites honnetes

- **Latence ~3s par evenement** : impose un pre-filtrage local. Pour un volume eleve d'events benins, on triage avec des regles ou un petit modele local (Nemotron sur GPU Spark) avant l'API cloud.
- **Cout au volume** : avec prompt caching le cout est gerable, mais sans optimisation un trafic non-filtre peut couter cher. Architecture en deux niveaux indispensable.
- **Hallucinations** : zone grise. Seuils de confiance eleves (>85%) avant toute action destructive. Mode "apprentissage initial" pendant 7 jours pour calibrer la baseline.
- **Defense complementaire requise** : pour les menaces fast-moving (ransomware qui chiffre en secondes), la latence LLM ne suffit pas. Sentinel coexiste avec une couche de defense classique pour ces cas.

---

## Threat intelligence

16 sources gratuites integrees ou en plan :

**P0 (implementation Phase 1)** : Abuse.ch ThreatFox + URLhaus + MalwareBazaar + Feodo Tracker, CISA KEV, Google OSV.

**P1 (Phase 2)** : NVD (NIST), AlienVault OTX, AbuseIPDB.

**P2 (Phase 3)** : VirusTotal (rate limit 4/min, 500/jour gratuit), Vulners, GreyNoise, CIRCL/MISP.

Cout total : **0 USD/jour** pour notre volume. Architecture en 3 couches (cache local SQLite refresh 5min + lookup API on-demand + sync periodique 2h).

---

## Pourquoi pas (juste) Darktrace

Darktrace est l'outil le plus proche philosophiquement — ML non-supervise sur le comportement du reseau, reponse autonome (Antigena). Mais Darktrace utilise des autoencoders et methodes Bayesiennes — il detecte des **anomalies statistiques**. Il ne peut pas *expliquer* pourquoi quelque chose est suspect en termes humains. Le LLM le peut, et c'est central pour le forensics conversationnel et la confiance operationnelle.

Sentinel et Darktrace ne sont pas concurrents — ils sont complementaires. Mais sur le critere "explicabilite native du verdict", Sentinel est unique.

---

## Roadmap publication

- Phase 0 (recherche + POC) — termine, rapports complets disponibles
- Phase 1 (daemon osquery + pipeline) — sprint en cours
- Phase 2 (Cortex basique + dashboard) — sprint suivant
- Phase 3 (Response autonome + forensics conversationnel) — Q3 2026
- Phase 4 (honeypots + mutation defense + supply chain audit) — Q4 2026
- Phase 5 (packaging produit, mode SaaS, agent marketplace) — 2027

Le code et les rapports techniques sont ouverts au fur et a mesure des livraisons.
