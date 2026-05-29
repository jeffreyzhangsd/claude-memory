---
name: discord-ping-session-labels
description: "Discord input-needed pings are tagged [hostname/cwd] so concurrent sessions are distinguishable; Stop-hook Discord intentionally disabled to avoid spam"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 30bb70d1-cc5b-4348-9fd4-424c021dcde0
---

The Discord webhook is shared across every Claude Code session the user runs (Mini + Air, multiple CWDs concurrently). To tell which session is pinging, `~/.claude/hooks/notify-input-needed.sh` prepends `[<hostname-short>/<cwd-basename>]` to the message. Example: `🔔 [jefmini/wiki] Claude Code needs your input`.

Implementation: hook reads `cwd` from the JSON stdin (Claude Code provides `session_id`, `transcript_path`, `cwd`), then prepends `[$(hostname -s)/$(basename "$cwd")] ` to `$msg`. Label flows through the sentinel + banner + Discord body unchanged.

**Why:** User has several Claude sessions running on Mini + Air across different folders (`jef-local-agents`, `wiki`, `Career`, etc.); without the tag every Discord ping looked identical and they couldn't tell which session needed attention.

**How to apply:**

- Notification (input-needed) Discord ping = ENABLED + LABELED. This is the right level of signal — fires only when Claude is actually waiting on the user.
- Stop-hook Discord ping = INTENTIONALLY DISABLED. Adding one would fire on every turn completion = spam. Stop hook stays as `afplay` + `jef-postcheck.sh` only.
- If extending labels elsewhere (e.g. agent daemons' Discord posts from `career`/`hobby`/`briefing`), reuse the same `[hostname/cwd]` shape so the user reads everything the same way. Those daemons currently post bare summaries — leave them unless the user asks; they're already self-identifying by content.
- Hook script body is re-read on each fire, so edits land without restarting sessions. Only `settings.json` edits need a session restart (see [[feedback-settings-local-no-reload]]).

Related: [[claude-code-hooks-ssh]] (the same hook's SSH-aware banner/Discord routing logic).
