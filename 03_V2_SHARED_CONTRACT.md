# Phase 0: The V2 Shared Contract (The Constitution)

**Immutable Source of Truth for All Agents working on V2**

> ⚠️ **CRITICAL INSTRUCTION FOR ALL AGENTS:**
> This document defines the **Shared Types, Schemas, and Contracts** for the V2 transition.
> V2 introduces interconnected entities (the "OS" concept) while remaining strictly backwards-compatible with MVP schemas.
> You **MUST** adhere to these definitions. Do not modify existing MVP columns.

---

## 1. Domain Types Overview (TypeScript)

### 1.1 Additive Task Extensibility & Workspaces

The `Task` and `Subtask` types from MVP are retained but extended with optional properties for V2 features. To support users planning multiple independent events, a `workspaceId` is introduced.

```typescript
export type TaskStatus = "todo" | "completed" | "snoozed" | "archived";
export type SubtaskType =
  | "checkbox"
  | "mood_board"
  | "vendor_browse"
  | "calendar_event"
  | "budget_line"
  | "rich_text"
  | "file_upload"
  | "external_link";

export interface Subtask {
  id: string;
  text: string; // Keep MVP naming (text = title)
  isCompleted: boolean;

  // V2 Additions:
  types?: SubtaskType[]; // Defines what portal embedded views open (e.g., ['mood_board', 'calendar_event'])
  portalConfig?: any; // Pre-filters applied when portal opens
  linkedObjects?: LinkedObjects; // Links created dynamically while using the portal
}

export interface Task {
  id: string;
  title: string;
  description: string;
  category: string;
  emoji: string;
  subtasks: Subtask[];
  isCompleted: boolean;
  deleted: boolean;
  snoozedUntil?: string;
  aiHelpPrompt?: string;

  // V2 Additions:
  linkedObjects?: LinkedObjects; // Connective tissue bridging domains
  notes?: any[]; // Rich text blocks (Notion style)
}

export interface LinkedObjects {
  vendorIds: string[];
  calendarEventIds: string[];
  budgetEnvelopeId?: string;
  inspirationPinIds: string[];
  notePageId?: string;
  externalLinks: { label: string; url: string; addedAt: string }[];
}
```

### 1.2 Core Domain Relationships (New Entities)

To turn tasks into an interconnected OS, the following new domain objects exist globally per wedding profile:

```typescript
// Vendor System
export interface SavedVendor {
  id: string;
  userId: string;
  name: string;
  category: string;
  websiteUrl?: string;
  instagramHandle?: string;
  status: "shortlisted" | "contacted" | "hired" | "contract_signed" | "paid";
  assignedTaskIds: string[];
}

export interface PlatformCoordinator {
  id: string;
  name: string;
  email: string; // Used for magic-link invite upon hiring
  portfolioUrl?: string;
  bio?: string;
  basePrice: number;
  platformFeePercentage: number; // Set by admin during onboarding
  isActive: boolean; // Admin toggle to hide/show in marketplace
}

// Budget System
export interface BudgetEnvelope {
  id: string;
  userId: string;
  category: string;
  allocatedAmount: number;
}
export interface Payment {
  id: string;
  envelopeId: string;
  vendorId?: string;
  amount: number;
  dueDate?: string; // ISO
  status: PaymentStatus;
  notes?: string;
}

// Inspiration System
export interface InspirationPin {
  id: string;
  userId: string;
  imageUrl: string;
  category: string;
  linkedTaskIds: string[];
}

// Calendar System
export interface CalendarEvent {
  id: string;
  userId: string;
  title: string;
  startTime: string; // ISO
  endTime: string; // ISO
  linkedTaskId?: string;
}

// AI Chat System
export interface AIChatSession {
  id: string;
  userId: string;
  contextType: "global" | "task" | "vendor" | "budget";
  contextId?: string; // e.g., taskId if contextType is 'task'
  createdAt: string;
  updatedAt: string;
}

export interface AIChatMessage {
  id: string;
  sessionId: string;
  role: "user" | "assistant";
  content: string;
  actionCards?: any[]; // e.g., embedded task or vendor cards
  createdAt: string;
}

// Guest Management System
export type RSVPStatus = "not_sent" | "pending" | "accepted" | "declined";

export interface GuestGroup {
  id: string;
  userId: string;
  name: string; // e.g. "The Smith Family"
  address?: string;
  contactEmail?: string;
  contactPhone?: string;
}

export interface Guest {
  id: string;
  groupId: string;
  firstName: string;
  lastName: string;
  isChild: boolean;
  isPlusOne: boolean;
  dietaryTags?: string[]; // e.g., ['Vegan', 'Gluten-Free']
  dietaryNotes?: string;
  tableId?: string; // Links to a seating chart table if using
}

export interface Event {
  id: string;
  userId: string;
  name: string; // e.g., "Welcome Dinner", "Ceremony", "Reception"
  date: string; // ISO
}

export interface GuestRSVP {
  id: string;
  guestId: string;
  eventId: string;
  status: RSVPStatus;
  mealChoice?: string;
  notes?: string;
}

// Notes & Docs System
export interface Note {
  id: string;
  userId: string;
  title: string;
  content: any; // e.g. JSON representation of block-style rich text
  category: "vows" | "vendor_notes" | "brainstorming" | "logistics" | "uncategorized";
  linkedTaskId?: string;
  isPinned: boolean;
  createdAt: string;
  updatedAt: string;
}

// Workspace & Collaboration System
export interface Workspace {
  id: string;
  userId: string; // The original creator
  name: string; // e.g. "John & Sarah's Wedding" or "Baby Shower"
  eventDate?: string;
  createdAt: string;
}

export type CollaboratorRole = "partner" | "coordinator" | "vendor_limited";

export interface WorkspaceCollaborator {
  id: string;
  workspaceId: string; // The specific workspace instance
  collaboratorEmail: string;
  collaboratorId?: string; // Links to auth.users if they have registered
  role: CollaboratorRole;
  createdAt: string;
}
```

---

## 2. Database Schema (Supabase)

> **Migration Principle:** V2 only _adds_ columns to existing tables and introduces _new_ tables. We will not alter or delete MVP columns (`profiles`, `tasks`).

### 2.1 Updating Existing Tables

**`tasks` Table Evolution:**

```sql
ALTER TABLE tasks
  ADD COLUMN IF NOT EXISTS workspace_id uuid, -- For grouping multiple distinct events
  ADD COLUMN IF NOT EXISTS linked_objects jsonb DEFAULT '{"vendorIds": [], "calendarEventIds": [], "inspirationPinIds": [], "externalLinks": []}'::jsonb,
  ADD COLUMN IF NOT EXISTS notes jsonb DEFAULT '[]'::jsonb;
```

> **Note on Workspaces:** All new V2 tables below should include a `workspace_id uuid` column. For backwards compatibility with MVP users, queries will fall back to reading the raw `user_id` if `workspace_id` is null.

### 2.2 New V2 Tables

```sql
create table saved_vendors (
  id uuid default gen_random_uuid() primary key,
  workspace_id uuid not null, -- Links to specific wedding event
  name text not null,
  category text not null,
  website_url text,
  instagram_handle text,
  status text not null,
  assigned_task_ids uuid[] default array[]::uuid[],
  created_at timestamptz default now()
);

create table platform_coordinators (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  email text unique not null,
  portfolio_url text,
  bio text,
  base_price numeric(10, 2) not null,
  platform_fee_percentage numeric(5, 2) not null default 10.00,
  is_active boolean default false, -- Set true by admins after vetting
  created_at timestamptz default now()
);

create table budget_envelopes (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  category text not null,
  allocated_amount numeric(10, 2) not null,
  linked_task_id uuid -- Envelope created from a task portal
);

create table payments (
  id uuid default gen_random_uuid() primary key,
  envelope_id uuid references budget_envelopes on delete cascade not null,
  vendor_id uuid references vendors on delete set null,
  amount numeric(10, 2) not null,
  due_date timestamptz,
  status text check (status in ('pending', 'due', 'paid', 'overdue', 'cancelled')) default 'pending',
  notes text,
  created_at timestamptz default now()
);

create table inspiration_pins (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  image_url text not null,
  category text,
  linked_task_ids uuid[] default array[]::uuid[],
  saved_at timestamptz default now()
);

create table calendar_events (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  title text not null,
  start_time timestamptz not null,
  end_time timestamptz not null,
  type text,
  linked_task_id uuid
);

create table ai_chat_sessions (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  context_type text not null check (context_type in ('global', 'task', 'vendor', 'budget')),
  context_id uuid, -- Can be null for global chat
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table ai_chat_messages (
  id uuid default gen_random_uuid() primary key,
  session_id uuid references ai_chat_sessions on delete cascade not null,
  role text not null check (role in ('user', 'assistant')),
  content text not null,
  action_cards jsonb default '[]'::jsonb,
  created_at timestamptz default now()
);

create table guest_groups (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  name text not null,
  address text,
  contact_email text,
  contact_phone text
);

create table guests (
  id uuid default gen_random_uuid() primary key,
  group_id uuid references guest_groups on delete cascade not null,
  first_name text not null,
  last_name text not null,
  is_child boolean default false,
  is_plus_one boolean default false,
  dietary_tags jsonb default '[]'::jsonb,
  dietary_notes text,
  table_id uuid
);

create table events (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  name text not null,
  date timestamptz not null
);

create table guest_rsvps (
  id uuid default gen_random_uuid() primary key,
  guest_id uuid references guests on delete cascade not null,
  event_id uuid references events on delete cascade not null,
  status text not null check (status in ('not_sent', 'pending', 'accepted', 'declined')),
  meal_choice text,
  notes text,
  unique (guest_id, event_id)
);

create table notes (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  title text not null,
  content jsonb not null default '[]'::jsonb,
  category text not null check (category in ('vows', 'vendor_notes', 'brainstorming', 'logistics', 'uncategorized')),
  linked_task_id uuid,
  is_pinned boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table workspaces (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null, -- The creator
  name text not null,
  event_date timestamptz,
  created_at timestamptz default now()
);

create table workspace_collaborators (
  id uuid default gen_random_uuid() primary key,
  workspace_id uuid references workspaces on delete cascade not null,
  collaborator_email text not null,
  collaborator_id uuid references auth.users on delete set null,
  role text not null check (role in ('partner', 'coordinator', 'vendor_limited')),
  created_at timestamptz default now(),
  unique (workspace_id, collaborator_email)
);
```

---

## 3. API Contracts (Gemini AI Gen System)

To support the V2 OS feeling, the LLM generating the Weekly Plan must specify the interconnected schema parameters in the Subtasks.

### 3.1 Prompt Input Update

The system config must evaluate `linkedObjects` on `competedTasks` to inform the next generation, retaining context.

### 3.2 Subtask Generation Target (JSON Schema Additions)

```json
{
  "type": "array",
  "items": { "type": "string", "description": "Max 10 words" }
}
```

**Evolves to:**

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "text": { "type": "string", "description": "Max 20 words" },
      "types": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["checkbox", "mood_board", "vendor_browse", "calendar_event", "budget_line"]
        }
      }
    }
  }
}
```

_Note: We must map the generated `types` and `text` properties into the client's MVP-compatible array schema before persisting the new structure._
