# Do Not

- **No external URLs, CDN links, or `import` statements** — single-file offline constraint is non-negotiable.
- **No `localStorage` as primary store** — IndexedDB is primary; localStorage is fallback only.
- **No `detectConflicts()` inside `renderConflictsPanel()` or `renderWorkloadPanel()`** — use `_lastConflicts`.
- **No `oninput` on color pickers** — use `onchange`; `oninput` fires every animation frame.
- **No shallow-spread of shift objects when copying** — always deep-clone `assignments: [...s.assignments]`.
- **No committing tokens or credentials** — use SSH for all git operations.
- **No awaiting `saveState()` inline** — it is fire-and-forget async by design.
