---
name: reference-dispatch-worker-preseed
description: How to set up per-owner pre-seed memory for a new dispatch worker — derived path, files, content shape
metadata:
  type: reference
---

# Dispatch worker pre-seed pattern

When adding a new spawnable owner to the [[orchestrator-dispatch]] system, pre-seed its per-project memory dir so the spawned Claude worker session skips warmup exploration and goes straight to the envelope.

**Why:** Each `claude -p` spawn would otherwise re-explore "what is this place / what's my job" before getting useful. Pre-seed cuts ~$0.10-0.30 + 20-40s off every dispatch and gives each owner a defined personality.

**How to apply:** When the user adds a new owner (currently wiki, career, jef-local-agents-air [v2], maybe minillm-dev), repeat the recipe below.

## Where the pre-seed lives

Claude Code derives a project memory dir from the spawn CWD:

```
~/.claude/projects/<sanitized-cwd>/memory/
```

The sanitization replaces `/` → `-`. Example: `/Users/jef/Library/Mobile Documents/iCloud~md~obsidian/Documents/Jef/wiki` → `-Users-jef-Library-Mobile-Documents-iCloud-md-obsidian-Documents-Jef-wiki`.

Verify the exact dir name by:

```bash
ls ~/.claude/projects/ | grep <owner-keyword>
```

If the dir doesn't exist yet, `mkdir -p .../<derived>/memory/` first.

## Files to write

Each pre-seed has the same 4-file shape:

1. **`MEMORY.md`** — index (≤30 lines, format `- [Title](file.md) — hook`). Loaded into every session opened in that CWD.
2. **`worker_role.md`** — shared content describing dispatch worker mode (DISPATCH_TASK_ID env detection, follow WORKER.md, no questions, always call `dispatch complete`).
3. **`<owner>_conventions_terse.md`** — owner-specific dense bullets. Page types, format gotchas, do-not-touch paths.
4. **`dispatch_complete_cheatsheet.md`** — shared content: exact bash invocation of `dispatch complete` with success / partial / error / artifact forms.

`worker_role.md` and `dispatch_complete_cheatsheet.md` are essentially the same across owners (modulo `[[wiki-conventions-terse]]` vs `[[career-conventions-terse]]` cross-links). Duplicated copies per owner are fine — they're tiny and isolation is cleaner than symlinking.

## Where to source content from

- **Worker role** — the `<vault>/agent/dispatch/WORKER.md` contract is the canonical version; the memory file is a terser pointer.
- **Owner conventions** — distill from the owner's `CLAUDE.md` file(s):
  - wiki: `<vault>/CLAUDE.md` (schema) + `<vault>/.claude/CLAUDE.md` (session scope).
  - career: `<vault>/.claude/CLAUDE.md` + `<vault>/Career/README.md`.
  - jef-local-agents: `~/jef-local-agents/.claude/CLAUDE.md`.
- **Dispatch complete cheatsheet** — copy from any existing owner's, swap example `--artifact` paths to match the new owner's likely targets.

## Existing pre-seeds

- `~/.claude/projects/-Users-jef-Library-Mobile-Documents-iCloud-md-obsidian-Documents-Jef-wiki/memory/` — wiki worker (2026-05-21).
- `~/.claude/projects/-Users-jef-Library-Mobile-Documents-iCloud-md-obsidian-Documents-Jef-Career/memory/` — career worker (2026-05-21).

## Related

- Dispatch protocol: `<vault>/agent/dispatch/SCHEMA.md`
- Spawned-Claude contract: `<vault>/agent/dispatch/WORKER.md`
- Source: `~/jef-local-agents/dispatch/`
- Slash command: `~/jef-local-agents/.claude/commands/dispatch.md`
- IDEAS.md item #21 (Orchestrator dispatch) in `~/jef-local-agents/IDEAS.md`
- [[orchestrator-dispatch]] · [[main]]
