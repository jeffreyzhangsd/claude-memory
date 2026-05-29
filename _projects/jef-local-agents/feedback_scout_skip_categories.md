---
name: scout-skip-categories
description: "Canonical repo_scout skip rule — 3 categories to skip wholesale (LLM observability, MCP scaffolding, agent/plugin marketplaces) + the specific repos already vetted-and-skipped so they aren't re-proposed."
metadata:
  node_type: memory
  type: feedback
  originSessionId: e2e-test-2026-05-25-batch
---

**Rule:** When repo_scout digest surfaces a candidate in any of the three categories below, default to **skip** without running `/vet-addition`. Lift only a specific named primitive if it fills a concrete gap.

**Why:** Batch antagonist pass on 2026-05-25 against 17 repos returned 0 adopts / 1 skim / 16 skips. Skips clustered into three categories, each with the same root reason — wrong scale or already-covered.

## Category 1 — LLM observability platforms

Examples: langfuse/langfuse, comet-ml/opik, mlflow/mlflow, promptfoo/promptfoo, vibrantlabsai/ragas.

**Why skip:** team-scale tools (Docker + Postgres + ClickHouse) for shipping LLM features to multiple humans. User's "observability" is `skill_tracker` (05:00) + `briefing` (05:30) writing morning Obsidian notes — single human, one read per day. Adding a dashboard inverts the load-bearing principle. `ragas`+`promptfoo` also need a judge-model API key → conflicts with [[user-cost-constraint]].

## Category 2 — MCP scaffolding / curricula

Examples: microsoft/mcp-for-beginners, mcp-use/mcp-use, CoplayDev/unity-mcp, grab/cursor-talk-to-figma-mcp, firecrawl/firecrawl-mcp-server.

**Why skip:** user is a **consumer** of MCP servers (obsidian + notion + others), not an **author**. None of the integrations match the stack (Roblox not Unity, no Figma, DDG replaces Firecrawl). **Exception:** if/when authoring an MCP server to expose `wiki_ask.py` / `career/grade.py`, revisit PrefectHQ/fastmcp specifically — logged in `ideas-backlog.md`.

## Category 3 — Agent / plugin marketplaces

Examples: ruvnet/ruflo, wshobson/agents, farion1231/cc-switch, ComposioHQ/composio, Upsonic/Upsonic.

**Why skip:** at lean (≤10 skill / ≤20 command) scale, marketplace scaffolding is premature. `/dispatch` + `/bus` + Haiku subagent profiles + agent_lock already cover multi-agent coordination ([[haiku-subagents-vs-dispatch]]). Detail from the wshobson/agents vet (83 plugins / 191 agents / ⭐35k → 0/4 adopt):

| Candidate                                     | Already covered by                                                        |
| --------------------------------------------- | ------------------------------------------------------------------------- |
| Three-tier model strategy (Opus/Sonnet/Haiku) | [[haiku-subagents-vs-dispatch]] — sharper task-shape pick rule            |
| Modular plugin dirs                           | flat `~/.claude/{skills,commands,agents}/` — not at marketplace scale     |
| Multi-harness generation (`make generate`)    | single-runtime user (Claude Code only) — zero gap                         |
| plugin-eval (static + LLM judge + MonteCarlo) | `/verify` + `agent/sync/` raise-on-failure — Monte Carlo eval unjustified |

## How to apply

- Reading a scout detail file in one of these categories → don't invoke `/vet-addition`. The PostToolUse hook still nudges; ignore unless the repo offers a SPECIFIC primitive outside the category critique.
- A repo can be in a skip category AND have one liftable primitive — treat THAT primitive as the candidate, not the whole repo.
- Re-vet a category only after a scale threshold: ≥3 MCP servers authored → revisit MCP scaffolding; ≥20 skills/commands → revisit marketplace; team > 1 → revisit observability.

## Vetted instances — don't re-propose these exact repos

- **wshobson/agents** (marketplace) — 0/4 adopt, 2026-05-25. See Category 3 table.
- **2026-05-27 trio, 0/3 adopt** — all "the current thing works but here's a bigger thing", no concrete gap:
  - `screenpipe/screenpipe` (24/7 screen capture → skill_tracker): wrong-input (bigger haystack, same needle) + OLLAMA_MAX_LOADED_MODELS=1 collision + OCR-PII explosion. Revisit only with a SPECIFIC skill_tracker blind spot named.
  - `bytedance/deer-flow` (SuperAgent gateway → bus/): multi-tenant scale; `bus/`+`dispatch/`+`agent/sync/` already ship. Defer until a concrete bus limitation exists.
  - `firecrawl/firecrawl-mcp-server` (web extract → trend_scout): duplicates stdlib urllib+xml; if link-following ever needed, 30 lines of urllib + `trafilatura` beats an MCP hop + quota.

**Meta lesson:** the recurring skip reason is "current works but here's a bigger thing" with no named existing-workflow gap. Demand a concrete gap before lifting.

## Related

- [[haiku-subagents-vs-dispatch]]
- [[user-cost-constraint]]
- [[no-cli-prefer-daemon-output]]
- [[wiki-goal-oriented-relevance-filter]]
- [[anti-hallucination-via-web-grounding]]
