# Spec: Settings & Profile (Frontend Agent)
**Scope: Profile Management + App Settings**

> ⚠️ **DEPENDENCY:** This agent must implement types defined in `SHARED_CONTRACT.md`.

---

## 1. Component Architecture

### 1.1 `Settings.tsx` (Page)
**Goal:** Allow users to update their plan inputs.
- **Layout:**
  - Header: "Settings".
  - Section 1: **Wedding Details**
    - Names (Editable Text).
    - Date (Date Picker).
    - Location (Text).
  - Section 2: **Plan Preferences**
    - Budget Tier (Dropdown/Pill).
    - Priorities (Multi-select Pills).
  - Section 3: **Account**
    - "Reset Plan" (Destructive Action).
    - "Sign Out" (Button).

---

## 2. Visual Requirements

- **Style:** Consistent with Dashboard (Clean, white cards).
- **Destructive Actions:** Red text/border for "Reset Plan".
- **Save Behavior:** Auto-save on blur OR explicit "Save Changes" button (MVP: Save button is safer).

---

## 3. Acceptance Criteria

### 3.1 Functionality
- [ ] Changing Date updates the countdown on Dashboard.
- [ ] Changing Budget updates the Budget Pill on Dashboard.
- [ ] "Reset Plan" clears all tasks and history (requires confirmation).
- [ ] Sign Out redirects to Onboarding/Login.

### 3.2 Integration
- [ ] Updates `profiles` table in Supabase.
- [ ] Triggers re-generation of *future* tasks if priorities change (Nice to have, not MVP).
