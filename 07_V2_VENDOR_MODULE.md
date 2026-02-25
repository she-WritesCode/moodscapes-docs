# V2 Vendor Directory Module Specification

## Overview

Finding and managing vendors is one of the most time-consuming parts of wedding planning. The Vendor Directory module serves as both a discovery platform and a relationship management tool (a mini-CRM) for the couple and their coordinators.

---

## 1. The Vendor CRM (For Stakeholders)

### 1.1 The Dual-Strategy: Personal CRM vs. Global Marketplace

To balance low operational overhead with strong monetization, V2 uses a dual-strategy for vendors:

- **General Vendors (Personal CRM Base):** For florists, DJs, and caterers, Moodscapes acts as your private workspace. We do not maintain a global database of every florist on Earth. Instead, users bring their own leads into the app via the "Link Sandbox".
- **The Coordinator Marketplace (Monetization Engine):** Hiring Professional Planners is how the platform generates revenue. This is the _only_ category that utilizes a **Global Directory**. Users can browse, contact, and officially hire Moodscapes-approved coordinators directly through the app.
  - **Admin Onboarding:** Moodscapes admins manually vet and onboard Coordinators onto the platform, setting their profile, portfolio, and platform fee percentage.
  - **The Hiring Flow:** When a couple clicks "Hire & Pay Deposit" on a Coordinator's profile, the system:
    1. Processes the transaction (triggering Moodscapes' revenue cut).
    2. Automatically creates a `workspace_collaborators` record.
    3. Dispatches a "You've been hired!" email to the Coordinator with a magic link.
    4. Upon clicking the link, the Coordinator's dashboard immediately grants them access to the couple's workspace.

### 1.2 The "Link Sandbox" (How Users Add Vendors)

Instead of searching a generic directory, couples find inspiration organically on social media, then let Moodscapes do the heavy lifting:

1. **The Discovery:** A bride finds a beautiful cake on an Instagram Reel or a TikTok video.
2. **The Import (Sandbox):** She opens the Moodscapes Vendor page and pastes the URL (e.g., `instagram.com/p/SweetTreatsBakery`) into the "Add Vendor" input field.
3. **AI Extraction Magic:** Behind the scenes, the AI rapidly parses the HTML/metadata of that link. It automatically fills out a new Vendor Card for her:
   - **Name:** Sweet Treats Bakery
   - **Category:** Baker / Cake
   - **Contact:** Parses email/phone number from their bio.
4. **The Result:** The vendor is instantly saved to her CRM as a "Shortlisted" lead, completely skipping manual data entry.

### 1.3 "My Team" Dashboard

Once a vendor goes from "Shortlisted" to officially "Hired," they are promoted to the **My Team** section on the Home Dashboard. These appear as "speed dial" avatars (e.g., a small circular profile picture of the photographer) for lightning-fast communication.

### 1.4 The Hiring Funnel

Couples don't just "book" a vendor instantly. It's a relationship that evolves.
Instead of just a "Save" button, the Moodscapes Vendor CRM allows users to track exactly where they are in the process with each professional:

- **Shortlisted** (Saved for later)
- **Contacted** (Reached out for a quote)
- **Meeting Set** (Consultation scheduled)
- **Hired** (Contract signed)

### 1.5 Task Portal Integration

If a weekly task is "Research Top 3 Florists," the user doesn't navigate away to search Google. They tap the task, and the **Vendor Portal** slides up perfectly pre-filtered to "Florists in your area." The user taps "Shortlist" on three options, the portal closes, and the task checks itself off automatically.

---

## 2. Technical Implementations & Schemas (For Developers)

The backend must track both the global inventory of vendors and the specific, stateful relationships a user has with them.

### 2.1 Entity Relationships

To support this dual-strategy, the data model utilizes two primary tables:

1. **`saved_vendors`**: A table directly tied to the user's `workspace_id` acting as the Personal CRM for general vendors (florists, etc.). Contains link-scraped details and their current funnel `status`.
2. **`platform_coordinators`**: A global database managed by Moodscapes admins. Professional coordinators apply to be listed here, and users can hire them directly, triggering the platform's revenue capture.

### 2.2 Extraction API & AI Parser Flow

- **The `POST /api/vendors/extract` Endpoint:**
  When a user pastes a URL, the frontend calls this endpoint. The backend fetches the raw HTML metadata from the URL and passes it to the LLM (e.g., Gemini) with a strict JSON schema asking for `name`, `category`, and `contact_methods`.
- **Dashboard "My Team" Widget:**
  Queries `saved_vendors` where `workspace_id = ? AND status = 'hired'`.
- **Vendor Task Portal (`<VendorDirectory sheetMode={true} />`):**
  When rendered inside a task, it receives a `category` prop (e.g., "florals").
  When a user adds a vendor link here, the component:
  1. Calls the Extraction API.
  2. Inserts the new row into `saved_vendors` with `status: 'shortlisted'`.
  3. Dispatches an event that the parent Task Sheet intercepts to append the generated `saved_vendor_id` to the task's `linkedObjects.vendorIds` array.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the Vendor Directory and CRM tracking, we will monitor:

- **Hiring Funnel Conversion Rate:** The percentage of vendors that progress from `Shortlisted` -> `Contacted` -> `Hired`. (Proves the utility of our internal messaging/tracking vs users taking it off-platform).
- **Task Portal Efficacy:** The percentage of saved/shortlisted vendors discovered organically inside the global directory vs discovered right inside a Task Portal.
- **"My Team" Engagement:** Frequency of users interacting with the quick-access avatars on the Dashboard to retrieve contact info.
