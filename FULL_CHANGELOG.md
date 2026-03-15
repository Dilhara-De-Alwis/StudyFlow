# StudyFlow — Full Changelog

---

## v4.3.0 — "Solid Ground" Update (March 15, 2026)

> A focused stability and polish release. v4.3 hunts down every crash, race condition, and rendering inconsistency that survived v4.1 — then delivers a complete mobile experience overhaul and a full rewrite of the drag-and-drop system to support cross-quadrant task moves. All changes apply to both `new_index.html` and `index.html`.

---

### 🔴 Critical Bug Fixes

**Bug 1 — `addXP()` corrupting the Dashboard GPA card**
`addXP()` was writing raw XP values directly into the GPA stat card DOM node (`#stat-xp`), overwriting the weighted GPA figure every time XP was awarded (task completion, pet interaction, session end). Removed all direct DOM writes from `addXP()`; the GPA card is now updated exclusively by `refreshDashboard()`. Added a `refreshDashboard()` call at the end of `petInteract()` so pet interactions no longer blank the card.

**Bug 4 — Streak date comparison unreliable across browsers**
Streak logic was comparing dates using `new Date().toDateString()`, which returns locale-dependent strings (e.g. `"Mon Mar 15 2026"`) that vary by browser timezone and locale settings, causing streaks to break or double-count on some devices. Changed to `new Date().toISOString().split("T")[0]` throughout — a fixed `YYYY-MM-DD` format that is consistent across all browsers and timezones.

**Bug 6 — `escHtml()` crashing on `null` or `undefined` input**
Any code path that passed a null/undefined task name, exam subject, or user field to `escHtml()` threw a `TypeError: Cannot read properties of null` and halted rendering. Added a null guard at the top of the function, `String()` coercion to safely handle numbers and booleans, and escaping of single quotes (`&#39;`) in addition to the existing double-quote and angle-bracket escaping.

**Bug 8 — Theme icon flashing moon icon on light-mode load**
The sidebar theme toggle rendered a moon SVG as its default icon regardless of the active theme, causing a visible flash on every page load when the default theme is light. Changed the default icon to the sun SVG to match `data-theme="light"`. The icon now correctly reflects the active theme from the very first paint with no flash.

---

### 🟠 Functional Bug Fixes

**Bug 11 — Pet interaction animation conflicting with CSS transitions**
`petInteract()` was applying a transform directly via `element.style.transform = "scale(1.2)"`, which overrode the CSS transition declarations on the pet container and caused the evolve animation to snap rather than ease. Replaced the inline style manipulation with toggling a `.pet-interact` CSS class that defines the transform alongside a matching transition, allowing both animations to coexist without conflict.

**Bug 13 — Settings toggles not reflecting their saved state on load**
`loadSettings()` was only ever removing the `on` class from toggle elements but never adding it, so all toggles rendered as off regardless of what was saved in `localStorage`. Fixed to explicitly call `classList.add("on")` when the saved value is `true` and `classList.remove("on")` when false, covering both states unconditionally.

**Bug 14 / 18 — Semester average and grade stats using a simple average instead of weighted average**
The GPA & Grade Tracker was calculating the semester average as `Σ(scores) / count`, ignoring assessment weights entirely. Changed to the correct weighted formula: `Σ(score × weight) / Σ(weights)`. The Avg, Best Score, and Module Count stats on the Grade Tracker page now also correctly respect the active subject and semester filters rather than always calculating across all data.

**Bug 15 — Calendar placing tasks on creation date instead of due date**
`renderCalendar()` was keying tasks to their `createdAt` timestamp, so tasks always appeared on the day they were created regardless of when they were due. Tasks with a `dueDate` or `dueDateTime` field now map to the correct calendar day. Tasks without a due date continue to appear on their creation date as a fallback.

**Bug 17 — Focus Mode timer ring showing wrong colour during break phases**
When the Pomodoro entered a break phase, the main timer ring updated its stroke colour to the break accent colour but the Focus Mode overlay ring did not — it retained the study-phase colour. Added `focusRing.style.stroke = mainRing.style.stroke` in the phase-transition handler so both rings always stay in sync.

**Bug 19 — Stagger entrance animation causing a 100ms white flash**
Elements that used the stagger-in animation were initialised without an explicit `opacity: 0`, meaning they were briefly visible at full opacity before the `setTimeout(100)` fired and began the animation. Replaced the timeout approach with `opacity: 0` set immediately on element creation, then `requestAnimationFrame(() => requestAnimationFrame(() => classList.add("visible")))` to trigger the transition on the next available paint frame — eliminating the flash entirely.

**Bug 21 — Data import crashing when JSON file is missing top-level keys**
The import validator required all known top-level keys to be present and rejected any file that was missing even one, making partial backups (e.g. tasks-only exports) unrestorable. Relaxed validation to accept any file containing at least one recognised key (`tasks`, `gamification`, `grades`, or `user`). All missing top-level keys (`sessions`, `exams`, `tasks`, `gamification`, `settings`, `user`) are now silently back-filled with their `DEFAULT_DATA` equivalents before merging.

**Bug 23 — Dashboard "Recent Study Sessions" showing in wrong order**
Sessions were rendered in insertion order, so older sessions could appear above newer ones after an import or out-of-order log. Sessions are now sorted by `session.date` descending before the latest 5 are sliced for display.

**Bug 24 — Timer settings accepting zero and negative values**
The focus, short break, and long break duration inputs had no lower bound, allowing a user to set `0` or a negative number which caused the timer to behave nonsensically (immediate completion or backwards countdown). All three values are now clamped to `Math.max(1, parsedValue)` on input and on save.

**Bug 25 — `AudioContext` created fresh on every sound call; sound effects not working**
A new `AudioContext` was being instantiated inside every call to `playSound()`, and browsers silently discard audio contexts created after the page load gesture in some environments. Replaced with a singleton `_audioCtx` variable that is created once on first use and reused thereafter. Added `_audioCtx.resume()` before every sound call to handle the browser autoplay suspension policy — this was the root cause of sound effects silently failing after the first interaction in Chrome and Safari.

**Bug 26 — Large avatar images overflowing `localStorage` quota**
Users uploading high-resolution profile photos were hitting the `localStorage` 5MB ceiling, causing a `QuotaExceededError` that silently wiped the save attempt. Avatar images are now passed through a canvas resize step (max 200px on the longest edge, preserving aspect ratio) and compressed to JPEG at 0.8 quality before the base64 string is written to `sf_avatar`. Typical reduction: 2–4 MB raw → 15–40 KB stored.

---

### 🔵 Drag & Drop System Rewrite

The task drag-and-drop system has been completely rewritten to support cross-quadrant moves.

| Before | After |
|---|---|
| `dragStart` / `drop` — reordered within the same quadrant list only | `taskDragStart` / `taskDrop` / `dropToQuadrant` — unified handler covering all cases |
| Dragging a card to a different quadrant had no effect | Full cross-quadrant moves work in all directions (Do First ↔ Schedule ↔ Delegate ↔ Eliminate) |
| No quadrant identity on card elements | Every task card carries a `data-quadrant` attribute for drop-target detection |
| Reorder logic ran unconditionally | Reorder only attempts when no filters are active; filtered views are read-only to prevent silent data corruption |

---

### 📱 Mobile Experience Overhaul

#### Sidebar & Navigation

- **Backdrop overlay** — a dark semi-transparent backdrop now appears behind the sidebar when it opens on mobile; tapping the backdrop closes the sidebar. The backdrop element is placed inside `#app` (not `<body>`) to share the correct stacking context with the sidebar and avoid z-index conflicts with modals
- `closeSidebar()` function extracted and called by `navigate()` on every mobile navigation so the sidebar always closes after a nav tap
- Hamburger button enlarged to a minimum tap target of **48×48px**

#### Responsive Breakpoints (4 tiers)

| Breakpoint | Changes |
|---|---|
| ≤ 1023px (tablet) | Sidebar goes off-canvas; backdrop visible on open; todo and exam card actions always visible (not hover-only) |
| ≤ 640px (mobile) | All multi-column grids → 1 column; stat cards → 2 per row; page header stacks vertically; buttons full-width; task input row stacks; modals → 95vw; timer ring → 200px diameter; filter tabs → horizontal scroll; matrix → 1 column; settings rows stack vertically; toasts centred |
| ≤ 380px (small phone) | Stat cards → 1 per row; font sizes reduced one step; padding tightened throughout |

#### Touch-Friendly Changes

- Task card actions (edit, delete, pin, complete) and exam card actions (edit, delete, archive) are always visible on touch devices — removed the hover-only reveal that made them inaccessible on mobile
- Card hover lift (`transform: translateY(-4px)`) disabled on touch devices via `@media (hover: none)` to prevent cards sticking in the lifted state after a tap
- Filter tab strips are horizontally scrollable with `overflow-x: auto; scrollbar-width: none` so all tabs are reachable without wrapping

#### Task Matrix on Mobile

The 2×2 quadrant grid overrides to a **1×4** single-column stack on viewports ≤ 640px using `#matrix-task-grid { grid-template-columns: 1fr !important }` to ensure the inline grid style set by the desktop layout cannot override the breakpoint rule.

---

### ✅ Verification

| Test | Result |
|---|---|
| Hamburger → tap nav items → sidebar closes | ✅ Working |
| Backdrop tap → sidebar closes | ✅ Working |
| Matrix grid 1-column on mobile | ✅ Verified |
| Sound effects across Chrome / Safari / Firefox | ✅ No console errors |
| GPA weighted average calculation | ✅ Correct |
| Cross-quadrant drag and drop (all 12 direction pairs) | ✅ Working |
| Data import with partial / incomplete JSON | ✅ Relaxed validation working |
| Avatar upload within localStorage limits | ✅ No QuotaExceededError |
| Streak date comparison across timezones | ✅ ISO format consistent |
| Theme icon on light-mode load | ✅ No flash |

---

*StudyFlow v4.3.0 — Stable, smooth, and ready for anything.*

---

## v4.1.0 — "Every Detail Matters" Update (March 15, 2026)

> A precision release that fixes nearly every reported issue across all seven sections of the app. v4.1 unifies the subject/module data model across Exams and Grades, merges the Eisenhower Matrix into the Task Board, overhauls the Exam card design with smart countdowns, repairs all broken timer logic, and fills in the missing load states that left the Dashboard feeling empty on first open.

---

### 🔴 Bug Fixes

**Bug 1 — Sidebar: Calendar and GPA Tracker listed under Insights instead of main navigation**
Both pages have been promoted out of the collapsible Insights group and into the primary sidebar navigation list, sitting alongside Dashboard, Tasks, Exams, Timer, Matrix, and Profile. The Insights group is now reserved for Analytics only.

**Bug 2 — Sidebar: Upcoming exam count not shown**
The sidebar task chip already showed a pending task count badge; the exam nav item had no equivalent. Added a live exam count badge next to the Exams nav label, reading the length of upcoming (future-dated) exams from `localStorage` on every render. Updates in real time when exams are added or archived.

**Bug 3 — Sidebar: Day streak not loading on initial page launch**
`renderSidebar()` was being called before `loadData()` completed its synchronous read. Moved `renderSidebar()` to fire after the data bootstrap sequence inside `DOMContentLoaded`, matching the pattern already used by `renderDashboard()`. Streak now displays correctly on first paint.

**Bug 4 — Sidebar: Day streak and dark/light mode toggle misaligned at the bottom**
Both elements were using `margin-top: auto` independently, causing them to stack in the wrong order with uneven spacing. Wrapped them in a single `.sidebar-footer` flex column with `margin-top: auto` on the container, `gap: 12px` between children, and consistent `padding: 16px`. Alignment is now correct at all sidebar widths (expanded and collapsed).

**Bug 5 — Dashboard: Exam Priority order incorrect**
Exams on the Dashboard "Next Exams" panel were rendering in insertion order. Sort now applies two keys: primary `examDateTime` ascending (soonest first), secondary `priority` descending (High → Medium → Low) as a tiebreaker. Same sort logic applied to the Exam Countdown page.

**Bug 6 — Dashboard: Multiple cards not loading on initial page launch**
Seven dashboard regions — Quote of the Moment, Loading Overview, Today's Tasks, Next Exams, Recent Study Sessions, and the four stat cards (Tasks, Study Hours, Upcoming Exams, XP/GPA) — were all dependent on a deferred `setTimeout(renderDashboard, 0)` that was firing after the paint but before `localStorage` was read in some browser environments. All seven regions now read data synchronously inside `renderDashboard()` which is called directly (no timeout) from `DOMContentLoaded`. Each region renders its own skeleton loader while data is being read and replaces it atomically.

**Bug 7 — Dashboard: XP Points stat card not meaningful**
The XP stat card has been replaced with a **Current GPA** card, pulling the weighted GPA value calculated by the Grade Tracker. Displays "—" if no grade entries exist yet.

**Bug 8 — Tasks: "Add Task" inside the card did not ask for Priority Quadrant**
The inline quick-add button inside each task card bypassed the full task modal and saved tasks without a `priority` field. The inline button has been removed entirely; the primary "Add Task" button in the section navbar is the single entry point, ensuring Priority Quadrant is always captured.

**Bug 9 — Tasks: Editing a task did not offer Priority Quadrant change**
The Edit Task modal was built from a subset of the Add Task modal fields and was missing the `priority` 2×2 radio grid. Added the full quadrant selector to the edit flow; the currently saved priority value is pre-selected on modal open.

**Bug 10 — Tasks: Archive Completed removed**
The "Archive Completed" button (formerly "Clear Done") has been removed completely from the Tasks section. Completed tasks remain visible inside their quadrant card under the Done group until manually deleted.

**Bug 11 — Study Timer: Focus, short break, and long break countdowns broken**
Root cause traced to a shared `interval` variable being reassigned without clearing the previous `setInterval` call, causing two concurrent countdowns to run and subtract time at double speed. All timer phase transitions now call `clearInterval(timerInterval)` before assigning a new interval. The SVG circle path `stroke-dashoffset` calculation was also using the wrong total-seconds reference for break phases — fixed to use `breakSeconds` and `longBreakSeconds` respectively.

**Bug 12 — Study Timer: Changing timer settings did not update the running timer**
Timer settings (focus duration, short break, long break) were only read once at startup and cached. Now read fresh from the settings form at every phase start so changes take effect on the next phase boundary without requiring a page reload.

**Bug 13 — Study Timer: Focus Mode inherited the same broken timer**
Focus Mode entered by rendering a clone of the timer display. The clone was not wired to the same `timerInterval` tick, so it showed a frozen time. Focus Mode now reads directly from the shared timer state object on every tick rather than from a separate DOM clone.

**Bug 14 — Profile: Profile picture cannot be removed once set**
A **Remove Photo** button (trash icon) is now shown below the avatar when a photo is set. Clicking it clears `sf_avatar` from `localStorage`, replaces the avatar with the default initials placeholder, and hides the remove button.

**Bug 15 — Settings: Version number still showing v3.0**
Version string updated to `v4.1` in the Settings page footer and in the document `<title>`.

---

### 🟠 Feature Changes & Reworks

#### Sidebar
- **Matrix removed as a standalone sidebar page** — merged into the Tasks section (see Tasks rework below). The Matrix sidebar nav item has been removed; the section is accessible via the redesigned Task Board.
- **Navigation order updated:** Dashboard → Tasks → Exams → Timer → Grades → Calendar → Analytics → Profile → Settings

#### Dashboard
- **First name only** — the greeting ("Good Morning, [Name]") now extracts only the first token of `user.firstName` rather than displaying the full name. Matches the natural way apps like Notion and Linear greet users.
- **Add Exam button** — a secondary "Add Exam" button has been added to the Dashboard header bar next to the existing "Add Task" button so users can log an incoming exam without leaving the Dashboard view.
- **Exam panel sort** — Next Exams panel sorted by `examDateTime` ascending then `priority` descending (consistent with the Exam Countdown page sort).

#### Tasks Section — Combined Task Board & Eisenhower Matrix

The standalone Matrix page has been retired. The Task Board now *is* the matrix:

- **2×2 quadrant grid layout** — the task board renders four quadrant cards in a 2×2 CSS grid (Do First / Schedule / Delegate / Eliminate) mirroring the v3.0 Matrix page design
- **Each quadrant card is collapsible/expandable** — a chevron button in the card header toggles the card body open or closed; collapse state persists per-quadrant in `localStorage`
- **Within each quadrant**, tasks are sorted: Pinned first, then Active tasks (incomplete), then Done tasks at the bottom — no manual reordering needed
- **Filtering system simplified** — tabs reduced to: `All` · `Today` · `Yesterday` · `Last 7 Days`. The `Active` and `Done` tabs have been removed since active/done ordering is handled within each quadrant card
- **Category filter** remains on the right side of the header (v2.0 position)
- **Pinned / starred tasks** — pin icon on each task card; pinned tasks float to the top of their quadrant. Gold top border applied to pinned cards. Dashboard task panel also respects pin order
- **Task completion percentage** — displayed as `X / Y tasks completed (Z%)` next to the task count in the section header. Updates live as tasks are checked off
- **Date & time picker on task creation** — the Add Task and Edit Task modals now include an optional datetime picker. Stored as `task.dueDateTime` (ISO string). Allows entering future-dated tasks today. Overdue logic continues to use this field for red border + Overdue badge

#### Exam Countdown Section — Subject System & Card Redesign

**Subject management (separate from adding an exam):**
- A **Manage Subjects** button opens a modal where students add subjects independently of exams
- Subject form fields: Subject Name (required), Total Credits, Year, Semester (1 or 2) — credits/year/semester are optional
- Subjects saved under `sf_subjects` as `[{ id, name, credits, year, semester }]`
- Subjects shared with the GPA & Grade Tracker (same data source — no double entry)

**Adding an exam now uses:**
- Subject dropdown (populated from `sf_subjects`)
- Exam date + time picker
- Priority (High / Medium / Low)
- Target Grade
- Exam weight % (used in GPA calculation)
- All fields except subject and date are optional

**Smart countdown display:**

| Time remaining | Display format |
|---|---|
| More than 2 weeks | `X weeks, Y days` |
| 1–14 days | `X days, Y hours` |
| Less than 24 hours | `Xh Ym` |
| Less than 1 hour | `Xm Ys` |
| Exam day | `TODAY – Xh Ym remaining` |

**Card redesign:**
- Dynamic accent colour retained but card layout rebuilt: subject name large at top, countdown prominent centre, priority badge top-right, weight % and target grade as small meta tags below the countdown. Card corners more rounded (`24px`); subtle gradient background per accent colour

**Sort order:** soonest exam first, then priority High → Low as tiebreaker (consistent with Dashboard)

#### Study Timer — UI Enhancements
- **Session title text field** moved inside the timer card, above the timer circle
- **Enter Focus Mode button** moved inside the timer card, below the timer circle and above the Start button
- Layout change applied consistently across normal view and Focus Mode overlay

#### GPA & Grade Tracker — Subject System & Filtering

**Subject dropdown** now reads from the shared `sf_subjects` store (same data added in Exam Countdown — no re-entry required).

**Assessment Type dropdown** — replaces the free-text Assessment Type field:
- Default types: Assignment, Quiz, Midterm Exam, Final Exam, Coursework, Presentation, Lab Report
- Users can add custom types via a "Manage Assessment Types" option in the dropdown

**Grade entry form fields:**
- Subject (dropdown from `sf_subjects`)
- Assessment Type (dropdown)
- Marks Obtained
- Total Marks
- Weight %
- Target Grade
- Date

**Grade Entries filtering:**
- Filter tabs: `All` (paginated, 10 per page) · `By Subject` · `By Semester` · `By Assessment Type`

**Performance Trend chart filtering:**
- Filter options: `All Subjects` · individual subject · `By Semester`
- Chart re-renders on filter change with a smooth transition

#### Profile — Name Split
- `user.name` field replaced by `user.firstName` and `user.lastName` stored separately
- Profile form shows two inputs: First Name and Last Name
- Dashboard greeting uses `user.firstName` only
- PDF report header uses `user.firstName + " " + user.lastName`
- Existing single-name saves are migrated: the first word becomes `firstName`, remainder becomes `lastName` (empty string if only one word)

---

### 🟢 New Additions

#### Achievements — 6 New Badges (Profile & XP)

Added to the existing 8 badges, bringing the total to 14:

| Badge | Condition |
|---|---|
| 📚 **Module Master** | Log grade entries for 5 or more different subjects |
| 🗓️ **Calendar King** | Have events on 20 or more distinct calendar days in a month |
| 🎯 **GPA Guardian** | Maintain a GPA of 3.5 or above for a full semester |
| ⏰ **Time Lord** | Accumulate 500 total study hours |
| 🧩 **Matrix Thinker** | Place at least one task in each of the four Eisenhower quadrants |
| 💾 **Data Defender** | Export a JSON backup at least once |

---

### 📱 Mobile Refinements

- 2×2 quadrant grid collapses to 1×4 stacked on viewports ≤ 640px (consistent with Matrix behaviour in v3.0)
- Exam cards on mobile show countdown and subject name prominently; weight/target badges hidden by default, revealed via a "More" tap
- Smart countdown format tested and verified on all breakpoints
- Subject and Assessment Type dropdowns use native `<select>` on mobile for better touch UX; custom styled dropdown on desktop
- Calendar and Grades pages promoted to main nav are now accessible from the mobile bottom nav bar

---

### 💾 Data & Storage

- `sf_subjects` — new shared key: `[{ id, name, credits, year, semester }]`; read by both Exams and Grades sections
- Task schema extended: `dueDateTime?: string` (replaces the date-only `dueDate` from v3.0; old records migrated by appending `T09:00:00`)
- Exam schema extended: `weight: number`, `subjectId: string` (reference to `sf_subjects`)
- Grade entry schema extended: `assessmentTypeId: string`, `subjectId: string`
- `user` object split: `firstName: string`, `lastName: string` (migrated from single `name` field)
- Quadrant collapse state stored per-quadrant: `sf_quadrant_collapse: { do, schedule, delegate, eliminate }` booleans
- Version string in `DEFAULT_DATA.meta.version` updated to `"4.1"`

---

*StudyFlow v4.1.0 — Every detail, finally in its right place.*

---

## v3.0.0 — "Full Stack Student" Update (March 15, 2026)

> The biggest release since launch. v3.0 adds six entirely new pages, a complete GPA & Grade Tracker, JSON import, profile photo uploads, PDF report export, an Eisenhower Priority Matrix, a full calendar view, pinned tasks, task due dates, Focus / Do Not Disturb mode, an adaptive XP system, and a thorough sweep of every bug and layout issue reported since v2.0. Mobile has been rebuilt again from scratch.

---

### 🔴 Bug Fixes

**Bug 1 — Hamburger menu icon overlaying the sidebar and blocking content on mobile**
On viewports ≤ 640px the hamburger button was positioned inside the content flow with `position: absolute` but no containing block, causing it to float over sidebar nav links and block taps. Moved the toggle into a dedicated fixed-position top bar that sits in its own stacking layer (`z-index: 50`). Page header titles are now shifted right by the toggle width (`padding-left: 56px`) so they never overlap on any screen size.

**Bug 2 — Level mismatch between Profile card and Virtual Companion section**
`gamification.level` was being read from two different code paths — the profile header used a cached copy set at login while the pet section pulled the live value from `localStorage`. Both now read exclusively from `loadData().gamification.level` through a single `getLevel()` utility, guaranteeing they always agree.

**Bug 3 — Pet SVG not updating when level changed**
Pet evolution check was only firing inside `addXP()` but not when the profile section was rendered fresh. Added `syncPetStage()` call inside `renderProfile()` and `renderVirtualCompanion()` so the correct SVG stage loads on every view. Pet SVG now also advances every 2 levels (not every 10) as intended, using `Math.floor(level / 2)` clamped to the 10 available SVG stages.

**Bug 4 — "Change Pet" modal covered by robo companion SVG**
The companion SVG had `position: absolute` with no bounded parent, letting it bleed over the modal overlay. Pet selection grid inside the modal rebuilt as a responsive `2×2` CSS grid (`grid-template-columns: 1fr 1fr`). SVG companion is now `position: relative` inside a constrained card so it cannot escape its bounds.

**Bug 5 — Edit Profile only reachable via hidden modal button**
The "Edit Profile" flow was buried behind a single button with no persistent signal. Profile fields (name, designation, university, email, avatar) are now displayed as an always-visible editable card directly inside the Profile & XP section. Changes auto-save on blur; a "Save Changes" button provides explicit confirmation. The separate modal has been removed.

**Bug 6 — Quote box resizing with quote length, causing layout shift**
The motivational quote container had no fixed height, so short quotes produced a compact card while long quotes pushed surrounding elements down. Container now uses `min-height: 80px` with `display: flex; align-items: center` so the card stays a consistent size regardless of quote length.

**Bug 7 — Mobile layout broken across multiple sections**
Full mobile audit and rework performed (see Mobile section below for complete list of changes).

---

### 🟠 Feature Changes & Reworks

**Quote refresh button repositioned to top of quote card**
The refresh icon button has been moved from beside the quote text to the top-right corner of the quote card, matching standard card action conventions. The fixed-height quote box (Bug 6) ensures the button position never shifts.

**Task filter tags replaced — new logic and labels**
Previous tags (`All`, `Active`, `Done`) replaced with a richer six-tab system:

| Tab | Logic |
|---|---|
| All | Every task regardless of state or date |
| Today | Tasks with `dueDate` equal to today's ISO date, or tasks created today with no due date |
| Active | Incomplete tasks only |
| Done | Completed tasks only |
| Yesterday | Tasks due or created yesterday |
| Last 7 Days | Tasks with `dueDate` or `createdAt` within the past 7 days |

**Category filter repositioned**
The category filter dropdown has been moved from the left side of the task header to the right, freeing the left side for the tab strip and reducing visual crowding on mobile.

**"Clear Done" button renamed and repositioned**
Button label changed from `Clear Done` → `Archive Completed`. Moved from the task header toolbar to the bottom of the task list, below all task cards, so it is not accidentally triggered while browsing tasks.

**Study timer session title — user-editable**
The session label previously hardcoded as `"Study Session"` is now a text input field at the top of the timer card. Placeholder text reads `"Study Session"` and is used verbatim if the user leaves the field blank. The entered title appears in the Focus Mode overlay, session history, and the Analytics session log.

**Browser tab favicon added**
A StudyFlow SVG icon (book + pencil mark) is now linked as `<link rel="icon" type="image/svg+xml" href="...">` in `<head>`. The tab no longer shows a blank page icon in Chrome, Firefox, or Safari.

---

### 🟢 New Features

#### GPA & Grade Tracker (New Page)

A dedicated **Grades** page accessible from the sidebar. Covers the full grade lifecycle from assignment submission to final GPA calculation.

- **Module grade log** — add assignments and exams per module with: assessment name, type (Assignment / Midterm / Final / Quiz), marks achieved, total marks, and weighting percentage
- **Actual grade on exams** — the Exam edit modal now includes an `Actual Grade` input so results can be recorded after the exam is sat, alongside the existing `Target Grade`
- **Per-subject average** — each module card shows the running weighted average, colour-coded against the student's target grade (green = at or above target, amber = within 10 points, red = below)
- **Weighted GPA** — overall GPA calculated as `Σ(module average × credit weight) / Σ(credit weights)` displayed in a prominent stat card at the top of the page
- **Semester average** — unweighted mean across all modules for the current semester
- **Grade trend chart** — line chart showing the student's running GPA over the last 8 assessment submissions
- **Target vs actual comparison** — each module card shows a progress bar comparing the student's actual average against their target grade, with a delta label (`+5.2` / `-3.1`)

#### JSON Data Import (Settings)

Export to JSON existed since v1.0 but had no matching import. Now complete:

- **Import Data button** added to the Settings page, opens a native file picker filtered to `.json` files
- `FileReader` parses the selected file and validates it against the `DEFAULT_DATA` schema — any of `tasks`, `gamification`, `grades`, `user`, `sessions`, `exams` keys triggers acceptance; unrecognised files are rejected with a clear error toast
- A confirmation dialog offers two choices: **Merge** (adds imported records without overwriting existing ones) or **Replace** (wipes current data and loads the import cleanly)
- Missing top-level keys in the import are back-filled with their defaults so a partial export (e.g. tasks only) does not corrupt the other sections

#### Profile Photo Upload

- The emoji avatar placeholder on the Profile page is replaced by a circular photo upload zone
- Clicking the avatar opens a native file picker accepting `image/*`; captured images from camera apps on mobile are also accepted via the `capture` attribute
- Selected image is resized to max 200×200px and compressed to JPEG 0.85 before being stored as a base64 data URL in `localStorage` under `sf_avatar`
- Avatar appears in the profile card header, the sidebar user chip (collapsed and expanded states), and the PDF report (see below)

#### Export Study Report as PDF (Analytics)

A **Generate Report** button in the Analytics section produces a downloadable PDF using `jsPDF` from `cdnjs.cloudflare.com`.

Report layout (single page, A4):
- Student name, designation, university, and avatar (top-left)
- Date range of report (last 30 days by default)
- Key stats row: total study hours, total sessions, tasks completed, current streak, XP level and Scholar Rank
- Weekly study hours bar chart rendered via `canvas.toDataURL()` and embedded as an image
- Grade summary table: module name, target grade, actual average, delta
- Footer: "Generated by StudyFlow — Your Academic Partner"

#### Adaptive XP System

Static XP-per-action values replaced with a dynamic scaling model:

- **Task XP** scales with estimated difficulty: tasks with sub-tasks award `10 + (5 × sub-task count)` XP; overdue completions award a 1.5× multiplier ("better late than never")
- **Session XP** scales with duration: `base 20 XP + (2 × minutes studied)`, capped at 120 XP per session
- **Focus rating bonus**: sessions rated 8–10 award an extra 15 XP; sessions rated 1–3 award no bonus
- **Streak multiplier**: active streak applies a `1 + (streak days × 0.05)` multiplier to all XP earned, capped at 2.0× (20-day streak)
- **Grade achievement bonus**: logging an actual grade that meets or beats the target awards 25 XP flat
- XP thresholds for levelling also scale: `level_threshold = 100 × level^1.15` so early levels feel achievable and later levels require sustained effort

#### Task Due Dates & Overdue Alerts

- **Date picker** added to the Add Task and Edit Task modals — field is optional; tasks without a due date behave as before
- `dueDate` stored as an ISO string (`YYYY-MM-DD`) per task
- In `renderTasks()`, each task is compared against today's date: tasks past their due date receive a red left border and an `Overdue` badge (pill, red background)
- **Dashboard overdue count** — a new stat card on the Dashboard shows the count of overdue tasks; clicking it navigates directly to the Tasks page with the `Active` filter applied
- Today and Yesterday filter tabs (see filter rework above) use `dueDate` as the primary date field

#### Pinned / Starred Tasks

- A `pinned: false` boolean added to the task schema; existing tasks default to `false` on load
- Each task card shows a pin icon button (top-right corner); clicking it toggles `task.pinned`, saves, and re-renders
- Pinned tasks sort before all other tasks in `renderTasks()` regardless of the active filter
- Dashboard task panel shows pinned tasks at the top of the list regardless of recency
- Pinned cards display a thin gold top border (`border-top: 3px solid #F59E0B`) to distinguish them at a glance

#### Eisenhower Priority Matrix (New Sidebar Page)

A dedicated **Matrix** page added to the sidebar navigation.

- **Task priority field** — the Add Task modal now includes a 2×2 radio-button quadrant selector before saving. The four options are:

  | Quadrant | Label | Meaning |
  |---|---|---|
  | Top-left | Do First | Urgent + Important |
  | Top-right | Schedule | Important, not Urgent |
  | Bottom-left | Delegate | Urgent, not Important |
  | Bottom-right | Eliminate | Neither |

- **Matrix page** renders a full 2×2 grid, each quadrant styled with a distinct accent colour. Task cards sit inside their assigned quadrant and can be **dragged between quadrants** — the existing `taskDragStart` / `taskDrop` system is reused with `data-quadrant` target detection
- Priority is stored as `task.priority: "do" | "schedule" | "delegate" | "eliminate"` and persists across sessions
- Tasks without a priority (created before v3.0) default to `"schedule"` on load

#### Focus / Do Not Disturb Mode (Study Timers)

- **Focus Mode button** added to the Study Timer page
- Clicking it expands the timer to fill the entire viewport: sidebar hides, all other UI dims to `opacity: 0.05`, and only the timer ring, countdown, current session title, and active task name are visible
- A subtle pulsing border animation runs on the timer ring during Focus Mode to reinforce the immersive state
- **Exit conditions:** pressing `Esc`, clicking the exit button (top-right corner, always visible), or the current timer phase completing naturally — all return the app to normal layout with a smooth fade-in
- Focus Mode state persists if the user accidentally navigates to another browser tab; re-entering the StudyFlow tab restores Focus Mode automatically

#### Full Calendar View (New Sidebar Page)

A dedicated **Calendar** page added to the sidebar navigation.

- **Monthly grid** — 7-column × 5–6 row layout built in pure JS; loops through days of the current month and creates one cell per day
- **Event chips** displayed inside each day cell, colour-coded by type:
  - 🔴 Red pill — exam on this date
  - 🔵 Blue pill — task with `dueDate` on this date
  - 🟢 Green pill — study session completed on this date
- **Prev / Next month navigation** — chevron buttons update `calendarMonth` and `calendarYear` state; the grid re-renders without a page reload
- **Day detail panel** — clicking any day cell expands a detail panel below the grid listing all events for that day with full information (exam subject + time, task name + priority + pinned state, session duration + focus rating)
- Pinned tasks shown with a gold star prefix in the detail panel

---

### 📱 Mobile Rework (Full Audit)

All sections tested at 320px, 375px, 390px, 414px, and 640px viewport widths.

- Hamburger button moved to a fixed top bar; no longer overlaps any content (Bug 1 resolution, applied globally)
- Task filter tabs scroll horizontally with `overflow-x: auto; scrollbar-width: none` — no more wrapping or clipping
- Matrix quadrant grid collapses to a single-column 1×4 stacked layout on viewports ≤ 640px
- Calendar grid scales month names and day cells; event chips truncated to icon-only on ≤ 375px
- Grade tracker module cards stack vertically below 640px; the GPA stat card spans full width at the top
- Focus Mode exit button repositioned to top-centre on mobile (top-right was outside safe area on notched devices)
- PDF report generation disabled gracefully on iOS Safari < 16 with a toast message directing users to a desktop browser
- All modals capped at `95vw` with `max-height: 90dvh` and internal scroll to prevent overflow on small screens

---

### 🎨 UI Refinements

- New sidebar navigation items for Matrix, Calendar, and Grades pages — each with a distinct SVG icon
- Focus Mode overlay uses `backdrop-filter: blur(20px)` on the dimmed background for a glass-frosted effect
- Pinned task gold border animates in with a 300ms ease transition when pinning/unpinning
- Overdue badge pulses gently (`opacity: 1 → 0.6 → 1`, 2s loop) to draw attention without being aggressive
- Grade target vs actual progress bars use a smooth `width` transition when values update
- Calendar event chips fade in staggered per-cell using `animation-delay` proportional to cell index

---

### 💾 Data & Storage

- Task schema extended: `dueDate?: string`, `pinned: boolean`, `priority: "do" | "schedule" | "delegate" | "eliminate"`
- Grades stored under `sf_grades` as `{ modules: [{ id, name, credits, targetGrade, assessments: [...] }] }`
- Avatar stored under `sf_avatar` as a base64 JPEG data URL string
- Session title stored as `session.title: string` alongside existing session fields
- Import merges by record `id`; duplicate IDs from the import file are skipped in merge mode and overwritten in replace mode
- All new fields back-filled with safe defaults when loading records created before v3.0

---

*StudyFlow v3.0.0 — Everything a student needs, nothing they don't.*

---

## v2.0.0 — "Academic Partner" Update (March 15, 2026)

> A major quality-of-life release focused on personalization, polish, and fixing every rough edge reported since v1.0. The app's identity has been refined, the analytics section rebuilt from the ground up, and a wave of UX improvements land across every module.

---

### 🔴 Bug Fixes

**Bug 1 — Motivational quote not loading on initial page load / refresh**
The quote engine was initialising after the DOM paint cycle, causing a blank quote card on first load and after hard refresh. Moved quote initialisation to fire synchronously inside `DOMContentLoaded` before any `setTimeout` chains. Quote now appears immediately on every load.

**Bug 2 — Good morning greeting hardcoded**
The dashboard header greeted every user with "Good Morning" regardless of time of day. Replaced static string with a time-aware function:
- 12:00 AM – 11:59 AM → **"Good Morning"**
- 12:00 PM – 3:59 PM → **"Good Afternoon"**
- 4:00 PM – 11:59 PM → **"Good Evening"**

Greeting now updates live without requiring a page reload.

**Bug 3 — Marks accepting values above 100 and negative values**
Grade input fields had no validation bounds. Added `min="0"` and `max="100"` HTML attributes plus a JavaScript clamp on `input` event: `value = Math.min(100, Math.max(0, parseInt(value) || 0))`. Marks can no longer be saved outside the 0–100 range.

**Bug 4 — Mobile header title obscured by sidebar toggle icon**
On viewports ≤ 640px the hamburger/sidebar icon was rendering above the page header title due to a missing `z-index` layering context on the header. Fixed by promoting the header to its own stacking context (`position: relative; z-index: 10`) and capping the sidebar toggle at `z-index: 9` so it no longer bleeds over the title text.

**Bug 5 — Study session timer required manual restart after each session**
After a study session ended, the break timer required the user to press Start manually. Timer now transitions automatically: study → short break → study in a continuous loop. After the configured number of sessions, the long break fires automatically. Users only need to press Start once per day.

**Bug 6 — Sound effects too quiet to notice**
`AudioContext` oscillator gain was set to `0.1`, inaudible on laptop speakers at moderate volume. Gain raised to `0.6` with a short linear ramp-down envelope to avoid clipping. Notification chime is now clearly audible without being jarring.

---

### 🟠 Feature Changes & Removals

**Motivational quote moved exclusively to Dashboard**
The duplicate quote display that previously appeared on the Profile page has been removed. The Dashboard is now the single home for the daily quote, paired with the new manual refresh button (see below). Profile page no longer renders a quote block.

**Profile section removed from Settings**
All profile fields (Name, Degree/Designation, University, Email) have been lifted out of the Settings panel and consolidated into the **Profile & XP** section where they belong. Settings now contains only app preferences (theme, notifications, timer defaults, sound toggles).

**Slogan updated**
Changed from `"Your Academic OS"` → `"Your Academic Partner"` across the sidebar, splash screen, and document `<title>`.

**Default theme changed to Light Mode**
`data-theme` now defaults to `"light"` on `<html>` at document load. Users who have previously saved a theme preference will still have that preference respected via `localStorage`.

---

### 🟢 New Features

#### To-Do List

- **Custom categories** — users can now create, rename, and delete their own task categories. The hardcoded default categories have been removed. A "Manage Categories" modal lets users add a category name and pick an accent colour.
- **Edit task** — each task card now has an edit button that reopens the task creation form pre-populated with the task's current name, category, emoji, and sub-tasks, allowing full in-place editing.
- **Edit category on existing task** — the edit form includes a category dropdown so tasks can be reassigned to a different category after creation.

#### Exam Countdown

- **Edit exam** — each exam card now exposes an edit (pencil) icon that opens a pre-filled modal allowing the user to update subject name, exam date, exam time, difficulty, and grade goal.
- **Exam time field** — a time input (`HH:MM`) has been added to the exam creation and edit forms. The countdown card now displays the exact time remaining down to hours and minutes on the day of the exam.

#### Dashboard

- **Motivational quote refresh button** — a circular refresh icon button sits beside the daily quote. Clicking it picks a new random quote from the local pool (100+ quotes) without waiting for the next day. The quote swaps in with a short fade transition.
- **Live clock with seconds** — the date display in the dashboard header now includes a live digital clock showing the current time with seconds (`HH:MM:SS`), updating every second via `setInterval`.

#### Sidebar

- **Collapse / expand toggle** — the sidebar now has a chevron toggle button at its base. When collapsed, only navigation icons are visible (no labels); when expanded, full icon + label layout is shown. State is persisted in `localStorage` so the sidebar remembers its last position across sessions.

#### Analytics (Full Rework)

The Analytics section has been completely rebuilt. Previous version showed only a static productivity hint string. New version includes:

- **Weekly study hours chart** — 7-day bar chart of total hours studied per day
- **Session quality trend** — line chart of 1–10 focus ratings over the last 14 sessions, with a rolling 7-session average trendline
- **Peak focus heatmap** — 7-day × 24-hour grid; each cell is shaded by average focus score during that hour/day combination, surfacing patterns like "You're most focused Tuesday 9–11 AM"
- **Productivity summary cards:**
  - Best focus hour range
  - Worst focus hour range
  - Average session duration
  - Total sessions this week
- **Consistency score** — "You studied X/7 days this week" stat with a 7-dot indicator row (filled = studied, empty = missed)
- **Subject time split** — doughnut chart showing proportion of study time attributed to each subject/category this week

#### User Profile & Gamification

- **Edit designation** — the "Student" label under the user's name was previously hardcoded. Designation is now an editable free-text field saved to `localStorage` (e.g. "Year 2 Student", "Postgraduate", "Self-learner").
- **Email field added** — an Email input has been added to the profile form alongside Name, Degree, and University.
- **4 new achievement badges** (added to the existing set of 4):
  - 📅 **Planner Pro** — create 20 or more tasks in a single week
  - ⚡ **Speed Runner** — complete 5 tasks in under one hour
  - 🌅 **Early Bird** — log a study session before 7 AM
  - 🏆 **Century Club** — accumulate 100 total study hours across all time
- **Virtual pet evolution with SVG animations** — each of the 4 selectable pets now has 10 distinct SVG stages (one per level bracket). When the pet levels up, the old SVG fades out and the new stage SVG fades/scales in via a CSS transition. Level brackets: 1, 5, 10, 15, 20, 25, 35, 50, 75, 100.

#### Icons (System-wide)

Replaced all emoji navigation and section markers with crisp SVG icons sourced from a consistent icon set. Applied to:
- Sidebar navigation items
- Section page headers
- Stat cards on the Dashboard
- Achievement badge indicators on the Profile page
- Timer controls
- Exam and task card action buttons (edit, delete, complete)

Emojis are retained only where they are user-defined content (task emojis, pet names).

---

### 🎨 UI Refinements

- Sidebar collapse state adds a smooth `width` transition (`250px → 64px`) with `overflow: hidden` to prevent label text flashing during animation
- Quote refresh button spins 360° on click via a CSS keyframe before the new quote fades in
- Pet evolution transition uses `opacity` + `scale(0.8 → 1.0)` over 400ms ease-out for a satisfying growth feel
- Timer auto-transition plays the updated louder chime, briefly flashes the ring colour, then starts the next phase after a 1.5-second pause so users can register the change
- Mobile header z-index layering corrected across all five section views

---

### 💾 Data & Storage

- Category data stored under `localStorage` key `sf_categories` as an array of `{ id, name, color }` objects
- Exam records updated to include `examTime: "HH:MM"` field; old records without this field default to `"09:00"` on load
- Profile object extended with `designation` and `email` fields; missing fields on existing saves are back-filled with empty strings
- Pet stage index derived from level at runtime; no additional storage needed beyond existing `gamification.level`

---

*StudyFlow v2.0.0 — Keep studying. Your Academic Partner is with you.*

---

## v1.0.0 — Initial Release (March 15, 2026)

> The first public release of **StudyFlow** — a fully local, gamified study planner built with HTML, CSS, and JavaScript. No backend. No account required. Everything lives in your browser.

---

### 🚀 What's New — Feature Overview

This release introduces all five core modules of StudyFlow from scratch.

---

### ✅ To-Do List

- **Checkbox & task name** with inline emoji support per task
- **Idea-board layout** — tasks displayed as visual cards, not a plain list
- **Sub-task branching** — break big tasks into checkable sub-items; press `Tab` inside a task input to nest a sub-task
- **Daily notification system** — end-of-day alert for incomplete tasks ("You have 7 tasks not done today") and a start-of-next-day reminder ("You still have X tasks from yesterday")
- **Completion percentage bar** — live progress indicator per list showing % of tasks done
- **Completion micro-animations** — confetti burst + subtle sound effect fires when a task is checked off
- **Streak freeze** — users are granted one "rest day" per week that does not break their active task streak

---

### ⏳ Exam Countdown

- **Subject name & exam date** inputs per card
- **Random card background colors** — each exam card is assigned a unique accent color on creation
- **Live countdown** — shows days, hours, and minutes remaining from current date
- **Priority scoring** — exams are auto-ranked by urgency (days remaining) combined with a user-set difficulty level (1–5)
- **Grade goals** — users can set a target grade per exam; required score to reach goal is calculated and displayed
- **Victory Hall** — completed/past exams are archived in a dedicated "Victory Hall" section with a gamified trophy display instead of being deleted

---

### ⏱️ Study Timers

- **Pomodoro timer** — fully adjustable study timer, short break timer, and long break timer (all values set in minutes by the user)
- **Auto-transition** — when the study timer ends, the break timer starts automatically; no manual intervention required
- **Long break control** — users set after how many sessions (e.g. 2, 4, or even 10) the long break triggers
- **Focus Time Record grid** — a 24-hour timeline (12 AM → 12 AM) shows time blocks when the timer was active, displayed with dotted segment markers per 15-minute slot; the grid resets at midnight for a fresh daily record
- **Session ratings** — after each session, rate your focus 1–10; rating is stored and correlated with time of day to surface productivity patterns
- **Weekly focus report** — end-of-week summary showing best/worst focus times and average session quality
- **Consistency badge** — "You studied 5/7 days this week" style badge displayed on the timer page

---

### 📊 Dashboard

- **Stat cards:**
  - Total study hours logged
  - Total upcoming exams
  - Total to-do items remaining for today
- **7-day to-do completion chart** — bar chart showing daily completed task counts for the last 7 days
- **Motivational quote system** — 100+ short quotes stored locally; a new quote appears each day automatically
- **Study analytics panel** — surfaces productivity patterns derived from session rating data (e.g. "You're most focused 9–11 AM")

---

### 👤 User Profile & Gamification

- **Local profile** — user saves Name, Degree, University, and Email; all data stored in `localStorage`, never sent anywhere
- **XP & Scholar Rank leveling** — completing tasks, finishing study sessions, and hitting streaks earns XP; accumulating XP levels up the user's Scholar Rank
- **Achievement badges:**
  - 🔥 **7-Day Streak** — study every day for 7 days
  - 🦉 **Night Owl** — log a study session after 10 PM
  - 🎯 **Deadline Crusher** — complete an exam task before its due date
  - 🧠 **Deep Diver** — accumulate 4+ hours of focus in a single day
- **Virtual study buddy (pet system):**
  - Choose from 4 pets on first setup
  - Pet visually evolves as XP milestones are reached
  - Interacting with your pet plays a small animation and grants a small XP bonus

---

### 🎨 UI & Design System

- **Design language:** Glassmorphism-inspired cards with `backdrop-filter: blur`, rounded corners (`border-radius: 16–24px`), and soft shadows
- **Color palette:** Blue-primary system with semantic accent colors; full support for both light and dark modes via CSS custom properties
- **Page load animations:** Elements stagger-animate in using `opacity` + `translateY` transitions on `requestAnimationFrame`
- **Responsive breakpoints (4 tiers):**

  | Breakpoint | Layout |
  |---|---|
  | ≥ 1280px (TV / wide desktop) | Full sidebar + multi-column grid |
  | 1024–1279px (laptop / desktop) | Sidebar collapsed to icon rail |
  | 641–1023px (tablet) | Off-canvas sidebar with backdrop |
  | ≤ 640px (mobile) | Single-column, full-width cards |
  | ≤ 380px (small phone) | Tighter padding, smaller font scale |

- **Touch-friendly interactions:** All tap targets ≥ 48×48px; hover-only actions exposed permanently on mobile
- **Sound effects:** Singleton `AudioContext` pattern; browser autoplay policy handled via `ctx.resume()` on first user interaction

---

### 💾 Data & Storage

- All user data (tasks, exams, sessions, profile, gamification state) stored entirely in `localStorage`
- **Export / Import JSON** — full data backup and restore supported
- `escHtml()` utility sanitizes all user-supplied strings before DOM insertion (null-safe, handles single quotes)
- ISO date format (`YYYY-MM-DD`) used throughout for cross-browser date comparison reliability

---

### ⚙️ Technical Foundations

- Pure **HTML + CSS + JavaScript** — zero dependencies, zero build step
- CSS custom properties power the entire theme system; switching light/dark requires only a `data-theme` attribute toggle on `<html>`
- `localStorage` keys namespaced to avoid collisions between modules
- All timer values clamped to `Math.max(1, value)` to prevent zero/negative inputs
- Avatar images resized to max 200px and compressed to JPEG 0.8 before storing to stay within `localStorage` size limits

---

*StudyFlow v1.0.0 — Built for students, by a student.*
