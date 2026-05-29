---
name: load-bearing-rule-ordering
description: 'When prompting local models with multiple "must do" rules, list the most-violated rule FIRST — models drop the last rule when others dominate. Combine with deterministic post-processing as a safety net.'
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

When a prompt has multiple "load-bearing" rules (citations, wikilinks, structure, no-invention), order them by priority and put the most-violated one first. Local models (qwen3:14b, hermes3:8b, gemma4:e4b) deprioritize whichever rule is listed last once another rule dominates the generated output.

**Why:** Discovered 2026-05-15 building `hobby/research.py`. Initial prompt listed CITATION RULES first (load-bearing), WIKILINK RULES second. Model produced 15-19 inline `[n]` citations but ZERO `[[wikilinks]]` — the wikilink rule fell out of attention once citation discipline kicked in. vault_harness validator rejects notes with <2 wikilinks. Reordering wikilinks → rule #1, citations → rule #2 immediately restored 2-4 wikilinks per output without losing citation density.

**How to apply:**

- When writing prompts for local-model daemons that require multiple constraints, order constraints from "most-dropped" to "least-dropped" based on observation.
- Use the phrasing "LOAD-BEARING RULES (in priority order — the validator rejects on rule 1)" to signal stakes to the model.
- For rules that MUST be satisfied (e.g. validator-blocking like wikilink count), add a deterministic post-processing safeguard. Example from `hobby/research.py`: `ensure_min_wikilinks()` injects a `## See also` block with `[[<hobby>]]` + slug-from-item if model produced fewer than 2. This guarantees the validator passes regardless of model variance.
- Don't trust "use 2-4 wikilinks" or "cite every fact" alone — they're aspirational, not enforcing.

Related: [[anti-hallucination-via-web-grounding]] (the prompt that surfaced this), [[pre-expanded-escape-hatch]] (why validation matters in vault_harness).
