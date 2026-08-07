---
name: feature-place
description: Feature a place in the What's Next MV guide with the full paid package at once. Use whenever the user types /whatsnext:feature-place or asks to "feature a place", "make it a featured business", "give it the full featured package", "set up a featured listing", or the reverse ("un-feature it", "remove the featured listing"). Applies all three perks together — top ranking, the gold map star, and (optionally) a custom description — then publishes. Orchestrates the three single-purpose feature skills.
---

# Feature a place (full package)

The one-shot version of the three featured perks. Applies them together so a paying business gets the whole treatment in a single pass:

1. **Top ranking** — `priority: true` (leads its category in the Guide and concierge; shows the gold "Featured" tag).
2. **Gold map star** — `starMarker: true` (a distinctive, always-visible star on the map).
3. **Custom description** — the business's own copy (optional; only if provided).

> Perks 1 and 2 are **base-app only**: private-label editions hide the Featured
> section, the pill and the map star, so a customer's guests never see WNMV's
> sponsors inside the customer's own app. The place still appears there,
> unbadged, and the custom description still applies. See
> `/whatsnext:private-label`.

## Project root

```
$HOME/Documents/mv-guide
```

## Step 1 — Confirm what to apply

Identify the place. Ask whether they're providing a **custom description**; if so, get the text. If not, keep the existing description (ranking + star still apply).

## Step 2 — Find the record

```
grep -rn "name: '<place name>'" "$HOME/Documents/mv-guide/src/data/"
```

## Step 3 — Apply the package

On that record:
- Add `priority: true, starMarker: true,` (a clean spot is just after the `id/name/category` line).
- If a custom description was provided, replace the `description` value with it. **Cap it at 250 characters**, front-load the first ~90 as the teaser, and use curly apostrophes/quotes (never `String.replace` with a `$`). See `/whatsnext:customize-description` for the description rules.

**To un-feature:** remove `priority: true` and `starMarker: true` from the record (and revert the description if they want the editorial copy back).

## Step 4 — Publish

Run `/whatsnext:publish`, then verify all three in the preview:
- **Guide** → the place leads its category with the gold "Featured" tag.
- **Map** → a gold star, visible even amid clusters.
- The **detail page / row** shows the custom description (if set).
