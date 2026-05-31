# Files & Git

## File map

| File | Purpose |
|------|---------|
| `index.html` | The entire application (~1370 lines) |
| `tests/index.html` | Standalone unit test harness (31 tests) — sync manually with index.html |
| `README.md` | User-facing guide |
| `docs/superpowers/specs/2026-05-30-shift-calendar-design.md` | Original design spec |
| `docs/superpowers/plans/2026-05-30-shift-calendar.md` | 10-task implementation plan |

## Git / GitHub
- Branch: `main`
- Remote: `git@github.com:fearlessresponsesolution/calendar.git` (SSH)
- SSH key: `~/.ssh/id_ed25519`
- GitHub Pages: `https://fearlessresponsesolution.github.io/calendar/`
- All commits include `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

## Test harness
`tests/index.html` duplicates functions from `index.html` (no ES module support on `file://`).
When changing a utility function (`rangesOverlap`, `getAvailableSwaps`, etc.), update both files and keep tests green.
