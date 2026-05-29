---
name: Mini Ollama LAN exposure — use Tailscale IP, not jefmini.local
description: How Mini's Ollama is reachable from Air. Bonjour is OFF on the Mini → .local hostnames don't resolve; use the Tailscale IP 100.110.92.114:11434 from Air. Menubar-toggle gotcha + Windows subnet quirk preserved.
type: reference
originSessionId: f0b42f5b-aa80-48cb-bcd8-36f0830d933b
---

Mini's Ollama listens on `*:11434` (all interfaces). Air reaches it via `http://100.110.92.114:11434` (Mini's Tailscale IP).

**Hostname change (2026-05-24):**

- Bonjour is off on the Mini, so `jefmini.local` does NOT resolve from anywhere — neither from Mini's own processes (e.g. `script_runner` couldn't hit `jefair.local`) nor from Air (autofill's draft-model calls failed with "nodename nor servname provided" until OLLAMA_BASE was flipped).
- Use Tailscale IPs end-to-end: Mini = `100.110.92.114`, Air = `100.83.219.92`. `tailscale ip -4` on each machine confirms.
- Air's `.env` (gitignored) now has `OLLAMA_BASE=http://100.110.92.114:11434`. Mini scriptrunner has `AIR_HELPER_URL=http://100.83.219.92:8766` in its launchd plist (`launchd/com.local.scriptrunner.plist`).
- `jef.py` still defaults `JEF_OLLAMA_HOST=http://jefmini.local:11434` in source — overridden via env in the hook scripts that call it. If you write new tools, default to the Tailscale IP or read from `OLLAMA_BASE`.

**Setup state (still current):**

- `OLLAMA_HOST=0.0.0.0:11434` set via `launchctl setenv` AND persisted in `~/Library/LaunchAgents/com.local.ollamaenv.plist` (RunAtLoad)
- Plus: Ollama menubar app's "Allow external connections" / network toggle is ON (this overrides the env var on newer Ollama versions; env var alone wasn't enough on Mini's previous Ollama version)
- Verify: `lsof -iTCP:11434 -sTCP:LISTEN` should show `*:11434`, not `localhost:11434`

**Topology footnote:**

- User's Windows PC is on parent router, Mini is on child router. mDNS didn't cross subnets even when Bonjour was on. Tailscale sidesteps the subnet split entirely — Windows would also use `100.110.92.114`.

**Used by:** `jef.py` CLI, `~/.claude/hooks/jef-condense-bash.sh`, `~/.claude/hooks/jef-postcheck.sh`, `career/autofill.py` (via OLLAMA_BASE on Air), `script_runner` (AIR_HELPER_URL → Air). Open WebUI runs on Mini at port 8080 (talks to Ollama over Mini-localhost — not affected).

**If reachability breaks:**

1. `lsof -iTCP:11434 -sTCP:LISTEN` on Mini — confirm `*:11434` bind
2. `launchctl getenv OLLAMA_HOST` on Mini — confirm `0.0.0.0:11434`
3. Check Ollama menubar app setting — toggle off+on
4. From Air: `ssh air 'cd ~/Desktop/Projects.nosync/jef-local-agents && venv/bin/python3 -c "import urllib.request, json; print(len(json.loads(urllib.request.urlopen(\"http://100.110.92.114:11434/api/tags\").read())[\"models\"]))"'` — should print model count

## Related

[[ollama]] [[ollama-host-menubar]] [[mac-mini]] [[macbook-air]] [[mdns-subnet]]
