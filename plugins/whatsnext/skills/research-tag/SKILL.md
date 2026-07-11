---
name: research-tag
description: Run a deliberate, in-depth search to populate a hand-curated menu filter (ribs, vegan, vegetarian, gluten-free) for the What's Next MV concierge. Use whenever the user types /whatsnext:research-tag, or asks to "research the vegan spots", "find gluten-free restaurants", "populate the ribs filter", "update the vegan/vegetarian/gluten-free list", or names one or more of those filters to fill in. Sweeps every restaurant for the named filter(s), returns a sourced candidate list for approval, then writes src/data/menuTags.ts and publishes. Manual and on-demand only; never auto-scheduled.
---

# Research a curated menu filter

Four filters cannot be reliably scraped and are kept out of any automatic
refresh: **ribs, vegan, vegetarian, gluten-free**. They live in a hand-curated
overlay, `src/data/menuTags.ts`, keyed by place id. This skill is the deliberate
in-depth search that fills or refreshes one or more of them. Between runs the
data just sits there, correct and dated. Nothing about this is scheduled.

Why manual: these attributes are stable (a kitchen's gluten-free capability
rarely changes month to month), they are buried in menus, allergen pages,
reviews, or only learned by calling, and getting them wrong has real stakes
(sending a celiac guest to a place with no options). Human-verified and sourced
is the standard, matching the app's policy of only claiming what a source says.

## Project root

```
$HOME/Documents/mv-guide
```

The overlay and its types:

```
src/data/menuTags.ts   — MENU_TAGS (keyed by place id), MenuTag type, idsWithMenuTag()
src/lib/ask.ts         — the concierge reads it via a menuAsk block before the food engine
```

## Step 0 — Parse the filters

The user names one or more of: `ribs`, `vegan`, `vegetarian`, `gluten-free`.
A run may bundle several, e.g. `/whatsnext:research-tag vegan, gluten-free`.
**Each filter is researched and reported separately.** Bundling is only a
convenience (one sitting instead of several); it never merges results. Vegan and
vegetarian are distinct: a place can have vegetarian dishes and nothing vegan, so
one never implies the other. If the user names none, ask which of the four to run.

## Step 1 — Build the restaurant list

The candidates are the app's restaurants (plus a few relevant venues like The
Ritz). Pull the names, ids, and towns so the search is grounded in real places:

```
cd "$HOME/Documents/mv-guide" && grep -rhoE "id: '[^']*', name: '[^']*'.*category: '(restaurant|venue)'" src/data/*.ts
```

(Or read `src/data/places.ts` and `src/data/morePlaces.ts` for the full records.)
There are roughly 162 restaurants; the sweep covers all of them for the filter.

## Step 2 — Search, per filter

For each named filter, work through the restaurants and find which ones qualify,
using real sources: the restaurant's own menu or allergen page, current reviews,
reputable listings. **Every tag must trace to a named source.** Do not infer from
cuisine alone, and never guess. If a place can't be confirmed, leave it out; an
absent tag is honest, a wrong one is not.

Record a **level**, because these come in degrees, and the concierge should
answer honestly rather than flatten everything to yes/no. Capture it in the
`note` field, e.g.:

- vegan / vegetarian: "fully plant-based" vs. "dedicated menu" vs. "a few options"
- gluten-free: "dedicated GF menu / fryer" vs. "GF bread or pasta on request" vs. "marked options"
- ribs: the item itself (e.g. "St. Louis ribs", "baby back ribs")

Note closures as you go (a place marked closed should be dropped from the app
separately, via /whatsnext:seasonal-refresh or a place edit, not tagged here).

## Step 3 — Present candidates for approval

Before writing anything, show the user one clearly-separated list per filter:
each place, what you found, the level, and the source. Let the user approve, cut,
or correct. **Do not write `menuTags.ts` until the user approves.** This approval
gate is the whole point of the manual tier.

## Step 4 — Write the overlay

For each approved place, add or update its entry in `MENU_TAGS` in
`src/data/menuTags.ts`. Merge, don't clobber: a place may already carry another
filter's tag, so add to its `tags` array rather than replacing the entry. Set
`verified` to the current month (`YYYY-MM`), and fill `source` and (where useful)
`note`.

```ts
export const MENU_TAGS: Record<string, MenuTagEntry> = {
  'v-ritz': { tags: ['ribs'], verified: '2026-07', source: 'The Ritz Soul Kitchen & BBQ menu (St. Louis ribs)' },
  'r-example': { tags: ['vegan', 'vegetarian'], verified: '2026-07', source: 'menu', note: 'dedicated vegan menu' },
};
```

Keys are Place ids. Confirm each id exists (grep it in `src/data/`) before adding;
a typo just means the tag silently matches nothing.

## Step 5 — Spot-check the concierge

The filter is answered by the `menuAsk` block in `src/lib/ask.ts`, which runs
before the food engine. Verify in the preview (`/whatsnext:start` if not running,
Ask tab): the researched filter now returns the approved places, and an
un-researched one still gives the honest "not tracked yet" reply. Ribs behaves
like a dish; vegan / vegetarian / gluten-free read as dietary filters.

## Step 6 — Publish

Run `/whatsnext:publish`. This edits only `src/data/menuTags.ts` (data), so
`tsc` plus the concierge spot-check above is enough. Commit message: name the
filter(s) refreshed and how many places each gained, e.g. "Populate vegan (11)
and gluten-free (7) menu filters".

## Notes

- Re-running a filter refreshes just that one; it never touches the others. Update `verified` on the entries you revisit.
- The list is intentionally small and hand-held. If a filter turns up nothing verifiable, that is a valid result: leave it empty and the concierge will say so.
- Keep the four filters as the only members of the `MenuTag` type. Adding a new curated filter is a code change (extend `MenuTag`, add the query patterns to the `menuAsk` block in `ask.ts`), not something this skill does on its own.
