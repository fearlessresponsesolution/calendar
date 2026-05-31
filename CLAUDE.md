# Shift Management Calendar — Project Context

## What This Is

A fully offline, self-contained shift scheduling calendar for ~30 team members.
Delivered as a **single `index.html`** file. No build step, no dependencies, no
external URLs. Opens directly from `file://` in any modern browser.

GitHub: https://github.com/fearlessresponsesolution/calendar  
Remote: `git@github.com:fearlessresponsesolution/calendar.git` (SSH)

---

## Architecture

### Single-file constraint
Everything lives in `index.html`: all CSS in `<style>`, all JS in `<script>`.
No imports, no modules, no CDN. This is intentional and must be preserved.

### State management
- One plain JS object (`state`) loaded on page load, written on every mutation
- **Primary store: IndexedDB** (`shift_cal_db` / `kv` store, key `shift_cal_v1`)
- localStorage is the fallback only — IndexedDB survives "Clear browsing data"
- `saveState()` is fire-and-forget async. Do not await it inline.
- `loadState()` is async — the DOMContentLoaded handler must `await` it before `render()`

### Rendering
- `render()` redraws the entire `#app` innerHTML on every state change
- `render()` is also where `_lastConflicts = detectConflicts(state)` is computed — **only here**
- After `render()`, panels that are open are re-rendered: `renderConflictsPanel()`, `renderWorkloadPanel()`
- Do not call `detectConflicts()` directly inside panel renderers — use `_lastConflicts`

### Key module-level state (transient, never persisted)
- `_lastConflicts` — cached conflict array, set at top of `render()`
- `_conflictsPanelOpen` — boolean sidebar toggle
- `_workloadPanelOpen` — boolean sidebar toggle
- `_dm` — day modal transient state (`{ dateStr, editId, adding, templateId, startTime, endTime, assignments, memberSearch }`)
- `_am` — appointment modal transient state
- `_tourStep`, `_tourSpotlight`, `_tourCard` — tour state
- `_db` — cached IndexedDB connection

---

## Data Model (stored in `state`)

```js
{
  members: [{ id, name, color, roleId }],
  roles: [{ id, name }],
  shiftTemplates: [{ id, name, startTime, endTime }],  // "HH:MM" 24h
  schedule: { "YYYY-MM-DD": [{ id, templateId, startTime, endTime, assignments: [memberId], isAdHoc }] },
  appointments: { "<memberId>": [{ id, date, startTime, endTime, allDay, note }] },
  settings: { viewYear, viewMonth }  // viewMonth is 0-indexed
}
```

---

## Critical Rules Learned

### Data integrity
- **Always deep-clone `assignments` when copying shifts**: use `assignments: [...s.assignments]`, never `{ ...shift }` alone — the spread is shallow and shares the array reference.
- **`_repeatWeekForMonth` needs an overwrite check** — currently missing, can silently double shifts on repeat invocation. `_copyShifts` has it; `_repeatWeekForMonth` does not.

### Conflict detection
- `detectConflicts()` only checks the current `viewMonth`. Cross-month conflicts are invisible.
- Night shifts (e.g. 22:00–06:00): the code checks next-day appointments via `isoDate(nextDay)`. This logic exists in BOTH `detectConflicts` and `getAvailableSwaps` — keep them in sync.
- `rangesOverlap()` handles midnight-spanning by adding 1440 to the end time when `endTime <= startTime`. Do not duplicate this logic.
- `timeToMinutes('')` returns `NaN` — all comparisons fail silently. Always validate `startTime`/`endTime` are non-empty before calling `rangesOverlap`.

### Role deletion edge case
- Deleting a role sets affected `member.roleId = ''` (empty string)
- Two role-less members (both `roleId: ''`) will match each other in `getAvailableSwaps` (`m.roleId !== conflicted.roleId` → `'' !== ''` → false = they DO match). This can produce incorrect swap recommendations.
- `deleteRole` does NOT clean up `shift.assignments` — deleted-role members remain in shifts.

### `_lastConflicts` cache staleness
- `_lastConflicts` is only refreshed inside `render()`.
- Functions that mutate state but skip `render()` (e.g., `updateMemberColor`, `toggleConflictsPanel`) leave `_lastConflicts` stale.
- `renderConflictsPanel()` calls `detectConflicts(state)` directly instead of reading `_lastConflicts` — this means the panel and the header badge can briefly diverge.

### saveState during migration
- In `loadState()`, `saveState()` is called then `localStorage.removeItem()` immediately follows.
- `saveState()` is async (fire-and-forget). If the tab closes between these two lines, data is gone from both stores.

### UI patterns
- **Color picker**: use `onchange`, never `oninput` — `oninput` fires on every animation frame and spams `saveState()`.
- **Context menu**: clamp to `window.innerWidth/innerHeight` — right-clicks near viewport edges overflow.
- **Backdrop click guard**: the unsaved-changes confirmation (`_renderDayModal`) is NOT wired to the appointment modal (`_renderApptModal`). Should be consistent.
- **Escape key**: must call `endTour()` if `_tourStep >= 0`, then `closeModal()`.

### Template editing
- Editing a shift template does NOT retroactively change existing scheduled shifts — shifts store their own `startTime`/`endTime` at creation time.
- Deleting a template marks existing shifts as `isAdHoc: true` and nulls `templateId`.

---

## Code Patterns

### Generating IDs
```js
generateId()  // Date.now().toString(36) + Math.random().toString(36).slice(2)
```

### Safe HTML rendering
```js
escapeHtml(s)  // always apply to user-entered strings rendered as innerHTML
// DOM methods (textContent, createElement) are safe without escaping
```

### Date helpers
```js
isoDate(year, month, day)  // month is 0-indexed → outputs "YYYY-MM-DD"
getDaysInMonth(year, month)
formatTime("HH:MM")  // → "9:00am" / "2:30pm"
timeToMinutes("HH:MM")  // NaN if input is empty/invalid
rangesOverlap(s1, e1, s2, e2)  // handles midnight-spanning correctly
```

### Next-day calculation (repeated 6+ times — candidate for extraction)
```js
const nd = new Date(dateStr + 'T12:00:00');
nd.setDate(nd.getDate() + 1);
const nextDateStr = isoDate(nd.getFullYear(), nd.getMonth(), nd.getDate());
```

### Midnight-spanning check (repeated — candidate for extraction)
```js
timeToMinutes(shift.endTime) <= timeToMinutes(shift.startTime)
```

---

## Known Issues (as of 2026-05-30)

1. `_saveAppt` allows saving a timed appointment with an empty `startTime` — `timeToMinutes('')` → NaN, silently breaks conflict detection for that appointment.
2. `_repeatWeekForMonth` has no overwrite confirmation — invoking twice doubles all shifts.
3. `_copyShifts` confirm guard (`state.schedule[target]?.length`) checks after `state.schedule[target] = []` is already set, making the guard dead code for new-target days.
4. `deleteRole` sets `roleId = ''` but doesn't purge member from shift assignments.
5. Two role-less members (roleId='') incorrectly match as swap candidates for each other.
6. `_lastConflicts` goes stale on any code path that mutates state without calling `render()`.
7. `renderConflictsPanel` re-runs `detectConflicts` instead of reading `_lastConflicts`.
8. `loadState` shallow merge (`{ ...DEFAULT_STATE, ...saved }`) drops new nested keys on schema upgrades.
9. Appointment modal has no backdrop-click unsaved-changes guard (day editor does).

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire application (~1370 lines) |
| `tests/index.html` | Standalone unit test harness (31 tests) — must be kept in sync with index.html manually |
| `README.md` | User-facing guide |
| `docs/superpowers/specs/2026-05-30-shift-calendar-design.md` | Original design spec |
| `docs/superpowers/plans/2026-05-30-shift-calendar.md` | 10-task implementation plan |

---

## Git / GitHub

- Branch: `main`
- Remote: `git@github.com:fearlessresponsesolution/calendar.git`
- Push via SSH (key in `~/.ssh/id_ed25519`)
- GitHub Pages enabled at `https://fearlessresponsesolution.github.io/calendar/`
- All commits include `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

---

## What NOT to Do

- Do not add external URLs, CDN links, or `import` statements — the single-file offline constraint is non-negotiable.
- Do not use `localStorage` as the primary store — IndexedDB is the primary, localStorage is fallback only.
- Do not call `detectConflicts()` from inside `renderConflictsPanel()` or `renderWorkloadPanel()` — use `_lastConflicts`.
- Do not use `oninput` on color pickers — use `onchange`.
- Do not shallow-spread shift objects when copying — always deep-clone `assignments`.
- Do not commit tokens or credentials — use SSH for git operations.
