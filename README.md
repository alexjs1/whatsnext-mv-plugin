# whatsnext plugin

Maintenance skills for the **What's Next MV** visitor-guide app (Expo / React Native).

- App repo: `alexjs1/whats-next-mv` (deploys to whatsnextmv.netlify.app on push to `main`)
- App root on disk: `$HOME/Documents/mv-guide`

Invoke any skill as `/whatsnext:<skill>`.

## Skills

| Skill | Purpose |
|---|---|
| `/whatsnext:start` | Front door. Reports guide state and routes to the right skill. |
| `/whatsnext:weekly-refresh` | The weekly cycle: new/closed/changed places + events, ratings, publish. |
| `/whatsnext:add-place` | Add a place: verify, geocode, write the record, publish. |
| `/whatsnext:add-event` | Add an event to the calendar. |
| `/whatsnext:refresh-ratings` | Google-ratings-only refresh. |
| `/whatsnext:audit-descriptions` | Accuracy sweep of place descriptions against real sources. |
| `/whatsnext:concierge-check` | Find gaps in the offline concierge's coverage. |
| `/whatsnext:seasonal-refresh` | Roll event dates to the new year; re-check seasonal windows. |
| `/whatsnext:publish` | The shared tail: typecheck, export, verify, commit, push. |

## Install

This repo is its own single-plugin marketplace (`.claude-plugin/marketplace.json`).

- **Claude Code:** `/plugin marketplace add alexjs1/whatsnext-plugin` then `/plugin install whatsnext@whatsnext-marketplace`.
- **Cowork (desktop):** enable it from Customize → Workspace, the same way the `toybox` plugin was installed.

## Conventions every skill follows

- After any data edit: `npx tsc --noEmit` → `node scripts/export-guide-data.mjs` → verify in the preview → commit → push.
- App copy does **not** follow the Island Analytics writing rules. Be clear and factual.
- Verify factual claims against real sources; never fabricate. If a claim can't be verified, keep the copy soft or leave it out.
- The Google API key is read from the environment / the app's `.env`. Never commit it.
