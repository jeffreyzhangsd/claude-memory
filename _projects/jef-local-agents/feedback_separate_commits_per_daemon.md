---
name: feedback-separate-commits-per-daemon
description: "When shipping a multi-daemon/multi-script batch, prefer one commit per logical daemon (or tight group) over a single mega-commit."
metadata:
  node_type: memory
  type: feedback
  originSessionId: 297d2bef-c435-4352-ae19-c891b37ab30e
---

When shipping a batch of multiple daemons/scripts in one session, commit each daemon (or a tight logical group) **separately** rather than as one giant commit. Said 2026-05-28 after two 9–10 file mega-commits (`96d47b1`, `fd16170`).

**Why:** smaller commits are easier to review, revert, and bisect; a mega-commit ("ship 9 daemons") buries which change broke what.

**How to apply:** during integration, after each daemon is smoke-tested green, stage just its files (`<name>.py` + `launchd/com.local.<name>.plist` + its slice of shared edits) and commit it on its own. Batch shared/doc edits (CLAUDE.md table, requirements.txt, ideas-backlog marks) into a final "docs + schedule" commit, or fold each daemon's row into its own commit. Still push once at the end. Pairs with [[project_github_for_cross_context_sync]] and the lifted git ban in this repo's CLAUDE.md.
