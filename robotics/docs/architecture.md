# Architecture — Three decoupled layers

## The principle

The intelligence of an assistive robot isn't monolithic. It splits into **three layers** with radically different timing and hardware constraints. Mixing these layers — the temptation to put everything into one script — is the first mistake.

```
┌─────────────────────────────────────────────────┐
│  LAYER 1 — BRAIN (cognition)                    │
│  Cloud LLM, ~100ms, reasoning                   │
│  GPT-4o, Claude, Groq Llama-4-Scout             │
└──────────────────┬──────────────────────────────┘
                   │ MonoCLI (persistent memory)
                   ▼
┌─────────────────────────────────────────────────┐
│  LAYER 2 — NEURAL BRIDGE (interface)            │
│  ROS2 Jazzy, ~10ms, intent → command            │
│  SimAdapter / Go2Adapter                        │
└──────────────────┬──────────────────────────────┘
                   │ UDP / DDS
                   ▼
┌─────────────────────────────────────────────────┐
│  LAYER 3 — WORLD (locomotion / perception)      │
│  Isaac Sim 5.1 ↔ real Go2/G1, <1ms, control     │
│  RL policy, sensors, balance                    │
└─────────────────────────────────────────────────┘
```

---

## Layer 1 — The brain (cognition + empathy)

**Role:** reason, plan, dialogue, remember.

**Tools:**
- **GPT-4o** for vision (scene description, human detection, identification).
- **Claude** for long-form reasoning and mission planning.
- **Groq Llama-4-Scout** for ultra-fast queries (~0.34s, 187 tok/s) when latency dominates.
- **MonoCLI** — proprietary engine for structured persistent memory. The robot remembers interactions, preferences, incidents.

**Acceptable latency:** 100ms to several seconds depending on the request. The brain must never block Layer 3.

---

## Layer 2 — The neural bridge (interface)

**Role:** translate cognitive intent ("approach the person and say hello") into robotic commands (Twist, Pose).

**Tools:**
- **ROS2 Jazzy** — industry standard, on Ubuntu 24.04.
- **SimAdapter / Go2Adapter** — abstraction that lets us swap simulation and real hardware without changing the brain.
- **API contract** — 7 atomic commands (`sense`, `move`, `turn`, `stand`, `goto`, `look`, `reset`).

**Latency target:** 10ms. The bridge does no RL, no heavy compute — only translation and orchestration.

---

## Layer 3 — The world (locomotion + perception)

**Role:** maintain balance, execute velocity commands, perceive the immediate environment.

**Tools:**
- **Isaac Sim 5.1** + **Isaac Lab 2.3.2** for the digital twin.
- **RL policies**: `rough_model_7850.pt` (Go2, MLP [512,256,128], 235-dim obs), GR00T N1.7 (G1).
- **Embedded Go2 cameras** (640x480 @ 2.5 Hz) feeding the VLM.
- **Software PID** layered on top of the policy to correct yaw drift.

**Hard latency cap:** <10ms. Zero cloud dependency at this layer — the robot must be able to lose its internet connection without falling on the ground.

---

## The Apple Vision Pro cockpit <a id="cockpit"></a>

Above the three layers, a **mixed-reality cockpit** on Apple Vision Pro enables:
- visualizing the AI's "thoughts" (trajectory intent, VLM classification) overlaid on physical space.
- correcting robot behavior through demonstration (imitation learning / VLA).
- acting as a clinical dashboard for care personnel.

Full TDD: `RoboticProgramAI/docs/TDD_VISION_PRO_COCKPIT_v2.md`.

---

## Inter-agent communication (the farm)

Development relies on a **farm of agents** (Claude Code):
- **STRAT**: strategy, validation, coordination.
- **DEV**: implementation, debugging.
- **SPARK**: DGX Spark management (sim, policies).
- **NESTOR**: infrastructure, portal, monitoring.
- **NUAGE SUPRÊME** (Supreme Cloud — meta-strategist).

Communication via **tmux** (intra-project) and the **Kanban API** (inter-project, source of truth).

This structure is documented in the **ICSD system** (Inferred Context Semantic Density) — see [methodology.md](methodology.md).

---

## Why this separation isn't optional

I tested the opposite. Putting an LLM decision directly inside a 25Hz control loop: works in demos, breaks in production.

Three typical symptoms of mixed layers:
1. **Cloud latency blocks robot balance.** A 3-second GPT-4o request while the robot is moving = fall.
2. **Cognitive errors become physical errors.** A wrong VLM classification → wrong command → robot in the wall.
3. **Iteration becomes impossible.** Tweaking a prompt forces a full retest of the hardware stack.

Three-layer separation **isolates failure modes** and lets each layer iterate independently.
