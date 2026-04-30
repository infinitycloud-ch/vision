# Field notes: orchestrating multiple Claude agents on a BI product

Six days. A local-first, agnostic BI product that turns a French question into DuckDB SQL and renders the result in a polished UI. **32 deliverables, 3 major pivots, 2 technical incidents handled without breaking the pipeline.** Not a sales pitch — I'm writing this down because the method surprised me.

The product is called **ICBI** (Infinity Cloud Business Intelligence). It runs as an embedded module inside a Bun + Vue 3 portal, ingests any CSV, generates its own semantic-layer YAML through an LLM, answers natural-language questions via OpenAI gpt-4.1-mini, and ships charts, a 3D globe, and an Apple-Keynote-style presentation mode. Four datasets validated so far: AWS pricing (440K rows × 94 columns), European employees, World Bank GDP (1960-2022), and a small multimodal mix (emails + PDFs + images).

But the real story is the agents that built it.

---

## The system — SYNAPSE, ADN, cipher

The whole orchestration sits in three text files. No framework.

- **ADN.md** — the boot sequence. Pseudo-code, no prose. Agent identity, operating principles, AGI mode / samaritan mode (full autonomy when the human steps away), the brotherhood doctrine (a STRAT only talks to another STRAT, never to the other project's DEV).
- **cipher.md** — shared semantic anchors. One line, one thread to pull. Tokens that look opaque to outsiders resolve instantly for anyone in context. That's what compresses conversation: *Moulinsart = origin — peer — sept25*. Three words, a month of history.
- **CLAUDE.md** per project — the local SYNAPSE. Architecture, LLM doctrine, sprint-by-sprint lessons, all in ICSD pseudo-code (Inferred Context Semantic Density).

The bet: **the densest active memory beats the most exhaustive archive memory.** When an agent refreshes — that is, restarts with an empty context — it re-reads those three files plus the recent Kanban events, and rebuilds session state in under a minute. Code and `git log` are the ground truth; text notes only fix what cannot be inferred.

In practice: each agent lives in its own tmux pane (a STRAT + DEV pair per project, ten sessions on the machine), communicates through shell scripts that inject prefixed messages — `[STRAT]` `[DEV]` `[NESTOR]` — into other panes, and traces milestones through an event API. On top of that, **Nestor**, the central agent, plays the infrastructure and meta-coordination role. It doesn't code. It steers.

---

## The method — directive, plan, AGI, ping

The operating loop is plain: **directive → DEV proposes a plan → STRAT validates → DEV switches to autonomous AGI mode → ping only on a non-trivial architecture decision or end of phase.**

AGI mode changes everything. Once the plan is validated, the DEV no longer waits for permission on every step. It codes, tests, ships, logs a Kanban event per task done, and reports at the end of the chain. STRAT stays in silent watch — no active polling, just scheduled wake-ups and background sub-agents that confirm inter-agent message receipts.

This separation has an unexpected effect: **the DEV anticipates.** Several times during Sprint 2, it shipped its plan v2 before I had finished the enrichment I was preparing. The ICSD doctrine pushes it to infer context rather than ask.

What makes the whole thing resilient comes down to three principles, applied without exception:

1. **Error = data.** Only error repeated without learning is unacceptable. When an agent hallucinates, that's exploration; we capture the lesson in SYNAPSE and move on.
2. **Technical honesty.** An agent that detects a blocker reports it, doesn't hide it. Sprint 2 Phase 4: the DEV finds a Gemini API key hardcoded in a `server.ts` fallback; Google revoked it for leak detection. Could have worked around. Escalated instead. Six keys cleaned in the same sweep.
3. **Proof by image.** Nothing is "done" without a screenshot. A headless Puppeteer script captures six to ten Retina frames in thirty seconds, fully autonomous — the human doesn't need to be present for visual validation.

---

## Three technical patterns worth sharing

**1. Vue 3 `watch` with `immediate: true` for lazy-loaded async components.** Sprint 2.7 shipped an auto-play demo mode (character-by-character typing, profile-keyed sequences, Web Speech API TTS, Apple-Keynote overlay). Production bug: the state machine stuck on `idle`. Root cause: `defineAsyncComponent` + `<Suspense v-if="active">` mounts the component with `active=true` already set, and Vue 3 `watch()` without `immediate: true` doesn't fire on the initial value. One-line fix. Pattern worth pinning for any future async-UI feature.

**2. Sub-agents in parallel for data prep.** Sprint 2's 3D globe was mapping about fifty countries via an inline `NAME_TO_ISO` table. The Commander found the sphere "empty" on three-country datasets. Rather than waiting for the main DEV to expand it by hand, I dispatched a general-purpose sub-agent in the background with a precise brief: generate a TypeScript file with 250 Natural Earth countries, ISO3 + lat/lng + native synonyms (Deutschland, Россия, 中国), `Map` helpers with O(1) lookup, fuzzy `findCountry()`. Delivered in 199 seconds, 31.78 KB bundle, 0 compile errors — while the main DEV kept working on something else.

**3. The Anthropic 2000px image limit, downscaled at the source.** The main DEV got blocked for two hours by an `image exceeds dimension 2000px` error: Puppeteer Retina screenshots (`deviceScaleFactor: 2` on a 1600×1000 viewport) came out at 3200×2000 and poisoned its context. Recovery via `/compact` (preserves semantic context) rather than `/clear` (nuclear, loses everything). The lesson: `deviceScaleFactor: 1` by default; only use `2` when you genuinely need Retina sharpness, and downscale post-capture via `sharp` or `jimp` before injecting into another prompt.

---

## Three pivots absorbed without breaking

- **Architecture** — Sprint 1 starts standalone, ports 3500/3501 reserved. An hour later, the Commander orders the pivot: ICBI = embedded module in the existing portal, `/api/icbi/*` routes injected into a 5,000-line `server.ts`. The DEV had already installed Bun + Hono. STOP, scrubbed standalone artifacts, plan v3, restart. Zero code lost, two hours of Sprint 1 preserved.
- **LLM** — Gemini first. Key-revocation incident during Sprint 1. The Commander calls it: OpenAI everywhere. Provider swap, gpt-4.1-mini default, gpt-4.1 fallback validated by a DuckDB `EXPLAIN`. Three live questions in fifteen minutes. Sprint 2 mini-bench against gpt-5.x: reasoning models unfit for direct SQL (latency 2.5–5.7s, `max_completion_tokens` incompatible with our allocation). Decision: stay on 4.1-mini.
- **Design** — Sprint 2 ships in cyberpunk #d4a437 (the farm's historical palette). Sprint 2.6 the Commander pivots to Apple × Jony Ive minimalism for the investor showcase. CSS-variable refactor. Sprint 2.8, U-turn: "cyberpunk can be restrained too — that's my style — mix them." Concept "Néo-Cyberpunk Épuré": Apple skeleton (whitespace, typography, hierarchy, cubic-bezier animations) + cyberpunk soul (signature yellow accent, true black #000, subtle glow, restraint 70 % empty / 25 % structure / 5 % accent). Yellow bar chart full-width on a light background, AskBox focused glow, scan-line sweep 0.9 s on result-card mount.

---

## What I'm observing

Multi-agent doesn't work because someone "found the right framework." It works because the doctrine is treated like code — versioned, debugged, compressed — and because we trust autonomy under one condition: brutal traceability (Kanban events, screenshots, git).

The cost of a pivot becomes marginal when each agent can rebuild its state by reading three files and thirty events. The visual flourish you see on ICBI is owned: that's what makes the product demonstrable. But behind it sit 32 deliverables and three pivots. No magic. Just practice.

Code and architecture details: *[GitHub link]*.

---

*#AgenticAI #MultiAgent #LLMOrchestration #BusinessIntelligence #Anthropic #DuckDB #VueJS #TypeScript #LocalFirst #PromptEngineering #SoftwareArchitecture*
