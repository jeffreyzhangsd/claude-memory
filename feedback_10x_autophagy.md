---
name: 10x autophagy / simplification rule
description: Default to far more cleanup, review, canonicalization, and adversarial testing than instinct suggests — by an order of magnitude.
metadata:
  type: feedback
---

# 10x autophagy rule

> "a rule of thumb here I think is just need to do about 10x more codebase autophagy/compression/simplification/systematic open-ended reviews/canonicalization, and adversarial testing than you probably want to do."

User shared this 2026-05-21 in the context of maintenance work (vault janitor, migration drift, daemon bug fixes). Treat it as the standing default for how to balance new work against upkeep.

**Why:** Cleanup work has a high local cost (slow, no visible feature delivered) but compounds — every accumulated piece of cruft taxes future sessions (load time, decision fatigue, broken refs, hidden bugs). The bias against it is strong because no individual cleanup task feels urgent. The user wants me to push past that bias.

## How to apply

- When choosing between "add feature X" and "clean up Y," default toward Y far more often than feels natural.
- Periodic open-ended audits — like the 2026-05-21 sweep that surfaced the topic-stub Air refs, the orphan plist, the 466 MB stale plugin, and the missing janitor — should be the norm, not the exception. Run them when nothing obvious is queued, not only when triggered.
- Adversarial testing: when shipping a fix, also write the test that would have caught the original bug AND probe related code paths for the same shape of failure (the discover→fix→verify loop with cross-model audits is one manifestation).
- Canonicalization: when two paths/configs/specs say similar things, fold them into one source of truth before doing anything else with them.
- Compression: prefer deletion over wrapping. Old code paths, dead flags, "just in case" branches — cut.
- Systematic ≠ exhaustive. The 10x bump is about applying maintenance habits more frequently and broadly, not about over-engineering any single sweep.
- For "should I clean this up now or note it for later" judgment calls, lean toward now if the cost is bounded (single-session, no irreversible moves) and the cruft is load-bearing on session context (CLAUDE.md, MEMORY.md, topic stubs, daemon code).

## Triggers that should prompt a sweep

- About to start new work ("before we build, are we clean?")
- After shipping a feature ("what cruft did this leave behind?")
- When a session's prompt cache misses unexpectedly (something is bloated)
- When tests fail in unrelated files (signal that someone touched shared state)
- Monthly via [[vault-janitor]] for the mechanical layers (plugin cache, transcripts, agent/done, wiki/meta, logs)

## Related

- [[vault-janitor]]
- [[orchestrator-dispatch]] — discover→fix→verify loop is a form of systematic adversarial review
- [[code/projects/jef-local-agents-patterns-2026-05-21]] — CLAUDE.md trim-and-relocate is the same pattern at the doc layer
