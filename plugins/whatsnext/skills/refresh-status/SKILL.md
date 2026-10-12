---
name: refresh-status
description: Refresh the open/closed status of every place in the What's Next MV guide from Google Places. Use whenever the user types /whatsnext:refresh-status or asks to "refresh the status", "check which places are closed", "update the closed-for-the-season labels", "run the closure check", "is anything temporarily closed", or similar. Runs the status script, regenerates the status data file, reports which places flipped to or from temporarily closed, flags anything Google says has closed permanently, and publishes. Intended as a weekly job.
---

# Refresh open/closed status

Seasonal businesses close for months at a time, and a guide that sends a visitor to a dark storefront loses trust fast. Google keeps a per-listing `business_status` (OPERATIONAL, CLOSED_TEMPORARILY, CLOSED_PERMANENTLY), and this skill pulls it for every place once a week. The app shows a red **Temporarily closed** pill on the town line of the Guide row, the map card and the place page; the AI concierge is told about the closure too. Permanent closures are never badged: they are listed here so a human can decide to remove the place.

Ratings are a different matter. Google's terms don't allow their star ratings to be stored or ranked on, and the old ratings refresh was deleted in Aug 2026 (see the Google Maps Platform terms section of `DEVELOPMENT.md`). Open/closed status is a separate decision Alex made in Oct 2026, recorded in the same section. Do not re-introduce ratings through this skill.

## Project root

```
$HOME/Documents/mv-guide
```

The script `scripts/fetch-status.mjs` makes one Find Place call per place (~665) and rewrites `src/data/status.ts`, a map keyed by place id that holds **only** the closed places. An open place is simply absent. The script trusts a Google match only when it is on the island, within 500 m of our pin, and shares the place's name; anything weaker is ignored (listed in the report, never written), so a wrong business can't mark an open place closed.

## Step 1 — Key

The script reads `GOOGLE_KEY` from `.env` in the project root (gitignored, never committed). If the key is missing the script exits with a message. The key has the classic Places API enabled.

## Step 2 — Capture the before state

So you can report what changed:

```
cp "$HOME/Documents/mv-guide/src/data/status.ts" /tmp/status.before.ts
```

## Step 3 — Run the refresh

```
cd "$HOME/Documents/mv-guide" && node --env-file=.env scripts/fetch-status.mjs
```

About two to three minutes for ~665 places. It prints the counts (operational / temporarily closed / permanently closed / unknown / weak matches / not found), lists every closed place by name and town, and writes two files: `src/data/status.ts` (the map the app ships) and `scripts/out/status-report.json` (every place's raw result, gitignored, for digging into a surprise).

## Step 4 — Report what moved

Diff old vs. new and tell the user, grouped:

```
diff /tmp/status.before.ts "$HOME/Documents/mv-guide/src/data/status.ts"
```

- **Newly temporarily closed** — closed since the last run (seasonal closings in fall, the odd renovation).
- **Reopened** — dropped out of the map (seasonal reopenings in spring).
- **Permanently closed** — anything Google marks `CLOSED_PERMANENTLY`. The app shows nothing for these. Verify against a second source (the business's site, the Gazette or MV Times) and ask the user per place whether to remove it; a removal follows the `weekly-refresh` removal procedure and is noted in the commit. Google is sometimes wrong here (a listing merged or relocated), so never remove on Google's word alone.

If a place you'd expect to be closed isn't flagged, check `scripts/out/status-report.json` for its row: a `WEAK_MATCH` or `NOT_FOUND` result means Google didn't resolve it to the right listing, which usually points at a name or coordinate worth fixing in the data.

## Step 5 — Publish

Run `/whatsnext:publish`. This edits `src/data/status.ts` (plus any approved removals), so `tsc`, the data export, and a preview spot-check are enough: open one newly closed place and confirm the red pill sits on the town line in the Guide row and on its page. Commit message: note it's the weekly status refresh with the closed count, e.g. "Weekly status refresh: 41 temporarily closed (+6, −2)".
