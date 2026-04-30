# LinkedIn — Article CyberDefense / Sentinel (English version)

**Audience:** engineers, AI/Robotics/Security experts, tech leaders
**Tone:** professional but conversational, humble + serious, no jokes, no marketing fluff
**Status:** active — Commandant decision: international audience priority, publish concurrent with FR
**Source:** translation of `linkedin-article.md`, idioms adapted for English-speaking cybersec audience

---

## Suggested title

**Detecting what we don't know yet: why I'm building an EDR that *reasons* instead of matching signatures**

(Alternatives: "My EDR has no signature database, and that's by design" / "Pattern matching vs reasoning: lessons from a 100-event benchmark")

---

## Article

I've spent the last six months looking closely at how modern endpoint defense tools actually work — CrowdStrike, SentinelOne, Microsoft Defender for Endpoint, plus the open-source cohort (osquery, Wazuh, Velociraptor, LimaCharlie). Benchmarking properly, on representative scenarios, and reading the steady stream of academic papers that have been coming out since 2024.

One observation keeps surfacing: **against modern polymorphic RATs, the best commercial EDRs detect somewhere between 30% and 65% of compromises** — depending on the scenario, the OS profile, and how fresh their threat intelligence is. It's measurable, documented, and it's not really anyone's fault. It's a consequence of architecture.

These tools — and they're well engineered — combine three things: signatures (hash matching), ML on extracted features (binary entropy, import tables, section structure), and behavioral rules (process chains, IOAs). The generative AI layers from the leaders (Charlotte AI, Purple AI, Security Copilot) are *on top* of detection: they triage alerts, write incident reports, accelerate investigation. They don't analyze raw events — the static engine still decides what becomes an alert.

This split works well when the malware looks like something the engine has seen before. It works poorly when the attacker compiles a fresh binary, which has been the default mode of operation since frameworks like Sliver, Mythic, or even trivial packers started generating unique hashes per build.

I wanted to test the inverse hypothesis. **What if an LLM did primary detection, not just alert triage?**

The idea is simple to state, less simple to execute: instrument the system with an existing observation layer (I picked osquery — Meta's stack, mature, low overhead, Apache 2.0), ship events to an analysis engine, and have the engine submit full context to an LLM with an outcome-oriented prompt: *"here's a binary that just installed itself as a LaunchAgent, here's its installation context, its process tree, its network connections, its code signing state, its OSINT correlation — is it malicious, and how confident are you?"*

What the LLM returns isn't a numeric score. It's reasoning. On the scenario that pushed me to start this project — a RAT discovered on my own machine, installed by a legitimate-but-trojanized 3D slicer — the LLM essentially wrote: *"The binary is named com.finder.helper but it's not signed by Apple. It was installed by a 3D slicer that has no legitimate reason to register a network service. It opens a socket to a Vultr IP known for hosting C2 infrastructure. RAT, 94% confidence."*

No signature matched. No hash database contained it. This is *semantic inference* on behavioral incoherence.

I ran a POC on 100 events (90 benign, 10 threats spanning 10 distinct attack vectors: RAT, cryptominer, reverse shell, info stealer, dropper, SSH brute force, DNS exfiltration, SUID escalation, crontab persistence, backdoor LaunchAgent). Results with Claude Haiku 4.5:

- **Recall**: 100% (10/10 threats detected)
- **Average confidence** on real threats: 97.6%
- **Latency per event**: 2.64 seconds
- **Estimated cost**: $1.52/day for 1,000 events (with prompt caching)

With a two-tier architecture — Haiku for triage, Sonnet for ambiguous cases — total operational cost for 3 protected machines comes in around **$2.32/day**, well under my $5/day target.

The false positives (13/90 on Haiku) were caused by my test data generator pairing legitimate processes with implausible network connections. The LLM was *right* to flag them: a syslogd connecting to Discord is suspicious, even if in my synthetic data it was just noise. After fixing the generator, corrected precision exceeds 90%.

This approach isn't magic. It has limits: latency (~3s per event) requires local pre-filtering of obviously benign events; cost scales fast if you send everything without triage; LLMs hallucinate in gray zones, so confidence thresholds need to be high (>85%) before any destructive action; you need a classical defense layer in parallel for fast-moving threats that LLM latency would let through. But on the class of threats current EDRs miss — unique polymorphic binaries, supply chain attacks, RATs targeting SMBs without a SOC — the balance tips.

In parallel with the research, I also wanted to validate that I could *execute*. Today (April 30), I shipped an annex sub-module: a self-hosted WireGuard VPN, broken into six work blocks (server setup, networking, client scripts, tests, documentation, HTTPS API). From idea to validated end-to-end tunnel from an iPhone, including management scripts (add/revoke/list/diag), an HTTPS API in Bun TypeScript with Bearer token auth and strict CORS, a LaunchAgent with auto-restart tested under real conditions (kill -9 → respawn in 500ms), and a sudoers NOPASSWD setup with minimal scope to avoid prompts. A few hours of agentic work. **Bonus: discovered a pre-existing bug in my own parser code, surfaced by deep audit during the sprint.**

This isn't Sentinel itself. It's evidence that the execution pipeline works. The production daemon (Phase 1, based on osquery) starts next week.

What I'll publish as the phases roll out: the competitive comparison reports (10 tools analyzed), detailed LLM benchmarks, the Cortex architecture, macOS detection techniques (Endpoint Security Framework, FSEvents) and Linux ones (eBPF). If reasoning-based cyber defense interests you — or if you see holes in my logic — I'm open to the exchange.

Code and technical reports will be opened progressively as they ship.

---

## Hashtags

`#Cybersecurity #EDR #LLMSecurity #ThreatDetection #WireGuard #SelfHosted #ClaudeAPI #macOSSecurity #SecurityResearch #AppliedAI`

(11 hashtags = 2026 LinkedIn analytics optimum — beyond 12 reach degrades.)

---

## Publication notes

- **Recommended publishing time**: 9-11am ET on a Tuesday/Wednesday/Thursday (US tech audience starting their day, European tech audience still active in late afternoon).
- **Suggested visual**: a screenshot of the benchmark report tables (poc-axe4/results/benchmark-report.md) or the 3-layer architecture diagram from `docs/sentinel-vue-densemble.html`. Numerical visuals consistently outperform symbolic visuals on tech LinkedIn — usually 2-3x engagement.
- **Pinned first comment**: "GitHub Vision repo with detailed sections here: [link]" to drive traffic.
- **Re-engagement**: respond seriously to the first 5-10 comments within the hour — boosts algorithm.

---

## Idiomatic adjustments from French version

- "decoupage classique" → "classic split" (more natural than "classic breakdown")
- "marche bien quand le malware ressemble" → "works well when the malware looks like" (avoid translating literally)
- "matching signatures" kept as English idiom (works in both languages)
- "un score numerique" → "a numeric score" (direct)
- "raisonnement" → "reasoning" (most precise English equivalent)
- "C'est volontaire" → "That's by design" (more natural English)
- "balance penche" → "the balance tips" (idiomatic English)
- "evidence that the execution pipeline works" preferred over "proof that..." (less corporate)

---

## Status

**Active — ready to publish.** Commandant decision (April 30): international audience priority, publish in parallel with the French version. Adjustments to tone or angle still possible if requested.
