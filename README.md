# FamilySchedule

A single source of truth for a busy family's weekly schedule — built to feed a meal planning system, an SMS assistant, and anything else that needs to know who's where and when.

## What It Does

**Standing schedule:** Captures recurring weekly activities per person — sports practices, school hours, work schedules — with start/end times and a flag for whether each activity affects dinner timing.

**Weekly overrides:** One-off changes keyed by date — game days, tournaments, work trips, school events — layered on top of the standing schedule without modifying it.

**Dinner constraint signaling:** Every entry carries an `affects_dinner` boolean. Downstream tools (meal planner, SMS assistant) query this to know which nights need quick meals, later starts, or simpler logistics.

**Calendar integration:** Game entries include jersey color instructions pulled from the team's ICS feed, so the schedule doubles as a game-day reference.

## Design Decisions

**JSON as single source of truth.** One file, two sections: `standing` for recurring patterns and `weekly_overrides` for one-offs. Downstream tools read the same file — no syncing, no duplication.

**Overrides layer, not replace.** Standing schedule stays clean. Weekly exceptions are keyed by ISO date and removed after the week passes. The file stays small and readable year-round.

**Affects-dinner as the key signal.** Rather than exposing full schedule details to every downstream tool, a single boolean surfaces what most tools actually need — whether a given night is constrained. The meal planner uses this to avoid complex recipes on busy nights; the SMS assistant uses it to answer "what's happening this week."

**People as a flat list.** No nested family structure — just a `people` array and a `person` field on each entry. Simple to query, easy to extend.

**ICS pull for game details.** Jersey colors and arrive-by times come from the team's PlayMetrics calendar feed rather than being manually maintained. Game entries in `weekly_overrides` are populated from that feed, so the schedule reflects what the team app publishes.

**Privacy boundary between AI and personal calendars.** Rather than granting Claude Code direct access to iCloud Calendar or Google Calendar, `schedule.json` is the only scheduling data the AI ever sees. It contains exactly what downstream tools need — activity names, times, and dinner impact — and nothing else. No event descriptions, no attendees, no location history, no calendar metadata. The file is also gitignored, so it never leaves the local machine.

## Repository Structure

```
schedule.example.json   # Schema reference with placeholder data (real file is local, not committed)
CLAUDE.md               # AI assistant context and workflow instructions
```

## Usage

Copy `schedule.example.json` to `schedule.json` and update with your family's data:

```bash
cp schedule.example.json schedule.json
```

Query examples (via Claude Code or any JSON tool):

```bash
# What's happening this Saturday?
# Check weekly_overrides for the date, then standing for the day of week

# Which nights this week affect dinner?
# Filter both standing and weekly_overrides for affects_dinner: true
```

## Schema

### `standing`

Recurring weekly activities keyed by day:

```json
"Monday": [
  {
    "person": "Child1",
    "activity": "Soccer practice",
    "start": "19:00",
    "end": "20:00",
    "affects_dinner": true,
    "note": "Start dinner by 5:30 PM"
  }
]
```

Optional fields: `frequency` (`"biweekly"`), `frequency_note`, `note`

### `weekly_overrides`

One-off changes keyed by ISO date:

```json
"2026-04-25": [
  {
    "person": "Child2",
    "note": "Tournament, Oxford OH — game times TBD",
    "affects_dinner": true
  }
]
```

Remove entries after the week passes to keep the file clean.

## Integration Points

**MenuBuilder** — before generating the weekly meal plan, reads the standing schedule and that week's overrides to identify constrained nights. Nights with `affects_dinner: true` get flagged for quick or flexible meals. Without this, the meal planner has no awareness of whether a given night has an early practice, a late game, or a parent traveling — it would propose recipes regardless of whether anyone has time to cook them.

**SMS Assistant** — queries `schedule.json` to answer questions like "what's happening Thursday?" or "does anything affect dinner this week?" Also writes back to `weekly_overrides` from natural language input ("Add that Parent2 is out Thursday"), keeping the file current without manually editing JSON. The assistant never connects to the family's actual calendars — FamilySchedule is the only scheduling context it has access to.

Both integrations read the same local file. There is no API, no sync service, and no shared database — just a JSON file that Claude Code knows how to read and update.
