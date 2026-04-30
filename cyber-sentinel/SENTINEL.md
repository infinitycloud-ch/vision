# Sentinel — the LLM-first EDR

> Detection by semantic reasoning, not by signature.

---

## The problem

Modern RATs (Sliver, Mythic, AsyncRAT) generate a unique hash per build. Signature databases are dead. ML on extracted features (binary entropy, section structure, import tables) catches resemblance to known malware but misses fresh binaries from adversarial frameworks.

Comparative measurements on the reference scenario (a RAT installed via a legitimate-but-trojanized 3D application):

| Tool | Likely detection |
|------|-----------------|
| Signature antivirus | ~0% — unknown hash |
| Defender for Endpoint macOS | ~25-35% |
| CrowdStrike Falcon | ~50-60% |
| SentinelOne Singularity | ~55-65% |
| **Sentinel** | **>90%** (POC measurement) |

Commercial EDRs integrate AI assistants (Charlotte AI, Purple AI, Security Copilot) — but these are **post-detection** triage layers. If the base sensor doesn't flag, the AI sees nothing.

---

## The hypothesis

What if the LLM did primary detection?

A model like Claude Haiku 4.5 can read the full context of an event (binary, code signatures, process tree, network connections, installation source, OSINT correlation) and reason about behavioral incoherence. Not a numeric score: an explicit verdict with justification.

Example verdict produced during the POC, on a fake LaunchAgent simulating a RAT:

> The binary is named `com.system.update.helper` but isn't signed by Apple. It was installed by a process downloaded via curl from a non-whitelisted domain. It opens a socket to a Vultr IP known for hosting C2. The combination Apple-mimicking-name + unsigned + unsolicited-outgoing-connection + curl-direct-provenance is a RAT pattern via LaunchAgent persistence. Verdict: RAT, 99% confidence. Recommended action: kill + quarantine.

This is an inference combining 5-6 weak signals into a strong conclusion. No traditional tool would do this.

---

## The architecture

```
                ┌─────────────────┐
                │  Mac M3 / M2 /  │  Protected endpoints
                │   Spark Linux   │
                └────────┬────────┘
                         │
                ┌────────▼────────┐
                │   osquery       │  Layer 1: observation
                │  (Apache 2.0)   │  <2% CPU, <50 MB RAM
                └────────┬────────┘
                         │ JSON events
                         ▼
                ┌─────────────────┐
                │     CORTEX      │  Layer 2: analysis
                │  Bun TypeScript │
                │                 │
                │  ┌───────────┐  │
                │  │  triage   │  │  Haiku — 95% of events
                │  │  Haiku    │  │  $0.0015/event, 2.64s
                │  └─────┬─────┘  │
                │        │        │
                │        │ suspect│
                │        ▼        │
                │  ┌───────────┐  │
                │  │ deep dive │  │  Sonnet — 5% escalation
                │  │  Sonnet   │  │  $0.008/event, 8.31s
                │  └─────┬─────┘  │
                │        │        │
                │  ┌─────▼─────┐  │
                │  │  threat   │  │  IOC enrichment
                │  │   intel   │  │  16 free APIs
                │  └───────────┘  │
                └────────┬────────┘
                         │ verdicts + threat level
                         ▼
                ┌─────────────────┐
                │    RESPONSE     │  Layer 3: actions
                │                 │
                │  log → alert    │
                │  isolate proc   │
                │  kill + quaran  │
                │  block IP       │
                │  isolate net    │
                └─────────────────┘
```

---

## Quantified POC results

Test on 100 representative events (90 benign + 10 threats):

| Metric | PRD target | Haiku 4.5 | Sonnet 4.6 |
|--------|-----------|-----------|------------|
| Detection confidence | >80% | 97.6% | 98.0% |
| Latency per event | <5s | 2.64s | 8.31s |
| Cost/day at 1,000 events | <$5 | $1.52 | $7.97 |
| Threat recall | >80% | 100% | 100% |
| Corrected F1 score | — | ~95% | ~95% |

The 10 threat vectors tested: RAT LaunchAgent, xmrig cryptominer, reverse shell, info stealer (keychain/browser), backdoor, dropper curl|bash, SSH brute force, DNS exfiltration, SUID escalation, crontab C2 persistence.

100% recall on all 10 vectors with both models.

---

## Honest limitations

- **~3s latency per event**: requires local pre-filtering. For high benign event volume, we triage with rules or a small local model (Nemotron on Spark GPU) before the cloud API.
- **Cost at scale**: with prompt caching the cost is manageable, but unfiltered traffic can get expensive fast. Two-tier architecture is essential.
- **Hallucinations**: gray zones. High confidence thresholds (>85%) before any destructive action. "Initial learning" mode for 7 days to calibrate baseline.
- **Complementary defense required**: for fast-moving threats (ransomware encrypting in seconds), LLM latency isn't enough. Sentinel coexists with a classical defense layer for these cases.

---

## Threat intelligence

16 free sources integrated or planned:

**P0 (Phase 1 implementation)**: Abuse.ch ThreatFox + URLhaus + MalwareBazaar + Feodo Tracker, CISA KEV, Google OSV.

**P1 (Phase 2)**: NVD (NIST), AlienVault OTX, AbuseIPDB.

**P2 (Phase 3)**: VirusTotal (rate limit 4/min, 500/day free), Vulners, GreyNoise, CIRCL/MISP.

Total cost: **$0/day** for our volume. Three-layer architecture (local SQLite cache refresh every 5min + on-demand API lookup + periodic 2h sync).

---

## Why not (just) Darktrace

Darktrace is the philosophically closest tool — unsupervised ML on network behavior, autonomous response (Antigena). But Darktrace uses autoencoders and Bayesian methods — it detects **statistical anomalies**. It cannot *explain* why something is suspicious in human terms. The LLM can, and that's central to conversational forensics and operational trust.

Sentinel and Darktrace aren't competitors — they're complementary. But on the "native verdict explainability" criterion, Sentinel is unique.

---

## Publication roadmap

- Phase 0 (research + POC) — complete, full reports available
- Phase 1 (osquery daemon + pipeline) — sprint in progress
- Phase 2 (basic Cortex + dashboard) — next sprint
- Phase 3 (autonomous Response + conversational forensics) — Q3 2026
- Phase 4 (honeypots + mutation defense + supply chain audit) — Q4 2026
- Phase 5 (product packaging, SaaS mode, agent marketplace) — 2027

Code and technical reports are opened progressively as they ship.
