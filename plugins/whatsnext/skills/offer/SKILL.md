---
name: offer
description: Set up, change, list, or turn off an offer to past guests in a private-label (Pocket Concierge) edition of What's Next MV — for example a January invitation to rebook next summer at a returning-guest rate. Use whenever the user types /whatsnext:offer or asks to "add an offer", "set up a returning-guest offer", "run a winter promotion for <hotel>", "change the discount", "extend the booking deadline", "what offers are live", "turn off the offer", "remove the promotion", or similar. Interview-style: asks which edition, what the offer is, and when it applies (several offers for different periods are fine), writes one record, verifies in the preview, and publishes. Offers switch off by themselves on their book-by date.
---

# Offers to past guests

A partner edition can carry **offers**: promotions to people who stayed before, shown
as a card on the edition's front door and as a panel of their own. This skill is the
only way they get written. It is an **interview**: ask, confirm, write, verify, ship.

## Project root

```
$HOME/Documents/mv-guide
```

Offers live on the partner's record in `src/data/partners.ts` (the `offers` array on a
`Partner`; type `PartnerOffer` in the same file). The date logic is in
`src/lib/offers.ts`; the UI is `src/components/PartnerOffers.tsx`. **Nothing in this
skill touches code** — an offer is data, and the same components render it for every
edition in that edition's own colors and typeface.

## How an offer behaves (tell the user this if they ask)

- **Live window.** An offer shows from `showFrom` (or immediately if unset) through the
  END of its `bookBy` day, in the guest's local time. The morning after `bookBy` it is
  gone from the app on its own — no deploy, nothing to remember. `bookBy` is required.
- **Several at once.** Each offer in the list is independent. Two live offers both show
  (the card leads with the first; the panel lists both). Use this for different periods,
  e.g. a spring offer and a summer offer, or an early-bird deal that `showFrom`/`bookBy`
  hands over to a later one.
- **Where it appears.** A card above the menu buttons on the front door, and a panel
  (via the card, or straight from a link ending in `?offers=1`, which is what a
  notification or the hotel's own email will point at). Not on the Info tab.
- **Measured.** `offer_open`, `offer_view` (with the offer ids) and `offer_book` land in
  the analytics log, tagged with the edition.

## Step 1 — Which edition, and what do they want to do?

List the editions and ask which one, unless the user named it:

```
grep -n "^const [A-Z_]* : Partner = {\|^const [A-Z_]*: Partner = {\|  slug: '" "$HOME/Documents/mv-guide/src/data/partners.ts"
```

Then establish the action: **add** a new offer, **change** an existing one, **list**
what is live, or **turn off** one. For list / change / turn off, show the edition's
current offers first (read its `offers:` block) with each one's status computed
against today:

```
TODAY=$(date +%F); echo "today $TODAY"
grep -n "id: '\|showFrom: '\|bookBy: '" "$HOME/Documents/mv-guide/src/data/partners.ts"
```

An offer is **scheduled** if today < `showFrom`, **expired** if today > `bookBy`,
otherwise **live**.

## Step 2 — The interview (for add or change)

Ask in plain language, a few questions at a time, and do not invent anything the user
did not say. Confirm the whole offer back before writing. You need:

1. **The offer in one sentence** — becomes `headline` (short, e.g. "Come back to the
   harbor next summer") and `summary` (one line with the numbers and deadline, under
   ~110 characters; it doubles as the notification text).
2. **The discount tiers** — one or more of `{ amount, applies, note? }`, e.g.
   `20%` / `off June & September` / `Quieter beaches, warm water.` Ask "is it one rate,
   or does it differ by period?" — different rates for different months are tiers of ONE
   offer; different booking windows are SEPARATE offers.
3. **When it applies** — the stay window in words (`stayDates`, e.g.
   "June 1 – September 30, 2027").
4. **When to show it and when it ends** — `showFrom` (optional, ISO `YYYY-MM-DD`) and
   `bookBy` (required, ISO). Say back the behaviour: "it will appear on <date> and
   disappear the morning after <bookBy>". If they want it up now, omit `showFrom`.
5. **How to book** — `bookingUrl` (ideally their booking engine with the promo code
   pre-applied), `promoCode`, and the button wording (`bookingLabel`, default
   "Book now").
6. **Fine print** — `terms` (minimum stay, exclusions, availability) and an optional
   `footnote` and `intro` (the thank-you paragraph) and `eyebrow` (default "For our
   returning guests").
7. **An id** — propose one from the season, e.g. `summer-2027`. Lowercase, no spaces,
   unique within the edition, never changed afterwards (it is the analytics key).

If the user gives a date like "March 1" with no year, pick the next occurrence and
confirm it.

## Step 3 — Write the record

Add, replace, or delete the entry in the edition's `offers: [ ... ]` array (create the
array after `preferredPlaceIds` if the edition has none). Follow the shape in the
`PartnerOffer` type and the Saltwind sample. Use curly apostrophes (’) and curly quotes
(“ ”) in strings so the single-quote delimiter never needs escaping. Keep comments to
one line saying what the offer is and that `/whatsnext:offer` manages it.

**Turning an offer off early:** delete its entry (or, to keep the history, set `bookBy`
to yesterday). Either way it is gone on the next deploy.

## Step 4 — Verify

Run the standard checks and look at it:

```
cd "$HOME/Documents/mv-guide" && npx tsc --noEmit
```

Then in the preview (`preview_start`, config `vineyard-guide-web`), load
`/?partner=<slug>` and confirm:

- the card shows above the menu with the right headline, summary and "Book by" date
  (or does NOT show, for a scheduled or turned-off offer);
- tapping it opens the panel; tiers, dates, promo code, terms and the book button read
  correctly, and "‹ Menu" returns to the menu;
- `/?partner=<slug>&offers=1` opens straight onto the panel;
- `/?partner=none` still shows the base app with no card.

Then run `/whatsnext:publish`. Mention in the commit message which edition and offer id.

## Step 5 — Tell the user what happens next

State the dates back one more time ("live now, gone the morning of March 2"), and give
them the link for the notification or email: the edition's address with `?offers=1`,
e.g. `https://saltwind-concierge.netlify.app/?offers=1`.
