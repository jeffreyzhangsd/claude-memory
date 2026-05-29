---
name: project-github-for-cross-context-sync
description: "Push to origin/main proactively in jef-local-agents — the private GitHub repo doubles as the cross-Claude-instance context sync layer, not just backup"
metadata:
  node_type: memory
  type: project
  originSessionId: 698a2418-1265-436c-9955-ad567d520623
---

`jeffreyzhangsd/jef-local-agents` (private) serves a dual purpose:

1. Code repo (obvious)
2. Cross-Claude-instance context sync — any future session, on any device (Mini, Air, Web Claude at claude.ai/code, a third machine spun up later), gets the latest state by pulling

**Why:** decided 2026-05-18 by the user explicitly: "i realized it would be good to use this private git repo for claude to recieve context and stuff anyway across the app". Means commits aren't just durability — they're how a future Claude bootstraps with current truth.

**How to apply:**

- Commit + push more aggressively in this project than the default cadence. After any non-trivial fix (e.g. wiki_builder bug fix, daemon change), push to make the new state visible to any future session.
- `.claude/CLAUDE.md` IS in the repo and IS the right place to document persistent project state for future sessions to pick up.
- `.claude/settings.local.json` is gitignored — it stays per-machine. That's intentional (permissions are personal).
- Default permission grant in this project (settings.local.json) allows `git add/commit/push` — see [[feedback-settings-local-no-reload]] for the gotcha that this only takes effect in sessions opened AFTER the grant landed.
- Push timing: after every commit, not "when convenient." The cost is ~1s; the value is that the next session opening on the Air or via Web Claude has truth.

Related: [[dev-machines]] (Mini primary, Air secondary), [[feedback-settings-local-no-reload]] (settings reload limitation).
