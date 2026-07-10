---
name: customize-description
description: Set a custom description for a place in the What's Next MV guide (a featured-listing perk where the business writes its own copy). Use whenever the user types /whatsnext:customize-description or asks to "customize the description for a place", "write a custom description", "set the copy for a business", "use the business's own wording", or gives replacement text for a place's description. Enforces the 250-character cap, then publishes.
---

# Customize a place's description

One of the featured-listing perks: the business supplies its own description. This skill swaps a place's `description` for the provided text, within the guide's length limit.

## Project root

```
$HOME/Documents/mv-guide
```

Descriptions live on each place record in `src/data/*.ts` (keyed by `id`).

## Step 1 — Find the place and get the text

Identify the place (by name) and locate its record:

```
grep -rn "name: '<place name>'" "$HOME/Documents/mv-guide/src/data/"
```

Get the new description text from the user.

## Step 2 — Enforce the limits

- **Hard cap: 250 characters.** If the text is longer, do not truncate silently. Tell the user the count and offer a tightened version under 250 that keeps their meaning, or ask them to trim.
- Put the strongest, most identifying detail in the **first ~90 characters** — the map card and Guide row show only two lines (~90 chars) before truncating; the full text shows on the detail page and in concierge answers.
- One or two sentences reads best. This is app copy, not Island Analytics brand copy, so no special writing rules — just clear and factual, and honor what the business asked for.

## Step 3 — Write it

Replace the `description` value on that record. Use curly apostrophes (’) and curly quotes (“ ”) so the single-quote string delimiter never needs escaping; never `String.replace` with a `$`-containing replacement.

## Step 4 — Publish

Run `/whatsnext:publish`. In the preview, open the place (Guide row or detail page) and confirm the new copy reads correctly and isn't awkwardly clipped in the two-line row.
