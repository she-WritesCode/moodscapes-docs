# Spec: Guide (AI Chat)

**Scope: Real-time AI wedding planning assistant**

---

## 1. Overview

The Guide feature provides users with a conversational interface to ask questions about their wedding planning, get advice, and receive context-aware suggestions based on their progress.

**Design Philosophy:** "Calm & Empathetic" - The assistant should be a supportive partner, not just a search engine.

---

## 2. Navigation

**Location:** Bottom navigation bar
**Icon:** `Sparkles` or `MessageCircle`
**Tab Label:** "Guide"

---

## 3. UI Components

### 3.1 Chat Interface

- **Headers:** "Guide" title with subtext "Your AI wedding planning assistant".
- **Message Bubbles:**
  - **User Messages:** Zinc-900 background, white text, aligned right.
  - **Assistant Messages:** White background, zinc-900 text, zinc-200 border, aligned left. Supports Markdown rendering.
- **Loading State:** Bubbling dots animation when AI is "typing".
- **Empty State / Welcome:**
  - Shows a personalized welcome message if no history exists.
  - Welcome message includes the user's name and primary wedding priorities.
  - **Crucial:** Welcome message should only be generated and saved ONCE per user.

### 3.2 Suggestion Chips

- Appears at the bottom (above input) when the conversation is empty.
- Provides common questions like "How do I book a photographer?" or "What's a good budget breakdown?".

### 3.3 Input Area

- Fixed to the bottom.
- Rounded pill-shaped text input.
- "Send" button (disabled when empty or loading).

---

## 4. Initialization Logic

- On mount, load chat history from `chat_messages` table.
- If history is empty:
  - Generate a greeting: `Hi [Name]! I'm Moodscapes... I see your top priorities are [Priorities]. How can I help you today?`
  - Save this greeting to the DB immediately to prevent duplication in future sessions.
- Use a `ref` or similar mechanism to prevent `StrictMode` from firing duplicate saves during development.

---

## 5. AI Context (Edge Function)

The `chat-guide` Edge Function receives:

1. Current message.
2. Last 15 messages of history.
3. User Profile (names, date, budget, priorities, location).
4. Current Task List (completed vs. pending).

**Persona Rules:**

- Calm, empathetic, and expert.
- Concise responses (max 3 paragraphs).
- Context-aware: Refs the user's actual tasks when appropriate.

---

## 6. Technical Notes

- **Table:** `chat_messages` (id, user_id, role, text, created_at).
- **Service:** `guideService.ts`.
- **Cache:** React Query for history (`chat-history`).
