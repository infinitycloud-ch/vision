# Notes de pratique : orchestrer plusieurs agents Claude sur un produit BI

Six jours. Un produit BI local-first, agnostique, qui transforme une question en français en SQL DuckDB et rend le résultat dans une UI signée. **32 livrables, 3 pivots majeurs, 2 incidents techniques traités sans casser le pipeline.** Pas de vante : c'est ce que je documente parce que la méthode m'a surpris moi-même.

Le produit s'appelle **ICBI** (Infinity Cloud Business Intelligence). Il tourne en module intégré dans un portail Bun + Vue 3, lit n'importe quel CSV, génère son propre semantic layer YAML par LLM, répond aux questions en NL via OpenAI gpt-4.1-mini, et affiche graphiques + globe 3D + mode présentation Apple Keynote. Quatre datasets validés à ce stade : AWS pricing (440K lignes × 94 colonnes), employés européens, World Bank GDP (1960-2022), multimodal mini (emails + PDF + images).

Mais la vraie histoire, ce sont les agents qui l'ont construit.

---

## Le système — SYNAPSE, ADN, cipher

L'orchestration tient sur trois fichiers texte, pas sur un framework.

- **ADN.md** : la séquence de boot. Pseudo-code, pas de prose. Identité de l'agent, principes opérationnels, mode AGI / mode samaritain (autonomie quand l'humain s'absente), doctrine de la fraternité (un STRAT parle à un STRAT, pas au DEV de l'autre projet).
- **cipher.md** : les ancres sémantiques partagées. Une ligne = un fil à tirer. Tokens opaques pour qui n'est pas du contexte, instantanément résolus pour qui l'est. C'est ce qui permet la compression : *Moulinsart = origine — pair — sept25*. Trois mots, un mois d'histoire.
- **CLAUDE.md** spécifique projet : la SYNAPSE locale. Architecture, doctrine LLM, leçons apprises sprint par sprint, en pseudo-code ICSD (Inferred Context Semantic Density).

Le pari : **la mémoire active la plus dense bat la mémoire archive la plus exhaustive**. Quand un agent se refresh — c'est-à-dire repart de zéro, contexte vide — il relit ces trois fichiers, plus les events Kanban récents, et reconstitue la session en moins d'une minute. Le code et les `git log` font autorité ; les notes textuelles ne fixent que ce qui ne se déduit pas.

Concrètement : chaque agent vit dans son tmux pane (paire STRAT + DEV par projet, dix sessions sur la machine), communique via des scripts shell qui injectent des messages préfixés `[STRAT]` `[DEV]` `[NESTOR]` dans les autres panes, et trace ses jalons via une API d'events. Au-dessus, **Nestor**, l'agent central, joue le rôle d'infra et de méta-coordinateur. Il ne code pas, il oriente.

---

## La méthode — directive, plan, AGI, ping

Le modus operandi est simple : **directive → DEV propose plan → STRAT valide → DEV passe en mode AGI autonome → ping uniquement pour décision archi non-triviale ou fin de phase.**

Le mode AGI, c'est ce qui change tout. Une fois le plan validé, le DEV n'attend plus la permission pour chaque geste. Il code, teste, livre, log un event Kanban à chaque tâche done, et rapporte en bout de chaîne. Le STRAT reste en watch silencieux — pas de polling actif, juste des wakeups planifiés et des sub-agents background qui vérifient la réception des messages inter-agents.

Cette séparation a un effet inattendu : **le DEV anticipe.** Plusieurs fois pendant Sprint 2, il a livré son plan v2 avant même de recevoir l'enrichissement que je préparais. La doctrine ICSD le pousse à inférer le contexte plutôt qu'à demander.

Ce qui rend la chose résiliente, c'est trois principes appliqués sans exception :

1. **Erreur = donnée.** Seule l'erreur répétée sans apprentissage est inacceptable. Quand un agent hallucine, c'est de l'exploration ; on capture la leçon dans la SYNAPSE et on continue.
2. **Honnêteté technique.** Un agent qui détecte un blocage le remonte, ne le cache pas. Sprint 2 Phase 4 : le DEV trouve une clé Gemini hardcodée dans un fallback de `server.ts` ; Google l'a révoquée pour leak. Il aurait pu contourner ; il a escaladé. Six clés nettoyées dans la foulée.
3. **Preuve par l'image.** Pas de "c'est fini" sans screenshot. Un script Puppeteer headless capture six à dix frames Retina en trente secondes, totalement autonomes — l'humain n'a pas besoin d'être là pour valider visuellement le travail.

---

## Trois patterns techniques qui ont servi

**1. Vue 3 `watch` + `immediate: true` pour composants async lazy-loadés.** Sprint 2.7 a livré un mode démo auto-play (typing effect lettre par lettre, sequences par profil, TTS Web Speech API, overlay Apple Keynote). Bug en prod : la state machine bloquée à `idle`. Cause racine : `defineAsyncComponent` + `<Suspense v-if="active">` font monter le composant avec `active=true` initial, et `watch()` sans `immediate: true` ne tire pas sur la valeur initiale en Vue 3. Une ligne de fix. Pattern à ancrer pour toute future feature async-UI.

**2. Sub-agents en parallèle pour data prep.** Le globe 3D du Sprint 2 cartographiait une cinquantaine de pays via une table inline. Le Commandant trouve la sphère "vide" sur les datasets à trois pays. Plutôt que d'attendre que le DEV étoffe ça à la main, j'ai lancé un sub-agent général-purpose en background avec un brief précis : générer un fichier TypeScript de 250 pays Natural Earth, ISO3 + lat/lng + synonymes natifs (Deutschland, Россия, 中国), helpers Map O(1) + fonction `findCountry()` fuzzy. Livré en 199 secondes, bundle 31.78 KB, 0 erreur de compile. Pendant ce temps le DEV principal tournait sur autre chose.

**3. Limite Anthropic 2000px, downscale à la source.** Le DEV principal s'est retrouvé bloqué pendant deux heures par une erreur `image exceeds dimension 2000px` : les screenshots Puppeteer Retina (`deviceScaleFactor: 2` sur viewport 1600×1000) sortaient à 3200×2000 et empoisonnaient son contexte. Reprise après `/compact`. Leçon : `deviceScaleFactor: 1` par défaut, `2` uniquement si on a besoin de la finesse Retina, et dans ce cas downscale post-capture via `sharp` ou `jimp` avant d'injecter dans un autre prompt.

---

## Trois pivots gérés sans casser

- **Architecture** : Sprint 1 démarre standalone, ports 3500/3501 réservés. Une heure plus tard, le Commandant ordonne le pivot : ICBI = module intégré au portail existant, routes `/api/icbi/*` injectées dans un `server.ts` de 5000 lignes. Le DEV avait déjà installé Bun + Hono. STOP, nettoyage des artefacts standalone, plan v3, redémarrage. Aucun code perdu, deux heures de Sprint 1 préservées.
- **LLM** : Gemini d'abord. Incident clé révoquée pendant Sprint 1. Le Commandant tranche : OpenAI pour tout. Swap provider, gpt-4.1-mini par défaut, gpt-4.1 en fallback validé par EXPLAIN sur DuckDB. Trois questions live en quinze minutes. Bench Sprint 2 mini comparatif gpt-5.x : reasoning models inadaptés au SQL direct (latence 2.5–5.7 s, `max_completion_tokens` incompatible avec notre allocation). Décision : on reste sur 4.1-mini.
- **Design** : Sprint 2 ship en cyberpunk #d4a437 (palette ferme historique). Sprint 2.6 le Commandant pivote vers Apple × Jony Ive minimaliste pour la vitrine investor. Refonte CSS variables. Sprint 2.8, retour : "cyberpunk peut être épuré aussi, c'est mon style — mélange." Concept "Néo-Cyberpunk Épuré" : squelette Apple (whitespace, typo, hiérarchie, animations cubic-bezier) + soul cyberpunk (accent jaune signature, vrai noir #000, glow subtle, restraint 70 % vide / 25 % structure / 5 % accent). Bar chart jaune pleine largeur sur fond clair, AskBox focused glow expressive, scan-line sweep 0.9 s au mount du result-card.

---

## Ce que j'observe

Multi-agent ne marche pas parce qu'on aurait trouvé "le bon framework". Ça marche parce qu'on traite la doctrine comme du code — elle se versionne, se débogue, se compresse — et parce qu'on fait confiance à l'autonomie sous condition de traçabilité brutale (events Kanban, screenshots, git).

Le coût d'un pivot devient marginal quand chaque agent peut reconstituer son état en lisant trois fichiers et trente events. La poudre aux yeux que vous voyez sur les visuels ICBI est assumée : c'est ce qui rend le produit démontrable. Mais derrière, il y a 32 livrables et trois pivots. Pas de magie, juste de la pratique.

Code et architecture détaillés : *[lien GitHub]*.

---

*#AgenticAI #MultiAgent #LLMOrchestration #BusinessIntelligence #Anthropic #DuckDB #VueJS #TypeScript #LocalFirst #PromptEngineering #SoftwareArchitecture*
