# Spec: AI & Algorithm Integration

**Scope: Edge Function (GenAI) + Prompt Engineering**

> ⚠️ **ARCHITECTURAL REALITY:** This project uses an **AI-First** approach via Supabase Edge Functions (`generate-tasks`). It does **not** use client-side filtering of a hardcoded database.

---

## 1. Task Generation Engine

**Source Code:** `supabase/functions/generate-tasks/index.ts`
**Model:** Gemini 2.5 Flash

### 1.1 The Prompt Logic

The Edge Function constructs a prompt dynamically based on 3 Key Personalization Vectors:

#### A. Urgency Tone

* **> 365 Days:** Relaxed ("You have plenty of time...")
* **180-365 Days:** Moderate ("Now's a great time...")
* **< 180 Days:** Focused ("Let's prioritize...")

#### B. Budget Tone

* **Under $30k:** Budget-conscious (DIY, save money)
* **$70k+:** Premium (Full-service, splurge-worthy)
* **Middle:** Balanced (Value & Quality)

#### C. Priority Ordering

* The User's **#1 Priority** (e.g., "Food", "Photos") strictly determines the **Category of Task #1** in Week 1.
* *Mapping:*
  * `photos` -> `creative`
  * `food`/`party` -> `social`
  * `stress` -> `admin`
  * `budget` -> `finance`

### 1.2 Output Schema

The AI generates a JSON object with 3 sections:

1. **Week 1:** 4 "Baby Step" tasks (Easy wins)
2. **Week 2:** 4 "Next Step" tasks (Deepen progress)
3. **Future:** 4 High-level milestones

**Task Structure:**

```json
{
  "title": "Book Venue",
  "description": "Visit 3 top choices...",
  "whyNow": "Venues book 12 months out...",
  "category": "admin",
  "subtasks": ["Check availability", "Compare packages", "Sign contract"]
}
```

---

## 2. Safety & Fallbacks

### 2.1 Retry Logic

* The function retries Gemini API calls up to **3 times** on failure (429 errors).
* It utilizes a JSON repair mechanism (`JSON.parse` fallback to regex extraction) to handle malformed AI responses.

### 2.2 Privacy

* Only strict onboarding data (Date, Guest Count, Budget, Vibes, Names) is sent.
* No personal emails or identifiers are passed to the prompt.
