# 📘 StudyFlow — Smart Study Planner

**Version 4.3** · A modern, all-in-one academic productivity app built as a single HTML file.

> All data is stored locally in your browser via `localStorage`. No server, no sign-up, no tracking.

---

## ✨ Features

### 📊 Dashboard
- Welcome banner with motivational quotes (refresh on click)
- Stat cards: Tasks completed, Study hours, Upcoming exams, Current GPA
- Quick-view of upcoming tasks and recent study sessions
- Virtual study pet that evolves with your XP level

### ✅ Tasks & Eisenhower Matrix
- Add, edit, pin, and delete tasks with category labels
- **Eisenhower Matrix** — tasks are organized into 4 quadrants:
  - 🚨 **Do First** (Urgent + Important)
  - 📅 **Schedule** (Important, Not Urgent)
  - 📤 **Delegate** (Urgent, Not Important)
  - 🗑️ **Eliminate** (Not Urgent, Not Important)
- Full **drag-and-drop** between all quadrants and within quadrants
- Filter by status (All / Today / Yesterday / Last 7 Days) and category
- Due date tracking with overdue alerts
- Custom category management with color coding

### 📅 Exams
- Add exams with subject, date, and time
- Live countdown timers (Days, Hours, Minutes, Seconds)
- Urgency badges (Critical / Soon / Comfortable / Past)
- Color-coded exam cards

### ⏱️ Study Timer (Pomodoro)
- Configurable focus / short break / long break durations
- Visual ring progress indicator with animated countdown
- Session tracking with automatic break cycling
- **Focus Mode** — full-screen distraction-free overlay
- Sound notifications on session completion
- Session history log

### 🏆 GPA & Grade Tracker
- Add assessments with module, name, score (%), and weight
- **Weighted GPA** calculated using credit-based formula
- Per-semester and per-subject filtering
- Stats: Semester average, Best score, Module count
- Grade distribution chart (Chart.js)

### 📆 Calendar
- Monthly calendar view with navigation
- Tasks and exams displayed on their respective dates
- Color-coded event dots

### 📈 Analytics
- Study time chart (weekly bar chart)
- Task completion statistics
- GPA trend tracking
- Powered by [Chart.js 4.4.1](https://www.chartjs.org/)

### 👤 Profile & XP System
- Customizable name, designation, degree, university, email
- Avatar upload with auto-resize
- **Gamification**: Earn XP for completing tasks, study sessions, and pet interactions
- Level progression (1–100) with evolving study pets
- Pet choices: 🦉 Owl, 🦊 Fox, 🐉 Dragon, 🤖 Robot
- Badge system for milestones

### ⚙️ Settings
- Light / Dark theme toggle
- Daily digest notifications
- Sound effects toggle
- Timer duration customization
- **Data export** (JSON backup)
- **Data import** (restore from backup)
- Full data reset

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Structure | HTML5 (single file) |
| Styling | Vanilla CSS with CSS custom properties |
| Logic | Vanilla JavaScript (ES6+) |
| Charts | [Chart.js 4.4.1](https://cdn.chartjs.org/) (CDN) |
| Fonts | [Raleway](https://fonts.google.com/specimen/Raleway) + [Lexend](https://fonts.google.com/specimen/Lexend) (Google Fonts) |
| Storage | `localStorage` (browser-only) |
| Sound | Web Audio API (`AudioContext`) |

**No build tools, no frameworks, no dependencies to install.**

---

## 🚀 Getting Started

### Option 1 — Live Demo
Open the app directly in your browser — no installation needed:
👉 [https://playful-hotteok-0d1547.netlify.app/](https://playful-hotteok-0d1547.netlify.app/)

### Option 2 — Run Locally
Clone the repo and open `index.html` directly in any modern browser:

```bash
git clone https://github.com/Dilhara-De-Alwis/StudyFlow.git
cd StudyFlow
```

Then double-click `index.html` in your file explorer — no server required.

---

## 📱 Responsive Design

Fully responsive with 4 breakpoints:

| Breakpoint | Target |
|------------|--------|
| > 1023px | Desktop — full sidebar |
| ≤ 1023px | Tablet — hamburger menu, collapsible sidebar with backdrop |
| ≤ 640px | Mobile — stacked layouts, full-width modals, compact stat cards |
| ≤ 380px | Small phone — single-column stats, tighter spacing |

Touch-friendly: action buttons always visible (no hover required), larger tap targets on mobile.

---

## 💾 Data & Privacy

- **100% client-side** — all data stays in your browser's `localStorage`
- No server calls, no cookies, no analytics
- Export your data anytime as a JSON file
- Import backups to restore or transfer data between devices

---

## 📂 Project Structure

```
StudyFlow/
├── index.html                            ← Main application (single-file SPA)
├── README.md                             ← This file
├── FULL_CHANGELOG.md                     ← Full version history (v1.0 → v4.3)
├── Enhanced Prompt of Version 1.0.txt    ← Refined AI build prompt for v1.0
├── Written Prompt of Version 1.0.txt     ← Original feature spec for v1.0
├── Written Prompt of Version 2.0.txt     ← Feature spec and bug list for v2.0
├── Written Prompt of Version 3.0.txt     ← Feature spec and bug list for v3.0
└── Written Prompt of Version 4.1.txt     ← Feature spec and bug list for v4.1
```

---

## 🎨 Design

- **Glassmorphism** — frosted glass cards with backdrop blur
- **Aurora background** — animated gradient blobs
- **Dark mode** — full dark theme with smooth transitions
- **Micro-animations** — page transitions, hover effects, XP floats, confetti on achievements
- **Typography** — Raleway (headings) + Lexend (body) for clean readability

---

## 📄 License

This project is created for academic purposes as part of a university assignment.