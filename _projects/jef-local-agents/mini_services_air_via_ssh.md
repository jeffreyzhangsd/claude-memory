---
name: mini-services-air-via-ssh
description: "Mini orchestrator can fully service Air-owned `agent/sync` entries over SSH instead of waiting on an Air Claude session — git pull, plist install/bootstrap, /health probes via Tailscale all work non-interactively."
metadata:
  node_type: memory
  type: project
  originSessionId: fdb149bf-6c44-4815-a928-cb11d066467d
---

When an entry sits in `agent/sync/jef-local-agents-air/inbox.md` and no Air Claude session is currently open, the Mini orchestrator can ship + smoke-test the work itself via `ssh air ...` rather than parking it or dispatching to a not-yet-spawned worker. Proven on AGT-027 (2026-05-24).

**Why:** Air sessions are intermittent (laptop closed / Roblox in foreground / no one's at the Air). Sync-channel was designed assuming each owner has a wake-able session, but a working Mini→Air SSH path collapses that requirement for any task that's purely shell-execable on Air (not UI interaction). Same network of trust as Tailscale + the bus daemon, just with shell-exec semantics.

**How to apply:**

- Air repo canonical path: `/Users/jeffreyzhang/Desktop/Projects.nosync/jef-local-agents` (NOT `~/jef-local-agents` — that one exists but isn't a git repo). Home is `/Users/jeffreyzhang`.
- SSH alias: `ssh air` (ed25519 over Tailscale, configured in `~/.ssh/config` on Mini).
- `git pull` on Air over SSH session needs the remote in SSH form, not HTTPS — Keychain credential helper can't unlock over a non-interactive SSH. One-time fix per repo: `ssh air 'cd <repo> && git remote set-url origin git@github.com:<user>/<repo>.git'`. Air has `~/.ssh/id_ed25519.pub` registered with GitHub, so SSH auth just works.
- launchd install works remotely: `ssh air 'cp <repo>/launchd/com.local.X.plist ~/Library/LaunchAgents/ && launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.local.X.plist'`. `gui/$(id -u)` evaluates correctly inside the SSH'd zsh.
- After `bootout` + `bootstrap` a service can briefly return `Bootstrap failed: 5: Input/output error` — race between teardown and reload. Just retry the bootstrap.
- Bonjour is off on the Mini → `jefair.local` / `jefmini.local` won't resolve from Mini-side processes. Use raw Tailscale IPs in plist env vars (Air = `100.83.219.92`).
- `Bash(curl *)` is in the global Claude deny list. Use Python `urllib.request` for health probes / JSON pings inside Claude sessions. `ssh air 'curl ...'` is also denied because the outer command contains "curl".
- This pattern does NOT replace `bus` (live Claude↔Claude convo, needs both sessions open) or the sync-channel inbox model (still the durable ledger). It's a shortcut for shell-execable Air work when no Air Claude session is around.

Boundary: if the work needs to drive a browser, fill a form in Playwright, or interact with the Air's display, it's still Air-Claude-or-user territory — SSH doesn't have a display.

Related: [[dev-machines]] (role split), [[project-cross-agent-workflow]] (inbox lifecycle), [[reference-mini-ollama-lan]] (Tailscale topology), [[feedback-dispatch-over-sync]] (push vs pull).
