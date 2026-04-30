# Vision — Infinity Cloud

> Three independent agentic projects, built solo from Switzerland on a personal supercomputer. Local-first. No cloud lock-in. Production-grade.

**Author**: Minh-Tam Dang — Infinity Cloud Sàrl, Geneva, Switzerland

---

## Context

Solo builder. Twenty years of program management at scale (Pernod Ricard, ICRC). Reconverted to AI in 2023 after an autoimmune diagnosis. Founded Infinity Cloud Sàrl in Switzerland in 2024.

Hardware footprint: a personal NVIDIA DGX Spark (Grace-Blackwell GB10, 128 GB unified memory) under the desk, paired with a Mac M3 Ultra. No cloud rental, no rented GPUs. Every workload runs on hardware I own.

Method: an "agentic farm" of specialized AI agents (STRAT/DEV pairs) coordinated through tmux sessions and structured protocols. Each project gets its own agent team. Sprints are short, deliverables are real, validation is QA-driven. The IEEE-format paper *PrivExpensIA: Observational Study of a Multi-Agent Orchestration Framework* (September 2025) documents the approach.

The orchestration backbone is [**MonoCLI**](https://github.com/infinitycloud-ch/monocli) — a Python CLI built specifically because no tool existed to give AI agents a persistent, evolving brain. SQLite knowledge base, YAML Playbooks for structured missions, Jedi Rank certification system, Scribe for continuous learning across sessions. 99.8% test pass rate. The same brain that drives the robotics platform also coordinates the project teams.

---

## The three projects

### 1. [Robotics](./robotics/) — Cognitive robotics from a desk

Quadruped robotics infrastructure (Unitree Go2 + NVIDIA Isaac Sim 5.1 + ROS2 Jazzy + RL PPO locomotion) running on the GB10 Blackwell. Three-layer architecture: Brain (MonoCLI + LLM planning) → Interface (RobotAdapter, sim/real agnostic) → World (Isaac Sim + OmniGraph). Sim-to-real transfer validated on the physical Go2 (videos in [`robotics/media/`](./robotics/media/)). Sprint 18 ran in fully autonomous mode and proved a complete genetic loop: −18.4% time, −11.8% distance, −17.6% cycles after one round of knowledge distillation. Custom X1 telemetry dashboard for real-time pilot view. Hy3D → Isaac Sim asset pipeline. GR00T N1.7 inference validated on Unitree G1.

[Read the LinkedIn article →](./robotics/docs/linkedin-article.md) · [See the dashboard, the farm, the robot →](./robotics/)

### 2. [Agentic ICBI](./agentic-icbi/) — Local-first business intelligence

CSV ingestion → DuckDB profiling → LLM-generated semantic layer (YAML) → natural-language-to-SQL with type-aware constraints → adaptive visualization (bar / line / scatter / heatmap / scorecard / orthographic globe). 32 deliverables in 6 days, three architectural pivots absorbed. 440K rows × 94 columns profiled in 3.7 seconds on local hardware. No cloud, no vendor lock-in.

[Read the LinkedIn article →](./agentic-icbi/docs/linkedin-article.md) · [Watch the demo (28 s) →](./agentic-icbi/media/demo-icbi.mp4)

### 3. [Cyber Sentinel](./cyber-sentinel/) — Defensive cybersecurity for solo operators

A defensive security toolkit for self-employed builders: hardened VPN module (WireGuard with strict sudoers scoping), Sentinel monitoring component, methodology for solo-operator threat modelling. Built because the existing tooling assumes either a SOC team or a consumer-grade attitude — neither fits the solo professional operating from home with valuable IP on disk.

[Read the LinkedIn article →](./cyber-sentinel/docs/linkedin-article.md)

---

## Why this repo exists

Three reasons:

1. **Public timestamping**. Every commit is a dated proof of work. The agentic farm pattern documented in the September 2025 IEEE paper predated mainstream multi-agent frameworks (Agent Teams, etc.) by several months.
2. **Reusable patterns**. The architecture decisions — local-first, type-aware NL→SQL, three-layer robotics, sudoers-scoped VPN — are reusable. The articles document the *why* behind each.
3. **Recruitment-quality artefacts**. Engineers and experts who land here can read working code, validated benchmarks, and honest postmortems — not marketing.

---

## Repository layout

```
vision/
├── README.md                       # This file
├── LICENSE                         # MIT
├── robotics/
│   ├── README.md                   # Project overview
│   ├── docs/
│   │   ├── linkedin-article.md     # Public-facing article
│   │   ├── architecture.md         # 3-layer cognitive model
│   │   ├── hardware.md             # GB10 + Mac M3 setup
│   │   ├── methodology.md          # Sprint workflow
│   │   ├── results.md              # Locomotion + cognitive evolution metrics
│   │   └── roadmap.md              # Next phases
│   ├── research/
│   │   ├── vlm_benchmark.md        # Vision-language model comparison
│   │   └── compatibility.md        # Stack compatibility notes
│   └── media/                      # X1 dashboard, multi-agent farm, Go2 real-condition videos, Isaac Sim
├── agentic-icbi/
│   ├── README.md
│   ├── docs/
│   │   ├── linkedin-article.md
│   │   ├── patterns.md             # Technical patterns discovered
│   │   ├── synapse-adn-cipher.md   # Compressed memory system
│   │   └── timeline.md             # 6-day chronology
│   └── media/                      # Screenshots + demo video (333 KB)
└── cyber-sentinel/
    ├── README.md
    ├── METHODOLOGY.md              # Solo-operator threat model
    ├── SENTINEL.md                 # Monitoring component
    ├── VPN_MODULE.md               # WireGuard hardening
    ├── links.md
    └── docs/
        └── linkedin-article.md
```

---

## License

MIT. See [LICENSE](./LICENSE).

## Contact

Minh-Tam Dang — Infinity Cloud Sàrl
Geneva, Switzerland
[infinitycloud.ch](https://infinitycloud.ch)
