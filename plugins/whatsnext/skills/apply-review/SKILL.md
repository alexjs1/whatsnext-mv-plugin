---
name: apply-review
description: Apply a filled-in What's Next MV review spreadsheet — the one the automated weekly and seasonal refreshes email to Alex. Use whenever the user types /whatsnext:apply-review, attaches or points to a "WNMV review" .xlsx, or says "apply this review", "here's the review sheet back", "make the approved changes", "process my decisions". Reads each row's Decision (Approve, Approve with my edit, Reject, or blank), makes the approved changes, records every decision so rejected changes are not proposed again, and publishes once.
---

# Apply a returned review sheet

The automated refreshes never edit the guide. They queue what they find in
`review/pending.json` and email it as a spreadsheet; Alex marks a Decision on
each row and hands it back. This skill makes the approved changes. The sheet
**is** the approval, so do not re-ask per row; do ask if a Notes cell changes
the intent of a row or is unclear.

The full protocol, including what each payload op means, is in
`review/README.md` in the app repo. Read it first.

## Project root

```
$HOME/Documents/mv-guide
```

## Step 1 — Get the sheet and the latest queue

Find the .xlsx the user gave you (an attachment path or a file they name;
Downloads is the usual place). Then pull, because the routine pushes the queue:

```
git -C "$HOME/Documents/mv-guide" pull --rebase origin main
```

## Step 2 — Read the decisions

```
cd "$HOME/Documents/mv-guide" && node scripts/review-sheet.mjs read "<path to xlsx>"
```

It prints `decided` (each with the queued item and its payload), `undecided`
(blank Decision; they stay in the queue for the next sheet), and
`alreadyHandled` (rows from an older sheet that were decided since; ignore
them). A malformed row (an unknown decision, or "Approve with my edit" with no
edit) stops the read with the row number; tell the user which row to fix.

Show the user a short summary before changing anything: how many to apply, how
many rejected, how many left for later, and any row whose Notes you are acting
on.

## Step 3 — Apply each approved row

Use the row's `payload`. For **Approve with my edit**, the user's edit replaces
the proposed value (for an `editPlace`/`editEvent`/`editTransport`, it is the
new `value`; for an add, it corrects whatever the user evidently meant, and if
that is ambiguous, ask).

- `addPlace` — follow the add-place skill: geocode with Places
  `findplacefromtext` (the key is in `.env`), run the false-match check (name and
  town must match, on-island, within 500 m of any address the source gives), and
  write the record into the suggested file next to its siblings. If the geocode
  fails or the match is doubtful, do **not** add it; record it as not applied
  and tell the user why.
- `removePlace` — delete the record. Also remove its id from anything that
  references it (grep the id: `status.ts`, `ownership.ts`, `menuTags.ts`,
  `dishTags.ts`, featured lists, partner editions).
- `editPlace` / `editEvent` — set the one field.
- `addEvent` / `removeEvent` — follow the add-event skill / delete the record;
  keep `events.ts` in date order.
- `editTransport` — set the value at `path` in `src/data/transport.ts`.
- `manual` — follow the instructions; these are the structural ones (a new
  Steamship period, a new bus route).
- `check` — nothing to change. Approving means the user confirmed it by hand.

Copy rules as always: curly apostrophes in strings, never `String.replace` with
a `$`-containing replacement, no prices in descriptions.

## Step 4 — Record the decisions

Write every decided row (applied, rejected, or failed) to a scratch file as
`[{ "key", "decision", "edit", "notes", "applied": true|false }]` and run:

```
cd "$HOME/Documents/mv-guide" && node scripts/review-sheet.mjs record <file>
```

This clears them from `pending.json` and logs them in `decisions.json`, which is
how a rejected change stays rejected. A row that could not be applied is
recorded with `"applied": false`; leave it out instead if the user wants it to
stay in the queue for next time.

## Step 5 — Publish

Run `/whatsnext:publish`, staging the data files you changed plus
`review/pending.json` and `review/decisions.json` (stage files by name, never
`git add -A`). Commit message: `Apply review YYYY-MM-DD: N applied, N rejected`,
with the applied changes listed in the body.

## Step 6 — Report

Tell the user what went live, what was rejected, anything that could not be
applied and why, and how many rows are still open for the next sheet.
