---
name: offer
description: Set up, change, or cancel the offer to past guests in a private-label (Pocket Concierge) edition of What's Next MV — for example a January invitation to rebook next summer at a returning-guest rate — and send the phone notification for it or list who opted in. Use whenever the user types /whatsnext:offer or asks to "add an offer", "set up a returning-guest offer", "run a winter promotion for <hotel>", "change the discount", "extend the booking deadline", "what offer is live", "cancel the offer", "send the notification", "push the offer to guests", "who opted in", "export the opt-ins for <hotel>", or similar. Interview-style: one offer active at a time; asks what the offer is and when it applies, writes the record, verifies in the preview, and publishes. Offers switch off by themselves on their book-by date.
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
- **Notified.** Guests opt in on the front door ("Get next-season offers from <hotel>",
  optional name and email). Each edition's opt-ins live on its own Netlify site. The
  notification is sent on request (Step 6) and opens the app at `?offers=1`. Reach is
  installs × opt-ins, so the property's own email should carry the same link.

## Step 1 — The one opening question

Identify the edition (ask only if the user did not name it; list the slugs with
`grep -n "  slug: '" "$HOME/Documents/mv-guide/src/data/partners.ts"`). Then read its
`offers:` block and work out the status of each entry against today
(`date +%F`): **scheduled** if today < `showFrom`, **expired** if today > `bookBy`,
otherwise **live**. An offer that is live or scheduled is the *current* offer.

Ask exactly one question, and nothing before it — no status dump, no field list:

- **If there is a current offer:** "The current offer is *<headline>* (<tiers>, book by
  <date>, live now / starts <date>). Do you want to **change** it, **cancel** it,
  **set up a new offer** to replace it, **send the notification** for it, or see
  **who opted in**?"
- **If there is none:** offer only **set up an offer** or **who opted in** (there is
  nothing to change, cancel or send). If the user already said which, skip the question.

Sending and the opt-in list are Steps 6 and 7; they need no code or data change.

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
e.g. `https://saltwind-concierge.netlify.app/?offers=1`. Offer to send the
notification now (Step 6) if the offer is live.

## Step 6 — Send the notification

The token lives in the app's local `.env`: `PUSH_ADMIN_TOKEN_<SLUG>` (upper-case
slug, e.g. `PUSH_ADMIN_TOKEN_SALTWIND`) for a site with its own token, otherwise the
shared `PUSH_ADMIN_TOKEN` (the four push variables set once at Netlify team level,
inherited by every site — the normal arrangement). Use the per-site one if both exist.
If neither is there, the edition's push is not set up — see `/whatsnext:private-label`
Step 4b — and stop here. In the commands below, `$TOKEN` means whichever applies.

1. Draft the notification from the live offer and confirm it with the user:
   **title** = the property's name (≤ 80 chars), **body** = the offer's `summary`
   (≤ 240 chars). The link is `/?offers=1` unless they say otherwise.
2. Count the audience first with a dry run, and tell the user the number:
   ```
   set -a; source "$HOME/Documents/mv-guide/.env"; set +a
   curl -s -X POST "https://<slug>-concierge.netlify.app/.netlify/functions/push-send" \
     -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
     -d '{"title":"<title>","body":"<body>","url":"/?offers=1","dryRun":true}'
   ```
3. **Ask for an explicit go** — this reaches real phones and cannot be recalled — then
   run the same call without `dryRun`. Report `sent`, `failed` and `pruned`
   (subscriptions whose phone no longer exists; they are removed automatically).

Never send twice for the same offer unless the user asks; the send history is in the
opt-in report (`?sends=1`).

## Step 7 — Who opted in

Same token. JSON, or a CSV the owner can open:

```
set -a; source "$HOME/Documents/mv-guide/.env"; set +a
curl -s "https://<slug>-concierge.netlify.app/.netlify/functions/push-subscribers?format=csv" \
  -H "Authorization: Bearer $TOKEN" -o "<property> opt-ins $(date +%F).csv"
```

Save it where the user asks (the property's folder in Google Drive is the usual place)
and summarise: how many opted in, how many left a name or email, by platform. Names
and emails are the guests' personal data: hand the file to the property, do not paste
the list into chat.
