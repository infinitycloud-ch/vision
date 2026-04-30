# Methodology — Simulation-first, ICSD, iteration discipline

## Rule 1 — Nothing touches the physical robot before 100 simulation runs

A real G1 fall costs ~CHF 8,000. A simulated fall costs an `env.reset()`. That asymmetry is the foundation of the whole method.

**Practical consequence:** simulation quality is itself a deliverable.

Half a day lost this session on a friction mismatch between the Isaac ground plane and Unitree's `velocity_env_cfg`. Another half-day on observation-space compatibility between walk-these-ways and Isaac Lab 2.3. These aren't accidents — they're the **real cost** of simulation-first. And it's worth far more than a broken G1.

---

## Rule 2 — Keep the stack thin

No Kubernetes orchestrator. No microservices. No message broker. Three Python scripts, one UDP bridge, one Three.js viewer. When something breaks, I know why in five minutes.

For comparison: an equivalent project I observed at a large group used Airflow + Kafka + Kubernetes + 4 separate Python services. At the slightest bug, 30 minutes digging through logs before locating the offending layer.

Complexity has a huge hidden cost for an independent developer. **Every added brick must earn its place.**

---

## Rule 3 — ICSD (Inferred Context Semantic Density)

This is the communication system developed for the agent farm (Claude Code).

**Principle:** dense pseudo-code > descriptive prose. 1 line = 1 concept. Compression > exhaustiveness.

ICSD-style memory example:
```
policy = rough_model_7850.pt — MLP[512,256,128] — obs235 — act12
policy.problem = yaw_drift_80deg_in_30s — pure_rotation(vx=0)=fall
policy.fix = PID_yaw(Kp=0.03,Ki=0.003,Kd=0.003) + arc_turn(vx=0.1+wz)
walk-these-ways = INCOMPATIBLE(Isaac_Gym_Preview4 ≠ Isaac_Sim_5.1)
```

These 4 lines hold the essence of a 6-hour debug session. An agent re-reading them after a refresh reconstructs context in seconds.

**Trade-off:** unreadable to anyone without context. That's by design — it's **technical memory** between self and self (and between peer agents), not public documentation.

---

## Rule 4 — Proof by image

Before declaring a visual task done: **screenshot mandatory**. No "should be working". No "diff looks right". Either you can see it, or you're not done.

For robotics: a custom Three.js viewer (port 8080 on Spark) showing real-time Go2/G1 position, assets, mosaic grid, walls. Any claim about robot behavior comes with a viewer screenshot.

---

## Rule 5 — Honesty about blockers

If something doesn't work, say it. Right away. To the right channel.

Honest list of currently unresolved items (as of 30 April 2026):
- Sim-to-real not crossed (everything is in simulation for the complex cognitive missions).
- Visual identification of non-photoreal assets is fragile (simplified Hy3D mesh ≠ humanoid recognizable by GPT-4o).
- RL policy fatigue after ~50s in production (last_action accumulates drift).
- No olfactory sensor integrated (BME690 planned, not yet implemented).

These are **public in the repo and the roadmap**. Not out of naivety — strategy. A project that only shows successes is suspect. A project that shows its limits too is credible.

---

## Rule 6 — Iterate over long-form planning

No 18-month roadmap. No multi-level Gantt charts. 1–2 week sprints with a measurable deliverable, and an immediate review.

Sprint 16 → 17 → 18: three sprints, three different proofs, in one month. Each unblocked the next.

This discipline is typical of successful research projects. It requires **a single decision-maker** or a tightly aligned team. With 12 engineers and middle management, this velocity becomes impossible.

---

## Rule 7 — End-of-prompt predicts the next move

At the end of every agent interaction, **propose one (and only one) logical next action**. Not a menu, not a list — a suggestion that flows naturally from the reasoning.

This is semantic prediction of the next iteration. When done well, the decision-maker only needs to say "go" to advance. When done poorly, you lose 5 minutes clarifying.

---

## Summary

| Rule | Measurable effect |
|---|---|
| Simulation-first | 0 broken robots in 6 months |
| Thin stack | Average debug time <5 min |
| ICSD | Post-refresh reload in seconds |
| Proof by image | 0 undetected visual regressions |
| Honest blockers | Technical credibility vs vendors |
| Short iterations | 3 major sprints / month |
| Next-step prediction | Decisions in 1 word vs 5 min |

None of these rules are new. None are revolutionary. But **applying them all simultaneously** without compromise is unusual — and that's what produces the velocity observed.
