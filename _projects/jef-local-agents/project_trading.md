---
name: project-trading
description: "trading module — Form 4 + 13F + congressional signals, LOCAL-ONLY hermes3:8b scoring (DeepSeek escalation removed 2026-05-29), Alpaca paper orders, dashboard tab + 3 launchd daemons. Shipped 2026-05-27."
metadata:
  node_type: memory
  type: project
  originSessionId: fdb149bf-6c44-4815-a928-cb11d066467d
---

`~/jef-local-agents/trading/` — built end-to-end on 2026-05-27.

**Architecture:** sources/{form4,funds,congress}.py → signals.py orchestrator (gate + dedup + price ctx) → analyze.py (LOCAL-ONLY hermes3:8b; DeepSeek escalation removed 2026-05-29 per local-first mandate — local fail now yields an "error" verdict, never a buy) → execute.py (alpaca-py notional market orders, ≤$50 auto-buy, $500/mo budget) → notify.py (Discord). monitor.py runs stop-loss/take-profit/swing every 30min. go_live.py is the only sanctioned paper→real flip.

**Daemons:** trading-signals (every 1h weekday PT 07:00–13:00 — tightened 2026-05-27 from 2h after caches + watchdogs landed), trading-monitor (every 30min), trading-daily (14:30 PT). missed_runs.py logs gaps + Discord-pings if gap overlapped NYSE hours.

**Hardening (2026-05-27 — post-deadlock):**

- **Deadlock fix.** `analyze.analyze_signal` used to acquire `agent_lock("trading-analyze")` while caller `daemon.run_scan` already held `agent_lock("trading-scan")`. Same-process / different-fd flock = self-deadlock (BSD flock is per-open-file-description). Removed inner lock; outer scan lock is sufficient. See [[feedback-bsd-flock-self-deadlock]].
- **Watchdogs.** trading/daemon.py SIGALRM wrapper: scan=10min, monitor=3min, daily=5min. Plus `agent_lock("trading-scan", timeout_s=180)`. On timeout, daemon logs, records run completed (no launchd retry-storm), returns {aborted}.
- **Caches.** yfinance per-ticker (60min TTL, 15min for empty/delisted) at `<vault>/trading/.price_cache.json`; form4 SEC fetch (75min TTL) at `<vault>/trading/.form4_cache.json`. Both flushed via try/finally so watchdog aborts don't lose work. yfinance flush is read-modify-write with newest-`cached_at` merge + atomic rename — safe under concurrent writers. form4 `days_old` recomputed at cache-read time (don't let stale age leak past freshness cutoff). Cold scan 226s → warm 3.5s = 64.7×.
- **Freshness gate strict on missing price.** `signals._freshness_gate(price_now=None)` returns "stale" so delisted/unknown tickers (LENB, HEIA, CMA, etc.) don't burn Hermes scoring calls.
- **Discord daily summary upgraded.** `daemon._format_daily_summary` builds an aligned monospace table (top 5 by market value, per-row flags 📈/📉/🎯/🔴), callout lines for at/near stop-loss + at/near take-profit (within 3% bands), biggest-mover line. ~580 chars, well under 2000 cap. Thresholds sourced from `trading.monitor` so they stay in sync.

**Dashboard:** http://jefmini:8765 → trading tab. 6 cards (account+regime / positions / alerts / signals / budget / equity / missed). `/api/trading` returns separate `actionable` list (filtered to buy|alert). `/api/trading/place-buy` for manual buys from the alerts card.

**Why:** Form 4 insider buys are the primary alpha (academic literature, 2-day filing deadline). 13F + congressional are supporting context (Pelosi alpha mostly arbitraged out by 2026, NANC ETF barely beats SPX once NVDA overweight is stripped).

**Manual vs auto split (post-ship):** `place_buy(manual=True)` bypasses TRADE_AUTO_MAX + TRADING_BUDGET; records spend in `manual_spend.json` instead. Dashboard endpoint passes manual=True. Auto-trade cap and monthly budget only apply to daemon-initiated buys.

**Acted-set:** `vault/trading/acted_signals.json` keyed by signal filename. Dashboard's actionable list (separate from `signals` field) filters this set out at read time. Populated by /api/trading/place-buy + /api/trading/dismiss-alert. Lets alerts clear immediately as the user works through them without mutating the signal notes.

**launchctl env load is load-bearing:** `script_runner/__main__.py` calls `load_dotenv()` BEFORE importing the FastAPI app. Without it, Alpaca/DeepSeek keys never reach the dashboard process even when present in .env.

**How to apply:** When debugging trading flow, start at signals.py (orchestrator), then analyze.py (router decisions), then execute.py (Alpaca interactions). Tests are in `tests/test_trading_*.py`. Verbatim design notes (incl. post-ship polish iteration) in `<vault>/code/projects/trading-patterns-2026-05-27.md`. User-facing primer at `<vault>/notes/trading.md`.

Related: [[reference-mini-ollama-lan]] (hermes via Ollama on Mini), [[project-coding-workflow]] (jef-prep + jef-capture for context handoff).
