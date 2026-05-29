---
name: Two-machine workflow — Air dev / Mini runtime — tag every command
description: User has split MacBook Air (dev) + Mac Mini (headless runtime). Always label shell commands with their target machine; know SSH alias + paths.
type: feedback
---

When showing shell commands or setup steps for any project that touches BOTH the MacBook Air (dev) and the Mac Mini (`jef-local-agents`, career scanner, TTS pipeline, anything with local Ollama/llama.cpp/Piper daemons), label every command block with the machine it runs on.

## Tags

- `[MacBook Air]` — local dev machine
- `[Mac Mini — SSH]` — headless runtime, accessed via `ssh mini`
- `[either]` — runs identically on both

Group steps by machine. Call out explicitly when work crosses machines (e.g. "rsync to Mini, then SSH in").

**Why:** User keeps a local Air terminal AND an SSH-into-Mini tmux session open simultaneously. Untagged commands have caused him to run things in the wrong place. He asked explicitly on 2026-04-30: _"clarify whether I am running it on the mac mini or this macbook air... try to do that every time so I don't run the wrong command in the wrong terminal."_

## Mac Mini access reference

- **SSH alias**: `ssh mini` — pre-configured in `~/.ssh/config`
- **Username**: `jef` (the macOS user on the Mini)
- **Hostname**: `jefmini` — the Mini's `.local` name
- **Home dir**: `/Users/jef/` (NOT `/Users/jefmini/` — that path was wrong in earlier memory and caused launchd plist EX_CONFIG failures)
- **Repo path**: `/Users/jef/jef-local-agents/`
- **Venv path**: `/Users/jef/jef-local-agents/venv/bin/python` (no leading dot — distinguishes from Air's `.venv`)

## Common transport

Air `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Jef/` ↔ Mini same path via iCloud. Fastest channel for ad-hoc file drops (smoke sources, debug stubs). For code: edit on Air → `git push` → `ssh mini 'cd ~/jef-local-agents && git pull --ff-only'`.

## scp / rsync / explicit user@host

Use `jef@jefmini` (NOT `jefmini@anything`). The SSH alias `mini` already supplies the user; for ssh-config-aware commands just `mini:~/path`.

## Related

[[two-machine-workflow]] [[mac-mini]] [[macbook-air]] [[warp-terminal]] [[icloud-sync]] [[obsidian-vault]] [[launchd]]
