---
name: calendar-tcc-launchd-grant
description: "macOS TCC (Calendar/Contacts/Photos/etc) attributes a grant to the RESPONSIBLE process — a request from Terminal/SSH grants Terminal, not the python binary, so launchd daemons stay denied. Grant via a one-shot launchd job so launchd is the parent and the grant attaches to the binary."
metadata:
  node_type: memory
  type: feedback
  originSessionId: 297d2bef-c435-4352-ae19-c891b37ab30e
---

When a launchd daemon (e.g. `calendar_conflict`, `briefing`'s EventKit block) needs a macOS TCC permission (Calendar, Contacts, Photos, Reminders, Full Disk…), you CANNOT grant it by running the request from a terminal.

**Why:** macOS attributes a TCC grant to the **responsible process**. A `requestAccess` call from an interactive shell makes **Terminal.app** (or the SSH/Claude-Code parent) the responsible process → the grant attaches to _Terminal_, not to `venv/bin/python3`. launchd daemons run python with launchd as the parent (no app), so Terminal's grant doesn't cover them — they stay denied. You also can't "+"-add an arbitrary CLI binary in System Settings; a binary only appears there after it has _requested_ access. And the consent dialog can't render over SSH/launchd-background, so a plain SSH request silently denies.

**Fix (works):** request from a **one-shot launchd job** so launchd is the parent and python is its own responsible process → the grant attaches to the python binary; all launchd daemons using that same binary inherit it. The dialog renders on the GUI (Aqua) session.

- `calendar_events.py --request` calls `requestFullAccessToEvents` (macOS 14+) + spins the NSRunLoop so the dialog appears.
- `launchd/com.local.calendar-request.plist` (RunAtLoad one-shot) runs it. Bootstrap on the Mac's screen / Screen Sharing, approve the dialog, then `launchctl bootout`.
- Verify it stuck via the launchd context, NOT a manual SSH run: `launchctl kickstart -k gui/$UID/com.local.calendar-conflict` then check its log shows no grant-hint. A manual `venv/bin/python3 -m calendar_events --dry-run` over SSH will STILL show "not granted" — that's the responsible-process quirk, not a real failure.

**How to apply:** any future daemon needing a TCC scope → ship a `--request` mode + a one-shot launchd plist; grant once from the GUI session; verify via `launchctl kickstart` of the real daemon, never via an SSH-context run. Done 2026-05-28 for Calendar. Pairs with [[dev_machines]] (Mini = launchd host).
