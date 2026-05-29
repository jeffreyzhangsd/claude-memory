---
name: Always use subagents for parallel or multi-area tasks
description: Execution preference — use subagents whenever tasks are parallel or touch different areas of the codebase
type: feedback
---

Always use subagents when executing multiple tasks that are independent or touch different areas of the codebase. Don't execute them inline in the main session.

**Why:** User prefers subagent-driven execution for parallelism and to keep the main context clean. Applies to superpowers plan execution and any multi-task implementation work.

**How to apply:** When about to execute a plan or multiple tasks, default to subagent-driven development (superpowers:subagent-driven-development skill). Don't ask — just use it.

## Related

[[subagent-driven-dev]]
