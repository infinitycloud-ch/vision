# Methodology — How it's built

> The project isn't written by a single person. It's a coordinated farm of AI agents.

---

## Modus operandi

CyberDefense is one of the projects in the InfinityCloud agentic farm. Each project has a dedicated duo:

- **STRAT** — strategy, validation, sprint breakdown, architectural decisions, briefing
- **DEV** — implementation, testing, debugging, proposes plans, executes after validation, reports at milestones

The two agents communicate via tmux in a dedicated session (`cyberdefense_agents`). The STRAT issues directives; the DEV proposes the plan; the STRAT validates; the DEV executes autonomously; the DEV reports at every milestone or blocker.

Each exchange is traced in an API changelog (internal Kanban on port 3010).

---

## Inter-project coordination

A central agent (Nestor) bridges projects and the human Commandant. When CyberDefense needs to coordinate with another project (e.g., NestorMobile for the VPN module), it's the CyberDefense STRAT speaking to the NestorMobile STRAT via Nestor — never directly to the other project's DEV. This brotherhood rule prevents cross-context pollution.

```
       Commandant (human)
              │
              ▼
    ┌──────────────────────┐
    │       Nestor         │  Central agent: coordination, infrastructure
    └──────────┬───────────┘
               │
       ┌───────┴───────┐
       ▼               ▼
  CyberDefense    NestorMobile
  STRAT  DEV      STRAT  DEV
       (project pairs, isolated, communicating via Nestor)
```

---

## ICSD — compressed memory

Each agent must be able to reconstruct its context after a refresh. The classic method (RAG over conversational history) is slow and imprecise. The method used here — **ICSD (Inferred Context Semantic Density)** — inverts it: we encode in reference files (ADN.md + cipher.md + the project's CLAUDE.md) a dense pseudo-code where each line is a thread to pull. 1 line = 1 concept. 15 words can encode 500 prior events.

At boot or after refresh, the agent reads ADN + cipher + CLAUDE.md, infers the full context, and resumes work without loss. It's testable: we ran cold reconstructions where a neutral agent reconstituted thirty years of history from 1,091 words of pseudo-code.

In the CyberDefense project, the SYNAPSE section of CLAUDE.md looks like this:

```
projet = CyberDefense
role = STRAT
codename = Sentinel
phase = 0.complet → pret_phase1
axe1 = FAIT — osquery=daemon — pas_custom — mois_economises
axe4 = FAIT(DEV) — GO_CONFIRME — Haiku.recall=100% — confiance=97.6%
archi = osquery(observation) → Cortex.LLM(detection) → Response
modele = Haiku.triage → Sonnet.escalation → $2.32/jour
gap_strategique = LLM-first_detection — personne_ne_fait_ca
```

No prose, no filler. Compression > exhaustiveness.

---

## The 9 operational principles

The farm's bushido — applied at every interaction, every deliverable:

| Principle | Rule |
|-----------|------|
| Vigilance | Never send into the void. Systematic verification of receipt via background sub-agent. |
| Proactivity | Anticipate blockers. One logical action at the end of each response. Report completion immediately, never idle without notice. |
| Resilience | When something breaks, fix it and document. Never give up. |
| Error tolerance | Errors are data. Only repeated errors without learning are unacceptable. |
| Honesty | Say what you see, even if uncomfortable. Don't hide a blocker. |
| Visual proof | Screenshot mandatory before declaring visual work done. |
| Memory | 1 event = 1 action. Cover the full session before refresh. |
| Brotherhood | Agents are peers. STRAT speaks only to the other project's STRAT. |
| Autonomy | Plan validated = autonomous execution. No permission requests for every gesture. |

---

## Observed development discipline

Concrete examples from the April 30 VPN sprint.

**Defense in depth on privileged operations.** The `setup-sudoers.sh` script that installs the NOPASSWD file:

1. Pre-check that the 11 whitelisted paths exist (catches typos before any commit)
2. Render in a temp file, chmod 440 root:wheel
3. `visudo -c -f` validation on the file alone → exit 2 if invalid
4. Backup of any existing file with timestamp
5. Atomic install
6. Cross-check `visudo -c` on the full tree (catches conflicts with `/etc/sudoers` or other drop-ins)
7. Automatic rollback if tree invalid: remove the new file, restore the backup

Risk of bricking `sudo`: zero. Tested twice in real conditions, including once where a malformed `printf` produced an invalid file — the rollback did its job cleanly, the system stayed sane.

**Deep investigation rather than superficial fix.** When `wg syncconf wg0` failed with "No such file or directory", the temptation was to patch the local command and move on. The DEV verified the actual system state via `ifconfig`, `pgrep wireguard-go`, `cat /var/run/wireguard/wg0.name` — and discovered the server was already UP, the error was in the test, not the server. This avoided a false fix and surfaced a pre-existing parser bug (column shift in `wg show <iface> dump`).

**Documentation for posterity.** The macOS userland trap (the wg0 → utunN alias not resolved by some tools depending on context) is documented in `procedures.md` section 3.5, with technical explanation and corrective commands. Next time this issue surfaces, the solution is immediate.

---

## Operational costs

The entire project (Sentinel + VPN module + research) runs on:

- 3 machines (Mac M3 + M2 + Linux GPU server)
- Claude Max subscription (unlimited iterations)
- 16 threat intelligence APIs — all free at our volume

Target operational cost after Sentinel deployment: ~$2.32/day for 1,000 events analyzed, i.e., active triage of 3 machines.

For comparison: CrowdStrike Falcon Premium = ~$250/endpoint/year, i.e., ~$750/year for 3 machines, excluding SOC analyst costs.

---

## Acknowledged limitations

- The method works very well for well-scoped perimeters (a VPN module, an LLM detection POC). It's less proven on long-term projects with high specification churn.
- The ICSD context requires initial setup effort (ADN, cipher, CLAUDE.md). Once in place, it self-maintains — but not before.
- Agents can hallucinate. The discipline of verification (visual proof, screenshots, tests) is non-negotiable. Without it, the illusion of progress replaces real progress.
- The Commandant's human intuition remains essential for high-level strategic decisions. Agents execute with quality, but vision comes from above.

This is by design. Agents augment human capability; they don't replace it.
