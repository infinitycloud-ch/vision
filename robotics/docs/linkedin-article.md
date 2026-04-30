# Building cognitive robotics infrastructure from Switzerland — solo, with a GB10 Blackwell on the desk

*Minh-Tam Dang — Infinity Cloud Sàrl*

---

## An uncomfortable observation

Serious robotics — the kind that gets out of LinkedIn demos and onto the field — is expensive. Very expensive. A team of twelve engineers, a six-figure monthly cloud bill, eighteen months before the first interesting prototype. That's the dominant pattern. I'm not judging it: it produced most of the published results in 2024–2026.

But it's not the only path.

Since late 2025, I've been building, alone, from Lausanne, a complete cognitive robotics infrastructure. Unitree Go2 quadruped, G1 humanoid, Isaac Sim digital twin, 3D asset generation pipeline, autonomous navigation, VLM-based identification, persistent memory. Everything runs on **a single NVIDIA DGX Spark — GB10 Blackwell silicon, 100W TDP, sitting on a desk** — paired with a Mac M3 Ultra acting as the hub.

This post isn't a complaint about budgets or a marketing promise. It's a technical write-up of what works, what breaks, and what you learn when you're forced to keep things lean.

---

## The hardware bet: 100W versus megawatts

A typical RL training cluster pulls the equivalent of a 5,000-resident town. The DGX Spark draws less than a hairdryer. The difference isn't just ideological — it changes what you can **afford to iterate on**.

- 128 GB unified memory (CPU/GPU shared) — enough for Isaac Sim 5.1, Isaac Lab 2.3, and several policies in parallel.
- aarch64 (ARM) architecture — the ML ecosystem on aarch64 has matured, but expect a few surprises (PyTorch CUDA available, some Python ML binaries not yet shipped).
- Passive cooling sufficient for multi-hour sessions without throttling.

For an independent developer, this is the equivalent of having an NVIDIA lab within reach. Not a replacement for a large group's DGX H100 cluster — but **enough to close a full learning loop**: simulate, deploy on the physical robot, measure, repeat.

---

## What's running today

A few bricks validated on the bench, no shortcuts, all reproducible.

**1. GR00T N1.7 inference on Unitree G1**
NVIDIA shipped GR00T N1.7 in late March 2026. The model runs inference on the DGX Spark. The G1 walks, turns, holds its balance. I also cloned the GEAR-SONIC project's Whole-Body Controller as a low-level baseline. Partial conclusion: N1.7 is usable outside a research lab, but per-task fine-tuning remains essential.

**2. Hy3D pipeline for scene assets**
Tencent's Hunyuan3D generates GLB meshes that drop straight into Isaac Sim via `add_reference_to_stage`. Three assets delivered and integrated into a hospital scene: *BestFriend* (companion fox, 7.8 MB), *Doctor* (stylized humanoid, 9.2 MB), *Go2 tron skin* (custom quadruped texture). The marginal cost of a new scene object dropped from hours (manual modeling) to minutes.

**3. Autonomous patrol with animal/human discrimination**
The Go2 walks along the walls of a simulated hospital room, captures camera frames every 5 seconds, sends each image to GPT-4o with a generic prompt ("is there an animal or a person?"). Animal detected → log + keep going. Human detected → stop + identification protocol → cross-check against an authorized list → agentic decision. Full pipeline validated end-to-end: camera → VLM → classification → decision → action.

**4. Real-time PID correction over RL drift**
The current walking policy drifts roughly 80° in 30 seconds (a known limitation of the *Velocity-Rough-Unitree-Go2* policy). A software PID reads actual yaw at 4 Hz, computes error against the desired heading, and injects a bounded corrective wz on top. Measured result: lateral drift cut from ~70% to 6% of distance travelled. No retraining, no fine-tuning — just a classic feedback loop layered on top of the policy.

---

## The method: simulation-first, iteration discipline

One rule: **nothing touches the physical robot until it has run a hundred times in simulation**. Not dogma — economics. A real G1 fall costs CHF 8,000. A simulated G1 fall costs an `env.reset()`.

The less obvious corollary: **simulation quality is itself a deliverable**. Half a day lost this week on a friction mismatch between the Isaac ground plane and the training's `velocity_env_cfg` (Unitree). Another half-day on observation-space compatibility between the open-source policies (walk-these-ways, unitree_rl_gym, GR00T) and current Isaac Lab 2.3: **all three depend on Isaac Gym Preview 4 (deprecated) and their observation spaces are incompatible with Isaac Lab**. Not a tragedy — a data point.

Iterating fast also means **keeping the stack thin**. No Kubernetes orchestrator. No microservices. Three Python scripts, a UDP bridge, a Three.js viewer. When something breaks, I know why in five minutes.

---

## What doesn't work yet

Honest section, because a technical post without weak spots is always suspect:

- **Visual identification of non-photoreal assets is fragile.** A simplified Hunyuan3D mesh isn't recognized as a human by GPT-4o. Current workaround: scene-aware prompts. Proper fix: better assets, or fine-tuned VLMs.
- **The RL policy fatigues after ~50 seconds in production.** `last_action` accumulates errors outside training episodes. Periodic reset is a workaround. Targeted retraining is the real answer.
- **Sim-to-real isn't crossed yet.** Everything I've described runs in simulation, plus the physical Go2 in controlled environments. The leap to autonomy in real human-occupied spaces is what the next few months are for.

---

## Where this is heading

Infinity Cloud's goal isn't performance for its own sake. It's putting reliable robotic presence at the service of vulnerable populations: home care, cognitive support, passive safety in nursing homes. No marketing timeline, no announcements before clinical validation. But a working pipeline, reproducible results, and an infrastructure that depends neither on a cloud cluster nor on a twelve-engineer team.

I'll publish more technical write-ups in the coming weeks. If you work on adjacent problems — assistive robotics, VLA, simulation-first development — I'd be glad to compare notes.

---

#Robotics #NVIDIA #Blackwell #IsaacSim #Switzerland #SimulationFirst #VLM #VLA #UnitreeGo2 #DGXSpark #ROS2
