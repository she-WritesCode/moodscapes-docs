# Moodscapes V2 UI/UX Architecture

## Foundation

The visual design language continues the Moodscapes brand:

- Soft, human-centric layouts with "vowcraft" serif italics used sparingly for branding.
- Colors: Romantic, warm tones, minimal hard borders (glassmorphism/blur effects).
- **Core interaction principle:** Notion-like blocks everywhere. Text fields support `/` commands, enabling rich interconnection.

## 1. Global Navigation

The `/v2` route prioritizes a streamlined **5-Tab Bottom Bar**, designed to focus on the active planning modes and financial peace of mind.

### 1. Home (Dashboard)

- **Hero:** Wedding countdown in large, elegant serif font, displaying couple's names.
- **Immediate Focus:** Horizontal scroll of "Today's Focus" tasks drawn from the current week.
- **Dynamic Widgets (The Hub):** Modular cards acting as quick-jump points for other tools:
  - Next payment due (Budget)
  - Upcoming appointments (Calendar)
  - "Your Team" (Hired Vendors)
  - Guest List snapshot

### 2. Plan (Tasks, Notes & Guests)

- A unified workspace for execution and people management.
  - Tasks (The week-by-week OS)
  - Notes (The Notion-like workspace aggregating all rich text)
  - Guests (RSVPs and Seating)
- **Header Toggle:** Switch between `Tasks` (The week-by-week OS), `Notes` (The Notion-like workspace aggregating all rich_text documents), and `Guests` (RSVPs and Seating).
- **Interaction:**
  - Swipe Right: Mark done.
  - Swipe Left: Snooze / Assign.
  - Tap: Expands to full-sheet Task Detail View.

### 3. Vision (Board & Moods)

- A unified workspace for inspiration.
- **Header Toggle:** Switch between `Pins` (The Pinterest-style masonry grid of saved images) and `Ideas` (A filtered view of creative text-based notes).
- **Sticky Filters:** Horizontal pills (All, Ceremony, Florals, Dress, etc.).
- **Pin Detail Modal:** Slides up 80%. Includes a "find similar vendors" button and a Notion-style notes field at the bottom.

### 4. Budget

- A dedicated space for the couple's finances, providing peace of mind.
- **Hero:** Donut chart of budget vs. spent.
- Scrollable list of category envelopes. Expanding an envelope reveals itemized payments and linked vendors.

### 5. More / Settings

- The catch-all for less frequent, high-level tools.
- Entry-points for **Vendor Directory** and **Full Calendar**.
- Partner access and global app settings.
- **PWA Installation:** A highly prominent "Install App" button. Because Moodscapes prioritizes a native-app feel on the web, users should be encouraged globally to install the PWA to their home screen for an immersive, full-screen, notification-enabled experience.

## 2. The Contextual Sheet Architecture

The V2 OS prioritizes **Bottom Sheets over Page Pushes** visually to preserve context. However, to ensure the web app functions like a native mobile app, **URL state must still drive these sheets.**

### 2.1 URL Routing Strategy (Handling the Back Button)

If a user presses the browser "Back" button, it must behave expectedly (closing the active sheet rather than navigating away from the page).

To achieve this, bottom sheets are driven by URL changes, typically via **Query Parameters** or **Nested Routes**:

- **Base Route:** `/v2/tasks` (Shows the task list)
- **Task Sheet Open:** `/v2/tasks?taskId=123` (Task list remains in background, Task Sheet slides up)
- **Nested Portal Open:** `/v2/tasks?taskId=123&portal=vendor_browse` (Vendor sheet slides up over the Task Sheet)

When the user clicks the browser Back button, the URL pops back to `/v2/tasks?taskId=123`, which naturally unmounts the `vendor_browse` sheet while keeping the Task sheet open. This provides true native behavior on the web.

### Task Detail Sheet (The OS Hub)

When a user taps a Task, a full-screen sheet slides up.

- **Top:** Task Title, Category Tag, and Due Window.
- **Body:** The subtasks. Remember, subtasks are interactive. If a subtask has `types: ['mood_board', 'calendar_event']`, two branded action buttons render under the subtask title:
  - `[ + Open Mood Board ]`
  - `[ + Add to Calendar ]`
- **Bottom:** Linked Objects Section. As the user completes subtasks using the portals above, the linked objects (vendors booked, events created) populate this sticky block.

### 2.3 Nested Portals (Sheets on Sheets)

If the user clicks `[ + Browse vendors ]` from within the Task Sheet, a _second_ sheet slides up, rendering the `<VendorDirectory>` component.
Once the user "contacts" a vendor, the `VendorDirectory` sheet closes, and the user is dropped seamlessly back into their Task Sheet, with the vendor now attached and the subtask checked off.

#### 2.4 State Management Strategy for Sheets

Because the "Nested Sheet" approach relies heavily on inter-component communication (a Vendor Directory sheet needs to tell the underlying Task Sheet that a vendor was hired), relying on standard React `useState` and prop-drilling will quickly become unmaintainable.

To make this complex native-app feel viable on the web, the architecture **requires a robust Global State Management tool** (like Zustand or Jotai).

- Global State handles the visibility array of which sheets are currently active.
- Global State acts as the Event Bus, allowing deeply nested portal sheets to dispatch update payloads that the underlying parent sheets subscribe to.

## 3. Persistent AI Assistant

To avoid the "Dual-FAB Problem", the AI interaction point adjusts contextually based on the screen's primary purpose:

- **Context-Aware UI Placement:**
  - **Top App Bar:** On screens that have their own primary action (like the "Tasks" list which needs an "Add Task" FAB in the bottom right), the AI is rendered as a glowing, rose-gold sparkle icon in the **Global Top App Bar**.
  - **Primary FAB (Bottom Right):** On screens that have NO primary creation action (like the Home/Dashboard or Budget overview), the AI promotes itself to inhabit the bottom-right FAB space entirely.
- **State Awareness:** If opened inside a Task Sheet, the chat UI pushes a header indicating context: _"Chatting about: Book a Florist"_.
- **Interactive Output:** The AI does not just output text. Messages can contain embedded action cards:
  - Task cards (Tap to add to your list).
  - Vendor cards (Tap to open profile).
  - Calendar event cards.
