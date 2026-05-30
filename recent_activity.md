---
name: recent-activity
description: Last 7 days of work — what Jeffrey was building/learning/touching across repos + vault. Auto-updated nightly by skill_tracker.py.
metadata:
  type: project
---

# Recent Activity (auto-tracked)

Updated nightly at 05:00 by `skill_tracker.py` on jef-local-agents. Rolling 7-day window; older entries dropped on each run.

## 2026-05-30

**Built:** Progressed the dashboard with inline report summaries and per-card trading height adjustments, and implemented the personal CRM daemon with reconnect nudges and a birthday radar.

**Topics:**
- local-ai-agents-velocity
- personal-crm-daemon
- dashboard-layout-refinement
- trading-fill-reconciliation
- conversation-prep-briefing

**Artifacts:**
c52c8a7
baae502
17ddfe9
41d8255
f42cb8a
a968199
381e72d
10da377
c298c8b
7a85659
b885322
e546d2c
66c2a5a
04c947e
6f9c82d
d0236c3
77df824
1dd1c76
22c0bc2
f780c36
f16a928
0f66862
agent/anomalies/2026-05-29.md
code/refactor/jef-local-agents-2026-05-30.md
trading/daily/2026-05-29.md

## 2026-05-29

**Built:** Shipped significant updates to the local agent suite, including the weekly net-worth aggregator, monthly spending audit categorizer, and the YouTube URL to vault summary daemon.

**Topics:**
- daemon-architecture
- memory-synchronization
- calendar-event-kit
- pattern-detection-algorithms
- local-ai-agent-tooling

**Artifacts:**
1503ba3 docs(claude): add spending-audit + net-worth to the daemon table
2c5b0d1 feat(net-worth): weekly net-worth aggregator daemon
a2b4bb0 feat(spending-audit): monthly local statement categorizer daemon
9038e76 feat(memory-sync): sync multiple dirs (global + jef-local-agents project memory)
db15fe6 feat(briefing): roll up overnight daemon reports into the daily brief
753fa3e feat(linkedin_draft): monthly LinkedIn bullet drafter from activity
d37caae feat(blind_spot_detector): weekly vault coverage-gap finder
ea79908 feat(pattern_detective): weekly cross-domain pattern-transfer proposer
4239403 feat: stable today.{md,mp3,aiff} paths for iOS Shortcut consumption
code/refactor/jef-local-agents-2026-05-29.md
wiki/meta/decisions-2026-05-29.md

## 2026-05-28

**Built:** The trading daemon and dashboard received significant feature parity updates, including a market clock banner, improved alert handling, and enhanced manual trading controls.

**Topics:**
- trading-daemon-refactoring
- dashboard-ui-enhancements
- websocket-multiplayer-architecture
- local-agent-orchestration
- signal-processing-pipelines

**Artifacts:**
3b088b6 docs(IDEAS): mark #22 wiki-ask endpoint as shipped
88417d5 docs(IDEAS): lock take-profit decision — bump 20% -> 25% (option 1)
96f3082 docs(IDEAS): #24 Trading strategy v2 — predictive exits + manual re-invest
ec0b87d feat(trading/daemon): richer Discord daily summary
6461269 fix(trading): deadlock + watchdog + caches; 2h -> 1h scan cadence
f369372 feat(repo_scout): velocity mode + safety filters (bot + adversarial)
a461584 fix(trading/daemon): mark untradeable + execute-error signals processed
7f9499e fix(trading/daemon): mark alerts + skips as processed so they don't re-trip
55fda08 fix(dashboard/trading): auto-dismiss untradeable alerts
d5b0abd fix(dashboard/trading): actionable list sorted by score+confidence, not mtime
cdd7d95 feat(dashboard/trading): market clock banner (open/close countdown, PT)
834cdca feat(trading): manual buys scale to model's suggested size
a56e1c2 feat(dashboard/trading): alert clarity — explainer + conf pill + size hint
e20460a feat(trading): sort positions, trim signals, split manual vs auto budget, vary confidence
578d979 feat(dashboard/trading): position colors, column reorder, buy size dropdown, negative sign
81241fe feat(dashboard/trading): clean account card — hero equity + colored P&L
b994342 feat(dashboard/trading): clear alerts on buy + dismiss button
32509e9 fix(dashboard/trading): mobile width + buy button reliability
5264629 feat(dispatch): add 'trading' owner
6804707 feat(dashboard): trading polish — env load, alerts card, mobile, size variety
dfdabcf fix(trading): -v works after subcommand too
code/projects/trading-patterns-2026-05-27.md

## 2026-05-27

**Built:** The core trading agent functionality was significantly progressed, adding the 'Trading' tab to the dashboard, implementing multiple data sources (13F, Form4, Congress), and building out the orchestration layer with `daemon.py` and `execute.py`.

**Topics:**
- trading-dashboard-architecture
- financial-data-ingestion (13F, Form4, Congress)
- daemon-orchestration-patterns
- local-first-scoring-pipelines
- launchd-plist-management

**Artifacts:**
5f6fdbb feat(dashboard): trading tab — positions, signals, regime, budget, equity
10d7219 feat(trading): dashboard 'Trading' tab — 7 commands
d5bcc5b feat(trading): launchd plists for scan / monitor / daily
f9867ea feat(trading): __main__.py CLI
0bfeea5 feat(trading): go_live.py — interactive paper→real confirmation gate
e36c2e9 feat(trading): daemon.py — scan/monitor/daily orchestrators
a969149 feat(trading): missed_runs.py — surface daemon gaps
eb1d591 feat(trading): monitor.py — intraday sell-strategy + swing alerts
9a99966 feat(trading): portfolio.py — account + positions snapshot
64570d7 feat(trading): execute.py — Alpaca paper orders + budget + positions
90680b2 feat(trading): analyze.py — local-first scorer with DeepSeek escalation
8bb8a0f feat(trading): signals.py orchestrator — gate, dedup, price context
9f77f3e feat(trading/sources): congress fetcher — SSW Senate + Finnhub House
5a409f0 feat(trading/sources): 13F fetcher via edgartools (multi-CIK)
5f6b67b feat(trading/sources): form4 insider-buy fetcher via edgartools
45aab58 feat(trading): notify.py — Discord wrapper
570e14c feat(trading): add deps, config keys, vault dirs
65a4d53 feat(scripts): replay_missed_daemons — chronological re-fire after downtime
5887f84 feat(launchd): auto-spawn 'main' tmux session at login
5ab43a9 feat(launchd): tailscale watchdog plist — re-up if backend not Running
33986af fix(briefing): snap glance email count + promote skill_log to own section

## 2026-05-26

**Built:** Updated the `jef-local-agents` to include snap glance email count and promoted `skill_log` to its own section.

**Topics:**
- vault-organization-structure
- aim-training-benchmarking
- markdown-templating
- career-narrative-structuring

**Artifacts:**
33986af
Career/Inbox/2026-05-26.md
Career/Inbox/jds/2026-05-26-reddit-analytics-engineer.md
Career/Inbox/jds/2026-05-26-spotify-data-engineer-wrapped-fixed-term.md
Career/MaybeReview/2026-05-26.md
ado.md
agent/quality_samples/2026-05-26-vault_harness.md
wiki/aim-training/meta/index.md
wiki/aim-training/meta/log.md
wiki/aim-training/sources/2026-05-26-aim-training-aim-trainer-benchmarks-kovaak-s-aim.md
wiki/aim-training/sources/2026-05-26-aim-training-counter-strafing-stop-shooting-vs-running.md
wiki/aim-training/sources/2026-05-26-aim-training-how-to-identify-your-specific-weakness.md
wiki/home.md
wiki/meta/decisions-2026-05-26.md

## 2026-05-25

**Built:** Progressed the autofill agent with support for cross-provider verification, React-Select scraping, and improved robustness for various field types. Also implemented persistent SSH tunneling for the Mini $\rightarrow$ Air Obsidian REST connection.

**Topics:**
autofill-field-scraping
cross-provider-verification
react-select-scraping
ssh-tunneling-implementation
message-bus-architecture

**Artifacts:**
6a1227c
8d07bdb
eafce9d
423ea5d
0236a0c
19af394
0ce8873
64d84af
96d9bdd
47dfac9
b89702f
2a99ef8
cadba54
dea0293
9e95f63
f457afc
d0568a4
e644fc9
d4cc659
75b0b59
87ef842
agent/quality_samples/2026-05-25-vault_harness.md
jef-folder/temp/session-handoff-2026-05-25.md
wiki/meta/decisions-2026-05-25.md

## 2026-05-24

**Built:** The script_runner was updated with mobile layout fixes for career and run tabs, and new features were added to the career section, including mail triage, a Needs Review bucket, and batch autoapply functionality.

**Topics:**
- mobile-layout-fixes
- script-runner-refactoring
- career-workflow-enhancements
- vault-data-structuring
- dashboard-component-polish

**Artifacts:**
568a638
9f71578
6063572
921a4f8
adba9c7
48dd093
be92c9e
Career/Inbox/2026-05-14.md
agent/quality_samples/2026-05-24-vault_harness.md
notes/career-email-triage.md
wiki/golf/sources/2026-05-23-golf-iron-sets-blade-vs-cavity-back.md
