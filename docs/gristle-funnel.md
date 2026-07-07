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
| Post-purchase offer, customer order page | `/account` order pages, native + editable | `sections/gk-order-status-offer.liquid` + `templates/customers/order.json` |
| Post-purchase offer, thank-you page | Settings > Checkout > Additional scripts (manual paste) | `docs/thank-you-page-offer.html` |
| Post-purchase offer, confirmation email | Settings > Notifications > Order confirmation (manual paste) | `docs/order-confirmation-offer.html` |
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

## Native post-purchase offers (no app, no Klaviyo)

Three channels show the same single offer per order, chosen by the same
priority mapping. None of them require an app or Klaviyo.

**Trigger mapping (priority order, first match wins; a rule is skipped if
the offer product is already in the order):**

| # | Order contains | Offer shown |
|---|---|---|
| 1 | Lil Heat | Gristle King |
| 2 | Gristle King | Lil Heat |
| 3 | Loaded Pickguard | Gristle King |
| 4 | Kochness Monster (not Supreme) | Gristle-Tone Signature Pickup Set |

**Channels:**

1. **Customer order page (theme, deploys with this repo, theme-editor
   editable):** `sections/gk-order-status-offer.liquid`, registered on
   `templates/customers/order.json` with the four rules pre-wired as
   blocks. Drag blocks to change priority; every pairing, copy line, image
   and label is a block setting. Accept is a cart permalink into a
   pre-filled cart. Decline links to /account/orders.
   Honest scope note: the checkout THANK YOU page cannot host theme
   sections, so the theme-owned surface is the account order page (which
   the confirmation email's "view your order" button also lands on).
2. **Checkout thank-you page (manual paste-in):**
   `docs/thank-you-page-offer.html` goes into Settings > Checkout > Order
   status page > Additional scripts. Same mapping, same look, inline
   styles. Legacy feature being phased out by Shopify in favor of checkout
   extensibility (app territory); works today, treat as temporary. Not
   theme-editor editable.
3. **Order confirmation email (manual paste-in):**
   `docs/order-confirmation-offer.html` goes into Settings > Notifications
   > Order confirmation. Backup channel showing the SAME offer, table-based
   HTML with inline styles (no web fonts in email). The mapping is
   duplicated in a comment at the top of the file: when you change the
   theme section's rules, update the email block by hand to match.
   Replace the two placeholder variant IDs before going live (the Gristle
   King's is pre-filled).

**What still needs an app (out of scope for all of the above, on purpose):**

- One-click accept that charges the saved card without re-entering payment
  (Zipify OCU / Rebuy / AfterSell territory). All native accepts land in a
  pre-filled cart and go through normal checkout.
- Click-based downsell chaining ("show the downsell only if they did not
  click the upsell"). A theme and a static email cannot detect a link
  click. The native approximation is time-based, via Shopify Flow below.

## Shopify Flow: delayed downsell (spec to build manually)

Flow is built in Shopify admin (Apps > Flow, free on every plan), so it
cannot ship in this repo. Build three workflows, one per trigger group.
The logic is time-based, not click-based: 24 hours after the order, if the
customer has not ordered again, they get the downsell email.

### Workflow A: "Downsell capo (Gristle King path)"
1. Trigger: **Order created**
2. Condition: **Check order line items** - any line item title contains
   "Lil Heat" OR "Loaded Pickguard" OR "Gristle King"
3. Condition (same step or nested): NO line item title contains "Gristle
   Capo" (already bought the downsell)
4. Action: **Wait** - 24 hours
5. Condition: **Customer's last order ID equals this order's ID**
   (in Flow's condition builder: customer > lastOrder > id, equals,
   order > id). If a newer order exists, they came back on their own: stop.
6. Action: **Send internal email** is for staff; use **Send email**
   (marketing-safe transactional alternative) or, once Klaviyo is
   reconnected, **Send Klaviyo event** instead. Email to
   {{ order.customer.email }}:
   - Subject: Since the King's not for today
   - Body: short Greg-voice note plus one link:
     https://gregkoch.com/pages/capo-offer
   Note: Flow's plain "Send email" sends from the store with basic
   formatting only; if you want the branded template, the Klaviyo event
   route is the upgrade path.

### Workflow B: "Downsell Lil Heat (Kochness Monster path)"
Same six steps with:
- Step 2 condition: any line item title contains "Kochness Monster" AND no
  line item title contains "Supreme"
- Step 3: NO line item title contains "Lil Heat"
- Step 6 link: https://gregkoch.com/pages/lil-heat-offer
- Subject: Since the pickups are for another day

### Downsell product per trigger pair (matches the existing decline chains)

| Bought | Upsell shown (status page/email) | Flow downsell after 24h |
|---|---|---|
| Lil Heat | Gristle King | Gristle Capo (/pages/capo-offer) |
| Gristle King | Lil Heat | Gristle Capo (/pages/capo-offer) |
| Loaded Pickguard | Gristle King | Gristle Capo (/pages/capo-offer) |
| Kochness Monster | Gristle-Tone Pickup Set | Lil Heat (/pages/lil-heat-offer) |
| Gristle ST / Supreme | none (collector tier) | none |

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
