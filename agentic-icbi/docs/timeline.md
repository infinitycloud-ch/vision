# Chronologie — 6 jours, 32 livrables

> Tracé brut, extrait des events Kanban du projet ICBI (project_id 56 sur le portail interne).

---

## Sprint 1 — MVP NL→SQL (≈ 2 h 44, 13 livrables)

**Goal** : ingestion AWS pricing CSV dans DuckDB, API REST `/api/icbi/*`, NL→SQL via LLM, interface web minimale.

| Tâche | Livré |
|-------|-------|
| 2741 | Setup deps + structure `src/backend/icbi/` |
| 2742 | `duckdb.ts` singleton + healthcheck |
| 2743 | `ingest.ts` — 440K rows × 94 cols / 3.7 s |
| 2744 | `profiler.ts` SUMMARIZE — 244 ms / 94 cols |
| 2745 | `POST /api/icbi/query` |
| 2746 | `GET /api/icbi/tables` |
| 2747 | `GET /api/icbi/profile/:table` |
| 2748 | Vues matérialisées `vw_ec2` (439 795) + `vw_bedrock` (92) |
| 2749 | NL→SQL chain (initialement Gemini, swapped vers OpenAI mid-sprint) |
| 2750 | `POST /api/icbi/ask` (graceful 503 + fallback EXPLAIN-validated) |
| 2751 | Frontend `IcbiView.vue` cyberpunk #d4a437 |
| 2752 | Prisma `IcbiQueryHistory` + `GET /history` |
| 2753 | Test E2E + screenshot UI live |

**Pivots Sprint 1** :

- Architecture : standalone (ports 3500/3501) → module intégré au portail.
- LLM : Gemini → OpenAI suite à révocation clé pour leak (incident sécurité, six clés hardcodées nettoyées dans la foulée).

---

## Sprint 2 — Framework agnostique + wow factor (≈ 3 h, 16 livrables)

| Phase | Tâches | Livré |
|-------|--------|-------|
| A — Socle agnostique | 2754–2757 | Profils CRUD, switch, reset, robustesse SQL, migration sha256-verified Sprint1→S2 |
| B — Design Apple × Ive | 2758–2759 | Refonte CSS variables, palette light/dark, animations cubic-bezier Apple, theme cycle |
| C — Viz futuriste | 2760–2762 | `PlotView` (Observable Plot, auto-detect chart type), `GlobeView` orthographic, `StoryView` narrative |
| D — Intelligence | 2763–2765 | `enrichSemantic`, `getSuggestions` cache 30 min, `multiAsk` decomposeQuestion |
| E — 2ème dataset | 2766 | World Bank GDP (13 979 rows / 264 pays / 1960-2022) |
| F — Multimodal mini | 2767 | 10 docs unifiés CSV (5 emails + 3 PDF + 2 images), cross-search SQL FTS |
| G — Keynote mode | 2768–2769 | `vocal-query` partial (hook backend), mode présentation fullscreen |

**Sortie** : 10 screenshots Puppeteer headless capturés en 30 s, totalement autonomes (sans présence humaine).

---

## Sprint 2.6 — Correctif (≈ 1 h 30, 2 fix)

- **Bug SQL type-aware** : LLM générait `WHERE col != ''` sur colonne `DOUBLE` → DuckDB `Conversion Error`. Fix : section `TYPE-AWARE FILTERING` dans `prompt.ts` avec quatre patterns (DOUBLE, VARCHAR, BOOLEAN, DATE) + four few-shots type-correct. Régression 4/4 PASS.
- **Bug UX fullwidth** : `result-card` plafonné à 720 px sur grand écran. Fix : layout dual-mode `is-empty` (880 px centré ergonomique) / `has-result` (1800 px fullwidth). Préserve typing AskBox ergonomique.

---

## Sprint 2.7 — Démo auto-play (≈ 3 h)

- `DemoPlayer.vue` state machine 7 états (idle → typing → submitting → showing → between → paused → done).
- Typing effect 30-90 ms / char jitter humain.
- TTS Web Speech API fr-FR (Amélie / Thomas / Virginie auto-detect).
- Sequences par profil dans `data/icbi/profiles/{slug}/demo-sequence.json`.
- Overlay Apple Keynote `backdrop-filter: blur(20px) saturate(180%)`, mini-timeline dots pulse.
- Focus blur `.icbi.demo-active` sur chrome (header / footer / hint-row opacity 0.18 + blur).

**Bug intermédiaire** : state machine bloquée à idle. Cause `defineAsyncComponent + Suspense v-if` → `watch` sans `immediate: true`. Fix STRAT fallback (DEV bloqué Anthropic 2000px) — une ligne, frontend rebuilt, validation 4 screenshots Puppeteer dsf=1.

---

## Sprint 2.8 Wave 1 — Néo-Cyberpunk Épuré (≈ 1 h 30, STRAT fallback)

Concept artistique : **squelette Apple × Jony Ive + soul cyberpunk**. Inspirations : Bladerunner 2049 + Linear.app dark + Apple Vision Pro.

- Palette `--accent #d4a437` + `--accent-bright #FFE873` + `--accent-glow rgba(212,164,55,0.25-0.65)`.
- Dark vrai noir `#000000` + `--bg-secondary #0A0A0A` + `--bg-tertiary #141414`.
- AskBox focus glow expressive (ring `0 0 0 4px` + diffuse `0 0 32px`).
- Submit + Demo buttons : `bg jaune + color #0A0A0A + hover bright + glow ring 6px`.
- Result-card : scan-line sweep `0.9 s` au mount via `::before` linear-gradient transparent → `accent-glow-strong` → transparent.
- Globe : graticule jaune `rgba(212, 164, 55, 0.18)`, sphere fill vrai noir, dots multi-layer (halo opacity 0.10 outer + mid 0.22 + core `accent-bright` 0.95).
- Scorecard 96 px ultralight `color: var(--accent)` + `text-shadow 0 0 32px var(--accent-glow)`.

7 screenshots Puppeteer dsf=1 (frames 20-26) validés visuellement.

**Restraint 70 % vide / 25 % structure / 5 % accent** appliqué scrupuleusement.

---

## Cumul

| | |
|--|--|
| Livrables totaux | **32** |
| Pivots majeurs absorbés | **3** (architecture, LLM, design) |
| Incidents sécurité résolus | **1** (clé Gemini leak, six hardcodées nettoyées) |
| Incidents techniques résolus | **2** (Vue 3 lifecycle, Anthropic 2000px) |
| Datasets validés | **4** (AWS, employés, World Bank, multimodal) |
| Endpoints API ICBI | **15** |
| Screenshots Puppeteer autonomes | **27** |
| Durée cumulée | **≈ 14 h sur 6 jours** (avec mode AGI nocturne) |
