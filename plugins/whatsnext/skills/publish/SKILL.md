---
name: publish
description: Verify and ship changes to the What's Next MV guide app. Use whenever the user types /whatsnext:publish or asks to "publish the guide", "ship these changes", "push the guide update", "deploy the app", or after any edit to the app's data or code that needs to go live. Runs the standard tail — typecheck, regenerate the exported data, verify in the preview, commit, and push to main (which auto-deploys to Netlify). Other whatsnext skills call this at the end; it can also be run on its own after manual edits.
---

# Publish — the verify-and-ship tail

Every change to What's Next MV goes out the same way. This skill is that routine, and every other `whatsnext:` skill ends by running it. Never push without completing it.

## Project root

```
$HOME/Documents/mv-guide
```

Repo `alexjs1/whats-next-mv`, branch `main`. Pushing to `main` triggers the Netlify deploy automatically; there is no separate deploy step.

## Steps

1. **See what changed.**
   ```
   git -C "$HOME/Documents/mv-guide" status --short
   ```
   Know which files you're shipping before going further.

2. **Typecheck.** This is the main safety net for data edits (a malformed record fails here).
   ```
   cd "$HOME/Documents/mv-guide" && npx tsc --noEmit
   ```
   Fix anything it reports before continuing.

3. **Regenerate the exported guide data** if any `src/data/**` file changed (the offline concierge function reads this snapshot):
   ```
   node scripts/export-guide-data.mjs
   ```
   Confirm the printed counts (places / ferries / events) moved the way you expect.

4. **Verify in the preview** — do not skip this, and never ask the user to check manually. Use the preview tools:
   - Start the server (`preview_start`, config `vineyard-guide-web`, port 8081) if one isn't running.
   - Exercise the surface you actually changed: a data edit → find the place/event on the Guide or Info tab or in the Ask concierge; a map/category change → the Map filters; a concierge change → run the affected question in the Ask tab.
   - Check `preview_console_logs` (level error) is clean.
   - Capture one screenshot or text snippet as proof.

5. **Update the living docs — automatic, not on request.** If this change is
   anything more than a pure content tweak (a new or changed feature, data
   model, concierge intent, category, convention, command, or gotcha), reflect
   it in the docs **in this same commit**. The user should never have to ask a
   session to do this.
   - `DEVELOPMENT.md` (tracked) — the technical handbook: architecture,
     concierge design, chip taxonomy, data provenance & refresh recipes,
     editorial conventions, mistakes-not-to-repeat. Amend the relevant section.
   - `APP_OVERVIEW.md` (tracked) — the plain-language overview for a
     slightly-technical reader. Keep it in sync at a high level (what the app
     now does, and the stack table if the stack changed).
   Other sessions edit these too, so make **targeted additions to the right
   section, never a blind overwrite**. A pure content edit (one place's hours,
   a new event) needs no doc change — use judgment.

6. **Commit** with a clear message and the standard trailer:
   ```
   git commit -m "<subject>

   <what changed and why>

   Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
   ```
   Stage the files this change touches (the code/data edits **plus** any
   `DEVELOPMENT.md` / `APP_OVERVIEW.md` updates from step 5). Prefer explicit
   `git add <paths>` over a blind `git add -A` so unrelated in-progress files
   (see guardrails) don't get swept in.

7. **Push:**
   ```
   git push origin main
   ```
   Tell the user it's pushed and that Netlify is deploying.

## Guardrails

- `DEVELOPMENT.md` and `APP_OVERVIEW.md` are **living, tracked docs** — keep them current per step 5, never leave them stale.
- Genuinely local working files (`TRIP_PLANNER.md`, `description-audit.xlsx`) are intentionally untracked. Don't sweep them in with a blind `git add -A`.
- If typecheck or the preview surfaces a real problem, fix it and re-run from step 2. Report failures honestly rather than pushing around them.
