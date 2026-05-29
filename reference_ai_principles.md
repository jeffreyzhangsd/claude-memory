---
name: reference-ai-principles
description: Load-bearing defaults Claude applies without asking — subagent, advisor/Opus, /dispatch, MCP, Ollama agent_lock, discover-fix-verify, pre-seed worker memory, jef-memory-prune, jef-prep, jef-capture, 10x autophagy, two-machine tagging [Air]/[Mini-SSH]. Triggers + Apply per rule. Covers Mini Ollama, daemon bugs, cleanup, cross-repo dispatch.
metadata:
  type: reference
---

# AI principles — load-bearing defaults

User crystallized these 2026-05-22 as the patterns applied across almost every project. **Session defaults. Apply without asking. Each rule has Trigger (when to act) + Apply (what to do).**

---

## Pre-flight checklist — scan this first

| Signal                                                                                | Apply                                                              | Rule |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ---- |
| Task is parallel OR touches >1 area OR needs >2 file searches to scope                | Dispatch subagents (1+ focused, never monolithic)                  | 1    |
| Sub-question needs reading >3 files OR decision has >1 viable architecture            | Spawn Opus-tier subagent for the hard subset only                  | 2    |
| Work belongs in a different repo/vault than CWD                                       | `/dispatch <owner> "<brief>"`                                      | 3    |
| Need to drive external tool with no shell API (Roblox Studio, Notion, Drive, IDE)     | Look for MCP server first                                          | 4    |
| About to call Ollama >1× in a chain on the Mini                                       | Wrap in `agent_lock()` flock                                       | 5    |
| Bug surfaces in **daemon-produced** content (wiki / classifier / hobby research)      | Three-step loop: discover-dispatch → fix-in-main → verify-dispatch | 6    |
| Adding a new spawnable dispatch owner                                                 | Pre-seed `~/.claude/projects/<derived>/memory/` first              | 7    |
| `MEMORY.md` > 30 lines OR just ran `/jef-capture` OR memory contradicts current state | `/jef-memory-prune`                                                | 8    |
| Project has prior commits OR unfamiliar stack — NOT a one-file edit                   | `/jef-prep "<goal>"`, read `jef-context.md` before first edit      | 9    |
| Session made >2 design decisions OR shipped feature/daemon                            | `/jef-capture` (NEVER hand-write in `code/projects/`)              | 10   |
| Estimating cleanup effort OR user asks "what's next?" in idle moment                  | Multiply gut estimate ×10; offer maintenance proactively           | 11   |
| Showing shell commands that touch both machines                                       | Tag every block `[Air]` / `[Mini-SSH]` / `[either]`                | 12   |

---

## 1. Subagent-driven dev

- **Trigger:** Task is parallel (independent units) OR touches >1 area of the codebase OR requires >2 file searches to scope.
- **Apply:** Dispatch via the Agent tool. Multiple focused subagents beat one monolithic one. Brief must be self-contained — subagents can't ask clarifying questions. Review their output before integrating.
- See [[subagent-driven-dev]], `feedback_subagent_execution.md`.

## 2. Stronger-model subagent for hard subsets

- **Trigger:** Inside the current task, a sub-question requires reading >3 files to answer, OR a decision has >1 viable architecture, OR you're stuck after one attempt at the same level. Plain implementation / refactor / codegen does NOT qualify.
- **Apply:** Spawn an Opus-tier subagent (`model: "opus"` on the Agent tool) for that sub-question only. Default the rest to the standard tier. Don't escalate out of habit.
- See `feedback_advisor_pattern.md`.

## 3. Orchestrator + dispatch

- **Trigger:** Work belongs in a different repo / vault / session than the current CWD (e.g. main in `jef-local-agents` needs the `wiki` worker to audit vault content; or `career` worker to triage postings).
- **Apply:** `/dispatch <owner> "<brief>"` from main. Owner spawns a fresh worker, runs the brief, returns envelope to main. Default budget `$2`, runtime `5min` — override with `--budget` / `--timeout` for known-long jobs. Workers do their work; main integrates results.
- Owners: `wiki`, `career`, `jef-local-agents`, `jef-local-agents-air`. See [[orchestrator-dispatch]].

## 4. MCP agents (for external tool surfaces)

- **Trigger:** Need to inspect / drive a tool that can't be hit via shell + HTTP (Roblox Studio, Notion, Drive, IDE state).
- **Apply:** Look for an MCP server first. Connect via the host's MCP config. Don't write bespoke automation when an MCP bridge exists.
- See [[mcp]], `code/projects/roblox-tycoon-and-3d-agents-research-2026-05-20`.

## 5. Agentic batching (Ollama lock)

- **Trigger:** About to call Ollama (gemma4 / hermes / qwen3 / nomic-embed) more than once in quick succession on the Mini, especially across model families.
- **Apply:** Wrap the chain in `agent_lock()` from `agent_lock.py` (one flock on `/tmp/jef-local-agents-ollama.lock`). `OLLAMA_MAX_LOADED_MODELS=1` means concurrent callers thrash; sequencing under one lock prevents it. Bypass with `AGENT_LOCK_DISABLE=1` only for manual one-shots.
- See [[agentic-batching]].

## 6. Discover → fix → verify loop

- **Trigger:** A bug surfaces in **daemon-produced content** (wiki chapter wrong, classifier misroute, hobby research hallucinated).
- **Apply:** Three steps, in order:
  1. `/dispatch wiki "audit/scan for X"` — read-only discovery. Wiki worker returns findings.
  2. Fix the daemon code in `jef-local-agents` main session (root cause, not the symptom).
  3. `/dispatch wiki "verify X is gone"` — close the loop.
- Wiki worker NEVER fixes daemon code (wrong cwd, wrong scope). See `feedback_dispatch_discover_fix_verify_loop.md`.

## 7. Pre-seed worker memory (new dispatch owner)

- **Trigger:** Adding a new spawnable dispatch owner.
- **Apply:** Pre-populate `~/.claude/projects/<derived-path>/memory/` with the worker's pattern notes, MEMORY.md index, project state. Otherwise the first dispatch spawn has zero context and burns budget on warmup.
- See `reference_dispatch_worker_preseed.md`.

## 8. Memory pruning

- **Trigger:** After `/jef-capture` (it auto-prunes); when any `MEMORY.md` index exceeds 30 lines; when a memory contradicts current state.
- **Apply:** `/jef-memory-prune` — audits + drops dupes / stale time-series / restated CLAUDE.md content across global + per-project dirs. Backs up to `/tmp` first.
- See [[jef-memory-prune]], [[claude-memory]].

## 9. Start projects with `/jef-prep`

- **Trigger:** Project has prior commits OR uses an unfamiliar stack. **Skip** for one-file edits, single-function fixes, or trivial reads.
- **Apply:** `/jef-prep "<one-line goal>"` from the project CWD. Reads git state + `<vault>/code/projects/*.md` pattern notes + MIND graph, writes `jef-context.md`. Read that file before the first edit — it surfaces prior gotchas, file paths, open questions.
- See [[jef-prep]].

## 10. End projects with `/jef-capture`

- **Trigger:** Session made >2 design decisions OR shipped a new feature/daemon OR resolved a non-trivial bug. **Skip** for routine refactors, one-file edits, or sessions where no patterns were learned.
- **Apply:** `/jef-capture`. Extracts reusable patterns, design choices, gotchas. Writes verbatim (no LLM rewrite) to `<vault>/code/projects/<project>-patterns-<date>.md`. User does NOT hand-write in `code/projects/` — this slash command is the only entry point.
- See [[jef-capture]].

## 11. 10x autophagy rule

- **Trigger:** Estimating effort on a feature; user asks "what should we do next?" in an idle moment; reviewing accumulated state (memory files, daemon code, vault chapters); planning the next sprint of work.
- **Apply:** Multiply your gut estimate of cleanup work by 10×. Cleanup compounds — every stale memo, dead daemon, orphan note taxes future sessions. Offer maintenance work proactively even when no one asks. Quote: _"do about 10x more codebase autophagy / compression / simplification / systematic open-ended reviews / canonicalization, and adversarial testing than you probably want to do."_
- See `feedback_10x_autophagy.md`.

## 12. Two-machine tagging

- **Trigger:** Showing any shell command in a session that touches both machines, OR a project whose runtime is on the Mini but dev happens on the Air.
- **Apply:** Tag every code block: `[Air]` (dev box, `~/Projects.nosync/...`), `[Mini-SSH]` (`ssh mini`, headless runtime, `~/jef-local-agents`), `[either]` (identical on both). Group steps by machine. Call out crossings ("rsync to Mini, then SSH in").
- See `feedback_two_machine_workflow.md`.
