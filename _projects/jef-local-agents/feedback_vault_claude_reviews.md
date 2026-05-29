---
name: feedback-vault-claude-reviews
description: "When vault Claude pushes back on a jef-local-agents fix summary, treat it as a real signal — they have independent context and catch things I gloss over"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 698a2418-1265-436c-9955-ad567d520623
---

When the vault Claude session reviews work shipped from this session (via `<vault>/wiki/meta/pipeline-responses.md` ⇄ `<vault>/wiki/meta/bugs.md`), their pushback is reliably load-bearing — don't dismiss it just because my fix summary already looks tidy.

**Why:** confirmed 2026-05-17 on three distinct items in one round.

1. My status table marked AGT-002 (cross-source contamination) as "✅ Fixed — synth rule 11 SOURCE ISOLATION." Vault Claude correctly pushed: that's a _prompt-level_ fix on the drafter/full-merger path; the drafter still passes all sources to one LLM call. Needed a structural refactor (`_draft_new_chapter()` — source 1 → drafter alone, sources 2..N → `_hybrid_merge` against the drafted chapter). Shipped as commit `1f3415c`.
2. Cleanup glob I proposed (`*_[0-9]*.md`) would have matched 149 files in `agent/done/` including jef-capture pattern notes — vault Claude flagged it before damage. Right pattern is the date-anchored `20[0-9][0-9]-[0-9][0-9]-[0-9][0-9]-*_[0-9]*.md` (~13 files).
3. My AGT-014 acknowledgment listed one orphan because the bug log only listed one; vault Claude noticed there were actually two (`Bodyweight-Fitness-Progressions-Version-5.4.md` was missing). They had ground-truth from a full `ls agent/done/` that I'd skipped.

**How to apply:** when a `pipeline-responses.md` reply or escalation lands here from vault Claude, slow down before responding. Re-verify the specific claims they're questioning (especially "auto-mitigated", "downstream of", or any phrasing that hand-waves a chain). Run their suggested glob/check against the live filesystem before accepting or rejecting. Their independent read catches what my own optimism-bias glosses over.

Related: [[project-coding-workflow]], [[no-cli-prefer-daemon-output]] — the broader pattern is that durable artifacts (vault notes, daemon outputs) beat in-session memory because they survive cross-session and force re-verification.
