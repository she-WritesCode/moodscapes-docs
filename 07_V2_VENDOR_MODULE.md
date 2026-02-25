# V2 Vendor Directory Module Specification

## Overview

Finding and managing vendors is one of the most time-consuming parts of wedding planning. The Vendor Directory module serves as both a discovery platform and a relationship management tool (a mini-CRM) for the couple and their coordinators.

---

## 1. The Vendor CRM (For Stakeholders)

### 1.1 The Dual-Strategy: Personal CRM vs. Global Marketplace

To balance low operational overhead with strong monetization, V2 uses a dual-strategy for vendors:

- **General Vendors (Personal CRM):** For florists, DJs, and caterers, Moodscapes acts as a Personal CRM. Users paste an Instagram or Website link, and the AI auto-extracts the details to save them as a "Shortlisted" card. We do not need to maintain a global database of every florist on Earth.
- **The Coordinator Marketplace (Monetization Engine):** Hiring Professional Coordinators and Planners is how the platform generates revenue. Therefore, this is the _only_ category that utilizes a **Global Directory**. Users can browse, contact, and officially hire Moodscapes-approved coordinators directly through the app.

- **The Sandbox:** Users can quickly add a vendor by pasting an Instagram or Website link. The AI will auto-extract the vendor's name, category, and contact info, saving them as a "Shortlisted" card.
- **My Team:** A curated view on the Home Dashboard showing _only_ the vendors the user has officially hired. These appear as "speed dial" avatars for lightning-fast communication.

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

To support this dual-strategy, the data model utilizes two primary tables:

1. **`saved_vendors`**: A table directly tied to the user's `workspace_id` acting as the Personal CRM for general vendors (florists, etc.). Contains link-scraped details and their current funnel `status`.
2. **`platform_coordinators`**: A global database managed by Moodscapes admins. Professional coordinators apply to be listed here, and users can hire them directly, triggering the platform's revenue capture.

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
