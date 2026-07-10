---
name: add-event
description: Add an event to the What's Next MV events calendar. Use whenever the user types /whatsnext:add-event or asks to "add an event to the calendar", "put the fair or festival in the guide", "the guide is missing an event", add a parade, fireworks, or road race, or gives an event name and (optionally) a URL. Verifies this year's date, time, and location against real sources, inserts it chronologically into events.ts, then publishes.
---

# Add an event to the calendar

## Project root

```
$HOME/Documents/mv-guide
```

Events live in `src/data/events.ts` as an ordered `IslandEvent[]`, rendered on the Info tab's Events segment and answered by the concierge's event intent.

## Step 1 — Verify (this year's dates matter)

Confirm against real sources (the host's site, town site, Vineyard Gazette / MV Times calendar): the exact date(s), time window, and location for the **current** year. Dates shift year to year, so a stale date is worse than none. If the time isn't published, state the day without inventing a clock time.

## Step 2 — Write the entry

The `IslandEvent` shape is: `id`, `name`, `when`, `where`, `description`, `url`. Match the style of existing entries (see the file). Example:

```ts
{
  id: 'e-<slug>',
  name: '<Event name>',
  when: '<Weekday, Month D, YYYY, time window>',
  where: '<Place, Town>',
  description: '<One factual sentence.>',
  url: '<official/source URL>',
},
```

Use curly apostrophes/quotes in the strings; the file uses single-quote delimiters.

## Step 3 — Insert chronologically

The list is ordered by date (the file header says so). Find the two entries your event falls between (`grep -n "when:" src/data/events.ts`) and insert it in order. Keep the ordering correct so the Info tab reads top-to-bottom by date.

## Step 4 — Publish

Run `/whatsnext:publish`. In the preview, open the Info tab → Events segment and confirm the entry renders in the right spot, and that a concierge event question surfaces it.
