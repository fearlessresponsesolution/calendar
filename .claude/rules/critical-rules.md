# Critical Rules

## Data integrity
- **Always deep-clone `assignments` when copying shifts**: `assignments: [...s.assignments]`
  Never use `{ ...shift }` alone — the spread is shallow and shares the array reference.
- Copy operations (`_copyShifts`, `_repeatWeekForMonth`) must warn before overwriting existing shifts.

## Conflict detection
- `detectConflicts()` only checks the current `viewMonth` — cross-month conflicts are invisible.
- Night shifts (22:00–06:00): check next-day appointments via `isoDate(nextDay)`. This logic must stay in sync across `detectConflicts` AND `getAvailableSwaps`.
- `rangesOverlap()` handles midnight-spanning by adding 1440 when `endTime <= startTime`. Do not duplicate this logic.
- `timeToMinutes('')` returns `NaN` — all comparisons fail silently. Always validate times are non-empty before calling `rangesOverlap`.

## Role deletion
- Deleting a role sets affected `member.roleId = ''` (empty string).
- Two role-less members (`roleId: ''`) match each other in `getAvailableSwaps` — incorrect swap recommendations result.
- `deleteRole` does NOT purge members from `shift.assignments` — deleted-role members remain in shifts.

## `_lastConflicts` cache
- Refreshed only inside `render()`. Any mutation that skips `render()` (e.g. `updateMemberColor`) leaves it stale.
- `renderConflictsPanel()` must read `_lastConflicts`, not call `detectConflicts()` directly.

## UI patterns
- **Color picker**: use `onchange`, never `oninput` — `oninput` fires every frame and spams `saveState()`.
- **Context menu**: clamp position to `window.innerWidth/innerHeight` — right-clicks near edges overflow.
- **Backdrop click guard**: `_renderDayModal` has it; `_renderApptModal` does not — inconsistency.
- **Escape key**: must call `endTour()` if `_tourStep >= 0`, then `closeModal()`.

## saveState during migration
- In `loadState()`, `saveState()` fires async then `localStorage.removeItem()` runs immediately.
- If the tab closes between those two lines, data is lost from both stores.
