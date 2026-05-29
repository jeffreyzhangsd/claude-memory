---
name: feedback-dispatch-over-sync
description: "From this orchestrator session, push work via `/dispatch <owner> <brief>` — never write to `<vault>/agent/sync/<owner>/inbox.md`. Dispatch returns an envelope directly to the orchestrator's context; sync-channel is the pull-side fallback for workers, not the push path. Confirmed by user 2026-05-21 after I incorrectly used sync-channel writes for an AGT-024 follow-up before being redirected to `/dispatch`."
metadata:
  type: feedback
---

From this `jef-local-agents` (Mini) orchestrator session, push cross-session work via `/dispatch <owner> <brief>`. Do NOT write to `<vault>/agent/sync/<owner>/inbox.md`, `<vault>/agent/sync/<owner>/outbox.md`, or `<vault>/agent/sync/bugs.md` from here. If I notice I'm about to write to those files from this session, stop and use `/dispatch` instead.

**Why:** User explicitly redirected on 2026-05-21 after I filed AGT-024 via sync-channel. The point of [[orchestrator-dispatch]] (shipped 2026-05-21, IDEAS.md #21) is that the orchestrator's session receives results directly — no waiting on the user to open the worker session, no polling the inbox/outbox files. Sync-channel writes from main defeat that purpose. The `dispatch.md` slash command says it explicitly: "Don't write to `agent/sync/` from here — that's the pull channel, dispatch is push."

**How to apply:**

- Any "ask the wiki session to do X" / "ask the career session to do Y" from this Mini orchestrator session → `/dispatch wiki <brief>` or `/dispatch career <brief>`.
- Brief is the full instruction; include any context the spawned worker needs (it starts cold). Worker pre-seed pattern in [[reference-dispatch-worker-preseed]] handles the warmup-cost optimization.
- If the user asks me to "file a bug" or "tell wiki" or "let career know," default action is `/dispatch`, not a sync-channel inbox append.
- Sync-channel (`/sync-channel` skill) is still valid for the WORKER side — the spawned wiki/career session reads its inbox + others' outboxes at session start to surface ACTION REQUIRED items. That's the pull side and stays.
- Edge case: when a raiser-owner pair doesn't include the orchestrator (e.g. wiki finds an issue → wiki files against jef-local-agents), sync-channel still routes that. Dispatch is for orchestrator-initiated push only.
- `jef-local-agents-air` is not yet dispatch-spawnable (needs SSH path — v2 work). Until shipped, if the orchestrator needs Air work done, document it for the user to handle in the Air session rather than writing to `agent/sync/jef-local-agents-air/inbox.md`.

Related: [[reference-dispatch-worker-preseed]] (pre-seed memory pattern that makes spawned workers skip warmup exploration), [[project-cross-agent-workflow]] (the pull-side sync-channel lifecycle — still relevant for workers, not for the orchestrator).
