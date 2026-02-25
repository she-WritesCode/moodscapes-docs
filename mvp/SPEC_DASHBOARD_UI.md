# Spec: Dashboard UI (Frontend Agent)
**Scope: Hero Section + Task List + Locked Preview**

> ⚠️ **DEPENDENCY:** This agent must implement types defined in `SHARED_CONTRACT.md`.

---

## 1. Component Architecture

### 1.1 `HeroSection.tsx`
**Goal:** Show high-level progress and context.
- **Props:**
  - `profile`: UserProfile
  - `completedCount`: number
  - `totalCount`: number
- **Visuals:**
  - Background: White card with soft shadow.
  - Content:
    - "Days to go" countdown.
    - **NEW:** Budget Context Pill.
      - Text: "$40-60k Budget • ~$440/guest" (Calculated from profile).
      - Style: Subtle gray badge, small text.

### 1.2 `Dashboard.tsx` (Main View)
**Goal:** Display current week's tasks and preview next week.
- **Layout:**
  - Header (Logo + Streak Counter).
  - Hero Section.
  - "This Week" Label.
  - Task List (Active).
  - "Coming Soon" Label.
  - Locked Preview (Upcoming).

### 1.3 `TaskCard.tsx` (Active)
- **Props:** `task`: Task
- **Visuals:**
  - White card, rounded corners.
  - Left: Checkbox (circular).
  - Center: Title + Description (truncated).
  - Right: Chevron.
  - **Tag:** "Vendor" or "Creative" pill if applicable.

### 1.4 `UpcomingPreview.tsx` (Locked)
**Goal:** Reduce anxiety by showing *structure* without *details*.
- **Visuals:**
  - Grayed out / Opacity 60%.
  - Lock icon overlay.
- **Content:**
  - Single message card: "🔒 Next week's plan unlocks Sunday"
  - Subtext: "Your tasks will be personalized based on this week's progress"

### 1.5 Navigation
**Bottom Navigation:**
- **Dashboard (Home):** Current week tasks (this view)
- **Plan:** Comprehensive task management (replaces Trophy Case)
  - See `SPEC_PLAN_TAB.md` for full specification
- **Guide:** AI chat assistance

**Note:** Trophy Case UI still exists but is **deprioritized for MVP**. Complex badge logic is Post-MVP.

---

## 2. Visual Requirements (aistudio-wedapp Match)

**Colors (Tailwind):**
- Background: `#F2F2F2` (Light Gray)
- Cards: `white`
- Text Primary: `zinc-900`
- Text Secondary: `zinc-500`
- Accent: `rose-400` (Buttons/Highlights)
- Success: `green-400` (Completion)

**Typography:**
- Font: Inter (or system sans).
- Headings: Bold, tight tracking.

**Spacing:**
- Mobile-first.
- `max-w-md mx-auto` (Center column).
- `p-4` or `p-6` padding.

---

## 3. Acceptance Criteria

### 3.1 Functionality
- [ ] Hero shows correct budget/guest math.
- [ ] Clicking active task opens detail view.
- [ ] Clicking locked task does nothing (or shows tooltip).
- [ ] "Coming Soon" section is visible but disabled.

### 3.2 Visuals
- [ ] Matches `aistudio-wedapp` aesthetic (clean, white cards on gray bg).
- [ ] No horizontal scrolling on mobile.
- [ ] Budget pill is visible but not distracting.
