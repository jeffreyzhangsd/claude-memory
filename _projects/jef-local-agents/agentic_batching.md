---
name: agentic-batching
description: "Batch local agentic/Hermes work so models don't swap mid-pipeline; serialize if pipelines must share Ollama"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

When designing pipelines that use [[dev-machines|the Mini's]] local models, batch agentic work (Hermes/MODEL_AGENT) into discrete runs rather than interleaving it with other model calls. If two pipelines both need Ollama, run them sequentially — let one batch finish before the next starts.

**Why:** `OLLAMA_MAX_LOADED_MODELS=1` is fixed on the 16GB Mini. Mixing Hermes calls with gemma calls forces a ~0.8s cold-load on every switch, and a busy day could rack up dozens of evictions. Confirmed approach 2026-05-14 after I flagged the swap cost — user agreed batching is the right pattern.

**How to apply:**

- When wiring Hermes into career-ops grading, daily brief, or wiki research: queue all Hermes calls into one pass per run, not one-call-per-item interleaved with gemma triage.
- Daemons that compete for Ollama (e.g. triage daemon + nightly career grader): schedule them in non-overlapping windows or guard with a lockfile, don't let them race.
- For new pipelines, prefer "load model → process N items → release" over "per-item load/release".
