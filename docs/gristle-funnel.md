# Gristle gear funnel: architecture, setup and Klaviyo flows

How the upsell/downsell funnel for the six gear landing pages fits together,
what needs configuring in Shopify admin, and the Klaviyo flows to build.

## Flag: post-purchase upsell app

**Could not confirm whether a post-purchase app (Zipify OCU, Rebuy,
AfterSell, CartFlows) is connected.** The store connection was unavailable
during this build, and installed apps aren't visible from the theme repo.

Why it matters: a true one-click post-purchase offer (charge the card the
customer just used, no re-entry) can only run inside Shopify's checkout via
one of those apps. A theme cannot vault payment.

What was built so it works either way:

- The upsell and downsell pages (`page.upsell-gristle-king`,
  `page.downsell-capo`) are complete, on-brand, full-page offers whose
  accept button is a cart permalink straight to checkout. They work today
  as destinations for the Klaviyo post-purchase flows below, and their copy
  and layout can be ported into OCU/Rebuy the day an app is connected.
- Everything else in the funnel (order bump, waitlist gate, bonus card,
  cross-sell module) is app-free theme code and is live once merged and
  configured.

## Component map

| Component | Where | File |
|---|---|---|
| Order bump (picks/capo toggle) | Cart page, between line items and checkout button | `sections/gk-order-bump.liquid` + `templates/cart.json` |
| Post-purchase upsell page (Lil Heat / Pickguard buyers) | `/pages/gristle-king-offer` (create page, template `upsell-gristle-king`) | `sections/gk-funnel-offer.liquid` + `templates/page.upsell-gristle-king.json` |
| Downsell page (declined Gristle King) | `/pages/capo-offer` (create page, template `downsell-capo`) | same section + `templates/page.downsell-capo.json` |
| Post-purchase upsell page (Kochness Monster buyers) | `/pages/pickup-set-offer` (create page, template `upsell-pickup-set`) | same section + `templates/page.upsell-pickup-set.json` |
| Downsell page (declined pickup set) | `/pages/lil-heat-offer` (create page, template `downsell-lil-heat`) | same section + `templates/page.downsell-lil-heat.json` |
| Waitlist gate (sold out) | Hero of any landing page whose product is sold out | `sections/gk-gristle-hero.liquid` |
| Collector's kit bonus card | Hero of Gristle ST and Supreme pages | hero section, `bonus_*` settings |
| Cross-sell module | Bottom of Gristle ST and Supreme pages | `sections/gk-gristle-crossell.liquid` |

## Setup checklist (Shopify admin)

1. **Create four pages** with these exact handles so the decline chains work:
   - "Gristle King offer", handle `gristle-king-offer`, template `upsell-gristle-king`
   - "Capo offer", handle `capo-offer`, template `downsell-capo`
   - "Pickup set offer", handle `pickup-set-offer`, template `upsell-pickup-set`
   - "Lil Heat offer", handle `lil-heat-offer`, template `downsell-lil-heat`
   Decline chains: Gristle King offer -> `/pages/capo-offer`; pickup set
   offer -> `/pages/lil-heat-offer`.
2. **Pick the bump product**: theme editor → cart page → "GK order bump"
   section → choose the Gristle Picks (or Capo) product. Add the Gristle ST
   and Kochness Monster Supreme products as "Hide when in cart" blocks so
   collector orders never see the bump. If the picks/capo products don't
   exist in the store yet, create them first; the bump hides itself until a
   product is picked.
   **Kochness Monster rule:** the bump section has a "Swap bump when in
   cart" rule pre-wired with the Monster as the trigger. Open it and pick
   the Gristle Capo as the rule's bump product; the rule stays inert until
   that product is picked. Result: Monster in cart -> capo bump; anything
   else -> the default picks bump.
3. **Pick the downsell product**: theme editor → Capo offer page → offer
   section → choose the Gristle Capo product.
4. **Bonus card thumbnails**: theme editor → Gristle ST and Supreme pages →
   hero → upload small square images of the picks and capo (icon
   placeholders show until then).
5. **Cross-sell links**: the two cards point at `/pages/gristle-tone-pickups`
   and `/pages/gristle-king`. If those pages use different handles, fix the
   links in the section settings. (Spec asked for the pickups-only set as the
   second card; it has no landing page yet, so the Gristle King page is used.
   Swap the card's product and link in the editor if preferred.)
6. **Fulfillment note**: the collector's kit is a display-only promise. Add
   the picks and capo to Gristle ST / Supreme orders during fulfillment, or
   attach them as a free-gift automation later.

## Klaviyo flows to build

All captured waitlist emails land in Shopify customers with per-product
tags, which sync to Klaviyo as profile properties. Suggested flows:

### Flow 1: post-purchase upsell (Lil Heat → Gristle King)
- Trigger: Placed Order where item title contains "Lil Heat"
- Timing: 1 hour after purchase (while the gear high is fresh)
- Email: subject "One more for the road" / preview "The King rides with the
  Lil Heat". Body copy mirrors the upsell page (tone-chain logic, Greg
  voice, no urgency), one button: "Add the Gristle King" →
  `https://gregkoch.com/pages/gristle-king-offer`
- Filter: has not purchased Gristle King (any time)

### Flow 2: post-purchase upsell (Loaded Pickguard → Gristle King)
- Trigger: Placed Order where item title contains "Loaded Pickguard"
- Same structure as Flow 1. Copy angle: new pickups deserve the drive Greg
  runs behind them.

### Flow 2b: post-purchase upsell (Kochness Monster -> Gristle-Tone pickup set)
- Trigger: Placed Order where item title contains "Kochness Monster" (and
  does not contain "Supreme": the Supreme is collector tier, no upsell)
- Timing: 1 hour after purchase
- Email: "Feed the Monster" -> `https://gregkoch.com/pages/pickup-set-offer`
- Filter: has not purchased the Gristle-Tone Signature Pickup Set
- Decline chain on the page goes to the Lil Heat offer

### Flow 3: downsell (clicked but didn't buy)
- Trigger: Clicked Email (Flow 1 or 2) and no Placed Order within 3 days
- Email: "Since the King's not for today" → `https://gregkoch.com/pages/capo-offer`

### Flow 4: back-in-stock (Kochness Monster Supreme)
- Trigger: profile tagged `kochness-monster-supreme-waitlist` (synced from
  Shopify customer tags)
- Email 1 (when stock returns, manual send or triggered on tag +
  inventory): "It's ready. We ride." → Supreme landing page
- The same pattern works per product: `gristle-king-waitlist`,
  `kochness-monster-waitlist`, `gristle-st-waitlist`, `lil-heat-waitlist`,
  `gristle-tone-pickups-waitlist`

## Inventory holds (do not run ads yet)

- **Kochness Monster** (1 in stock) and **Kochness Monster Supreme**
  (0 in stock): the FULL funnel is wired for both (Monster: capo bump rule,
  pickup-set upsell, Lil Heat downsell, waitlist tag; Supreme: collector's
  kit bonus, cross-sell module, notify-me gate). The only thing held back
  is paid traffic, not the funnel: hold ads to both until inventory is
  confirmed. Everything else is ad-ready.

## Voice and behavior guardrails baked in

- No countdown timers, no fake urgency, no scarcity language anywhere.
- Decline links are visible and honest. No exit-intent popups.
- All funnel copy is Greg-voice, sentence case, zero em dashes.
- Order bump is a quiet toggle, not a button; no page reload.
- Upsell framing is always addition ("Add the..."), never "upgrade".
