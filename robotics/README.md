# Cognitive Robotics — from a desk, in Switzerland

> A Unitree Go2 quadruped, a 100-watt supercomputer under the desk, and the belief that assistive robotics shouldn't have to wait for big-tech billions to exist.

[![Hardware](https://img.shields.io/badge/Hardware-NVIDIA%20DGX%20Spark%20GB10-76b900)](docs/hardware.md)
[![Sim](https://img.shields.io/badge/Sim-Isaac%20Sim%205.1-76b900)](docs/architecture.md)
[![Robot](https://img.shields.io/badge/Robot-Unitree%20Go2%20%2B%20G1-blue)](docs/architecture.md)
[![Country](https://img.shields.io/badge/Made%20in-Switzerland-red)](#)

---

## The ecosystem in motion

![X1 Dashboard — real-time telemetry of the quadruped](./media/01-x1-dashboard.jpg)

*X1 Dashboard — real-time pilot interface: battery, IMU, position, motor states, instantaneous velocity. The cockpit of a cognitive vehicle.*

---

## The story

One person, in Switzerland, who spent fifteen years running industrial programs (Pernod Ricard, ICRC), pivoted to AI after an autoimmune diagnosis in 2023, founded Infinity Cloud Sàrl in 2024, and in 2025 placed a **NVIDIA DGX Spark (Grace-Blackwell GB10, 128 GB unified memory, 100 W TDP)** on his desk.

No cloud cluster. No team of twelve engineers. No board-driven roadmap.

The bet: that a GB10 under the desk, a simulation-first discipline, and an **agentic farm** of specialized AI agents can close a complete loop — simulation → policy → real robot → measurement → iteration — at a cost and tempo that no large structure can match for sheer fluidity.

This repo documents where that bet stands today.

---

## The ecosystem, piece by piece

### The brain — [MonoCLI](https://github.com/infinitycloud-ch/monocli)

At the core of the orchestration: **MonoCLI**, a Python CLI I built because no tool existed to give a robot a structured, persistent memory. SQLite knowledge base, YAML Playbooks for structured missions, Jedi Rank certification system (I → IV) to validate the robot's competence levels, a Scribe that distils lessons across sessions for continuous learning, VLM perception via Groq Maverick. 99.8% pass rate over 390 tests, 0 falls across 13+ navigation runs.

Public repo: [github.com/infinitycloud-ch/monocli](https://github.com/infinitycloud-ch/monocli)

### The body — Unitree Go2 (and soon G1)

A 12-DoF EDU quadruped: affordable for a solo developer, serious enough for research. The same RL PPO policy trained on the GB10 runs in simulation and on the physical robot through the `RobotAdapter` pattern (sim/real agnostic). The G1 humanoid joins the ecosystem in the next wave.

### The world — Isaac Sim 5.1 on GB10

| | |
|--|--|
| ![Isaac Sim arena 25 by 15 meters with calibrated ground](./media/06-arena-25x15.png) | ![Multiple Unitree Go2 quadrupeds running in parallel in Isaac Sim](./media/07-multi-robot.png) |
| 25 × 15 m simulation arena, calibrated ground, friction 1.0 (matches RL training) | Multi-robot in Isaac Sim — multiple Go2 in parallel, OmniGraph driven by ROS2 |
| ![RL PPO locomotion policy executing at 25 Hz on Unitree Go2](./media/08-locomotion.png) | ![Custom cyberpunk-themed Isaac Sim viewer for debugging](./media/09-cyberpunk-viewer.png) |
| RL policy in action: 25 Hz, 48-dim observation, 12 torques per tick | Custom cyberpunk-themed viewer — internal debug tooling |

### The agentic farm in action

![Multi-agent farm: parallel Claude Code sessions in tmux orchestrating Isaac Sim and sprint reports](./media/02-multi-agent-farm.jpg)

*Main screen view: several Claude Code sessions orchestrated through tmux, Isaac Sim viewport in the center, sprint reports and comparison tables on the sides. STRAT and DEV pairs per project, Nestor as conductor.*

---

## Proof by the numbers — Sprint 18 in autonomous mode

![Sprint 18 autonomous mode report: adaptive patrol with measured cognitive evolution gains](./media/03-sprint18-success.jpg)

**Mission**: prove the complete genetic loop — the robot perceives, analyses, learns, evolves.

**Result**: TOTAL SUCCESS. Five phases delivered (Gen 1 naive → Gen 2 evolved), measured gains: **−18.4% time, −11.8% distance, −17.6% cycles**. The loop works end-to-end. 4/5 resilience levels validated. Crunched in 21 min 53 s by the agentic farm in autonomous mode.

This is what "cognitive" means in the context of this repo: today's robot is not yesterday's, and the improvement is **quantifiable and reproducible**.

---

## The robot, in real-world conditions

Two video sequences captured on April 30, 2026, on the physical Go2 outside simulation:

📹 [`media/04-go2-real-test-1.mp4`](./media/04-go2-real-test-1.mp4) — 6.9 MB
📹 [`media/05-go2-real-test-2.mp4`](./media/05-go2-real-test-2.mp4) — 4.3 MB

The policy trained on the GB10 in simulation is transferred to the physical robot. This is the sim-to-real step — the step that separates LinkedIn demos from serious projects.

---

## In 30 seconds — current state

| Component | Status | Reference |
|---|---|---|
| **GR00T N1.7 inference on Unitree G1** | Walk + turn validated | [docs/results.md](docs/results.md) |
| **GEAR-SONIC WBC clone** | Low-level reference | [docs/results.md](docs/results.md) |
| **Hy3D → Isaac Sim pipeline** | 3 GLBs delivered | [docs/results.md](docs/results.md) |
| **Go2 patrol + VLM (GPT-4o)** | E2E validated | [docs/results.md](docs/results.md) |
| **PID yaw correction** | Drift 70% → 6% | [docs/results.md](docs/results.md) |
| **Apple Vision Pro cockpit** | TDD v2.0 | [docs/architecture.md](docs/architecture.md) |
| **Adaptive genetic loop** | Sprint 18 TOTAL SUCCESS | See report above |
| **MonoCLI brain CLI** | Production, 99.8% pass rate | [github.com/infinitycloud-ch/monocli](https://github.com/infinitycloud-ch/monocli) |

---

## The bet: 100 W against megawatts

A standard RL training cluster consumes the equivalent of a town of 5,000 people. The DGX Spark consumes less than a hair dryer.

This difference isn't just ideological. It changes what you can **afford to iterate**.

- **128 GB unified memory** — Isaac Sim 5.1 + Isaac Lab 2.3 + several policies in parallel, no swap.
- **aarch64 (ARM) architecture** — the ML ecosystem is mature, with a few blind spots to know.
- **Passive cooling** — multi-hour sessions without throttling.
- **100 W TDP** — plugs into any standard outlet, transportable, silent.

Not a substitute for a big-corp DGX H100. A **tool sufficient to close a complete cycle**: simulation → policy → robot → measurement → iteration.

---

## Who this repo is for

- **Robotics engineers** curious about what a solo developer with a GB10 can produce in a few months.
- **VLA / VLM researchers** interested in perception in simulated then real environments.
- **Health / silver economy decision-makers** looking for a Swiss partner for assistive robotics for vulnerable populations — that's where this is heading.
- **Independent developers** hesitating to start in robotics without a cloud cluster — here is the proof that it's possible.

---

## Structure

```
robotics/
├── README.md                 ← you are here
├── docs/
│   ├── linkedin-article.md   ← public article
│   ├── architecture.md       ← 3 layers: brain / bridge / world
│   ├── hardware.md           ← GB10 Blackwell, technical choices
│   ├── results.md            ← metrics, screenshots, logs
│   ├── methodology.md        ← simulation-first, ICSD, iteration
│   └── roadmap.md            ← upcoming milestones, sim-to-real
├── media/                    ← X1 dashboard, agentic farm, Go2 videos, Isaac Sim
└── research/
    ├── compatibility.md      ← walk-these-ways, unitree_rl_gym, GR00T
    └── vlm_benchmark.md      ← Groq vs GPT-4o vs qwen3-vl vs Nemotron
```

---

## Links

- **LinkedIn article**: [Building a cognitive robotics infrastructure from Switzerland — alone, with a GB10 Blackwell under the desk](docs/linkedin-article.md)
- **Brain of the ecosystem**: [MonoCLI — github.com/infinitycloud-ch/monocli](https://github.com/infinitycloud-ch/monocli)
- **Main RoboticProgramAI repo**: [github.com/infinitycloud-ch/roboticprogramai](https://github.com/infinitycloud-ch/roboticprogramai)
- **Contact**: Minh-Tam Dang — Infinity Cloud Sàrl, Geneva, Switzerland

---

*Last updated: 2026-04-30 — iterated weekly at the rhythm of breakthroughs.*
