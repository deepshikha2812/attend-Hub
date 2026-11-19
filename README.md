# Role: Design Lead 
# Student Self-Attendance Tracker

A simple, offline, single-page web app that lets students track their own attendance, see exactly where they stand against their minimum attendance requirement, and get early warnings before they fall short.

> Project for **Software Design Engineering**. Data is stored in the browser's local storage under the key prefix `attendly`.

---

## Table of Contents

- [Features](#features)
- [Design Goals](#design-goals)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [How Calculations Work](#how-calculations-work)
- [Status and Alerts](#status-and-alerts)
- [Core Flow](#core-flow-marking-todays-attendance)
- [User Interface](#user-interface)
- [Validation Rules](#validation-rules)
- [Getting Started](#getting-started)
- [Limitations](#limitations)
- [Requirement Traceability](#requirement-traceability)

---

## Features

- Register and log in with a local account
- Add subjects, each with its own minimum attendance % (or use a global default)
- Build a weekly timetable for Section A or B
- Mark classes as **Attended**, **Missed** or **Off**, one at a time or in bulk
- Edit past days from the calendar view
- Live attendance %, **safe skips** and **recovery classes needed** for every subject
- Early-warning alerts in a notification (bell) drawer
- Works fully offline, with no server

## Design Goals

| Goal | Meaning |
|---|---|
| **Simple** | Single-page app with five tabs |
| **Accurate** | All percentages and buffers come from one Calculator |
| **Modular** | UI, state, logic, alerts and storage are kept separate |
| **Early warning** | Alerts when below or approaching the Min % |
| **Offline** | No server; uses browser local storage |

**Approach:** object-oriented and component-based, with high cohesion and low coupling.

## Architecture

The app is organised into four layers:

| Layer | Responsibility |
|---|---|
| **Presentation** | Auth screen, header, 5 tabs, Quick Add modal, notification drawer |
| **State** | Central state: section, subjects, timetable, records, default min |
| **Logic** | Attendance Calculator, Alert Service, Validator |
| **Storage** | Storage Service over local storage |

**Data flow:**

```
action → state → calculate → alerts → save → re-render
```

Tabs and the header talk only to the central state. The Calculator and Validator are **pure functions** with no storage access and no side effects.

## Project Structure

Components grouped by role:

| Group | Components |
|---|---|
| UI | `AuthScreen`, `TrackerShell`, `HeaderBar`, `QuickAddModal`, `NotificationDrawer` |
| Tabs | `Today`, `Timetable`, `Calendar`, `Subjects`, `Settings` |
| Logic | `AttendanceCalculator`, `AlertService`, `Validator` |
| Services | `AuthService`, `StorageService` |

### Classes and main operations

| Class | Main operations |
|---|---|
| `User` | `register()`, `login()`, `logout()` |
| `Subject` | `add()`, `edit()`, `delete()`, `effectiveMin()` |
| `TimetableSlot` | `addSlot()`, `removeSlot()`, `slotsFor()` |
| `AttendanceRecord` | `mark()`, `update()`, `clear()`, `bulkMark()` |
| `AttendanceCalculator` | `percentage()`, `safeSkips()`, `recoveryNeed()` |
| `AlertService` | `statusOf()`, `buildAlerts()` |
| `StorageService` | `loadWorkspace()`, `saveWorkspace()` |

## Data Model

| Entity | Key attributes |
|---|---|
| User | email, name, passwordHash |
| Workspace | section (A/B), defaultMin |
| Subject | id, name, minPct, manual counts |
| TimetableSlot | subjectId, section, day, startTime |
| AttendanceRecord | slotId, date, status (`ATTENDED` / `MISSED` / `OFF`) |

**Relationships:** User 1–1 Workspace · Workspace 1–N Subject · Subject 1–N Slot and Record.

**Storage:** JSON per user, under the keys `attendly-users` and `attendly-data-<email>`.

Percentages and alerts are **calculated, never stored**.

## How Calculations Work

Definitions:

- `A` = classes attended
- `M` = classes missed
- `T` = `A + M` (total conducted; **Off** days are excluded)
- `m` = minimum required %

| Metric | Formula |
|---|---|
| Attendance % | `A / T × 100` |
| Safe skips | `max(0, floor(100·A/m − T))` |
| Recovery classes needed | `ceil((m·T − 100·A) / (100 − m))` |

**Example:** `A = 12`, `M = 8` (60%)

- With `m = 75`: attend the next **12** classes in a row, reaching 24/32 = 75%
- With `m = 50`: you can safely skip **4** classes, ending at 12/24 = 50%

## Status and Alerts

| Status | Condition |
|---|---|
| 🟢 **Safe** | `% ≥ Min % + margin` |
| 🟡 **Warning** | `Min % ≤ % < Min % + margin` (default margin = 5) |
| 🔴 **Deficit** | `% < Min %` |

Alerts are recalculated after every change. Warning and Deficit subjects appear in the bell drawer and clear automatically once the subject is Safe again.

## Core Flow: Marking Today's Attendance

1. **Student marks class**: Attended / Missed / Off
2. **Validate**: status, date, one record per slot
3. **Update state**: add, change or clear the record
4. **Calculate**: %, safe skips, recovery
5. **Check alerts**: compare against Min %
6. **Save**: write to local storage
7. **Refresh UI**: header, progress rings, bell

## User Interface

Five tabs: **Today · Timetable · Calendar · Subjects · Settings**

| Area | What it does |
|---|---|
| Header | Overall %, Target %, Quick Add, notification bell |
| Today | Mark classes singly or in bulk |
| Timetable | Choose section A/B and manage slots |
| Calendar | See attendance dots and edit past days |
| Subjects | Progress rings, safe skips, recovery needed |
| Settings | Global Min %, reset demo data, logout |

## Validation Rules

- Attended count must not exceed classes conducted; no future dates
- Min % must be between 1 and 100
- Timetable slots must not overlap
- One record per slot per date (updated, never duplicated)
- Deleting a subject requires confirmation

## Getting Started

> Update the commands below to match your actual setup.

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd student-attendance-tracker

# 2. Install dependencies (if any)
npm install

# 3. Start the app
npm start
```

If the project is plain HTML/CSS/JS, open `index.html` directly in a modern browser.

**First use:** register an account, add your subjects, set up the timetable, then start marking attendance from the **Today** tab.

## Limitations

- Works on **one device only**; there is no sync across devices
- **No reminders** are sent when the app is closed
- Data lives in browser local storage, so clearing site data erases it

## Requirement Traceability

| Requirement | Design element |
|---|---|
| Register / log in | `AuthScreen`, `AuthService` |
| Subjects and criteria | Subjects tab, Settings, Quick Add |
| Enter and edit attendance | Today, Calendar, Quick Add |
| Calculate % | `AttendanceCalculator` |
| Classify and alert | `AlertService`, bell drawer, header |
| Timetable | Timetable and Today tabs |

---

## License

Add your license here (e.g. MIT).
