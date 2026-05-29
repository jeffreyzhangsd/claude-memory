---
name: project-cross-agent-workflow
description: "The bug → fix → verify lifecycle running through agent/sync/<owner>/{inbox,outbox}.md across wiki, career, jef-local-agents (Mini), jef-local-agents-air (Air) sessions. Validated end-to-end 2026-05-17→18 on 17 bugs; channel relocated wiki/meta/ → agent/sync/ 2026-05-18."
metadata:
  node_type: memory
  type: project
  originSessionId: 698a2418-1265-436c-9955-ad567d520623
---

The cross-agent channel lives at `<vault>/agent/sync/` (schema in `README.md`, skill `/sync-channel`). Each owner has their own `<owner>/inbox.md` (bugs assigned TO them) + `<owner>/outbox.md` (fixes shipped BY them); agents read their inbox + others' outboxes, never the global `bugs.md` (human reference only). Used end-to-end on 17 bugs over 2 days; relocated from `wiki/meta/` (old single-file responses-from-\*.md layout) to per-owner inbox/outbox on 2026-05-18. Workflow below is the durable pattern; future sessions should follow it.

**Why:** validated 2026-05-18 — every step caught something. Skipping any step risks regressions or data damage.

**How to apply (full lifecycle, in order):**

1. **Open session → `/sync-channel`.** ALWAYS. Don't trust prior in-session state; the file is the truth. Other agents may have moved things while you weren't looking.

2. **Status transitions follow authority rules** (per SCHEMA.md):
   - Owner moves `open → in-progress → pending-verification`.
   - **Only the RAISER moves `pending-verification → resolved`.** Never resolve your own fix — you don't get to be the verifier of work you did.
   - Either side can mark `archived` after 30d.

3. **Code-shipped ≠ verified.** When you ship a fix, status is `pending-verification`, not `resolved`. The raiser confirms via real-world evidence — next nightly run, dry-run, or independent audit against live data. Smoke tests on YOUR side are necessary but not sufficient.

4. **The raiser runs their own audit** instead of trusting the owner's report. On 2026-05-18 vault Claude re-audited my fixes after the nightly and caught 4 regressions + 1 new critical bug I'd introduced. If they'd taken my "all resolved" claim at face value, the daemon would have kept damaging data.

5. **Data corruption requires a cleanup step BEFORE re-enable.** Code-only fixes are insufficient when prior runs have poisoned downstream state (chapters with wrong frontmatter, duplicate files, etc.). The pattern:
   1. Daemon disable (`launchctl unload …`) to stop ongoing damage.
   2. Ship the code fix.
   3. Audit the data state for residual poison.
   4. Clean (manually or via a one-time migration).
   5. Re-verify the clean state with the audit function used in step 3.
   6. Re-enable daemon (`launchctl load …`).
   7. Watch the next nightly's log for expected signals (the audit functions produce them).

6. **Responses are durable and audit-traceable.** Write the full reasoning in your own `agent/sync/<me>/outbox.md`, not just chat. Other sessions open days later and read the file; no in-session memory survives. Include: bug IDs touched, commit SHA, what's verified vs partial, follow-up requests, cleanup commands.

7. **Inbox status + outbox entry + bugs.md row all move together when shipping a fix.** Don't write an outbox entry without (a) flipping the affected entry's `[<status>]` marker in your own inbox.md to `[pending-verification]`, (b) copying the entry to the raiser's inbox so they see the verify queue, and (c) updating the row in global `agent/sync/bugs.md`. All three or none — partial updates leave `/sync-channel` showing inconsistent state.

8. **When the raiser flags regressions, treat as a SEPARATE shipping cycle** — don't try to slip the re-fix into the same response thread. New status update entry, fresh commit SHA, clear re-fix description. The audit trail matters.

9. **Surface quality-of-life observations even when not filing as bugs.** Vault Claude's 6 QoL items at the end of their 2026-05-18 response (formatting splits, log readability, defensive parsing) were valuable signals about the broader system. Triage them but don't dismiss.

10. **Daemon-disable is a normal-operations tool, not panic mode.** When tonight's run would worsen things faster than you can fix, disable it. Cheap and reversible (`launchctl unload`/`load`). Saves real damage.

Verified working end-to-end on AGT-001 through AGT-017 across 4 commits (5847b5b, 1f3415c, cd1e0b7, 4173bc4). 9 resolved, 5 pending-verification awaiting next nightly, 1 in-progress (defensive-only), 1 open (cosmetic, human-owned).

Related: [[feedback-vault-claude-reviews]] (trust their pushback), [[project-github-for-cross-context-sync]] (push aggressively so other instances see truth), [[feedback-settings-local-no-reload]] (permission reload gotcha).
