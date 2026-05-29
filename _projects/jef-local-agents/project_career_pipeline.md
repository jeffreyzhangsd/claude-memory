---
name: Career pipeline (jef-local-agents → vault → career-ops)
description: Hybrid local-AI scanner + Claude grader for job search. Vault canonical; career-ops stateless renderer. Daemon runs hourly on Mini via launchd.
type: project
originSessionId: 38e80489-d2eb-467a-807c-f4000568d7d4
---

# Career Pipeline

Local AI scans portals + pre-filters job postings. Survivors land in Obsidian vault Inbox. User runs `/grade` (custom slash command) in career-ops Claude session to grade w/ A-F + generate PDFs.

**Why:** User (Jeffrey, ~1.5y SWE) wanted to keep Claude budget on grading/cover-letter work, push scanning + filtering to local stack. Vault chosen as canonical state so career-ops can be wiped/updated without losing pipeline data.

## Two-machine setup

- **Air = dev**: `~/Desktop/Projects.nosync/jef-local-agents` (.venv), career-ops at `~/Desktop/Projects.nosync/career-ops`. No Ollama. Edit + commit + push only. Runs `/grade` Claude sessions for grading.
- **Mini = runtime**: `ssh mini` (alias). Username `jef`, hostname `jefmini`, home `/Users/jef/`. Repo at `/Users/jef/jef-local-agents` (venv, no dot). Ollama models live here. Daemon under launchd.
- **Vault** at `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Jef/Career/` is shared via iCloud — Mini writes, Air reads via Obsidian.
- **Mirror** at `Career/.career-ops-mirror/` — Air pushes static career-ops assets (portals.yml, cv.md, profile.yml, \_profile.md) to vault so Mini reads without needing the career-ops repo. Sync helper: `cd ~/Desktop/Projects.nosync/jef-local-agents && .venv/bin/python -m career.sync_mirror` (Air-side).

## Vault layout

- `Career/Inbox/{YYYY-MM-DD}.md` — daily files, entries grouped by `## ⏳ Pending Grade`, `## ✅ Graded`, `## 📤 Applied`, `## 💬 Active`, `## ❌ Closed`. Status flips happen in place.
- `Career/Inbox/jds/{date}-{slug}.md` — compressed JD bodies, linked from each entry's `**jd:**` field.
- `Career/MaybeReview/{date}.md` — soft rejects (DS/quant/ML/SRE/security/hardware). Hard rejects deleted, no log.
- `Career/Graded/` — copies of career-ops reports.
- `Career/CoverLetters/` — drafts (manual).
- `Career/state/scan-history.tsv`, `last-seen.json`, `cooldowns.json`, `dead-companies.json`, `discovered.json` — daemon state.
- `Career/Dashboard.md` — Dataview rollup. Requires Obsidian Dataview plugin enabled.
- `Career/README.md` — full workflow guide.

## Scanner / filter (jef-local-agents/career/)

- Modules: `scan.py` (Greenhouse + Lever + Ashby APIs w/ fallback chain), `extract.py` (gemma4:e4b JD field extraction + compression), `classify.py` (regex title classifier), `filter.py` (recency → title → years/edu/loc rule ladder), `handoff.py` (vault writers), `daemon.py` (main loop), `sync_mirror.py` (Air-side helper), `state.py` (vault path helpers + dataclasses).
- Entry: `python -m career` (or via launchd).
- Filters tuned for Junior/Mid SWE: max_age=14d, max_years_required=4, title_blocklist={Senior, Staff, Principal, Lead, Manager+}, hard_reject={non-SWE}, soft_reject={DS, quant, ML, SRE, security, hardware}.
- Survivor cap = 75 per run (overflow → scan-history).
- Scanner coverage: ~120 of 143 portals.yml companies covered. 23 need future Playwright fallback.
- ATS fallback chain: Greenhouse 404 → try Lever w/ slugify(name) → try Ashby. Recovers ~10 companies that 404 on Greenhouse direct (Notion/Plaid/Palantir/Hopper/Ro/Benchling/Lemonade/Nerdwallet/Synthego/Olo/Arcadia/etc).
- Dead-company auto-disable after 3 consecutive misses, 14-day TTL.

## launchd on Mini (CRITICAL — past mistake)

- Plist at `~/jef-local-agents/launchd/com.local.career-scan.plist`, installed at `~/Library/LaunchAgents/com.local.career-scan.plist`.
- Fires every hour at :05 PT.
- Daemon skips quiet hours 0–6 PT internally (no-ops at night).
- **Paths in the plist MUST use `/Users/jef/...` NOT `/Users/jefmini/...`**. The macOS user on the Mini is `jef`. Wrong paths → exit code 78 (EX_CONFIG) → silent failure. Past memory note about `/Users/jefmini/` being the home was incorrect; corrected in `reference_mac_mini_ssh.md`.

## /grade slash command (vault-aware grading)

- Lives at `career-ops/.claude/commands/grade.md` — custom Claude Code slash command.
- Survives `update-system.mjs apply` (`.claude/commands/` is NOT in career-ops `SYSTEM_PATHS`).
- Reads vault `Career/Inbox/*.md`, finds `status: pending-grade` entries across all daily files, follows the **Local Pre-filter Contract** in `modes/_profile.md`.
- Trusts local extraction (no JD re-fetch via Playwright unless missing), runs full A-F per `modes/oferta.md`, generates PDF per `modes/pdf.md`, writes report to `career-ops/reports/` + vault `Career/Graded/`, flips entry to `## ✅ Graded`.
- **Use `/grade`, NOT `/career-ops pipeline`** — the latter looks at `data/pipeline.md` which the daemon does not populate.

## Career-ops user-layer additions (persist through updates)

- `modes/_profile.md` has a "Local Pre-filter Contract (jef-local-agents → vault)" section describing pipeline source + grading writeback contract.
- `config/profile.yml` has `local_prefilter:` block w/ filter thresholds + vault paths.
- `.claude/commands/grade.md` for the /grade slash command.

## Status (2026-05-01)

- Daemon code complete, smoke-tested on Mini (522 listings → 25 survivors, 10 soft rejects, 487 rejected on first real run).
- launchd plist path bug fixed (was `/Users/jefmini/`, now `/Users/jef/`). Reinstalled + verified running clean — hourly cron is live.
- /grade command shipped on Air at `.claude/commands/grade.md`.
- Next: first `/grade` test on a small batch (3 entries) to verify the contract gets followed end-to-end (vault inbox → A-F report → PDF → status flip back into vault).

## Open follow-ups

- Playwright fallback for non-API portals (~23 companies, currently skipped).
- `/cover` slash command for cover letter generation (currently inline ask).
- Auto-fill apply forms via local Playwright (human-submit).
- Promote-from-MaybeReview CLI helper.
- Weekly career-ops mirror snapshot to `Career/state/career-ops-mirror-backup/` for full DR.
- Local cover letter draft pass (gemma 4B), Claude polish.

## Related

[[career-pipeline]] [[career-ops]] [[mac-mini]] [[obsidian-vault]] [[launchd]] [[two-machine-workflow]]
