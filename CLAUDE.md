# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A fully offline shift scheduling calendar for ~30 team members, delivered as a **single `index.html`** file with no build step, no dependencies, and no external URLs. Opens directly from `file://` in any modern browser.

- Live: https://fearlessresponsesolution.github.io/calendar/
- Remote: `git@github.com:fearlessresponsesolution/calendar.git` (SSH only — no tokens)

## Commands

**Run the app:**
```bash
wslview index.html          # open in Windows browser from WSL
python3 -m http.server 8080 # or serve locally, then visit http://localhost:8080
```

**Run tests:**
```bash
python3 -m http.server 8080
# then open http://localhost:8080/tests/index.html in a browser
# results render directly in the page — no CLI runner
```

There is no build, lint, or compile step.

## Architecture

Everything lives in `index.html` (~1370 lines): `<style>` → CSS, `<script>` → all JS. This single-file constraint is intentional and must be preserved — no imports, no CDN, no modules.

### State & persistence
One plain JS object (`state`) is the single source of truth. It loads from **IndexedDB** (`shift_cal_db` / `kv` store, key `shift_cal_v1`) on startup and is written back on every mutation via `saveState()`. localStorage is a silent fallback only. `saveState()` is fire-and-forget async — never await it inline. `loadState()` is async — `DOMContentLoaded` must `await loadState()` before calling `render()`.

### Render cycle
`render()` wipes and rebuilds all of `#app` innerHTML on every state change — no diffing, no virtual DOM. It is the **only** place `_lastConflicts` is refreshed (`_lastConflicts = detectConflicts(state)`). Panel renderers (`renderConflictsPanel`, `renderWorkloadPanel`) must read `_lastConflicts` — never call `detectConflicts()` inside them. After `render()`, any open panels are re-rendered.

### Conflict detection
`detectConflicts(st)` loops the current view-month, checking each shift assignment against `appointments[memberId]`. It also checks the next calendar day for midnight-spanning shifts (end ≤ start). `getAvailableSwaps()` mirrors this same next-day logic for swap candidates — keep them in sync. `rangesOverlap()` handles midnight-spanning by adding 1440 to the end time; do not duplicate this logic elsewhere.

### Test harness
`tests/index.html` duplicates utility functions from `index.html` verbatim (no module system on `file://`). When changing `rangesOverlap`, `getAvailableSwaps`, `detectConflicts`, or any other tested utility, update both files manually and keep the 31 tests passing.

## Key Rules

- **Deep-clone assignments on copy:** `assignments: [...s.assignments]` — `{ ...shift }` is shallow and shares the array.
- **`_lastConflicts` only updates in `render()`** — mutations that skip `render()` (e.g. `updateMemberColor`) leave the cache stale.
- **Color picker:** `onchange` only, never `oninput` — `oninput` fires every animation frame and spams `saveState()`.
- **Escape key** must call `endTour()` when `_tourStep >= 0` before `closeModal()`.
- **`deleteRole`** sets `member.roleId = ''` but does not purge members from `shift.assignments`.
- **`timeToMinutes('')`** returns `NaN` — validate times are non-empty before calling `rangesOverlap`.

See `.claude/rules/` for full detail on architecture, data model, code patterns, known issues, and hard prohibitions.
