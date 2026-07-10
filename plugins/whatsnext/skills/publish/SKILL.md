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

5. **Commit** with a clear message and the standard trailer:
   ```
   git add -A && git commit -m "<subject>

   <what changed and why>

   Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
   ```
   If the working tree has unrelated in-progress files (design docs, an audit spreadsheet), stage only the files this change touches rather than `-A`.

6. **Push:**
   ```
   git push origin main
   ```
   Tell the user it's pushed and that Netlify is deploying.

## Guardrails

- Local working docs (`TRIP_PLANNER.md`, `APP_OVERVIEW.md`, `description-audit.xlsx`) are intentionally untracked / gitignored. Don't sweep them in with a blind `git add -A`.
- If typecheck or the preview surfaces a real problem, fix it and re-run from step 2. Report failures honestly rather than pushing around them.
