# Memory Index — global preferences (loaded every session, every project)

## Recent activity (auto)

- [Recent activity (auto, last 7d)](recent_activity.md) — what Jeffrey has been building/learning lately

## User profile

- [Design preferences — site aesthetic + structure](user_design_preferences.md) — muted dark, 480px centered container, CSS custom props, system-ui, no frameworks for simple sites
- ["main" = Mini-resident jef-local-agents session](user_naming_main.md) — user's shorthand for the orchestrator; interpret "main session/agent" as this one
- [ML background — first-time learner, HS math only](user_background.md) — assume no calculus / linear algebra / probability theory; frame explanations for a young adult learning ML for the first time

## Tone / writing style

- [Wiki tone — plain language for beginners](feedback_wiki_tone.md) — wiki topic + chapter pages explain math in plain English first, then formulas; analogies + concrete examples; rewrite dense CS229 sections on update

## Reusable references (cross-project)

- [AI principles — load-bearing defaults](reference_ai_principles.md) — 12 triggers, apply without asking: parallel→subagents · hard step→advisor/Opus · cross-repo→`/dispatch` · external tool→MCP · chained Ollama→`agent_lock` · daemon bug→discover-fix-verify · new owner→pre-seed memory · start project→`/jef-prep` · end project→`/jef-capture` · cleanup estimate→×10 · two machines→tag `[Air]`/`[Mini-SSH]`
- [Cloudflare Turnstile gotchas](reference_turnstile_gotchas.md) — invisible mode deprecated at render, Managed > Invisible for low-trust sessions, hostname allowlist + build-time env, console iframe noise unfixable
- [jef-folder/temp/ — scratch dropbox](reference_jef_folder_temp_dropbox.md) — when user says "in temp" read `<vault>/jef-folder/Temp.md` + glob `<vault>/Pasted image *`. Claude writes multi-line bash there instead of chat; clears artifacts after request resolves.
- [Backlog files — check first for "what's next"](reference_backlog_files.md) — `<vault>/ideas-backlog.md` (cross-project dormant) + `<repo>/IDEAS.md` (per-repo active). Rank by return ÷ build-time when no explicit ranking exists.

## Active brands / products

- [minillm.dev brand pointer](minillm_dev_brand.md) — pseudonymous indie product (Notion template on Gumroad + Astro blog at minillm.dev). Runbook at `<vault>/notes/minillm-dev.md`. Read first when working on either surface.

## Dispatch / orchestrator

- [Dispatch worker pre-seed pattern](reference_dispatch_worker_preseed.md) — when adding a new spawnable owner, how to populate `~/.claude/projects/<derived>/memory/` so the spawned worker skips warmup
- [Discover → fix → verify loop](feedback_dispatch_discover_fix_verify_loop.md) — wiki dispatches are read-only discovery + verification; daemon-code fixes happen in main

## Workflow feedback

- [10x autophagy rule](feedback_10x_autophagy.md) — default to far more cleanup / review / canonicalization / adversarial testing than instinct suggests; standing bias toward maintenance over new features
- [Advisor pattern — Sonnet default, Opus for hard calls](feedback_advisor_pattern.md) — model selection by task complexity
- [Subagent execution — default to subagent-driven dev](feedback_subagent_execution.md) — parallel/multi-area tasks via subagents, don't ask
- [Two-machine workflow — tag every command [Air] / [Mini-SSH]](feedback_two_machine_workflow.md) — Air=dev, Mini=runtime; SSH alias `mini`, user `jef`, home `/Users/jef/`
- [Gitignore AI tooling](feedback_gitignore_ai_files.md) — add `.claude/` + `docs/superpowers/` to .gitignore on every new project
- [Git commands authorized in jef-local-agents](feedback_git_commands_authorized.md) — run git up to push without asking in that repo only (global ban still applies elsewhere)
