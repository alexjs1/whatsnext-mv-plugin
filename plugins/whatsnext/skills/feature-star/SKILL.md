---
name: feature-star
description: Give a place a gold star on the What's Next MV map instead of the usual circle (a featured-listing perk). Use whenever the user types /whatsnext:feature-star or asks to "give a place a star on the map", "make it a star", "change its circle to a star", "add a star marker", or the reverse ("remove the star", "put it back to a circle"). Sets or clears the place's starMarker flag, then publishes.
---

# Give a place a gold star on the map

Sets the `starMarker` flag on a place. A star-marked place renders as a distinctive **gold star** on the map, in its own layer that is never clustered, so it stays visible at every zoom level (unlike the circles, which collapse into cluster bubbles).

## Project root

```
$HOME/Documents/mv-guide
```

The app already implements the gold star (web and native). This skill just toggles the flag on the place record.

## Step 1 — Find the place

```
grep -rn "name: '<place name>'" "$HOME/Documents/mv-guide/src/data/"
```

## Step 2 — Toggle the flag

- **To add the star:** add `starMarker: true,` to the record (alongside `priority: true` if it's already there).
- **To remove it:** delete `starMarker: true` from the record — it goes back to a clustered category-colored circle.

## Step 3 — Publish

Run `/whatsnext:publish`. In the preview, open the Map (whole-island view) and confirm the place shows a gold star that stays visible even where nearby points cluster into azure bubbles.

> The star is map-only. Top ranking and the "Featured" tag are a separate perk — see `/whatsnext:feature-to-top`.

> The star is also **base-app only**: private-label editions draw the place's
> ordinary category marker instead, so a customer's guests never see WNMV's
> sponsors inside the customer's own app (`showsFeaturedStar()` in
> `src/lib/partner.ts`). See `/whatsnext:private-label`.
