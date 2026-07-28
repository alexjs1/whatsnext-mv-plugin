---
name: pocket-concierge
description: Build a curated "Pocket Concierge" front door for a private-label hotel edition of What's Next MV — a hand-picked menu of nearby dining (by meal and budget), can't-miss island activities, the African American Heritage Trail, and an "ask" box, shown after the splash at whatsnextmv.com/h/<slug>. Use whenever the user types /whatsnext:pocket-concierge or asks to "add a pocket concierge for <hotel>", "curate the hotel's front page", "make a curated concierge menu for <property>", "set up the hotel's what-to-do menu", or the reverse ("remove the pocket concierge"). Curates from the hotel's town (seeded from live Google ratings, approval-gated), verifies price tiers and walkability, writes the partner's pocketConcierge config, verifies in the preview, and publishes.
---

# Build a hotel's Pocket Concierge

A **Pocket Concierge** is a curated front door for a private-label hotel edition, shown after the (longer) branded splash instead of the generic intent/town wizard. It has two views, kept deliberately simple:

- **Menu** — a lean launcher: the title ("<hotel> recommendations around town"), one button per section, then an "or ask" box that routes to the AI concierge. Sections: **Hotel information** (first, built from the partner's `infoSections`), **Local restaurants we love**, **Where to shop in town**, **Must-see \<town\> landmarks**, **African American Heritage Trail**, **Can't-miss Vineyard activities** (the hotel words its own button labels via `labels`, including `hotelInfo`). The **Hotel information** category replaces the old Info-tab hotel subtab — that subtab is now retired for any edition that has a Pocket Concierge.
- **Panel** — tapping a button opens the picks as **collapsible accordions** (the tapped section opens first, the rest stay closed so it's never a wall of text), with a "‹ Menu" control back to the launcher. Dining groups by meal (price tier, Google rating, a "short trip out" flag for anything not walkable); the heritage accordion shows a summary, the in-town stops pulled live from the trail data, and a link to the official site.

Every pick is a real guide place, so a tap opens its detail card. A hotel can give any pick its **own words** (`desc`), which override the guide's description in this view. It is **data-driven per hotel**: you fill in one config block, no component code. The base app has no config, so it is untouched. Analytics tag this traffic as `edition = pl:<slug>`.

## Prerequisite

The hotel must already be a **Partner** (run `/whatsnext:private-label` first). This skill only adds the `pocketConcierge` config to that partner's record. If the property isn't in `partners.ts` yet, do the private-label edition first.

## Project root

```
$HOME/Documents/mv-guide
```

## How it's wired (read before editing)

| File | Role |
|---|---|
| `src/data/partners.ts` | The `Partner` record. The `PocketConcierge` type and its parts (`PocketMeal`, `DiningPick`, `PocketPick`, `PocketActivity`) live here. You add a `pocketConcierge: {...}` to the target partner. |
| `src/components/PocketConcierge.tsx` | Renders the menu from the config. **No edits per hotel.** |
| `src/components/OnboardingGate.tsx` | Shows the menu when `partner.pocketConcierge` exists, and uses `pocketConcierge.splashMs` for the splash. **No edits per hotel.** |
| `src/data/heritageTrail.ts` | `TRAIL_SITES` (with `town`) and `OFFICIAL_TRAIL_URL` — the heritage card pulls the in-town stops from here automatically. |
| `src/data/ratings.ts`, `src/data/places.ts` | `RATINGS` and `PLACES` — the source for curation. |

**Adding a Pocket Concierge is data-only.** You write one `pocketConcierge` block; you should not touch the components.

## The config shape

```ts
pocketConcierge: {
  splashMs: 5000,                 // longer splash for the curated edition (base app stays 4000)
  menuTitle: '…',                 // optional; default "<shortName> recommendations around town"
  labels: {                       // optional per-section button wording
    restaurants: 'Local restaurants we love',
    shopping: 'Where to shop in town',
    landmarks: 'Must-see Oak Bluffs landmarks',
    heritage: 'African American Heritage Trail',
    cantMiss: 'Can’t-miss Vineyard activities',
  },
  intro: '…',                     // one line under the title
  dining: [                       // meal-first; repeats across meals are fine
    { meal: 'Breakfast', picks: [ { id, tier: '$'|'$$'|'$$$', note?, walkOut?, desc? }, … ] },
    { meal: 'Lunch',     picks: [ … ] },
    { meal: 'Dinner',    picks: [ … ] },
  ],
  shopping:  [ { id, note?, desc? }, … ],
  landmarks: [ { id, note?, desc? }, … ],
  cantMiss:  [ { id?, title, blurb?, season? }, … ],   // id optional → text-only, non-tappable
  heritageIntro: '…',             // your own summary; the stops + link are automatic
  askCta: '…',                    // the "or ask" line above the concierge box
  askPlaceholder: '…',            // ask-box placeholder
}
```

`desc` on any dining/shopping/landmark pick is the **hotel's own words** for that place — it overrides the guide's description in the panel. Use it where the property wants to say something the guide doesn't (a favorite dish, "ask for a table by the window"). Leave it off to fall back to the guide copy.

## Step 1 — Anchor the hotel in its town

Get the property's `town` from its Partner record, and pick a **downtown anchor** — a central, well-known landmark in that town (e.g. the Flying Horses Carousel for Oak Bluffs, the Edgartown Lighthouse for Edgartown). You'll measure walkability from it.

## Step 2 — Pull the candidates (seed from real ratings)

Ground every pick in real venues and live ratings. Node can import `PLACES` and `RATINGS` (relative imports) but NOT `rating.ts` (it has an `@/` value import that fails in Node), so read `RATINGS` directly:

```
cd "$HOME/Documents/mv-guide" && node --input-type=module -e '
import { PLACES } from "./src/data/places.ts";
import { RATINGS } from "./src/data/ratings.ts";
const R=(id)=>RATINGS[id]?.rating??0, rc=(id)=>RATINGS[id]?.count??0;
const TOWN="Oak Bluffs";                 // <-- the hotel town
const ANCHOR=/flying horses/i;            // <-- a central landmark name
const coord=(p)=>[p.lat??p.latitude, p.lng??p.longitude];
const a=PLACES.find(p=>ANCHOR.test(p.name)), [aLat,aLng]=coord(a);
const mi=(la,lo)=>{const d=Math.PI/180,x=Math.sin((la-aLat)*d/2)**2+Math.cos(aLat*d)*Math.cos(la*d)*Math.sin((lo-aLng)*d/2)**2;return 3959*2*Math.asin(Math.sqrt(x));};
const inTown=PLACES.filter(p=>(p.town||"").includes(TOWN));
const walk=(p)=>{const[la,lo]=coord(p);return la&&lo?mi(la,lo):9;};
const line=(p)=>`  ${p.name} [${p.tier||"?"}] ★${R(p.id).toFixed(1)}(${rc(p.id)}) meals:${(p.meals||[]).join("/")||"-"} ${walk(p)<0.45?"DOWNTOWN":walk(p).toFixed(2)+"mi"} id=${p.id}`;
for(const t of ["$","$$","$$$"]){console.log("== restaurants "+t+" ==");inTown.filter(p=>p.category==="restaurant"&&p.tier===t).sort((x,y)=>R(y.id)-R(x.id)).slice(0,10).forEach(p=>console.log(line(p)));}
console.log("== shops ==");inTown.filter(p=>p.category==="shop"||p.category==="toystore").sort((x,y)=>R(y.id)-R(x.id)).slice(0,12).forEach(p=>console.log(line(p)));
console.log("== landmarks/sights ==");inTown.filter(p=>["landmark","venue","beach"].includes(p.category)||p.landmark).sort((x,y)=>R(y.id)-R(x.id)).slice(0,14).forEach(p=>console.log(line(p)));
' 2>&1 | grep -v -iE "warning|reparsing|trace-warnings|module_typeless|es module|performance overhead|eliminate"
```

Note the exact **place ids** (a typo silently drops the row, since the component skips ids it can't resolve). Prefer well-reviewed, recognizable, genuinely in-town spots; be wary of a 5.0 with three reviews. Watch for venues that are technically in-town but not walkable (a golf club, a spot on the far chop) — those get `walkOut: true`, not exclusion, if they're worth including.

## Step 3 — Draft and get approval (the gate)

Curate, then **present the full draft to the user before writing anything** — meal-first dining with each pick's real price tier, rating, and walkable-vs-short-trip flag, plus shopping, landmarks, and 6–8 island-wide can't-miss activities. This mirrors `/whatsnext:research-tag`: the human sign-off is the point.

- **Verify every price tier from the data**, don't assume. Flag any the user placed in the wrong tier (e.g. a "casual" pick that's actually `$$`).
- Repeats across meals are fine and expected (a spot that does breakfast and lunch appears under both).
- For **can't-miss**, use a place `id` when the activity maps to one (it becomes tappable); leave `id` off for text-only entries ("Sunset at Menemsha", "Bike the paved paths"). Mark seasonal items with `season`.
- **Heritage Trail is automatic**: the card pulls the in-town stops from `heritageTrail.ts` and links to `OFFICIAL_TRAIL_URL`. You only write `heritageIntro` — a short summary **in your own words** (never copy the organization's mission statement verbatim).

Let the user approve, swap, or cut. Do not write the config until they approve.

## Step 4 — Write the config

Add the approved `pocketConcierge: {...}` block to the target partner's record in `src/data/partners.ts` (see the shape above and the Saltwind record for a worked example). Default `splashMs: 5000`.

## Step 5 — Verify in the preview

Start the preview (`/whatsnext:start` if not running) and open `/h/<slug>`:

- The splash runs the longer duration, then the curated menu appears (not the generic wizard).
- Dining is grouped by meal with correct tiers, ratings, and walkable flags; shopping, landmarks, can't-miss, and the Heritage Trail card (with in-town stops) all render.
- Tapping a pick opens its place card; the "ask" box routes to the Ask concierge.
- Load the base app (`/`) to confirm it still shows the normal wizard (the config is partner-only).

Then `npx tsc --noEmit`.

## Step 6 — Publish

Run `/whatsnext:publish`. Commit message names the hotel, e.g. "Add Pocket Concierge for The Saltwind Inn".

## Notes

- **Data-only, per hotel.** Never edit `PocketConcierge.tsx` or `OnboardingGate.tsx` to add a property; if a hotel needs a section the type doesn't support, that's a deliberate component change, not this skill.
- **Every id is verified** against `src/data/` — a wrong id doesn't error, it just silently omits the row.
- **Tiers and walkability must be real**, pulled from the data, not guessed.
- To **remove** a Pocket Concierge, delete the partner's `pocketConcierge` block; the edition falls back to the generic wizard automatically.
