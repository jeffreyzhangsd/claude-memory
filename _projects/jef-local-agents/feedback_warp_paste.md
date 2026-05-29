---
name: Warp paste — no multi-line bash blocks
description: User pastes commands into Warp terminal; multi-line code blocks lose newlines. Prefer Write-direct or single-line commands.
type: feedback
originSessionId: a17a9910-54e2-400c-892c-c336c0d5ce0e
---

When giving the user terminal commands to run, do NOT use multi-line bash blocks (heredocs, `cat <<EOF`, multi-line `printf` chains, multi-line Python `-c` strings). Warp's "highlight and paste" flattens whitespace and breaks them.

**Why:** Warp paste mangled a heredoc (typo `ccat`, missing EOF) during the wiki_builder smoke test on 2026-04-26, sent us into a multi-step git stash recovery loop. Avoidable.

**How to apply:**

1. **Prefer writing files directly via the Write tool** when the file is on a path I can reach (Air's filesystem, the iCloud vault, the repo). User then runs only a short command to invoke it. This is the default.
2. **Fall back to single-line commands** when Write isn't applicable. Use `python -c "..."` with `\n` escapes, or `printf '%s\n' 'line1' 'line2' > file`, on one logical line.
3. **Multi-line scripts only via "save then run"**: instruct the user to `nano /tmp/foo.sh`, paste inside the editor (which preserves newlines), then `bash /tmp/foo.sh`. Reserved for genuinely complex scripts.

The user's stack: develops on Air (MacBook Air, `~/<repo>` — moved out of `Desktop/Projects.nosync/` after the home-dir rename), deploys to Mini (Mac Mini at `~/<repo>` over ssh, user `jef`). iCloud vault is shared between both — Write to vault on Air syncs to Mini in seconds; that's the cleanest channel for cross-machine file drops.

**2026-05-16 update**: HEREDOCs `cat <<'EOF' ... EOF` actually work fine when `git commit -m "$(cat <<'EOF' ... EOF)"` is run by Claude via the Bash tool (no Warp paste involved). The Warp-paste problem is real ONLY when _handing the user_ a multi-line command to copy-paste. When Claude runs the command itself (and the user explicitly authorized it for the session), the HEREDOC is fine.

## Related

[[warp-terminal]]
