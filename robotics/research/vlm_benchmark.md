# VLM Benchmark — Vision-Language Models for robotic patrol

## Setup

**Test image:** Isaac Sim camera 640×480 JPEG (quality 80, ~25 KB).
**Prompt:** "Describe what you see in 2 sentences. Is there a person or an animal?"
**Date:** March–April 2026.
**Hardware:** cloud calls from Mac M3 Ultra or Spark (10 Gbit Ethernet).

---

## Results

| Model | p50 latency | Throughput | Cost (~) | Verdict |
|---|---|---|---|---|
| **Groq Llama-4-Scout 17B** | 0.34 s | 187 tok/s | <USD 0.001/req | Lowest latency |
| **OpenAI GPT-4o** | 3.57 s | 7.6 tok/s | ~USD 0.005/req | Current production |
| **Ollama qwen3-vl (local Spark)** | 9.15 s | 38 tok/s | USD 0 (local) | Offline fallback |
| **Nemotron-12B-VL** | n/a | n/a | n/a | Blocked on aarch64 |

---

## Per-model details

### Groq Llama-4-Scout 17B

**The fastest.** 0.34s end-to-end latency includes the network. Faster than what's needed for patrols.

**Observed limits:**
- Cloudflare initially blocked vision requests without a User-Agent header (error 1010). Fixed by adding `User-Agent: patrol/1.0`.
- Description slightly more generic than GPT-4o on complex scenes.
- Groq's preview vision models (`llama-3.2-11b-vision-preview`, `llama-3.2-90b-vision-preview`) were decommissioned in early April 2026 — only Llama-4-Scout supports vision on Groq now.

**Use case:** routine scans where latency dominates (every 5s during patrol). No longer used for critical identification decisions.

### OpenAI GPT-4o

**Current production.** Migration from Groq decided in April 2026.

**Strengths:**
- Higher reasoning quality on simulated scenes (non-photoreal meshes).
- Capable of understanding contextualized prompts (e.g. "The Doctor in this simulation is represented by a large blue shape").
- Stable API, complete documentation, standard multimodal format.

**Weaknesses:**
- 10× higher latency than Groq.
- Non-trivial cost at scale.

**Current use case:** critical identification + access-authorization decisions. The latency is acceptable because the decision is intermittent (a few times per mission, not in a tight loop).

### Ollama qwen3-vl (local on Spark)

**Offline fallback.** Runs entirely on the DGX Spark, no cloud dependency.

**Strengths:**
- USD 0 per request, scalable.
- No data leakage (all images stay on Spark).
- 9s latency acceptable for asynchronous analyses.

**Weaknesses:**
- 25× slower than Groq.
- Quality slightly below GPT-4o on subtle scenes.

**Use case:** degraded mode (no internet) or offline batch processing.

### Nemotron-12B-VL (NVIDIA)

**Blocked.** The NVIDIA Nemotron Vision-Language model requires `mamba-ssm`, which has no aarch64 wheel at the time of testing.

Worth watching — situation may change if NVIDIA ships an official ARM wheel.

---

## Current production strategy

```
Critical decision (human identification) → GPT-4o
   └─ acceptable since latency < 5s, intermittent

Routine scan (every 5s) → GPT-4o (quality wins here too, to avoid false positives)
   └─ may shift to Groq if volume becomes a factor

Offline / batch scan → local Ollama qwen3-vl
   └─ zero marginal cost, asynchronous
```

---

## Negation regex filter

**Problem observed without filter:** the VLM describes "There are no visible people" → naive matcher finds the word "people" → false positive.

**Solution:** regex filter on 4 negation patterns (FR/EN), applied after the human-keywords match:

```python
NEGATION_PATTERNS = [
    r"(?:pas|no|not|aucun|sans|n.y a pas)\s+(?:de\s+)?(?:" + "|".join(HUMAN_KEYWORDS) + r")",
    r"(?:" + "|".join(HUMAN_KEYWORDS) + r")\s+(?:not|aren.t|isn.t|n.est pas)",
    r"(?:no|zero|aucun)\s+\w*\s*(?:" + "|".join(HUMAN_KEYWORDS) + r")",
    r"n.y a\s+(?:" + "|".join(HUMAN_KEYWORDS) + r")",
]
```

**Metric:** across 16 post-filter test scans, **100% of false positives eliminated**. No false negatives observed (every real human was detected).

---

## Lessons learned

1. **Always send a User-Agent** on cloud VLM APIs. Cloudflare blocks otherwise.
2. **VLM latency forces an asynchronous design.** The robot can't wait 3s to decide — either you stop during the scan, or you run a parallel pipeline.
3. **Non-photoreal 3D models trip up VLMs.** A blue sphere representing a doctor isn't recognized. Solutions: better assets (Hy3D), or contextually-tuned prompts.
4. **The negation filter is non-negotiable.** Without it, ~80% false positives. With it: 0%.
