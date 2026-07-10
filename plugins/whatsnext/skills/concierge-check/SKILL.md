---
name: concierge-check
description: Check the What's Next MV offline concierge (the Ask tab) for coverage gaps. Use whenever the user types /whatsnext:concierge-check or asks to "check the concierge", "test the Ask tab", "find gaps in the concierge", "why does asking X give a weak answer", or after adding a new category/data type. Runs representative questions, flags any that hit the generic fallback or return too few results, and adds a dedicated intent where one is missing.
---

# Concierge coverage check

The Ask tab is a rule-based engine in `src/lib/ask.ts`: ordered intents that match a question to bundled data. When no intent matches, it drops to a brittle generic keyword fallback that returns "Here is what fits from the guide:" and often too few (or wrong-town) results. This skill finds those gaps and closes them. Two real examples this caught in the past: "what hotels are in Edgartown" returning one result, and lodging price asks ignoring "cheap".

## Project root

```
$HOME/Documents/mv-guide
```

## Step 1 — Static coverage check

List the categories that exist in the data, then confirm each has a dedicated intent in `ask.ts`:

```
cd "$HOME/Documents/mv-guide"
node --input-type=module -e "import('./src/data/places.ts').then(m=>{const s=new Set(m.PLACES.map(p=>p.category));console.log([...s].sort().join('\n'))})"
grep -nE "category === '|=== 'restaurant'|listOf\(" src/lib/ask.ts | head -60
```

Any category with lots of places but no matching intent branch is a likely gap.

## Step 2 — Live battery (via the preview Ask tab)

Start the preview (`preview_start`, `vineyard-guide-web`, 8081), open `/ask`, and run a battery, reading each answer. Cover, per a couple of towns:

- Food: "where should I eat", "cheap eats in <town>", "best lobster roll", "breakfast", "dessert"
- Stay: "hotels in <town>", "budget places to stay", "luxury hotels"
- Do: "beaches for kids", "hikes", "rainy day", "things to do", "markets", "what's free"
- Info: "how do I get to <town> without a car", "events this week", "gas stations"

**Flag** any answer that: begins with "Here is what fits from the guide:" (the fallback), returns only one result where several exist, leaks the wrong town, or ignores a price/kid/town qualifier.

## Step 3 — Close the gap

For a confirmed gap, add a dedicated intent in `ask.ts`, following the existing patterns:

- Place it **before** the generic keyword fallback near the end.
- Reuse the shared helpers: `inTown(...)` to narrow by a named town, `listOf(...)` to format a rating-sorted list with the standard proviso, and `sortByRating`.
- For categories with price tiers (restaurants, lodging), degrade gracefully when a tier is absent in a town — filter to the requested tier, and if none exist, fall to the nearest tier with entries and say so, rather than silently showing everything.

## Step 4 — Publish

Run `/whatsnext:publish`, and in the preview re-run the exact questions that failed to confirm they now answer well.
