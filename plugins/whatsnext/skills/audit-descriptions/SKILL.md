---
name: audit-descriptions
description: Audit the What's Next MV place descriptions for factual accuracy against real sources. Use whenever the user types /whatsnext:audit-descriptions or asks to "audit the descriptions", "fact-check the guide", "check the descriptions for accuracy", "find wrong or made-up details", or wants to apply a filled-in audit spreadsheet. Flags descriptions that make specific, falsifiable claims, verifies them on the web, produces a review spreadsheet (current / suggested / your edits), and applies the user's approved edits.
---

# Description accuracy audit

Many descriptions were originally generated and can carry unsourced specifics. This skill finds the checkable ones, verifies them, and fixes what's wrong, in two phases: **audit** (produce a spreadsheet) and **apply** (write the user's approved edits).

## Project root

```
$HOME/Documents/mv-guide
```

## Phase A — Audit

1. **Dump descriptions.**
   ```
   cd "$HOME/Documents/mv-guide" && node --input-type=module -e "import('./src/data/places.ts').then(m=>{const r=m.PLACES.map(p=>({id:p.id,name:p.name,town:p.town,category:p.category,description:p.description||''}));require('fs').writeFileSync('/tmp/alldesc.json',JSON.stringify(r));console.error('total',r.length);})"
   ```

2. **Flag the falsifiable ones.** Soft descriptions ("a cozy spot with great views") can't be wrong; only flag specific claims: years / "since" / founding, superlatives ("oldest/only/first/largest"), ownership or operator, "former X space", named affiliations, seasonal-move or day/hours claims, counts (acres/seats/rooms). A street name alone is low-risk; don't flag on that alone. This typically narrows ~370 places to a few dozen.

3. **Verify in parallel.** Split the flagged set into batches and launch a research agent per batch (general-purpose). Each reads its batch, web-verifies every claim against real sources (business site, Vineyard Gazette, MV Times, Trustees, town sites), and returns only the WRONG or UNVERIFIABLE ones with: `id, name, issue, current, suggested, source, confidence`. Rules for `suggested`: use the verified correct fact, or if a claim is merely unverifiable, drop it — never invent a replacement. Keep the same concise style/length.

4. **Build the spreadsheet** with openpyxl (`python3`): columns **Place · Town · Issue found · Current description · Suggested description · Your edits (final wording) · Source · Confidence**, worst-first, with the "Your edits" column highlighted for the user. Save to `$HOME/Documents/mv-guide/description-audit.xlsx` (gitignored; it's a working doc). Tell the user only the confirmed problems are listed; accurate ones were dropped.

## Phase B — Apply

When the user returns the spreadsheet with the "Your edits" column filled:

1. Read it back (openpyxl). For each row: use the user's edit if present; otherwise use the suggested text. Treat their notes as instructions to fold in.
2. Replace each description **value** in place. Locate the record by `id`, then swap the string after `description:`. Use curly apostrophes/quotes to avoid escaping the single-quote delimiter; **never** `String.replace` with a `$`-containing replacement. A small Python script that finds the id anchor and replaces the quoted value is the reliable way.
3. Run `/whatsnext:publish`. In the preview, spot-check one or two corrected descriptions render.
