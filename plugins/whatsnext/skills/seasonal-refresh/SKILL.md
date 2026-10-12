---
name: seasonal-refresh
description: Roll the What's Next MV guide into a new year or season. Use whenever the user types /whatsnext:seasonal-refresh or asks to "update for next season", "roll the event dates", "refresh for the new year", "check what's stale", or at the start of a season. Updates event dates to the current year, re-checks seasonal open/close windows, and flags stale year references and "new for YYYY" language that's no longer new.
---

# Seasonal / yearly refresh

Content that was accurate last season goes stale: event dates roll over, places open and close for the season, and "new for 2026" stops being new. This skill sweeps for that.

## Project root

```
$HOME/Documents/mv-guide
```

## Step 1 — Event dates

Every entry in `src/data/events.ts` has a `when` (and often specific dates). For each, verify the **current** year's date against a real source (host site, town site, Vineyard Gazette / MV Times) and update it. Keep the list in chronological order after edits. Never guess a date; if one isn't published yet, use the month/pattern the source gives ("mid-June") rather than a fabricated day.

## Step 2 — Stale year references

Find hard-coded years in copy and judge each:

```
grep -rnE "\\b20(2[4-9]|3[0-9])\\b|new for 20|new 20" "$HOME/Documents/mv-guide/src/data/"
```

- A "new for 2026" place that's no longer new → soften the description.
- A dated fact ("renovated in 2025", "opened in 2026") → fine to keep if still true; fix if wrong.
- A one-off event breach/closure note ("a January 2026 storm...") → keep if still accurate, update if the situation changed.

## Step 3 — Seasonal windows

Spot-check `seasonal` fields on places whose season is time-sensitive (a place that changed hours, went year-round, or closed). This is judgment plus a quick source check, not a bulk edit. If a place has closed for good, remove it (and note it in the commit).

## Step 4 — Steamship Authority schedules

The SSA card on the Info tab lists its timetable by operating period (`schedules[]` on the `ssa` record in `src/data/transport.ts`). The rule (Alex, Oct 2026): **only confirmed periods, and not far out** — the current period plus the next one the SSA has actually published, never the proposed schedule from its autumn notice. Each seasonal pass drops the period that has ended and adds the next confirmed one, so in winter the card should run through the coming summer.

1. Open https://www.steamshipauthority.com/schedules and note the Martha's Vineyard periods listed (it shows the current and next). The PDFs live at `https://www-steamship-assets.s3.amazonaws.com/versioned_downloadable_schedules/path/mv<N>_-_<MDDYY>_to_<MDDYY>.pdf`.
2. Pull the text: `curl -sLo /tmp/ssa.pdf "<pdf url>" && pdftotext -layout /tmp/ssa.pdf -`. Check the fourteen core sailings still match `SSA_WH_TO_VH` / `SSA_VH_TO_WH` (they have been identical every period since 2025), which of them use Oak Bluffs (OB) in that period, and any extra summer trips.
3. Add a `FerrySchedule` entry (label, dates, ISO `from`/`to`, a one-sentence `summary` of what differs, departures) and remove any whose `to` has passed. For a summer period, list the core sailings and say in the summary that extra trips are layered on; do not transcribe the freight boats.
4. If the SSA raised fares, fix the `fares` line too.

## Step 5 — Open/closed status

A season boundary is when the most places flip between open and closed, so run `/whatsnext:refresh-status` as part of the seasonal pass. Its "reopened" list is also a good cross-check on Step 3: a place Google says is open again should not still carry a closed-for-the-season note in its copy. There is no ratings step (Google ratings were removed in Aug 2026 — see `DEVELOPMENT.md`); open/closed status is a separate, later decision recorded in the same section.

## Step 6 — Publish

Run `/whatsnext:publish`. In the preview, open the Info → Events segment and confirm the rolled dates read correctly in order.
