---
name: anti-hallucination-via-web-grounding
description: "Recipe for grounding local-model research notes in real web sources via DuckDuckGo (no API key) — snippets become numbered context, model cites by [n], programmatic Sources block appended. Pattern from gpt-researcher."
metadata:
  node_type: memory
  type: reference
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

When a local-model daemon needs to produce research notes without hallucinating URLs, book titles, or person names, ground it in real web search results. Use the gpt-researcher / Open Deep Research recipe:

1. **Search**: top-N results via `ddgs` (DuckDuckGo, no API key needed) or Tavily (API key, higher quality, ~1000 free queries/mo).
2. **Pack**: render snippets as numbered context `[1] Title / URL / Excerpt`.
3. **Strict prompt**: forbid invention; require inline `[n]` citation per factual claim; use ONLY information from snippets.
4. **Programmatic sources block**: tell the model NOT to write `## Sources`; append it yourself with the real URLs after the model's output. Removes a class of model errors (mangled source listings, dropped URLs).
5. **Fallback path**: when search returns nothing (network, rate limit, blocked), use a tightened pure-knowledge prompt that requires `[unverified]` annotations on specific named claims.

**Reference implementation**: `/Users/jef/jef-local-agents/hobby/research.py` — `web_search()`, `_GROUNDED_PROMPT_TEMPLATE`, `_FALLBACK_PROMPT_TEMPLATE`, `render_sources_pack()`, `render_sources_section()`. Search backend can be swapped (Tavily) by changing the `web_search()` body — the prompt + pack format stays the same.

**ddgs install gotcha**: `pip install ddgs` may go to the wrong site-packages on venvs with mismatched Python versions (the jef-local-agents venv had pip → python3.12 but python → python3.14). Use `venv/bin/python -m pip install ddgs` to route through the active interpreter.

**Known quirks**:

- DDG sometimes returns title fields that are concatenations of multiple page titles (artifact of list-page scrapes). Cosmetic; can truncate at display time.
- DDG ranking is noisy — same query on different runs can return different top-5 (some adjacent-domain pages slip in). Trust ranking for v1; content-level filtering is future work.
- Models occasionally wrap citations in backticks (`` `[1]` ``) instead of plain `[1]`. Renders as inline code. Strip in post-processing if it matters.

Related: [[load-bearing-rule-ordering]] (how to structure the prompt's constraint list), [[pre-expanded-escape-hatch]] (so vault_harness doesn't re-paraphrase the grounded output).
