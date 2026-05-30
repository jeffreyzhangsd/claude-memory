---
name: feedback-deepseek-default-offload
description: "Operating policy — Claude is the thin orchestrator; defer bounded text tasks to DeepSeek (utils/deepseek.py, flash/pro) by default and verify. Claude subagents only when tools/repo access needed."
metadata:
  node_type: memory
  type: feedback
  originSessionId: c1d85370-6c6a-4870-bc95-b735fd0cc58e
---

# Default-offload to DeepSeek; Claude orchestrates + verifies

Standing directive (2026-05-29): Claude stays the OVERHEAD ORCHESTRATOR almost
all the time — route, decide, verify; don't do grunt work itself. Token
optimization on the Pro plan; the user is explicitly fine using DeepSeek heavily.

**Routing, in order of preference:**

1. **DeepSeek = the default subagent we defer to** — for bounded, SELF-CONTAINED
   text tasks where Claude can hand over all the context: explain / draft /
   transform / classify / summarize / answer a scoped question / adversarial
   review. Tool: `utils/deepseek.py` — `deepseek.ask(prompt, tier=...)` or CLI
   `venv/bin/python3 -m utils.deepseek "..." [--pro] [--file X]`. `tier="flash"`
   (deepseek-v4-flash, trivial, ~$0.14/$0.28 per M) is the default; `"pro"`
   (deepseek-v4-pro, reasoning, ~$1.74/$3.48) when it needs to actually think.
2. **Claude Haiku/Sonnet subagent (Agent tool)** — only when the small task
   needs TOOLS / repo search / file ops, which DeepSeek (text-in/text-out, no
   tools, no repo) can't do. Match model to task per [[feedback-advisor-pattern]].
3. **Claude (Opus orchestrator)** — routing, the hard calls, and ALWAYS verifying
   offloaded output before trusting or integrating it.

**Why:** Save Claude tokens; the user doesn't mind paying DeepSeek (cheap) and
wants Claude reserved for orchestration + judgment, not bulk text work.

**How to apply:**

- Default to DeepSeek for any bounded text task. But DON'T reflexively round-trip
  GENUINELY trivial things (a one-liner I already know) — CLI latency + the
  verify step costs more than just answering. Offload when the work is
  non-trivial enough to be worth it AND fully self-contained.
- Always read DeepSeek's output critically before using it — it can be wrong;
  verification is the whole reason Claude stays in the loop.
- flash for trivial, pro for reasoning. The build-time code-review antagonist
  (`utils/deepseek_review.py`) is the pro specialization, sharing the same
  `deepseek.chat()` client + cost path.

Related: [[deepseek-v4-antagonist-pipeline]], [[plan-claude-pro-deepseek-orchestrator]], [[feedback-advisor-pattern]].
