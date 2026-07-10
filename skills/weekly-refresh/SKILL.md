---
name: weekly-refresh
description: Run the full weekly maintenance cycle for the What's Next MV guide. Use whenever the user types /whatsnext:weekly-refresh or asks to "do the weekly update", "run the weekly refresh", "weekly guide maintenance", "capture new openings and price changes", "what changed on the island this week", or similar. Sweeps local sources for new, closed, and changed places and events since the last run, proposes adds and edits for approval, refreshes Google ratings, and publishes once.
---

# Weekly refresh — the whole cycle

The guide is kept current on a weekly cadence in summer (less often off-season, per the in-app "About this guide" note). This skill is that cycle: it catches what changed on the island (new restaurants, closings, relocations, price or season changes, new events) and refreshes ratings, in one consolidated pass ending in a single publish.

It orchestrates the focused skills rather than duplicating them: `add-place`, `add-event`, `refresh-ratings`, and `publish`.

## Project root

```
$HOME/Documents/mv-guide
```

## Order matters

Do the content sweep and apply adds **before** the ratings fetch, so any place added this week gets its Google rating in the same run.

## Step 1 — Figure out the window

Find when the guide was last refreshed so the sweep only looks at what's new:

```
git -C "$HOME/Documents/mv-guide" log -1 --format=%cs -- src/data/ratings.ts
git -C "$HOME/Documents/mv-guide" log --oneline -8
```

Use that date as "since"; if it's been longer, widen the window accordingly.

## Step 2 — Sweep local sources for changes

Search the Vineyard's own press and listings for anything that happened since the window date. Good sources: the Vineyard Gazette and MV Times (news + calendars), thisweekonmv.com, mvacay.com, gomarthasvineyard.com, and the businesses' own sites. Look for:

- **New openings** — restaurants, shops, markets, lodging that opened or are about to.
- **Closings / relocations** — places that closed, moved, or rebranded (a "former X space" now something else).
- **Price or season changes** — a spot that went year-round, changed tiers, or shifted hours/days.
- **New or newly-dated events** — additions to the calendar, or dates now announced.

Then cross-check against what's already in the guide so you only surface real deltas:

```
cd "$HOME/Documents/mv-guide" && node --input-type=module -e "import('./src/data/places.ts').then(m=>{const names=m.PLACES.map(p=>p.name.toLowerCase());console.log(m.PLACES.length+' places');globalThis.__names=names;})"
```

Compare found names against the guide: not present → candidate **add**; reported closed but present → candidate **remove**; changed detail → candidate **edit**.

## Step 3 — Review with the user (do not auto-change)

Present a short, grouped list: proposed **adds**, **removals**, and **edits** (with the source for each). Accuracy is the whole point of this guide, so get the user's yes/no per item before writing anything. Note anything you could not verify rather than guessing.

## Step 4 — Apply the approved changes

- **Adds**: follow the `add-place` procedure for each (verify, geocode via Places `findplacefromtext`, right file/category, schema-correct record). New events follow `add-event`.
- **Removals**: delete the record (grep the id, remove the object), and note it in the eventual commit.
- **Edits**: update the specific fields (tier, seasonal, description). Use curly apostrophes in strings; never `String.replace` with a `$`-containing replacement.

Do not run `publish` yet — batch everything into one publish at the end.

## Step 5 — Refresh ratings

Now run the ratings fetch (this also picks up the places you just added). The key is in `$HOME/Documents/mv-guide/.env` as `GOOGLE_KEY`.

```
cd "$HOME/Documents/mv-guide" && GOOGLE_KEY="$(grep -E '^GOOGLE_KEY=' .env | cut -d= -f2)" node scripts/fetch-ratings.mjs
```

Skim the change: newly-rated places, notable swings, any dropped listing (often a real-world closing worth acting on).

## Step 6 — Publish once

Run `/whatsnext:publish`. The commit message should summarize the week: N added, N removed/edited, ratings refreshed. Verify in the preview that a couple of the changes render and there are no console errors.

## Off-season

Outside summer, run this less often (the in-app note says as much). At a season boundary, use `/whatsnext:seasonal-refresh` instead, which rolls event dates and re-checks seasonal windows in bulk.
