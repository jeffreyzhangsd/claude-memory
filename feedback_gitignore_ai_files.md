---
name: Gitignore AI tooling files
description: Always add .claude/mind.mv2 and similar AI session files to .gitignore when creating new projects
type: feedback
---

Always add `.claude/` to `.gitignore` when scaffolding or setting up a new project.

**Why:** The entire `.claude/` directory is session-local Claude tooling (mind.mv2, settings, hooks) — none of it belongs in version control.

**How to apply:** When creating a new project (scaffold task or first commit), add a `# Claude / AI tooling` section to `.gitignore`:

```
# Claude / AI tooling
.claude/
docs/superpowers/
```

Add it early — before the first commit if possible.

## Related

[[claude-memory]]
