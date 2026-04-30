# Roadmap — What's next, and what's left to prove

> No marketing promises. Concrete technical milestones, dated where possible.

---

## Short term (1–2 months)

### Sim-to-real Go2 boustrophedon (Tier 1)
**Status:** in progress.
- `go2_bridge.py`: `/move`, `/stop`, `/status` endpoints ✅ (April 2026).
- BoustrophedonService Swift on iPhone: in progress on the MonoCLI team side.
- Physical test on a 10×10m zone expected: May 2026.
- Success criterion: 100/100 cells covered, 0 falls, ARKit tracking maintained.

### Patrol mission with animal/human discrimination (physical)
**Status:** validated in simulation, to port to the real Go2.
- VLM + identification pipeline already E2E in sim.
- Remaining work: validate in a real environment with 3D-printed assets or figurines.

### BME690 olfactory sensor
**Status:** spec'd, hardware not yet ordered.
- Gas detection, scent-signature search.
- Use case: fall detection (urine) in nursing homes.
- ETA: during Q2 2026.

---

## Medium term (3–6 months)

### G1 humanoid — autonomy in a structured environment
**Status:** GR00T N1.7 inference OK, autonomy 0.

Goals:
- Point-to-point navigation in a simulated nursing-home room.
- Simulated fall detection and alerting.
- Natural dialogue with a simulated patient.

### Apple Vision Pro cockpit — clinical deployment
**Status:** TDD v2.0 finalized, implementation not started.
- Visualise the robot's "thoughts" in MR.
- Demonstration-based learning / VLA via Vision Pro.
- ETA: Q3 2026.

### Enhanced Hy3D pipeline
**Status:** 3 GLBs delivered, pipeline reproducible.

Goals:
- 20+ nursing-home scene assets (beds, chairs, lighting, signage).
- Automatic generation of full scenes (procedural + Hy3D).

---

## Long term (6–12 months)

### Pilot in a real Swiss nursing home
**Status:** preliminary discussions with HES-SO, Fegems.
- Go/no-go criterion: success of Tier 1 + Tier 2 sim-to-real tests.
- Risk: medical regulation, GDPR, safety.
- Strategy: start in passive observation mode (no physical interaction with patients).

### Venture Kick / funding validation
**Status:** pitch v1 drafted.
- CHF 5,000 companion targeting the Swiss nursing-home market.
- Application for CHF 150k Phase 1.

### Partial open-sourcing
**Status:** under consideration.

Decision pending: which bricks to make public?
- **Likely open-source:** API contracts, Isaac Sim templates, validation pipelines, ICSD memory format.
- **Likely proprietary:** MonoCLI engine, full agentic methodology, nursing-home integrations.

---

## Acknowledged limits

What this roadmap explicitly does **not** include:

- **Training foundational policies from scratch** — reserved for actors with H100 clusters.
- **Custom hardware** — sticking with Unitree + iPhone + Vision Pro + DGX Spark.
- **Rapid internationalization** — Switzerland focus first (then French-speaking Europe).
- **Major fundraising** — the goal is to demonstrate that progress is possible without it.

---

## Progress metrics

At every sprint, two questions:

1. **What works that I couldn't demonstrate last sprint?**
2. **What still doesn't work, and why?**

No fancy OKRs. No dashboard KPIs. Just these two questions, every review.

That's enough to move fast. And it forces honesty.
