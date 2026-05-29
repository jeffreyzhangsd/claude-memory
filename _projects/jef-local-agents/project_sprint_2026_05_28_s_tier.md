---
name: project-sprint-2026-05-28-s-tier
description: "S-tier infra sprint 2026-05-28 — 9 items shipped (cost tracker, secrets hooks, daemon-health, memory-sync script, CLAUDE.md drift, subscription tracker, cert watcher, wake-audio brief, plus today.* stable paths) + trading restructure (purchase_type split, monthly + per-trade caps both removed) + briefing Alpaca dedupe fix. Patterns worth carrying forward."
metadata:
  node_type: memory
  type: project
  originSessionId: 0221bf17-defc-479a-b13e-862600b7bfb8
---

# S-tier sprint — 2026-05-28

Single-session push: 9 high-ROI scripts shipped + trading restructure + briefing bugfix. All committed (push completed). User asked for "E2E + antagonist per script"; antagonist passes on S1/S2/S3/trading caught real bugs that would have shipped silent.

## Shipped

| ID    | What                          | Path / plist                                   | Schedule                                   |
| ----- | ----------------------------- | ---------------------------------------------- | ------------------------------------------ |
| S1    | LLM cost tracker              | `utils/llm_cost.py` + analyze.py instrument    | per-call                                   |
| S2/S3 | Pre-commit + pre-push secrets | `tools/secrets_scan.py` + `tools/git-hooks/`   | git hooks                                  |
| S4    | Daemon health watcher         | `daemon_health.py` + `com.local.daemon-health` | daily 10:00                                |
| S5    | Memory sync script            | `scripts/sync_memory.sh` + plist               | daily 09:15 (user must do `git init` once) |
| S6    | CLAUDE.md drift detector      | `claudemd_drift.py` + plist                    | Sat 10:00                                  |
| S7    | Subscription expiry tracker   | `finance/subscription_watcher.py`              | daily 09:30                                |
| S8    | SSL cert expiry monitor       | `finance/cert_watcher.py`                      | daily 09:45                                |
| S9    | Wake-up audio brief           | `wake_audio_brief.py` + plist                  | Mon-Fri 05:50                              |

Plus:

- Briefing.py Alpaca financial bucket dedupe fix (was hitting `n_predict=2000` cap on ~90 verification emails)
- Trading restructure: `purchase_type` field on positions, monthly TRADING_BUDGET cap removed, per-trade TRADE_AUTO_MAX also removed (auto-buy now spends model's full suggested_size)
- `trading/backfill_purchase_type.py` — one-shot tag of 87 existing positions (81 manual, 6 auto)
- `com.local.careermomentum.plist` bootstrapped (was in repo since 2026-05-20 but never loaded → silent missing daemon)
- Stable `today.{md,mp3,aiff}` mirror paths in `daily-brief/` for iOS Shortcut consumption (dated files become archive)
- iOS Shortcut runbook at `<vault>/jef-folder/wake-audio-shortcut-setup.md` — note iOS has NO native "Play Audio" action; need `Open File` + Quick Look, OR VLC URL-scheme, OR `Speak Text` directly from today.md

## Patterns worth carrying forward

### Per-script antagonist > batch antagonist

Subagent code review AFTER build (not before) caught real bugs in 3/3 reviewed items: S1 had reasoning-token double-counting + dated-model-name silent-zero, S2/S3 had OLDPWD bug + lockfile false-positives + core.hooksPath blind spot, trading-restructure had stale briefing prompt example + cost_basis-always-zero math. None of these would have surfaced in functional smoke tests — antagonist reading the code with adversarial intent is load-bearing.

**How to apply:** spawn antagonist subagent in background mode after each meaningful change, keep building in foreground; fix when it reports back.

### Python `logging` defaults to STDERR

Most of the daemon swarm uses `logging.basicConfig()` which writes to stderr. A watcher that only stats `StandardOutPath` mtime will report every logging-driven daemon as "stale" forever. **Always stat both stdout + stderr paths and take `max(mtime)`.** Caught by reinvestigation agent after I reported 10 "stale" daemons that had all actually run.

**How to apply:** any future log-mtime-based watcher must use both paths. See `daemon_health.py:check_one` for the canonical pattern.

### `plistlib` chokes on `<!-- ... -> ... -->` in XML comments

Several of the repo's `launchd/*.plist` files have block comments containing `->` arrows (e.g. "no Mini->Air SSH"). XML spec disallows `--` runs inside comments; `plistlib` rejects with "invalid token". Workaround: regex-extract `<key>X</key><string>Y</string>` pairs directly. See `daemon_health.py:_grep_plist`.

**How to apply:** for plist parsing in this repo, prefer regex extraction over `plistlib.load`. Or use `plutil -convert json` to launder the file first.

### Stable `today.*` paths for iCloud/iOS clients

iCloud-synced files with dated paths (`YYYY/MM/<date>.md`) break any client that hardcodes the path — the directory rotates monthly. Solution: write a stable `today.*` mirror **as a copy, not a symlink** (iCloud doesn't reliably sync symlinks to iOS). Dated files = archive; today.\* = the live target. See `wake_audio_brief.py` and `briefing.py:write_brief`.

### purchase_type backfill via filer field + acted_signals.json

For position-attribution after-the-fact: any vault `positions.json` entry with `filer="dashboard"` is manual; everything else is auto. For Alpaca positions WITHOUT a vault entry, intersect ticker with `acted_signals.json` keys (which track dashboard buy clicks + dismissals — dismissed alerts wouldn't have a position, so intersection ≈ manual buys). See `trading/backfill_purchase_type.py`.

## User-side TODOs remaining

- ~~`git init ~/.claude/memory` to enable memory sync~~ DONE 2026-05-28 (see Batch 5 — global repo + project memory via symlink, daemon bootstrapped + pushing).
- Populate `<vault>/finance/subscriptions.yaml` + `<vault>/finance/domains.yaml`
- iOS Shortcut for wake-audio (see runbook) — VLC URL scheme is the cleanest autoplay path
- Push pending commit (now done, branch synced)

## Batch 2 — 10 maintenance daemons (same day, Opus 4.8 resume)

Shipped commit `96d47b1`: auto_changelog, stale_project_archive, dep_vuln_scan, brew_outdated_check, todo_fixme_triage, citation_link_rot, dead_code_detector (no-Ollama) + monthly_best_of (gemma4:e4b), random_idea_gen (qwen3:14b), downloads_renamer (gemma4:e2b). All 10 ideas-backlog items marked shipped.

Two reusable gotchas the antagonist + smoke-test caught — **apply to every future daemon in this repo**:

1. **`load_dotenv()` MUST run before `from config import ...`** in any Discord/secret-using daemon. launchd inherits NO shell env; `config.py` reads `os.environ.get()` at import time, so without `load_dotenv()` first the webhook is `""` and pings silently never fire. Pattern: `from dotenv import load_dotenv; load_dotenv()` then `from config import ...`. (briefing.py was the only prior example.)
2. **SIGALRM watchdog handler must raise a `TimeoutError` subclass, not a bare `Exception`.** A `class _Timeout(Exception)` handler whose `main()` only does `except TimeoutError` lets the watchdog crash the daemon with an uncaught traceback instead of a clean `return 1`. Either make `_Timeout(TimeoutError)` or `except (TimeoutError, _Timeout)`. Pairs with [[feedback_bsd_flock_self_deadlock]].

downloads_renamer mutates `~/Downloads` — safe-by-construction (10-min settle, never overwrites via -2/-3 suffix, tightly-anchored generic-name regexes, content-hash dedup, reversible audit log at `agent/downloads-renamed.md`, slug-itself-generic rejection prevents rename loops). `vulture`+`pip-audit` added to requirements.txt.

## Batch 3 — 9 daemons via Workflow (T1/T3/T2), commit `fd16170`

Built by a **Workflow** (18 agents: author + adversarial-review-fix per daemon), then integrated serially. Shipped: daemon_anomaly_watch (22:00, "what broke today"), menubar_health (rumps KeepAlive), vault_hygiene (Sun 03:00, 4 passes), connect_ideas (Sat 13:00, embed+qwen rerank), phishing_triage (06:15, mail_router reuse + gemma4:e2b), api_doc_keeper (07:00, gemma4:e4b changelog diff), practice_potd (07:45, piano+vocal POTD), calendar_events (EventKit + surgical briefing.py edit), idle_work (every 30m, gated opportunistic). New deps: `rumps`, `pyobjc-framework-EventKit`.

New reusable gotcha (apply to all future watcher/report daemons):

3. **A scheduled report/watcher daemon must `return 0` even when it FINDS something.** daemon*anomaly_watch first did `return 1 if anomalies else 0` — that makes launchd log it as failed and trips daemon_health, i.e. the anomaly watcher flags \_itself* every time it does its job. The report + Discord ping ARE the signal; process exit code = process success, not "clean swarm". (KeepAlive probes like menubar_health's `--dry-run` 0/1 healthcheck semantic are fine — they never exit under launchd.)

Also: any daemon checking another daemon's OUTPUT must use the REAL path — briefing writes `daily-brief/YYYY/MM/<date>.md` (+ stable `today.md`), NOT `daily-brief/<date>.md`. Workflow agents flagged this couldn't be statically verified; caught at smoke-test. New daemons auto-registered in `daemon_health.INTERVAL_OVERRIDES` (always-on=0 for KeepAlive, 24\*8 for weekly) or they false-flag stale.

USER-SIDE TODO (batch 3): grant macOS Calendar access to `venv/bin/python3` (System Settings > Privacy > Calendars) — until then briefing's EventKit block is a clean no-op.

## Batch 4 — 10 daemon units via Workflow (natural-next + standalone), commits `6c26c0d`..`0fb9dd5`

Another Workflow (20 agents). Shipped + smoke-tested E2E, **separate commit per daemon** (per [[feedback_separate_commits_per_daemon]]): `refactor_scout` (qwen task ADDED to idle_work's queue — first Ollama-using idle task; rides the lock idle_work already holds, no nesting), `calendar_conflict` (06:45), `email_to_calendar` (08:15, draft-only — E2E pulled 111 real mails → 4 drafts), `pattern_detective` (Sun 13:00), `blind_spot_detector` (Sun 14:00), `graph_audit` (Sun 04:00, merges node-degree + graph-visualizer, reuses `wiki_graph_audit.py`), `trading/dividend_tracker` (Mon 08:15, yfinance), `reddit_digest` (Sat 14:30), `linkedin_draft` (Day1 11:00, drafts), `symptom_tracker` (Sun 15:30). No new pip deps.

Known limitation: **`reddit_digest` is inert** — reddit returns HTTP 403 to the Mini's IP even with a User-Agent. The plumbing (fetch→nomic-embed score→report) is correct; it writes an empty digest gracefully. To make it useful needs reddit OAuth (script app) or an unblocked RSS/proxy path. Don't re-debug as a code bug — it's an external block.

Pattern confirmed: idle_work is now extensible via its `TASKS` list and can host Ollama tasks (bounded by SIGALRM backstop inside the held lock). Good home for future "RAM-permitting" work (refactor scout was the canonical case). Total fleet ~40+ daemons after today's 3 batches (29 shipped 2026-05-28).

## Batch 5 — consumption funnel + dashboard + calendar + memory infra (same session)

After the 3 daemon batches, a cleanup/infra arc (all pushed, commits through `8ed04d0`):

- **Consumption funnel** — Discord = ACT-NOW only. 10 FYI daemons (connect_ideas, pattern_detective, blind_spot_detector, random_idea_gen, reddit_digest, monthly_best_of, linkedin_draft, practice_potd, vault_hygiene, graph_audit) no longer ping; they surface via the dashboard **reports** tab (`/api/reports` in `script_runner/app.py`) + a "DAEMON REPORTS (last 24h)" rollup in `briefing.py` (`recent_reports_block()`). Recorded in CLAUDE.md "Key technical decisions". Rule for new daemons: ping only if actionable-now.
- **Calendar TCC** — granted to the venv python via a one-shot launchd job; see [[calendar-tcc-launchd-grant]] for the responsible-process gotcha + the verify-via-launchd-kickstart trick.
- **Memory sync (no 2nd repo)** — global `~/.claude/memory` is the git repo (user set up); jef-local-agents PROJECT memory is included by symlinking `~/.claude/projects/-Users-jef-jef-local-agents/memory` → `~/.claude/memory/_projects/jef-local-agents/` (real files in the one repo). `sync_memory.sh` loops `SYNC_DIRS` (currently just global); daemon `com.local.memory-sync` bootstrapped + verified pushing under launchd (09:15 daily). Original project dir kept as `…/memory.bak-20260528` until trusted.
- **Memory prune** — merged 3 `scout_skip_*` → 1, deleted 2, graph_refresh (69 stubs / 227 backlinks). Saved [[feedback_separate_commits_per_daemon]].
- **Roblox memory removed** — Air-origin `-Users-jeffreyzhang-…-roblox-speed-math` dir removed from the Mini AFTER salvaging 9 uncaptured reusable lessons (test-UID gate, debug-bridge persistence risk, team-create-vs-Rojo, Toolbox audio, leaderstats/badges/group perks, iPad `isCompactTouch` tier, scrapped-handwriting lesson) into `<vault>/code/projects/roblox-speed-math-patterns-2026-05-13.md`. Dir parked at `/tmp/roblox-mem-removed-20260528`.

## Memory recall trigger

When sessions touch any of: daemon-health, secrets-scan hooks, LLM cost tracking, trading P&L attribution, today.\* iCloud pattern, "why did daemon_health flag X as stale", **launchd Discord daemon not pinging (→ load_dotenv before config import), a SIGALRM-watchdog daemon crashing instead of exiting cleanly (→ \_Timeout must subclass TimeoutError), the consumption funnel / dashboard reports tab, a launchd daemon needing a macOS TCC grant (→ [[calendar-tcc-launchd-grant]]), or memory-sync / project-memory symlink** — read this note first.
