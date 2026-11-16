# Duespy — repository notes

## What this is
Marketing site + interactive product demo for **duespy.com**, a "subscription
detective": it finds recurring charges, forecasts upcoming ones, and advises
what to cancel. Not affiliated with any other subscription-management product.

## Stack
Plain static site — no build step, no dependencies.
- `index.html` — full page markup (hero, case board, method, evidence, pricing, FAQ, footer)
- `styles.css` — all styling (CSS custom properties, no preprocessor)
- `app.js` — IIFE, no modules/imports. Contains the sample "case file" data and
  all detection/forecast/advisor logic. Everything runs client-side.

## Design language
"Audit ledger": bone paper background with a faint grid, heavy ink rules,
hard offset box-shadows (no blur), electric lime (`--lime`) primary with coral
(`--coral`) alerts and plum accent. Fonts: Bricolage Grotesque (display),
Space Mono (labels/tags), Archivo (body). Intentionally a light, high-contrast,
print-like look — the opposite of dark purple-gradient SaaS.

## Running locally
```
python3 -m http.server 12000
# http://localhost:12000
```

## Key logic in app.js
- `SUBSCRIPTIONS[]` — 14 sample subscriptions. Fields: `price`, `cycle`,
  `priceThen` (sign-up price for price-creep), `lastUsed` (days), `usage`
  (`heavy|moderate|light|none`), `dupGroup` (overlap detection).
- `classify(s)` → `{ verdict: keep|review|cancel }` — the rule engine.
- `nextCharge(s)` / `daysUntil()` — forecast used by the upcoming calendar and KPIs.
- `renderAll()` re-renders KPIs + all four panels. `state.cut` is a `Set` of ids
  marked to cut; it feeds the "recoverable" KPI.
- Hero transaction feed (`TXNS`) animates in with a staggered reveal.

## Gotchas
- Keep `index.html` id/class hooks in sync with `app.js` selectors.
- `node --check app.js` is a quick syntax gate; there is no test suite.
