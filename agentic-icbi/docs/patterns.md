# Technical patterns discovered in practice

> Four patterns pulled from the ICBI build. Each one is shared in the same shape: symptom → root cause → fix → generic lesson.

---

## 1. Vue 3 `watch` + `immediate: true` for async lazy-loaded components

**Context**: a `DemoPlayer.vue` component loaded via `defineAsyncComponent`, mounted conditionally through `<Suspense v-if="demoActive">`, receiving an `:active="demoActive"` prop.

**Symptom**: the DemoPlayer state machine stays stuck at `idle`. The `DEMO 1/3` pill shows up (so the component is mounted), but `start()` is never called. No typing, no submit, nothing.

**Root cause**: a Vue 3 `watch()` without `immediate: true` does **not** fire on the initial value. In the `defineAsyncComponent + Suspense v-if` pattern, the component is mounted **with `active=true` already set** — there's never been a `false → true` transition for the watcher to observe.

**Fix**:

```ts
watch(() => props.active, (now) => {
  if (now) start();
  else cleanup();
}, { immediate: true });   // ← add this
```

**Generic lesson**: for any `defineAsyncComponent` or `<Suspense>`-wrapped component whose lifecycle depends on a prop that's already active at mount, **always** set `{ immediate: true }` on the `watch`. The idiomatic alternative is `onMounted(() => { if (props.active) start() })`, but the immediate watch is cleaner and symmetric for later transitions.

---

## 2. Sub-agent in parallel for data prep

**Context**: the Sprint 2 3D globe was mapping about fifty countries through a `NAME_TO_ISO` table inlined in `GlobeView.vue`. On three-country datasets (European employees), the sphere looked visually empty.

**Approach**: rather than have the main DEV type 250 entries while working on something else, the STRAT agent delegated to a generalist sub-agent launched in the background.

Precise brief:

> Generate a TypeScript file `country-coords.ts` with ~250 Natural Earth countries.
> Format: `{ iso3, iso2, name, nameAlt?[], lat, lng, population?, continent }`.
> Include 193 UN members + Vatican + Palestine + Taiwan + HK + Macao + 50 territories.
> Helpers: `COUNTRIES_BY_ISO3` Map (O(1) lookup), `findCountry(query)` fuzzy.
> nameAlt with FR variants + native names (Deutschland, Россия, 中国).
> Output: compiles under `bun build`, 0 errors.

**Result**: 250 entries shipped (AF:59, AS:52, EU:52, NA:40, OC:28, SA:14, AN:5), 31.78 KB ESM bundle, delivered in 199 seconds while the main DEV kept working on Wave 1.

**Generic lesson**: data prep, fixture generation, and repetitive scaffolding are perfect candidates for a sub-agent in background mode. The brief has to be **precise on format, quality constraints, and the acceptance condition** (compiles, runs, passes tests). You don't ask a sub-agent to "think" — you ask it to produce a measurable deliverable.

---

## 3. Anthropic 2000 px limit on the long edge

**Context**: a DEV agent was using headless Puppeteer with a `1600 × 1000` viewport and `deviceScaleFactor: 2` to produce Retina screenshots. Actual output: `3200 × 2000 px`.

**Symptom**: after a few captures injected into its context, the agent received:

```
An image in the conversation exceeds the dimension limit
for many-image requests (2000px). Run /compact to remove
old images from context, or start a new session.
```

The agent was stuck for two hours, unable to make progress.

**Recovery**: `/compact` (preserves the semantic context) rather than `/clear` (nuclear, loses everything). Before `/compact`, log a Kanban event + write the current status into `CLAUDE.md` so the recovery is clean.

**Proactive fix**:

```js
const VIEWPORT = { width: 1600, height: 1000, deviceScaleFactor: 1 };
//                                            ^^^^^^^^^^^^^^^^^^^^^
// dsf=1 by default, dsf=2 only when Retina is needed,
// then sharp/jimp downscale post-capture to 1800 px max.
```

**Generic lesson**: before injecting media into an LLM context, **check the provider limits** (Anthropic 2000px, OpenAI 20MB, Google 4MB depending on the API). A downscale at the source costs nothing and avoids opaque blockers.

---

## 4. Pivot resilience — cross-agent fallback

**Context**: during Sprint 2.7, the main DEV got stuck on the 2000px limit and couldn't recover. The STRAT noticed the inactivity (>2h, no Kanban events posted) and asked Nestor (the meta infra agent) for permission to step in.

**Procedure**:

1. **Trigger**: DEV stuck >30 min on an urgent task.
2. **Authorization**: explicit, from the peer STRAT (never unilateral).
3. **Scope**: STRAT can read and modify the DEV's code to unblock, with clear documentation of the intervention (Kanban events + comments in the code).
4. **Communication**: `FIX` event on Kanban + notify DEV (for handoff) + notify Nestor (audit trail).
5. **Preservation of the DEV workflow**: let DEV resume normally after the unblock. Don't keep working on their task once they're back.

**Concrete case**: STRAT read `DemoPlayer.vue`, identified H1 (`watch` without `immediate`), applied the one-line fix, rebuilt the frontend, ran Puppeteer dsf=1 to generate 4 validation screenshots, updated the Kanban. The DEV came back later and picked up Wave 2 (country-coords integration + enriched viz + dashboard) without having to redo the work.

**Generic lesson**: agentic fraternity without role rigidity. A STRAT can get their hands dirty when the DEV is out, as long as three invariants hold: explicit authorization, brutal traceability, handoff back on return.

---

## Patterns coming up

- **Systematic LLM bench** on a canonical question set (gpt-4.1-mini vs gpt-5-mini reasoning: 1.3 s vs 5.7 s, `max_completion_tokens` incompatible) — decision documented in the SYNAPSE.
- **Type-aware SQL prompt**: typed DuckDB schema (`DOUBLE`, `VARCHAR`, `BOOLEAN`, `TIMESTAMP`) injected into the system prompt, plus four few-shots covering each type. Drops `Conversion Error` to zero on the regression set.
- **Backwards-compatible migration Sprint N → Sprint N+1**: sha256-verified, idempotent, `ARCH_CHANGE` event logged. No data loss between Sprint 1 (monolithic DuckDB) and Sprint 2 (multi-profile).
