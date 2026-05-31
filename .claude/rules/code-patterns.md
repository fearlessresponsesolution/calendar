# Code Patterns

## Generating IDs
```js
generateId()  // Date.now().toString(36) + Math.random().toString(36).slice(2)
```

## Safe HTML rendering
```js
escapeHtml(s)  // always apply to user-entered strings rendered as innerHTML
// DOM methods (textContent, createElement) are inherently safe — no escaping needed
```

## Date helpers
```js
isoDate(year, month, day)      // month is 0-indexed → "YYYY-MM-DD"
getDaysInMonth(year, month)
formatTime("HH:MM")            // → "9:00am" / "2:30pm"
timeToMinutes("HH:MM")         // returns NaN if input is empty/invalid
rangesOverlap(s1, e1, s2, e2)  // handles midnight-spanning correctly
```

## Next-day calculation (repeated 6+ times — extract if touching this code)
```js
const nd = new Date(dateStr + 'T12:00:00');
nd.setDate(nd.getDate() + 1);
const nextDateStr = isoDate(nd.getFullYear(), nd.getMonth(), nd.getDate());
```

## Midnight-spanning check (repeated — extract if touching this code)
```js
timeToMinutes(shift.endTime) <= timeToMinutes(shift.startTime)
```

## HTML rendering strategy
- `renderWorkloadPanel` and `renderConflictsPanel` use `createElement`/`appendChild` (XSS-safe by default)
- `_renderDayModal`, `_renderApptModal`, `openSettings` use template-literal `innerHTML` + `escapeHtml()`
- Do not mix strategies in new code — pick DOM methods for new panels, template literals for new modals
