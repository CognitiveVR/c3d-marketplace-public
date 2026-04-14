# Example Track Plans

These examples are intentionally generic.

They are not meant to be copied verbatim. They are meant to show what a good answer looks like after discovery. Adapt the names, events, and properties to the app's real domain language.

---

## Example 1: Fire safety simulation

### Project readback

- The experience is a training simulation.
- The users are trainees on shared devices.
- The team needs to prove completion, identify where people fail, and compare outcomes by site and role.
- **Motion:** Performance and compliance
- **Archetype:** Procedural assessment
- **Overlays:** Shared device, LMS

### Phase 1

Priorities:
- `module_started`, `module_ended`
- `ftue_stage_started`, `ftue_stage_completed`
- Dynamic objects: extinguisher, breaker panel, instruction board
- Session properties: `app_mode`, `development_mode`
- Participant ID
- Exit poll hooks at start and end

### Phase 2

Add:
- `step_started`, `step_completed`, `step_skipped`, `incorrect_action`
- Session properties: `accuracy_percent`, `number_of_steps_completed`
- Participant properties: `role`, `site_id`
- Module completion objective
- Sequential procedure objective

### Phase 3

Add:
- `assistance_used`
- `movement_style_changed`, if relevant
- Targeted post-module confidence survey

### Why this plan works

This plan makes it possible to:
- see where trainees stall
- replay failed sessions with object context
- compare completion by role or site
- send completion logic downstream through objectives

---

## Example 2: Meditation and wellness library

### Project readback

- The experience is a content library with guided sessions.
- Users choose meditation or movement content and may repeat, pause, or bookmark it.
- The team wants to know which content gets completed, repeated, or abandoned.
- **Motion:** Content engagement and habit formation
- **Archetype:** Session-based content loop
- **Overlays:** Content catalog, pause and bookmark

### Phase 1

Priorities:
- `content_started`, `content_ended` with content metadata
- FTUE by content type
- Dynamic objects for key player UI surfaces only if gaze matters
- Session properties: `input_mode`, `development_mode`
- Exit poll hooks at end of content

### Phase 2

Add:
- `content_paused`, `content_bookmarked`, `content_selected`
- `ui_opened`, `ui_closed`
- Participant properties only if there is a meaningful user segment or tester cohort
- Objective: first completed content unit
- Objective: two completed units in one session

### Phase 3

Add:
- Recommendation or curation variant tracking
- Content-surface interaction detail
- Satisfaction or recommendation survey after selected sessions

### Why this plan works

This plan separates:
- browsing from actual consumption
- completion from abandonment
- content preference from habit formation

It also gives the team enough metadata to compare content by type, instructor, program, or format.

---

## Example 3: Prototype comparison study

### Project readback

- The experience is a product evaluation flow.
- Participants inspect and compare several prototypes.
- The team needs to understand interaction order, time spent, ratings, and preference.
- **Motion:** Exploration and evaluation
- **Archetype:** Exploration or object-interaction loop
- **Overlays:** Survey, cohort tags

### Phase 1

Priorities:
- `item_interaction_started`, `item_interaction_ended`
- Dynamic objects for each prototype
- Exit poll hook at end
- Session properties: `condition`, `study_batch`

### Phase 2

Add:
- `component_interaction_started`, `component_interaction_ended`
- `rating_submitted`, `choice_selected`
- Participant properties for approved demographic or experience bands
- Objective: completed comparison flow

### Phase 3

Add:
- Cohort tags for pilot versus main study
- Deeper UI instrumentation if people inspect labels or detail panels
- Guided assistant interaction events, only if the study actually includes that feature

### Why this plan works

This plan makes it possible to compare:
- which item was handled first
- how long people spent with each item
- whether component attention predicts rating
- whether one condition outperformed another
