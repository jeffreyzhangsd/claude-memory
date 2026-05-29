---
name: feedback-deferred-not-demoted
description: 'When deferring backlog items, mark with status (⏸️) + explicit revisit trigger; keep them at their current priority slot. Do NOT move to "Deferred / out-of-scope" just because they''re paused.'
metadata:
  node_type: memory
  type: feedback
  originSessionId: e121a5e7-3bb2-4c9d-adad-7a3ed97b4c19
---

When deferring a backlog item, **keep it at its current priority slot** with a visible status marker (⏸️) and an explicit revisit trigger. Do NOT demote it to a "Deferred / out-of-scope" section.

**Why:** Stated explicitly 2026-05-19 after I'd nearly moved memory-consolidation and resilience-chain into the "Deferred / out-of-scope" tail of IDEAS.md. The user's exact wording: "defer those two but not [push them] down in the backlog". A paused item is still on the roadmap — moving it down implies it's been dropped, which is wrong. The marker + trigger pattern means future-you can spot the item and re-activate it when the trigger fires, instead of having to remember to check a forgotten section.

**How to apply:**

- Mark active-but-paused items inline with ⏸️ in the numbered list (or equivalent priority structure).
- Every deferred item gets a one-line **"Trigger to revisit:"** clause naming the concrete signal that would re-activate it (e.g. "first real rate-limit incident", "when the wiki feels cluttered"). No vague timelines like "later" or "Q3".
- Only push items into a separate "Deferred / out-of-scope" section if they've actually been dropped (superseded, judged unnecessary, or moved to another project). Pausing != dropping.
- Items that ship get a ✅ + commit hash, also inline, also at the same slot — same principle: history stays visible.

See [[project_coding_workflow]] for related backlog/work-tracking conventions in this repo.
