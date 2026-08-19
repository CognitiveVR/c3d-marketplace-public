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

---

## Example 4: VR puzzle adventure

### Project readback

- The experience is a chapter-based puzzle game with a tutorial and repeatable mechanics.
- The users are consumers on their own headsets.
- The team wants to fix early drop-off, find progression blockers, and learn which mechanics players actually use.
- **Motion:** Progression and retention
- **Archetype:** Progression loop
- **Overlays:** Live tuning

### Phase 1

Priorities:
- `session_started`, `session_ended` with `chapter_reached` and outcome
- `tutorial_step_started`, `tutorial_step_completed` with durations
- Dynamic objects: progression-gating portals, the core tool/weapon
- Session properties: `movement_style`, `input_mode`, `development_mode`
- Exit poll hooks at session end (dormant until playtests)

### Phase 2

Add:
- `mission_started`, `mission_ended` with `is_completed`, `duration_seconds`
- Recurring mechanic events with properties (not one event per mechanic variant)
- `setting_changed` for non-default comfort/movement settings
- Platform identity (OculusSocial / Steam ID) for review attribution
- Objective: FTUE completion; objective: first mission completed

### Phase 3

Add:
- `ui_opened` / `ui_closed` on key menus
- Remote controls for difficulty/balance variants with a `variant` session property
- Milestone feedback survey for playtest cohorts

### Why this plan works

This plan makes it possible to:
- see exactly which tutorial step loses players, and how long each step takes
- separate "players quit the game" from "players got stuck at a chapter"
- measure mechanic adoption as recurring behavior, not one-time milestones
- compare outcomes across movement styles and, later, balance variants

---

## Example 5: Academic experiment

### Project readback

- The experience is a controlled study comparing two environment conditions across repeated trials.
- Participants are recruited subjects in a lab on shared devices.
- The team needs trial-level data, recoverable condition assignment, and protocol-deviation visibility.
- **Motion:** Structured research and experimentation
- **Archetype:** Trial or condition loop
- **Overlays:** Shared devices, cohort tags, privacy-sensitive capture

### Phase 1

Priorities:
- `study_started` with `study_id`, `protocol_version`
- `condition_assigned` with `condition_id`, `assignment_method`
- `trial_started`, `trial_ended` with `trial_order`, `response_correct`, `response_time_seconds`
- Participant ID set at session start (lab devices are shared)
- Session properties: `condition_id`, `study_site`, `development_mode`
- Dynamic objects: stimuli and fixation targets only
- Exit poll hook at debrief

### Phase 2

Add:
- `block_started` / `block_ended` for block-order analysis
- `protocol_exception` with `exception_type`
- `questionnaire_completed` to separate protocol steps from XR activity
- Participant properties: cohort, counterbalancing group
- Objective: completed all required trials

### Phase 3

Add:
- Session tags for pilot vs main study, site comparisons
- Deeper workload/comfort survey instruments
- Export to downstream statistical analysis

### Why this plan works

This plan makes it possible to:
- recover every participant's condition and trial order after the fact
- analyze responses at trial and block level, not just study level
- spot protocol deviations and dropouts instead of silently losing them
- keep research governance data separate from behavioral analytics
