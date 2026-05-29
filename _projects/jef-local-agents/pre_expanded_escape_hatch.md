---
name: pre-expanded-escape-hatch
description: "`pre_expanded: true` frontmatter flag in vault_harness — when present, skips the gemma4:e4b expansion step and uses the raw .md as-is. For daemons that already produce structured wiki notes (hobby/research.py)."
metadata:
  node_type: memory
  type: reference
  originSessionId: 5779bb56-8ecc-4a4c-bf18-f29a4f326aed
---

`vault_harness.py` (nightly 02:00) routes raw .md files from `agent/raw/` into `wiki/<domain>/sources/`. Step 2 of its pipeline calls gemma4:e4b to "expand" the raw content into a full wiki note. For daemons that already produce structured wiki notes (`hobby/research.py`, anything writing well-formed markdown with headings + wikilinks + frontmatter), re-expansion is harmful — gemma paraphrases the higher-quality output.

**Escape hatch**: set `pre_expanded: true` in the raw .md's frontmatter. vault_harness will skip Step 2 (expansion) and use the original content. Validation (Step 3) and the semantic verifier (Step 5) still run.

**Code location**: `/Users/jef/jef-local-agents/vault_harness.py` `process_note()`, search for `"pre_expanded"`.

**Important nuance**: if validation fails (<2 wikilinks, missing required frontmatter field, body too short), the revision pass (Step 4) STILL runs and invokes gemma. Defeats the purpose of `pre_expanded` for that file. Solution: ensure the daemon's output reliably passes validation. `hobby/research.py` does this via `ensure_min_wikilinks()` post-processing — see [[load-bearing-rule-ordering]].

**When to set the flag**:

- ✓ Daemon writes structured markdown with `# H1`, multiple `## H2` sections, ≥2 `[[wikilinks]]`, complete frontmatter.
- ✗ Raw PDF/video text extraction (still needs gemma expansion).
- ✗ User's own notes dropped into `agent/raw/` (presumably needs structure).

**Auto-tag still runs**: Step 5b (auto-tag concept-tags via gemma) runs on pre_expanded notes too. That's by design — even well-structured drafts benefit from gemma-suggested topical tags beyond the daemon's generic ones (`[<domain>, research-draft]`).

Related: [[load-bearing-rule-ordering]], [[anti-hallucination-via-web-grounding]].
