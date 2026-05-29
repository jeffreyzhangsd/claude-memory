---
name: reference-trading-data-sources
description: "Free trading data source URLs + auth + quirks for the trading module. SSW JSON path, Finnhub House gotchas, edgartools identity requirement, dead HSW endpoint."
metadata:
  node_type: memory
  type: reference
  originSessionId: fdb149bf-6c44-4815-a928-cb11d066467d
---

External data sources used by [[project-trading]]:

**SEC EDGAR (Form 4 + 13F):** `edgartools` library. Must call `edgar.set_identity("name email@example.com")` before any request — without it everything 401s. Form 4 `get_filings(form="4", filing_date="YYYY-MM-DD:YYYY-MM-DD")` returns thousands of filings/day; call `.obj()` on each to get a `Form4` instance. `obj.common_stock_purchases` is a pandas DataFrame; the alpha rows are `Code=='P'`, `AcquiredDisposed=='A'`, not in 10b5-1 plans (check `obj.footnotes` text).

**Senate Stock Watcher:** the senatestockwatcher.com domain itself was flaky in 2026-05. Pull the raw GitHub mirror instead: `https://raw.githubusercontent.com/timothycarambat/senate-stock-watcher-data/master/aggregate/all_transactions.json`. Row schema: `{transaction_date (MM/DD/YYYY), owner, ticker, asset_description, asset_type, type, amount, senator, ptr_link, disclosure_date}`.

**House Stock Watcher:** dead. `house-stock-watcher-data.s3-us-west-2.amazonaws.com` returns 401 since mid-2025, repo abandoned. Replaced with Finnhub `congressional_trading(symbol, _from, to)` keyed to a watchlist of ~35 large-cap tickers. Free tier; no-ops gracefully when `FINNHUB_API_KEY` is unset.

**Alpaca:** `alpaca-py` SDK (NOT deprecated `alpaca-trade-api`). Paper account is separate from live; same API surface, just `paper=True` on `TradingClient`. Market-hours check via `client.get_clock().is_open` — replaces `pandas_market_calendars`.

**DeepSeek:** OpenAI-compatible SDK pointed at `https://api.deepseek.com`. Default model `deepseek-reasoner` (the SKU "v4-pro" maps to). Used only for escalation per `[[project-trading]]` routing logic — cost target ≤$5/mo.

**Alpaca paper signup:** `app.alpaca.markets/signup` — free, no card needed. Switch top-left dropdown to "Paper" before generating keys. Starting balance: $100k.

**Finnhub free tier:** `finnhub.io/register` — free, requires email. 60 calls/min.
