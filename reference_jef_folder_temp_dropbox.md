---
name: reference-jef-folder-temp-dropbox
description: <vault>/jef-folder/temp/ is the user's scratch dropbox for short-lived stuff (pasted images, multi-line commands Claude wants to hand off, ephemeral notes). Claude reads from it on cue and clears it after the related request is done.
metadata:
  type: reference
---

# jef-folder/temp/ — the scratch dropbox

Path: `<vault>/jef-folder/temp/`. Established 2026-05-23. Sits inside the renamed [[jef-folder]] (was `runbooks/`), which is the user's general dropbox for things to share with Claude or save for themselves (`README.md`, `Temp.md`, pasted images, setup notes, etc.).

**Why it exists:** The user wanted one stable, namespaced place to (a) drop a screenshot for Claude to read without dragging it into chat, and (b) receive multi-line shell commands from Claude that would otherwise get flattened by Warp paste or be awkward to copy from chat (see [[feedback-warp-paste]]). Keeps the rest of the vault clean of throwaway artifacts.

**How to apply:**

- **When the user says "I dropped X in temp" / "I've got something in temp" / similar:** read `<vault>/jef-folder/Temp.md` for the latest paste (an Obsidian embed like `![[Pasted image YYYYMMDDhhmmss.png]]`) — the actual file usually lands at vault root, not inside `temp/` itself, because Obsidian's default attachment location is the note's parent dir. Resolve the embed by globbing `<vault>/Pasted image YYYYMMDDhhmmss*`. Also check `<vault>/jef-folder/temp/` for direct file drops.
- **When Claude wants to hand the user a multi-line bash block:** write it to `<vault>/jef-folder/temp/<verb>-<topic>.sh` (or `.md` for mixed prose+commands) instead of pasting the heredoc in chat. Tell the user the path; they `bash <path>` or open it.
- **Cleanup is Claude's job:** delete the temp artifact (image, script, note) once the request that prompted it is fully resolved. Don't pile up. If unsure whether the request is done, leave it and ask. The `Temp.md` index file can stay (empty) — only the embedded asset gets removed.
- **Don't promote anything here.** If something in `temp/` turns out to matter long-term, the user (or Claude on request) moves it into the right home (`notes/`, `code/projects/`, `wiki/<domain>/`, a memory file). Temp is strictly ephemeral.

Related: [[runbooks-renamed-jef-folder]] context note in [[notes/jef-dashboard]] / [[MIND]] if a topic stub exists. The parent `jef-folder/README.md` is for permanent setup runbooks (`air-autofill-setup.md` etc.) — distinct from `temp/`.
