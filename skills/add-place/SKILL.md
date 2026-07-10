---
name: add-place
description: Add a place (restaurant, beach, lodging, market, landmark, trail, shop, etc.) to the What's Next MV guide. Use whenever the user types /whatsnext:add-place or asks to "add a place to the guide", "put a business on the map", "the guide is missing a spot", "add this restaurant/beach/hotel/market", or gives a name and (optionally) a URL to include. Verifies the facts against real sources, geocodes the location, writes a schema-correct data record in the right file and category (building a new category end-to-end if needed), then publishes.
---

# Add a place to the guide

## Project root

```
$HOME/Documents/mv-guide
```

All content is static TypeScript in `src/data/`, typed by `src/data/types.ts`, aggregated into `PLACES` by `src/data/places.ts`. You add a record; the app does the rest.

## Step 1 — Gather and verify (do not fabricate)

Get the name and town. Then verify the specifics against **real sources** (the business's own site, the Vineyard Gazette, MV Times, Trustees of Reservations, town sites, Google/Yelp): category, price tier, season/days/hours, and anything you'll put in the description. If a detail can't be verified, leave it out or keep the wording soft rather than inventing it. This app has had fabricated specifics before; accuracy is the whole point.

App copy does **not** follow the Island Analytics writing rules. Write a plain, factual one- or two-sentence description in the same voice as neighboring records.

## Step 2 — Geocode

The Geocoding API is **disabled** on the key (it returns `REQUEST_DENIED`). Use Places `findplacefromtext` with the geometry field. The key comes from the environment (`GOOGLE_KEY`) or the app's `.env`; never hardcode it into a commit.

```
KEY="$GOOGLE_KEY"
q="<Business Name>, <Town>, Martha's Vineyard, MA"
enc=$(python3 -c "import urllib.parse,sys;print(urllib.parse.quote(sys.argv[1]))" "$q")
curl -s "https://maps.googleapis.com/maps/api/place/findplacefromtext/json?input=$enc&inputtype=textquery&fields=name,formatted_address,geometry&key=$KEY"
```

Sanity-check the returned address is the right town before using the lat/lng.

## Step 3 — Pick the file and category

Find where siblings live by grepping for an existing place of the same category, and add next to them:

```
grep -rn "category: '<category>'" "$HOME/Documents/mv-guide/src/data/" | head
```

Rough map (confirm by grep, since files can drift): restaurants → `morePlaces.ts`; lodging & farmstands/markets → `moreStay.ts`; beaches → `places.ts` / `moreStay.ts`; trails & outdoor → `moreOutdoors.ts` / `outdoor.ts`; landmarks → `landmarks.ts`; camps/playgrounds/community/toystores → `kids.ts`; gyms → `gyms.ts`; food stores → `foodStores.ts`; liquor/cannabis → `drinks.ts`; gas stations → `gasStations.ts` (its own `GAS_STATIONS` list, not `PLACES`).

## Step 4 — Write the record

Match the fields the neighboring records of that category use (see `src/data/types.ts` for the full `Place` shape: `id, name, category, tier?, meals?, kidFriendly?, scene?, seasonal?, town, lat, lng, cuisine?, tags?, description, url?, phone?, rules?, fees?, ...`). Conventions:

- **id**: kebab-case, matching the neighbors' prefix (`r2-` restaurants, `m-` markets, `lm-` landmarks, `kc-` kids, etc.). Make sure it's unique.
- **Strings**: files use single-quote delimiters. Use curly apostrophes (’) and curly quotes (“ ”) in copy so you never have to escape the delimiter. **Never** do a `String.replace` with a `$`-containing replacement.
- Keep `town` to the real town string the data uses (village names like "Menemsha" are fine; town filters alias them).

## Step 5 — New category only

If the place needs a category that doesn't exist yet, wire the full chain (missing one shows a broken filter or an unstyled pin):

1. `src/data/types.ts` — add to the `Category` union.
2. `src/constants/theme.ts` — add `CategoryColors.<cat>` (a distinct hex) **and** `CategoryLabels.<cat>`.
3. `src/lib/filters.ts` — add the sub-type + filter branch, and include the category in the relevant default list.
4. `src/components/FilterBar.tsx` — add the filter chip.
5. `src/app/(tabs)/explore.tsx` — add the category to `CATEGORY_ORDER`.
6. `src/lib/ask.ts` — add a `listOf` concierge intent (placed **before** the generic keyword fallback).

## Step 6 — Publish

Run `/whatsnext:publish`. During the preview check, confirm the new place appears where expected (Guide/Map, and in a relevant concierge question) with no console errors.

> New places have **no Google rating** until the next `/whatsnext:refresh-ratings` run, so they'll show no star and sort after rated places. That's expected.
