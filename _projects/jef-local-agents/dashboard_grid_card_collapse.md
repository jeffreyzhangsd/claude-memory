---
name: dashboard_grid_card_collapse
description: jef-dashboard .cards grid collapses flex-column cards — set grid-auto-rows:max-content
metadata:
  node_type: memory
  type: feedback
  originSessionId: f92e43d8-6807-421b-b030-d1ab099ecaf8
---

In `script_runner/static/index.html`, the `.cards` grid (2-col) holds `.card`s that are `display:flex; flex-direction:column; min-height:0`. When a card's content is short/dynamic, the default `auto` row track collapses to roughly the FIRST child's height (the card-head / meta row), then:

- under default `align-items: stretch` → the rest of the card is clipped (card looks empty / "gone").
- under `align-self/align-items: start` → the rest overflows downward and OVERLAPS the next row.

**Fix:** set `grid-auto-rows: max-content` on the specific grid (e.g. `#reports-body`, `#trading-cards`). Rows then fit full card content; short cards just leave grid gap below. Cap tall bodies with `max-height + overflow-y:auto` AFTER this (the cap alone, without the row fix, re-triggers the collapse because an `overflow:auto` child's intrinsic height reads as ~0 for `auto` track sizing).

**Why:** burned 2 wrong CSS-blind fixes (2026-05-29) toggling align-items/overflow before finding the real cause was row-track sizing.

**How to apply:** for any dashboard card-height bug, VERIFY headless before shipping — `venv/bin/python3` has Playwright; `pg.goto("http://localhost:8765")`, click the tab, `getBoundingClientRect().height` on the cards. Don't ship CSS by reasoning alone. Related: [[feedback_block_dangerous_regex]] (curl|… is hook-blocked — use urllib in Python instead).
