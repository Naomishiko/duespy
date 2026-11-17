# Duespy — repository notes

## What this is
Marketing site + interactive product demo for **Duespy**, a "subscription
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

## Signup and email
Two separate paths, do not conflate them:

**Paid signup → Stripe.** Every pricing CTA is an `a[data-plan]` resolved by
`syncCtas()` from `STRIPE_LINKS`. Keys are `` `${plan}|${cycle}` ``. Stripe
collects the buyer's email and sends receipts and trial reminders; this site
never touches it. Stripe is in **test mode**, and test mode deliberately sends
no email, so nothing arriving is expected until live links replace the test
ones.

**Free email capture → Formspree.** The finale section has a real form
(`#finaleForm`, input `#finaleEmail`) posting to `FORM_ENDPOINT` at the top of
`app.js`. An earlier version of this form used `onsubmit="return false;"` with
no `name` attribute, so it silently discarded every address; it was removed and
has since been rebuilt to actually submit. Set `FORM_ENDPOINT` to the Formspree
URL (`https://formspree.io/f/xxxxxxxx`) to activate it. While it is empty the
form validates the address then reports plainly that it is not connected — it
must never claim success it cannot deliver.

Note the input needs `name="email"`; without it the POST body is empty. There is
also a `_gotcha` honeypot, and Formspree's free tier allows 50 submissions/month
(their copy calls it "for testing and development"). Formspree only forwards to
an inbox — it is not a mailing list, so bulk updates need a list service that
handles unsubscribe.

## Gotchas
- Keep `index.html` id/class hooks in sync with `app.js` selectors.
- `node --check app.js` is a quick syntax gate; there is no test suite.
- Board interaction feedback depends on scroll position, not just state. The
  panels range from ~380px to ~1970px tall and the reset button sits at the
  bottom of the board while the KPIs it clears sit at the top. Any change that
  swaps panel content or mutates state must also bring the affected region into
  view, or it will look broken even though the logic ran. Verify such changes by
  measuring changed *viewport* pixels, not by asserting DOM state.
- `styles.css` disables all animation/transition globally near the bottom of the
  file, so feedback cannot rely on a transition firing — use a discrete class
  toggle and remove it on a timer.
- The sticky `.nav` (z-index 900) overlaps board controls at some scroll
  offsets; `scroll-padding-top` on `html` is 86px, so scrollIntoView targets
  should clear the nav height.

## Known open items
- No domain is owned yet. `duespy.com` is **not registered** (RDAP 404, DNS
  NXDOMAIN) and the footer no longer prints any domain, so nothing advertises an
  address the project does not control. Register one before adding it back.
- Stripe is in test mode; going live needs the sandbox claimed and the four test
  Payment Links swapped for live ones.
- No `CNAME` file: the site is served from the `naomishiko.github.io/duespy/`
  project path. Adding a custom domain requires a `CNAME` file, an apex A record
  to GitHub's four IPs (or an ALIAS/ANAME), a `www` CNAME to
  `naomishiko.github.io`, and "Enforce HTTPS" enabled after the cert issues.
