---
name: feedback-settings-local-no-reload
description: "settings.local.json permission edits don't reload mid-session — the harness uses the permission map from session start"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 698a2418-1265-436c-9955-ad567d520623
---

When `update-config` skill (or a direct edit) adds new entries to `.claude/settings.local.json` mid-session, those new permissions do NOT take effect in the current session. The Claude Code harness loads the permission map at session start and doesn't watch the file for changes.

**Why:** confirmed 2026-05-17 — added `Bash(git push *)` and ran `git push`; got "Permission to use Bash with command git push has been denied" even though the new entry was correctly written and `jq` validated the JSON. Same constraint that the `update-config` skill docs note for hooks ("the settings watcher isn't watching `.claude/` — it only watches directories that had a settings file when this session started").

**How to apply:**

- When adding permissions the user might want to use _in the current turn_: warn them up front that they'll need to push/run the affected command themselves OR restart the session for the permission to take effect.
- New permissions ARE live in any session opened AFTER the edit — so the cost is just "use the new permission starting next session," not "the settings change failed."
- The `/hooks` slash command supposedly reloads config but it's a UI menu and opening it ends this turn — can't trigger from a tool call.
- Don't waste the user's time retrying the same denied command after editing settings; just tell them to do it manually OR open a new session.

Related: [[project-coding-workflow]] (when changing my own permissions, plan for one-session-delay before the change is usable).
