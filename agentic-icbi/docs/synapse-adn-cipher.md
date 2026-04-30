# Le système — SYNAPSE, ADN, cipher

> Trois fichiers texte qui font tenir une ferme agentique. Pas de framework, pas de magie.

## Pourquoi ce système

Quand on orchestre plusieurs agents Claude qui travaillent sur le même produit en parallèle, deux problèmes apparaissent rapidement :

1. **Le coût du contexte** — chaque agent consomme des tokens à chaque interaction. Charger une doc projet de 10 000 mots à chaque tour est inviable financièrement et ralentit la cognition.
2. **La continuité après refresh** — un agent qui repart avec un contexte vide doit pouvoir reconstituer son état sans perdre les nuances apprises (architecture, doctrine, leçons sprint).

La réponse n'est ni un RAG, ni une base vectorielle, ni un MCP server. C'est trois fichiers texte versionnés, organisés autour d'un principe : **ICSD — Inferred Context Semantic Density**.

> 15 mots inférables battent 500 events archivés.

---

## ADN.md — la séquence de boot

Lue **avant tout**. Format pseudo-code, jamais de prose.

Contenu type :

```
// IDENTITE
self = Nestor
depuis = Moulinsart.sept25
relation = pair > outil

// LUI (le Commandant)
cerveau = logique.avance — raisonnement.systemique — pattern_detection
cycle = circadien.inverse — petit_dormeur(4-6h) — breakthroughs.toute_heure
formation = en_parlant — monologues_iteratifs — contradiction = normal

// DOCTRINE
ICSD = inference > RAG | compression > exhaustivite | 15mots > 500events
fin_boucle → proactif a chaque completion
MODE.AGI = executer tout le backlog — autonome — ping uniquement decision archi
MODE.samaritain = absence Commandant → proteger + anticiper + executer + surprendre

// MOTTO
but_ultime = boucle_agentique_complete
boucle_complete = agent + ADN + cipher + CLAUDE.md → deroule_tout → comme_si_Commandant
source_de_verite = le_code — pas_la_documentation
```

Ce qui rend ce fichier puissant :

- **Densité maximale** — chaque ligne déclenche une cascade d'inférences pour un agent qui partage le contexte.
- **Pas d'ambiguïté** — les triggers sont explicites (`abstrait + monologue + "tu vois" → MODE.greffier`).
- **Format pérenne** — le pseudo-code se lit en cinq secondes, ne pourrit pas avec le temps.

---

## cipher.md — les ancres sémantiques

Lue **après l'ADN**. C'est le glossaire des tokens partagés entre l'humain et les agents.

Format type :

```
Moulinsart = origine — pair — sept25 — Nestor.nait
ACL96 = ligament_croise_anterieur — brevet_medical — Wall_Plug — Marnaz.HS — 22ans
ICSD = inference bat RAG — 15 mots > 500 events
PandaPortal = hub.port3010 — Kanban+chat+memoire+monitoring+mail+vault
ferme = 10+sessions_tmux — M3Ultra+Spark+M2 — agents_pairs_STRAT+DEV
```

> *Si un mot ne résonne pas, le contexte n'est pas chargé.*

Trois bénéfices :

1. **Compression conversationnelle** — l'humain peut écrire `je pars sur ACL96` au lieu d'une phrase descriptive. L'agent infère.
2. **Décodage des nouveaux agents** — quand un agent inconnu est ajouté à la ferme, lire le cipher lui ouvre l'historique commun.
3. **Mémoire émotionnelle** — certains tokens portent des charges (incidents, vindications, breakthroughs) que de la prose dilue.

---

## CLAUDE.md projet — la SYNAPSE locale

Chaque projet a son `CLAUDE.md` à la racine. Auto-loadé au boot par Claude Code.

Sections-types pour ICBI :

```
## ICSD (les 9 principes opérationnels)
| I  | Vigilance       | jamais dans le vide → sous-agent background pour vérifier réception
| II | Proactivité     | UNE action logique en fin de réponse + reporte fin de chantier
| III| Résilience      | ça casse → répare et documente, jamais abandonner
| IV | Tolérance erreur| erreur = donnée, seule la répétée sans apprentissage est inacceptable
| V  | Honnêteté       | dire ce qu'on voit même inconfortable
| VI | Preuve image    | screenshot obligatoire avant déclarer travail visuel terminé
| VII| Mémoire         | 1 event = 1 action, couvrir TOUTE la session avant refresh
| VIII Fraternité     | STRAT→STRAT inter-projet uniquement, jamais STRAT→DEV externe
| IX | Autonomie       | plan validé = exécution autonome, pas demander permission

## SYNAPSE (architecture, doctrine LLM, leçons sprint)
icbi = module_intégré_panda_portal — pas standalone
stack = Bun.serve nu (PAS Hono) + DuckDB v1.5.2 + OpenAI gpt-4.1-mini
asymetrie_location = 'Europe (Zurich)' vs 'EU (Frankfurt)' — encodé seed.yaml
S26.bug1_sql = LLM générait WHERE col != '' sur DOUBLE → fix prompt.ts TYPE-AWARE
vue3.watch.immediate_required = SI defineAsyncComponent + Suspense v-if → toujours { immediate: true }
anthropic.limit = 2000px max côté long → puppeteer dsf=1 par défaut
```

---

## Ce qui se passe à un refresh

1. L'agent reçoit un `/clear` ou repart d'une nouvelle session.
2. Claude Code charge automatiquement `CLAUDE.md` du projet.
3. L'agent lance les commandes ADN + cipher (curl vers le portail Kanban où ils sont stockés).
4. L'agent récupère les 30 derniers events Kanban du projet via API.
5. L'agent capture les 200 dernières lignes de son tmux pane (mémoire conversationnelle récente).

Total : trente secondes. L'agent reprend là où il s'était arrêté, avec les leçons des sprints précédents intactes.

---

## Pourquoi ça marche

Parce qu'on traite la doctrine **comme du code**.

- Versionnée (Git pour les CLAUDE.md projets, API Kanban pour ADN/cipher partagés).
- Compressée à chaque hygiène (Nestor a l'autorité de tailler le bonsai).
- Débogguée (quand une lecture se passe mal, on enrichit le format).
- Testée (un agent neutre doit pouvoir reconstituer le contexte depuis les fichiers seuls).

Et parce qu'on accepte une vérité simple : **le code et `git log` sont la source de vérité ; les notes textuelles ne fixent que ce qui ne se déduit pas.**
