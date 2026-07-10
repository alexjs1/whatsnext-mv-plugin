---
name: start
description: What's Next MV concierge — reports the current state of the guide app and routes to the right child skill. Use whenever the user types /whatsnext:start or asks "what's the state of the guide", "what should I do for What's Next MV", "where are we with the app", "give me the guide status", "anything pending for the MV app", or similar open-ended questions about the What's Next MV project. Also use when the user makes a natural-language maintenance request without naming a skill — e.g., "add the new taco place in Oak Bluffs", "refresh the ratings", "the street fair date changed" — and route to /whatsnext:add-place, /whatsnext:refresh-ratings, /whatsnext:add-event, etc. Do not use for the focused single-purpose skills below; those have their own triggers.
---

# What's Next MV concierge

The front door. The user types `/whatsnext:start` (or asks an open-ended question about the app) and this skill reports state and points to the next action. It does no editing itself.

## Project root

```
$HOME/Documents/mv-guide
```

Repo `alexjs1/whats-next-mv`, branch `main` → auto-deploys to whatsnextmv.netlify.app on push.

## Available child skills

| Skill | What it does |
|---|---|
| `/whatsnext:weekly-refresh` | The weekly cycle: sweep for new/closed/changed places + events, refresh ratings, publish |
| `/whatsnext:add-place` | Add a place: verify, geocode, write the record, publish |
| `/whatsnext:add-event` | Add an event to the calendar |
| `/whatsnext:refresh-ratings` | Google-ratings-only refresh (the mechanical part of the weekly cycle) |
| `/whatsnext:audit-descriptions` | Accuracy sweep of descriptions vs. real sources |
| `/whatsnext:concierge-check` | Find gaps in the offline concierge's coverage |
| `/whatsnext:seasonal-refresh` | Roll event dates to the new year; re-check seasonal windows |
| `/whatsnext:publish` | Typecheck, export, verify, commit, push |

The default weekly action is `/whatsnext:weekly-refresh`. If uncommitted work or a stale `ratings.ts` is sitting around, or it's been about a week, suggest it.

## Step 1 — Inspect state

Run these and summarize for the user:

```
git -C "$HOME/Documents/mv-guide" status --short
git -C "$HOME/Documents/mv-guide" log --oneline -5
```

Report the current content counts:

```
cd "$HOME/Documents/mv-guide" && node --input-type=module -e "import('./src/data/places.ts').then(m=>{const P=m.PLACES;const byCat={};for(const p of P)byCat[p.category]=(byCat[p.category]||0)+1;console.log('places:',P.length);console.log(byCat);})"
```

Note anything worth flagging: uncommitted changes, how long since the last `ratings.ts` refresh (`git log -1 --format=%cr -- src/data/ratings.ts`), whether it's near a season/year boundary (events dates rolling over).

## Step 2 — Route

- If the user's request maps to a child skill, hand off to it (invoke that skill).
- If they just wanted status, give the summary and stop.
- If uncommitted work is sitting in the tree, offer `/whatsnext:publish`.

Keep the summary short. This skill orients; the child skills do the work.
