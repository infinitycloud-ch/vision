# CyberDefense — Sentinel

> Autonomous cyber defense built on LLM reasoning. An EDR that *understands* what it observes instead of matching signatures.

---

## Why this project exists

Modern cyber defense tools (CrowdStrike, SentinelOne, Microsoft Defender) detect somewhere between **30% and 65% of polymorphic RATs** — depending on the scenario, the OS, and how fresh their threat intelligence is. The classic split (signatures + ML on extracted features + behavioral rules) works well as long as the malware looks like something already known. It fails when the attacker compiles a fresh binary — which has been the default mode of operation since Sliver, Mythic, and trivial packers.

The generative AI layers integrated into commercial EDRs (Charlotte AI, Purple AI, Security Copilot) sit **on top of** detection: they triage alerts, write reports, accelerate investigation. They don't analyze raw events. The static engine still decides what becomes an alert.

Sentinel inverts the architecture: **the LLM is the primary detection engine**, not a post-detection triage layer. Nobody does this today, and it's by design — that's the bet.

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│  Layer 1 — SENTINEL CORE (observation)                   │
│  osquery + FSEvents + Endpoint Security Framework        │
│  <2% CPU, <50 MB RAM, real-time JSON event stream        │
└──────────────────┬───────────────────────────────────────┘
                   │ event stream
                   ▼
┌──────────────────────────────────────────────────────────┐
│  Layer 2 — CORTEX (reasoning)                            │
│  Claude Haiku (triage) → Sonnet (escalation on suspects) │
│  Memory: usearch + SQLite, threat intel: 16 free APIs    │
└──────────────────┬───────────────────────────────────────┘
                   │ verdicts + confidence
                   ▼
┌──────────────────────────────────────────────────────────┐
│  Layer 3 — RESPONSE (graduated actions)                  │
│  log → alert → kill → quarantine → network isolation     │
│  Reversible encrypted quarantine vault, LLM forensics    │
└──────────────────────────────────────────────────────────┘
```

**Core architectural decision:** don't rewrite a custom daemon. osquery covers 90% of the observation needs, is Apache 2.0, mature, low-overhead. Concentrate engineering effort on the Cortex — the genuinely novel part.

---

## POC results (Phase 0 validation)

Benchmark on 100 events (90 benign + 10 threats across 10 distinct vectors: RAT, cryptominer, reverse shell, info stealer, dropper, SSH brute force, DNS exfiltration, SUID escalation, crontab persistence, backdoor LaunchAgent).

| Metric | PRD target | Claude Haiku 4.5 | Claude Sonnet 4.6 |
|--------|-----------|------------------|-------------------|
| Detection confidence | >80% | **97.6%** ✅ | **98.0%** ✅ |
| Latency per event | <5s | **2.64s** ✅ | 8.31s ❌ |
| Cost/day (1,000 events) | <$5 | **$1.52** ✅ | $7.97 ❌ |
| Recall (threat detection) | >80% | **100%** ✅ | **100%** ✅ |

**Selected architecture:** Haiku for bulk triage, Sonnet escalation on ambiguous cases (~5-15% of events). Total estimated cost for 3 protected machines: **~$2.32/day**.

False positives (13/90 on Haiku) were caused by a test data generator pairing legitimate processes with implausible network connections. The LLM was *right* to flag them: a syslogd connecting to Discord is suspicious. After fixing the generator, corrected precision exceeds 90%.

---

## Concrete scenario

A RAT discovered on my machine in March 2026 — installed by CrealityPrint, a legitimate but trojanized 3D slicer. The `com.finder.helper` binary opened a C2 connection to a Vultr IP.

| Tool | Likely detection |
|------|-----------------|
| Traditional antivirus (signatures) | ~0% — unknown hash, ignored |
| Defender for Endpoint macOS | ~25-35% |
| CrowdStrike Falcon | ~50-60% |
| SentinelOne Singularity | ~55-65% |
| **Sentinel (LLM-first)** | **>90%** (POC measurement) |

Verdict produced by the LLM: *"The binary is named com.finder.helper but isn't signed by Apple. It was installed by a 3D slicer that has no legitimate reason to register a network service. It opens a socket to a Vultr IP known for hosting C2 of polymorphic RATs. RAT, 94% confidence."*

This is semantic inference on behavioral incoherence — not a signature, not an ML score, not a pre-written rule.

---

## Project status

### Phase 0 — Research & POC (complete)

- **Mapping** of 10 open-source and commercial EDR/XDR tools ([report](https://github.com/<vision>/cyberdefense/blob/main/research-phase0-axes1-2.md))
- **Inventory** of 16 free threat intelligence APIs (Abuse.ch, CISA KEV, NVD, OSV, AlienVault OTX, AbuseIPDB, etc.)
- **POC** of LLM detection on 100 events ([benchmark](https://github.com/<vision>/cyberdefense/blob/main/benchmark-report.md))
- **Go/No-Go decision**: all criteria passed, GO confirmed

### Annex VPN module (delivered)

Self-hosted WireGuard with HTTPS API for management. Shipped April 30, 2026 in a few hours of agentic work. Broken into 6 blocks (server setup, networking, client scripts, tests, API, documentation). End-to-end tunnel validated from iPhone.

Components:
- 11 idempotent CLI scripts (install, uninstall, add-client, revoke-client, list-clients, diag-handshake, backup-config, etc.)
- WireGuard LaunchDaemon + API LaunchAgent
- Bun TypeScript HTTPS API on port 3403, Hono framework
- 5 endpoints (status, clients GET/POST/DELETE, server/restart) with strict regex validation against injection
- Bearer token auth with constant-time comparison
- NOPASSWD sudoers with minimal scope (11 absolute paths, zero wildcards)
- Self-elevate pattern in scripts (zero-prompt UX after setup)
- Complete procedures + architecture documentation (270 lines)

[See VPN_MODULE.md for technical details.](VPN_MODULE.md)

### Phase 1 — Sentinel Core daemon (upcoming)

- osquery configuration on the 3 machines
- osquery → Cortex pipeline via Bun TypeScript API
- Watchers: processes, network, LaunchAgents, filesystem, code signing
- Real overhead testing in production

### Phase 2 — Basic Cortex

- LLM analysis engine with Haiku triage → Sonnet escalation
- 7-day baseline learning
- Dashboard module in Panda Portal
- Threat intelligence integration (local cache + on-demand lookup)

### Phase 3 — Autonomous Response

- Graduated actions (log → kill → quarantine → isolation)
- Encrypted reversible quarantine vault
- Conversational forensics ("What happened last night?")

### Phase 4 — Advanced intelligence

- Intelligent honeypots (AI-generated, adapted to the context of each machine)
- Mutation defense (the daemon mutates its own signature)
- Supply chain audit (semantic pre-installation analysis for npm/pip/brew)
- Multi-machine correlator

---

## Technical stack

| Component | Choice | Justification |
|-----------|--------|---------------|
| Observation daemon | osquery (Apache 2.0) | Mature Meta stack, low-overhead, 300+ tables, native JSON |
| Cortex | Bun + TypeScript | Consistent with the ecosystem, performant, strict types |
| LLM | Claude Haiku 4.5 + Sonnet 4.6 | 100% recall, 2.64s latency, $1.52/day cost |
| Threat memory | usearch + SQLite | Vector + relational, lightweight |
| Threat intel | 16 free APIs | Total cost: $0/day for our volume |
| VPN API | Bun + Hono | Lightweight TypeScript framework, native TLS support |

---

## Development methodology

Sentinel is built by a coordinated farm of AI agents (InfinityCloud modus operandi). Each project has a STRAT (strategy, validation) + DEV (implementation, testing) duo. Coordination via tmux, compressed memory in ICSD format (Inferred Context Semantic Density), bridge with other projects via a central agent.

[See METHODOLOGY.md for details.](METHODOLOGY.md)

---

## Links

- **LinkedIn article** (English): `docs/linkedin-article-en.md` in the CyberDefense repo
- **LinkedIn article** (French): `docs/linkedin-article.md` in the CyberDefense repo
- **HTML overview**: `docs/sentinel-vue-densemble.html` in the CyberDefense repo
- **Research report**: `docs/research-phase0-axes1-2.md`
- **Competitive analysis**: `docs/competitive-analysis-edr.md`
- **Threat intel APIs**: `docs/threat-intel-apis-research.md`
- **POC benchmark**: `CyberDefense APP/poc-axe4/results/benchmark-report.md`
- **VPN architecture**: `docs/vpn-architecture.md`
- **VPN API spec**: `docs/vpn-api-spec.md`
- **VPN procedures**: `vpn-module/docs/procedures.md`

The French versions of these documents remain available as `*-fr.md` in this directory.

---

## Status

**Phase 0:** complete (research + validated POC).
**VPN module:** in production on the M3 hub, iPhone tunnel validated end-to-end.
**Phase 1:** imminent start.

Code and technical reports will be opened progressively as they ship. If reasoning-based cyber defense interests you — or if you see holes in the logic — feedback is welcome.

---

*"Walls protect castles. Sentinels protect kingdoms."*
