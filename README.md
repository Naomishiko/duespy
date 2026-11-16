# Duespy — the subscription detective

Marketing site and interactive product demo for **Duespy**: it finds every
recurring charge on your accounts, forecasts what's coming next, and tells you
what to cancel.

> Not affiliated with any other subscription-management product.

---

## What's in here

A plain static site. No build step, no dependencies, no framework.

| File | What it is |
|---|---|
| `index.html` | The whole page — hero, case board, method, evidence, pricing, FAQ, footer |
| `styles.css` | All styling. CSS custom properties, no preprocessor |
| `app.js` | The detective engine: sample data, detection rules, forecasting, cancel advisor |
| `AGENTS.md` | Notes for AI coding agents working in this repo |

## The interactive part

The **case board** is real, working code — not a screenshot. Everything below
runs in your browser against sample data:

- **Subscriptions** — 14 sample services, each stamped *Keep / Review / Cut*,
  with flags for idle days, price creep and duplicates. Filter chips and an
  annual-cost toggle recompute every card.
- **Upcoming charges** — a 45-day calendar where each bar's height is the size
  of the charge, plus "biggest hit" and "next 7 days" panels.
- **Price creep** — before/after bars showing silent price increases.
- **Cancel advisor** — everything ranked by cost × idleness × creep ×
  duplicates, with Keep/Cut buttons that update the recoverable-per-year total.

### The rule engine

`app.js` contains a small classification function that assigns each
subscription a verdict from usage, cadence, sign-up price versus current price,
and overlap groups:

```js
function classify(s) {
  const dup  = s.dupGroup && SUBSCRIPTIONS.some(o => o.id !== s.id && o.dupGroup === s.dupGroup);
  const rose = s.price > s.priceThen + 0.005;
  if (s.usage === 'none' && monthlyCost(s) >= 8) return { verdict: 'cancel', dup, rose };
  if (dup && s.usage !== 'heavy')                return { verdict: 'cancel', dup, rose };
  // …review cases, then keep
}
```

## Run it locally

Any static file server works:

```bash
python3 -m http.server 12000
# open http://localhost:12000
```

Or just open `index.html` in a browser — there's nothing to compile.

## Deploy

The included workflow at `.github/workflows/deploy-pages.yml` publishes the
site to GitHub Pages on every push to `main`.

1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, choose **GitHub Actions**.
4. Push to `main` (or run the workflow manually from the **Actions** tab).

The site then lives at `https://<your-username>.github.io/<repo-name>/`.

## Design

An "audit ledger" aesthetic: bone-paper background with a faint grid, heavy ink
rules, hard offset shadows (no blur), electric lime for primary actions and
coral for alerts.

- **Display:** Bricolage Grotesque
- **Body:** Archivo
- **Labels / data:** Space Mono

## License

Add a license before distributing. MIT is a common choice for a site like this.
