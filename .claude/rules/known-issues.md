# Known Issues (as of 2026-05-30)

These are confirmed bugs not yet fixed. Do not introduce workarounds elsewhere that depend on this broken behavior.

1. **`_saveAppt` empty startTime** — allows saving a timed appointment with `startTime = ''`; `timeToMinutes('')` → NaN silently breaks conflict detection for that appointment.
2. **`_repeatWeekForMonth` no overwrite guard** — invoking twice doubles all propagated shifts silently. `_copyShifts` has the guard; `_repeatWeekForMonth` does not.
3. **`_copyShifts` dead confirm guard** — `state.schedule[target] = []` runs before the `?.length` check, so the confirmation is never shown for new-target days.
4. **`deleteRole` leaves shift assignments dirty** — sets `roleId = ''` on members but does not remove them from `shift.assignments`.
5. **Role-less member swap collision** — two members with `roleId = ''` match each other as swap candidates.
6. **`_lastConflicts` staleness** — any mutation path that skips `render()` leaves the cache stale.
7. **`renderConflictsPanel` redundant scan** — calls `detectConflicts(state)` directly instead of reading `_lastConflicts`.
8. **`loadState` shallow merge** — `{ ...DEFAULT_STATE, ...saved }` drops new nested keys on schema upgrades.
9. **Appointment modal missing backdrop guard** — clicking outside silently discards unsaved appointment data; day editor has this guard, appointment modal does not.
