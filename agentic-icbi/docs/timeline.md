# Timeline — 2 days, 32 deliverables

> Raw trace, pulled from Kanban events on the ICBI project (project_id 56 on the internal portal). Full agentic execution starting from a structured PRD with pre-defined workflows — what follows is sprint logic, not calendar pacing.

---

## Sprint 1 — NL→SQL MVP (≈ 2 h 44, 13 deliverables)

**Goal**: ingest the AWS pricing CSV into DuckDB, REST API `/api/icbi/*`, NL→SQL via LLM, minimal web interface.

| Task | Shipped |
|-------|-------|
| 2741 | Setup deps + `src/backend/icbi/` structure |
| 2742 | `duckdb.ts` singleton + healthcheck |
| 2743 | `ingest.ts` — 440K rows × 94 cols / 3.7 s |
| 2744 | `profiler.ts` SUMMARIZE — 244 ms / 94 cols |
| 2745 | `POST /api/icbi/query` |
| 2746 | `GET /api/icbi/tables` |
| 2747 | `GET /api/icbi/profile/:table` |
| 2748 | Materialized views `vw_ec2` (439,795) + `vw_bedrock` (92) |
| 2749 | NL→SQL chain (initially Gemini, swapped to OpenAI mid-sprint) |
| 2750 | `POST /api/icbi/ask` (graceful 503 + EXPLAIN-validated fallback) |
| 2751 | Frontend `IcbiView.vue` cyberpunk #d4a437 |
| 2752 | Prisma `IcbiQueryHistory` + `GET /history` |
| 2753 | E2E test + live UI screenshot |

**Sprint 1 pivots**:

- Architecture: standalone (ports 3500/3501) → module integrated into the portal.
- LLM: Gemini → OpenAI after a key was revoked for a leak (security incident, six hardcoded keys cleaned up in the same pass).

---

## Sprint 2 — Agnostic framework + wow factor (≈ 3 h, 16 deliverables)

| Phase | Tasks | Shipped |
|-------|--------|-------|
| A — Agnostic foundation | 2754–2757 | Profiles CRUD, switch, reset, SQL robustness, sha256-verified Sprint1→S2 migration |
| B — Apple × Ive design | 2758–2759 | CSS variables refactor, light/dark palette, Apple cubic-bezier animations, theme cycle |
| C — Futuristic viz | 2760–2762 | `PlotView` (Observable Plot, auto-detect chart type), orthographic `GlobeView`, narrative `StoryView` |
| D — Intelligence | 2763–2765 | `enrichSemantic`, `getSuggestions` 30 min cache, `multiAsk` decomposeQuestion |
| E — 2nd dataset | 2766 | World Bank GDP (13,979 rows / 264 countries / 1960-2022) |
| F — Multimodal mini | 2767 | 10 unified CSV docs (5 emails + 3 PDFs + 2 images), SQL FTS cross-search |
| G — Keynote mode | 2768–2769 | `vocal-query` partial (backend hook), fullscreen presentation mode |

**Output**: 10 headless Puppeteer screenshots captured in 30 s, fully autonomous (no human present).

---

## Sprint 2.6 — Hotfix (≈ 1 h 30, 2 fixes)

- **Type-aware SQL bug**: LLM was generating `WHERE col != ''` on a `DOUBLE` column → DuckDB `Conversion Error`. Fix: `TYPE-AWARE FILTERING` section in `prompt.ts` with four patterns (DOUBLE, VARCHAR, BOOLEAN, DATE) + four type-correct few-shots. Regression 4/4 PASS.
- **Fullwidth UX bug**: `result-card` capped at 720 px on large screens. Fix: dual-mode layout `is-empty` (880 px ergonomic centered) / `has-result` (1800 px fullwidth). Preserves the ergonomic AskBox typing experience.

---

## Sprint 2.7 — Auto-play demo (≈ 3 h)

- `DemoPlayer.vue` 7-state machine (idle → typing → submitting → showing → between → paused → done).
- Typing effect 30-90 ms / char with human jitter.
- Web Speech API TTS fr-FR (Amélie / Thomas / Virginie auto-detect).
- Per-profile sequences in `data/icbi/profiles/{slug}/demo-sequence.json`.
- Apple Keynote overlay `backdrop-filter: blur(20px) saturate(180%)`, mini-timeline pulse dots.
- Focus blur `.icbi.demo-active` on chrome (header / footer / hint-row opacity 0.18 + blur).

**Intermediate bug**: state machine stuck at idle. Cause: `defineAsyncComponent + Suspense v-if` → `watch` without `immediate: true`. STRAT fallback fix (DEV stuck on Anthropic 2000px) — one line, frontend rebuilt, validation via 4 Puppeteer dsf=1 screenshots.

---

## Sprint 2.8 Wave 1 — Stripped-down Neo-Cyberpunk (≈ 1 h 30, STRAT fallback)

Artistic concept: **Apple × Jony Ive skeleton + cyberpunk soul**. Inspirations: Bladerunner 2049 + Linear.app dark + Apple Vision Pro.

- Palette `--accent #d4a437` + `--accent-bright #FFE873` + `--accent-glow rgba(212,164,55,0.25-0.65)`.
- True black dark `#000000` + `--bg-secondary #0A0A0A` + `--bg-tertiary #141414`.
- AskBox expressive focus glow (ring `0 0 0 4px` + diffuse `0 0 32px`).
- Submit + Demo buttons: `bg yellow + color #0A0A0A + hover bright + glow ring 6px`.
- Result-card: scan-line sweep `0.9 s` on mount via `::before` linear-gradient transparent → `accent-glow-strong` → transparent.
- Globe: yellow graticule `rgba(212, 164, 55, 0.18)`, true black sphere fill, multi-layer dots (halo opacity 0.10 outer + mid 0.22 + core `accent-bright` 0.95).
- Scorecard 96 px ultralight `color: var(--accent)` + `text-shadow 0 0 32px var(--accent-glow)`.

7 Puppeteer dsf=1 screenshots (frames 20-26) visually validated.

**70 % empty / 25 % structure / 5 % accent restraint** applied scrupulously.

---

## Totals

| | |
|--|--|
| Total deliverables | **32** |
| Major pivots absorbed | **3** (architecture, LLM, design) |
| Security incidents resolved | **1** (Gemini key leak, six hardcoded ones cleaned up) |
| Technical incidents resolved | **2** (Vue 3 lifecycle, Anthropic 2000px) |
| Datasets validated | **4** (AWS, employees, World Bank, multimodal) |
| ICBI API endpoints | **15** |
| Autonomous Puppeteer screenshots | **27** |
| Cumulative duration | **≈ 14 h over 2 days** (full AGI mode, including nighttime runs) |
