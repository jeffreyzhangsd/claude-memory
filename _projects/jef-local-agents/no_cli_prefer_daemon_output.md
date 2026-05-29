---
name: no-cli-prefer-daemon-output
description: "For personal-automation tools, prefer nightly daemons that produce browseable Obsidian artifacts over CLIs the user has to invoke. User reads the artifact when free time appears."
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

When proposing automation for jef-local-agents that serves the user's daily life (hobbies, learning, decisions, etc.), default to **daemons that produce browseable output in Obsidian**, NOT interactive CLIs the user has to remember to invoke.

**Why:** User explicitly pushed back 2026-05-15 on a proposed `jef micro stretch|cali|meditate` CLI ("I don't really want to record/use commands to ask, I just wanted the research to be able to pick up some and then whenever I have time I'll look at the list and perform the exercise I want in the short time I would have"). The friction of remembering to type a command at exactly the right moment defeats the purpose. A nightly daemon that produces a "library" of options the user can scroll on their phone via Termius (or Obsidian sync) wins because it's there when free time appears, not gated on the user proactively asking.

**How to apply:**

- Default automation shape: scheduled daemon → Obsidian markdown output → user reads asynchronously.
- Reserve CLIs for cases where (a) the input must come from the user at runtime (e.g., a specific bullet to polish, a git diff to review) OR (b) the action has to fire immediately (commit message before pushing).
- For "pick something from a curated set" (routines, phrases, drills, research topics), the answer is almost always a daemon that builds the set + Obsidian as the picker UI, not a CLI that does the picking.
- Same principle pushes against bidirectional Discord — see [[claude-code-hooks-ssh]] for the parallel "use the existing Obsidian/SSH path instead of building new infra" pattern.

See [[dev-machines]] for the SSH-tmux-from-phone workflow that makes Obsidian-as-browser viable from anywhere.
