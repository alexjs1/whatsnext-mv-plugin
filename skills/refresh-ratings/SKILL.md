---
name: refresh-ratings
description: Refresh the Google star ratings for every place in the What's Next MV guide. Use whenever the user types /whatsnext:refresh-ratings or asks to "refresh the ratings", "update the Google ratings", "pull the latest reviews", "run the weekly ratings update", or similar. Runs the ratings script, regenerates the ratings data file, reports what changed, and publishes. Intended as a weekly job.
---

# Refresh Google ratings

Ratings are baked into the app weekly, so the app never calls Google at runtime (visitor traffic costs nothing) and Google's terms are met (refreshed regularly, shown with a "Ratings from Google" credit). This skill is that weekly refresh.

## Project root

```
$HOME/Documents/mv-guide
```

The script `scripts/fetch-ratings.mjs` pulls each place's star rating and review count and rewrites `src/data/ratings.ts` (a map keyed by place id). The app ranks lists and the concierge by a review-count-weighted score computed in `src/lib/rating.ts`.

## Step 1 — Key

The script reads `GOOGLE_KEY` from the environment. It is **not** stored in this plugin or committed anywhere public. Source it from the app's `.env` (create `$HOME/Documents/mv-guide/.env` with `GOOGLE_KEY=...` once if it doesn't exist; it's gitignored), or export it for the run. If the key is missing the script exits with a message.

Note: the key has the **Places API** enabled but **not** the Geocoding API.

## Step 2 — Capture the before state

So you can report what changed:

```
cp "$HOME/Documents/mv-guide/src/data/ratings.ts" /tmp/ratings.before.ts
```

## Step 3 — Run the refresh

```
cd "$HOME/Documents/mv-guide" && GOOGLE_KEY="$GOOGLE_KEY" node scripts/fetch-ratings.mjs
```

It fetches all ~370 places (about three to four minutes, one call each, well within the free tier) and rewrites `src/data/ratings.ts`, printing how many of the total came back rated. Places with no Google rating are simply absent from the map and show no badge.

## Step 4 — Report what moved

Diff old vs. new and surface the interesting changes: places that just earned their first rating (newly added ones get picked up here), any large swings, and any that dropped out (a listing removed on Google's side).

```
diff /tmp/ratings.before.ts "$HOME/Documents/mv-guide/src/data/ratings.ts" | head -40
```

## Step 5 — Publish

Run `/whatsnext:publish`. This edits only `src/data/ratings.ts`, so `tsc` + preview spot-check (any list still ranks correctly, a rating shows) is enough. Commit message: note it's the weekly ratings refresh and the rated count.
