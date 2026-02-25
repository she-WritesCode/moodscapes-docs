# Spec: Task Detail (Frontend Agent)
**Scope: Task Detail Modal/Page**

> ⚠️ **DEPENDENCY:** This agent must implement types defined in `SHARED_CONTRACT.md`.
> 
> ⚠️ **MVP SCOPE:** Chat functionality is handled by global Guide (see `SPEC_NAVIGATION_GUIDE.md`). Task Detail does NOT include chat interface.

---

## 1. Component Architecture

### 1.1 `TaskDetail.tsx` (Modal/Page)
**Goal:** The workspace for completing a single task.
- **Props:**
  - `task`: Task
  - `profile`: UserProfile
  - `onClose`: () => void
  - `onCompleteTask`: (taskId: string) => void
  - `completedSubtasks`: Set<string>
  - `toggleSubtask`: (subtaskId: string) => void
  - `isTaskCompleted`: boolean
- **Layout:**
  - **Header:** 
    - Close button (X)
    - "Week X" label
    - **3-Dot Menu (Top Right):** iOS-style "More" ellipsis button
  - **Title Section:** Large Title + Description + Category pill.
  - **Motivational Message:** Displayed as subtle text or card.
  - **Subtasks Section:**
    - List of subtasks (exactly 3 per task) with checkboxes.
    - Progress indicator showing completed/total.
  - **Action Section:**
    - "Mark Complete" button (Primary, full width, disabled until ready).

---

## 2. Visual Requirements (aistudio-wedapp Match)

**Task Detail:**
- **Mode:** Full-screen view on mobile (matches App.tsx pattern from aistudio-wedapp).
- **Background:** `#F2F2F2` (Light Gray) matching Dashboard.
- **Card Sections:** White (`#FFFFFF`) cards with iOS-style shadows (`ios-md`).
- **Typography:**
  - Title: `text-2xl font-bold text-zinc-900`.
  - Description: `text-base text-zinc-600`.
  - Motivational Message: `text-sm text-zinc-500 italic`.
- **Subtasks:**
  - Large tap targets (min 44px).
  - Checked state: strikethrough + opacity fade.
  - Smooth transition (200ms).
- **Buttons:**
  - Complete: `bg-zinc-900 text-white rounded-full py-3 font-bold shadow-lg`.
  - **3-Dot Menu:** iOS-style pull-down menu
    - Ellipsis icon ("⋯" or SF Symbol equivalent)
    - Button opacity: 50% when menu is open
    - Menu items:
      - **Snooze:** Normal text, no special styling
      - **Delete:** Red text (`text-red-600`), faint line separator above

---

## 3. Acceptance Criteria

### 3.1 Functionality
- [ ] Clicking subtask checkbox toggles state.
- [ ] Subtask state persists in parent component (`completedSubtasks` Set).
- [ ] "Mark Complete" button is enabled when task is ready.
- [ ] "Mark Complete" triggers:
  - Task completion (adds to `completedTaskIds`).
  - "+100 Points" added to user's total.
  - Celebration animation (confetti or toast).
  - Navigation back to Dashboard.
- [ ] 3-dot menu displays correctly (iOS pull-down style)
- [ ] Menu button opacity lowers to 50% when menu is open
- [ ] Snooze action moves task to "Next Week" (sets `snoozed_until`)
- [ ] Delete action triggers confirmation dialogue:
  - Title: "Delete Task?"
  - Message: "This task will be archived. You can restore it later."
  - Actions: "Cancel" (default), "Delete" (destructive/red)
- [ ] Delete action sets `deleted = true` in database (soft delete)
- [ ] Close button navigates back to Dashboard.

### 3.2 Visuals
- [ ] Matches `aistudio-wedapp` aesthetic (verified from actual code).
- [ ] Completion triggers visual celebration.
- [ ] All interactive elements have clear hover/active states.
