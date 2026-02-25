# V2 Notes & Docs Module Specification

## Overview

The Notes & Docs module acts as the "Brain" of the Moodscapes OS. While tasks dictate _what_ needs to be done, Notes capture _how_ the user feels, plans, and thinks.

---

## 1. The Planning Brain (For Stakeholders)

### 1.1 The Block Editor Experience

Simple text boxes aren't enough for planning a wedding. Moodscapes V2 introduces a rich, Notion-style block editor. Users can type `/` to instantly insert Checklists, Bullet points, Headers, or Dividers. It feels modern, clean, and powerful—perfect for drafting Wedding Vows, noting down interview questions for a photographer, or brainstorming speech ideas.

### 1.2 Contextual Notes (No More Junk Drawers)

A major problem with note apps is forgetting where you put things. Moodscapes solves this with Contextual Creation.

- If a user opens the "Write your vows" task, the rich-text editor opens _inside that exact task_. When they close it, the note is saved directly to that task for easy retrieval next week.
- Users can also create generic, high-level structural notes by going directly to the global Notes tab.

### 1.3 Auto-Aggregation & Folders

All notes—whether created globally or attached deeply inside a specific task—are automatically aggregated into the central **Notes Tab**. To prevent this view from becoming cluttered, the system automatically sorts them into Folders/Tags like `Vows & Speeches`, `Vendor Notes`, and `Logistics`.

---

## 2. Technical Implementations & Schemas (For Developers)

The Notes module relies on a JSON-based rich text schema to support modular blocks rather than raw HTML strings.

### 2.1 Entity Relationships

1. **`notes`**: The core document entity containing `title` and `category` ('vows', 'vendor_notes', 'brainstorming', 'logistics'). Tied to a `workspace_id`.
2. **`content` column**: This is stored as `jsonb` instead of `text` to support the block-editor library's output format (e.g., arrays of paragraph, list, and header nodes).

### 2.2 UI Tab Priority

Within the primary **Plan** tab navigation, the elements are rendered in strict priority order regarding visual hierarchy:

1. **Tasks** (Primary: The OS execution view)
2. **Notes** (Secondary: Brainstorming docs)
3. **Guests** (Tertiary: Lists and RSVPs)

### 2.3 Task Portal Component (`<RichTextEditor sheetMode={true} />`)

When a subtask carries the `rich_text` type, the Task Sheet renders the full-screen editor overlay.

**Data Flow:**

1. The component initializes with an empty block state or fetches the existing `note` if `linkedObjects.notePageId` is populated.
2. The UI auto-saves the `jsonb` payload to the database on a delayed debounce or strictly `onBlur`/Dismiss.
3. If this is a new note, the resulting `id` is pushed back to the subtask's `linkedObjects.notePageId` to maintain the contextual link.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of providing a rich text block editor, we will monitor:

- **Contextual vs Global Creation Ratio:** The percentage of Notes created specifically through a Task Portal vs globally. (Validates if users appreciate the contextual grouping).
- **Block Diversity:** The percentage of notes that utilize advanced blocks (e.g., checklists, headers) vs just plain paragraph text. (Shows if the `/` command is discoverable and useful).
- **Average Word Count per Workspace:** Indicates if the platform is truly hosting their "brain" or if they are still using external tools like Google Docs.
