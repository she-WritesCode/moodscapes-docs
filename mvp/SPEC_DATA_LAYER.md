# Spec: Data Layer (Data Agent)
**Scope: Supabase Schema + State Management + Migrations**

> ⚠️ **DEPENDENCY:** This agent must implement schemas defined in `SHARED_CONTRACT.md`.

---

## 1. Database Schema (Supabase)

### 1.1 Tables (DDL)

#### `profiles`
```sql
create table public.profiles (
  id uuid references auth.users not null primary key,
  names text not null,
  wedding_date date,
  budget_tier text check (budget_tier in ('under_30k', '30k_50k', '50k_70k', '70k_plus', 'figuring_it_out')),
  priorities jsonb default '[]'::jsonb,
  location text,
  guest_count integer,
  points integer default 0,
  streak integer default 0,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- RLS: Users can only read/update their own profile
alter table profiles enable row level security;
create policy "Users can view own profile" on profiles for select using (auth.uid() = id);
create policy "Users can update own profile" on profiles for update using (auth.uid() = id);
```

#### `tasks`
```sql
create table public.tasks (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  week_number integer not null,
  title text not null,
  description text not null,
  motivational_message text,
  subtasks jsonb not null, -- Array of {id, text, is_completed}
  status text default 'todo' check (status in ('todo', 'completed', 'snoozed')),
  category text,
  vendor_type text,
  is_ai_generated boolean default true,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- RLS: Users can only read/update their own tasks
alter table tasks enable row level security;
create policy "Users can view own tasks" on tasks for select using (auth.uid() = user_id);
create policy "Users can update own tasks" on tasks for update using (auth.uid() = user_id);
create policy "Users can insert own tasks" on tasks for insert with check (auth.uid() = user_id);
```

#### `reference_tasks` (Seed Data)
```sql
create table public.reference_tasks (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  description text not null,
  subtasks jsonb not null,
  category text,
  timeline_months_out integer,
  created_at timestamptz default now()
);

-- RLS: Public read-only
alter table reference_tasks enable row level security;
create policy "Public read access" on reference_tasks for select using (true);
```

---

## 2. Implemented Services (As of PR #75, Nov 26 2025)

### 2.1 `authService.ts` - Authentication & Profile Management

**Status:** ✅ Implemented, partially wired to UI

#### `createAnonymousAccount(): Promise<AuthResult>`
Creates an anonymous Supabase auth session for first-time users.

**Returns:**
```typescript
{
  userId: string;
  success: boolean;
  error?: string;
}
```

**Behavior:**
- Calls `supabase.auth.signInAnonymously()`
- Fallback to mock mode if Supabase unavailable
- Used for onboarding flow (no email/password required in MVP)

#### `saveProfile(userId: string, profileData: Omit<UserProfile, 'id' | 'createdAt'>): Promise<{success: boolean; error?: string}>`
Saves user profile to `profiles` table after onboarding.

**Schema Mapping:**
- `names` → `names` (text)
- `weddingDate` → `wedding_date` (date)
- `budgetTier` → `budget_tier` (text)
- `priorities` → `priorities` (jsonb array)
- `points` → `points` (default 0)
- `streak` → `streak` (default 0)

**Current Status:** Function exists, not called in onboarding flow yet.

#### `loadProfile(userId: string): Promise<UserProfile | null>`
Loads user profile from `profiles` table.

**Behavior:**
- Queries `profiles` WHERE `user_id = userId`
- Maps snake_case DB fields to camelCase TypeScript
- Returns `null` if profile not found

**Current Status:** ✅ Called on app mount in `App.tsx`

#### `getCurrentUser(): Promise<{userId: string | null}>`
Gets current authenticated user's ID from Supabase session.

**Current Status:** ✅ Called on app mount to check for existing session

---

### 2.2 `taskService.ts` - Task Generation & Persistence

**Status:** ✅ Implemented, AI generation working, persistence not wired

#### `generatePersonalizedTasks(profile: UserProfile): Promise<{tasks: Task[] | null; error?: string}>`
Calls Supabase Edge Function `generate-tasks` to create AI-generated tasks.

**Process:**
1. Transforms `UserProfile` → `onboardingData` format for Edge Function
2. Calls `supabase.functions.invoke('generate-tasks', {body: {onboardingData}})`
3. Transforms Edge Function response → `Task[]` format
4. Returns tasks in app-compatible format

**Edge Function Data Format:**
```typescript
{
  weddingDate: string; // YYYY-MM-DD
  guestCount: string;
  budgetRange: string;
  venueContext: string;
  vibes: string[];
  userName: string;
  partnerName: string;
}
```

**Current Status:** ✅ Called in `Onboarding.tsx`, working successfully

#### `saveTasks(userId: string, tasks: Task[], weekNumber: number = 1): Promise<{success: boolean; error?: string}>`
Persists AI-generated tasks to `tasks` and `subtasks` tables.

**Behavior:**
- Inserts each task into `tasks` table
- Inserts associated subtasks into `subtasks` table
- Maps camelCase TypeScript → snake_case DB fields

**Schema Mapping:**
- `task.title` → `title`
- `task.description` → `description`
- `task.category` → `category`
- `task.emoji` → `emoji`
- `task.isCompleted` → `completed`
- `subtask.text` → `text`
- `subtask.isCompleted` → `is_completed`

**Current Status:** ⚠️ Function exists but NOT called after task generation

#### `loadUserTasks(userId: string): Promise<Task[]>`
Loads user's tasks from database with subtasks.

**Query:**
```sql
SELECT *, subtasks (*)
FROM tasks
WHERE user_id = userId
  AND deleted = false
ORDER BY created_at DESC
```

**Current Status:** ✅ Called on app mount, but returns empty array (no tasks saved yet)

---

## 3. State Management (Service Layer)

### 3.1 Legacy `supabase.ts` Service Pattern (Deprecated)

> **Note:** The pattern below was the original spec. Actual implementation uses `authService.ts` and `taskService.ts` (see Section 2).

#### Old Pattern (for reference):
#### `createProfile(profile: UserProfile)` → Now `authService.saveProfile()`
- Inserts new row into `profiles`.
- Handles "on conflict" (upsert) if user already exists.

#### `getWeeklyTasks(userId: string, weekNumber: number)` → Now `taskService.loadUserTasks()`
- Selects tasks where `user_id = userId` AND `week_number = weekNumber`.
- Returns `Task[]`.

#### `updateTaskStatus(taskId: string, status: TaskStatus)` → **Not yet implemented**
- Updates `status` column.
- If `status === 'completed'`, sets `updated_at`.

#### `getHistory(userId: string)` → **Not yet implemented**
- Returns:
  - `completed_ids`: IDs of tasks with status 'completed'
  - `snoozed_ids`: IDs of tasks with status 'snoozed'
  - `deleted_ids`: (If we implement soft delete, otherwise ignored)

---

## 4. Weekly Refresh Logic (Edge Function - Optional for MVP)

*For MVP, this logic can live in the frontend `useEffect`.*

**Logic:**
1. Check `tasks` table for current week's tasks.
2. If tasks exist → Return them.
3. If NO tasks exist → Trigger `geminiService.generateWeeklyPlan`.
4. Save generated tasks to `tasks` table.
5. Return new tasks.

---

## 5. Acceptance Criteria

### 5.1 Schema
- [x] Tables created in Supabase.
- [x] RLS policies enabled and tested (user A cannot see user B's data).
- [x] JSONB columns accept correct structure.

### 5.2 Service (PR #75 Status)
- [x] Profile save/load implemented (`authService.ts`)
- [x] Task generation implemented (`taskService.generatePersonalizedTasks()`)
- [x] Task save/load functions exist (`taskService.saveTasks/loadUserTasks()`)
- [ ] **Persistence wired:** `saveTasks()` called after generation
- [ ] **Task updates:** Complete/snooze/delete functions implemented
- [ ] **Points/streak:** Update logic implemented
