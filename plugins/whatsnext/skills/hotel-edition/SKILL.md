---
name: hotel-edition
description: Set up a complete private-label hotel edition of What's Next MV end to end in one flow — the branded "Mobile Concierge" edition AND its curated Pocket Concierge front door. Use whenever the user types /whatsnext:hotel-edition or asks to "set up a hotel edition", "onboard a hotel", "create the whole hotel version of the app", "build a hotel concierge for <property>", or "private-label a hotel and curate its concierge". Orchestrates the two single-purpose skills (private-label, then pocket-concierge) so you don't run them separately, and publishes once at the end.
---

# Set up a hotel edition (end to end)

A full hotel edition is two layers:

1. **The branded edition** (`/whatsnext:private-label`) — re-skins the app as the property's own concierge at `whatsnextmv.com/h/<slug>`: logo, colors, font, contact, the hotel Info tab, a Favorites tab, and lodging removal.
2. **The Pocket Concierge** (`/whatsnext:pocket-concierge`) — the curated front door shown after the splash: a lean menu (restaurants, shopping, landmarks, heritage, can't-miss) and a panel of accordion picks, plus an "or ask" box.

This skill runs both, in order, as one sitting, so the hotel is set up with a single command. **It does not replace the two skills** — it sequences them and publishes once, so read each of them for the detail of each step.

## Project root

```
$HOME/Documents/mv-guide
```

## Flow

1. **Confirm scope.** Get the property name and slug. If it's an update to an existing hotel, note which layer(s) to (re)do. If the user only wants the branded edition (no curated menu yet), run just `/whatsnext:private-label` and stop.

2. **Layer 1 — branded edition.** Follow `/whatsnext:private-label` **through writing the `Partner` record** (branding, contact, `infoSections`, `preferredPlaceIds`, logo) — but **do not publish yet**; the single publish happens at the end. Verify the branded surfaces in the preview at `/h/<slug>` (splash, header "Contact <hotel>" sheet, Info tab, Favorites tab, lodging removed).

3. **Layer 2 — Pocket Concierge.** Follow `/whatsnext:pocket-concierge` **through writing the `pocketConcierge` config** (curate nearby dining by meal/budget seeded from live ratings with walkability flags, shopping, landmarks, can't-miss, the heritage summary, and any bespoke `desc` copy). Keep its **approval gate**: present the curated draft and get sign-off before writing. Do not publish yet.

4. **Verify both together.** In the preview at `/h/<slug>`: the longer splash → the Pocket Concierge menu → each accordion; every pick opens its card; the "ask" box routes to the concierge; the Favorites/Info tabs and branding are right. Load the base app (`/`) to confirm it still shows the normal wizard. Run `npx tsc --noEmit`.

5. **Publish once.** Run `/whatsnext:publish`. Commit message names the hotel and both layers, e.g. "Add hotel edition for The Saltwind Inn (branding + Pocket Concierge)".

## Notes

- **Both layers are data-only** — one `Partner` record (with an optional `pocketConcierge` block) plus a logo. You should not touch components.
- **Single publish** avoids two deploys; the sub-skills each end in a publish when run alone, so here you defer that to step 5.
- To do just one layer later, run the sub-skill directly (`/whatsnext:private-label` or `/whatsnext:pocket-concierge`).
