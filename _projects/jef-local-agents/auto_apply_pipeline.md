---
name: auto-apply-pipeline
description: "Two-stage auto-apply (Phase 1 nightly packet + Phase 2 manual Playwright fill). LLM-as-resolver not LLM-as-driver. Air for browser, Mini for models. Submit stays manual."
metadata:
  node_type: memory
  type: project
  originSessionId: 84f0bff7-a0d1-4c1b-a2ba-b3ecc8ae973f
---

The career auto-apply system is split across two stages with a deliberate human gate at the **submit** step:

**Phase 1 (auto, nightly on Mini)** — `career/autoapply.py`. Auto-triggered from `careergrade` for yes-verdicts at confidence ≥ `CAREER_GRADE_CONFIDENCE_MIN`. Pure HTTP, no Ollama. Detects provider from URL (Greenhouse / Lever / Ashby; everything else silent-skipped), fetches the form schema, classifies fields, resolves boilerplate from `cv.md` + `_profile.md` Personal Facts, writes a markdown packet to `Career/AutoApply/{slug}-{date}.md` (no blockers) or `Career/QueuedForReview/{slug}-{date}.md` (custom Qs the classifier can't handle).

**Phase 2 (manual, on Air)** — `career/autofill.py`. `python -m career fill <slug>` opens headed Chromium on the Air, drives the form, calls gemma4:e4b on Mini via LAN to draft prose/option-picks for any blockers, leaves browser open for user to review and click Submit. `--batch` walks the queue; `--notify` Discord-pings the queue; `--verify` reads the DOM back and prints a PASS/FAIL table.

**Why the two-stage split:**
A broken auto-submit lands in front of a hiring manager — different blast radius than a webhook misfire. The packet (Phase 1) is portable to any device via iCloud and lets the user manually paste values into Simplify if they prefer. Phase 2 removes the typing friction but keeps the human at the "click Submit" gate.

**Why LLM-as-resolver, not LLM-as-driver:**
Hermes/gemma struggle with browser-automation tool loops without vision. Instead, Python deterministically resolves and types every boilerplate field (name/email/resume/EEO/work-auth via label patterns + Personal Facts), and the LLM only generates prose/picks for `COMPLEX` blockers (textareas like "Why this company?", multi_selects like "How did you hear?"). One batched Ollama call returns `===Q1===...===Qn===` sections; Python validates option picks against the known `answer_options` list and types them in. Reliable boilerplate, model only does what it's good at.

**Daemons / artifacts:**

- `careergrade` (Mini, 23:30) — runs autoapply after grading completes; one Ollama swap per night (Hermes → gemma4:e4b for interview-prep + autoapply) inside `agent_lock("careergrade")`
- `batchnotify` (Mini, 08:00, `com.local.batchnotify.plist`) — silent unless last 1 day produced yes-graded pending fills; posts to `DISCORD_CAREER_WEBHOOK`
- CV PDF resolution: tailored `Career/Graded/cv-{slug}-{date}.pdf` if present, else `Career/cv-generic.pdf` (user-provided fallback). No CV at all → autoapply silent-skips that job.

**Surprising gotchas (don't re-derive these):**

- **Ashby DOM quirks**: selects render as radio groups with name `<formInstanceId>_<fieldPath>` (suffix-match `[name$="_<fieldPath>"]`); yes/no booleans wrap a hidden `<input>` in a `[class*="_yesno_"]` div with two visible `<button>`s — click by EXACT text (`get_by_text("Yes", exact=True)`), substring `:has-text("Yes")` was a bug; file inputs match by `id` not `name`; location typeahead has no name/id, find by `placeholder*="Start typing"` then click the first `role="option"` to commit; date picker accepts MM/DD/YYYY typed input.
- **`config.py:_resolve_career_asset` prefers `~/Desktop/Projects.nosync/career-ops/` over the vault `.career-ops-mirror/`** if it exists (legacy Air-as-dev-box pattern). If career data looks stale on a machine, rename that local dir to force vault-mirror fallback. Bit us hard on Air during initial fill testing.
- **JD URL parse**: `Career/Inbox/jds/*.md` use Dataview double-colon `- **url**:: <url>`; `Inbox/{date}.md` uses single-colon `- **url:** <url>`. `prefill.py:parse_jd_header` handles both, but the break condition can't include `not line` or the loop quits at the blank line BEFORE the URL field.
- **Air username is `jeffreyzhang`, Mini is `jef`**. Same iCloud account, so `Path.home() / Library/Mobile Documents/...` works on both via Apple ID sync.
- **Air's venv has a broken `python` symlink** (points at 3.14, packages live in 3.12 site-packages). Invoke as `venv/bin/python3.12 -m career ...` not `python -m career ...` until that's fixed.

**Field-resolution priority order** in `autoapply.resolve_field`:

1. Hidden inputs → empty string (provider auto-populates server-side)
2. AUTO_DECLINE (demographic_questions) → match Personal Facts gender/race/veteran_status against options, else decline-to-self-identify
3. ACK (consent checkboxes) → first option
4. Fixed-name boilerplate per provider (Greenhouse `first_name`, Lever `urls[LinkedIn]`, Ashby `_systemfield_*`)
5. Label-regex fallback (work-auth, visa, restricted-country, LinkedIn/GitHub/portfolio, phone, school/grad/degree/pronouns/relocate/in-office/internships)
6. Unresolved → COMPLEX → LLM-drafted at Phase-2 fill time if textarea or select-with-options

**Related**:

- [[reference-mini-ollama-lan]] — how Air reaches Mini's Ollama over LAN
- [[no-cli-prefer-daemon-output]] — Phase 1 follows this; Phase 2 is a deliberate exception (CLI for the destructive submit action)
- [[load-bearing-rule-ordering]] — same pattern for LLM-drafted option picks
- [[runbooks/air-autofill-setup]] in vault — bootstrap commands for Air
