# Moodscapes V2 Specifications

Welcome to the Moodscapes V2 Technical and Product Specifications.

Please use the sidebar on the left to navigate through the core modules of the Wedding OS. Alternatively, start reading at the [Product Requirements Document (PRD)](01_V2_PRD.md).

---

## 🎯 We Need Your Help with Prioritization

As a stakeholder, your feedback is critical in deciding **what we build first**. We use a community-driven prioritization model.

### How to Vote and Comment

At the very bottom of every specification page, you will see a **Discussion Board**.

1. **Sign In:** Click "Sign in with GitHub" in the comment box if you aren't already logged in.
2. **Leave a Comment:** Read through a module (e.g., the [Budget Module](08_V2_BUDGET_MODULE.md)) and write a comment at the bottom telling us if you think it is a high, medium, or low priority for our users. You can also suggest missing features!
3. **Upvote:** If another stakeholder has already left a comment that perfectly matches your thoughts, you can just click the **👍 Upvote** reaction on their comment to cast your vote!

We will use the total number of comments and upvotes on each page to finalize our V2 development roadmap. Let us know what you want to see come to life first!

### 📊 Feature Prioritization Matrix

Please review the upcoming V2 features below. In the **Discussion Board at the very bottom of this page**, leave a single comment listing these features and tell us if you think they are **High, Medium, or Low priority** for our immediate roadmap.

| Core Feature Module   | Description                                                   | Your Priority Vote             |
| :-------------------- | :------------------------------------------------------------ | :----------------------------- |
| **Budget Tracker**    | Visual envelope budgeting with payment reminders              | _Comment below (High/Med/Low)_ |
| **Vendor Directory**  | CRM to discover, shortlist, and manage hired pros             | _Comment below (High/Med/Low)_ |
| **Guest Management**  | Unified roster handling RSVPs for multiple cultural events    | _Comment below (High/Med/Low)_ |
| **Vision Board**      | Filterable masonry grid for styled inspiration                | _Comment below (High/Med/Low)_ |
| **Calendar Sync**     | Two-Way Google Calendar integration to prevent double-booking | _Comment below (High/Med/Low)_ |
| **Multiplayer Roles** | Ability to invite a Partner or professional Coordinator       | _Comment below (High/Med/Low)_ |

---

**💡 Pro Tip:** If another stakeholder has already left a comment that perfectly matches your thoughts, you can just click the **👍 Upvote** reaction on their comment to cast your vote!

We will use the total number of comments and upvotes on each page to finalize our V2 development roadmap. Let us know what you want to see come to life first!

---

## 🤖 How to Build this App with AI

This documentation is specifically engineered to be read by AI coding assistants (like Lovable, Cursor, or Devin). Because we have strict data contracts and UI rules, the AI will not hallucinate database structures or guess variable names.

**CRITICAL RULE:** Do NOT dump all these files into an AI at once. It will overwhelm the context window. Feed it the specs sequentially using the prompts below:

### Prompt 1: The Foundation (Do this first)

> **Action:** Upload the `inspo` folder images, `02_V2_UI_ARCHITECTURE.md`, and `03_V2_SHARED_CONTRACT.md` to your AI.
>
> **Prompt:** _"I am building V2 of Moodscapes. I have structured the architecture and data contracts. Please read the UI Spec to understand our tech stack, visual design rules, and aesthetic derived from the attached inspiration images. Then, read the Shared Contract spec and explicitly generate all the TypeScript interfaces and Supabase database schema migrations defined in that file. Do not invent new variables; strictly adhere to the contract."_

### Prompt 2: The Core UI Shell

> **Action:** Do not upload any new files.
>
> **Prompt:** _"Now that we have the types and visual rules, please build the global layout wrapper. We are using a 'Nested Sheet' architecture to mimic a native mobile app. Build the empty global Zustand store to manage nested sheets, and build the Bottom Navigation Bar with Home, Plan, Vision, Budget, and More tabs."_

### Prompts 3 through 12: Building Feature-by-Feature

> **Action:** Upload ONE module specification file at a time (e.g., `08_V2_BUDGET_MODULE.md`).
>
> **Prompt:** _"Please read the attached module specification. Using our existing TypeScript interfaces and following the visual aesthetic we established, build this specific UI module. Ensure you follow the 'Zero-Data Empty States' rule."_
