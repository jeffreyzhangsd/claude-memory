---
name: user-cost-constraint
description: User's economic constraint — Claude Max may not be affordable indefinitely. Bias every architectural choice toward local-AI + token efficiency as a hedge.
metadata:
  type: user
---

# Cost-conscious local-AI bias

User actively worried about long-term Claude Max affordability. Concrete signals:

- Saves Twitter notes on local-AI setups (sudoingX, leopardracer, Hermes/Ollama workflows) — see `<vault>/notes/jef notes.md` + `<vault>/topics/vram-math.md`
- Built `jef-local-agents` explicitly to "reduce Claude Max burn 30-40% via local preprocessing" — the framing predates today
- Asks repeatedly about ways to "save tokens" and "cut corners"
- Has 16GB Mac Mini M4 + 16GB Air M1 — modest hardware, not a power user setup

## How to apply

For every architectural / tooling choice in this user's projects, weigh:

1. **Can a local model do it adequately?** (gemma4:e2b for classification, gemma4:e4b for summarization/prose, hermes3:8b for constrained decisions, qwen3:14b for synthesis, qwen3.5:35b via llama.cpp for big-context heavy work) — see `<vault>/jef-local-agents/.claude/CLAUDE.md` for current task→model assignments
2. **Can the context be pre-compressed locally before Claude sees it?** (`jef condense` pattern — gemma4:e4b extracts root cause + key context from big errors / logs / files)
3. **Can the result be cached / re-used?** (avoid re-doing work — skill_tracker daily log, wiki_ask embed cache, etc.)
4. **Does this need to load at session-start, or can it lazy-load?** (CLAUDE.md trims: load-bearing index in CLAUDE.md, full detail in docs/ that Claude pulls on demand)

Claude is for irreducible reasoning + tool use that local models can't do reliably on 16GB. Not for tasks that gemma4 can handle.

## What this does NOT mean

- Don't refuse Claude usage when it's the right tool. Long-horizon reasoning, code generation past ~50 lines, multi-step tool use over 10+ steps — Claude is correct here.
- Don't degrade quality to save tokens on tasks the user cares about (cover letters, interview prep, real code).
- Don't pre-emptively switch existing high-quality pipelines to weaker local models without A/B (the `docs/model-decisions.md` discipline already in place).

## Related

- [[two-machine-workflow]] — Air dev / Mini runtime is part of the local-AI hedge
- [[jef-local-agents]] — main vehicle for local preprocessing
- [[jef-cli]] — `condense`, `prep`, `postcheck` commands route appropriate work to local
- [[vram-math]] — heuristic for what local models user's hardware can run
