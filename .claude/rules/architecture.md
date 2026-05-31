# Architecture

## Single-file constraint
Everything lives in `index.html`: all CSS in `<style>`, all JS in `<script>`.
No imports, no modules, no CDN. This is intentional and must be preserved.

## State management
- One plain JS object (`state`) loaded on page load, written on every mutation
- **Primary store: IndexedDB** (`shift_cal_db` / `kv` store, key `shift_cal_v1`)
- localStorage is the fallback only — IndexedDB survives "Clear browsing data"
- `saveState()` is fire-and-forget async. Do not await it inline.
- `loadState()` is async — the DOMContentLoaded handler must `await` it before `render()`

## Rendering
- `render()` redraws the entire `#app` innerHTML on every state change
- `render()` is where `_lastConflicts = detectConflicts(state)` is computed — **only here**
- After `render()`, open panels are re-rendered: `renderConflictsPanel()`, `renderWorkloadPanel()`
- Do not call `detectConflicts()` inside panel renderers — use `_lastConflicts`

## Key module-level state (transient, never persisted)
- `_lastConflicts` — cached conflict array, set at top of `render()`
- `_conflictsPanelOpen`, `_workloadPanelOpen` — boolean sidebar toggles
- `_dm` — day modal state (`{ dateStr, editId, adding, templateId, startTime, endTime, assignments, memberSearch }`)
- `_am` — appointment modal transient state
- `_tourStep`, `_tourSpotlight`, `_tourCard` — tour state
- `_db` — cached IndexedDB connection
