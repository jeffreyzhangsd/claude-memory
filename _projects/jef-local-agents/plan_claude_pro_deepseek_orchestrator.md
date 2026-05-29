---
name: plan-claude-pro-deepseek-orchestrator
description: User downgrading Claude Max → Pro; Claude becomes orchestrator of DeepSeek v4-pro for heavy reasoning. DeepSeek API key not yet in .env.
metadata:
  node_type: memory
  type: project
  originSessionId: db6b198f-0bfd-4bfd-b3c6-f9243db6ae5b
---

# Plan: Claude Pro + DeepSeek v4-pro orchestrator pattern

Going forward Claude plan reverts Max → Pro. Claude acts as orchestrator; DeepSeek v4-pro handles heavy reasoning calls from API. DeepSeek API key NOT yet in `.env` here on Mini (was told to Air session already, not Mini until now).

**Why:** Budget. Max plan unaffordable long-term. Pro + paid DeepSeek API cheaper than Max. Aligns with [[user-cost-constraint]] — concrete execution of the cost-cutting bias.

**How to apply:**

- When designing new pipelines or escalation paths: route heavy reasoning to DeepSeek v4-pro via API, NOT to Claude Opus/Sonnet
- Existing "Cloud — Claude Sonnet/Opus" tier in CLAUDE.md architecture table is going to be partially displaced by DeepSeek v4-pro
- Trading module already uses DeepSeek-reasoner escalation — pattern to copy for future modules
- Don't bake assumptions about Max-tier Claude quota into new features
- When DeepSeek key lands in `.env` watch for `DEEPSEEK_API_KEY` — flag if config.py needs a slot

Related: [[user-cost-constraint]], [[project-trading]] (already uses DeepSeek-reasoner), [[feedback-advisor-pattern]] (match model to task — extend to local/cloud/Claude/DeepSeek tiers).
