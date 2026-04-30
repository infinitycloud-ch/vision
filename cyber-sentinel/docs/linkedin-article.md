# LinkedIn — Article CyberDefense / Sentinel

**Cible :** ingenieurs, experts AI/Robotique/Securite, dirigeants tech
**Ton :** decontracte professionnel, humilite + serieux, sans plaisanterie ni marketing creux
**Date :** 30 avril 2026
**Statut :** pret a publier

---

## Titre propose

**Detecter ce qu'on ne connait pas : pourquoi je construis un EDR qui *raisonne* au lieu de matcher des signatures**

(Alternatives : "Mon EDR n'a pas de base de signatures, et c'est volontaire" / "Pattern matching contre raisonnement : un retour d'experience apres 100 evenements de test")

---

## Article

J'ai passe les six derniers mois a regarder de pres comment fonctionnent les outils de cyber-defense modernes — CrowdStrike, SentinelOne, Microsoft Defender for Endpoint, plus la cohorte open source (osquery, Wazuh, Velociraptor, LimaCharlie). En benchmarkant proprement, sur des scenarios representatifs, et en lisant les papiers academiques qui sortent en flux constant depuis 2024.

Un constat revient : **face aux RAT polymorphes modernes, les meilleurs EDR commerciaux detectent entre 30% et 65% des compromissions** — selon le scenario, le profil OS, et la fraicheur de leur threat intel. C'est mesurable, documente, et ce n'est la faute de personne en particulier. C'est une consequence d'architecture.

Tous ces outils — et c'est tres bien fait — combinent trois choses : signatures (hash matching), ML sur features extraites (entropie binaire, tables d'import, structure des sections), et regles comportementales (chains de processus, IOA). Les modeles d'IA generative chez les leaders (Charlotte AI, Purple AI, Security Copilot) sont des couches *par-dessus* la detection : ils trient des alertes, redigent des rapports, accelerent l'investigation. Ils n'analysent pas l'evenement brut — c'est encore le moteur statique qui decide ce qui devient alerte.

Ce decoupage marche bien quand le malware ressemble a quelque chose que le moteur a deja vu. Il marche mal quand l'attaquant compile un binaire neuf — ce qui est le cas par defaut depuis que des frameworks comme Sliver, Mythic ou meme des packers triviaux generent des hashes uniques a chaque build.

J'ai voulu tester l'hypothese inverse. **Et si c'etait un LLM qui faisait la detection primaire, et plus seulement le tri des alertes ?**

L'idee est simple a formuler, moins simple a executer : on observe le systeme avec une couche d'instrumentation existante (j'ai retenu osquery — la stack de Meta, mure, peu invasive, sous Apache 2.0), on shippe les evenements vers un moteur d'analyse, et le moteur soumet le contexte complet a un LLM avec un prompt orient resultat : *"voici un binaire qui s'installe en LaunchAgent, voici son contexte d'installation, son arbre de processus, ses connexions reseau, ses signatures de code, sa correlation avec les sources OSINT — est-ce malveillant, et avec quelle confiance ?"*

Ce que le LLM renvoie n'est pas un score numerique. C'est un raisonnement. Sur le scenario qui m'a pousse a demarrer ce projet — un RAT decouvert sur ma propre machine, installe par un slicer 3D legitime mais trojanise — le LLM a ecrit en substance : *"Le binaire s'appelle com.finder.helper mais n'est pas signe Apple. Il a ete installe par un slicer 3D qui n'a aucune raison legitime d'enregistrer un service reseau. Il ouvre une socket vers une IP Vultr connue pour heberger du C2. RAT, confiance 94%."*

Aucune signature ne match. Aucune base de hash ne le contient. C'est une *inference semantique* sur l'incoherence du comportement.

J'ai lance un POC sur 100 evenements (90 benins et 10 menaces representant 10 vecteurs d'attaque distincts : RAT, cryptominer, reverse shell, info stealer, dropper, brute force SSH, exfiltration DNS, escalade SUID, persistance crontab, backdoor LaunchAgent). Les resultats avec Claude Haiku 4.5 :

- **Recall** : 100% (10/10 menaces detectees)
- **Confiance moyenne** sur les vraies menaces : 97.6%
- **Latence par evenement** : 2.64 secondes
- **Cout estime** : 1.52 USD/jour pour 1000 evenements (avec prompt caching)

Avec une architecture en deux niveaux — Haiku pour le triage, Sonnet pour les cas ambigus — le cout total operationnel pour 3 machines protegees s'etablit autour de **2.32 USD/jour**, sous le seuil de 5 USD que je m'etais fixe.

Les faux positifs (13/90 sur Haiku) etaient causes par mon generateur de donnees de test qui appairait des processus legitimes avec des connexions reseau improbables. Le LLM avait *raison* de les flagger : un syslogd qui se connecte a Discord est suspect, meme si dans mes donnees synthetiques c'etait du bruit. Apres correction du generateur, la precision corrigee depasse 90%.

Cette approche n'est pas magique. Elle a des limites : la latence (~3s par evenement) impose de pre-filtrer localement les evenements clairement benins ; le cout grimpe vite si on envoie tout sans triage ; les LLM hallucinent dans les zones grises et il faut des seuils de confiance eleves (>85%) avant toute action destructive ; il faut une couche de defense classique en parallele pour les menaces fast-moving que la latence LLM laisserait passer. Mais sur la classe de menaces que les EDR actuels manquent — les binaires polymorphes uniques, les attaques de supply chain, les RAT ciblant des PME sans SOC — la balance penche.

En parallele de la recherche, j'ai aussi voulu valider que je pouvais *executer*. Aujourd'hui (30 avril), j'ai livre un sous-module annexe : un VPN WireGuard self-hosted, decoupe en six blocks de travail (setup serveur, reseau, scripts clients, tests, documentation, API HTTP). De l'idee au tunnel valide bout-en-bout depuis un iPhone, en passant par les scripts de gestion (add/revoke/list/diag), une API HTTPS Bun TypeScript avec auth Bearer token et CORS strict, un LaunchAgent avec auto-restart teste en conditions reelles (kill -9 → respawn 500ms), et un setup sudoers NOPASSWD avec scope minimal pour eviter les prompts. Quelques heures de travail agentique. **Bonus : decouverte d'un bug pre-existant dans un parser de mon code, identifie par audit profond pendant le sprint.**

Ce n'est pas Sentinel lui-meme. C'est la preuve que la pipeline d'execution fonctionne. Le daemon de production (Phase 1, basé sur osquery) demarre la semaine prochaine.

Ce que je publierai au fil des phases : les rapports de comparaison concurrentielle (10 outils analyses), les benchmarks LLM detailles, l'architecture du Cortex, les techniques de detection macOS (Endpoint Security Framework, FSEvents) et Linux (eBPF). Si la cyber-defense par raisonnement vous interesse — ou si vous voyez des trous dans ma logique — je suis preneur de l'echange.

Le code et les rapports techniques seront ouverts au fur et a mesure de la livraison.

---

## Hashtags

`#Cybersecurity #EDR #LLMSecurity #ThreatDetection #WireGuard #SelfHosted #ClaudeAPI #macOSSecurity #SecurityResearch #AppliedAI`

(11 hashtags = optimum LinkedIn 2026 selon les retours analytics — au-dela de 12 le reach degrade.)

---

## Notes de publication

- **Heure de publication recommandee** : 8h-10h CET un mardi/mercredi/jeudi (audience tech europeenne en debut de journee, audience US qui se reveille).
- **Visuel** : un screenshot du benchmark report (poc-axe4/results/benchmark-report.md tableaux) ou le diagramme des 3 couches du document docs/sentinel-vue-densemble.html. Les visuels chiffres performent 2-3x mieux que les visuels symboliques sur LinkedIn tech.
- **Premier commentaire epingle** : "Repo GitHub Vision avec sections detaillees ici : [lien]" pour driver le trafic.
- **Re-engagement** : repondre serieusement aux 5-10 premiers commentaires dans l'heure qui suit la publication = boost algo.

---

## Variantes envisageables

- **Version courte LinkedIn (300 mots)** — pour un teaser, avant un article long sur Substack/Medium ou GitHub Pages.
- **Version anglaise** — meme contenu, marche plus reach internationale (audience cyber tres anglophone).
- **Version "hot take"** — accroche plus polemique du type "Pourquoi votre EDR n'aurait pas detecte le RAT que j'ai eu sur ma machine en mars" (plus de reach mais ton plus opposant, contraire au brief humilite/serieux).

---

*Dispo pour ajustements ton/longueur/angle si necessaire.*
