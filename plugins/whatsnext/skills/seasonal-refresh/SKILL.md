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

## Step 4 — Ratings

Seasonal reopenings and closings are exactly what the seasonal pass is for; there is no ratings step (Google ratings were removed in Aug 2026 — see `DEVELOPMENT.md`).

## Step 5 — Publish

Run `/whatsnext:publish`. In the preview, open the Info → Events segment and confirm the rolled dates read correctly in order.
