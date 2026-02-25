# V2 Collaboration & Roles Specification

## Overview

Wedding planning is inherently multiplayer. While the app is typically created by one person (the "Primary Owner"), they need to seamlessly invite their Partner and their hired professionals (like a Coordinator) to function as app users with shared access to the same centralized workspace.

---

## 1. Multiplayer Planning (For Stakeholders)

### 1.1 The Workspace Architecture

In V2, users aren't just limited to planning one event. A user can create a **Workspace** (e.g., "Sarah & John's Wedding").
If that user is a professional Wedding Planner, they can create _multiple_ workspaces (e.g., "Client A", "Client B").

### 1.2 Inviting Your Team

The Primary Owner can invite people to join their specific Workspace directly from the Settings tab. They simply enter an email address and select a Role. The invited person receives an email, downloads the app, and logs in to see that specific wedding.

### 1.3 Roles & Permissions

- **Primary Owner:** The creator. Can manage billing, delete the workspace, and invite/revoke others.
- **Partner:** The other half of the couple. Has full access to read, write, and chat with the AI. They cannot lock out the Primary Owner.
- **Coordinator (Vendor):** A hired professional. Couples typically discover and hire these verified planners through the platform's internal **Coordinator Marketplace**, generating revenue for Moodscapes. Once hired:
  - They have _Full Access_ to Tasks, the Calendar, and the Vendor Directory.
  - They have _Read-Only_ access to the Budget (to see what is paid/unpaid without deleting budgets).
  - They have _Read-Only_ access to the Guest List (with optional permissions to edit seating charts).
  - If they chat with the AI, their messages are tagged so the couple knows the answers were provided to the planner, ensuring full transparency.

---

## 2. Technical Implementations & Schemas (For Developers)

Because Moodscapes MVP anchored all data to a single `user_id`, we maintain backwards compatibility by keeping `user_id` as the "Primary Owner" ID on all records as a fallback. However, the V2 architecture explicitly introduces the `workspace_id` routing layer.

### 2.1 Workspace Entity Relationships

1. **`workspaces`**: Represents a distinct planning environment. Contains `user_id` (the original creator) and `name`.
2. **`workspace_collaborators`**: Resolves access rights by linking an invited user (`collaborator_email` or `collaborator_id`) to a specific `workspace_id` with a designated `role` ('partner', 'coordinator', 'vendor_limited').

_Note: All core domain tables (tasks, vendors, budget_envelopes, calendar_events, etc.) must now include a `workspace_id` foreign key. Row Level Security (RLS) policies should validate access by checking if the current `auth.uid()` matches either the `workspaces.user_id` OR exists in `workspace_collaborators` for that `workspace_id`._

### 2.2 The Professional Dashboard

When a Coordinator (or any invited user) logs into the app, the frontend queries `workspace_collaborators` for their `auth.uid()`.
If they belong to multiple workspaces, the Home screen renders a "Workspace Selector" list.
Tapping a workspace sets that `workspace_id` in the global application state/context, ensuring all downstream queries (tasks, budget, vendors) fetch data strictly for that specific event.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of building a robust RBAC (Role-Based Access Control) multi-tenant system, we will monitor:

- **Invite Virality:** Average number of invitations sent per active workspace. (A direct measure of network effects).
- **Partner Activation Rate:** The percentage of invited 'Partners' who actually create an account and log > 3 sessions.
- **Coordinator Capture:** The number of users utilizing the 'Coordinator' role. (This hints at potential B2B pivot/expansion opportunities).
