# Spec: Admin Dashboard

**Scope:** Internal multi-tab analytics and user management dashboard.

---

## 1. Overview

The Admin Dashboard is a restricted internal tool designed to provide a 360-degree view of product health, user engagement, and system reliability. It follows a "dummy-proof" design philosophy, using plain English tooltips and consolidated views to make data accessible to non-technical stakeholders.

---

## 2. Security (The Admin Gate)

**Route:** `/admin`

**Authentication Mechanism:**
- Gated by a mandatory password entry screen.
- **Default Password:** `moodscapes2026`
- **Configuration:** Can be overridden via `VITE_ADMIN_PASSWORD` environment variable.
- **Persistence:** Authenticated state is stored in `sessionStorage`. 
  - Access persists across page refreshes.
  - Access is cleared when the browser tab is closed or when the user manually signs out.

**UI:** A premium, dark-themed login screen featuring glassmorphism and security icons.

---

## 3. Navigation

**Tab Structure:**
- **Overview**: High-level heartbeats (Users, Activity, Funnels).
- **People**: Searchable user management table + Retention metrics.
- **User Activity**: Detailed interaction patterns, Task usage, and AI Guide performance.
- **System Status**: Technical vitals and error monitoring.

**Mobile Experience:**
- Horizontal scrolling tab bar with a hidden scrollbar.
- Responsive grid layouts for metric cards.

---

## 4. Feature Sections

### 4.1 Overview Tab
- **Metric Cards**: Total Users, Weekly Active, Onboarding Completion %, Avg Tasks/User.
- **User Growth Chart**: Visualizes signups over time.
- **Conversion Funnel**: Multi-stage funnel tracking (Landing → Signup → Onboarding → First Task).
- **Onboarding Breakdown**: Detailed view of each step in the onboarding flow.

### 4.2 People Tab
- **User Table**: Comprehensive list of users with engagement scores, wedding dates, and activity status.
- **Retention Summary**: Simple D1/D7/D30 retention figures.
- **Retention Heatmap**: Cohort-based analysis of user stickiness by week.

### 4.3 User Activity Tab
- **Active User Tiers**: Daily/Weekly/Monthly Active counts.
- **Task Analytics**: Distribution of tasks by category and status.
- **AI Guide Stats**: Usage frequency, response times, and message volume.
- **Milestones**: Breakdown of user-reached planning milestones (e.g., "Budget Set").

### 4.4 System Status Tab
- **Technical Performance**: Page load times, AI generation latency, and DB response times.
- **Reliability Dashboard**: Error rates, API success/failure logs, and system uptime markers.

---

## 5. Design Guidelines

- **Typography**: Large, bold headlines for clarity.
- **Colors**: 
  - Neutral zinc background for readability.
  - Vibrant accent bars (Purple: Onboarding, Teal: Retention, Blue: Interaction, Rose: AI).
- **Tooltips**: Every complex metric or header must have a plain-English tooltip accessible via an info icon (e.g., "The percentage of users who...").

---

## 6. Analytics Accuracy
- **Admin Exclusion**: The `/admin` path is excluded from automated `session_started` tracking to prevent admin testing from inflating DAU/WAU metrics.

---

## 7. Technical Implementation

**Service Layer:** `/src/services/adminService.ts`
- Centralized data fetching using Supabase JS client.
- Custom RPC `count_unique_users_by_event` for accurate unique user funnel tracking.

**State Management:**
- React state for metric loading.
- `Promise.all` for parallel loading of dashboard sections.
- `sessionStorage` for authentication persistence.
---

## 8. Acceptance Criteria

- [ ] `/admin` route is password protected with the gate UI.
- [ ] Correct password ("moodscapes2026") unlocks the dashboard.
- [ ] Authentication state persists across page refreshes via `sessionStorage`.
- [ ] Dashboard is consolidated into 4 functional tabs (Overview, People, Activity, Status).
- [ ] "Overview" tab shows key engagement metrics and conversion funnel.
- [ ] "People" tab displays searchable user list and cohort retention heatmap.
- [ ] "Activity" tab tracks AI Guide messages and User Milestones.
- [ ] "Status" tab monitors technical vitals (Latency, Load Times).
- [ ] All charts and headers include plain-English "dummy-proof" tooltips.
- [ ] Navigation is mobile-responsive with horizontal scrolling.
- [ ] Sign Out button explicitly clears the session.
- [ ] No `session_started` events are tracked for admin users.

---

## 9. Future Enhancements (Post-MVP)

- [ ] **Role-Based Access Control (RBAC)**: Support multiple admin levels with different permissions.
- [ ] **Data Export**: Allow exporting user list and interaction logs to CSV/JSON.
- [ ] **Real-Time Updates**: Integrated Supabase Realtime for live user activity feeds.
- [ ] **Actionable User Management**: Ability to reset passwords or manually trigger task generation from the admin view.
- [ ] **Advanced Filtering**: Categorize users by wedding date ranges or budget tiers.
- [ ] **A/B Testing Integration**: Track conversion rates based on different onboarding variations.
