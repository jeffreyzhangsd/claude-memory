---
name: Coding workflow — jef CLI + slash commands + Phase 2 hooks (code memory split out 2026-05-03)
description: How the user wants to build new code projects with local-AI augmentation — jef prep at start, hooks auto-fire during, /jef-capture at end. Project memory now lives at vault/code/projects (NOT wiki/code). Reference for any implementation work in their repos.
type: project
originSessionId: f0b42f5b-aa80-48cb-bcd8-36f0830d933b
---

User has a wired-up coding workflow that augments Claude with a local AI helper running on their Mac Mini. Three pieces:

**`jef` CLI on Air** at `/usr/local/bin/jef` — wraps `~/jef-local-agents/jef.py` (Air repo moved out of `Desktop/Projects.nosync/` after the home-dir rename `jeffreyzhang` → `jef`). Talks to Mini's gemma4:e4b at `http://jefmini.local:11434` by default; can override via `JEF_OLLAMA_HOST` (Air now has local Ollama too as of 2026-05-15). Five subcommands as of 2026-05-15: `condense` (stdin → root-cause TLDR), `prep "<desc>"` (CWD + `<vault>/code/projects/*.md` → `./jef-context.md`), `postcheck` (intent vs `git diff`), `commit` (staged diff → conventional commit msg via Hermes), `review [base]` (`git diff <base>...HEAD` → real-issues-only review via qwen3:14b).

**Slash commands** in `~/.claude/commands/`:

- `/jef-prep "<description>"` — run before starting work. Generates briefing, summarizes Goal / file paths / open questions for user review BEFORE coding.
- `/jef-condense <file-or-paste>` — manual TLDR escape hatch (auto-hook usually handles this).
- `/jef-capture` — at end of project. Claude scans repo, extracts patterns, writes **directly** to `<vault>/code/projects/<project>-patterns-<date>.md`. **No wiki pipeline, no LLM rewrite — captures stay verbatim.** Frontmatter is now `type: project-patterns` (not `type: source` / `domain: code`).

**Auto-firing hooks** in `~/.claude/settings.json`:

- **PostToolUse Bash** → `~/.claude/hooks/jef-condense-bash.sh` — auto-condenses Bash output ≥8000 chars. Adds TLDR as additionalContext (raw output stays in context too — net effect is faster focus, not raw token reduction).
- **Stop** → `~/.claude/hooks/jef-postcheck.sh` — runs postcheck if `./jef-context.md` exists in CWD; surfaces verdict via systemMessage.

**Why code is split out from wiki/:** Tried wiki_builder synth for `wiki/code/` and it failed badly: classifier mapped one source to multiple chapter slugs, drafter produced near-identical content per slug (design.md ≈ components.md), and the wikilink ≥2 gate rejected everything in cold-start. Project memory wants verbatim notes, not chapter synthesis. Pulled `code/` out entirely on 2026-05-03.

**Why this stack at all:** Fewer Claude round-trips on big errors + automatic intent verification + accumulating verbatim project memory at `<vault>/code/projects/` so each new project benefits from past patterns via `jef prep`.

**How to apply:**

- For new project work in any of their repos, start with `/jef-prep "<one-liner>"` rather than diving in cold.
- Don't manually pipe through `jef condense` for Bash failures — the hook does it. Use `/jef-condense` only for explicitly chosen content (a file you read separately).
- At the end of meaningful project work (feature ships, big refactor lands), suggest `/jef-capture` so `code/projects/` accumulates the lessons.
- `<vault>/code/projects/` is plain markdown owned by the user. Don't run any LLM rewrite or summarize on captured notes — preserve their language. Each note follows a fixed heading structure (Design choices / Components / Workflow patterns / Common errors / Stack-specific gotchas) so cross-project grep works.
- Backward-compat: if a `.md` lands in `<vault>/agent/raw/` with `domain: code`, vault_harness now short-circuits — copies to `code/projects/` and archives. No expand/validate.
- Hooks fail closed: silent no-op when `jef` not on PATH or jef-context.md missing, so the workflow degrades gracefully when not used.

## Related

[[jef-cli]] [[jef-prep]] [[jef-capture]] [[jef-condense]] [[jef-memory-prune]] [[claude-hooks]] [[code-projects]] [[verbatim-project-memory]] [[mac-mini]] [[ollama]] [[gemma4-e4b]]
