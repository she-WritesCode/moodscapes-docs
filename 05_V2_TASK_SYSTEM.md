# Moodscapes V2 Task System Architecture

## Overview

In the MVP, tasks were static checklists. In V2, the Task System is completely re-imagined as the central orchestrator of the Wedding OS. **Every subtask is a gateway, not a redirect.**

---

## 1. The Interactive Portal Experience (For Stakeholders)

### 1.1 Tasks as Hubs, Not Just Checklists

When a user opens a task like "Find the perfect Florist," they don't just see a text box that says "Look online." Instead, the system hands them the tools they need to complete it, right inline.

- **Checkboxes** are for simple, manual things (e.g., "Call mom").
- **Portals** are interactive buttons attached to subtasks that connect to other parts of the app.

```bash
[ Choose Wedding Dress          Attire · 6 months out ]

  Progress ━━━━░░░░░░ 0 of 4

  ┌─ Subtask 1 ──────────────────────────────────────┐
  │ ○  Build your dress mood board                   │
  │                                                  │
  │   [ + Start mood board ]  ← tapping this opens   │
  │     the inspiration board IN THIS SHEET,         │
  │     pre-filtered to "Dress" — pins saved here    │
  │     auto-attach to this task                     │
  └──────────────────────────────────────────────────┘

  ┌─ Subtask 2 ──────────────────────────────────────┐
  │ ○  Research bridal boutiques                     │
  │                                                  │
  │   [ + Browse vendors ]  ← opens vendor directory │
  │     filtered to "Bridal" inside this sheet.      │
  │     Shortlisting a boutique links it here.       │
  └──────────────────────────────────────────────────┘

  ┌─ Subtask 3 ──────────────────────────────────────┐
  │ ○  Book appointments (2–3 boutiques)             │
  │                                                  │
  │   [ + Add appointment ]  ← opens calendar        │
  │     event creator inline. Event saves to         │
  │     calendar AND links back to this task.        │
  └──────────────────────────────────────────────────┘

  ┌─ Subtask 4 ──────────────────────────────────────┐
  │ ○  Set dress budget                              │
  │                                                  │
  │   [ + Set budget ]  ← opens the Attire budget   │
  │     line inline. Amount saves to Budget module   │
  │     AND shows here as a linked chip.             │
  └──────────────────────────────────────────────────┘

  ────────────────────────────────────────────────
  📎 Linked objects (live, auto-updating)

  🖼 Mood board  →  4 pins saved
  🏪 Vendors     →  Atelier Rose (shortlisted)
  📅 Calendar    →  Mar 14, 11am · Mar 21, 2pm
  💰 Budget      →  $3,500 envelope set

  ────────────────────────────────────────────────
  💬 Notes                            [/ command]

  "Love the A-line silhouette from pin 3.
   Ask Atelier Rose about alterations timeline."

  ────────────────────────────────────────────────
  🤖 AI  →  "Appointments usually book 4–6 weeks
             out. Want me to draft a boutique
             inquiry email?"        [ Yes, draft it ]
```

### 1.2 The "Nested Sheet" Workflow

A key frustration with standard planning apps is getting lost. You leave your to-do list to go to the budget tab, and forget what task you were working on.

V2 solves this with **Nested Sheets**.
When you tap an "Add to Budget" portal inside a task, you do _not_ navigate to a new page. Instead, a sleek "Budget Editor" panel simply slides up over your current task. You type in the number, hit save, the panel slides down, and you are exactly where you started—but your task is complete and your master budget is updated.

### 1.3 Task Delegation

Planning is a multiplayer game. Any task can be explicitly assigned to a Collaborator (e.g., assigning "Review vendor contracts" to the Planner/Coordinator). On the dashboard, users and coordinators can filter their view to see only tasks assigned strictly to them, ensuring clear accountability.

---

## 2. Technical Implementations & Schemas (For Developers)

The following defines how the frontend renders and handles these nested portals without massive prop-drilling.

### 2.1 The Portal Schemas (Subtask Types)

Based on the V2 Shared Contract, subtasks evaluate a `types` array to determine which interactive buttons to render inside the Task Card component.

- **`mood_board` Portal:**
  - _Component:_ Render `<VisionBoard sheetMode={true} />`
  - _Data flow:_ Any pins saved update `linkedObjects.inspirationPinIds`.
- **`vendor_browse` Portal:**
  - _Component:_ Render `<VendorDirectory sheetMode={true} />`
  - _Data flow:_ Hiring/saving a vendor updates `linkedObjects.vendorIds`.
- **`calendar_event` Portal:**
  - _Component:_ Render `<CalendarEventCreator inline={true} />`
  - _Data flow:_ Saving the event updates `linkedObjects.calendarEventIds`.
- **`budget_line` Portal:**
  - _Component:_ Render `<BudgetEnvelopeAllocation inline={true} />`
  - _Data flow:_ Updates `linkedObjects.budgetEnvelopeId`.

### 2.2 Event-Driven Architecture Examples

To avoid hard-coded spaghetti logic within the Task Sheet component, portals rely on an Event-Driven architecture.

**Example: Booking a Florist**

1. User clicks `[ + Browse vendors ]` on the "Schedule Consultation" subtask.
2. The global UI Context triggers the `<VendorDirectory sheetMode={true} />` overlay.
3. User taps "Contact" on a florist list item.
4. The Vendor component dispatches a standard `VENDOR_CONTACT_CREATED` event.
5. A global interceptor detects this event _and_ detects that a Task Sheet is currently active in context.
6. The interceptor quietly patches the active subtask's `linkedObjects.vendorIds` array with the newly generated `vendorId`.
7. The Vendor overlay closes, and the Task Sheet rerenders displaying the new linked vendor chip.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of building this complex portal system, we will monitor:

- **Portal Execution Rate:** % of subtasks completed via interacting with a portal button vs manual checkbox subtasks. (Proves value of interconnectedness).
- **Portal Drop-off Rate:** The percentage of times a nested sheet is opened but closed without saving data. (Highlights UI friction).
- **Time-to-Completion:** The average time taken to complete Action-Card/Portal tasks compared to manual tasks. (Proves efficiency gains).
