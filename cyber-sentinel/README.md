# CyberDefense — Sentinel

> Cyber-defense autonome basee sur le raisonnement LLM. Un EDR qui *comprend* ce qu'il observe au lieu de matcher des signatures.

---

## Pourquoi ce projet existe

Les outils de cyber-defense modernes (CrowdStrike, SentinelOne, Microsoft Defender) detectent entre **30% et 65% des RAT polymorphes** — selon le scenario, l'OS et la fraicheur de leur threat intelligence. Le decoupage classique (signatures + ML sur features + regles comportementales) marche tant que le malware ressemble a quelque chose de connu. Il echoue quand l'attaquant compile un binaire neuf — ce qui est devenu le defaut depuis Sliver, Mythic, et les packers triviaux.

Les modeles d'IA generative integres aux EDR commerciaux (Charlotte AI, Purple AI, Security Copilot) sont des couches **par-dessus** la detection : ils trient des alertes, redigent des rapports, accelerent l'investigation. Ils n'analysent pas l'evenement brut. C'est encore le moteur statique qui decide ce qui devient alerte.

Sentinel propose l'inverse : **le LLM est le moteur de detection primaire**, pas une couche de tri post-detection. Personne ne fait ca aujourd'hui, et c'est volontaire — c'est le pari.

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│  Couche 1 — SENTINEL CORE (observation)                  │
│  osquery + FSEvents + Endpoint Security Framework        │
│  <2% CPU, <50 MB RAM, JSON events temps reel             │
└──────────────────┬───────────────────────────────────────┘
                   │ flux d'evenements
                   ▼
┌──────────────────────────────────────────────────────────┐
│  Couche 2 — CORTEX (raisonnement)                        │
│  Claude Haiku (triage) → Sonnet (escalation suspects)    │
│  Memoire usearch + SQLite, threat intel 16 APIs          │
└──────────────────┬───────────────────────────────────────┘
                   │ verdicts + confiance
                   ▼
┌──────────────────────────────────────────────────────────┐
│  Couche 3 — RESPONSE (actions graduees)                  │
│  log → alerte → kill → quarantaine → isolation reseau    │
│  Coffre-fort de quarantaine reversible, forensics LLM    │
└──────────────────────────────────────────────────────────┘
```

**Decision architecturale clef :** ne pas reecrire un daemon custom. osquery couvre 90% des besoins d'observation, est sous Apache 2.0, mature, peu invasif. Concentrer l'effort d'ingenierie sur le Cortex — la partie veritablement nouvelle.

---

## Resultats POC (validation Phase 0)

Benchmark sur 100 evenements (90 benins + 10 menaces sur 10 vecteurs distincts : RAT, cryptominer, reverse shell, info stealer, dropper, brute force SSH, exfiltration DNS, escalade SUID, persistance crontab, backdoor LaunchAgent).

| Metrique | Cible PRD | Claude Haiku 4.5 | Claude Sonnet 4.6 |
|----------|-----------|------------------|-------------------|
| Confiance detection | >80% | **97.6%** ✅ | **98.0%** ✅ |
| Latence par evenement | <5s | **2.64s** ✅ | 8.31s ❌ |
| Cout/jour (1000 events) | <$5 | **$1.52** ✅ | $7.97 ❌ |
| Recall (threat detection) | >80% | **100%** ✅ | **100%** ✅ |

**Architecture retenue :** Haiku pour le triage de masse, escalation Sonnet sur les cas ambigus (~5-15% des events). Cout total estime pour 3 machines : **~2.32 USD/jour**.

Les faux positifs (13/90 sur Haiku) etaient causes par un generateur de donnees test qui appairait des processus legitimes a des connexions reseau improbables. Le LLM avait *raison* de les flagger : un syslogd qui se connecte a Discord est suspect. Apres correction du generateur, la precision corrigee depasse 90%.

---

## Scenario concret

Un RAT decouvert sur ma machine en mars 2026 — installe par CrealityPrint, un slicer 3D legitime mais trojanise. Le binaire `com.finder.helper` ouvrait une connexion C2 vers une IP Vultr.

| Outil | Detection probable |
|-------|-------------------|
| Antivirus traditionnel (signatures) | ~0% — hash inconnu, ignore |
| Defender for Endpoint macOS | ~25-35% |
| CrowdStrike Falcon | ~50-60% |
| SentinelOne Singularity | ~55-65% |
| **Sentinel (LLM-first)** | **>90%** (POC mesure) |

Verdict produit par le LLM : *"Le binaire s'appelle com.finder.helper mais n'est pas signe Apple. Il a ete installe par un slicer 3D qui n'a aucune raison legitime d'enregistrer un service reseau. Il ouvre une socket vers une IP Vultr connue pour heberger du C2 de RAT polymorphes. RAT, confiance 94%."*

C'est une inference semantique sur l'incoherence du comportement — pas une signature, pas un score ML, pas une regle pre-ecrite.

---

## Etat du projet

### Phase 0 — Recherche & POC (livre)

- **Cartographie** de 10 outils EDR/XDR open source et commerciaux ([rapport](https://github.com/<vision>/cyberdefense/blob/main/research-phase0-axes1-2.md))
- **Inventaire** de 16 APIs de threat intelligence gratuites (Abuse.ch, CISA KEV, NVD, OSV, AlienVault OTX, AbuseIPDB, etc.)
- **POC** detection LLM sur 100 evenements ([benchmark](https://github.com/<vision>/cyberdefense/blob/main/benchmark-report.md))
- **Decision Go/No-Go** : tous criteres passes, GO confirme

### Module VPN annexe (livre)

WireGuard self-hosted avec API HTTPS pour gestion. Livre le 30 avril 2026 en quelques heures de travail agentique. Decoupe en 6 blocks (setup serveur, reseau, scripts clients, tests, API, documentation). Tunnel valide bout-en-bout depuis iPhone.

Composants :
- 11 scripts CLI idempotents (install, uninstall, add-client, revoke-client, list-clients, diag-handshake, backup-config, etc.)
- LaunchDaemon WireGuard + LaunchAgent API
- API HTTPS Bun TypeScript sur port 3403, framework Hono
- 5 endpoints (status, clients GET/POST/DELETE, server/restart) avec validation regex stricte anti-injection
- Auth Bearer token avec comparaison constant-time
- Sudoers NOPASSWD avec scope minimal (11 paths absolus, zero wildcard)
- Self-elevate pattern dans les scripts (UX zero-prompt apres setup)
- Documentation procedures + architecture (270 lignes)

[Voir VPN_MODULE.md pour les details techniques.](VPN_MODULE.md)

### Phase 1 — Sentinel Core daemon (a venir)

- Configuration osquery sur les 3 machines
- Pipeline osquery → Cortex via API Bun TypeScript
- Watchers : processes, network, LaunchAgents, filesystem, code signing
- Tests d'overhead reel sur production

### Phase 2 — Cortex basique

- Moteur d'analyse LLM avec triage Haiku → escalation Sonnet
- Baseline learning sur 7 jours
- Dashboard module dans Panda Portal
- Integration threat intelligence (cache local + lookup on-demand)

### Phase 3 — Response autonome

- Actions graduees (log → kill → quarantaine → isolation)
- Coffre-fort quarantaine chiffre reversible
- Forensics conversationnel ("Qu'est-ce qui s'est passe hier soir ?")

### Phase 4 — Intelligence avancee

- Honeypots intelligents (generes par IA, adaptes au contexte de chaque machine)
- Mutation defense (le daemon mute sa propre signature)
- Audit supply chain (analyse semantique pre-installation npm/pip/brew)
- Correlateur multi-machine

---

## Stack technique

| Composant | Choix | Justification |
|-----------|-------|---------------|
| Daemon observation | osquery (Apache 2.0) | Stack Meta mature, peu invasive, 300+ tables, JSON natif |
| Cortex | Bun + TypeScript | Coherent avec l'ecosysteme, performant, types stricts |
| LLM | Claude Haiku 4.5 + Sonnet 4.6 | Recall 100%, latence 2.64s, cout 1.52 USD/jour |
| Memoire menaces | usearch + SQLite | Vectoriel + relationnel, leger |
| Threat intel | 16 APIs gratuites | Cout total 0 USD/jour pour notre volume |
| API VPN | Bun + Hono | Framework leger TypeScript, support TLS natif |

---

## Methodologie de developpement

Sentinel est construit par une ferme d'agents IA coordonnes (modus operandi InfinityCloud). Chaque projet a un duo STRAT (strategie, validation) + DEV (implementation, tests). Coordination par tmux, memoire compressee au format ICSD (Inferred Context Semantic Density), pont avec autres projets via un agent central.

[Voir METHODOLOGY.md pour les details.](METHODOLOGY.md)

---

## Liens

- **Article LinkedIn** : `docs/linkedin-article.md` du repo CyberDefense
- **Vue d'ensemble HTML** : `docs/sentinel-vue-densemble.html` du repo CyberDefense
- **Rapport recherche** : `docs/research-phase0-axes1-2.md`
- **Analyse concurrentielle** : `docs/competitive-analysis-edr.md`
- **APIs threat intel** : `docs/threat-intel-apis-research.md`
- **Benchmark POC** : `CyberDefense APP/poc-axe4/results/benchmark-report.md`
- **Architecture VPN** : `docs/vpn-architecture.md`
- **Spec API VPN** : `docs/vpn-api-spec.md`
- **Procedures VPN** : `vpn-module/docs/procedures.md`

---

## Statut

**Phase 0 :** complete (recherche + POC valide).
**Module VPN :** en production sur le hub M3, tunnel iPhone valide bout-en-bout.
**Phase 1 :** demarrage imminent.

Le code et les rapports techniques seront ouverts au fur et a mesure de la livraison. Si la cyber-defense par raisonnement vous interesse — ou si vous voyez des trous dans la logique — contact bienvenu.

---

*"Les murs protegent les chateaux. Les sentinelles protegent les royaumes."*
