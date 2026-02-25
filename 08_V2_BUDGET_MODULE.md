# V2 Budget Module Specification

## Overview

The Budget module is designed to provide financial peace of mind. It transforms the traditionally stressful experience of managing wedding expenses in a spreadsheet into a clear, visual, and highly actionable system.

---

## 1. The "Buddy" Budget (For Stakeholders)

### 1.1 Envelopes vs. Line Items

A wedding budget needs to be managed from the top-down and the bottom-up simultaneously. Moodscapes handles this by splitting the budget into two concepts:

- **Envelopes (The Plan):** High-level category limits established early on (e.g., "We will spend $4,000 total on Photography").
- **Line Items (The Reality):** The actual payments made within those envelopes (e.g., "Photographer Deposit: $1,000", "Final Balance: $3,000").

### 1.2 The "Hero" Visualization

At the top of the Budget tab sits a massive, dynamic visualization (like a colorful donut chart). It provides immediate financial clarity:

- What is our Total Budget?
- How much have we actually Paid?
- What is our Remaining Balance?
  This chart updates in real-time as line items are checked off.

### 1.3 Automatic Payment Reminders

Missing a final vendor payment can be disastrous. In V2, when a user logs a Line Item payment, they attach a Due Date.
As that date approaches, the system automatically surfaces a reminder on the Home Dashboard: _"Your $3,000 Catering Balance is due in 3 days."_

### 1.4 Task Portal Integration

If the AI gives the user a task to "Pay the Florist Deposit", the user taps the task. A simple interface slides up showing the amount and a large "Mark as Paid" button. Tapping it updates the budget, checks off the task, and recalculates the Hero Visualization instantly.

---

## 2. Technical Implementations & Schemas (For Developers)

The budget logic relies on hierarchical data structures tied to the workspace.

### 2.1 Entity Relationships

1. **`budget_envelopes`**: The high-level category allocations linked to a `workspace_id`. Contains the `category` string and the `allocated_amount` integer.
2. **`payments`**: The specific line-item transactions. These belong to an `envelope_id` (foreign key) and optionally link to a `vendor_id`. They track `amount`, `due_date`, and `status` ('pending', 'due', 'paid').

_Note: Total Spent is calculated on the fly by summing `payments.amount` where `status = 'paid'` and grouping by `envelope_id` or `workspace_id`._

### 2.2 Task Portal Component (`<BudgetEnvelopeAllocation inline={true} />`)

When a subtask carries the `budget_line` type, the Task Sheet renders the embedded budget portal.

**Data Flow:**

1. The portal component queries `payments` linked to the current subtask (via `linkedObjects.budgetEnvelopeId`).
2. If the user clicks "Mark as Paid", an API call updates `payments.status = 'paid'`.
3. The component dispatches a global `BUDGET_UPDATED` event, triggering a refetch for the Hero Visualization on the main Budget tab (if globally cached) to ensure data consistency across the OS.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the Budget Envelopes and Reminders, we will monitor:

- **Due Date Utilization:** The percentage of entered `payments` that utilize the optional `due_date` field.
- **Reminder Interaction Level:** Do users click through the Home Dashboard reminders to mark items as paid?
- **Envelope Completion Ratio:** The number of users who input a custom allocated budget limit vs leaving defaults blank.
- **On-Platform Task Closing:** The % of payments marked "Paid" via a Task Portal inline vs navigating manually to the Budget Tab.
