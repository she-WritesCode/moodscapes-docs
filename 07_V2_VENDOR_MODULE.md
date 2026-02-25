# V2 Vendor Directory Module Specification

## Overview

Finding and managing vendors is one of the most time-consuming parts of wedding planning. The Vendor Directory module serves as both a discovery platform and a relationship management tool (a mini-CRM) for the couple and their coordinators.

---

## 1. The Vendor CRM (For Stakeholders)

### 1.1 A Personal CRM vs. A Global Dashboard

To avoid the massive operational overhead of building a "Yelp-style" marketplace (which requires onboarding vendors and admins), the Moodscapes Vendor module is strictly designed as a **Personal CRM**.

Couples already find their inspiration and vendors on Instagram, TikTok, and Google. Our job is to help them organize those external findings.

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

To support this module, the data model utilizes a single, workspace-scoped table rather than a complex global pivot structure:

1. **`saved_vendors`**: A table directly tied to the user's `workspace_id` containing the vendor details (Name, Category, Website, IG Handle) and their current funnel `status` (shortlisted, contacted, hired). There is no "global" vendor database for admins to maintain.

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
