# Patterns techniques découverts en pratique

> Quatre patterns extraits du chantier ICBI. Chacun est partagé sous forme : symptôme → cause racine → fix → leçon générique.

---

## 1. Vue 3 `watch` + `immediate: true` pour composants async lazy-loadés

**Contexte** : composant `DemoPlayer.vue` chargé via `defineAsyncComponent`, monté conditionnellement via `<Suspense v-if="demoActive">`, reçoit une prop `:active="demoActive"`.

**Symptôme** : la state machine du DemoPlayer reste bloquée à `idle`. Le pill `DEMO 1/3` s'affiche (le composant est bien monté), mais `start()` n'est jamais appelé. Pas de typing, pas de submit, rien.

**Cause racine** : Vue 3 `watch()` sans `immediate: true` ne tire **pas** sur la valeur initiale. Or, dans le pattern `defineAsyncComponent + Suspense v-if`, le composant est monté **avec `active=true` déjà présent** — il n'y a jamais eu de transition `false → true` que le watcher pourrait observer.

**Fix** :

```ts
watch(() => props.active, (now) => {
  if (now) start();
  else cleanup();
}, { immediate: true });   // ← ajouter ceci
```

**Leçon générique** : pour tout composant `defineAsyncComponent` ou `<Suspense>`-wrapped dont le cycle de vie dépend d'une prop déjà active au mount, **toujours** poser `{ immediate: true }` sur le `watch`. L'alternative idiomatique est `onMounted(() => { if (props.active) start() })`, mais le watch immediate est plus propre et symétrique pour les transitions ultérieures.

---

## 2. Sub-agent en parallèle pour data prep

**Contexte** : le globe 3D du Sprint 2 cartographiait une cinquantaine de pays via une table `NAME_TO_ISO` inline dans `GlobeView.vue`. Sur des datasets à trois pays (employés européens), la sphère apparaissait visuellement vide.

**Approche** : plutôt que de faire taper 250 entrées au DEV principal pendant qu'il travaillait sur autre chose, l'agent STRAT a délégué à un sub-agent généraliste lancé en background.

Brief précis :

> Génère un fichier TypeScript `country-coords.ts` avec ~250 pays Natural Earth.
> Format : `{ iso3, iso2, name, nameAlt?[], lat, lng, population?, continent }`.
> Inclure 193 membres ONU + Vatican + Palestine + Taiwan + HK + Macao + 50 territoires.
> Helpers : `COUNTRIES_BY_ISO3` Map (lookup O(1)), `findCountry(query)` fuzzy.
> nameAlt avec variations FR + natives (Deutschland, Россия, 中国).
> Sortie : compile sous `bun build`, 0 erreurs.

**Résultat** : 250 entrées balancées (AF:59, AS:52, EU:52, NA:40, OC:28, SA:14, AN:5), bundle 31.78 KB ESM, livré en 199 secondes pendant que le DEV principal continuait Wave 1.

**Leçon générique** : les tâches de data prep, génération de fixtures, scaffolding répétitif sont des candidats parfaits pour un sub-agent en mode background. Le brief doit être **précis sur le format, les contraintes de qualité, et la condition d'acceptation** (compile, runs, passes tests). On ne demande pas un sub-agent de "réfléchir" — on lui demande de produire un livrable mesurable.

---

## 3. Limite Anthropic 2000 px côté le plus long

**Contexte** : un agent DEV utilisait Puppeteer headless avec viewport `1600 × 1000` et `deviceScaleFactor: 2` pour produire des screenshots Retina. Output réel : `3200 × 2000 px`.

**Symptôme** : après quelques captures injectées dans son contexte, l'agent a reçu :

```
An image in the conversation exceeds the dimension limit
for many-image requests (2000px). Run /compact to remove
old images from context, or start a new session.
```

L'agent était bloqué pendant deux heures, incapable d'avancer.

**Reprise** : `/compact` (préserve le contexte sémantique) plutôt que `/clear` (nucléaire, perd tout). Avant `/compact`, log d'event Kanban + écriture dans `CLAUDE.md` du status courant pour permettre la reprise propre.

**Fix proactif** :

```js
const VIEWPORT = { width: 1600, height: 1000, deviceScaleFactor: 1 };
//                                            ^^^^^^^^^^^^^^^^^^^^^
// dsf=1 par défaut, dsf=2 uniquement si Retina nécessaire,
// puis sharp/jimp downscale post-capture à 1800 px max.
```

**Leçon générique** : avant d'injecter des médias dans un contexte LLM, **vérifier les limites du fournisseur** (Anthropic 2000px, OpenAI 20MB, Google 4MB selon API). Un downscale à la source coûte rien et évite des blocages opaques.

---

## 4. Pivot resilience — fallback inter-agent

**Contexte** : pendant Sprint 2.7, le DEV principal s'est retrouvé bloqué par la limite 2000px et n'arrivait pas à reprendre. Le STRAT a constaté l'inactivité (>2h, aucun event Kanban posté) et a demandé à Nestor (l'agent infra méta) l'autorisation d'intervenir.

**Procédure** :

1. **Trigger** : DEV bloqué >30 min sur tâche urgente.
2. **Autorisation** : explicite par STRAT homologue (jamais unilatérale).
3. **Scope** : STRAT peut lire et modifier le code du DEV pour débloquer, avec documentation claire de l'intervention (events Kanban + comments dans le code).
4. **Communication** : event `FIX` sur Kanban + notify DEV (pour reprise) + notify Nestor (audit trail).
5. **Préservation du workflow DEV** : laisser DEV reprendre normalement post-déblocage. Ne pas continuer son travail si lui revient.

**Cas concret** : STRAT a lu `DemoPlayer.vue`, identifié H1 (`watch` sans `immediate`), appliqué le fix d'une ligne, rebuild frontend, lancé Puppeteer dsf=1 pour générer 4 screenshots de validation, mis à jour le Kanban. Le DEV est revenu plus tard et a repris la Wave 2 (intégration country-coords + viz enrichies + dashboard) sans avoir à refaire le travail.

**Leçon générique** : la fraternité agentique sans rigidité de rôle. Un STRAT peut se mouiller les mains quand le DEV est out, à condition de respecter trois invariants : autorisation explicite, traçabilité brutale, restitution dès retour.

---

## Patterns à venir

- **Bench LLM systématique** sur un set de questions canoniques (gpt-4.1-mini vs gpt-5-mini reasoning : 1.3 s vs 5.7 s, `max_completion_tokens` incompatible) — décision documentée dans la SYNAPSE.
- **Type-aware SQL prompt** : schema typé DuckDB (`DOUBLE`, `VARCHAR`, `BOOLEAN`, `TIMESTAMP`) injecté dans le system prompt, plus quatre few-shots couvrant chaque type. Réduit les `Conversion Error` à zéro sur le set régression.
- **Migration rétro-compatible Sprint N → Sprint N+1** : sha256-verified, idempotent, log d'event ARCH_CHANGE. Aucune perte de données entre Sprint 1 (DuckDB monolithique) et Sprint 2 (multi-profils).
