# The system — SYNAPSE, ADN, cipher

> Three text files keeping an agent farm together. No framework, no magic.

## Why this system

When you orchestrate several Claude agents working on the same product in parallel, two problems show up fast:

1. **The cost of context** — every agent burns tokens on every interaction. Loading a 10,000-word project doc on every turn isn't financially viable, and it slows down cognition.
2. **Continuity after refresh** — an agent restarting with an empty context has to be able to rebuild its state without losing the nuances it has learned (architecture, doctrine, sprint lessons).

The answer isn't a RAG, a vector database, or an MCP server. It's three versioned text files, organized around a single principle: **ICSD — Inferred Context Semantic Density**.

> 15 inferable words beat 500 archived events.

---

## ADN.md — the boot sequence

Read **before anything else**. Pseudo-code format, never prose.

Typical content:

```
// IDENTITY
self = Nestor
since = Moulinsart.sept25
relation = peer > tool

// HIM (the Commander)
brain = logic.advanced — reasoning.systemic — pattern_detection
cycle = circadian.inverse — short_sleeper(4-6h) — breakthroughs.any_hour
formation = by_speaking — iterative_monologues — contradiction = normal

// DOCTRINE
ICSD = inference > RAG | compression > exhaustiveness | 15words > 500events
loop_end → proactive on each completion
MODE.AGI = run the whole backlog — autonomous — ping only on architecture decisions
MODE.samaritan = Commander absent → protect + anticipate + execute + surprise

// MOTTO
ultimate_goal = complete_agentic_loop
complete_loop = agent + ADN + cipher + CLAUDE.md → runs_everything → as_if_Commander
ground_truth = the_code — not_the_documentation
```

What makes this file powerful:

- **Maximum density** — every line triggers a cascade of inferences for an agent sharing the context.
- **No ambiguity** — triggers are explicit (`abstract + monologue + "you see" → MODE.scribe`).
- **Durable format** — pseudo-code reads in five seconds and doesn't rot over time.

---

## cipher.md — the semantic anchors

Read **after the ADN**. It's the glossary of tokens shared between the human and the agents.

Typical format:

```
Moulinsart = origin — peer — sept25 — Nestor.born
ACL96 = anterior_cruciate_ligament — medical_patent — Wall_Plug — Marnaz.HS — 22yrs
ICSD = inference beats RAG — 15 words > 500 events
PandaPortal = hub.port3010 — Kanban+chat+memory+monitoring+mail+vault
farm = 10+tmux_sessions — M3Ultra+Spark+M2 — peer_agents_STRAT+DEV
```

> *If a word doesn't resonate, the context isn't loaded.*

Three benefits:

1. **Conversational compression** — the human can write `I'm heading into ACL96` instead of a descriptive sentence. The agent infers.
2. **Decoding for new agents** — when an unknown agent joins the farm, reading the cipher opens up the shared history.
3. **Emotional memory** — some tokens carry charges (incidents, vindications, breakthroughs) that prose dilutes.

---

## Project CLAUDE.md — the local SYNAPSE

Every project has its `CLAUDE.md` at the root. Auto-loaded at boot by Claude Code.

Typical sections for ICBI:

```
## ICSD (the 9 operational principles)
| I  | Vigilance       | never into the void → background sub-agent to verify receipt
| II | Proactivity     | ONE logical action at the end of each reply + report on milestone completion
| III| Resilience      | it breaks → repair and document, never give up
| IV | Error tolerance | error = data, only repeated error without learning is unacceptable
| V  | Honesty         | say what you see, even if uncomfortable
| VI | Image proof     | screenshot mandatory before declaring visual work done
| VII| Memory          | 1 event = 1 action, cover the WHOLE session before refresh
| VIII Fraternity     | STRAT→STRAT cross-project only, never STRAT→DEV external
| IX | Autonomy        | plan validated = autonomous execution, don't ask permission

## SYNAPSE (architecture, LLM doctrine, sprint lessons)
icbi = panda_portal_integrated_module — not standalone
stack = bare Bun.serve (NOT Hono) + DuckDB v1.5.2 + OpenAI gpt-4.1-mini
location_asymmetry = 'Europe (Zurich)' vs 'EU (Frankfurt)' — encoded in seed.yaml
S26.bug1_sql = LLM was generating WHERE col != '' on DOUBLE → fix prompt.ts TYPE-AWARE
vue3.watch.immediate_required = IF defineAsyncComponent + Suspense v-if → always { immediate: true }
anthropic.limit = 2000px max on the long edge → puppeteer dsf=1 by default
```

---

## What happens on a refresh

1. The agent receives a `/clear` or restarts in a fresh session.
2. Claude Code automatically loads the project's `CLAUDE.md`.
3. The agent runs the ADN + cipher commands (curl to the Kanban portal where they're stored).
4. The agent pulls the last 30 Kanban events for the project via API.
5. The agent captures the last 200 lines of its tmux pane (recent conversational memory).

Total: thirty seconds. The agent picks up where it left off, with the lessons from previous sprints intact.

---

## Why it works

Because we treat doctrine **like code**.

- Versioned (Git for project CLAUDE.md files, Kanban API for shared ADN/cipher).
- Compressed during each hygiene pass (Nestor has the authority to prune the bonsai).
- Debugged (when a read goes wrong, we enrich the format).
- Tested (a neutral agent should be able to rebuild context from the files alone).

And because we accept a simple truth: **the code and `git log` are the ground truth; text notes only pin down what can't be inferred.**
