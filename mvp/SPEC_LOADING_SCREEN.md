# Spec: Loading Screen

**Scope:** Post-onboarding loading state while AI generates personalized plan

---

## 1. Overview

The loading screen appears after the user completes Step 4 (Names) and triggers AI task generation. It provides visual feedback and builds anticipation while the plan is being created.

---

## 2. Visual Design

### 2.1 Layout

- **Centered content** on screen
- **Background:** Soft gradient (`from-blue-50 via-purple-50 to-pink-50`) or solid light (`#F2F2F2`)
- **Max width:** `max-w-md` (consistent with other screens)

### 2.2 Animated Element

- **Visual:** Pulsing ✨ emoji (sparkles)
- **Size:** Large (`text-6xl`)
- **Animation:** Gentle pulse (scale 1 → 1.1), CSS `animate-pulse`
- **No:** spinning rings, confetti, bouncing hearts

### 2.3 Messages

- **Count:** 3-4 rotating messages
- **Transition:** Fade in/out, 2-second intervals
- **Style:** `text-lg font-medium text-zinc-700`

**Message Copy:**

1. "Building your personalized plan..."
2. "Tailoring tasks to your priorities..."
3. "Adding the finishing touches..."
4. "Almost there..."

### 2.4 What NOT to Include

- No tips or "Did you know?" facts
- No progress bar (duration is unpredictable)
- No complex animations (confetti, multiple elements)

---

## 3. Personalization

If the user's name is available, display:
> "{Name}, your plan is almost ready!"

Otherwise:
> "Your plan is almost ready!"

---

## 4. Acceptance Criteria

- [ ] Pulsing ✨ emoji centered on screen
- [ ] 3-4 messages rotate with fade transition
- [ ] No confetti or complex animations
- [ ] Personalized greeting if name available
- [ ] Mobile-friendly layout

---

## 5. Technical Notes

**Component:** `LoadingScreen.tsx`

**Trigger:** Called from `OnboardingFlow.tsx` after Step 4 submission

**Duration:** Displays until task generation API completes
