# StudyFlow — Full Changelog

---

## v5.3.0 — "Living System" Update (March 18, 2026)

> A deep-systems update focused on making every interactive layer feel genuinely alive. v5.3 rebuilds the notification system from scratch (stacked banners, colour-coded accents, 7-day exam warnings, daily study reminders), overhaults the floating pet companion (gradient-border ring, idle float animation, pulsing glow, level pip, ring hue progression), tightens the recurring task gating logic, slashes the XP requirement to make the pet evolve faster, and irons out eight concrete bugs reported against v5.2 — including a critical HTML nesting error that made the entire Mark Tracker section non-functional.

---

### 🟢 New & Changed Features

#### 01 — Charts: Destroy-and-Recreate on Navigation (no flicker)

All Chart.js instances (weekly bar, category doughnut, grade line) are now **destroyed and freshly recreated** on every navigation event rather than being updated in-place. A short 300 ms creation animation prevents any visual pop. Instances are also destroyed when navigating *away* from the Analytics or Grade Tracker pages, so no stale canvas state can carry over to subsequent visits.

**Why the change:** The previous `chart.update()` approach (introduced in v5.2 Bug 17) caused subtle rendering artefacts when switching themes or filtering data — axes would retain old tick values and the canvas context occasionally became detached after a tab switch. Destroy-recreate is more robust and the 300 ms animation eliminates any perceived flicker.

---

#### 02 — Dashboard: Recent Study Sessions Hidden Until Today

The **Recent Study Sessions** card on the Dashboard is now completely hidden when no study sessions have been logged today. It reappears the moment a session completes.

**Before:** The card was always visible and showed sessions from previous days, making it feel stale and cluttered on mornings when you haven't yet studied.

**After:** The card is absent from the page until you complete at least one session today, keeping the Dashboard clean and focused on what's actionable now. When the card is visible it shows the **last 5 sessions** across all time (sorted newest-first) so your most recent work is always at the top.

---

#### 03 — Add Task Modal: Layout Fix + Smart Start Time Rounding

Two UX improvements to the Add Task modal:

**Layout fix:** The Start Time and End Time fields were previously placed in a two-column CSS grid. On narrow modals and mobile viewports the End Time input overflowed its container, clipping the input border and label. Both fields are now rendered as full-width stacked rows, matching every other field in the modal.

**Start Time rounding:** When the modal opens, the Start Time field is pre-filled with the current time rounded up to the nearest 15-minute mark:

| Current time | Pre-filled value |
|---|---|
| 1:20 PM | 1:30 PM |
| 1:32 PM | 1:45 PM |
| 1:45 PM | 1:45 PM (already on boundary) |
| 1:46 PM | 2:00 PM |

This avoids ugly fractional start times like `13:47` and aligns with how people naturally think about scheduling.

---

#### 04 — Recurring Tasks: Gated Occurrence Display

Future occurrences of a recurring task series are now **hidden from the Task Board** until the immediately preceding occurrence is completed. Undoing a completion (un-checking a task) re-hides the next occurrence instantly.

**Logic (two-pass algorithm):**
1. Children are sorted by `startTime` ascending within each `_recurParent` group.
2. All uncompleted children after the first uncompleted child in the sorted list are added to a hidden set.
3. If the **parent task itself** is uncompleted, all uncompleted children are hidden regardless of their individual completion state.

**Calendar:** All occurrences remain visible in the Calendar views regardless of completion gating, so the full schedule is always accessible for planning.

---

#### 05 — Floating Pet: Full Visual Overhaul

The floating pet companion received the most significant visual update since its introduction in v5.1.

**Widget idle animation:** The entire widget now gently **floats up and down** on a 3-second ease-in-out cycle, pausing when hovered.

**Gradient-border ring:** The circular ring now uses a CSS `background-clip: border-box` technique, producing a **transparent interior** with a true gradient border — the border colour flows from `--primary` through `--accent` to `--accent2`. Previous versions used a filled gradient circle that obscured the background behind the pet.

**Pulsing glow:** The ring emits a **soft pulse animation** (`petRingPulse`) that cycles in sync with the widget float — expanding a shadow halo outward on the downstroke and contracting it on the upstroke.

**Ring hue progression:** As the pet levels up, the ring gradient **hue-shifts from blue toward teal-green**, giving a visual sense of growth. The shift is computed dynamically in `updateFloatingPet()` based on the current pet level index (0–9).

**Level pip:** A small `Lv.X` badge appears in the top-left corner of the widget **on hover**, so you can always check your pet's current level without navigating to the Profile page.

**Richer click messages:** Clicking the pet now draws from a wider pool of context-aware messages including: pending task count, current streak, pet stage name and level, and player XP/level — in addition to the existing time-of-day greetings.

**Level-up celebration:** Every player level-up (not just profile-page visits) now triggers the `react` animation on the floating widget and shows a speech bubble like `🎉 Evolved to Juvenile!` when the level-up causes a pet stage evolution.

---

#### 06 — GPA: Exact Interpolated Score-to-GPA Conversion

`scoreToGPA()` now **linearly interpolates within each grade band** rather than returning a fixed step value at the band floor.

| Score | Old value | New value |
|---|---|---|
| 50% (B- floor) | 2.70 | 2.70 |
| 52.5% (midpoint of B- band) | 2.70 | **2.85** |
| 54.9% (near B ceiling) | 2.70 | **2.99** |
| 55% (B floor) | 3.00 | 3.00 |

The interpolation uses a lookup table of `[lowerBound, upperBound, gpaAtLower, gpaAtUpper]` tuples, so conversion accuracy is maintained across all 12 grade bands (A+ → F). Weighted GPA calculations throughout the app (dashboard stat card, GPA Tracker, badge checks) all use this function and benefit automatically.

---

#### 07 — Mark Tracker: Modals Fixed (Critical HTML Bug)

The **Add Mark** and **Subjects** modals in the Mark Tracker tab were completely non-functional in v5.2 due to a critical HTML nesting error.

**Root cause:** The `modal-profile` overlay element was opened (`<div class="modal-overlay" id="modal-profile">`) but never closed before the `modal-mark` and `modal-mark-subjects` overlays began. This caused all three modals to be nested *inside* the `modal-profile` overlay, making them unreachable via `classList.add("open")` because the outer overlay was itself never opened.

**Fix:** The `modal-profile` overlay is now correctly self-contained with its own opening and closing tags. All other modal overlays are siblings at the same DOM level. The `openManageMarkSubjects()` function also gained a `data.markSubjects` initialisation guard to prevent rendering before the array exists.

---

#### 08 — Settings: 2-Column Layout Restored

The Settings page is back to the **classic 2-column grid** layout:

| Left column | Right column |
|---|---|
| Notifications card | Data Management + About card |
| Appearance card | |

**Before (v5.2 intermediate state):** A single stacked column with four separate cards was accidentally introduced during the fix pass, making the settings page much taller than necessary.

**After:** The grid-2 wrapper places Notifications and Appearance in the left column and Data Management + About in the right column, matching the original compact side-by-side design.

---

#### 09 — Notification System: Rebuilt from Scratch

The push notification system was replaced with a fully new implementation.

**Stacked banner queue:** Notifications now render into a `#notif-stack` container column anchored to the top-right of the screen. Up to 4 banners are visible simultaneously; when the limit is reached the oldest is auto-dismissed before the new one enters. Each banner:
- Slides in from the right with a spring animation (`notifSlideIn`)
- Has a **✕ close button** in the top-right corner
- Has a **colour-coded left accent border** matching the notification urgency (red for exams, orange for warnings, teal for informational)
- Has a **progress bar** that drains over 5 seconds, giving a clear visual signal of when the banner will auto-dismiss
- Slides out upward when dismissed, with `max-height` collapsing to 0 so the remaining banners shift up smoothly

**Richer scheduling:**

| Trigger | When | Accent |
|---|---|---|
| Exam — 7 days before | 7 days before exam | Orange |
| Exam — 24 hours before | 24 h before exam | Red |
| Exam — 1 hour before | 1 h before exam | Red |
| Task due — 1 hour before | 1 h before `dueDate` | Orange |
| Task due — 30 minutes before | 30 min before `dueDate` | Red |
| Daily study reminder | 9 AM if no session logged yet today | Teal |

The 7-day exam warning and daily study reminder are new in v5.3. Toggling notifications OFF now immediately clears all pending `setTimeout` timers.

---

#### 10 — XP & Pet Leveling: Dramatically Reduced Threshold

The XP required to level up has been changed from a **scaling `level × 100`** formula to a **flat 50 XP per level**.

| Metric | v5.2 | v5.3 |
|---|---|---|
| XP to reach Level 2 | 100 XP | 50 XP |
| XP to reach Level 5 | 500 XP | 50 XP |
| XP to reach Level 10 | 1,000 XP | 50 XP |
| Tasks needed to level up (approx.) | 5–50 | ~3 |

`getPetLevel()` now advances **every single player level** (previously every 2), so the pet visibly evolves far more frequently:

| Player level | Pet stage |
|---|---|
| 1 | Lv.1 Hatchling |
| 2 | Lv.2 Chick |
| 3 | Lv.3 Fledgling |
| … | … |
| 10+ | Lv.10 Legendary ✨ MAX |

The `renderPet()` level label now shows `"evolves at player Lv.X"` so you always know how close the next evolution is.

---

#### 11 — Study Timer: Expandable Session History

The Today's Sessions list in the Study Timer has been redesigned with two tiers:

**Preview (always visible):** The last 3 sessions from today are shown as a clean list with session title, time, and a coloured duration pill.

**Expandable full history:** A **Show All** button (with a rotating chevron) in the card header reveals every session ever logged, sorted newest-first, with a green "Today" badge on today's entries. Clicking the button again collapses the list. The panel animates open and closed using `max-height` CSS transitions for a smooth feel. The panel resets to collapsed whenever you navigate away and return to the Timer page.

The toggle button is hidden entirely when fewer than 4 total sessions exist (no need to expand if everything already fits in the preview).

---

### 💾 Data Schema Changes

No new fields introduced in v5.3. All existing v5.2 saves are fully forward-compatible.

---

### ✅ Verification

| Test | Result |
|---|---|
| Charts destroyed on navigation, recreated without flicker | ✅ Verified |
| Dashboard sessions card hidden when no sessions today | ✅ Verified |
| Dashboard sessions card shows max 5 entries | ✅ Verified |
| Add Task modal End Time no longer overflows | ✅ Verified |
| Add Task Start Time rounds to next 15-min boundary | ✅ Verified |
| Recurring task second occurrence hidden until first is done | ✅ Verified |
| Completing first occurrence reveals second occurrence | ✅ Verified |
| Un-checking first occurrence re-hides second occurrence | ✅ Verified |
| Calendar always shows all recurring occurrences | ✅ Verified |
| Pet widget floats with idle animation | ✅ Verified |
| Pet ring is transparent inside, gradient border | ✅ Verified |
| Pet ring pulses with glow animation | ✅ Verified |
| Level pip appears on hover | ✅ Verified |
| Ring hue shifts as pet levels up | ✅ Verified |
| Level-up triggers react animation on floating widget | ✅ Verified |
| 52.5% score returns GPA ~2.85 (not flat 2.70) | ✅ Verified |
| Add Mark modal opens and saves correctly | ✅ Verified |
| Manage Mark Subjects modal opens and saves correctly | ✅ Verified |
| Settings renders in 2-column grid layout | ✅ Verified |
| Notifications stack up to 4 banners | ✅ Verified |
| Oldest banner auto-dismissed when limit exceeded | ✅ Verified |
| Progress bar drains over 5 seconds | ✅ Verified |
| ✕ close button dismisses individual banner | ✅ Verified |
| 7-day exam warning fires correctly | ✅ Verified |
| Daily 9 AM study reminder fires when no session today | ✅ Verified |
| Disabling notifications clears all pending timers | ✅ Verified |
| Level-up requires exactly 50 XP | ✅ Verified |
| Pet evolves at every player level (not every 2) | ✅ Verified |
| Timer page shows last 3 today's sessions in preview | ✅ Verified |
| Show All expands full session history | ✅ Verified |
| Show Less collapses history panel | ✅ Verified |
| Session panel resets to collapsed on page navigation | ✅ Verified |

---

*StudyFlow v5.3.0 — Every interaction, alive.*

---

## v5.2.0 — "Clean Slate" Update (March 18, 2026)

> A focused polish and consolidation release. v5.2 removes the experimental database/login/leaderboard systems introduced in v5.1 to keep StudyFlow a clean, zero-dependency, single-file app. In their place: a rearchitected Grade Tracker with a proper tabbed UI, a redesigned Mark Tracker experience with a full add-mark modal, a relocated dark mode toggle, an animated floating pet with reactive celebrations, and a series of UX refinements across the board.

---

### 🟢 New & Changed Features

#### 01 — Dark Mode Toggle Moved to Settings

The dark/light theme toggle has been removed from the sidebar footer and relocated to **Settings → Appearance**, where it lives alongside all other preference controls.

**Before:** A small toggle at the bottom of the sidebar — hidden when the sidebar is collapsed, invisible on mobile.

**After:** A full-width labelled button in a dedicated **Appearance** card in Settings, accessible from any screen size. The button shows the current theme name ("Light Mode" / "Dark Mode") and the matching sun/moon icon. In dark mode the button takes on an accent-purple tint to visually confirm the active state.

This change makes theme switching equally accessible on desktop, tablet, and mobile without requiring the sidebar to be open.

---

#### 02 — Grade Tracker — Unified Tabbed Interface

The **Grade Tracker** and **Mark Tracker** (introduced separately in v5.1) are now consolidated into a single sidebar section named **Grade Tracker**, reducing sidebar clutter.

**Navigation:** One nav item → "Grade Tracker"

**Inside the section:** Two tabs at the top of the page switch between the two tools:

| Tab | Tool | Subtitle |
|---|---|---|
| GPA Tracker | Weighted GPA calculation, module grades, grade chart (unchanged from v5.1) | "Track your academic performance by module" |
| Mark Tracker | Simple mark entry and analytics | "Simple mark analytics — Just your scores" |

Each tab shows its own action buttons in the page header (GPA tab shows **Subjects** + **Add Grade**; Marks tab shows **Subjects** + **Add Mark**). Navigating away from the page and returning always resets to the GPA Tracker tab.

---

#### 03 — Mark Tracker — Full Add Mark Modal

The previous `prompt()` chain for adding marks (three sequential browser dialogs) has been replaced with a proper **Add Mark modal**, consistent with the rest of the app's UX.

**Modal fields:**

| Field | Type | Notes |
|---|---|---|
| Subject | Dropdown | Populated from the Mark Tracker's own subject list (see below) |
| Assessment Name | Text | Required — e.g. "Mid-term Exam", "Assignment 3" |
| Marks Scored | Number | Required |
| Total Marks | Number | Required; scored cannot exceed total |
| **Live preview** | Auto | Percentage and grade letter update in real time as values are typed |
| Date | Date picker | Defaults to today |
| Notes | Text (optional) | Free-form annotation |

**Live percentage preview:** Appears between the score inputs and the date field as soon as both numbers are present — shows the calculated percentage in large text and the corresponding grade letter in a coloured pill, giving instant feedback before the mark is saved.

---

#### 04 — Mark Tracker — Independent Subject List

The Mark Tracker now maintains its own subject list (`data.markSubjects`), completely separate from the GPA Tracker's subject list (`data.subjects`). This allows school students and non-university users who don't have modules or credit-weighted subjects to use the Mark Tracker without needing to set up a GPA-oriented subject structure.

**Manage Subjects button** (visible only on the Mark Tracker tab) opens a dedicated **Mark Tracker Subjects** modal where subjects can be added and deleted independently.

Mark entries are grouped by subject in the Mark Tracker list view, with a per-subject average bar shown at the top of each group.

---

#### 05 — Mark Tracker — Grouped List View

Marks are now displayed grouped by subject rather than as a flat chronological list.

Each subject group shows:
- Subject name and the group's average percentage with a coloured progress bar
- Individual entries sorted newest-first within the group, each showing assessment name, scored/total, date, notes, grade letter pill, and percentage
- Delete button per entry

---

#### 06 — Floating Pet — Circle Ring & Reaction Animations

Two visual upgrades to the floating pet companion:

**Circle ring:** The pet SVG now sits inside a **72×72px circular container** with a gradient background (primary → accent) and a soft primary-colour glow drop-shadow. This gives the pet a clear defined boundary at all times and makes it more visually distinct against both light and dark backgrounds.

**Reaction animations** — the pet now visibly reacts when something good happens:

| Trigger | Animation | Message |
|---|---|---|
| Task completed | Spin-wobble (`petReact` keyframe) + sparkle burst | "Task done! 🎉" |
| Study session complete | Spin-wobble + sparkle burst | "+20 XP! ⭐" |
| Mark added | Spin-wobble + sparkle burst | "📊 Mark added!" |
| User clicks pet | Bounce animation | Contextual message |

**Sparkle burst:** A circular SVG overlay (70×70px) animates outward from the pet's position and fades out over 0.7 seconds. Rendered as a fixed-position DOM element appended to `<body>` and removed after the animation completes — no layout impact.

---

### 🔴 Removed

#### Login / OTP System

The email-based login, sign-up, and OTP verification interface introduced in v5.1 has been removed.

**Reason:** The single-file, no-server architecture means any "login" system is necessarily a client-side simulation. Storing credentials in `localStorage` and simulating OTP via a browser toast provided no real security or cross-device sync benefit, and added significant UI complexity (login screen, signup panel, OTP input grid, account management, data namespacing by email) that distracted from the core study tools.

**Impact on data:** Existing `studyflow-v2` data is fully preserved. Users who created accounts in v5.1 can continue using the app — data is simply no longer gated behind a login screen.

---

#### Leaderboard

The XP leaderboard (All Time / This Week / Today tabs) has been removed.

**Reason:** Without a real backend, the leaderboard was seeded with fictional demo accounts and only tracked the current device's user. A simulated leaderboard adds no genuine competitive value and was removed to keep the sidebar clean. The Insights section now contains only Analytics and Profile & XP.

---

#### MySQL Database Integration

All references to MySQL, server-side storage, and cross-device sync have been removed. StudyFlow remains a fully **client-side, single-file application** with `localStorage` as the only data store.

---

### 💾 Data Schema Changes

| Field | Change |
|---|---|
| `data.markSubjects` | New (carried from v5.1) — `[{ id, name }]`; independent Mark Tracker subject list |
| `data.accounts` | **Removed** — login simulation data no longer stored |
| `data.leaderboard` | **Removed** — leaderboard cache no longer stored |
| `data.user.loggedIn` | **Removed** — login gate no longer exists; all users go directly to `initApp()` |
| `mark.subject` | New — subject name string (from `markSubjects`), stored per mark entry |
| `mark.notes` | New — optional free-text annotation per mark entry |

All v5.1 saves are forward-compatible. `data.accounts` and `data.leaderboard` keys are silently ignored if present.

---

### ✅ Verification

| Test | Result |
|---|---|
| Dark mode toggle accessible from Settings on mobile | ✅ Verified |
| Dark mode toggle absent from sidebar on all screen sizes | ✅ Verified |
| Grade Tracker tab defaults to GPA Tracker on navigation | ✅ Verified |
| Switching to Mark Tracker tab shows correct actions | ✅ Verified |
| Add Mark modal opens with pre-populated today date | ✅ Verified |
| Live % preview updates as scored/total are typed | ✅ Verified |
| Scored > total blocked with toast error | ✅ Verified |
| Mark subjects independent of GPA subjects | ✅ Verified |
| Marks grouped by subject in list view | ✅ Verified |
| Pet ring visible on light and dark backgrounds | ✅ Verified |
| Pet react animation fires on task completion | ✅ Verified |
| Pet react animation fires on session completion | ✅ Verified |
| Sparkle burst appears and cleans up after 0.7s | ✅ Verified |
| No login screen on app open | ✅ Verified |
| No leaderboard nav item in sidebar | ✅ Verified |
| Existing localStorage data loads correctly | ✅ Verified |

---

*StudyFlow v5.2.0 — Focused, clean, and yours.*

---

## v5.1.0 — "War Room" Update (March 18, 2026)

> The biggest feature release since v3.0. v5.1 adds five brand-new systems (Recurring Tasks, Push Notifications, Floating Pet, War Room Mode, and a rebuilt Calendar), expands the Grade Tracker into two distinct tools, fixes ten bugs that survived v4.3, and raises the motivational quote library from 100 to 250 entries. Every change is fully self-contained in a single HTML file — zero dependencies added.

---

### 🟢 New Features

#### 01 — Recurring Tasks & Task Durations

Tasks can now repeat on a schedule and carry optional start/end timestamps.

**Recurring schedule options (set per task in the Add Task modal):**

| Option | Behaviour |
|---|---|
| No recurrence | Default — one-off task, no change from previous behaviour |
| Daily | Repeats every day |
| Weekdays | Repeats Monday–Friday only; weekends skipped |
| Weekly | Repeats on the same weekday each week |
| Every 2 weeks | Biweekly repeat |
| Monthly | Repeats on the same calendar date each month |

When a recurring task is saved, 8 future occurrences are generated automatically and added to `data.tasks`, each carrying a `_recurParent` ID linking them back to the original. Start and end times are offset by the same interval so schedules stay aligned.

**Task duration fields (all optional):**
- **Start time** — `datetime-local` input; when quick-adding inline, start time is automatically set to the moment of creation so calendar placement is always correct
- **End time** — `datetime-local` input; paired with start time to show a time-range chip (e.g. `⏰ 09:00 → 10:30`) on each task card

**Recurring badge** — tasks with an active recurrence pattern display a small `🔁 weekly`-style pill next to the task name in the list view and on the calendar.

---

#### 02 — Push Notifications

In-app and native browser notifications for exam deadlines and task due dates. Notifications are **suppressed entirely when Focus Mode is active** to prevent interruptions during a study session.

**Notification triggers:**

| Event | When |
|---|---|
| Exam — 24 hours before | 24 h before the exam datetime |
| Exam — 1 hour before | 1 h before the exam datetime |
| Task due — 30 minutes before | 30 min before `task.dueDate` |
| Focus session complete | Immediately on session end (if not in Focus Mode) |

**Implementation:**
- Browser `Notification` permission is requested when the user enables push notifications in Settings via a clear opt-in toggle
- If permission is denied, the toggle reverts to off automatically and a guidance toast is shown
- In-app banners (`push-banner`) display for all users regardless of native permission status, providing a fallback
- `setTimeout`-based schedule is rebuilt on every app load and whenever an exam or task is added/edited
- New **Push Notifications** toggle added to Settings → Notifications (off by default)

---

#### 03 — Floating Pet Companion

The virtual companion (previously only visible on the Profile & XP page) is now a persistent floating widget anchored to the bottom-right corner of every page.

**Behaviour:**
- Pet SVG is rendered inside a circular gradient container with a soft primary-colour glow ring
- Clicking the pet triggers a bounce animation, shows a contextual speech bubble, and awards +1 XP
- Speech bubbles are context-aware: time of day (morning / afternoon / evening), pending task count, and current XP level are all factored into the message
- An automatic subtle message appears every 12 minutes (non-intrusive, no sound) to maintain presence without being distracting
- Pet type and level stay in sync with the Profile & XP page — changing the pet or levelling up is reflected instantly in the floating widget

---

#### 04 — War Room Mode

An auto-activating single-focus study mode triggered when an exam is within 24 hours.

**Activation:** War Room checks run every 5 minutes. When any upcoming exam is detected within 24 hours, a full-screen overlay activates showing:
- Exam subject and a live `HH:MM:SS` countdown to the exam
- A filtered list of tasks related to the exam subject (matched by task text and category name)
- A **Start Focus Session** shortcut that exits War Room and navigates directly to the Study Timer

**Deactivation:** The user can dismiss the overlay manually at any time via the ✕ button. Dismissal does not prevent the mode from re-activating on the next 5-minute check if the exam is still within 24 hours.

**Settings:** A new **War Room Mode** toggle in Settings → Notifications controls whether the feature is active. It is **off by default** — users must explicitly opt in.

---

#### 05 — Study Heatmap with Outcome Correlation

The 28-day activity heatmap on the Analytics page now cross-references study sessions with grade entries to surface study-vs-performance insights.

**Visual changes:**
- Heatmap cells that fall within ±3 days of a recorded grade entry gain a coloured border:
  - 🟢 Green border — grade ≥ 70%
  - 🟡 Amber border — grade 50–69%
  - 🔴 Red border — grade < 50%
- Cell tooltip updated to include `| Nearby exam: 87%` when applicable

**Correlation insight card** — displayed below the heatmap whenever correlated data exists:
> 💡 **Calculus II**: You studied ~3.5h in the 5 days before and scored **87%**

The insight uses the most recent correlated grade entry. If no grade entries exist within the date range, the heatmap behaves identically to v4.3.

---

#### 06 — Login Interface (Local Simulation)

A login and sign-up flow providing a per-user data experience within the single-file app.

**Sign-up flow:**
1. Enter full name, email address, and password (8+ characters)
2. Disposable/temporary email domains are blocked (mailinator, guerrillamail, tempmail, yopmail, and 11 others)
3. A 6-digit OTP is generated; in this local simulation the code is shown in a toast for demo purposes. In a production deployment, this would be sent via SMTP
4. Entering the correct OTP creates the account and starts the session

**Log-in flow:**
- Email + password match against locally stored accounts
- Passwords are stored using FNV-1a 32-bit hash concatenated with the length as a hex suffix — never stored in plaintext
- Per-user data is stored under `sf-data-{email}` in `localStorage`, allowing multiple accounts on the same device

**Skip option:** Users can continue without an account — all data is stored under the default `studyflow-v2` key exactly as in previous versions.

---

#### 07 — Leaderboard

A community XP ranking board showing the top scholars across three time windows.

**Period tabs:**

| Tab | Ranks by |
|---|---|
| All Time | Total XP ever earned |
| This Week | XP earned in the current 7-day window |
| Today | XP earned in the last 24 hours |

**Privacy protections:**
- Only name, designation, level, and XP are visible for other users — tasks, grades, exams, and sessions are never exposed
- A persistent privacy note ("Only names, designations and XP are visible. Tasks and personal data are never shared.") is displayed above all leaderboard entries
- Avatar thumbnails are shown only for users who have uploaded a profile photo; no placeholder photo is generated from other users' data

The leaderboard is seeded with 5 demo entries on first load so the board is never empty. Real entries replace demo entries as accounts are created.

---

#### 08 — Calendar — Weekly & Daily Views

The Calendar page now supports three view modes selectable via a tab strip: **Month** (existing), **Week** (new), and **Day** (new).

**Weekly view:**
- 7-column hour-by-hour grid (24 rows × 7 days)
- Exams, tasks (keyed to `startTime` or `dueDate`), and study sessions appear as colour-coded chips in their correct hour slot
- Clicking any day column header switches to the Day view for that date
- Navigation via Prev/Next moves the window by one full week

**Daily view:**
- 24-row single-day timeline with one-hour resolution
- Events are rendered as full-width chips inside their hour row, sorted by start time
- Recurring task instances appear with a `🔁` marker
- On load, the view automatically scrolls to the current hour
- Navigation via Prev/Next moves one day at a time

**Month view improvements:**
- Calendar grid cells and the day-detail panel now use the same `dueDate`-first, `createdAt`-fallback logic — clicking a cell shows exactly the same events rendered on the cell (Bug 11 fix, see below)
- Events in the day-detail panel are sorted by time (exams → tasks → sessions, each group sorted by start time)
- Task items in the panel include a live checkbox so tasks can be completed directly from the calendar

---

#### 09 — Grade Tracker Rename + Mark Tracker

**GPA Tracker renamed to Grade Tracker** in the sidebar, page title, and all internal references.

**Mark Tracker (new standalone section)** added to the sidebar under Grade Tracker:

| Feature | Detail |
|---|---|
| Subjects | Separate subject list, independent of the GPA Tracker subject list |
| Add Mark modal | Assessment name, scored/total fields, date |
| Analytics | Overall average %, highest %, lowest %, entry count |
| Grade letter | Computed from percentage: A+ (≥85) through F (<40) |
| Bar chart | Per-entry coloured progress bar — green ≥70%, amber ≥50%, red <50% |
| Slogan | *"Simple mark analytics — no GPA, just your scores"* |

The Mark Tracker is designed for non-university students, school students, and anyone who tracks performance by raw marks rather than weighted grade points.

---

### 🔴 Bug Fixes

**Bug 11 — Calendar grid and day-detail panel showing different events for the same date**
`renderCalendar()` filtered tasks by `dueDate` first, falling back to `createdAt` — so a task created on March 1st with a due date of March 15th appeared in the March 15th grid cell. `showCalDay()` filtered only by `createdAt`, so clicking that cell showed nothing. Applied identical `dueDate`-first logic to `showCalDay()`. Both code paths now always agree.

**Bug 12 — Drag-and-drop state leak between the Task Board and Eisenhower Matrix**
Two separate drag-state variables (`taskDragId` and `matrixDragId`) existed in parallel. `dropToQuadrant()` checked only `taskDragId`, causing matrix drags to silently fail or use a stale ID when both systems had been interacted with in the same session. Unified to a single `currentDragId`; `matrixDragStart()` now writes to `taskDragId` directly so all drop handlers read from one source of truth.

**Bug 13 — Duplicate subject names allowed due to case-sensitive comparison**
`addSubject()` checked `s.name === name` (exact match), so `"Math"` and `"math"` were treated as different subjects and both saved. Changed to `s.name.toLowerCase() === name.toLowerCase()` throughout all subject deduplication paths.

**Bug 14 — Exam past-date validation bypassable via browser DevTools**
`openAddExam()` set `input.min = today` in HTML, but `submitExam()` performed no server-side-style validation. The `min` attribute is trivially editable in DevTools. Added an explicit JS guard in `submitExam()`:
```js
if (new Date(date) < new Date().setHours(0,0,0,0)) {
  showToast("⚠️", "Invalid date", "Cannot add exam in the past");
  return;
}
```

**Bug 15 — Session counter growing unbounded across Pomodoro cycles**
`sessionCount` incremented indefinitely. The modulo arithmetic `sessionCount % sessionsBeforeLong` produced correct results but the raw counter grew without limit, causing subtle issues with session dot rendering on very long sessions. After each long break, `sessionCount` is now reset to `0`.

**Bug 16 — Session title lost on page refresh during an active session**
The session title input was an ephemeral DOM value — if the page was refreshed mid-session the title was gone and the completed session was logged as `"Study Session"`. Title now persists to `data.currentSessionTitle` on every keystroke (`oninput`) and is restored from `data` during `init()`.

**Bug 17 — Charts destroyed and recreated on every navigation, causing visual flicker**
`renderAnalytics()` and `renderGrades()` called `chart.destroy()` then `new Chart()` every time the Analytics or Grade Tracker section was opened. Replaced with `chart.update()` when a chart instance already exists, falling back to `new Chart()` only on first render. Eliminates the white-flash flicker when switching pages.

**Bug 18 — Quote index not restored on page load; always starting from quote 0**
`data.quoteIndex` was written to `localStorage` correctly but never read back during `init()`. Added `if (typeof data.quoteIndex !== 'number') data.quoteIndex = 0` to the data migration block so the saved index is always honoured. Quote count also increased from **100 to 250 entries**.

**Bug 19 — New `AudioContext` created on every `playSound()` call**
Each invocation of `playSound()` was constructing a `new AudioContext()`, violating browser best-practice limits (Chrome logs a warning after 6 concurrent contexts) and causing resource exhaustion on extended sessions. Replaced with a module-level singleton `_audioCtx` variable initialised once on first use and reused for all subsequent calls.

**Bug 20 — No browser notification permission requested**
The app had no mechanism to request `Notification` permission, meaning deadline alerts could never fire as native notifications. Permission is now requested via `Notification.requestPermission()` when the user enables Push Notifications in Settings (see Feature 02 above). The request is triggered only by an explicit user action, satisfying browser autoplay/interaction requirements.

---

### 💾 Data Schema Changes

| Field | Change |
|---|---|
| `task.startTime` | New — ISO datetime string; set to creation time on quick-add |
| `task.endTime` | New — ISO datetime string; optional end time for duration display |
| `task.recur` | New — one of `"none"`, `"daily"`, `"weekdays"`, `"weekly"`, `"biweekly"`, `"monthly"` |
| `task._recurParent` | New — ID of the originating task for generated recurring occurrences |
| `data.marks` | New — Mark Tracker entry array: `[{ id, subject, name, scored, total, date, notes }]` |
| `data.markSubjects` | New — Mark Tracker subject list: `[{ id, name }]` (separate from GPA subjects) |
| `data.currentSessionTitle` | New — persisted session title string; restored on init |
| `data.settings.warRoom` | New — boolean, default `false` |
| `data.settings.pushNotif` | New — boolean, default `false` |
| `data.accounts` | New — local account store for login simulation |
| `data.leaderboard` | New — leaderboard entry cache |

All existing `studyflow-v2` saves are forward-compatible. Missing new fields are silently back-filled with their defaults on load.

---

### ✅ Verification

| Test | Result |
|---|---|
| Recurring task generates 8 future occurrences | ✅ Verified |
| Quick-add sets `startTime` to creation time | ✅ Verified |
| Push notification banner suppressed in Focus Mode | ✅ Verified |
| Floating pet renders on all 9 pages | ✅ Verified |
| War Room triggers within 5 min of exam entering 24h window | ✅ Verified |
| War Room off by default; settings toggle works | ✅ Verified |
| Heatmap shows coloured borders near grade entries | ✅ Verified |
| Correlation insight card appears when data exists | ✅ Verified |
| Calendar weekly view shows correct events per hour | ✅ Verified |
| Calendar daily view auto-scrolls to current hour | ✅ Verified |
| Month/week/day views share consistent event data | ✅ Verified |
| Mark Tracker subjects independent of GPA subjects | ✅ Verified |
| Bug 11 — clicking any calendar cell shows correct events | ✅ Verified |
| Bug 12 — matrix drag no longer uses stale ID | ✅ Verified |
| Bug 13 — "Math" and "math" treated as same subject | ✅ Verified |
| Bug 14 — past exam date rejected even via DevTools edit | ✅ Verified |
| Bug 15 — session counter resets after long break | ✅ Verified |
| Bug 16 — session title survives page refresh | ✅ Verified |
| Bug 17 — no flicker when switching Analytics/Grades tabs | ✅ Verified |
| Bug 18 — quote index restores correctly after reload | ✅ Verified |
| Bug 19 — single AudioContext across full session | ✅ Verified |
| Bug 20 — permission prompt fires on toggle enable | ✅ Verified |

---

*StudyFlow v5.1.0 — Smarter study. Deeper focus. Always with you.*

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
