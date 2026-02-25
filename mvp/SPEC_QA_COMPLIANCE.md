# Spec: QA & Compliance (Quality Agent)
**Scope: Test Scenarios + Accessibility + Brand Voice Verification**

> ⚠️ **DEPENDENCY:** This agent validates against `SHARED_CONTRACT.md`.

---

## 1. Test Scenarios (The "Gauntlet")

### 1.1 Onboarding Flow
- [ ] **Happy Path:** Select Date -> Select Budget -> Select Priorities -> Enter Names -> Submit.
- [ ] **Skip Path:** Skip Date (Not decided) -> Skip Budget -> Skip Priorities -> Enter Names -> Submit.
- [ ] **Edge Case:** Select 3 priorities, try to select 4th (should fail/block).
- [ ] **Edge Case:** Enter very long names (should truncate or wrap gracefully).

### 1.2 Task Generation (AI)
- [ ] **Constraint Check:** Verify exactly 4 tasks generated.
- [ ] **Constraint Check:** Verify exactly 3 subtasks per task.
- [ ] **Logic Check:** If "Photos" priority selected, is Photographer in top 2 suggestions?
- [ ] **Logic Check:** If "Budget" priority selected, is there a budget-related task?
- [ ] **History Check:** Verify no tasks from `completed_task_ids` are repeated.

### 1.3 Vendor Prioritization
- [ ] **Timeline Check:** If wedding is 18 months out, Venue should be #1.
- [ ] **Timeline Check:** If wedding is 6 months out, Dress/Attire should be high priority.
- [ ] **Dependency Check:** If Venue NOT booked, Caterer should NOT be suggested (or warned).

---

## 2. Accessibility Checklist (WCAG AA)

- [ ] **Color Contrast:** Text vs Background ratio > 4.5:1.
- [ ] **Tap Targets:** All buttons/pills > 44x44px.
- [ ] **Keyboard Nav:** Can complete onboarding using only Tab/Space/Enter.
- [ ] **Screen Readers:** All inputs have `aria-label` or associated `<label>`.
- [ ] **Focus States:** Visible focus ring on all interactive elements.

---

## 3. Brand Voice Rubric ("Balanced Joy")

**Review 5 random generated tasks against this rubric:**

| Criteria | Pass | Fail |
|----------|------|------|
| **Tone** | Warm, encouraging ("You got this!") | Bossy, clinical ("Do this now.") |
| **Inclusion** | Uses "You both", "Discuss together" | Uses "You", "Your day" (singular) |
| **Framing** | Focuses on guest experience/fun | Focuses on perfection/stress |
| **Brevity** | Title < 5 words, Desc < 15 words | Long paragraphs |
| **Pressure** | No "Urgent!", "ASAP!", "Critical!" | Artificial urgency |

---

## 4. Error Handling & Edge Cases

### 4.1 Onboarding Errors
- [ ] **Past Date:** User selects wedding date in the past → Red border + error message "Please pick a date in the future" + Continue button disabled.
- [ ] **No Priority:** User doesn't select any priority → Continue button stays disabled.
- [ ] **Long Names:** User enters names >50 characters → Text truncates with ellipsis OR wraps gracefully.
- [ ] **Special Characters:** User enters emoji/special chars in names → Accepts gracefully (no crashes).

### 4.2 API & Network Errors
- [ ] **Offline Onboarding:** User completes onboarding while offline → Profile saves to localStorage, loads when online.
- [ ] **AI Generation Fails:** Gemini API returns error → Toast: "Couldn't generate tasks. Here are some suggestions." + Load fallback tasks.
- [ ] **Timeout:** API takes >10 seconds → Show timeout error + retry button.
- [ ] **Malformed JSON:** AI returns invalid JSON → Catch parse error + load fallback tasks.
- [ ] **Network Lost Mid-Task:** User loses connection while marking task complete → Save locally, sync when reconnected.

### 4.3 Loading States
- [ ] **Onboarding Submit:** "Start Planning" button shows spinner while saving.
- [ ] **Dashboard First Load:** Shows 4 skeleton Task Cards while fetching.
- [ ] **AI Generation:** Full-screen overlay "Creating your next tasks..." (max 10s).
- [ ] **Settings Save:** Toast notification "Saving..." → "Saved!" when complete.

### 4.4 Edge Cases
- [ ] **Empty State:** User completes all tasks → Shows celebratory message + "Week 2 unlocks Sunday".
- [ ] **All Tasks Snoozed:** User snoozes all 4 tasks → Show empty state "All tasks snoozed. Check back next week!"
- [ ] **Rapid Clicking:** User clicks "Mark Complete" multiple times → Disable button after first click (debounce).
- [ ] **Browser Back:** User presses back button in Task Detail → Navigates to Dashboard (state preserved).

---

## 5. MVP "Go/No-Go" Checklist

1. [ ] User can sign up and complete onboarding.
2. [ ] User receives 4 personalized tasks.
3. [ ] User can mark a task as complete.
4. [ ] User can snooze a task.
5. [ ] Week 2 preview is visible (locked).
6. [ ] Mobile view looks broken? (No horizontal scrolls).
7. [ ] No console errors during standard flow.
