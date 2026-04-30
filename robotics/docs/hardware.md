# Hardware — Why GB10 Blackwell

## The machine

| Spec | Value |
|---|---|
| **Model** | NVIDIA DGX Spark |
| **GPU** | GB10 Blackwell |
| **Unified memory** | 128 GB |
| **CPU architecture** | aarch64 (ARM) |
| **TDP** | ~100 W |
| **OS** | Ubuntu 24.04 |
| **CUDA** | 13.0 |
| **PyTorch** | 2.9.1+cu130 |
| **Python** | 3.11 (Isaac env) / 3.12 (system) |
| **Network** | 10 Gbit Ethernet |
| **Local IP** | <LAN_IP_SPARK> |

---

## Why GB10 and not an H100

Fair question: why not a cloud H100 cluster, or an on-prem DGX H100?

**Iteration economics.** A pay-as-you-go cloud H100 sits around USD 3/hour at steady state, more during peaks. A useful RL training cycle (1500 PPO iterations on Go2) takes ~6 hours. Five cycles a week = USD 360/month minimum, before storage, bandwidth, and the friction of provisioning every time.

A DGX Spark is a one-off purchase, negligible electricity, **available 24/7 with zero marginal cost**. Iteration becomes free. That's what changes the game for an independent developer — not raw throughput.

**Technical sufficiency.** The GB10 has the VRAM (128 GB unified) for Isaac Sim 5.1 + Isaac Lab 2.3 + several policies running in parallel. For very large training runs (foundation-model-scale GR00T training), you genuinely need a cluster. But for **fine-tuning, deploying, measuring, iterating** on existing policies — the GB10 is more than enough.

**Mobility.** 100W, passive cooling, compact form factor. The robot can come to the machine, or the machine to the robot. No physical tether to a datacenter.

---

## Why not a Jetson

The Jetson Orin Nano is the natural choice for real-time embedded work, and it has a place in the production stack (future Companion).

For **development** — training, simulation, benchmarking — the Jetson is too constrained. Not enough VRAM for Isaac Sim, insufficient throughput for iterating across multiple policies, a more limited Python ecosystem.

The **GB10 (dev) ↔ Jetson (deploy)** pairing makes sense: anything trained on GB10 gets exported to JIT/ONNX, then ships to embedded Jetson. For now, the Go2 is piloted by an embedded iPhone (see the main Nuage article), not a Jetson — a pragmatic choice driven by iOS-native capabilities (LiDAR, ARKit, Vision, microphones).

---

## aarch64 surprises

The aarch64 ML ecosystem is mature but not universal. A few findings:

- **PyTorch CUDA on aarch64**: available (`torch 2.9.1+cu130`). No issues.
- **NVIDIA WebRTC**: no official ARM package for LiveStream. Worked around via a custom web viewer.
- **Nemotron VLM 12B**: blocked because `mamba-ssm` isn't compiled for aarch64 — fell back to Groq Llama-4-Scout, then migrated to GPT-4o.
- **Isaac Sim 5.x**: ARM supported since 5.0 — **5.1** is the first version that's fully operational on this hardware.

Heads-up for anyone starting out: budget 1–2 days of dependency debugging in the first weeks.

---

## The Mac M3 Ultra hub

The DGX Spark doesn't run alone. A Mac M3 Ultra (<LAN_IP_HUB>) acts as the main hub:
- Panda Portal (Kanban, API, monitoring) on port 3010.
- Multi-agent orchestration (tmux sessions).
- Asset storage, documentation, centralized logs.
- ssh/scp connection to Spark for deployment.

This split — CPU-strong (Mac) + GPU-dedicated (Spark) — covers the essentials without a cluster.

---

## Total cost

For reference, no drama intended:

| Item | Order of magnitude |
|---|---|
| DGX Spark | CHF 4,000 |
| Mac M3 Ultra | CHF 8,000 |
| Unitree Go2 (Edu) | CHF 5,500 |
| Unitree G1 (Education Pro) | CHF 16,000 |
| Apple Vision Pro | CHF 4,000 |
| iPhone 16 Pro Max | CHF 1,500 |
| **Total** | **~CHF 39,000** |

Compare with one month's payroll for a twelve-engineer ML team, or the annual TCO of a dedicated H100 cluster.

This isn't a competition. It's a demonstration that **the entry ticket to cognitive robotics has dropped by an order of magnitude** — and that this shift opens projects that large players have not been funding.
