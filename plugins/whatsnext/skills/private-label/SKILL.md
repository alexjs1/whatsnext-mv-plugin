---
name: private-label
description: Create (or update) a private-label "Mobile Concierge" edition of What's Next MV for a specific hotel or inn, living at its own web address (e.g. saltwind-concierge.netlify.app). Use whenever the user types /whatsnext:private-label or asks to "make a hotel version of the app", "private-label the guide for a hotel/inn", "white-label edition", "build a mobile concierge for <property>", "co-brand the guide", or the reverse ("remove a hotel edition"). Gathers the property's logo, colors, font, contact info, hotel Info-tab content, and preferred businesses, writes a Partner record, verifies it in the preview, walks the user through creating the edition's Netlify site, and publishes.
---

# Private-label a hotel/inn edition

A **private-label edition** re-skins What's Next MV as one property's own concierge. When a partner is active the app:

- opens on a splash with the hotel's **logo, name, font, and colors** over "Mobile Concierge";
- carries a **"Contact <hotel>" credit on every tab** (the sponsor slot, repurposed) that offers call / website / email;
- adds a **hotel Info subtab** (checkout time, pool/gym hours, room-service menu, key numbers, WiFi — anything the hotel wants);
- **removes all other lodging** from the Map, Guide, and concierge (competing hotels are not listed);
- adds a **Favorites tab** (its own tab, second — Map | Favorites | Guide | Info | Ask) listing the hotel's recommended businesses, grouped by category, each opening the normal place-detail page.

The hotel's picks live in the **Favorites tab** — a curated subset of the Guide — rather than being boosted to the top of every list (that older ranking approach was removed as too fragile). The regular Guide/Map/concierge sort normally; only lodging is filtered out.

With **no** partner active the base app is byte-for-byte unchanged, so this is purely additive.

## Every edition gets its own web address

**This is the part that is easy to get wrong.** An edition does NOT just live at `whatsnextmv.com/h/<slug>` any more. It gets its **own Netlify site**, built from the same repository, at its own address (e.g. `saltwind-concierge.netlify.app`).

The reason is phones, not tidiness. A phone decides which installed app owns a link by its **web address**. When every edition shared `whatsnextmv.com`, installing one blocked the others and links opened the wrong edition — the public guide would open inside a hotel's app, and Android refused to install a second one at all. On its own address, an edition's app covers only that address and they stop fighting.

`/h/<slug>` still works, so QR codes and emails already in the wild keep opening. It is a **legacy path**, not where new editions are sent.

So a finished edition is: a Partner record (Steps 1–2) **plus** a Netlify site (Step 4). Skipping Step 4 leaves the edition reachable only at the legacy path, and with no analytics of its own.

## What gets measured (automatic)

Nothing to configure per hotel. Every edition reports, tagged `edition = pl:<slug>`: visits and repeat visitors, screens, searches, which businesses came up and were opened, taps on directions/website/phone, and full-text concierge questions. On the hotel's OWN surfaces it also reports which sections of the curated menu were opened, which picks were taken up (from the menu and from the Favorites tab), front-desk contact taps, and use of the hotel's ask box. See `analytics/README.md` in the project. This is what a property gets shown at renewal, so **confirm Step 4's `ANALYTICS_DATABASE_URL` is set** — without it the edition silently records nothing.

## Project root

```
$HOME/Documents/mv-guide
```

## How it's wired (read before editing)

| File | Role |
|---|---|
| `src/data/partners.ts` | The `Partner` **type**, `PARTNERS` (REAL signed partners), and `DEMO_PARTNERS` (deployed sales samples, e.g. The Saltwind Inn). `slug` sets the URL key; optional `hosts: []` lists extra addresses the edition answers to (a hotel's own `concierge.theirdomain.com`). |
| `src/lib/partner.ts` | Resolves the active partner, in order: the **host** (`<slug>-concierge.netlify.app`, or anything in `hosts`), then the legacy `/h/<slug>` path, then `?partner=<slug>` for local previews. Also `useBrand()` / `isHiddenByPartner()`. |
| `scripts/build-partner-pages.mjs` | Post-build step. With `PARTNER_SLUG` set on a Netlify site it brands that site's `index.html` and writes its manifest, so the site IS that edition. Also writes the legacy `/h/<slug>.html` pages, and points `og:image` at `public/og-<slug>.png` when one exists. |
| `scripts/make-pwa-icons.mjs` | Home-screen icons. **Add a row to its `PARTNERS` list per hotel** and run it. |
| `src/lib/filters.ts`, `src/lib/ask.ts` | Call `isHiddenByPartner` to drop competing lodging — no edits needed per partner. |
| `src/app/(tabs)/favorites.tsx` | The hotel Favorites tab (curated subset of the Guide). |
| `src/components/BrandHeader.tsx`, `OnboardingGate.tsx`, `src/app/(tabs)/schedules.tsx`, `src/app/(tabs)/ask.tsx`, `(tabs)/_layout.tsx` | The branded surfaces — no edits needed per partner. |
| `netlify.toml` | Serves `/h/*`. |

**Adding a partner is data-only in the app**: you write ONE `Partner` record, drop in a logo, add an icon row, and (for a real property) a share image. You should not need to touch the components. The one thing that is NOT in the repo is the edition's Netlify site — see Step 4.

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

Then two assets, both needed before the edition's own site goes up:

1. **Home-screen icons.** Add a row to `PARTNERS` in `scripts/make-pwa-icons.mjs` (`{ slug, logo, bg }`) and run `node scripts/make-pwa-icons.mjs`. Check the result is legible at 192px and clearly NOT the island guide's WN/MV wordmark — two similar icons on one home screen is a real complaint.
2. **Share image** (real properties; optional for a demo). A 1200×630 `public/og-<slug>.png` matching the edition's splash: logo chip on the splash colour, name in the brand's heading font, concierge label in the accent colour, tagline beneath. Without one the link previews as the island guide's card, which is a poor first impression for a hotel's own link. It is a static asset on purpose — a hotel's card needs their licensed font, which is a design step, not a build step.

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
- from a pick's page, **Back returns to the curated list** you were reading, not the map;
- the hotel logo in the header offers **"Back to the \<shortName\> Guide"** above the contact actions;
- open the **bare** URL (`/`) and confirm the base app is unchanged (no Favorites tab, lodging back, "Ask our AI Concierge").

Run `npx tsc --noEmit` and confirm no console errors.

## Step 4 — Give the edition its own address (ASK THE USER)

The Netlify site cannot be created from the repo, so **prompt the user to do it** and wait for confirmation. Offer these steps verbatim; it takes about five minutes.

> 1. In Netlify, choose **Add new site → Import an existing project**.
> 2. Pick the same repository as the main site (`alexjs1/whats-next-mv`). Nothing is copied or forked — every edition builds from the same code.
> 3. Leave the build settings as they come across.
> 4. Under **Project configuration → Environment variables**, add two:
>    - `PARTNER_SLUG` = `<slug>` — the only thing that makes this site a different edition.
>    - `ANALYTICS_DATABASE_URL` — the same value the main site uses. **Without it the edition silently records nothing**: the log function accepts every batch and discards it.
> 5. Under **Project configuration → Change site name**, set `<slug>-concierge`, giving `<slug>-concierge.netlify.app`.
> 6. Deploy.

Notes to hold on to:

- **No Gemini key is needed.** The concierge endpoint is absolute (`extra.askEndpoint` in `app.json` → the What's Next MV site), so every edition inherits that one key. The copy of `ask.mjs` on an edition's own site is never called — testing it with `curl` proves nothing about the app.
- Environment variables only take effect on a **new deploy**, and on Netlify Pro a variable must have **Functions** in its scope to be readable by a function.
- A site-level variable **overrides** a shared/team one with the same name.

Then verify from outside, and say what you checked:

```
curl -s https://<slug>-concierge.netlify.app/ | grep -o '<title>[^<]*</title>'
curl -s -X POST -H 'Content-Type: application/json' -d '{"events":[]}' \
  https://<slug>-concierge.netlify.app/.netlify/functions/log-event
```

The title should be the hotel's. The log call should answer `Provide 1-500 events` (the database is wired) and **not** `no database configured` (it is not). Best proof is to open the site and confirm rows land with `edition = 'pl:<slug>'`.

## Step 5 — Publish (real partners only)

For a real signed partner, follow `/whatsnext:publish` (typecheck → export → verify → commit → push). Every Netlify site built from the repo redeploys, so the edition's site picks the change up automatically.

**Demos** in `DEMO_PARTNERS` deploy with the app but are never linked from the base app, and the site is `noindex`. Keep their contact details fake. Drop them from `DEMO_PARTNERS` once real partners exist.

## Removing an edition

Delete the partner's entry from `PARTNERS` (or `DEMO_PARTNERS`), its logo, its icon row in `make-pwa-icons.mjs`, its `public/icons/<slug>-*.png`, and `public/og-<slug>.png`. Then **delete the edition's Netlify site** — ask the user; nothing in the repo can. Leaving it up serves a stale copy of the edition at its own address forever.

## Notes

- The demo logo was generated with a small `sharp` script (SVG → PNG) run from inside the project so it resolves `node_modules`. Reuse that approach only to compose a PNG from assets the user provides — not to invent a brand.
- Keep `DEVELOPMENT.md` / `APP_OVERVIEW.md` current when the partner SYSTEM changes; adding an individual partner is data and needs no doc change.
