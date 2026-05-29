---
name: haiku-subagents-vs-dispatch
description: "Pick rule for vault/career work — Haiku subagent (Agent tool) vs /dispatch worker session, and when to default Sonnet vs Haiku for each"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 944135c1-ac8e-4129-ba73-e4850a45c6a1
---

Main Mini session stays on Opus. Delegate cheaper work via two mechanisms — and pick between them by task shape, not by gut.

**Why:** User is cost-conscious ([[user-cost-constraint]]) but rejected per-project Sonnet override on 2026-05-25 because manual `/model opus` mid-session is friction. Inverse strategy: Opus orchestrator + auto-delegated weaker workers. Pattern decided 2026-05-25 after attempting and reverting `.claude/settings.json` model overrides in vault/wiki + vault/Career + jef-local-agents.

**How to apply — pick rule:**

| Use **Agent tool** subagent if         | Use **/dispatch** worker if   |
| -------------------------------------- | ----------------------------- |
| Answer fits in a return message        | Task produces files / commits |
| Want context preserved in main session | Worker can be fire-and-forget |
| Sub-2-min lookup                       | Could take >2 min             |
| In-session continuation expected       | Independent workspace OK      |

**Available Haiku subagent profiles** (`~/.claude/agents/`, written 2026-05-25):

- `vault-note-finder` — grep vault, ranked file list + 1-line snippet. For "where is the note about X".
- `career-inbox-sweep` — read-only triage of `Career/Inbox/`, status tallies + recent entries. For "what's open / how many graded A / find Stripe entry".
- `wiki-link-check` — extract all `[[...]]` from a file/dir, report broken vs resolved. For post-edit audits.

**Dispatch model flag** (`/dispatch <owner> <brief> --model haiku|sonnet|opus`):

- `--model haiku` — shallow grep/list/count tasks against vault (~$0.10-0.30)
- default Sonnet — content lints, frontmatter sweeps, moderate edits (~$0.30-1.50)
- `--model opus` — only architectural / multi-step refactors (rare in vault context)

**Don't:**

- Don't add per-project `model` overrides in `.claude/settings.json` for vault subdirs — tried 2026-05-25, reverted same day. They downgrade direct sessions too, not just dispatch.
- Don't use a Sonnet/Opus subagent for tasks the Haiku profile is built for — defeats the point.
- Don't /dispatch a task the subagent could finish in <30s in-session.

## Related

- [[feedback-advisor-pattern]]
- [[feedback-dispatch-over-sync]]
- [[user-cost-constraint]]
