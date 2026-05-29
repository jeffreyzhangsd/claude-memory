---
name: deepseek-v4-antagonist-pipeline
description: DeepSeek v4-pro as the ship-daemon Phase-2 antagonist (utils/deepseek_review.py); v4 model retirement + the reasoner max_tokens gotcha.
metadata:
  node_type: memory
  type: project
  originSessionId: c1d85370-6c6a-4870-bc95-b735fd0cc58e
---

# DeepSeek v4-pro antagonist pipeline (ship-daemon variant)

Shipped 2026-05-29 (`utils/deepseek_review.py`). On the Pro plan, the ship-daemon
Phase-2 antagonist runs against **DeepSeek v4-pro** instead of a Claude subagent —
v4-pro does the bulk adversarial read, Claude triages/verifies/synthesizes. Loop:
build → `deepseek_review` → Claude triages (accept real, reject false positives) →
fix → smoke-test → ship. First use (finance/spending_audit + net_worth) caught real
bugs (non-atomic writes, watchdog-before-lock, --force gap, history RMW race) and
I rejected 2 false positives + 1 overclaim.

**Why:** [[plan-claude-pro-deepseek-orchestrator]] — keep Claude quota for
orchestration/verify, push the heavy adversarial read to the paid DeepSeek API.

**How to apply:**

- Model strings: the live DeepSeek API serves ONLY `deepseek-v4-pro` (heavy
  reasoner) + `deepseek-v4-flash`. `deepseek-reasoner` / `deepseek-chat` are
  RETIRED — verify with `client.models.list()` before assuming a model exists.
  `config.DEEPSEEK_MODEL` default is now `deepseek-v4-pro`. Pricing tabled in
  `utils/llm_cost.py` (v4-pro $1.74/$3.48 full rate; 75% promo through 2026-05-31).
- **v4-pro reasoner gotcha:** `max_tokens` caps reasoning_content + visible content
  COMBINED. A code review burns ~8k on reasoning alone, so a tight cap returns an
  EMPTY `.content` (all budget went to hidden reasoning). `deepseek_review` uses
  32k + a finish_reason=="length" truncation guard. Same applies to any v4-pro call
  that needs substantial output. The visible answer is `message.content`; CoT is
  `message.reasoning_content`; `reasoning_tokens` is inside `completion_tokens` for
  billing (don't double-count — see llm_cost compute_cost).
- Run it: `venv/bin/python3 -m utils.deepseek_review <files...> --context "..." --out report.md`.
  It loads `.env` itself; cost is logged under provider="deepseek".
- New shared helper `utils/atomicio.py` (`write_text_atomic` / `write_json_atomic`)
  for iCloud-safe writes — prefer it in new daemons over plain `write_text`.

Related: [[plan-claude-pro-deepseek-orchestrator]], [[project-trading]] (other
v4-pro caller — its escalation was silently hitting the dead reasoner model until
this fix), [[feedback-separate-commits-per-daemon]].
