# Data Model

Stored in `state` object, persisted to IndexedDB key `shift_cal_v1`:

```js
{
  members: [{ id, name, color, roleId }],
  roles: [{ id, name }],
  shiftTemplates: [{ id, name, startTime, endTime }],  // "HH:MM" 24h
  schedule: {
    "YYYY-MM-DD": [{ id, templateId, startTime, endTime, assignments: [memberId], isAdHoc }]
  },
  appointments: {
    "<memberId>": [{ id, date, startTime, endTime, allDay, note }]
  },
  settings: { viewYear, viewMonth }  // viewMonth is 0-indexed
}
```

## Key decisions
- `schedule` keyed by ISO date string — O(1) per-day lookup
- `assignments` stores member IDs only — resolved at render time
- Editing a template does NOT retroactively change existing scheduled shifts
- Deleting a template sets `isAdHoc: true` and nulls `templateId` on existing shifts
- `loadState` shallow merge (`{ ...DEFAULT_STATE, ...saved }`) drops new nested keys on schema upgrades — known limitation
