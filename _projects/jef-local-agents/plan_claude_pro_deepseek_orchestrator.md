---
name: plan-claude-pro-deepseek-orchestrator
description: User now on Claude Pro (confirmed via claude.ai/usage 2026-05-29); Claude orchestrates DeepSeek v4-pro for heavy reasoning. DEEPSEEK_API_KEY is in .env.
metadata:
  node_type: memory
  type: project
  originSessionId: db6b198f-0bfd-4bfd-b3c6-f9243db6ae5b
---

# Plan: Claude Pro + DeepSeek v4-pro orchestrator pattern

**Status 2026-05-29: LIVE on Pro.** `claude.ai/usage` reports Pro (login method still shows Max — lag, usage page is authoritative). `DEEPSEEK_API_KEY` is set in `.env` on Mini. Claude acts as orchestrator; DeepSeek v4-pro handles heavy reasoning calls from API.

**Why:** Budget. Max plan unaffordable long-term. Pro + paid DeepSeek API cheaper than Max. Aligns with [[user-cost-constraint]] — concrete execution of the cost-cutting bias.

**How to apply:**

- When designing new pipelines or escalation paths: route heavy reasoning to DeepSeek v4-pro via API, NOT to Claude Opus/Sonnet
- Existing "Cloud — Claude Sonnet/Opus" tier in CLAUDE.md architecture table is going to be partially displaced by DeepSeek v4-pro
- Trading module already uses DeepSeek-reasoner escalation — pattern to copy for future modules
- Don't bake assumptions about Max-tier Claude quota into new features
- `DEEPSEEK_API_KEY` now in `.env` — confirm `config.py` has a slot/reader before any new module uses it

Related: [[user-cost-constraint]], [[project-trading]] (already uses DeepSeek-reasoner), [[feedback-advisor-pattern]] (match model to task — extend to local/cloud/Claude/DeepSeek tiers).
