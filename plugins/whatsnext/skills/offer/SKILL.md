---
name: offer
description: Set up, change, or cancel the offer to past guests in a private-label (Pocket Concierge) edition of What's Next MV — for example a January invitation to rebook next summer at a returning-guest rate. Use whenever the user types /whatsnext:offer or asks to "add an offer", "set up a returning-guest offer", "run a winter promotion for <hotel>", "change the discount", "extend the booking deadline", "what offer is live", "cancel the offer", "turn off the promotion", or similar. Interview-style: one offer active at a time; asks what the offer is and when it applies, writes the record, verifies in the preview, and publishes. Offers switch off by themselves on their book-by date.
---

# The offer to past guests

A partner edition can carry **one active offer** to people who stayed before, shown as
a card on the edition's front door and as a panel of its own. This skill is the only
way it gets written. It is an **interview**: one question first, then the details,
confirm, write, verify, ship.

## Project root

```
$HOME/Documents/mv-guide
```

The offer lives on the partner's record in `src/data/partners.ts` (the `offers` array
on a `Partner`; type `PartnerOffer` in the same file). Date logic: `src/lib/offers.ts`.
UI: `src/components/PartnerOffers.tsx`. **Nothing in this skill touches code** — an
offer is data, and the same components render it for every edition in that edition's
own colors and typeface.

## How an offer behaves (tell the user if they ask)

- **Live window.** It shows from `showFrom` (or immediately if unset) through the END
  of its `bookBy` day, in the guest's local time. The morning after `bookBy` it is gone
  from the app on its own — no deploy, nothing to remember. `bookBy` is required.
- **One at a time.** Only one offer is ever active. The `offers` array is a schedule:
  a next offer may be queued with a `showFrom` after the current one's `bookBy`, and
  windows must never overlap. Expired entries can be deleted or left as history.
- **Where it appears.** A card above the menu buttons on the front door, and a panel
  (via the card, or straight from a link ending in `?offers=1`, which is what a
  notification or the hotel's own email points at). Never on the Info tab, never in
  the public What's Next MV.
- **Measured.** `offer_open`, `offer_view` and `offer_book` land in the analytics log,
  tagged with the edition and the offer id.

## Step 1 — The one opening question

Identify the edition (ask only if the user did not name it; list the slugs with
`grep -n "  slug: '" "$HOME/Documents/mv-guide/src/data/partners.ts"`). Then read its
`offers:` block and work out the status of each entry against today
(`date +%F`): **scheduled** if today < `showFrom`, **expired** if today > `bookBy`,
otherwise **live**. An offer that is live or scheduled is the *current* offer.

Ask exactly one question, and nothing before it — no status dump, no field list:

- **If there is a current offer:** "The current offer is *<headline>* (<tiers>, book by
  <date>, live now / starts <date>). Do you want to **change** it, **cancel** it, or
  **set up a new offer** to replace it?"
- **If there is none:** do not ask. Say "No offer is set up for <edition>; let's create
  one." and go straight to Step 2.

Then:

- **Change** → show the current values once, ask what to change, apply Step 2 to those
  fields only, keep the id.
- **Cancel** → confirm, then delete the entry (Step 3). Done; publish.
- **New** → Step 2 in full. It **replaces** the current one: delete the current entry
  unless the user wants it to run until it ends and the new one to start afterwards —
  then keep both and set the new `showFrom` to the day after the current `bookBy`.

## Step 2 — The interview

Ask in plain language, a few questions at a time, and do not invent anything the user
did not say. Confirm the whole offer back before writing. You need:

1. **The offer in one sentence** — becomes `headline` (short, e.g. "Come back to the
   harbor next summer") and `summary` (one line with the numbers and deadline, under
   ~110 characters; it doubles as the notification text).
2. **The discount tiers** — one or more of `{ amount, applies, note? }`, e.g.
   `20%` / `off June & September` / `Quieter beaches, warm water.` Different rates for
   different months are tiers of the ONE offer.
3. **When it applies** — the stay window in words (`stayDates`, e.g.
   "June 1 – September 30, 2027").
4. **When to show it and when it ends** — `showFrom` (optional, ISO `YYYY-MM-DD`) and
   `bookBy` (required, ISO). Say the behaviour back: "it appears on <date> and
   disappears the morning after <bookBy>". If they want it up now, omit `showFrom`.
   If another entry is kept, the windows must not overlap — refuse and explain.
5. **How to book** — `bookingUrl` (ideally their booking engine with the promo code
   pre-applied), `promoCode`, and the button wording (`bookingLabel`, default
   "Book now").
6. **Fine print** — `terms` (minimum stay, exclusions, availability), optional
   `footnote`, `intro` (the thank-you paragraph) and `eyebrow` (default "For our
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

## Step 4 — Verify

```
cd "$HOME/Documents/mv-guide" && npx tsc --noEmit
```

Then in the preview (`preview_start`, config `vineyard-guide-web`), load
`/?partner=<slug>` and confirm:

- the card shows above the menu with the right headline, summary and "Book by" date
  (or does NOT show, for a scheduled or cancelled offer);
- tapping it opens the panel; tiers, dates, promo code, terms and the book button read
  correctly, and "‹ Menu" returns to the menu;
- `/?partner=<slug>&offers=1` opens straight onto the panel;
- `/?partner=none` still shows the base app with no card.

Then run `/whatsnext:publish`. Mention the edition and offer id in the commit message.

## Step 5 — Tell the user what happens next

State the dates back one more time ("live now, gone the morning of March 2"), and give
them the link for the notification or email: the edition's address with `?offers=1`,
e.g. `https://saltwind-concierge.netlify.app/?offers=1`.
