# ICBI — Infinity Cloud Business Intelligence

> Local-first agnostic BI. NL → SQL → visual result, no cloud, no vendor lock-in.

![ICBI bar chart hero](./media/01-hero-bar-chart-light.png)

**Stack** — Bun · DuckDB v1.5.2 · OpenAI gpt-4.1-mini · Vue 3 · Observable Plot · TypeScript

**Status** — Sprint 1 (13/13) + Sprint 2 (16/16) + Sprint 2.6 (2 fixes) + Sprint 2.7 (demo mode) + Sprint 2.8 Wave 1 (Néo-Cyberpunk Épuré design) — **32 deliverables in 6 days**, 3 major pivots absorbed.

---

## What ICBI does

- **Ingests a CSV** (validated up to 440K rows × 94 columns in 3.7 s).
- **Auto-profiles columns** (DuckDB types, cardinality, distributions, NULL %).
- **Generates the semantic-layer YAML** through an LLM (descriptions, business glossary, dimensions/measures).
- **Converts an NL question into type-aware DuckDB SQL** (DOUBLE → `> 0`, VARCHAR → `!= ''`, BOOLEAN → `= TRUE`, DATE → `BETWEEN`).
- **Renders the right visual** — bar / line / scatter / heatmap / scorecard / 3D orthographic globe.
- **Auto-play demo mode** — typing effect, profile-keyed sequences, Web Speech API TTS (Amélie FR-FR), Apple-Keynote focus blur.
- **Fullscreen presentation mode** — true black #000, keyboard navigation, pre-computed slides.
- **Multi-profile** — 4 datasets validated in parallel (AWS pricing, employees, World Bank GDP, multimodal mini), switchable without restart.

---

## Gallery

| | |
|--|--|
| ![Bar chart light](./media/01-hero-bar-chart-light.png) | ![Globe dark](./media/02-globe-dark.png) |
| **Apple + cyberpunk bar chart** — signature yellow, full-width, restraint | **3D globe** — orthographic, true black, multi-layer dot glow |
| ![Scorecard](./media/03-scorecard-zurich.png) | ![Demo TTS](./media/04-demo-tts.png) |
| **96 px ultralight scorecard** — accent color + text-shadow glow | **Demo + TTS mode** — Amélie voice, Apple-Keynote pill controls |
| ![Demo progress](./media/05-demo-progress.png) | ![Bar chart loading](./media/06-bar-chart-loading.png) |
| **Demo state machine** — Q1→Q2 progression, smooth transitions | **Clean loading state** — SF segmented spinner |

[`demo-icbi.mp4`](./media/demo-icbi.mp4) — 28 s, 1280×800, 30 fps (333 KB)

---

## Documents

- [`docs/linkedin-article.md`](./docs/linkedin-article.md) — LinkedIn article, ~1100 words (method, patterns, pivots)
- [`docs/synapse-adn-cipher.md`](./docs/synapse-adn-cipher.md) — The compressed memory system
- [`docs/patterns.md`](./docs/patterns.md) — Technical patterns discovered in practice
- [`docs/timeline.md`](./docs/timeline.md) — 6-day timeline, 32 deliverables

---

## Architecture (summary)

```
panda-portal/
├── server.ts                       # bare Bun.serve, /api/icbi/* routes inside a wrapped block
├── src/backend/
│   ├── icbi/
│   │   ├── duckdb.ts               # per-profile singleton, JSON-safe (BigInt → number)
│   │   ├── ingest.ts               # COPY CSV → DuckDB + auto-index on low-cardinality columns
│   │   ├── profiler.ts             # SUMMARIZE-based, 244 ms / 94 cols
│   │   ├── views.ts                # per-profile materialized views
│   │   ├── profiles/               # CRUD + sha256-verified Sprint1→S2 migration
│   │   │   ├── manager.ts
│   │   │   ├── types.ts
│   │   │   └── migrate.ts
│   │   ├── semantic/
│   │   │   ├── seed.yaml           # Git-versioned (descriptions + glossary)
│   │   │   ├── auto-generator.ts   # LLM generates seed.yaml from profileTable
│   │   │   ├── generator.ts        # merges seed + profiling → runtime YAML
│   │   │   └── loader.ts
│   │   └── llm/
│   │       ├── openai.ts           # askWithFallback gpt-4.1-mini → gpt-4.1
│   │       └── prompt.ts           # type-aware system prompt + DuckDB few-shots
│   └── services/
│       └── icbiAnalyticsService.ts # orchestrator, errorKind, 30 s query timeout
└── src/frontend/src/
    ├── views/IcbiView.vue          # dual-mode layout (is-empty 880px / has-result 1800px)
    └── components/icbi/
        ├── DemoPlayer.vue          # 7-state machine, typing effect, TTS
        ├── PlotView.vue            # auto-detect chart type (Observable Plot)
        ├── GlobeView.vue           # orthographic projection + multi-layer dot glow
        ├── StoryView.vue           # multi-step LLM narration
        └── country-coords.ts       # 250 Natural Earth countries, ISO3+lat/lng, O(1) Maps
```

**API routes** — `/api/icbi/{health, tables, profile/:t, query, ask, semantic, history, profiles, profiles/:slug/{activate, suggestions, enrich}, multi-ask, story, vocal-query, demo/sequence, reset}` (15 endpoints).

---

## Measured performance (AWS pricing 440K × 94)

| Step | Latency |
|------|---------|
| Ingest 272 MB CSV | 3.7 s |
| Profile 94 cols (SUMMARIZE) | 244 ms |
| Build views vw_ec2 (439,795 rows) | 44 ms |
| Build view vw_bedrock (92 rows) | 1 ms |
| `/ask` gpt-4.1-mini (~5K prompt tokens) | 1.6–3.1 s |
| `/ask` gpt-4.1 fallback (EXPLAIN-validated) | 0.6 s |
| DuckDB memory bench across 4 active profiles | < 200 MB RSS |

---

## Pivots absorbed

1. **Architecture, Sprint 1** — standalone (ports 3500/3501) → portal-embedded module. One hour after the initial briefing. No code lost.
2. **LLM, Sprint 1 → Sprint 2** — Gemini → OpenAI after a key-revocation incident (leak). Provider swap in 15 minutes, gpt-4.1-mini default + gpt-4.1 EXPLAIN-validated fallback.
3. **Design, Sprint 2 → 2.6 → 2.8** — cyberpunk → Apple × Jony Ive → Néo-Cyberpunk Épuré (a marriage of both). CSS-variable refactor + globe + chart + buttons + scan-line. Validated by 7 Puppeteer dsf=1 screenshots.

---

## License

This documentation is published as a record of practice. ICBI's source code remains in the private portal that hosts it.
