# Testing Requirements (MVP)

**Scope:** Full end-to-end testing of the Moodscapes Wedding Planning App (Phases 1-5).
**Audience:** QA, Developers, AI Agents.

---

## 1. Core Flow (Onboarding)
**Goal:** Verify user can successfully create a profile.

- [ ] **Step 1: Date Selection**
  - Select a future date → "Continue" enables.
  - Select a past date → Error message or disabled.
  - Verify progress bar shows 25%.
- [ ] **Step 2: Budget**
  - Select a budget tier (e.g., "$30k - $50k").
  - Verify single-select behavior (clicking another deselects previous).
  - "Continue" enables only after selection.
- [ ] **Step 3: Priorities**
  - Select 1-3 priorities.
  - Verify max 3 selection (4th click ignored or replaces oldest).
  - Verify pills change visual state (zinc-900 bg).
- [ ] **Step 4: Names**
  - Enter valid names (e.g., "Sarah & Alex").
  - Verify "Let's Plan" button enables.
  - **Submit:** Redirects to Dashboard.

## 2. Dashboard & Task Management
**Goal:** Verify task interaction and state management.

- [ ] **Hero Section**
  - Verify names display correctly ("Sarah & Alex").
  - Verify countdown days are accurate.
  - Verify streak counter shows 0 (initially).
- [ ] **Task List**
  - Verify 4 tasks are displayed.
  - Verify "Week 1" theme is visible.
- [ ] **Task Detail Modal**
  - Click a task → Modal opens.
  - Verify Title, Description, Emoji match card.
  - **Subtasks:**
    - Click checkbox → Visual update (strikethrough).
    - Progress bar updates (1/3 → 2/3).
  - **Completion:**
    - Complete all 3 subtasks → "Mark Complete" button enables.
    - Click "Mark Complete" → Confetti animation 🎉.
    - Modal closes automatically.
    - Dashboard card updates to "Completed" state (grayed out/checked).
  - **Snooze:**
    - Click "Snooze" → Modal closes.
    - Task remains on dashboard (future: visual indicator).
    - Re-opening task shows "😴 Snoozed" button state.

## 3. AI Integration (Gemini)
**Goal:** Verify task generation logic (Mock vs Real).

- [ ] **Mock Mode (Default)**
  - Verify fallback to `MOCK_WEEKLY_PLAN` if no API key.
  - Tasks should load instantly.
- [ ] **Real AI Mode (with API Key)**
  - Verify `generateWeeklyPlan` is called on onboarding complete.
  - Verify tasks match user priorities (e.g., "Photos" priority → Photography task).
  - Verify JSON schema validation (no missing fields).
  - Verify graceful fallback if API fails.

## 4. Gamification (Trophies & Points)
**Goal:** Verify rewards system.

- [ ] **Points Tracking**
  - Verify initial points = 0.
  - Complete a task → Points increase by 100.
  - Verify points update in Navigation bar.
- [ ] **Trophy Case**
  - Navigate to "Trophies" tab.
  - Verify Badge Grid displays.
  - **Unlock Logic:**
    - "Rookie Planner" (100 pts) unlocks after 1 task.
    - Locked badges show grayscale/opacity.
  - Verify "Prototype Lab" is locked (<1000 pts).

## 5. Polish & Navigation
**Goal:** Verify app structure and auxiliary pages.

- [ ] **Navigation**
  - Switch tabs: Plan ↔ Trophies ↔ Guide.
  - Verify active state styling on bottom nav.
- [ ] **Guide (Chat)**
  - Navigate to "Guide" tab.
  - Send a message → Message appears in chat bubble.
  - Receive placeholder response (or AI response if wired).
  - Verify suggestion chips work.
- [ ] **Settings**
  - Click Profile/Settings icon (if accessible) or navigate via URL.
  - Edit Names/Date → Save → Verify updates reflect on Dashboard.
  - **Reset Plan:** Click Reset → Confirm → Redirect to Onboarding.

## 6. Technical & Accessibility
**Goal:** Ensure code quality and compliance.

- [ ] **Mobile Responsiveness**
  - Test on 375px width (iPhone SE).
  - No horizontal scrolling.
  - Tap targets accessible (>44px).
- [ ] **Accessibility (a11y)**
  - Tab navigation works through onboarding.
  - Screen reader announces task state (checked/unchecked).
  - Color contrast meets WCAG AA.
- [ ] **State Persistence**
  - Refresh page → User session/profile persists (if Supabase wired).
  - *Note: Current MVP uses local state/mock, persistence is a known future item.*

---

**Success Criteria:**
- All "Happy Path" tests pass.
- No critical console errors.
- UI matches "Balanced Joy" aesthetic (clean, zinc-900, minimal).
