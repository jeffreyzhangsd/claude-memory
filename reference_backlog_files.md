---
name: reference-backlog-files
description: TRIGGER on any of these phrases — "backlog", "what's next", "next item", "next task", "continue", "pick something", "ship X items", "work on backlog", "anything to build", "another one", "do the next one", "what should I build". When ANY of these appear, READ `<vault>/ideas-backlog.md` and `<repo>/IDEAS.md` BEFORE asking the user what to work on. Do not ask — surface the backlog ranking yourself.
metadata:
  type: reference
---

# Backlog files — check these first

## Trigger phrases (auto-apply)

If the user's message includes any of these, READ the backlog files FIRST. Do not ask "what would you like to work on" — read the files and present a ranking.

- "backlog" / "the backlog"
- "what's next" / "next item" / "next task" / "the next one"
- "continue" / "continue working"
- "pick something" / "what should I build"
- "ship X items" / "ship N more"
- "another one" / "another backlog item"
- "any open work" / "anything to build"

## Files, in order

1. **`<vault>/ideas-backlog.md`** — canonical cross-project dormant ideas + future-project research stubs. Each entry has a revisit trigger. Look here first for "ship anytime" items.
2. **`~/jef-local-agents/IDEAS.md`** — active per-repo backlog for jef-local-agents. Numbered items, ✅ shipped, ⏸️ deferred-with-trigger, dropped sections.
3. **Other repo `IDEAS.md` files** — each project owns its own when it has a repo. Wiki / career / etc. live with the repo.
4. **`<vault>/jef-folder/temp/session-handoff-*.md`** — if a recent handoff note exists, it lists exactly where the previous session paused. Read it before re-ranking from scratch.

## Ranking workflow when picking

If the user asks "what's next" without specifying which item:

- If a backlog has an explicit numbered ranking → use that.
- If a recent handoff note exists with an in-flight item list → resume from there.
- Otherwise sort by **return ÷ build-time**: high-utility, low-hours items first. Show a table; let the user pick.

## What NOT to do

- Don't ask "what would you like to work on?" — read the backlog first.
- Don't surface "trigger-deferred" items as actionable — they're waiting on conditions (incident, 30d data, vault-clutter felt-pain). List them only if asked.
- Don't propose ideas not in these files unless the user asks for brainstorming.

## Related

- [[jef-local-agents]]
- `<vault>/MIND.md` "Future / exploratory" section points to ideas-backlog.md
- `~/.claude/skills/ship-daemon/SKILL.md` — pipeline for shipping a picked-up item
