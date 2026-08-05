# whatsnext plugin

Maintenance skills for the **What's Next MV** visitor-guide app (Expo / React Native).

- App repo: `alexjs1/whats-next-mv` (deploys to whatsnextmv.com on push to `main`)
- App root on disk: `$HOME/Documents/mv-guide`

Invoke any skill as `/whatsnext:<skill>`.

## Skills

### The guide

| Skill | Purpose |
|---|---|
| `/whatsnext:start` | Front door. Reports guide state and routes to the right skill. |
| `/whatsnext:weekly-refresh` | The weekly cycle: new/closed/changed places + events, ratings, publish. |
| `/whatsnext:add-place` | Add a place: verify, geocode, write the record, publish. |
| `/whatsnext:add-event` | Add an event to the calendar. |
| `/whatsnext:refresh-ratings` | Google-ratings-only refresh. |
| `/whatsnext:audit-descriptions` | Accuracy sweep of place descriptions against real sources. |
| `/whatsnext:concierge-check` | Find gaps in the offline concierge's coverage. |
| `/whatsnext:research-tag` | Research a hand-curated menu filter (ribs, vegan, vegetarian, gluten-free). |
| `/whatsnext:seasonal-refresh` | Roll event dates to the new year; re-check seasonal windows. |

### Selling a listing

| Skill | Purpose |
|---|---|
| `/whatsnext:feature-place` | Feature a business (top ranking + gold map star + custom description). |
| `/whatsnext:feature-to-top` | Just the top-of-category ranking + "Featured" tag. |
| `/whatsnext:feature-star` | Just the gold star on the map. |
| `/whatsnext:customize-description` | Just a custom description (≤250 chars). |

### Hotel editions (Pocket Concierge)

| Skill | Purpose |
|---|---|
| `/whatsnext:hotel-edition` | The whole thing end to end: branded edition + curated concierge + its own site. |
| `/whatsnext:private-label` | Just the branded edition: logo, colors, font, contact, Favorites tab, lodging removed. |
| `/whatsnext:pocket-concierge` | Just the curated front door: hand-picked dining, shopping, landmarks, heritage, can't-miss. |

An edition lives at **its own web address** (`<slug>-concierge.netlify.app`), on its own
Netlify site built from the same repo. That is what stops editions colliding with the
island guide on a phone. `private-label` walks you through creating it; the legacy
`/h/<slug>` path still works for links already shared.

### Shipping

| Skill | Purpose |
|---|---|
| `/whatsnext:publish` | The shared tail: typecheck, export, verify, commit, push. |

## Install

This repo is its own single-plugin marketplace (`.claude-plugin/marketplace.json`).

- **Claude Code:** `/plugin marketplace add alexjs1/whatsnext-mv-plugin` then `/plugin install whatsnext@whatsnext-mv`.
- **Cowork (desktop):** enable it from Customize → Workspace, the same way the `toybox` plugin was installed.

## Updating

The plugin manifest deliberately declares **no `version`**, so every commit counts as a
new version and a push is all it takes to ship a change.

That was not always true. While `plugin.json` pinned `"version": "0.7.0"`, pushing
commits without bumping that string did nothing for anyone who already had the plugin —
Claude Code saw the same version and kept its cached copy. That is why updates appeared
to require deleting and reinstalling. See
[version resolution](https://code.claude.com/docs/en/plugin-marketplaces#version-resolution-and-release-channels).

To pull the latest:

- **Claude Code:** `/plugin marketplace update` then `/plugin update whatsnext`.
- **Cowork:** refresh the plugin from Customize → Workspace.

If a version is ever reintroduced, it must be bumped on **every** release, and it must
live in only one place: `plugin.json` wins over the marketplace entry silently, so a
stale manifest can mask a version set in `marketplace.json`.

## Conventions every skill follows

- After any data edit: `npx tsc --noEmit` → `node scripts/export-guide-data.mjs` → verify in the preview → commit → push.
- App copy does **not** follow the Island Analytics writing rules. Be clear and factual.
- Verify factual claims against real sources; never fabricate. If a claim can't be verified, keep the copy soft or leave it out.
- The Google API key is read from the environment / the app's `.env`. Never commit it.
- Keep `DEVELOPMENT.md` and `APP_OVERVIEW.md` in the app repo current in the same commit as the change.
