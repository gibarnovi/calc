# GBOX — corrugated box & sheet price calculator

A single-file, mobile-first price calculator for corrugated boxes (ყუთი) and
sheets (ლისტი), built for one-handed use on an iPhone in portrait.

**Live:** https://gibarnovi.github.io/calc/

Everything lives in **`index.html`** — no build step, no dependencies, no network
requests. Open it, type the dimensions the customer gives you, read the price.

## Add it to the home screen (iPhone)

Open the live URL in **Safari** (not Chrome — only Safari can install web apps on
iOS) → **Share** → **Add to Home Screen**.

It launches full-screen with no browser chrome, sits under the Dynamic Island
correctly, and respects the home indicator. The home-screen icon (green box) is
confirmed working on iPhone 17 Pro Max.

> **Updates are cached for about 10 minutes.** GitHub Pages serves with a
> 10-minute cache and there is no service worker, so after uploading a new
> `index.html` the home-screen app can keep showing the old version for a few
> minutes. Force-quit it and reopen to pick up changes sooner.

> **Offline:** iOS usually keeps a home-screen web app cached, so it will normally
> open with no signal — but that is the system's cache, not a guarantee. If you
> need certainty, this needs a service worker (one extra file).

## Republishing after a change

Upload the new `index.html` through the GitHub web UI (Add file → Upload files),
or push it. GitHub Pages redeploys automatically within a minute.

## Using it

**Calculator tab**

- Three taps set type / layers / colour, then four numbers: length, width,
  height, quantity.
- The price updates **as you type** — there is no Calculate button.
- **Shortcut:** type `400x300x200` into the length field and it splits itself
  across the three fields. Typing `x` or a space anywhere jumps to the next
  field. A comma is a decimal point (`2,75`), never a separator.
- Choosing **ლისტი / Sheet** hides the height field, because the sheet formula
  does not use it.
- The chevron on the total opens the breakdown: per-product lines, combined
  area, the x2 factor and the resulting markup.
- The copy button puts a customer-safe quote on the clipboard — dimensions,
  quantity, unit price and totals in GEL only. No m², no USD, no discount factor.

**Variables tab**

Exchange rate, the four base prices, the box construction allowances, the four
discount-curve constants, defaults for new product cards, language (ქართული /
English) and decimal places. Everything is saved on the device and survives
closing the app; dimensions always start blank for the next customer.

The **Backup** section exports every setting as JSON. Worth doing once after you
have tuned your prices — iOS can evict local storage.

## The pricing model

Prices are **VAT-inclusive**: the base prices already include VAT, so no VAT is
added anywhere.

```
box   →  blank length = (L + W) × 2 + 40 mm
         blank width  = H + W + 2 mm  (3-layer)  /  H + W + 4 mm  (5-layer)
sheet →  blank length = L,  blank width = W

m² per unit   = blank length × blank width ÷ 1,000,000
combined area = Σ (quantity × m² per unit)      ← across ALL products

x2 = 0.57 × e^(−0.00008 × combined area),  or 0.15 when the area exceeds 10,000 m²

unit price (USD) = base price ÷ (1 − x2) × m² per unit
unit price (GEL) = unit price (USD) × exchange rate
```

Base prices, USD per m², VAT included:

| | რუხი / Grey | თეთრი / White |
|---|---|---|
| 3-layer | 0.39 | 0.41 |
| 5-layer | 0.63 | 0.65 |

Two things worth knowing about this model:

- **x2 is shared.** It comes from the combined area of *every* product in the
  quote, so adding a product changes the price of the others too. That is how the
  original calculator worked, and it is why the numbers move while you type.
- **There is a step at 10,000 m².** Just below it x2 ≈ 0.2561 (markup ×1.344);
  just above, x2 drops to 0.15 (markup ×1.176). An order of 10,001 m² is priced
  lower per m² than one of 9,999 m². This is deliberate and is preserved exactly;
  all four constants are editable in the Variables tab if you ever want to change
  it.

## Editing the constants by hand

Every pricing constant is in one `DEFAULTS` object near the top of the `<script>`
block in `index.html` — search for `const DEFAULTS`. Editing it changes the
built-in defaults; the Variables tab overrides them per device.

Note that values you have already changed in the Variables tab are stored on the
device and take priority. Use **Variables → Backup → Reset** to fall back to the
values in the file.

## Provenance

Rebuilt from an earlier desktop-shaped version of this calculator. The pricing
model was carried over unchanged and verified: 10,005 randomised quotes plus
explicit cases straddling the 10,000 m² threshold were run through both the
original formulas and the new engine, and every blank size, area, unit price and
total came out bit-identical.
