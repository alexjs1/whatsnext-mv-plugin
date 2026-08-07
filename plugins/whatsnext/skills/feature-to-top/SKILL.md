---
name: feature-to-top
description: Make a place show first in its category in the What's Next MV guide (a featured-listing perk). Use whenever the user types /whatsnext:feature-to-top or asks to "bring a place to the top", "feature it first", "make it show first", "give it top ranking", "prioritize it in results", or the reverse ("stop featuring it at the top", "remove top ranking"). Sets or clears the place's priority flag, then publishes.
---

# Feature a place to the top of results

Sets the `priority` flag on a place. A priority place gets the paid position: it
appears in its own **Featured** section at the top of the Guide, still appears in
its own category below (a paid position, not a replacement for the organic one),
leads the concierge's answers for that category, and carries a gold "Featured"
tag on the row, card, and detail page wherever it is seen.

Since Google ratings were removed in Aug 2026 this is the ONLY way to reach the
top of a list — everything else is alphabetical — which is what makes the
sponsorship worth selling. The Featured section is built only when a sponsored
place matches the current filters, so there is never an empty header.

> **The base app only.** Private-label editions hide the Featured section, the
> pill and the map star: a customer's guests are not shown WNMV's sponsors
> inside the customer's own app. The place still appears there, unbadged. Say so
> when quoting reach — a sponsor's placement is seen on whatsnextmv.com, not in
> the hotel and association editions. See `/whatsnext:private-label`.

## Project root

```
$HOME/Documents/mv-guide
```

The app already implements the behavior (the Featured section + priority-first sorting + the Featured tag). This skill just toggles the flag on the place record.

## Step 1 — Find the place

```
grep -rn "name: '<place name>'" "$HOME/Documents/mv-guide/src/data/"
```

## Step 2 — Toggle the flag

- **To feature:** add `priority: true,` to the record (a good spot is right after the `id/name/category` line). If the line already has other flags like `starMarker: true`, add alongside them.
- **To un-feature:** remove the `priority: true` from the record (or the user may want the whole featured package removed — see `/whatsnext:feature-place`).

Keep TS-string rules in mind for any nearby edits (curly apostrophes; never `String.replace` with a `$`).

## Step 3 — Publish

Run `/whatsnext:publish`. In the preview, open the Guide, filter to the place's category, and confirm it now appears in the gold **Featured** section at the top of the list with the "Featured" tag (and, for a removal, that the Featured section no longer carries it — and disappears entirely if it was the only one — and the place sits in its normal alphabetical spot with no tag).

> Note: "first" means first **within its category** (a featured restaurant leads the restaurant list, not the whole guide). The gold map star is a separate perk — see `/whatsnext:feature-star`.
