---
name: private-label
description: Build (or update) a private-label edition of What's Next MV for a customer — a hotel or inn, a business association, a chamber, a town — as a custom app on the WNMV framework, living at its own web address (e.g. oba-concierge.netlify.app). Use whenever the user types /whatsnext:private-label or asks to "make a hotel version of the app", "set up a hotel edition end to end", "private-label the guide for a hotel/inn/association", "white-label edition", "build a mobile concierge for <customer>", "curate the front page for <customer>", "add a pocket concierge", "co-brand the guide", or the reverse ("remove an edition"). Gathers the customer's logo, colors, fonts, contact details and the contents of their custom menu, writes one Partner record, verifies it in the preview, walks the user through creating the edition's Netlify site, and publishes.
---

# Private-label edition

A private-label edition is **a custom app built on the What's Next MV framework**. The framework is fixed; everything the visitor reads is the customer's.

This skill owns the whole job — branding, menu, curation, verification, deployment. (It absorbed the former `pocket-concierge` and `hotel-edition` skills, which split the same work in two and drifted apart.)

## What every edition gets

Regardless of customer, an edition:

- lives at **its own web address**, so two editions on one phone never collide;
- opens on a **splash** with the customer's logo, name, font and colors;
- shows a **custom menu** — their sections, their words, their order — that should fit above the fold;
- carries the customer's **mark in two places** (top of the menu, top right of every other screen), both opening the same actions;
- sits on top of the **full WNMV map, Guide and AI concierge**, which a visitor can always reach;
- offers a **"Download the app"** route that always works;
- **reports its own analytics**, tagged `edition = pl:<slug>`.

With no partner active the base app is byte-for-byte unchanged. This is purely additive.

## The menu is the customer's. Do not invent it.

**Nothing about the custom menu — not the number of entries, not their names, not their contents — is specified by this skill.** Either the user tells you what goes on it, or you ask. Do not carry over the last edition's menu; the two built so far (a demo inn, a business association) share almost no sections.

What IS fixed is how the menu behaves:

- Every entry opens into an **accordion**. Long or complex content accordions again inside, by category or logical break.
- One entry is **"Download the app"**.
- Below the entries sits an **"or" divider and a line about the AI concierge**, over the ask box.
- The customer's **logo sits above the entries** with the same tap actions as the top-right mark.

---

# Rules that break the app if you get them wrong

Read these before writing anything. Each one cost a real bug.

### A visitor must always reach the menu — and the guide

Someone three taps into the Guide, or on a place page, has no way home unless you give them one.

The customer's mark at the top of the menu and at the top right of every other screen open the **same actions**, rendered from one component, `src/components/PartnerContactActions.tsx`. Never hand-roll a second button list — that is exactly how the two drifted apart before. Those actions lead with **"Back to the \<shortName\> menu"** everywhere except the menu itself, where it would do nothing.

The WNMV map, Guide, Info and Ask tabs stay reachable throughout. An edition is a front door onto the guide, not a replacement for it.

### Ask about installing once, not repeatedly

Two install surfaces exist: the `install` menu section (permanent, on demand) and the `AddToHomeScreen` banner (a timed nudge on the map). The banner must be genuinely once-per-device — record it as seen when it is **shown**, not when it is dismissed, or a visitor who ignores it gets it again on every trip back to the map.

### Never start `askCta` with "Or"

The menu draws a hard-coded "or" divider directly above it, so a CTA opening with "Or" prints the word twice. Write a bare instruction: "Ask any question about the town or MV".

### A dark logo on a dark splash is unreadable, transparent or not

Transparency does not make a dark mark legible on a deep color. Either supply a light/knocked-out copy as `logoReversed` (the splash then drops its white chip), or leave it unset and take the chip. Inverting pure black-and-white line art is a legitimate mechanical treatment; recoloring a brand is not. A mark **taller than it is wide** needs `logoPortrait: true`, or it shrinks to a stripe in boxes sized for a landscape wordmark.

### Never ship a coordinate you did not confirm

Walking-tour stops and map pins are frequently **private homes**. A guessed pin sends a stranger to someone's front door. `lat`/`lng` are optional precisely so an unconfirmed stop can render its text with no Directions link. See "Curating from real data" for how to resolve them safely.

### Verify every place id programmatically

A wrong id does not error — the row silently vanishes. Check every id resolves against `src/data/` before you finish, in a script, not by eye.

### Menu real estate is the scarce resource

Every row competes with the ask box below it. Before adding a section, check whether it belongs **inside** an existing one. Show the user the menu at 375px; if it overflows, cut the section blurbs (see Step 1.6) — together, and never silently.

---

# What the framework can hold

Each menu section carries one content **shape** (`PocketSectionBody` in `src/data/partners.ts`):

| `kind` | For |
|---|---|
| `prose` | Long-form copy split into `chapters`, each its own nested accordion — a history reads as a contents list, not a wall of text |
| `places` | Guide places in named groups. `collapsibleGroups` makes each heading its own accordion with a count, so a long directory collapses to one screen |
| `walks` | Self-guided walking tours: named walks, each an accordion of numbered stops, with a "walk this route" link and per-stop Directions. `heritage: true` appends the Heritage Trail card as a final nested entry |
| `events` | `eventIds` from the guide's own `EVENTS` (so dates stay current with the seasonal refresh) plus the customer's own `items` |
| `activities` | Things to do, whether or not they map to a guide place |
| `info` | The customer's `infoSections` (a lodging property's guest information) |
| `heritage` | The African American Heritage Trail card; `nested` tucks its stop list into a sub-accordion |
| `install` | "Download the app" — the written home-screen steps for both platforms, always shown |
| `comingSoon` | Announced but not built — renders one note |

Other per-section options: `color` (its accent), `blurb` (one line under the label), `emphasis: 'filled'` (draws the menu row as a solid button — use once, for a utility item, or it stops meaning anything), and `key` (the analytics key; keep it stable across relabels).

**Walking-tour routes** deliberately omit an origin from the Google Maps URL, so Maps starts from the visitor's **current location** rather than assuming they are at stop 1.

**If a customer wants something no shape can express**, add a variant to `PocketSectionBody` and a branch to `sectionBody()` in `PocketConcierge.tsx` — then say so, because that is a framework change and belongs in this file.

An edition written before `sections` existed (the Saltwind demo) uses the legacy `dining` / `shopping` / `landmarks` / `cantMiss` fields. Those still render through the same code. Leave them alone; do not use them for new work.

---

# Curating from real data

When a section lists places, ground it in the guide's own data rather than memory — then **get the user's approval before writing anything**. The human sign-off is the point, as in `/whatsnext:research-tag`.

**There are no ratings to sort by.** Google's star ratings were removed in Aug 2026 (`src/data/ratings.ts` and `fetch-ratings.mjs` are gone — see `DEVELOPMENT.md` → "Google Maps Platform terms"). Do not reinstate them, and do not rank a customer's list by a number. Curation here is editorial: read each candidate's own `description`, `tier`, `meals`, `scene` and location, and pick.

Node can import `PLACES` (relative import, explicit `.ts`) but NOT `rating.ts`, which has an `@/` value import that fails outside Metro. Print the candidates and read them:

```
cd "$HOME/Documents/mv-guide" && node --input-type=module -e '
import { PLACES } from "./src/data/places.ts";
const TOWN="Oak Bluffs";
const inTown=PLACES.filter(p=>(p.town||"").includes(TOWN));
const line=(p)=>`  ${p.name} [${p.tier||"?"}]${p.priority?" *FEATURED*":""} id=${p.id}\n      ${(p.description||"").slice(0,140)}`;
for (const c of ["restaurant","shop","landmark","venue","beach","lodging"]) {
  console.log("== "+c+" ==");
  inTown.filter(p=>p.category===c).sort((a,b)=>a.name.localeCompare(b.name)).forEach(p=>console.log(line(p)));
}
' 2>&1 | grep -v -iE "warning|reparsing|trace-warnings|module_typeless|es module|performance overhead|eliminate"
```

- Pick the recognizable, clearly-described places a concierge would name out loud. If a candidate's own description does not make the case, it does not belong on a customer's short list.
- **Verify price tiers and any distance/walkability claim from the data**, never assume.
- For an association, membership belongs in `src/data/memberships.ts`; derive the list from it (`membersOf('oba', 'Oak Bluffs').map(p => p.id)`) rather than hand-listing, so a re-derived roster updates for free.
- Present the full draft. Let the user approve, swap or cut. **Do not write the record until they approve.**

### Coordinates, when a section needs them

Resolve them with a script that writes a **review file and never edits data** (`scripts/geocode-walking-tour.mjs` is the working example), then read the file. Two things learned the hard way:

- The Cloud project has **Geocoding disabled** and Places enabled. Places is the better tool regardless: it echoes back the address it matched, so you can assert the house number and street came back as asked instead of trusting a confidence grade.
- Anything that resolves to a **street or park centroid** should be re-queried **by name**. That is how four Oak Bluffs stops were rescued from the wrong pin — a statue that resolved to the middle of a park, a house that resolved to the next town.

Leave `lat`/`lng` unset for anything you could not confirm.

---

# Customizing a place's description

Two different things, often confused:

- **Globally** — the business's own copy shows in WNMV *and* in every edition. This already has a skill: call **`/whatsnext:customize-description`**. Do not reinvent it here.
- **In this edition only** — today a menu pick takes a `desc` that overrides the guide's wording, **but only on that menu row**. Tap through to the place page and the global description returns.

A full edition-scoped override — one that follows the place into the Guide, the map and the second tab — is **not built**. If a customer asks, say so plainly rather than promising it, and treat it as a framework change.

**Expected direction:** customers will likely want an edition's custom description **promoted to the global WNMV description** on request. Design any future override with that promotion path in mind rather than as a dead end.

---

# How it's wired

Project root: `$HOME/Documents/mv-guide`

| File | Role |
|---|---|
| `src/data/partners.ts` | The `Partner` type, the `PocketSection` menu types, `PARTNERS` (real signed customers) and `DEMO_PARTNERS` (deployed sales samples). `slug` sets the URL key; `hosts: []` lists extra addresses the edition answers to |
| `src/components/PocketConcierge.tsx` | Renders the menu and its panels from `sections`. Section-agnostic — adding a section is data |
| `src/components/PartnerContactActions.tsx` | The single button list behind BOTH marks. Add actions here, never in a host |
| `src/components/InstallAction.tsx` | The contact sheet's install control, and the exported `InstallSteps` the `install` section renders |
| `src/components/AddToHomeScreen.tsx` | The timed install banner on the map. Once per device |
| `src/lib/partner.ts` | Resolves the active partner: host, then legacy `/h/<slug>`, then `?partner=<slug>` for previews. Also `useBrand()` / `isHiddenByPartner()` / `ensureInstallable()` |
| `src/data/memberships.ts` | Association membership tags. Invisible in the base app |
| `src/app/(tabs)/favorites.tsx` | The customer's second tab; all headings overridable |
| `scripts/build-partner-pages.mjs` | Post-build. With `PARTNER_SLUG` set it brands that site's `index.html` and rewrites its manifest; also writes the legacy `/h/<slug>.html` pages |
| `scripts/make-pwa-icons.mjs` | Home-screen icons. Add a row per edition and run it |
| `src/lib/filters.ts`, `src/lib/ask.ts` | Call `isHiddenByPartner`; no per-customer edits |
| `netlify.toml` | Serves `/h/*` |

**Adding a customer is data-only**: one `Partner` record, a logo, an icon row, a share image. You should not touch the components.

## Every edition gets its own web address

An edition does NOT just live at `whatsnextmv.com/h/<slug>`. It gets its **own Netlify site**, built from the same repository.

The reason is phones. A phone decides which installed app owns a link by its **web address**. When editions shared one address, installing one blocked the others and links opened the wrong edition — Android refused to install a second at all. `/h/<slug>` still works so existing QR codes keep opening, but it is a **legacy path**, not where new editions are sent.

A finished edition is a Partner record **plus** a Netlify site. Skip the site and it is reachable only at the legacy path, with no analytics of its own.

## What gets measured (automatic)

Nothing to configure. Every edition reports, tagged `edition = pl:<slug>`: visits and repeat visitors, screens, searches, which businesses surfaced and were opened, taps on directions/website/phone, and full-text concierge questions. On the customer's own surfaces it also reports which menu sections were opened (by `key`), which picks were taken up, contact taps, and use of their ask box. See `analytics/README.md`. This is what a customer is shown at renewal, so confirm `ANALYTICS_DATABASE_URL` is set — without it the edition silently records nothing.

---

# Step 1 — Gather the customer's details

Offer sensible placeholders for anything they don't have yet.

1. **Name + slug** — "The Harbor View Hotel" → `harborview`. Set a clean `shortName` ("Harbor View"): it drives "Ask your \<shortName\> …", so avoid a leading "The".
2. **Logo** — a real file from the user, at `assets/images/partners/<slug>-logo.png`. Never hand-fake a mark. Check it against the logo rule above (`logoReversed`, `logoPortrait`).
3. **Colors** — header/background, primary, accent, plus one per menu section. Pull them from the customer's own site rather than eyedropping a screenshot: a Squarespace site exposes `--accent-hsl` / `--darkAccent-hsl` / `--lightAccent-hsl` in `site.css`; most builders expose an equivalent. **Show the user the palette and type before building anything.**
4. **Fonts** — family + stylesheet URL (Google Fonts is fine) → `fontHeadingWeb` + `webFontLink`. If their face is licensed (Adobe Fonts, a foundry), pick the closest free equivalent and say so — **Jost** is already bundled and is the app's Futura stand-in, so it costs nothing and works on native.
5. **Contact** — phone, website, email, address, and what the phone reaches (`deskLabel`). Anything else they want one tap from every screen — a membership application, a booking page — goes in `contact.links`.
6. **The menu** — which sections, in what order, in whose words. Ask; do not assume. Write real content, not placeholders; use `comingSoon` for anything genuinely not ready. Write a `blurb` per section by default, and cut them all if the menu overflows a phone screen.
7. **Their places** — see "Curating from real data". Approval gate applies.
8. **ASK: is this edition for a resort property — a hotel or an inn?** Do not infer it from the name or from the fact that they have rooms. The answer decides whether **competing lodging is hidden**, and both wrong answers are costly: a hotel listing rival inns advertises its competitors; an association whose inns vanish has deleted paying members from its own map, and their rows open nothing.
   - **Yes** → leave `hidesOtherLodging` at its default (true). Set `placeId` if the property has a guide record, so it survives the cull and pins first.
   - **No** → set **`hidesOtherLodging: false`**, and fix the copy that assumes guests: `welcomeBlurb` (splash card), `favoritesTitle` / `favoritesBlurb` (second tab). The defaults say "your stay" and "places we love".

# Step 2 — Write the Partner record

- **Real, signed customer** → append to `PARTNERS`. Use their real contact details; never seed a live customer with 555 placeholders.
- **Demo / sales sample** → add to `DEMO_PARTNERS`, set `demo: true`, and keep contact details obviously fake (555, example.com) so a sample can't be mistaken for a real business.

Match the `Partner` interface exactly; it is fully commented. `conciergeLabel` is the splash subtitle — the menu card does not repeat it, because the splash has just said it and the line costs a row.

Then two assets:

1. **Home-screen icons.** Add a row to `PARTNERS` in `scripts/make-pwa-icons.mjs` and run it. A dark mark needs the **reversed** copy here — the icon sits on the brand color. Check it is legible at 192px and clearly NOT the island guide's WN/MV wordmark; two similar icons on one home screen is a real complaint.
2. **Share image** — a 1200×630 `public/og-<slug>.png` matching the splash. Without one the link previews as the island guide's card. Note that librsvg ignores a base64 `@font-face`: to render the customer's real faces, point `FONTCONFIG_FILE` at a temp dir holding the TTFs.

**Manifests:** the edition's own site gets its manifest rewritten at build time by `build-partner-pages.mjs`, so nothing to do. Only if you also want the **legacy `/h/<slug>` path** installable under the customer's name do you need `public/manifest-<slug>.webmanifest` (copy `manifest-saltwind.webmanifest`). Keep it at the root of `public/` — the `/h/*` rewrite would shadow it.

# Step 3 — Verify in the preview

```
npx expo start --web --port 8081
```

Use **`/?partner=<slug>`** — `/h/<slug>` is written at build time, not by Metro. Check:

- splash: logo legible and correctly proportioned, name in its font, the colors;
- the menu **fits above the fold at 375px**, and every section and nested accordion opens and closes;
- a place row opens a place page with Directions + Call, and **Back returns to the list you were reading**, not the map;
- **both marks** (menu top, header top-right) open the same actions, and every surface except the menu leads with "Back to the … menu" — **test from a place page**, the case that used to strand people;
- the map, Guide, Info and Ask tabs all work;
- **lodging** is absent for a resort property and present otherwise — compare the map's place count against the base app's;
- the second tab is worded for this customer, not "Favorites … places we love";
- the **Saltwind demo still works** (`/?partner=saltwind`) — it exercises the legacy menu path;
- **`/?partner=none`** shows the unchanged base app. A plain `/` will NOT do: the active edition deliberately sticks for the browsing session, which is correct behavior, not a bug.

Then `npx tsc --noEmit` and a clean console.

# Step 4 — Give the edition its own address (ASK THE USER)

The Netlify site cannot be created from the repo. Offer these steps and wait:

> 1. In Netlify, **Add new site → Import an existing project**.
> 2. Pick the same repository as the main site (`alexjs1/whats-next-mv`). Nothing is forked — every edition builds from the same code.
> 3. Leave the build settings as they come across.
> 4. Under **Project configuration → Environment variables**, add `PARTNER_SLUG` = `<slug>`. That single variable is what makes this site a different edition.
> 5. Under **Change site name**, set `<slug>-concierge`.
> 6. Deploy.

`ANALYTICS_DATABASE_URL` is usually inherited from a shared/team variable — check before asking them to add it. Notes worth holding on to:

- Variables take effect only on a **new deploy**, and on Netlify Pro a variable must have **Functions** in scope to be readable by a function.
- A site-level variable **overrides** a shared one of the same name, so don't add one "to be safe".
- **No Gemini key is needed.** The concierge endpoint is absolute (`extra.askEndpoint`), so every edition inherits one key. The copy of `ask.mjs` on an edition's site is never called.

Then verify from outside:

```
curl -s https://<slug>-concierge.netlify.app/ | grep -o '<title>[^<]*</title>'
curl -s -X POST -H 'Content-Type: application/json' -d '{"events":[]}' \
  https://<slug>-concierge.netlify.app/.netlify/functions/log-event
```

The title should be the customer's. The log call should answer `Provide 1-500 events` — `no database configured` means the analytics variable is not reaching the function.

# Step 5 — Publish

Follow `/whatsnext:publish` (typecheck → export → verify → commit → push). Every Netlify site built from the repo redeploys, so the edition picks the change up automatically.

Keep `DEVELOPMENT.md` current when the partner **system** changes; adding an individual customer is data and needs no doc change.

# Removing an edition

Delete the entry from `PARTNERS` / `DEMO_PARTNERS`, its logo, its icon row in `make-pwa-icons.mjs`, `public/icons/<slug>-*.png`, `public/og-<slug>.png`, and any `manifest-<slug>.webmanifest`. Then **ask the user to delete the edition's Netlify site** — nothing in the repo can. Leaving it up serves a stale copy at its own address forever.

# Known gaps

Say so plainly when a customer asks for these; do not promise them.

- **Edition-only descriptions that follow a place everywhere** — see "Customizing a place's description". Only the menu row is overridable today.
- **Promoting an edition's custom description to WNMV globally** — expected to be wanted; not built.
- **Native (App Store) builds** — editions are web apps installed to the home screen. A bundled native font (`fontHeadingNative`) is wired but inactive until a native build adds the face.
