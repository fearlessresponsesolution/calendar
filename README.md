# Shift Management Calendar

A fully offline, self-contained shift scheduling calendar for managing teams of up to 30+ members. No installation, no internet connection, no dependencies — just open `index.html` in any modern browser.

---

## Getting Started

1. Download or clone this repository
2. Open `index.html` in your browser (Chrome, Firefox, Edge, or Safari)
3. All data is saved automatically to your browser's local storage — your schedule persists between sessions

---

## Setup: Roles, Members, and Shift Templates

Before building your schedule, configure your team in **⚙ Settings**.

### 1. Create Roles

Go to **⚙ Settings → Roles** and add your work roles (e.g. *Nurse*, *Technician*, *Supervisor*). Roles are used to group members and drive swap recommendations — the scheduler will only suggest swaps between members who share the same role.

### 2. Add Team Members

Go to **⚙ Settings → Members**. For each member enter their name and assign a role. Each member is automatically assigned a distinct color for easy identification across the calendar.

- **Change a member's color** by clicking the color swatch next to their name

### 3. Define Shift Templates

Go to **⚙ Settings → Shift Templates**. Create named shift patterns with a start and end time (e.g. *Morning 06:00–14:00*, *Evening 14:00–22:00*, *Night 22:00–06:00*). Templates can be reused across any day of the month.

> Night shifts that span midnight (e.g. 22:00–06:00) are handled automatically — conflict detection accounts for the overnight overlap.

---

## Building the Schedule

### Adding Shifts to a Day

Click any day cell in the calendar to open the **Day Editor**. From here you can:

- **Add a shift** — pick a template (which pre-fills the times) or enter custom start/end times
- **Assign members** — checkboxes are grouped by role; members already scheduled on an overlapping shift that day are greyed out as *busy*
- **Edit or remove** any existing shift for that day

### Copying Shifts

**Right-click** any day cell for quick copy options:
- *Copy shifts to next day* — duplicates all shifts forward by one day
- *Copy shifts to next week* — duplicates all shifts to the same weekday next week

### Navigating Months

Use the **◄ ►** arrows in the header (or your keyboard's left/right arrow keys) to move between months. Click **Today** to jump back to the current month. Each month's data is stored independently.

---

## Appointment Overlay (Conflict Detection)

The appointment overlay lets you record a member's unavailability — medical appointments, PTO, training, personal days — and automatically flags conflicts with their scheduled shifts.

### Adding Appointments

1. Click **👁 Appointments** in the header to switch to Appointment View
2. Click any day to open the **Appointment Editor**
3. Select the team member, enter a note (e.g. *Doctor appointment*, *PTO*), and set a time range or check *All day*

### Identifying Conflicts

Days with a conflict between a scheduled shift and an appointment show a **red ⚠ badge** on the date number. In Appointment View, conflicting entries are highlighted in red with a ⚠ marker. Toggle back to Shift View at any time — your shifts are unchanged.

### Resolving Conflicts with the Conflicts Panel

Click **⚠ Conflicts** in the header to open the Conflicts Dashboard. This panel:

- Lists every conflict in the current month with the date, shift time, member name, and appointment note
- Provides a **swap dropdown** for each conflict showing available same-role members who are free that shift (not already scheduled and no conflicting appointment)
- Selecting a swap member from the dropdown **immediately reassigns** the shift
- The conflict count badge updates live as conflicts are resolved

---

## Viewing a Member's Schedule

Click any **member name chip** inside a shift bar to open their personal schedule view — a read-only list of every shift they are assigned to during the current month, including dates, times, and shift template names.

---

## Coverage Summary

The footer at the bottom of the calendar shows coverage statistics for each shift template across the current month — for example *Morning: 18/30 days*. Days are counted as covered when at least one member is assigned to that shift.

---

## Printing the Calendar

Click **🖨** in the header (or press Ctrl+P / Cmd+P) to print the calendar. The print layout:

- Switches to landscape orientation automatically
- Hides all UI controls, panels, and modals
- Preserves shift bar colors
- Displays a clean header with the month and year

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `←` / `→` | Navigate to previous / next month |
| `Escape` | Close any open modal or panel |

---

## Data & Privacy

All data is stored exclusively in your browser's **localStorage** — nothing is sent to any server. The app works entirely offline. To back up your schedule, export the localStorage entry `shift_cal_v1` from your browser's developer tools.

---

## Browser Support

Any modern browser running locally via `file://` or a simple web server:

- Chrome / Chromium 90+
- Firefox 88+
- Edge 90+
- Safari 14+
