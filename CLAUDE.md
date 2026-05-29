# FamilySchedule Project

## Purpose
Single source of truth for family schedule. Feeds meal planning and SMS assistant.

## Data File
- `schedule.json` — standing weekly activities + weekly overrides

## Schema

### `standing`
Recurring weekly activities keyed by day. Each entry:
- `person` — family member
- `activity` — name of activity
- `start` / `end` — 24h time strings
- `frequency` — omit if weekly; `"biweekly"` with `frequency_note` for alternating weeks
- `note` — optional free text

### `weekly_overrides`
One-off changes keyed by ISO date (`YYYY-MM-DD`). Each entry:
- `person`
- `note` — free text description

Example:
```json
"weekly_overrides": {
  "2026-04-22": [
    { "person": "Parent1", "note": "Work trip, back Thursday" }
  ]
}
```

## Integration Points

### Meal Planning (MenuBuilder)
- Read `standing` and `weekly_overrides` for the target week; MenuBuilder reasons about evening constraints directly from activity times and notes
- Check `weekly_overrides` for the target week before generating a plan

### SMS Assistant (future)
- Read: query schedule for a given day or week
- Write: add weekly overrides via SMS (e.g. "Add that Parent2 is out Thursday")

## Updating the Schedule
- Standing changes (new activity, time change): edit `standing` in `schedule.json`
- One-off weekly changes: add to `weekly_overrides` keyed by date
- Remove overrides after the week passes to keep the file clean
