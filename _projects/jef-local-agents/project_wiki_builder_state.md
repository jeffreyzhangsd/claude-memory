---
name: wiki_builder.py — v1.0.0 shipped (2026-05-01)
description: wiki_builder v1 in production. Daemon installed on Mini, first real run clean, branch merged to main. 101 tests passing. See notes/jef-local-agents.md for full handoff + ops runbook.
type: project
originSessionId: a17a9910-54e2-400c-892c-c336c0d5ce0e
---

## Current state (2026-05-01) — v1.0.0 SHIPPED

**Branch:** merged from `feat/wiki-builder` into `main` at commit `c3c82ad`. Daemon `com.local.wikibuilder.plist` bootstrapped on Mini, fires nightly 02:30. First real run (no `--dry-run`) completed clean.

**Tests:** 101 passing across `tests/test_wiki_builder.py` (94), `tests/test_verify.py` (7), and `tests/test_embeddings.py` (7).

**Trailing fix shipped 2026-05-01 evening (post-merge, on main):** `_bump_home_updated` patches the `updated:` YAML field in `home.md` frontmatter when `order_chapters_pass` rewrites the TOC. Previously the field went stale because no code path touched it. Wired through `today` kwarg from `_populate`. +4 tests (existing-field replace, insert-when-missing, no-op-without-frontmatter, preserves-field-order).

### Optimizations landed this session

1. **Fuzzy section matcher (Aider 0.85)** — `_find_section` falls through to `SequenceMatcher` ratio ≥ 0.85 when exact case-insensitive match fails. Threshold bumped from 0.8 after observing the false-positive `'Attention functions' → 'Activation functions'` (ratio 0.82). Logs `· fuzzy-matched 'X' → 'Y' (ratio=0.NN)` when it fires.

2. **Ollama JSON-schema constrained extractor** — `make_extractor()` now returns `_ConstrainedExtractor`, a duck-typed Agent replacement that POSTs `format=<inlined SourceExtract schema>` to `/api/chat`. Decoder constrains sampling at token level, so `UnexpectedModelBehavior: Exceeded maximum retries` failures are gone (validated: 2-3/run → 0/run). `_inline_refs` flattens pydantic v2 `$defs`/`$ref` for grammar backends. Existing test mocks (`monkeypatch.setattr(wiki_builder, "make_extractor", ...)`) untouched — duck-typed `.run().output` contract preserved.

3. **Paragraph pre-filter via embeddings** — `_hybrid_merge` embeds each existing h2 section once per slug (first 500 chars + heading), then per source drops paragraphs whose max cosine sim to any section is > 0.85 (`PARAGRAPH_PREFILTER_THRESHOLD`). Best-effort: any embed failure falls through to unfiltered source. Logs `· slug stem pre-filter: dropped N/M paragraphs (~K chars saved)` when it fires. **Result on full-rebuild ML run: zero section-merge timeouts** (vs prior runs hitting 1-3 timeouts at 300s).

### Optimization passes attempted but BACKED OUT

- **Embedding pre-router for classifier (Idea 2A — original notes brief):** classifier shortlist via top-K cosine ranking. Tried with K=4 and K=8. K=4 dropped multi-topic source coverage badly (15/22 pages, phantom `attention` chapter). K=8 recovered to 17/22 but still missed `vector-calculus`/`generative-learning-algorithms` for textbook-style sources. Speedup vs cost was ~25-30s/run saved at the cost of routing complexity + coverage holes — not worth it. Reverted in same session.
  - Surviving from that work: classifier system prompt **hardened** with "MUST be copied verbatim from TOC, do NOT invent" — kept (bug fix); **invented-slug filter + slug dedup** in `_populate` — kept (defense in depth).

### Pre-existing hardenings (still in place from prior sessions)

- Per-source isolation: extractor failure on one source = skip that source, mark in `failed_source_files`, hash not bumped → retried next nightly. Siblings still apply via hybrid path. Full-merger fallback only on `mode=conflict`.
- Coalesced section_merger: blocks targeting same h2 batched into one LLM call.
- Defensive: `_ensure_wikilinks_preserved` (post-full-merger), frontmatter inject/repair, footnote-def normalization, adaptive shrunk-input extractor retry.
- Extractor: slim chapter view (h2 + 250 chars/section), source capped 4000, output budget ≤6 blocks/≤800 chars per system prompt.
- Per-call timeouts: extractor 240s primary + 180s shrunk-retry, section-merger 300s, full-merger 240s × (1+n_sources), drafter 600s.

## Hybrid merge architecture

| Mode       | Trigger                                     | Writer                                                                    | Cost      |
| ---------- | ------------------------------------------- | ------------------------------------------------------------------------- | --------- |
| `append`   | pure-new content under existing/new h2      | `_apply_append` (Python splice)                                           | ~10ms     |
| `merge`    | extractor flags rewording of target section | `_apply_merges` → `make_section_merger` (qwen3:14b, scoped, 300s timeout) | ~60-260s  |
| `conflict` | extractor flags contradiction               | full merger (today's flow over all sources)                               | ~165-720s |

Extractor sees existing chapter (slim view) + h2 list + source stem, returns `SourceExtract.blocks: list[ExtractedBlock]` via Ollama format-constrained sampling. Empty `target_section` skipped per-block; extractor exception isolates source. Footnote defs auto-normalized; frontmatter `sources:` list updated.

## Hardware reality (Mini M4 16GB)

| Tier                    | Model                      | Server         | Job                                          |
| ----------------------- | -------------------------- | -------------- | -------------------------------------------- |
| Fast                    | gemma4:e2b                 | Ollama 11434   | classifier                                   |
| Primary                 | gemma4:e4b                 | Ollama 11434   | drafter (empty pages) + extractor (pre-pass) |
| Wiki-synth              | qwen3:14b                  | Ollama 11434   | section-merger, full-merger, clusterer       |
| Embed                   | nomic-embed-text           | Ollama 11434   | paragraph pre-filter (Idea 2B)               |
| Heavy (triage/briefing) | qwen3.5:35b-a3b-ud-iq3_xxs | llama.cpp 8081 | unchanged — used by router/triage/briefing   |

`OLLAMA_MAX_LOADED_MODELS=1`. One model hot at a time. Embed model swaps in fast (~270MB) per pre-filter pre-pass.

`--force-rebuild` of full ML domain: ~30-40 min (down from prior 1-2h pre-hybrid). Steady-state nightly with 1-2 changed sources should be ~3-5 min.

## Latest verified run (2026-05-01, full ML rebuild)

- 21 pages affected (full domain coverage)
- 0 quarantined, 0 conflicts, 0 phantom chapters
- 0 extractor `UnexpectedModelBehavior` failures
- 0 section-merge timeouts (vs prior runs: 1-3 per run)
- Pre-filter fired on 3/24 source-extracts, dropping 6 paragraphs total (~3.7KB saved)
- Long pole: dense merges 215-260s but all complete under 300s ceiling

## Failed experiments (do not retry without new evidence)

- **Pre-summarize via Gemma e4b before merge** (2026-04-27). Net slowdown ~70%. Three-model RAM thrash on 16GB.
- **Search-replace as 4th edit mode (Aider-style)** (notes only — never tried). Aider's leaderboard shows 7-14B models do worse with diff format than whole-file. qwen3:14b same regime.
- **Bundled per-source extractor with shared block list** — one bad block invalidates whole slug. Per-block isolation is what's shipped.
- **Heavier model than qwen3:14b for synth** — 35B-A3B was tried via llama.cpp mmap, too slow on M4 16GB.
- **Embedding pre-router for classifier (Idea 2A)** — see above, backed out same session.

## Open paths (low priority, only if future need surfaces)

1. **Conflict-aware section merger** — extend section-merger to emit Conflict callouts inline so `mode=conflict` no longer needs full-page fallback. Eliminates the last ~720s code path. Not pressing — conflict mode is rare in observed runs.
2. **Cloud fallback for stuck full-merger** — ~$1-3/month. User declined for cost.
3. **Tighten paragraph pre-filter threshold** to 0.80 — would drop more paragraphs but risks losing nuanced "topically similar but additive" content. Stay at 0.85 unless we observe over-merge.
4. **Tighten extractor prompt** to bias `use=true` (less risk of false-skip).

## Open caveats

- launchd `com.local.llamacpp.plist` keepalive auto-restarts — manual `kill PID` doesn't stick. Use `launchctl unload/load` or update plist + reload.
- iCloud vault is shared between Air and Mini; smoke sources can be dropped via that path.
- Verifier skipped on hybrid path output (Python-spliced + section-merged); only runs on full-merger fallback.

## Files of note

- `wiki_builder.py` — pipeline + agent factories. Central hooks: `_hybrid_merge`, `_apply_append`, `_apply_merges`, `make_section_merger`, `make_extractor` (`_ConstrainedExtractor`), `_run_agent`, `_find_section` (fuzzy fallback), `_prefilter_paragraphs`, `_section_snippets_for_filter`.
- `embeddings.py` — `embed(texts)` + `cosine_sim(a, b)`. Used by paragraph pre-filter.
- `config.py` — model names (incl. `MODEL_EMBED = "nomic-embed-text"`), ctx, paths.
- `launchd/com.local.llamacpp.plist` — heavy server config.
- `tests/test_wiki_builder.py` (90), `tests/test_verify.py` (7), `tests/test_embeddings.py` (7).

## Mini setup (one-time, done)

```bash
ollama pull nomic-embed-text   # ~270MB, used by paragraph pre-filter
```

## Related

[[wiki_builder]] [[wiki-domains]] [[vault_harness]] [[ingest]] [[wiki-ingest-pipeline]] [[qwen3-14b]] [[nomic-embed-text]] [[hybrid-python-llm]] [[verifier-shared-module]] [[mac-mini]] [[launchd]]
