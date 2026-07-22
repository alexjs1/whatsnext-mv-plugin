---
name: private-label
description: Create (or update) a private-label "Mobile Concierge" edition of What's Next MV for a specific hotel or inn, reached at whatsnextmv.com/h/<hotel>. Use whenever the user types /whatsnext:private-label or asks to "make a hotel version of the app", "private-label the guide for a hotel/inn", "white-label edition", "build a mobile concierge for <property>", "co-brand the guide", or the reverse ("remove a hotel edition"). Gathers the property's logo, colors, font, contact info, hotel Info-tab content, and preferred businesses, writes a Partner record, verifies it in the preview, and publishes.
---

# Private-label a hotel/inn edition

A **private-label edition** re-skins What's Next MV as one property's own concierge, reached at `whatsnextmv.com/h/<slug>`. When a partner is active the app:

- opens on a splash with the hotel's **logo, name, font, and colors** over "Mobile Concierge";
- carries a **"Contact <hotel>" credit on every tab** (the sponsor slot, repurposed) that offers call / website / email;
- adds a **hotel Info subtab** (checkout time, pool/gym hours, room-service menu, key numbers, WiFi — anything the hotel wants);
- **removes all other lodging** from the Map, Guide, and concierge (competing hotels are not listed);
- adds a **Favorites tab** (its own tab, second — Map | Favorites | Guide | Info | Ask) listing the hotel's recommended businesses, grouped by category, each opening the normal place-detail page.

The hotel's picks live in the **Favorites tab** — a curated subset of the Guide — rather than being boosted to the top of every list (that older ranking approach was removed as too fragile). The regular Guide/Map/concierge sort normally; only lodging is filtered out.

With **no** partner active the base app is byte-for-byte unchanged, so this is purely additive.

## Project root

```
$HOME/Documents/mv-guide
```

## How it's wired (read before editing)

| File | Role |
|---|---|
| `src/data/partners.ts` | The `Partner` **type**, `PARTNERS` (REAL signed partners), and `DEMO_PARTNERS` (deployed sales samples, e.g. The Saltwind Inn at `/h/saltwind`). |
| `src/lib/partner.ts` | Resolves the active partner from the URL (`/h/<slug>` or `?partner=<slug>`), and the `useBrand()` / `isHiddenByPartner()` helpers. |
| `src/lib/filters.ts`, `src/lib/ask.ts` | Call `isHiddenByPartner` to drop competing lodging — no edits needed per partner. |
| `src/app/(tabs)/favorites.tsx` | The hotel Favorites tab (curated subset of the Guide). |
| `src/components/BrandHeader.tsx`, `OnboardingGate.tsx`, `src/app/(tabs)/schedules.tsx`, `src/app/(tabs)/ask.tsx`, `(tabs)/_layout.tsx` | The branded surfaces — no edits needed per partner. |
| `netlify.toml` | Serves `/h/*`. |

**Adding a partner is data-only**: you write ONE `Partner` record and drop in a logo. You should not need to touch the components.

## Step 1 — Gather the property's details

Ask the user for (offer to proceed with sensible placeholders for anything they don't have yet):

1. **Name + slug** — e.g. "The Harbor View Hotel" → `harborview` (lowercase, URL-safe). Set a clean **`shortName`** too (e.g. "Harbor View"): it drives the concierge label — the wizard and Ask-tab prompt read **"Ask your \<shortName\> Pocket Concierge"** — so avoid a leading "The" that would read as "Ask your The …".
2. **Logo** — a file from the user. Put it at `assets/images/partners/<slug>-logo.png`. If they have only an SVG or a wordmark, rasterize/compose a clean PNG (see the demo generator note below). Never hand-fake a logo you weren't given — ask for the real asset.
3. **Colors** — header/background, primary (buttons, links, active tab), accent (splash highlight). Pull from their brand or logo.
4. **Font** — a web font family + its stylesheet URL (Google Fonts is fine). Sets `fontHeadingWeb` + `webFontLink`. Native use needs the face bundled in `useFonts` (`src/app/_layout.tsx`) — wire `fontHeadingNative` but note it's inactive until a native build adds the face.
5. **Contact** — front-desk phone, website, email, address, and what to call the desk.
6. **Hotel Info sections** — the content for the hotel subtab: check-in/out, pool & fitness hours, room-service menu, key phone numbers, WiFi/parking, and anything else they want. Each becomes a `PartnerInfoSection` (`body` paragraph and/or `rows` of label→value).
7. **Preferred businesses** — the places they recommend. Find each one's `id` in `src/data/*.ts` (grep by name) and confirm it resolves. These fill the **Favorites tab**, grouped by category in the order listed.
8. **Is the hotel itself a listed place?** If it has a guide record, set `placeId` so it survives the lodging cull and pins first; otherwise leave it unset (the hotel lives in its branding + Info tab).

## Step 2 — Write the Partner record

- **Real, signed partner** → append to `PARTNERS` in `src/data/partners.ts`.
- **Demo / sales sample** → add to `DEMO_PARTNERS` in the same file (use the `saltwind` entry as the template) and set `demo: true`. Demos DO deploy but are reachable only at their `/h/<slug>` deep link; keep contact details obviously fake (555 number, example.com) so a sample can't be mistaken for a real bookable property.

Match the `Partner` interface in `src/data/partners.ts` exactly (it is fully commented). Optional `favoritesLabel` renames the Favorites tab (e.g. "SW Favorites"); `conciergeLabel` is the splash subtitle; `shortName` drives the "Ask your … Pocket Concierge" prompt.

## Step 3 — Verify in the preview

```
npx expo start --web --port 8081
```

Open **`/h/<slug>`** (or `/?partner=<slug>`) and check:

- splash shows the hotel logo, name (in its font), colors, "Mobile Concierge";
- the header + tab bar carry the hotel colors, and the top-right credit opens the contact sheet;
- the **Info** tab leads with the hotel subtab and its sections;
- **Lodging is gone** from the wizard, Map, Guide, and concierge ("where should I stay" returns nothing);
- the **Favorites tab** shows the hotel's picks grouped by category, and a row opens a place page with Directions + Call;
- open the **bare** URL (`/`) and confirm the base app is unchanged (no Favorites tab, lodging back, "Ask our AI Concierge").

Run `npx tsc --noEmit` and confirm no console errors.

## Step 4 — Publish (real partners only)

For a real signed partner, follow `/whatsnext:publish` (typecheck → export → verify → commit → push). The `/h/<slug>` rewrite is already in `netlify.toml`.

**Demos** in `DEMO_PARTNERS` deploy with the app but are reachable only at their `/h/<slug>` deep link — the base app never links to them and the site is `noindex`. Keep their contact details fake. Drop them from `DEMO_PARTNERS` once real partners exist.

## Removing an edition

Delete the partner's entry from `PARTNERS` (or `DEMO_PARTNERS`) and its logo asset. Nothing else references it.

## Notes

- The demo logo was generated with a small `sharp` script (SVG → PNG) run from inside the project so it resolves `node_modules`. Reuse that approach only to compose a PNG from assets the user provides — not to invent a brand.
- Keep `DEVELOPMENT.md` / `APP_OVERVIEW.md` current when the partner SYSTEM changes; adding an individual partner is data and needs no doc change.
