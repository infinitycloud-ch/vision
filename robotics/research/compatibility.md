# Compatibility — State of the Go2 policy ecosystem

## TL;DR

None of the three major open-source repos for Go2 walking are directly compatible with **Isaac Sim 5.1 / Isaac Lab 2.3** (our stack).

They all rely on **Isaac Gym Preview 4** — deprecated by NVIDIA, replaced by Isaac Lab. Observation spaces differ radically, which prevents drop-in use of any pre-trained policy.

---

## Compatibility matrix

| Repo | Robot | Sim required | Obs dims | Action dims | Pre-trained | Compatible with Isaac Sim 5.1 |
|---|---|---|---|---|---|---|
| **Teddy-Liao/walk-these-ways-go2** | Go2 | Isaac Gym Preview 4 | 70 + 2100 history | 12 | YES (51 ckpts + JIT) | ❌ |
| **Improbable-AI/walk-these-ways** (MIT) | Go1 | Isaac Gym Preview 4 | 42 | 12 | YES (Go1 only) | ❌ |
| **unitreerobotics/unitree_rl_gym** | Go2, G1, H1 | Isaac Gym Preview 4 | 48 (Go2) | 12 | NO for Go2 | ❌ |
| **Our setup (Isaac Lab 2.3)** | Go2 | Isaac Sim 5.1 | **235** (height scan) | 12 | rough_model_7850.pt | — |

---

## Per-repo details

### Teddy-Liao/walk-these-ways-go2

**Strengths:**
- Pre-trained on Go2 (51 checkpoints + JIT body + adaptation module).
- Supports gait conditioning: trot, pronk, bound, pace.
- Teacher-student architecture with history-based adaptation module.

**Blocker:**
- Observations: 70 dims per step + 2100 dims of history (30 steps × 70).
- Format completely different from our 235-dim obs (which includes a 187-dim height scan).
- Practical decision: not directly portable.

**Estimated porting effort:**
- Rebuild the obs vector respecting exact ordering.
- Implement the 30-step history buffer.
- Load the JITs with the correct device map.
- 1–2 weeks of work for an experienced Isaac Lab developer.

### Improbable-AI/walk-these-ways

The MIT reference, **Go1 only**. Models not transferable to Go2 (different URDF, dynamics, masses). Useful only as a reference implementation to understand the architecture.

### unitreerobotics/unitree_rl_gym

Official Unitree repo. Multi-robot (Go2, G1, H1, H1_2). Good code quality, but:
- No pre-trained Go2 policy provided (you have to train it yourself).
- 48-dim obs — incompatible with our 235-dim obs.

Decision: useful as a reference for config (PD gains: Kp=20, Kd=0.5, decimation=4) but not drop-in.

---

## Our choice

**Our strategy:** stay on the native Isaac Lab `rough_model_7850.pt` + software corrections (PID yaw).

Reasons:
1. Immediate compatibility with Isaac Sim 5.1 / Isaac Lab 2.3.
2. The PID yaw correction solves 90% of the drift problem without touching the policy.
3. Any eventual fine-tuning will happen in the native Isaac Lab framework.
4. walk-these-ways brings interesting gait conditioning but **not critical** for our use case (assistive robotics, slow safe navigation).

---

## If you do want to port walk-these-ways-go2

Minimal steps:

1. **Load the JITs**:
   ```python
   body = torch.jit.load("checkpoints/body_latest.jit")
   adapt = torch.jit.load("checkpoints/adaptation_module_latest.jit")
   ```

2. **Rebuild the obs vector** (70 dims, ordering matters):
   - projected_gravity (3)
   - commands (15) — vx, vy, yaw, height, frequency, gait[3], duration, footswing, pitch, roll, stance_width, stance_length
   - dof_pos (12)
   - dof_vel (12)
   - actions (12)
   - prev_actions (12)
   - clock_inputs (4)

3. **Maintain the history buffer**: ring buffer of 30 steps × 70 dims = 2100.

4. **Inference**:
   ```python
   latent = adapt(obs_history)
   action = body(torch.cat((obs_history, latent), dim=-1))
   ```

5. **Validate PD gains and decimation**: Kp=25, Kd=0.6, decimation=4, action_scale=0.25.

If everything is aligned, the policy should run. **Not tested here** — decision made not to invest the time as long as `rough_model` + PID is enough.
