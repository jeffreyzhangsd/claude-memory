# Memory Index — jef-local-agents

## Project state (current)

- [Monetization: active pursuit](monetization_active_pursuit.md) — shipping dev Notion templates for local-AI-builders (easy → hard: Model Library → Agent Inventory → Prompt Library → Builder Pipeline → Reading Tracker). Notion Gallery + Gumroad. Started 2026-05-19.
- [wiki_builder v1.0.0 shipped](project_wiki_builder_state.md) — v1 in prod on Mini since 2026-05-01, nightly 02:30. Optimization history: fuzzy section matcher, JSON-schema constrained extractor, embedding pre-filter. Migrated from Air memory 2026-05-21.
- [Career pipeline (jef-local-agents → vault → career-ops)](project_career_pipeline.md) — hybrid local-AI scanner + Claude grader. Vault canonical; career-ops stateless renderer. Daemon hourly on Mini. Migrated from Air memory 2026-05-21.
- [Dev machines](dev_machines.md) — Mini = orchestrator + primary thing-fixer (`jef-local-agents`); Air = UI/autofill/Roblox (`jef-local-agents-air`); both edit the repo, distinct scopes. Persistent tmux `main` on Mini, reconnect via `ssh mini -t tmux attach -t main`
- [Mini services Air via SSH](mini_services_air_via_ssh.md) — Mini orchestrator can ship Air-owned sync entries directly (`ssh air` git pull / plist install / Tailscale health probes) instead of waiting on an Air Claude session. Shipped 2026-05-24 (AGT-027).
- [Coding workflow — jef CLI + slash commands + hooks](project_coding_workflow.md) — `/jef-prep` before, hooks fire during, `/jef-capture` writes verbatim to `<vault>/code/projects/`; 5 subcommands as of 2026-05-15
- [Auto-apply pipeline (Phase 1 packet + Phase 2 Playwright fill)](auto_apply_pipeline.md) — autoapply auto-triggered nightly on Mini, autofill manual on Air over LAN to Mini's Ollama, submit stays manual. Greenhouse/Lever/Ashby provider quirks recorded.
- [GitHub repo as cross-Claude-instance context sync](project_github_for_cross_context_sync.md) — push aggressively after every commit; private repo doubles as the way future sessions on any device get current truth.
- [Cross-agent bug/fix workflow](project_cross_agent_workflow.md) — `agent/sync/<owner>/{inbox,outbox}.md` lifecycle: owner ships → pending-verification (entry copied to raiser's inbox) → ONLY raiser resolves; data cleanup precedes re-enable; daemon-disable is a normal tool. Channel relocated wiki/meta/ → agent/sync/ on 2026-05-18.
- [Dispatch over sync from orchestrator](feedback_dispatch_over_sync.md) — From this Mini session, push work via `/dispatch <owner> <brief>`; never write to `agent/sync/` files from here. Sync is the worker-side pull channel, dispatch is the orchestrator-side push channel. Shipped 2026-05-21 (IDEAS.md #21).
- [Trading module (Form 4 + 13F + congress → hermes → Alpaca paper)](project_trading.md) — Shipped 2026-05-27. Local-first scorer (hermes3:8b), DeepSeek escalation (now deepseek-v4-pro — reasoner retired, fixed 2026-05-29), ≤$50 auto-buy + $500/mo budget, 3 launchd daemons, dashboard tab + per-row paper-buy at `/api/trading/place-buy`. Owner added to dispatch. **2026-05-28: BOTH caps (monthly + per-trade) removed; auto vs manual P&L split via purchase_type field.**
- [S-tier sprint 2026-05-28 (9 infra scripts shipped)](project_sprint_2026_05_28_s_tier.md) — cost tracker / secrets hooks / daemon-health / memory-sync / drift detector / subscription + cert watchers / wake-audio brief. Patterns: per-script antagonist load-bearing, Python logging defaults stderr, plistlib trips on `->`, today.\* iCloud copy-mirror pattern.
- [Plan: Claude Pro + DeepSeek v4-pro orchestrator](plan_claude_pro_deepseek_orchestrator.md) — LIVE on Pro (confirmed via claude.ai/usage 2026-05-29); `DEEPSEEK_API_KEY` in `.env`. Claude orchestrates, DeepSeek v4-pro handles heavy reasoning via API.
- [DeepSeek v4-pro antagonist pipeline](deepseek_v4_antagonist_pipeline.md) — ship-daemon Phase-2 antagonist via `utils/deepseek_review.py` (v4-pro reviews, Claude triages). v4 model retirement (reasoner/chat dead) + the reasoner `max_tokens` gotcha (caps reasoning+content → tight cap = empty output, use 32k). Shipped 2026-05-29.

## Reference

- [Mini Ollama LAN exposure](reference_mini_ollama_lan.md) — Mini Ollama on `*:11434`. Bonjour off → use Tailscale IP `100.110.92.114:11434` from Air (jefmini.local doesn't resolve). Menubar toggle overrides env var.
- [Trading data sources](reference_trading_data_sources.md) — edgartools needs set_identity; SSW JSON path on GitHub raw; HSW S3 endpoint dead; Alpaca SDK is alpaca-py not alpaca-trade-api.

## Patterns & A/B findings

- [Agentic batching](agentic_batching.md) — batch Hermes/agent calls per run; serialize pipelines that share Ollama (OLLAMA_MAX_LOADED_MODELS=1)
- [Local-model A/B testing](hermes_ab_testing.md) — A/B every new local-model pipeline; Hermes wins grading/orderer, loses prose; qwen3:14b wins code review (hermes hallucinates false positives)
- [Load-bearing rule ordering](load_bearing_rule_ordering.md) — local models drop the LAST rule when others dominate; restate most-violated rule as rule #1, add deterministic post-processing safeguard
- [Anti-hallucination via web grounding](anti_hallucination_via_web_grounding.md) — gpt-researcher recipe: DDG snippets → numbered context → strict cite-by-n prompt → programmatic `## Sources` block
- [pre_expanded: true escape hatch](pre_expanded_escape_hatch.md) — vault_harness skips gemma expansion step when this frontmatter flag is truthy; for daemons that already produce structured wiki notes
- [No-CLI, prefer daemon output](no_cli_prefer_daemon_output.md) — default automation shape is nightly daemon → Obsidian browseable artifact, NOT interactive CLI the user has to invoke

## Feedback (project-specific gotchas)

- [venv Python 3.12 → 3.14 drift](feedback_venv_python_version_drift.md) — `venv/bin/python3` = 3.14 but most requirements.txt packages live in 3.12 site-packages; reinstall into 3.14 on `ModuleNotFoundError`
- [Separate commits per daemon](feedback_separate_commits_per_daemon.md) — when shipping a multi-daemon batch, one commit per logical daemon/group, not a mega-commit. Said 2026-05-28 after two 9–10-file mega-commits.
- [Monetization: no-persona preference](monetization_no_persona_preference.md) — for side-hustle/income work, default to platform-marketplace products + capital plays. Skip newsletter/Discord/Substack/audience-build pitches.
- [Voice for user-facing copy](voice_for_user_facing_copy.md) — match user's casual voice when writing product copy under their brands. Lowercase-leaning, "tbh"/"honestly"/"kinda", self-deprecating, no marketing-speak. Brand-facing only — internal artifacts stay normal.
- [Wiki goal-oriented relevance filter](wiki_goal_oriented_relevance_filter.md) — wiki's organizing principle: filter world info by "what serves this user's this goal right now," not topic comprehensiveness. Drives all wiki_builder design choices.
- [Claude Code hooks over SSH](claude_code_hooks_ssh.md) — Notification hook is SSH-aware (fires Discord when remote); leave Stop hook as afplay only
- [Discord ping session labels](discord_ping_session_labels.md) — input-needed Discord pings tagged `[hostname/cwd]` so concurrent sessions are distinguishable; Stop-hook Discord intentionally disabled (spam)
- [Warp paste — no multi-line bash blocks](feedback_warp_paste.md) — Warp flattens newlines on PASTE. Claude-issued Bash HEREDOCs work fine; the gotcha is only when handing user multi-line commands to copy-paste.
- [block-dangerous.sh hook regex blocks ANY ".sh" command](feedback_block_dangerous_regex.md) — PreToolUse `curl.*|.*sh` blocks anything containing "sh". Route through Python scripts.
- [Trust vault Claude's reviews](feedback_vault_claude_reviews.md) — when vault Claude pushes back on a fix summary via `wiki/meta/pipeline-responses.md`, re-verify before dismissing. Caught a wrong "auto-mitigated" claim, a 149-file cleanup glob, and a missed orphan in one round.
- [settings.local.json doesn't reload mid-session](feedback_settings_local_no_reload.md) — new permissions take effect only in sessions opened AFTER the edit. Don't retry denied commands; tell user to run manually or open new session.
- [Defer ≠ demote in backlogs](feedback_deferred_not_demoted.md) — paused items stay at their priority slot with ⏸️ + a concrete revisit trigger; only the "Deferred / out-of-scope" tail is for truly dropped items.
- [Dashboard cards filter at read time](feedback_dashboard_filter_at_read_time.md) — jef-dashboard cards backed by a vault dir filter by frontmatter `status:` at read time, never via file moves on writers. QFR card precedent (2026-05-23).
- [Haiku subagents vs /dispatch pick rule](feedback_haiku_subagents_vs_dispatch.md) — Opus orchestrator + delegate cheap work via 3 Haiku subagent profiles (vault-note-finder / career-inbox-sweep / wiki-link-check) OR `/dispatch --model haiku|sonnet`. Don't add `.claude/settings.json` model overrides for vault subdirs (tried + reverted 2026-05-25).
- [Scout skip categories + vetted instances](feedback_scout_skip_categories.md) — 3 skip-wholesale categories (LLM-observability / MCP-scaffolding / agent-marketplaces) + the exact repos already vetted-and-skipped (wshobson, screenpipe, deer-flow, firecrawl) so they aren't re-proposed. Skip without /vet-addition unless a specific primitive stands out. (Merged 3→1 on 2026-05-28.)
- [BSD flock self-deadlock — never nest agent_lock](feedback_bsd_flock_self_deadlock.md) — `fcntl.flock` is per-open-file-description; same PID nesting `agent_lock()` calls deadlocks against itself silently. Caught 2026-05-27 trading scan wedge (5h21m, PID 58768). Always set timeout_s + use SIGALRM watchdog at daemon entry.
- [Calendar/TCC grant for launchd daemons](feedback_calendar_tcc_launchd_grant.md) — macOS attributes TCC to the responsible process; grant from a one-shot launchd job (not Terminal/SSH) so it attaches to the python binary. Verify via `launchctl kickstart` of the real daemon, not an SSH run.
