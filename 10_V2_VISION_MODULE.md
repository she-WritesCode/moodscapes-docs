# V2 Vision (Board & Moods) Module Specification

## Overview

The Vision tab is the dedicated space for "Dreaming." It handles the aesthetic and creative aspects of wedding planning, serving as a repository for inspirational images (Pins) and creative text brainstorming (Ideas).

---

## 1. The Inspiration Engine (For Stakeholders)

### 1.1 The Pinterest Experience

Wedding planning is highly visual. The Vision tab provides a beautiful, edge-to-edge masonry grid of saved images. Users can browse global inspiration feeds or upload their own photos directly from their camera roll.

### 1.2 Sticky Filtering

Because inspiration can become overwhelming quickly, the board is easily filterable. Horizontally scrolling sticky pills (`All`, `Ceremony`, `Reception`, `Florals`, `Attire`) sit at the top of the screen. Tapping a pill instantly filters the grid to only show relevant images.

### 1.3 Turning Inspiration Into Action (Coordinator Handoff)

Inspiration is only useful if it leads to execution.
When a user taps an inspiration image to view it full-screen, they see options to turn that dream into reality:

- **Share with Team:** Instantly tag the hired Coordinator or Partner so they see the exact vibe the couple wants.
- **Create Task:** Turn the pin into an actionable to-do item (e.g., "Find a florist who can do this") and assign it directly to a Coordinator to execute, bridging the gap between dreaming and delegating.

### 1.4 Task Portal Integration

If a weekly task is to "Gather floral inspiration," tapping the task slides up a specialized mini-version of the Vision tab, pre-filtered to the "Florals" category. Once the user adds 3 images and closes the sheet, the task checks off automatically.

---

## 2. Technical Implementations & Schemas (For Developers)

The Vision module relies on a simple tagging structure and an interconnected UX flow to the Vendor Directory.

### 2.1 Entity Relationships

1. **`inspiration_pins`**: The core schema containing `image_url` and `category`. All records are tied to a `workspace_id` to support multi-event filtering.

### 2.2 Component Architecture

- **Global Vision Tab:**
  Queries `inspiration_pins` where `workspace_id = ?`. Leverages a Masonry layout library for optimized image loading.
- **Task Portal Component (`<VisionBoard sheetMode={true} />`):**
  When rendered inside a task, it receives a `category` prop (e.g., "florals").
  **Data Flow:**
  1. User uploads/saves an image.
  2. Component `POST`s to `inspiration_pins`.
  3. Component dispatches an `INSPIRATION_SAVED` event.
  4. The parent Task Sheet intercepts this and appends the new `pinId` to the subtask's `linkedObjects.inspirationPinIds` array.

- **Creating Tasks from Pins:**
  The Pin Detail Modal includes an "Add to Task" action. This triggers a standard task creation flow, but pre-populates the new task's `linkedObjects.inspirationPinIds` array with the current pin's ID, ensuring context isn't lost when delegating the work.

---

## 3. Success Metrics & KPIs (How we know it's working)

To validate the ROI of the "Dreaming into Doing" workflow, we will check:

- **Pin Delegation Rate:** The percentage of saved Inspiration Pins that are eventually linked to a Task or explicitly shared/assigned to a Collaborator. (Shows if inspiration is turning into execution).
- **Average Pins Per User:** A leading indicator of user stickiness and app investment.
- **Tag Filtering Usage:** The percentage of users who interact with the horizontal filter pills once their board exceeds X pins.
