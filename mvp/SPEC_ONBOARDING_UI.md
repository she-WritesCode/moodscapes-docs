# Spec: Onboarding & UI (Frontend Agent)
**Scope: 4-Step Onboarding Flow + Shared Components**

> ⚠️ **DEPENDENCY:** This agent must implement types defined in `SHARED_CONTRACT.md`.

---

## 1. Component Architecture

### 1.1 Shared Components
*Must be reusable across the app.*

#### `PillSelector.tsx`
- **Props:**
  - `options`: Array of `{ id: string, label: string }`
  - `selected`: string | string[]
  - `multiSelect`: boolean
  - `onChange`: (value: string | string[]) => void
- **Visuals:**
  - Large tap targets (min 44px height)
  - Selected state: Primary color background, white text
  - Unselected state: Gray-100 background, black text
  - Hover state: Gray-200 background
- **Accessibility:**
  - `role="button"` or `role="checkbox/radio"`
  - Keyboard navigable (Tab + Space/Enter)

#### `ProgressBar.tsx`
- **Props:**
  - `currentStep`: number (1-4)
  - `totalSteps`: number (4)
- **Visuals:**
  - Thin line with filled portion
  - Smooth transition animation

### 1.2 Onboarding Pages (Wizard Flow)

#### `Step1Date.tsx`
- **Input:** Date Picker
- **Copy:** "When is the big day?"
- **Feature:** "Not decided yet" checkbox (sets date to null)
- **Validation:** Required (either date selected OR checkbox checked)

#### `Step2Budget.tsx`
- **Input:** `PillSelector` (Single Select)
- **Options:** Import `BUDGET_OPTIONS` from `SHARED_CONTRACT`
- **Copy:** "What's your rough budget?"
- **Subcopy:** "It's okay if you're still figuring it out."

#### `Step3Priorities.tsx`
- **Input:** `PillSelector` (Multi Select)
- **Options:** Import `PRIORITIES` from `SHARED_CONTRACT`
- **Copy:** "What matters most to you?"
- **Subcopy:** "Pick up to 3."
- **Validation:** Max 3 selections

#### `Step4Names.tsx`
- **Input:** Two separate text fields (PR #75 Update)
  - `userName` - First person's name
  - `partnerName` - Second person's name (optional)
- **Copy:** "Who is getting married?"
- **Placeholders:** 
  - userName: "Your name"
  - partnerName: "Partner's name (optional)"
- **Button:** "Create My Plan" (Triggers AI task generation)
- **Behavior:** 
  - Can handle single name (solo event) or both names
  - Names stored separately in profile, formatted as "User & Partner" for display
  - More robust than ampersand-splitting pattern

**Implementation Note (PR #75):** Changed from single text field to separate fields to handle edge cases where users don't follow "Name & Name" format.

---

## 2. Visual Requirements (Lovable/Tailwind)

- **Typography:** Sans-serif, modern (Inter or similar)
- **Spacing:** Generous whitespace (p-6 or p-8 container)
- **Colors:**
  - Primary: Brand Color (e.g., Soft Sage or Warm Terracotta)
  - Background: White or very light gray
  - Text: Gray-900 (headings), Gray-600 (subtext)
- **Mobile:**
  - Stacked layout
  - Full-width buttons
  - No horizontal scrolling

---

## 3. Acceptance Criteria (Test Scenarios)

### 3.1 Functionality
- [x] User can navigate Next/Back ✅ **Complete**
- [x] Data persists between steps (state management) ✅ **Complete**
- [x] "Not decided yet" clears the date picker ✅ **Complete**
- [x] Multi-select limits to 3 items ✅ **Complete**

### 3.2 Accessibility
- [x] All inputs have labels ✅ **Complete**
- [x] Focus states are visible ✅ **Complete**
- [x] Color contrast passes WCAG AA ✅ **Complete**
- [ ] Screen reader announces step changes ⚠️ **To be tested**

### 3.3 Integration (PR #75 Status)
- [x] Output matches `UserProfile` interface from `SHARED_CONTRACT` ✅ **Complete**
- [x] Final submit generates AI tasks via `taskService.generatePersonalizedTasks()` ✅ **Working**
- [ ] Profile saved via `authService.saveProfile()` ⚠️ **Service exists, not called**
