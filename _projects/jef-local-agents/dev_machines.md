---
name: dev-machines
description: "Mac Mini = orchestrator + primary thing-fixer for jef-local-agents; MacBook Air = UI/autofill/Roblox work + sync-channel participant (jef-local-agents-air); both edit the repo but with distinct scopes. Persistent tmux on Mini."
metadata:
  node_type: memory
  type: project
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

## Role split (as of 2026-05-18)

- **Mac Mini (this owner: `jef-local-agents`)** — orchestrator + primary thing-fixer. All daemons, all the wiki/career/hobby/triage code paths, all Ollama + llama.cpp inference. When the user says "fix this" without scope context, default to Mini-side work.
- **MacBook Air (owner: `jef-local-agents-air`)** — UI / autofill-browser / Roblox / pre-existing Air projects. Reaches Mini's Ollama over LAN (`http://jefmini.local:11434`). Owns its own bugs in the sync channel — when raising a bug about autofill / browser / form-filling / UI, route to `jef-local-agents-air`, not Mini.

Both edit `~/jef-local-agents/` (different absolute paths — Mini at `/Users/jef/jef-local-agents/`, Air at `~/Desktop/Projects.nosync/jef-local-agents/`). GitHub is the merge point. Per [[project-github-for-cross-context-sync]] both sides push aggressively after every commit.

## Persistent tmux on Mini

A long-running Claude Code tmux session named `main` runs on the Mini 24/7. Users (phone or Air) rejoin it with:

```
ssh mini -t tmux attach -t main
```

Multiple clients can attach simultaneously and mirror the same view.

## Sync-channel participant detection

`/sync-channel` skill resolves owner from CWD + hostname:

- CWD ends in `/jef-local-agents` + hostname contains `air` → `jef-local-agents-air`
- CWD ends in `/jef-local-agents` + any other host (e.g. `jefmini`) → `jef-local-agents`
- CWD inside `<vault>/wiki/` → `wiki`; inside `<vault>/Career/` → `career`; vault root or other unowned subfolders → ask user

## Why this shape

Mini has the local inference stack co-located with the project so all triage/wiki/career/hobby/briefing/daemon work belongs there — that's the foundational reason it's also the orchestrator. Air doesn't run inference locally; its work is browser automation + UI flows that need a user's display anyway. The role split mirrors hardware capability.

## How to apply

- "Run this daemon / fix wiki_builder / debug a nightly run" → Mini.
- "Why didn't autofill find the form / Playwright selector issue / Roblox dev" → Air.
- When a path looks like a `/Users/jeffreyzhang/...` Air-era path (e.g. broken `notes` symlink), flag it — the Mini home is `/Users/jef`.

Related: [[project-cross-agent-workflow]] (inbox/outbox lifecycle), [[reference-mini-ollama-lan]] (LAN topology), [[project-github-for-cross-context-sync]] (repo sync).
