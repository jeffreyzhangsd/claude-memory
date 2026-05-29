---
name: venv Python version drift (3.12 → 3.14)
description: jef-local-agents venv ships two pythons; venv/bin/python3 → 3.14 but most requirements.txt packages were installed into 3.12 site-packages. Reinstall into 3.14 when imports fail.
metadata:
  type: feedback
---

# venv Python version drift

`~/jef-local-agents/venv/` was originally built on Python 3.12. After a brew Python upgrade, `venv/bin/python3` was relinked to point at Python 3.14, but `requirements.txt` was never reinstalled — so 3.14 only has whatever subset of packages got pulled in by ad-hoc `pip install` later. 3.12's site-packages still holds the original full set.

```text
venv/bin/python3        → 3.14   (active; this is what daemons run with)
venv/bin/python3.12     → 3.12   (still on disk, has the full requirements.txt set)
venv/bin/python3.14     → 3.14   (subset — most of requirements.txt missing)
venv/lib/python3.12/site-packages/   ← full original install
venv/lib/python3.14/site-packages/   ← partial; what's been added since
```

Discovered 2026-05-21 building `script_runner`: `fastapi` was importable via `python3.12` but not via `python3` (= 3.14). The TTS daemon (`tts_server.py`) would hit the same wall if you ever flip it on — it's in requirements.txt but probably missing from 3.14.

## Why

Daemons keep working because they only need the subset that has been pip-installed into 3.14 via ad-hoc commands after the relink. Anything that lived only in 3.12 silently became unimportable.

## How to apply

- **Import fails with `ModuleNotFoundError`** for something listed in `requirements.txt`? Fix: `venv/bin/python3 -m pip install <pkg>` (installs into the active 3.14).
- **Don't rely on `python3.12` paths** in any new code — that interpreter is the stale one and may get pruned.
- **launchd plists already point at `venv/bin/python3`** (= 3.14), so they pick up new installs automatically. Nothing to change there.

## Resolved 2026-05-21

Ran `venv/bin/python3 -m pip install -r requirements.txt` and brought 3.14 fully in line. Picked up 16 missing packages: babel, beautifulsoup4, courlan, dateparser, ebooklib, greenlet, htmldate, justext, lxml_html_clean, playwright, pyee, pytz, soupsieve, tld, trafilatura, tzlocal. All `requirements.txt` imports verified clean from 3.14.

The 3.12 dir (`venv/lib/python3.12/`) is now safe to remove if you want to reclaim disk — but it's small and harmless. Leave it unless doing a deliberate cleanup.

If you actually use Playwright (Air-side autofill), you'll also need `venv/bin/python3 -m playwright install chromium` to fetch the browser binary into 3.14's location. The 3.12 setup may have its own binary already in place.

## Related

- [[script_runner]] — first new code path that hit this; required `fastapi` reinstall
- [[mac-mini]] — only affects Mini; Air has its own venv at `~/Desktop/Projects.nosync/jef-local-agents/.venv/`
