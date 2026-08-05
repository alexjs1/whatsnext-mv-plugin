---
name: private-label
description: Create (or update) a private-label "Mobile Concierge" edition of What's Next MV for a specific partner — a hotel or inn, or a business association, chamber or town — living at its own web address (e.g. saltwind-concierge.netlify.app). Use whenever the user types /whatsnext:private-label or asks to "make a hotel version of the app", "private-label the guide for a hotel/inn/association", "white-label edition", "build a mobile concierge for <partner>", "co-brand the guide", or the reverse ("remove an edition"). Gathers the partner's logo, colors, font, contact info, the sections of its custom menu, and its recommended or member businesses, writes a Partner record, verifies it in the preview, walks the user through creating the edition's Netlify site, and publishes.
---

# Private-label edition

A **private-label edition** re-skins What's Next MV as one partner's own concierge. Two kinds exist and they differ in more than wording:

| | **Lodging** (hotel, inn) | **Association** (business association, chamber, town) |
|---|---|---|
| Front door | Guest information and where to eat | The town's story, its member businesses, what's on |
| Other lodging | **Culled** — it does not advertise competitors | **Kept** — the local inns are paying members |
| Second tab | "Favorites" — the places it recommends | "Members" — its own roster |
| Info subtab | Checkout, pool hours, room service | Usually none; "about us" belongs on the menu |

When a partner is active the app:

- opens on a splash with the partner's **logo, name, font, and colors** over "Mobile Concierge";
- carries a **"Contact <partner>" credit on every tab** (the sponsor slot, repurposed) that offers call / website / email;
- can add an **Info subtab** for a lodging partner's guest information;
- **removes all other lodging** — but only when `hidesOtherLodging` is left at its default. An association MUST set it `false`;
- adds a **second tab** (Map | Favorites | Guide | Info | Ask) listing the partner's businesses, grouped by category, each opening the normal place-detail page. Rename it with `favoritesLabel` / `favoritesTitle` / `favoritesBlurb`.

The partner's picks live in that tab — a curated subset of the Guide — rather than being boosted to the top of every list (that older ranking approach was removed as too fragile). The regular Guide/Map/concierge sort normally.

With **no** partner active the base app is byte-for-byte unchanged, so this is purely additive.

## The custom menu is the partner's, not ours

The front-door menu (`pocketConcierge.sections`) is an **ordered list of whatever that partner wants**, each section labelled in their words and given its own accent color. This skill does not prescribe the sections and you should not assume last edition's set — ask.

In practice most editions want some mix of **retail establishments**, **historical context**, and **must-see locations**, and that is a good starting proposal. But a hotel may want its room-service menu where an association wants its membership form, and both are correct. Build what the partner asks for.

Each section carries one content **shape** (`PocketSectionBody` in `src/data/partners.ts`):

| `kind` | For |
|---|---|
| `prose` | Long-form copy, split into `chapters` — each its own nested accordion, so a history reads as a contents list rather than a wall of text |
| `places` | Guide places in named groups (by meal, by category, by street). Set `collapsibleGroups` when there are more than a handful — each heading becomes its own accordion with a count, and a long directory collapses to one screen |
| `activities` | Things to do, whether or not they map to a guide place |
| `events` | `eventIds` from the guide's own EVENTS (dates stay current with the seasonal refresh) plus the partner's own `items` |
| `info` | The partner's `infoSections` |
| `heritage` | The African American Heritage Trail card |
| `comingSoon` | Announced but not built — renders one note |

**Accordions wherever the content allows.** Phones read them far better than long pages, and `prose` chapters plus `places` groups both collapse by default.

### A visitor must always be able to get back to the menu

The curated menu is the front door, and someone three taps deep in the Guide or on a place page has no obvious way home unless you give them one. So, in every edition:

- The partner's **mark sits at the top of the menu** and at the **top right of every other screen**, and both open the SAME actions — call, website, email, the partner's `contact.links`, and add-to-home-screen. They render from one component, `src/components/PartnerContactActions.tsx`; do not hand-roll a second button list, which is exactly how the two drifted apart before.
- Those actions lead with **"Back to the \<shortName\> menu"** on every surface except the menu itself, where it would do nothing. It works from a place page too (the header steps back to a tab first, then the gate reopens the concierge).

Treat "can I get to the menu from here?" as part of Step 3's verification, not a nicety.

An edition written before `sections` existed (the Saltwind demo) still uses the legacy `dining` / `shopping` / `landmarks` / `cantMiss` fields; those are folded into the same renderer and keep working untouched.

## Every edition gets its own web address

**This is the part that is easy to get wrong.** An edition does NOT just live at `whatsnextmv.com/h/<slug>` any more. It gets its **own Netlify site**, built from the same repository, at its own address (e.g. `saltwind-concierge.netlify.app`).

The reason is phones, not tidiness. A phone decides which installed app owns a link by its **web address**. When every edition shared `whatsnextmv.com`, installing one blocked the others and links opened the wrong edition — the public guide would open inside a hotel's app, and Android refused to install a second one at all. On its own address, an edition's app covers only that address and they stop fighting.

`/h/<slug>` still works, so QR codes and emails already in the wild keep opening. It is a **legacy path**, not where new editions are sent.

So a finished edition is: a Partner record (Steps 1–2) **plus** a Netlify site (Step 4). Skipping Step 4 leaves the edition reachable only at the legacy path, and with no analytics of its own.

## What gets measured (automatic)

Nothing to configure per partner. Every edition reports, tagged `edition = pl:<slug>`: visits and repeat visitors, screens, searches, which businesses came up and were opened, taps on directions/website/phone, and full-text concierge questions. On the partner's OWN surfaces it also reports which sections of the curated menu were opened (by their `key`, so keep keys stable across relabels), which picks were taken up (from the menu and from the second tab), contact taps, and use of the partner's ask box. See `analytics/README.md` in the project. This is what a partner gets shown at renewal, so **confirm Step 4's `ANALYTICS_DATABASE_URL` is set** — without it the edition silently records nothing.

## Project root

```
$HOME/Documents/mv-guide
```

## How it's wired (read before editing)

| File | Role |
|---|---|
| `src/data/partners.ts` | The `Partner` **type**, the `PocketSection` menu types, `PARTNERS` (REAL signed partners), and `DEMO_PARTNERS` (deployed sales samples, e.g. The Saltwind Inn). `slug` sets the URL key; optional `hosts: []` lists extra addresses the edition answers to (a partner's own `concierge.theirdomain.com`). |
| `src/components/PocketConcierge.tsx` | Renders the menu + panel from `sections` — or synthesises them from the legacy fields. Section-agnostic; adding a section is data. |
| `src/data/memberships.ts` | Association membership tags. An invisible data layer — nothing in the base app renders it. |
| `src/lib/partner.ts` | Resolves the active partner, in order: the **host** (`<slug>-concierge.netlify.app`, or anything in `hosts`), then the legacy `/h/<slug>` path, then `?partner=<slug>` for local previews. Also `useBrand()` / `isHiddenByPartner()`. |
| `scripts/build-partner-pages.mjs` | Post-build step. With `PARTNER_SLUG` set on a Netlify site it brands that site's `index.html` and writes its manifest, so the site IS that edition. Also writes the legacy `/h/<slug>.html` pages, and points `og:image` at `public/og-<slug>.png` when one exists. |
| `scripts/make-pwa-icons.mjs` | Home-screen icons. **Add a row to its `PARTNERS` list per edition** and run it. A dark mark needs the REVERSED copy here — the icon sits on the edition's own background color. |
| `src/lib/filters.ts`, `src/lib/ask.ts` | Call `isHiddenByPartner` to drop competing lodging — no edits needed per partner. |
| `src/app/(tabs)/favorites.tsx` | The partner's second tab (curated subset of the Guide); headings overridable. |
| `src/components/BrandHeader.tsx`, `OnboardingGate.tsx`, `src/app/(tabs)/schedules.tsx`, `src/app/(tabs)/ask.tsx`, `(tabs)/_layout.tsx` | The branded surfaces — no edits needed per partner. |
| `netlify.toml` | Serves `/h/*`. |

**Adding a partner is data-only in the app**: you write ONE `Partner` record, drop in a logo, add an icon row, and (for a real partner) a share image. You should not need to touch the components — the menu renderer is section-agnostic, so a brand-new set of sections is still just data.

The exception is a genuinely new **content shape**. If a partner wants something none of the `kind`s can express, add the variant to `PocketSectionBody` and a branch to `sectionBody()` in `PocketConcierge.tsx` — then say so, because that IS a system change and belongs in this file.

The one thing that is NOT in the repo is the edition's Netlify site — see Step 4.

## Step 1 — Gather the partner's details

Ask the user for (offer to proceed with sensible placeholders for anything they don't have yet):

1. **Name + slug** — e.g. "The Harbor View Hotel" → `harborview` (lowercase, URL-safe). Set a clean **`shortName`** too (e.g. "Harbor View"): it drives the concierge label — the wizard and Ask-tab prompt read **"Ask your \<shortName\> Pocket Concierge"** — so avoid a leading "The" that would read as "Ask your The …".
2. **Logo** — a file from the user, at `assets/images/partners/<slug>-logo.png`. Never hand-fake a logo you weren't given — ask for the real asset. Two things to check before you accept it:
   - **A transparent background does not make a dark mark usable.** Dark line art on the splash color is unreadable. Either get a light/knocked-out copy and set `logoReversed` (the splash then drops the white chip and shows it directly on the color), or leave it unset and take the chip. Inverting pure black-and-white line art is a legitimate mechanical treatment; recoloring a brand is not.
   - **A mark taller than it is wide** needs `logoPortrait: true`, or it shrinks to a stripe in the splash, header and menu boxes, all of which are sized for a landscape wordmark.
3. **Colors** — header/background, primary (buttons, links, active tab), accent (splash highlight), plus a color per menu section. Pull them from the partner's own site rather than eyedropping a screenshot: a Squarespace site exposes `--accent-hsl` / `--darkAccent-hsl` / `--lightAccent-hsl` in its `site.css`, and most site builders expose something equivalent. **Show the user the palette and type before building.**
4. **Font** — a web font family + its stylesheet URL (Google Fonts is fine). Sets `fontHeadingWeb` + `webFontLink`. If their face is licensed (Adobe Fonts, a foundry), pick the closest free equivalent and say so — **Jost** is already bundled and is the app's Futura stand-in, so it costs nothing and works on native too. Any other native face must be bundled in `useFonts` (`src/app/_layout.tsx`); wire `fontHeadingNative` but note it's inactive until a native build adds it.
5. **Contact** — phone, website, email, address, and what the phone reaches (`deskLabel`). Anything else they want one tap from every screen — a membership application, a booking page — goes in `contact.links` as `{ label, url }`.
6. **The menu** — which sections, in what order, in whose words. See "The custom menu is the partner's" above. This is the bulk of the work; write real content, not placeholders, and use `comingSoon` for anything genuinely not ready.
7. **Their businesses** — the places they recommend, or (for an association) their members. Find each `id` in `src/data/*.ts` and confirm it resolves — **verify every id programmatically before you finish**, a typo silently renders nothing. Association membership belongs in `src/data/memberships.ts`; derive `preferredPlaceIds` from it (`membersOf('oba', 'Oak Bluffs').map(p => p.id)`) rather than hand-listing, so a re-derived roster updates the tab for free.
8. **Is the partner itself a listed place?** If it has a guide record, set `placeId` so it survives the lodging cull and pins first.
9. **Does it sell rooms?** If not, set **`hidesOtherLodging: false`** and check the copy that assumes guests: `welcomeBlurb` (splash card), `favoritesTitle` / `favoritesBlurb` (second tab). The defaults all say "your stay".

## Step 2 — Write the Partner record

- **Real, signed partner** → append to `PARTNERS` in `src/data/partners.ts`. Use their REAL contact details; never seed a live partner with 555 placeholders.
- **Demo / sales sample** → add to `DEMO_PARTNERS` in the same file (use the `saltwind` entry as the template) and set `demo: true`. Demos DO deploy but are reachable only at their `/h/<slug>` deep link; keep contact details obviously fake (555 number, example.com) so a sample can't be mistaken for a real business.

Match the `Partner` interface in `src/data/partners.ts` exactly (it is fully commented). Optional `favoritesLabel` renames the Favorites tab (e.g. "SW Favorites"); `conciergeLabel` is the splash subtitle; `shortName` drives the "Ask your … Pocket Concierge" prompt.

Then two assets, both needed before the edition's own site goes up:

1. **Home-screen icons.** Add a row to `PARTNERS` in `scripts/make-pwa-icons.mjs` (`{ slug, logo, bg }`) and run `node scripts/make-pwa-icons.mjs`. Check the result is legible at 192px and clearly NOT the island guide's WN/MV wordmark — two similar icons on one home screen is a real complaint.
2. **Share image** (real partners; optional for a demo). A 1200×630 `public/og-<slug>.png` matching the edition's splash: logo chip on the splash colour, name in the brand's heading font, concierge label in the accent colour, tagline beneath. Without one the link previews as the island guide's card, which is a poor first impression for a partner's own link. It is a static asset on purpose. Note that librsvg ignores a base64 `@font-face`: to render the partner's real faces, point `FONTCONFIG_FILE` at a temp dir holding the TTFs.

## Step 3 — Verify in the preview

```
npx expo start --web --port 8081
```

In the dev preview use **`/?partner=<slug>`** (`/h/<slug>` is written at build time, not by Metro), and check:

- splash shows the logo — legible, right proportions — the name in its font, the colors, "Mobile Concierge";
- the header + tab bar carry the partner's colors, and the top-right mark opens the contact sheet;
- **every menu section** opens, and each accordion inside it opens and closes;
- a place row opens a place page with Directions + Call, and **Back returns to the list you were reading**, not the map;
- **lodging** is gone for a hotel edition ("where should I stay" returns nothing) and **present** for an association — compare the map's place count against the base app's; they should differ only if the cull is on;
- the second tab is labelled and worded for this partner, not "Favorites … places we love";
- the mark at the top of the MENU and the one at the top RIGHT of every other screen open the same actions, and every surface except the menu leads with **"Back to the \<shortName\> menu"** — test it from a place page, which is the case that used to strand people;
- the **Saltwind demo still works** (`/?partner=saltwind`) — it exercises the legacy menu path;
- open **`/?partner=none`** and confirm the base app is unchanged (no second tab, lodging back, "Ask our AI Concierge"). A plain `/` will NOT do: the active edition deliberately sticks for the browsing session.

The session-stickiness catches people out. If the bare URL still shows the partner, that is correct behavior, not a bug.

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

The title should be the partner's. The log call should answer `Provide 1-500 events` (the database is wired) and **not** `no database configured` (it is not). Best proof is to open the site and confirm rows land with `edition = 'pl:<slug>'`.

## Step 5 — Publish (real partners only)

For a real signed partner, follow `/whatsnext:publish` (typecheck → export → verify → commit → push). Every Netlify site built from the repo redeploys, so the edition's site picks the change up automatically.

**Demos** in `DEMO_PARTNERS` deploy with the app but are never linked from the base app, and the site is `noindex`. Keep their contact details fake. Drop them from `DEMO_PARTNERS` once real partners exist.

## Removing an edition

Delete the partner's entry from `PARTNERS` (or `DEMO_PARTNERS`), its logo, its icon row in `make-pwa-icons.mjs`, its `public/icons/<slug>-*.png`, and `public/og-<slug>.png`. Then **delete the edition's Netlify site** — ask the user; nothing in the repo can. Leaving it up serves a stale copy of the edition at its own address forever.

## Notes

- The demo logo was generated with a small `sharp` script (SVG → PNG) run from inside the project so it resolves `node_modules`. Reuse that approach only to compose a PNG from assets the user provides — not to invent a brand.
- Keep `DEVELOPMENT.md` / `APP_OVERVIEW.md` current when the partner SYSTEM changes; adding an individual partner is data and needs no doc change.
