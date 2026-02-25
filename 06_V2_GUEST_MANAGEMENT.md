# V2 Guest Management Module Specification

## Overview

The Guest Management module is a core pillar of the Moodscapes V2 "Plan" tab. It transforms the chaotic process of managing RSVPs across multiple events into a streamlined, centralized dashboard.

---

## 1. The Unified Guest Experience (For Stakeholders)

### 1.1 The Global Roster

Couples hate maintaining multiple spreadsheets. In V2, the user builds their master guest list **once**. Aunt Mabel is added to the system as a primary entity. She has one profile where her dietary restrictions, address, and table assignment are stored permanently.

### 1.2 Multi-Event Toggling (Crucial for Multi-Day Celebrations)

A wedding is rarely just one party. In many cultures, a wedding consists of several distinct, major events spanning multiple days and locations (e.g., a Traditional Wedding on Friday, a White Wedding and Reception on Saturday, and a Thanksgiving service on Sunday).

Instead of creating five different guest lists, the user just taps on Aunt Mabel's master profile. Inside her profile, they see simple toggle switches:

- [ON] Invited to Rehearsal Dinner
- [ON] Invited to Ceremony & Reception
- [OFF] Invited to Farewell Brunch

RSVP statuses (`Attending`, `Declined`) and meal choices are tracked independently for _each specific event_.

### 1.3 Calendar Auto-Sync

Because guests are tied specifically to "Events", the system does the heavy lifting for the user's itinerary. If the user tells the Guest Module that the Rehearsal Dinner is at 7:00 PM on Friday, that event automatically appears as a milestone in the global **Calendar Module**. It is a true "single source of truth."

---

## 2. Technical Implementations & Schemas (For Developers)

The backend schema requires clear separation between a "Person" and an "Invitation" to support the multi-event flow described above.

### 2.1 Entity Relationships

1. **`workspaces`**: The overarching boundary for the application context. All entities below require a `workspace_id`.
2. **`guest_groups`**: Represents a single "Invitation Envelope" (e.g., "The Smith Family"). Addresses and emails sit here.
3. **`guests`**: Individual people belonging to a `guest_group`. Dietary tags sit here.
4. **`events`**: The sub-events of the wedding (Ceremony, Reception, Brunch). Sits directly under the workspace.
5. **`guest_rsvps`**: The crucial mapping table linking a `guest_id` to an `event_id`, including their `meal_choice` and `status` (`pending`, `accepted`, `declined`).

### 2.2 UI Component Architecture

When rendering the **More / Settings** Tab (or main Plan tab navigation):

- **Master List View:**
  Queries `guests` joined with `guest_groups` to display everyone.
- **Event Filter Tabs:**
  The UI provides sub-navigation pills (e.g., "All Guests", "Rehearsal", "Reception").
  Clicking "Rehearsal" modifies the query to `INNER JOIN guest_rsvps ON guest_rsvps.guest_id = guests.id WHERE guest_rsvps.event_id = [REHEARSAL_EVENT_ID]`.

- **Dynamic Stats Header:**
  The top of the UI calculates aggregates based on the current filter context:
  `SELECT count(*) FROM guest_rsvps WHERE event_id = ? AND status = 'accepted'`.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the multi-event Guest Management system, we will monitor:

- **Multi-Event Adoption:** The percentage of workspaces that define 2 or more sub-events (e.g., Rehearsal + Ceremony). (Proves if users actually find value in multi-party planning).
- **Average Guests per Workspace:** Indicates the depth of platform integration/commitment by the couple.
- **RSVP Tracking Completion Rate:** The percentage of guests marked with a definitive 'Accepted/Declined' status per wedding versus left as 'Pending'. (Proves the UI effectively helps them clear their backlog).
