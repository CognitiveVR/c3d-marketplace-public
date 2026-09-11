# Example Track Plans

These examples are intentionally generic.

They are not meant to be copied verbatim. They are meant to show what a good answer looks like after discovery. Adapt the names, events, and properties to the app's real domain language.

For brevity, Examples 1 to 5 omit the **Target SDK** line that a real plan must carry (see `track_plan_template.md` section 1, which requires the SDK and, on Unreal, the Blueprint-versus-C++ authoring surface). Example 6 shows it in place.

Examples 1 to 5 are written for a fully-featured target and assume dynamic objects, exit polls and unconstrained event properties are all available, so they map cleanly onto Unity and Unreal and need screening anywhere else. They are not automatically safe elsewhere. Example 4 leans on remote controls and store-platform identity (store identity is Unity-only, remote controls are undocumented on Android XR and WebXR); every example places exit poll hooks in Phase 1, which needs confirming on Android XR; and Example 5 uses session tags, which are not documented there either. Screen any example against `sdk_capability_matrix.md` before reusing its shape. Example 6 shows what a plan looks like once a capability limit is taken seriously rather than worked around.

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


---

## Example 6: Browser-based product configurator (WebXR, PlayCanvas)

Included to show how a capability limit changes a plan rather than merely shrinking it.

### Project readback

- The experience is a WebXR product configurator embedded in a retail site: customers open it from a product page, enter immersive mode on a headset or view it on desktop, and compare finishes and configurations.
- The users are prospective consumers, anonymous, mostly single-session.
- The team needs to know which configurations get explored, which get abandoned, and whether the experience shortens or lengthens the path to a quote request.
- **Motion:** Exploration and evaluation
- **Archetype:** Exploration or object-interaction loop
- **Overlays:** Content catalog, multi-SDK delivery (a Unity showroom build exists for trade shows)
- **Target SDK:** WebXR on PlayCanvas

### What the SDK constrains

PlayCanvas supports the core API, WebXR gaze, performance sensors, custom events, sensors, exit polls and properties, but **not dynamic objects, object export or per-object heatmaps**. The obvious plan for an exploration-archetype project leans heavily on dynamic objects, so that part is unavailable and the plan has to answer the same questions differently.

The substitute is custom events carrying an object identifier property. Say plainly what is lost: you will know which components the customer interacted with and in what order, but not what they looked at without touching, and not how long attention rested on each. If dwell-before-action is a question the team genuinely needs answered, the honest recommendation is to move the experience to the Three.js adapter, not to fudge it.

### Phase 1

Priorities:
- `configurator_started`, `configurator_ended` with `duration_seconds`, `exit_reason`
- `ftue_stage_started`, `ftue_stage_completed` with `stage_name`, `duration_seconds`
- `component_inspected` with `component_id`, `component_name`, `interaction_order` — the dynamic object substitute
- Session properties: `product_line`, `entry_point`, `development_mode`, `session_device_class`
- `development_mode` set from the build environment, non-negotiable: there is no editor exclusion on WebXR
- Exit poll hook at the end, **plus the in-scene survey UI**, scoped as its own task
- Scene uploaded through the Upload Web App, with `sceneId` and `versionNumber` wired into `allSceneData`

### Phase 2

Add:
- `configuration_changed` with `option_category`, `option_id`, `previous_option_id`
- `configuration_saved`, `quote_requested` with `configuration_hash`
- `comparison_opened` with `option_ids`, `comparison_duration_seconds`
- Participant ID from the site's existing account system where the customer is logged in; anonymous otherwise, and say so rather than inventing a browser-local identifier
- Objective: reached a saved configuration
- Sensor: `active_options_count`, sampled on change

### Phase 3

Add:
- `ui_opened` / `ui_selected` for the option panels
- Variant tracking for configurator layouts, with the condition assigned by the site's own feature-flag service and recorded as a session property, since remote controls are not documented for WebXR
- Deeper post-session preference survey

### Why this plan works

This plan makes it possible to:
- see which configurations are explored and which are abandoned, without object-level gaze
- connect configurator behavior to quote requests
- compare entry points and device classes, both free from the platform's built-in fields
- keep local development traffic out of the dashboard from day one

And it keeps the Unity showroom build comparable: `component_id`, `configuration_hash` and the event names are identical across both, so the two builds query as one series instead of two.
