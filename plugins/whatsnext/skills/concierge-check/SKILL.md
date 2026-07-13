---
name: concierge-check
description: Check the What's Next MV offline concierge (the Ask tab) for coverage gaps AND routing collisions. Use whenever the user types /whatsnext:concierge-check or asks to "check the concierge", "test the Ask tab", "run the concierge battery", "find gaps in the concierge", "why does asking X give the wrong answer", or after adding a new category/data type or editing intents. Runs the regression battery and representative questions, flags misroutes and fallbacks, and fixes the intent (a guard/reorder for a collision, a new intent for a gap).
---

# Concierge coverage & routing check

The Ask tab is a rule-based engine in `src/lib/ask.ts`: an ORDERED list of regex intents (first match wins) that map a question to bundled data. Two failure modes:

1. **Coverage gap** — no intent matches, so it drops to the generic fallback ("Here is what fits from the guide:") or the honest SORRY_TEXT. Past examples: "hotels in Edgartown" returning one result; lodging price asks ignoring "cheap".
2. **Routing collision** — a broad word in an EARLY intent steals a question meant for a later one. Past examples: "bike **trails**" → hiking; "movie th**eat**ers" → food; "raw **bar**" → nightlife; "**liquor** stores" → alcohol-rules. This is the fragility that grows with every new intent, so it is the primary thing this skill guards.

Standard (Alex): **a wrong answer is worse than the honest fallback.** SORRY_TEXT and OUT_OF_SCOPE_TEXT still SAY something, so an uncertain question should fall through to them, never be forced into a wrong intent.

## Project root

```
$HOME/Documents/mv-guide
```

## Step 0 — Run the regression battery (do this first, every time)

`scripts/concierge-battery.json` holds ~91 `{q, expect, notExpect}` cases (grown with every new category — shops, charters, sweets landed July 2026) (a question with a substring its answer must/must-not contain). Run it against the REAL router via the dev-only `window.__askLocal` hook:

1. `preview_start` (`vineyard-guide-web`, 8081), wait for the bundle, then navigate to `http://localhost:8081` so `__DEV__` is set and the hook is installed. Confirm `typeof window.__askLocal === 'function'`.
2. In one `javascript_tool` call, `fetch('/concierge-battery.json')` won't work (scripts/ isn't web-served) — instead read the JSON with the Read tool, inline its `cases` into the browser script, and for each call `window.__askLocal(q)` and assert `expect` is present and `notExpect` (if set) is absent, case-insensitive. Return the list of failures.
3. **Every failure is a misroute or a gap.** Fix per the steps below, then re-run until 0 failures. When you find a NEW misroute not in the battery, ADD a case for it so it can never regress. (Avoid accidental-substring assertions — e.g. never assert absence of "eat", which hides inside "theaters".)

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

## Step 3 — Fix it

**For a routing collision** (the battery's most common failure): the guilty intent is an EARLIER one whose regex matched a word that belongs to another domain. GUARD it — add a negative lookahead so it does NOT fire on the other domain's phrasing (e.g. hiking `&& !/\bbik(e|ing)\b|bicycl|cycling/`, the bars/alcohol intents `&& !/(raw|sushi|oyster|coffee|...) ?bar/`), or tighten a bare word to a word boundary (`where.*\beat\b`, not `where.*eat`). Reordering is a last resort; a guard is safer because it's local.

**For a coverage gap**, add a dedicated intent, following the existing patterns:

- Place it **before** the generic keyword fallback near the end.
- Reuse the shared helpers: `inTown(...)` to narrow by a named town, `listOf(...)` to format a rating-sorted list with the standard proviso, and `sortByRating`.
- For categories with price tiers (restaurants, lodging), degrade gracefully when a tier is absent in a town — filter to the requested tier, and if none exist, fall to the nearest tier with entries and say so, rather than silently showing everything.

After any fix, **re-run the battery (Step 0)** to confirm the fix works AND nothing else regressed.

## Step 4 — Publish

Run `/whatsnext:publish`, and in the preview re-run the exact questions that failed (plus the battery) to confirm they now answer well.
