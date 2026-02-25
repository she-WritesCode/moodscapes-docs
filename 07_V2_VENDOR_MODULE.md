# V2 Vendor Directory Module Specification

## Overview

Finding and managing vendors is one of the most time-consuming parts of wedding planning. The Vendor Directory module serves as both a discovery platform and a relationship management tool (a mini-CRM) for the couple and their coordinators.

---

## 1. The Vendor CRM (For Stakeholders)

### 1.1 The Global Directory vs. "My Team"

Planning involves looking at hundreds of options, but ultimately you only hire about 10-15 people. V2 makes a strict visual distinction between these two states:

- **Global Directory:** A massive, searchable marketplace of available vendors, filterable by category, location, and price point.
- **My Team:** A curated view showing _only_ the vendors the user has saved, contacted, or hired. On the home dashboard, these hired vendors appear as a quick-access "speed dial" (e.g., small circular avatars of the photographer and planner) for lightning-fast communication.

### 1.2 The Hiring Funnel

Couples don't just "book" a vendor instantly. It's a relationship that evolves.
Instead of just a "Save" button, the Moodscapes Vendor Directory allows users to track exactly where they are in the process with each professional:

- **Shortlisted** (Saved for later)
- **Contacted** (Reached out for a quote)
- **Meeting Set** (Consultation scheduled)
- **Hired** (Contract signed)

### 1.3 Task Portal Integration

If a weekly task is "Research Top 3 Florists," the user doesn't navigate away to search Google. They tap the task, and the **Vendor Portal** slides up perfectly pre-filtered to "Florists in your area." The user taps "Shortlist" on three options, the portal closes, and the task checks itself off automatically.

---

## 2. Technical Implementations & Schemas (For Developers)

The backend must track both the global inventory of vendors and the specific, stateful relationships a user has with them.

### 2.1 Entity Relationships

To support this module, the data model utilizes two primary tables mapped to the overarching workspace:

1. **`vendors`**: The global database of professionals. Contains static details (Name, Category, Rating, Location, Tags).
2. **`vendor_relationships`**: The pivot table linking a `workspace_id` to a `vendor_id`. This table tracks the current funnel `status` (shortlisted, contacted, hired).

### 2.2 UI Component Architecture & Data Flow

- **Dashboard "My Team" Widget:**
  Queries `vendor_relationships` where `workspace_id = ? AND status = 'hired'`, joining with `vendors` to display quick-access avatars and contact buttons.
- **Vendor Task Portal (`<VendorDirectory sheetMode={true} />`):**
  When rendered inside a task, it receives a `category` prop (e.g., "florals").
  When a user taps "Shortlist", the component:
  1. Upserts a `vendor_relationships` row.
  2. Dispatches an event that the parent Task Sheet intercepts to append the `vendor_id` to the task's `linkedObjects.vendorIds` array.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the Vendor Directory and CRM tracking, we will monitor:

- **Hiring Funnel Conversion Rate:** The percentage of vendors that progress from `Shortlisted` -> `Contacted` -> `Hired`. (Proves the utility of our internal messaging/tracking vs users taking it off-platform).
- **Task Portal Efficacy:** The percentage of saved/shortlisted vendors discovered organically inside the global directory vs discovered right inside a Task Portal.
- **"My Team" Engagement:** Frequency of users interacting with the quick-access avatars on the Dashboard to retrieve contact info.
