# Spec: Plan Tab

**Scope:** Comprehensive task management view (Replaces Trophy Case)

---

## 1. Overview

The Plan Tab provides users with a centralized view of all their tasks, organized by status and timeline. This view gives users control over their wedding planning without overwhelming them.

**Design Philosophy:** "One week at a time" - Focus on actionable next steps while making future tasks visible but non-intrusive.

---

## 2. Navigation

**Location:** Bottom navigation bar (replaces Trophy Case icon)

**Icon:** TBD (suggest: 📋 or similar planning icon)

**Tab Label:** "Plan"

---

## 3. Sticky Pill Filters

Filters appear at the top of the view, always visible while scrolling.

### 3.1 Up Next (Default)

**Content:**

- **This Week Section**
  - Current week's AI-generated tasks (week 1 tasks)
  - 4 Most urgent and actionable items
  - Includes any snoozed tasks that are due this week
  - Expandable to show all subtasks
  
- **Next Week Section**
  - 4 AI-generated tasks for the upcoming week (week 2 tasks)
  - Includes any snoozed tasks for next week
  - Expandable to show all subtasks
  
- **Later Section (Collapsed by Default)**
  - Future tasks beyond next week (week 3+ tasks)
  - **Collapsed State**: Shows only section header and "Tap to expand"
  - **Expanded State**: Contains a **non-actionable message card** followed by future tasks:

    ```text
    Headline: One week at a time. 💎
    Body: Your plan updates automatically as you make progress 
          and your wedding approaches.
    ```

  - **Rationale:** Prevents decision overload and keeps the initial view clean.

**Order Rationale:** This Week appears first because it contains the most urgent, actionable tasks that need immediate attention.

### 3.2 Done

**Content:**

- All completed tasks
- Sorted by completion date (most recent first)
- Shows checkmark icon
- No subtasks expanded (collapsed by default)

**Future Enhancement (Post-MVP):**

- "Deleted" subsection (collapsed)
- Allows restoration of soft-deleted tasks

### 3.3 All (Last Position)

**Content:**

- Combined view showing both Up Next and Done tasks
- Displayed with clear section headers:
  - "NEXT WEEK (N)" - incomplete tasks
  - "DONE (N)" - completed tasks
- No "Later" section in All view (simplification for MVP)

**Rationale:** Quick full overview without overwhelming detail

---

## 4. Task Display

**Task Card Format:**

- Same design as Dashboard cards
- Icon + icon background color
- Title + Description
- Progress indicator (X/Y steps)
- Completion checkbox

**Interactions:**

- Tap card → Navigate to Task Detail view
- Tap checkbox → Mark complete (with animation)

---

## 5. Snooze Behavior

**Important:** There is **NO "Snoozed" filter** in this view.

**Logic:**

- When user snoozes a task, it is automatically added to "Next Week"
- Snoozed tasks appear alongside AI-generated tasks
- Visually identical (no special "snoozed" badge for MVP)
- **MVP Note:** Snoozing adds to Next Week without replacing an existing task (may temporarily exceed 4 tasks)

**Rationale:** Simplifies UI and reduces cognitive load

---

## 6. Delete Behavior

**Action:** Soft delete (sets `deleted = true` in database)

**UI Impact:**

- Task disappears from Plan Tab immediately
- No "Recently Deleted" view in MVP

**Fast Follow:** Add "Deleted" subsection under "Done" filter

---

## 7. Empty States

### Up Next (No Tasks)

```text
💍 All caught up!
Check back next week for new tasks.
```

### Done (No Completed Tasks)

```text
🎯 Complete your first task
Your wins will show up here.
```

### All (No Tasks)

```text
✨ Welcome to your plan
Complete onboarding to generate your personalized tasks.
```

---

## 8. Acceptance Criteria

- [ ] Plan Tab appears in bottom navigation
- [ ] Sticky pill filters work (Up Next, Done, All)
- [ ] "Up Next" shows This Week + Next Week + Later sections
- [ ] "This Week" appears first with most urgent tasks
- [ ] "Later" section is collapsed by default with message card
- [ ] "Done" shows completed tasks only
- [ ] "All" shows combined view with section headers
- [ ] Snoozed tasks appear in appropriate week section
- [ ] Task cards match Dashboard design
- [ ] Empty states display correctly
- [ ] Tapping task card navigates to Task Detail

---

## 9. Technical Notes

**Data Source:**

- Query `tasks` table with proper filters
- Week 1 tasks (`weekNumber = 1`): Show in "This Week"
- Week 2 tasks (`weekNumber = 2`): Show in "Next Week"
- Week 3+ tasks (`weekNumber >= 3`): Show in "Later"
- `snoozed_until` tasks: Include in appropriate week based on snooze date
- `deleted = true`: Exclude from all views (except future "Deleted" section)

**State Management:**

- Filter selection persists within session
- Default to "Up Next" on tab load
- Scroll position resets when changing filters

---

## 10. Future Enhancements (Post-MVP)

- [ ] "Deleted" subsection under "Done" filter
- [ ] Restore deleted tasks
- [ ] Bulk actions (select multiple tasks)
- [ ] Drag-to-reorder task priority
- [ ] Custom filters (by category, vendor type, etc.)
- [ ] Snooze replacement logic (bump a Next Week task to Later when snoozing)
