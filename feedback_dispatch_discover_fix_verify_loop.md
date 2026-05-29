---
name: feedback-dispatch-discover-fix-verify-loop
description: Wiki dispatches are for DISCOVERY and VERIFICATION; daemon-code fixes happen in main. Loop is wiki→main→wiki, not wiki→wiki.
metadata:
  type: feedback
---

# Dispatch workflow — discover → fix → verify loop

When [[orchestrator-dispatch]] uncovers bugs in daemon-produced wiki content, the wiki worker session is the **detector and verifier**, never the fixer of daemon code. The loop is:

1. **`/dispatch wiki "audit / scan / look for X"`** — read-only discovery dispatch. Wiki worker returns findings + suggested fixes.
2. **Main investigates the daemon root cause.** Read the relevant `wiki_builder.py` / `vault_harness.py` / `hobby/research.py` code, identify why the bug was produced (prompt drift, missing filter, dead code path, etc.). Fix at the source, add a regression test, run pytest. This work happens IN the main session OR via a self-dispatch to `jef-local-agents` — but stays in code-repo land.
3. **`/dispatch wiki "re-audit / verify X is gone"`** — verification dispatch. Wiki worker confirms the bug is no longer present in rendered content (after `vault_harness` / `wiki_builder` re-runs OR by re-checking that the fix prevents recurrence).

**Why:** Wiki workers can't edit daemon code (`<vault>/wiki/` CWD has no access to `~/jef-local-agents/`). Even if they could, daemon code is the orchestrator's territory — wiki sessions per their CLAUDE.md are content-only.

**How to apply:**

- Don't `/dispatch wiki "fix the daemon bug"` — wrong cwd, wrong scope.
- Do `/dispatch wiki "audit for X"` — get a clean read-only finding.
- Then fix in main / self-dispatch to `jef-local-agents` — that's where the daemon code lives.
- Then `/dispatch wiki "verify X is now clean"` — close the loop.

Pattern proven 2026-05-21: AGT-021/022 template-leak loop, then wiki source-quality audit → main fix → wiki re-audit found 2 NEW bugs Opus missed (smoke-test pollution, ML sources/ emptiness). Cross-model second pass (Sonnet verifying Opus) is a free bonus — catches confidently-wrong hallucinations.

## Related

- [[orchestrator-dispatch]]
- [[main]] (the orchestrator)
- [[reference-dispatch-worker-preseed]]
- IDEAS.md #21 in `~/jef-local-agents/`
