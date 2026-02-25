# V2 AI Integration Specification

## Overview

In the original version of our app, the AI acted like a simple list-maker: it generated plain text checklists. For V2, the AI is promoted to a **Proactive Wedding Planner**.

Instead of just telling you _what_ to do, the AI now hands you the exact _tools_ you need to do it. It acts as the intelligent glue connecting all the different modules (Budgets, Vendors, Guests, Calendar) into a single, cohesive "Wedding OS."

---

## 1. The Proactive "Wedding OS" Guide (For Stakeholders)

### 1.1 Generating Interactive Tasks

The AI still generates a personalized weekly plan of 4 overarching tasks to prevent overwhelm. However, the subtasks inside these plans are no longer just checkboxes—they are **Interactive Portals**.

When the AI generates a task like "Book the Photographer," it automatically attaches the tools you need:

- A portal to your **Mood Board** (to find photography styles you like).
- A portal to the **Vendor Directory** (filtered to photographers in your area).
- A portal to the **Budget Module** (pre-populated with your photography budget limit).

### 1.2 Context-Aware Conversations

In V2, when a user taps the "Ask AI" chat bubble, the AI immediately knows _where_ the user is and _what_ they are doing. It stops giving generic wedding advice and starts giving highly specific, actionable help.

- **If on the Home page:** The AI acts as a high-level guide ("You're 6 months out, let's focus on catering this week!").
- **If inside a specific Task (e.g., "Find a Florist"):** The AI reads the current task and helps execute it ("I see you're looking for a florist. Should I search the directory for options under $3,000?").
- **If looking at the Budget:** The AI acts as a financial advisor ("You're over budget on the venue. Let's look at ways to save on decorations.")

### 1.3 Action Cards: UI Inside the Chat

The biggest leap in V2 is that the AI doesn't just reply with text. It can reply with **Interactive UI Panels** directly in the chat stream!

If a user asks the AI: _"Can you remind me to pay the DJ next Friday?"_
Instead of saying: _"Sure, go to your calendar and add it."_
The AI says: _"Here is the calendar event for next Friday. Tap save to confirm!"_ and embeds a real, tappable **Calendar Event Form** right there in the chat.

---

## 2. Technical Implementations & Schemas (For Developers)

The following sections define the underlying data contracts required to power the narrative features described above.

### 2.1 Prompt Context Enhancements

To allow the AI to generate relevant tasks that link properly to the new modules, the AI context payload sent to the LLM must include the wedding's _live state_.

```typescript
interface GenerationContext {
  profile: UserProfile;
  current_week: number;
  completed_task_ids: string[];

  // V2 Additions:
  hired_vendor_categories: string[]; // e.g. ["florist", "photographer"]
  set_budget_categories: string[]; // e.g. ["dress", "venue"]
}
```

### 2.2 Updating the LLM Task Generation Prompt

The system instructions for Gemini must be explicitly updated to handle the new `types` array on subtasks.

> **System Prompt Fragment (To be Added):**  
> "You are generating tasks. Subtasks are interactive portals. For each subtask, evaluate the required tool and assign the appropriate `types`: `mood_board`, `vendor_browse`, `calendar_event`, `budget_line`, or `checkbox`. A subtask CAN have multiple portals (e.g., `['mood_board', 'vendor_browse']`)."

**Expected LLM Output (JSON):**

```json
{
  "weekNumber": 3,
  "tasks": [
    {
      "title": "Book the Caterer",
      "subtasks": [
        { "text": "Set catering budget", "types": ["budget_line"] },
        { "text": "Schedule 3 tasting appointments", "types": ["vendor_browse", "calendar_event"] }
      ]
    }
  ]
}
```

### 2.3 Contextual Chat State

When opening the AI chat bubble from within a specific module, the frontend must pass contextual prop data to the Chat Route:

```typescript
interface ChatContext {
  activeLocation: "global" | "task" | "vendor" | "budget";
  activeLocationId?: string; // e.g., the specific taskId
  subtaskCompletionState?: object;
  relevantBudgetEnvelope?: object;
}
```

The backend uses this `activeLocation` to prepend the correct system directive (e.g., "The user is currently executing Task X. Do not give generic advice.") before forwarding the user's message to the LLM.

### 2.4 Action Cards Payload (Interactive Portals)

To make conversations strictly actionable, the AI is instructed to return `action_cards` arrays.

> **System Prompt Instruction:**  
> "If the user needs to schedule a meeting, find a vendor, or log an expense, DO NOT just tell them how. Append an `action_cards` JSON array specifying the UI portal to render in the chat."

**Example backend payload returned to the client:**

```json
{
  "role": "assistant",
  "content": "I'd love to help you book that Cake Tasting! I've prepared a calendar event for you.",
  "action_cards": [
    {
      "type": "calendar_event_form",
      "prefill_data": {
        "title": "Cake Tasting with Sweet Treats Bakery",
        "suggested_duration": 60
      }
    }
  ]
}
```

The Frontend is responsible for mapping the `"type": "calendar_event_form"` string to the actual React/Vue `<CalendarEventForm />` component, spreading `prefill_data` as default props.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the context-aware Prompts and Action Cards, we will monitor:

- **Contextual vs Global Chat Ratio:** Comparing how often chat is invoked from _inside_ a module (e.g. while in a task) vs from the global home screen.
- **Action Card Execution Rate:** The percentage of Action Cards generated by the AI where the user actually clicks the embedded UI button (e.g., tapped 'Save' on the AI-generated calendar event).
- **Task Generation Acceptance Rate:** Do users actually complete the generated Task Portals, or do they delete/ignore them?
