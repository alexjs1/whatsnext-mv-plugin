---
name: weekly-refresh
description: Run the full weekly maintenance cycle for the What's Next MV guide. Use whenever the user types /whatsnext:weekly-refresh or asks to "do the weekly update", "run the weekly refresh", "weekly guide maintenance", "capture new openings and price changes", "what changed on the island this week", or similar. Sweeps local sources for new, closed, and changed places and events since the last run, proposes adds and edits for approval, refreshes each place's open/closed status from Google, and publishes once.
---

# Weekly refresh — the whole cycle

The guide is kept current on a weekly cadence in summer (less often off-season, per the in-app "About this guide" note). This skill is that cycle: it catches what changed on the island (new restaurants, closings, relocations, price or season changes, new events) and refreshes every place's open/closed status, in one consolidated pass ending in a single publish.

It orchestrates the focused skills rather than duplicating them: `add-place`, `add-event`, `refresh-status`, and `publish`.

## Project root

```
$HOME/Documents/mv-guide
```

## Two modes

- **Interactive** (someone typed `/whatsnext:weekly-refresh`): the steps below, with the user approving each change in chat.
- **Automated** (the Thursday 11:00 UTC routine, unattended): nobody is there to approve, so **nothing is written to the guide**. Do Steps 1, 2, 5, 5b, and 7 below for the research, skip Steps 3, 4, 6, and 8, and follow `review/README.md` in the app repo exactly: every finding becomes a proposal in the review queue, and the queue goes to alexjs1@gmail.com as a spreadsheet he returns through `/whatsnext:apply-review`. The open/closed refresh (Step 6) has already been done by the GitHub Action an hour earlier; read its result rather than re-running it (the routine has no Places key). In the cloud, the path is the cloned repo, not `$HOME/Documents/mv-guide`.

## Order matters

Do the content sweep and apply adds **before** the status fetch, so any place added this week gets its open/closed check in the same run, and everything ships in one publish.

## Step 1 — Figure out the window

Find when the guide was last refreshed so the sweep only looks at what's new:

```
git -C "$HOME/Documents/mv-guide" log -1 --format=%cs -- src/data/places.ts
git -C "$HOME/Documents/mv-guide" log --oneline -8
```

Use that date as "since"; if it's been longer, widen the window accordingly.

## Step 2 — Sweep local sources for changes

Search the Vineyard's own press and listings for anything that happened since the window date. Good sources: everything in `netlify/functions/trusted-sources.json` (the Vineyard Gazette and MV Times news and calendars, MV Online, gomarthasvineyard.com, mvacay.com, and the rest), thisweekonmv.com, and the businesses' own sites. Look for:

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

## Step 5 — Sweep the what's-on sources for gaps

There is no ratings step any more — Google ratings were removed from the guide in
Aug 2026 (see `DEVELOPMENT.md` → "Google Maps Platform terms"; do not reinstate
a ratings pull). Use the slot for coverage instead: skim MV Online
(https://mvol.com/events/, which also carries a business directory) and the MV
Times' Things to Do (https://www.mvtimes.com/things-to-do). A business or event
they carry that the guide does not is a hole worth filling — feed anything found
back through Step 3. MVacay (https://www.mvacay.com) is a third research source,
but is never cited to guests.

## Step 5b — Discover new sources for the concierge

The AI concierge searches the web live, but only the sites in
`netlify/functions/trusted-sources.json` (plus the websites of the guide places
it pulls up for a question). That list is what lets it beat a general AI: it
reads the island's own papers, calendars, town and transit sites instead of the
open web. Keep it growing.

While doing Steps 2 and 5, note every site that carried useful, current Martha's
Vineyard information and is **not** already in the list. Then run one or two
deliberate searches for new ones (for example "Martha's Vineyard events
calendar", "Martha's Vineyard restaurant openings", "Edgartown town notices",
"Martha's Vineyard off-season hours"). A site qualifies when it is:

- **About the island**, mostly or wholly (not a national travel site with one page on it).
- **Current**: updated within the last three months.
- **A primary or well-kept source**: a paper, a calendar, a town or county office, a transit operator, a conservation group, a venue, a festival, or a long-running local guide.
- **Not** a social network, a link hub, a search engine, or a content farm (thin, AI-written, or copied pages).

Also drop any listed site that has died or gone stale for six months.

Each addition is one entry: `domain` (no `www.`), `name` (how a guest would hear
it credited, e.g. "the MV Times"), `covers`, and `cite` (`false` for research-only
sources such as MVacay, which the concierge may read but never names). Set
`updated` to today. OpenAI allows 100 search domains per question and each
question also adds its places' own websites, so keep the list under about 60;
past that, replace the weakest source rather than adding. Add each new source to
`WNMV_Links.md` too.

Interactive: put the proposed additions and removals through Step 3's yes/no
with the rest. Automated: they become review-queue proposals like any other
finding.

## Step 6 — Refresh open/closed status

Run `/whatsnext:refresh-status` steps 2–4 (this also checks the places you just added). It rewrites `src/data/status.ts`, the map behind the app's red **Temporarily closed** pill, and reports what flipped: newly closed, reopened, and anything Google marks permanently closed. Treat a permanent closure as a candidate **removal** and put it through the same per-item yes/no as Step 3 before writing anything; never remove on Google's word alone.

## Step 7 — VTA service alerts (quick check)

The full bus-schedule pass belongs to `/whatsnext:seasonal-refresh` (its Step 6),
but two things can change any week, so skim <https://www.vineyardtransit.com/news>:

- **The fare-free end date.** The Info tab states it outright in
  `TRANSPORT.vta.fares`, so a changed or lapsed date makes the app wrong.
- **New or suspended service.** Anything newly announced belongs in the data.

If the news page shows a new timetable period has started, stop treating this as
a quick check and run `/whatsnext:seasonal-refresh` instead. Use the built-in
browser; the site 403s WebFetch.

## Step 8 — Publish once

Run `/whatsnext:publish`. The commit message should summarize the week: N added, N removed/edited, N temporarily closed. Verify in the preview that a couple of the changes render and there are no console errors.

## Off-season

Outside summer, run this less often (the in-app note says as much). At a season boundary, use `/whatsnext:seasonal-refresh` instead, which rolls event dates and re-checks seasonal windows in bulk.
