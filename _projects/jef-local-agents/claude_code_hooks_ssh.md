---
name: claude-code-hooks-ssh
description: "When working from SSH-tmux to Mini, Notification hook is the right Discord signal; do NOT add Discord to Stop (per-response = too noisy). afplay on Mini is harmless even though user can't hear it from Air."
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

When Claude Code runs on the Mini and the user is SSH-tmux-attached from the Air, hook behavior should follow this preference:

- **Notification hook** (`~/.claude/hooks/notify-input-needed.sh`) — SHOULD fire Discord. This is the "Claude needs your input" signal. As of 2026-05-15, the hook is SSH-aware: when `SSH_TTY`/`SSH_CLIENT`/`SSH_CONNECTION` is set, it bypasses the "is a terminal frontmost on this Mac?" suppression and always schedules the Discord ping.
- **Stop hook** — keep `afplay /System/Library/Sounds/Glass.aiff` as-is. Do NOT add Discord posts on Stop, even with SSH-gating. Stop fires ~once per Claude response; in an active session that's dozens of Discord pings per hour. User explicitly chose "leave as-is" on 2026-05-15.

**Why:** Per-response Discord pings would drown out actually useful signals. The Notification hook already covers "Claude is idle/waiting for you, come back" — the use case the user actually wants alerted on. The wasted afplay on the Mini is acceptable since it's harmless and the Discord ping from Notification gives the audible-on-Air alert (Discord's own client sound) when it matters.

**How to apply:**

- If the user asks "why isn't my Discord webhook firing from these sessions?" — check `is_remote_session()` / SSH env detection in the Notification hook before assuming it's the webhook URL or osascript permissions.
- If the user wants per-response feedback on the Air, suggest `NTFY_TOPIC` (phone push) rather than wiring Discord into Stop. The existing notify-input-needed.sh script already supports `NTFY_TOPIC` alongside `DISCORD_WEBHOOK`.
- Don't propose Stop→Discord again unless the user explicitly asks for "every response" alerts.

See [[dev-machines]] for the underlying SSH-tmux workflow.
