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
- `affects_dinner` — boolean, used by meal planning to flag timing constraints
- `frequency` — omit if weekly; `"biweekly"` with `frequency_note` for alternating weeks
- `note` — optional meal planning hint

### `weekly_overrides`
One-off changes keyed by ISO date (`YYYY-MM-DD`). Each entry:
- `person`
- `note` — free text description
- `affects_dinner` — boolean

Example:
```json
"weekly_overrides": {
  "2026-04-22": [
    { "person": "Parent1", "note": "Work trip, back Thursday", "affects_dinner": true }
  ]
}
```

## Integration Points

### Meal Planning (MenuBuilder)
- Use `affects_dinner: true` entries to flag busy nights when running meal suggestions
- Check `weekly_overrides` for the target week before generating a plan

### SMS Assistant (future)
- Read: query schedule for a given day or week
- Write: add weekly overrides via SMS (e.g. "Add that Parent2 is out Thursday")

## Updating the Schedule
- Standing changes (new activity, time change): edit `standing` in `schedule.json`
- One-off weekly changes: add to `weekly_overrides` keyed by date
- Remove overrides after the week passes to keep the file clean
