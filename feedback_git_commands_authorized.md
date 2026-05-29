---
name: git-commands-authorized
description: In the jef-local-agents project, run git commands up to push without asking. User has authorized this in CLAUDE.md and re-stated multiple times; treat it as a durable session preference, not per-turn permission.
metadata:
  type: feedback
---

**Rule:** In `~/jef-local-agents/`, run `git add`, `git commit`, and any other read/local git command without asking. The only command that still needs explicit per-turn approval is `git push` — surface that and let the user run it.

**Why:** The project's `.claude/CLAUDE.md` explicitly lifts the global git ban for this repo via a `settings.local.json` allowlist. The global `~/.claude/CLAUDE.md` blocks `git add/commit/push` by default for safety, but the project override applies here. User has re-stated this preference across multiple sessions ("I've said you can do all git commands up to push several times") — so asking each time is friction, not safety. They commit at natural boundaries themselves and want the same behavior from Claude.

**How to apply:**

- After a logical unit of work (bug fix shipped, helper added, test suite green), stage + commit the relevant files with a descriptive message and Co-Authored-By trailer.
- Use multi-file `git add <paths>` over `git add .` to avoid staging unrelated changes (the repo often has unstaged work from other sessions).
- Stop short of `git push` — output the push command so the user can run it.
- This authorization scope is `~/jef-local-agents/` only. Other projects still respect the global ban.
- See [[git-commit-style]] if it exists for the project's commit message style; otherwise `git log --oneline -5` to mimic recent entries.

**Counter-rule:** Outside `~/jef-local-agents/`, the global ban still applies — output commands for the user.
