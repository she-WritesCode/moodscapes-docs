# Phase 0: The Shared Contract (The Constitution)

**Immutable Source of Truth for All Agents**

> ⚠️ **CRITICAL INSTRUCTION FOR ALL AGENTS:**
> This document defines the **Shared Types, Schemas, and Contracts**.
> You **MUST** adhere to these definitions.
> You **MAY NOT** modify these definitions without explicit approval.
> If your code contradicts this contract, your code is wrong.

---

## 1. Core Domain Types (TypeScript)

### 1.1 User Profile

```typescript
export type BudgetTier = 
  | 'under_30k' 
  | '30k_50k' 
  | '50k_70k' 
  | '70k_plus' 
  | 'figuring_it_out';

export type Priority = 
  | 'photos'          // Amazing Photos
  | 'food'            // Great Food & Drinks
  | 'party'           // Fun, Memorable Party
  | 'stress'          // Low-Stress Planning
  | 'budget'          // Stay on Budget
  | 'personal';       // Personal & Meaningful

export interface UserProfile {
  id: string;
  names: string;          // "Alex & Jordan" (Display format)
                          // Note: Onboarding collects userName + partnerName separately,
                          // then combines as "userName & partnerName" before saving
  weddingDate: string | null;   // ISO Date "2026-06-15" or null
  budgetTier: BudgetTier;
  priorities: Priority[];
  location?: string;      // Collected post-onboarding
  guestCount?: number;   // Collected post-onboarding
  createdAt: string;
}


```

### 1.2 Task Structure

```typescript
export type TaskStatus = 'todo' | 'completed' | 'snoozed';

export type TaskCategory = 'admin' | 'creative' | 'finance' | 'social';

export interface Subtask {
  id: string;
  text: string;
  isCompleted: boolean;  // Note: camelCase to match aistudio-wedapp
}

export interface Task {
  id: string;
  title: string;          // Max 5 words
  description: string;    // Max 15 words
  category: TaskCategory;
  emoji: string;          // Single emoji character
  subtasks: Subtask[];    // Max 3 items
  isCompleted: boolean;   // Note: camelCase to match aistudio-wedapp
  deleted: boolean;       // Soft delete flag
  snoozedUntil?: string;  // ISO timestamp, null if not snoozed
  aiHelpPrompt?: string;  // Optional: Prompt for AI assistance
}

export interface WeeklyPlan {
  weekNumber: number;
  theme: string;          // e.g., "The Foundation", "Vibe Check"
  motivationalMessage: string; // Max 15 words, no quotes
  tasks: Task[];          // Current week (exactly 4)
  upcomingTasks: Task[];  // Next week preview (exactly 4)
}
```

> **IMPORTANT:** The types above match the actual aistudio-wedapp implementation. Use these EXACT field names (camelCase, not snake_case).

### 1.3 Vendor Prioritization

```typescript
export interface VendorScore {
  vendorType: string;    // "photographer"
  score: number;          // 0-100
  reason: string;         // "Books out fastest for peak season"
  urgencyScore: number;
  priorityBonus: number;
}
```

---

## 2. Database Schema (Supabase)

> **Note:** Database columns use `snake_case` (SQL convention), but TypeScript interfaces use `camelCase`. The application layer handles the mapping between these conventions.

### 2.1 `profiles` Table

```sql
create table profiles (
  id uuid references auth.users not null primary key,
  names text not null,
  wedding_date date,
  budget_tier text check (budget_tier in ('under_30k', '30k_50k', '50k_70k', '70k_plus', 'figuring_it_out')),
  priorities jsonb default '[]'::jsonb, -- Array of strings
  location text,
  guest_count integer,
  created_at timestamptz default now()
);
```

### 2.2 `tasks` Table

```sql
create table tasks (
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
  deleted boolean default false,
  snoozed_until timestamptz,
  created_at timestamptz default now()
);
```

### 2.3 `reference_tasks` Table (Read-Only)

*Used for Few-Shot Prompting (Style Transfer)*

```sql
create table reference_tasks (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  description text not null,
  subtasks jsonb not null,
  category text,
  timeline_months_out integer -- Used to select relevant examples
);
```

---

## 3. API Contracts (Gemini Service)

### 3.1 Input (Prompt Context)

The AI Agent must accept this structure to generate the prompt:

```typescript
interface GenerationContext {
  profile: UserProfile;
  current_week: number;
  completed_task_ids: string[];
  snoozed_task_ids: string[];
  top_vendors: VendorScore[]; // From prioritization service
  reference_examples: Task[]; // 15 examples from DB
}
```

### 3.2 Output (JSON Schema)

The AI Agent must enforce this exact JSON schema:

```json
{
  "type": "object",
  "properties": {
    "weekNumber": { "type": "integer" },
    "theme": { "type": "string" },
    "tasks": {
      "type": "array",
      "maxItems": 4,
      "items": {
        "type": "object",
        "properties": {
          "title": { "type": "string", "description": "Max 5 words" },
          "description": { "type": "string", "description": "Max 15 words" },
          "motivationalMessage": { "type": "string", "description": "Max 10 words, no quotes" },
          "category": { "type": "string" },
          "vendorType": { "type": "string", "nullable": true },
          "subtasks": {
            "type": "array",
            "maxItems": 3,
            "items": { "type": "string", "description": "Max 10 words" }
          }
        },
        "required": ["title", "description", "subtasks", "category"]
      }
    }
  },
  "required": ["weekNumber", "tasks"]
}
```

---

## 4. Shared Constants

### 4.1 Priorities List

```typescript
export const PRIORITIES = [
  { id: 'photos', label: '📸 Amazing Photos' },
  { id: 'food', label: '🥘 Great Food & Drinks' },
  { id: 'party', label: '🎉 Fun, Memorable Party' },
  { id: 'stress', label: '🧘 Low-Stress Planning' },
  { id: 'budget', label: '💰 Stay on Budget' },
  { id: 'personal', label: '💕 Personal & Meaningful' }
];
```

### 4.2 Budget Options

```typescript
export const BUDGET_OPTIONS = [
  { id: 'under_30k', label: 'Under $30k' },
  { id: '30k_50k', label: '$30k - $50k' },
  { id: '50k_70k', label: '$50k - $70k' },
  { id: '70k_plus', label: '$70k+' },
  { id: 'figuring_it_out', label: '💭 Still figuring it out' }
];

### 4.3 Gamification Badges
> **MVP NOTE:** The entire Badge/Trophy system is **deprioritized for MVP**. Badge types are defined here for reference, but unlocking logic and UI are **Post-MVP**.

```typescript

```

---

## 5. File Structure (Standardized)

```
src/
├── components/
│   ├── onboarding/       # UI Agent
│   ├── shared/           # UI Agent (Pills, Buttons)
│   └── dashboard/        # UI Agent (Task Cards)
├── services/
│   ├── geminiService.ts  # AI Agent
│   ├── vendorPrioritization.ts # AI Agent
│   └── supabase.ts       # Data Agent
├── types/
│   └── index.ts          # SHARED CONTRACT (This file)
└── pages/
    └── Onboarding.tsx    # UI Agent
```
