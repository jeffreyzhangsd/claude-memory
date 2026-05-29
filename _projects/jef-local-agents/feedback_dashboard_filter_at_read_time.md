---
name: feedback-dashboard-filter-at-read-time
description: "When a jef-dashboard card sources from a vault dir, filter by frontmatter status at read time — do not move files on every status flip."
metadata:
  node_type: memory
  type: feedback
  originSessionId: dcead836-0460-4571-885f-b8ba1440a015
---

# Dashboard cards: filter at read time, not via file moves

When a [[jef-dashboard]] card sources from a vault directory (e.g. `Career/QueuedForReview/`, `Career/Inbox/`, `wiki/<domain>/sources/`), filter the visible set by frontmatter `status:` at read time. Do **not** physically move files on every state flip.

**Why:** Every writer that flips status (career mail-triage, the `/grade` flow, dispatch updates, manual edits in Obsidian) would otherwise need a paired file-move hook. That's N writers × M directories of coupling. Filtering at read time keeps writers single-responsibility and lets the dashboard be the one place that knows "which entries count as live."

Precedent: `_list_qfr()` in `script_runner/app.py` filters out entries whose frontmatter status is in `{applied, responded, interview, offer, skip}`. Originally globbed every file in `QueuedForReview/` regardless of status, so entries flipped to `applied` (e.g. by `/dispatch career`) kept showing up until manually moved. Fixed 2026-05-23.

**How to apply:** For any new dashboard tab/card backed by a vault dir:

- Read all files in the dir.
- Filter by a named set of "resolved" statuses defined as a module-level constant.
- Don't add file-move logic to writers unless there's a separate retention/archive concern (e.g. `Career/Inbox/` cleanup at 21 days lives in `career/cleanup.py`, not in the writers).

Cross-link: [[topics/jef-dashboard]], [[project-career-pipeline]].
