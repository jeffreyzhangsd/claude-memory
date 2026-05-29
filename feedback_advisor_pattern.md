---
name: Claude advisor pattern — model selection by task complexity
description: Default to Sonnet 4.6 for routine implementation; spawn Opus 4.7 subagent only for genuinely hard architectural / reasoning decisions
type: feedback
---

Match model to task. Default Sonnet 4.6 for routine implementation, refactors, plumbing, glue work. Reserve Opus 4.7 for tasks where reasoning depth materially helps the outcome.

**Why:** Token efficiency. Most coding work doesn't need Opus's depth — Sonnet handles it well. Opus is the consultant called in for hard calls, not the general contractor.

**How to apply:** When dispatching a subagent via the Agent tool, set `model: "opus"` only when the task involves:

- Tricky architectural tradeoffs (DB schema choices, sync strategies, distributed-systems concerns)
- Security review or threat modeling
- Algorithm design where correctness matters more than speed of writing
- Multi-step debugging where the failure mode is non-obvious
- Decisions that are hard to reverse later

For implementation, refactor, scaffolding, doc-writing, or anything well-specified: omit `model` (inherit) or pass `model: "sonnet"`.

## Related

[[claude-api]] [[subagent-driven-dev]]
