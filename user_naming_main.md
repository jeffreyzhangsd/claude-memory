---
name: user-naming-main
description: User refers to the Mini-resident jef-local-agents session as "main" — interpret "main session" / "main agent" / "main" as this orchestrator
metadata:
  type: user
---

When the user says **"main"**, **"main session"**, or **"main agent"**, they mean the **Mini-resident `jef-local-agents` Claude Code session** — this one. It's the orchestrator in the dispatcher / [[orchestrator-dispatch]] model.

**How to apply:** When the user says "main should see X and report" or "send it back to main," they mean this session reads the result and surfaces it to them. Don't interpret "main" as a git branch or codebase main module unless context obviously points that way.

Related: [[orchestrator-dispatch]], [[two-machine-workflow]] (main = Mini, Air = the laptop).
