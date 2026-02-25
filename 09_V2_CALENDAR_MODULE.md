# V2 Calendar Module Specification

## Overview

The Calendar module provides a single, unified view of all scheduled events, vendor meetings, deposit deadlines, and wedding weekend itineraries. It ensures couples never miss a beat without needing to exit the Moodscapes app.

---

## 1. The Wedding Itinerary (For Stakeholders)

### 1.1 The "Zero-Surprise" Dashboard

Users don't always remember to check a dedicated calendar tab. To prevent missed meetings or deadlines, the Moodscapes Home Dashboard features a "Next Up" widget. This automatically surfaces the 2-3 most immediate chronological events right when the user opens the app.

### 1.2 Everything in One Place

The Calendar handles several different types of time-bound events effortlessly:

- **Meetings:** Consultations with vendors (Cake Tastings, Dress Fittings).
- **Deadlines:** Contract signings, deposit due dates (auto-synced from the Budget module).
- **Sub-Events:** The actual itinerary for the wedding weekend (Rehearsal Dinner, Farewell Brunch) auto-synced from the Guest Management module.

### 1.3 Hybrid Calendar Integration (Two-Way & ICS Fallback)

Couples still live their lives in external calendars. Moodscapes provides a **Hybrid Sync Architecture** to support all users:

1. **Two-Way Google Calendar Sync (For Power Users):**
   A direct connection to Google accounts.
   - **Pushing Out:** Any meeting or deadline created in Moodscapes is instantly pushed to the user's Google Calendar.
   - **Pulling In (Availability):** Moodscapes reads the user's Google Calendar to block out "Busy" times. When trying to schedule a vendor meeting in the app, the UI visually grays out times where the user already has a work meeting. _(Note: We only read the "busy" state, not the external event titles)._
2. **ICS Subscription (Universal Fallback):**
   For Apple Calendar, Outlook, or users who prefer not to grant OAuth permissions, Moodscapes provides a secure, one-way ICS sync link. The user subscribes to this URL, guaranteeing that their planned activities appear in their preferred app (though updates rely on the external app's refresh rate, typically every 12-24 hours).

#### UI Entry Point for Integration

- **Primary Setup:** A prominent banner at the top of the **Calendar Tab** ("Connect Google Calendar to prevent scheduling conflicts"). Below it sits a secondary option: "Or generate an ICS link for Apple/Outlook".
- **Settings Fallback:** Both options are permanently available in the **More / Settings** tab under "Integrations".
- **In-Context Prompt:** If a user opens a Task Portal to schedule a meeting (e.g. "Cake Tasting"), the portal will offer a seamless "Connect Calendar to view availability" inline button.

### 1.4 Task Portal Integration

If a weekly task is to "Schedule a Venue Tour," tapping the task slides up an "Add Event" form directly over the checklist. The user picks the date and time, hits save, and the event populates the calendar while simultaneously checking off the task.

---

## 2. Technical Implementations & Schemas (For Developers)

The Calendar is a relatively flat data structure that aggregates timestamps from various parts of the OS.

### 2.1 Entity Relationships

1. **`calendar_events`**: The core schema carrying `title`, `start_time`, `end_time`, and `type`. All records are tied to a `workspace_id`.
2. **Implicit Calendar Sources**: To provide a truly unified calendar, the frontend calendar view should combine records from `calendar_events` together with:
   - `events` (From the Guest Management module, representing the sub-events like the Ceremony).
   - `payments` (From the Budget module, where `due_date` is not null).

### 2.2 Task Portal Component (`<CalendarEventCreator inline={true} />`)

When a subtask carries the `calendar_event` type, the Task Sheet renders the embedded event creator.

**Data Flow:**

1. The component accepts pre-fill props (e.g., `title="Venue Tour"`).
2. Upon submission, it `POST`s a new `calendar_events` row.
3. It dispatches a `CALENDAR_EVENT_CREATED` event.
4. The parent Task Sheet intercepts this, grabs the returned `event.id`, and appends it to the subtask's `linkedObjects.calendarEventIds` array.

### 2.3 Hybrid Sync Architecture (OAuth + ICS)

To support bi-directional syncing alongside a universal fallback:

1. **OAuth Flow (Google):** The frontend triggers a standard Google OAuth consent screen requesting `calendar.readonly` (for busy slots) and `calendar.events` (to write/update).
2. **Token Storage:** Access and refresh tokens are securely encrypted and stored in `workspace_integrations` (or `user_integrations`).
3. **Writing to Google:** When a `calendar_events` row is created/updated, a backend service hits the Google Calendar API to upsert the event to the user's primary calendar. The resulting `google_event_id` is stored in our row.
4. **Reading Availability (FreeBusy API):** When the `<CalendarEventCreator />` component mounts, it calls a Moodscapes API endpoint that proxies the Google Calendar `FreeBusy` query. It returns an array of blocked timestamps to automatically disable timeslots in the UI date-picker.
5. **ICS Fallback Endpoint:** The backend exposes an unauthenticated (but cryptographically hard-to-guess, e.g., `/api/calendar/sync/:uuid.ics`) endpoint. When hit, it gathers all `calendar_events`, `payments`, and `events` for that workspace and formats them into a standard `text/calendar` response string.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of building a unified platform Calendar, we will monitor:

- **Sync Adoption Rate:** The percentage of active workspaces that subscribe to the Calendar either via Google OAuth OR via an ICS link. (Shows if the feature actually replaces manual double-entry).
- **OAuth vs ICS Split:** Tracking which method users prefer (validates the dev effort spent on the specific Google integration).
- **Conflict Prevention Events:** How often the frontend FreeBusy query (for OAuth users) actually causes a UI timestamp to be disabled, preventing a double-booking.
