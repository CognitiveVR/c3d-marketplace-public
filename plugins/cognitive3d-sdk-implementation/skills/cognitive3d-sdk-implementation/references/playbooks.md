# Cognitive3D Tracking Playbooks

Use this file **after** discovery and **after** reading `data_strategy.md`.

Do not paste the whole file into the answer. Pull only the playbook or overlay sections that fit the project.

**This file is SDK-neutral.** The archetypes describe the shape of an experience, not the engine it runs on. Several recommendations here are unavailable on some SDKs and frameworks, so screen the finished plan against `sdk_capability_matrix.md` before presenting it. The ones that bite most often: **dynamic objects** (Unity, Unreal and Android XR, and WebXR only on Three.js and Mattercraft), **remote controls** and **multiplayer components** (Unity and Unreal; not documented for Android XR or WebXR), **ExitPoll** (not documented for Android XR), **audio recording** and **store-platform identity** such as Oculus Social or Steam (Unity only). Two further checks this file cannot make for you: on **Unreal**, whether the comfort, framerate and boundary metrics a playbook assumes are backed by built-in components the team actually added, and whether numeric properties come from C++ rather than Blueprint; on **Android XR**, whether any event in the plan exceeds the ten-property cap.

## How to use this file

1. Choose the best-fit archetype from the classification in SKILL.md.
2. Pull the matching event families, properties, dynamic objects, and dashboard ideas.
3. Adapt the names to the app's real domain language.
4. Keep only the recommendations that answer the developer's actual questions.
5. Use `track_plan_template.md` to turn the result into a clean recommendation.

---

## Playbook 1: Performance and assessment

### Use this when

The project is trying to measure completion, competency, accuracy, compliance, or process adherence.

Common fits: training simulation, SOP walkthrough, onboarding with formal pass/fail, education or certification module, task-based assessment.

### Primary questions

- Did participants complete the procedure?
- Which step took too long?
- Which steps were skipped?
- Which incorrect actions are most common?
- Which cohorts need more support?
- What should be sent to LMS or downstream reporting?

### Recommended event families

| Event family | When it fires | Core properties | Why it matters |
| --- | --- | --- | --- |
| `module_started` | module begins | module_id, module_name, module_type, attempt_count | top-level activity |
| `module_ended` | module ends | module_id, duration_seconds, completion_status, score_percent | completion/outcome |
| `step_started` | step begins | step_id, step_name, step_order | timing/sequence |
| `step_completed` | step succeeds | step_id, step_name, step_order, duration_seconds, success_boolean | progress/friction |
| `step_skipped` | step skipped | step_id, step_name, step_order, skip_reason | non-completion |
| `incorrect_action` | critical mistake | error_type, step_name, severity | failure analysis |
| `assistance_used` | help requested | step_name, help_type, count | supported vs unsupported success |

### Recommended dynamic objects

Use for: critical tools/equipment, safety objects, instruction panels, control surfaces, target objects, objects in objectives. Only include objects where attention or interaction changes interpretation. Remember: registering the object in the app is not enough — meshes must be exported and uploaded separately from the scene, on every SDK (see the SDK reference for the project's target).

### Recommended session properties

Common examples: `app_mode` (practice or test), `scenario_type`, `development_mode`, `device_choice`, `input_mode`, `movement_style`, `number_of_steps_completed`, `accuracy_percent`

### Recommended participant properties

Common examples: `role`, `department`, `site_id`, `training_level`, `cohort`, `years_experience`

Use participant properties when the value should follow the person across sessions.

### Objectives and analysis setup

Good first objectives: module completion, sequential step-completion, safety-critical gaze, error-free completion, exit poll.

Strong dashboard reads: completion rate by module, average time per step, most common incorrect action, completion rate by role/site/cohort, replay of failed sessions.

### Exit polls

Useful prompts: confidence before/after, perceived success, usefulness, open feedback for unclear steps. Place hooks at beginning and end at minimum.

### Common overlays

Usually relevant: shared device or SSO, LMS or xAPI export, dev vs prod separation, privacy-sensitive audio only when clearly justified.

### Integration notes

These are high-leverage observations from real integrations. See `field_notes.md` for full details.

- **Shared devices need participant ID at session start.** Device ID alone makes every trainee from the same headset look like the same person. See field note: _Shared devices: device ID is not enough_.
- **Design completion objectives with LMS forwarding in mind.** If the team mentions LMS or xAPI, the module completion event and objective need score, attempt, and status metadata from day one. See field note: _LMS and objective forwarding_.
- **Audio recording is high value but high sensitivity.** Only when verbal output is relevant to assessment. Always flag consent and governance. See field note: _Audio recording: high value, high sensitivity_.
- **Track planned versus actual duration.** For timed modules, recording both expected and actual duration reveals partial completions. See field note: _Content duration: planned versus actual_.

### Common mistakes

- Only tracking final completion, nothing step-level
- Putting role or department on session instead of participant
- Forgetting incorrect-action events
- Dynamic objects on every prop instead of safety-critical objects
- Recommending audio capture without explicit consent or governance

---

## Playbook 2: Progression and mechanics

### Use this when

The project needs to understand onboarding, progression, mechanics, balance, level flow, or retention.

Common fits: games, narrative adventures, puzzle apps, progression-based interactive experiences, repeat-play loops.

### Primary questions

- Where does FTUE break?
- Which chapter, mission, or level causes drop-off?
- Which mechanics are actually being used?
- Are players finding key objectives or portals?
- Which settings or movement styles affect success?
- What should the team tune next?

### Recommended event families

| Event family | When it fires | Core properties | Why it matters |
| --- | --- | --- | --- |
| `session_started` | run begins | mode, difficulty, variant, movement_style | play context |
| `session_ended` | run ends | duration_seconds, outcome, score_value, chapter_reached | retention/completion |
| `tutorial_step_started` | tutorial step starts | step_name, step_order | early friction |
| `tutorial_step_completed` | tutorial step completes | step_name, step_order, duration_seconds | onboarding pacing |
| `mission_started` | mission begins | mission_id, mission_name, difficulty | progression |
| `mission_ended` | mission ends | mission_id, duration_seconds, is_completed, score_value | progression analysis |
| mechanic-specific events | mechanic used | mechanic_name or stable action with weapon, spell, target, result | system adoption |
| `achievement_unlocked` | milestone reached | achievement_name, chapter_name, time_in_game_seconds | cross-session progression |
| `setting_changed` | non-default setting | setting_name, setting_type, value_set | comfort/success patterns |
| `ui_opened` / `ui_closed` | key interfaces opened | ui_name, duration_seconds, context | confusion and repeated menu use |

### Recommended dynamic objects

Good candidates: portals and wayfinding, core weapons or tools, tutorial targets, instruction surfaces, HUD/maps if attention matters, interactable objects that gate progression. Remember: registering the object in the app is not enough — meshes must be exported and uploaded separately from the scene, on every SDK (see the SDK reference for the project's target).

### Recommended session properties

Common examples: `movement_style`, `input_mode`, `chapter_reached`, `variant`, `difficulty`, `device_choice`, `development_mode`

### Recommended participant properties

Use only when behavior should carry across sessions: lifetime progression, leaderboard score, total kills or wins, meta progression or skill-tree state if stable enough.

### Objectives and analysis setup

Good first objectives: FTUE completion, mission completion, required mechanic used at least once, portal or progression gate engaged.

Strong dashboard reads: drop-off by FTUE step, chapter completion by movement style, mechanic usage before success or failure, attention to key objects before progression, replay of stalled sessions.

### Exit polls

Usually not the first priority for live games. Best fits: beta testing, controlled playtests, milestone feedback.

### Common overlays

Usually relevant: live tuning or A/B testing, platform identity (Steam or Meta), mixed reality or device-mode differences, multiplayer if present.

### Integration notes

- **The two-minute window is real.** FTUE duration per stage — not just completion — reveals whether onboarding reaches the spatial payoff fast enough. Strong A/B testing candidate. See field note: _First-time user experience and the two-minute window_.
- **Track recurring mechanics, not just milestones.** If the question is "which mechanic are players actually using?", milestones alone cannot answer it. See field note: _Recurring behavior versus milestones_.
- **Platform identity is a low-effort win for consumer apps.** OculusSocial on Quest and Steam ID on Steam enable review attribution and store-level correlation. See field note: _Platform identity: Oculus Social and Steam_.
- **Flag remote controls early even if they ship in Phase 3.** Designing events for comparison across control states needs to happen before Phase 3. See field note: _Remote controls for live tuning_.

### Common mistakes

- Encoding level or difficulty into event name
- Tracking only one-time achievements, not recurring mechanics
- Not tracking movement or control choices when they affect outcome
- Treating every UI click as equally important
- Forgetting FTUE durations, only tracking completion

---

## Playbook 3: Content, wellness, and practice loops

### Use this when

The project is built around repeatable units of content and the team needs to understand choice, completion, repeat use, or habit formation.

Common fits: fitness apps, meditation apps, music practice or instrument trainers, content libraries, reading or chapter-based experiences, instructor-led sessions.

### Primary questions

- Which content gets selected most often?
- Which content gets completed?
- Which content gets repeated?
- Where do users pause, abandon, or bookmark?
- Which teacher, technique, or content type performs best?
- What content should the team surface more often?

### Recommended event families

| Event family | When it fires | Core properties | Why it matters |
| --- | --- | --- | --- |
| `content_started` | content begins | content_id, content_name, content_category, difficulty, instructor, program, planned_length_seconds, is_mr | content unit |
| `content_ended` | content ends | content_id, duration_seconds, completion_rate, end_reason | completion/abandonment |
| `content_paused` | playback pauses | content_id, pause_duration_seconds | true consumption time |
| `content_bookmarked` | user saves | content_id, content_category | preference |
| `content_selected` | user chooses | content_id, selection_surface, recommendation_source | browsing vs consumption |
| FTUE stages | onboarding | content_mode, stage_name, duration_seconds | onboarding by content type |
| `ui_opened` / `ui_closed` | tablets, watches, players opened | ui_name, duration_seconds, context_surface | friction and support-surface use |

### Recommended dynamic objects

Good candidates: instructors or trainer objects, helper objects, musical instruments or practice tools, key player UI surfaces, navigation tablets or watches, content hotspots. Remember: registering the object in the app is not enough — meshes must be exported and uploaded separately from the scene, on every SDK (see the SDK reference for the project's target).

### Recommended session properties

Common examples: `content_mode`, `input_mode`, `device_choice`, `number_of_activities_completed`, `total_active_time_seconds`, `variant`, `development_mode`

### Recommended participant properties

Common examples: `tester_type`, `user_segment`, `cohort`, `preferred_mode` if stable, subscription or entitlement context only when approved and truly useful.

### Key insight

Track both **planned duration** (`content_duration_seconds`) and **actual duration** (`duration_seconds`). A user completing 4 of 30 minutes is very different from 28 of 30.

### Objectives and analysis setup

Good first objectives: completed first content unit, completed two or more units in one session, completed a key onboarding path, revisited content after first completion.

Strong dashboard reads: completion rate by content type, repeat selection by category/teacher/program, bookmark rate vs completion rate, pause behavior by content type, replay for early abandonment.

### Exit polls

Often valuable in this archetype. Useful prompts: satisfaction, perceived value, energy or confidence before/after, recommendation likelihood.

### Common overlays

Usually relevant: content catalog metadata, subscription or entitlement context, recommendation or curation experiments, input-mode differences.

### Integration notes

- **Track planned versus actual duration on every content event.** A user who completes 4 minutes of a 30-minute class is a very different signal from 28 minutes. See field note: _Content duration: planned versus actual_.
- **The two-minute window applies to content apps too.** If the first content experience is slow to load or confusing to navigate, users will leave. See field note: _First-time user experience and the two-minute window_.
- **Recurring behavior matters more than first completions.** Whether content gets repeated, bookmarked, or returned to is often more important than whether it was completed once. See field note: _Recurring behavior versus milestones_.
- **Platform identity enables content preference analysis at the user level.** See field note: _Platform identity: Oculus Social and Steam_.

### Common mistakes

- Tracking only content start, not end
- No content metadata on lifecycle events
- Ignoring pause, bookmark, or repeat behavior
- Not separating browsing from consumption

---

## Playbook 4: Exploration, evaluation, and consumer research

### Use this when

The goal is to understand how people explore, compare, inspect, interact with, or rate items in an experience.

Common fits: product evaluation, prototype comparison, virtual retail, showrooms, architecture and design review.

### Primary questions

- Which objects were handled?
- In what order?
- For how long?
- Which components drew attention?
- Which option did people prefer?
- What attributes correlate with preference or rating?

### Recommended event families

| Event family | When it fires | Core properties | Why it matters |
| --- | --- | --- | --- |
| `item_interaction_started` | handling begins | item_id, item_name, interaction_order, repeat_count | order/comparison |
| `item_interaction_ended` | interaction ends | item_id, duration_seconds | engagement depth |
| `component_interaction_started` | subcomponent used | item_id, component_id, component_name | isolate what matters |
| `component_interaction_ended` | ends | item_id, component_id, duration_seconds | component-level comparison |
| `rating_submitted` | participant rates | item_id, question_id, rating_value | pair behavior with preference |
| `choice_selected` | preference made | option_id, comparison_set, choice_reason | direct preference |
| `ui_opened` / `ui_closed` | labels, detail views opened | ui_name, duration_seconds | how people inspect info |

### Recommended dynamic objects

Especially high-value here. Good candidates: each item/prototype, major components, packaging/label surfaces, signage, comparison surfaces. Remember: registering the object in the app is not enough — meshes must be exported and uploaded separately from the scene, on every SDK (see the SDK reference for the project's target).

### Recommended session properties

Common examples: `condition`, `variant`, `layout`, `store_id`, `device_choice`, `development_mode`, `study_batch`

### Recommended participant properties

Common examples: age band, experience level, cohort, recruitment source, persona segment, category familiarity. Keep privacy-conscious.

### Session tags

This archetype often benefits from tags because analysts may want to compare: day one vs day two, on-site vs remote, pilot vs main study, recruited segment A vs B.

### Objectives and analysis setup

Good first objectives: interacted with every required item, looked at required component before rating, completed comparison flow, completed all ratings.

Strong dashboard reads: average interaction duration by item, rating by interaction order, gaze on components before preference choice, replay of high-rating vs low-rating sessions.

### Exit polls

Often essential here. Useful prompts: category experience, prior familiarity, overall preference, why a choice was made, qualitative feedback.

### Common overlays

Usually relevant: cohort tags, shared-device study stations, survey-heavy flow, AI guide if present, privacy-sensitive demographic capture.

### Integration notes

- **Shared-device study stations need explicit participant ID.** Lab and kiosk setups are shared-device environments. See field note: _Shared devices: device ID is not enough_.
- **Dynamic objects are especially high-value here.** Each prototype as a dynamic object gives gaze and fixation data tied to the specific item. See field note: _Dynamic objects: quality over quantity_.
- **Exit polls are often essential, not optional.** Behavioral data and self-report together tell the story. Place hooks early. See field note: _Exit poll hooks are cheap on the engine SDKs, expensive on WebXR, and unconfirmed on Android XR_.

### Common mistakes

- Relying on surveys without behavioral tracking
- Forgetting interaction order
- Session-specific condition as participant property
- Only whole-item tracking when component-level matters
- Collecting sensitive participant data without clear need

---

## Playbook 5: Structured research and experimentation

### Use this when

The project is explicitly designed around conditions, trials, blocks, or repeated measures.

Common fits: academic studies, UX experiments, repeated-measure protocols, pilot programs, lab workflows.

### Primary questions

- Which condition performed best?
- Did participants complete required trials?
- Did order or block effects matter?
- Which cohort differed significantly?
- Were there protocol deviations or dropouts?

### Recommended event families

| Event family | When it fires | Core properties | Why it matters |
| --- | --- | --- | --- |
| `study_started` | protocol begins | study_id, protocol_version | research context |
| `condition_assigned` | condition set | condition_id, assignment_method | recoverable split |
| `block_started` / `block_ended` | block of trials | block_id, block_order, duration_seconds | block-level analysis |
| `trial_started` | trial begins | trial_id, trial_order, stimulus_id, condition_id | unit of work |
| `trial_ended` | trial ends | trial_id, response_correct, response_time_seconds | trial-level analysis |
| `response_submitted` | participant responds | response_type, response_value, trial_id | explicit result |
| `questionnaire_completed` | survey completed | instrument_id, completion_status | separates protocol from XR activity |
| `protocol_exception` | protocol broken | exception_type, block_id or trial_id | data quality |

### Recommended dynamic objects

Good candidates: stimuli, fixation targets, key interactive objects, instruction surfaces, response surfaces. Remember: registering the object in the app is not enough — meshes must be exported and uploaded separately from the scene, on every SDK (see the SDK reference for the project's target).

### Recommended session properties

Common examples: `condition_id`, `block_order`, `study_site`, `protocol_version`, `development_mode`

### Recommended participant properties

Common examples: `participant_id`, cohort/group, relevant prior experience.

### Objectives and analysis setup

Good first objectives: completed all required trials, followed required sequence, responded within timing bounds, completed questionnaire or debrief.

Strong dashboard reads: completion by condition, response time by block, dropout by protocol stage, replay of exceptions, comparison across cohorts or sites.

### Exit polls

Useful when design allows: workload, comfort, confidence, debrief responses.

Governance: consent, debrief, and protected study data follow the study's approved process. Do not treat analytics as a shortcut around research governance.

### Common overlays

Usually relevant: cohort assignment, counterbalancing, site tags, privacy-sensitive participant data, export to downstream analysis.

### Common mistakes

- Only recording final score, not trial-level detail
- Putting condition inside event names
- Forgetting protocol deviations or early exits
- Mixing research governance data casually into analytics
- Recommending identified storage when anonymous handling is required

---

## Overlay cards

Use these only when they clearly fit the project.

### Overlay: Shared devices and identified participants

**Use when:** multiple people share headsets or workstations, or longitudinal user-level tracking is needed.

**Add:** participant id, participant properties for stable person-level descriptors, session tags for analyst-driven cohorting, validation that participant data appears on the participant directory.

**Be careful with:** secrets in SSO flows, PII that is not actually needed, assuming device id is enough.

### Overlay: Multiplayer and collaboration

**Use when:** two or more people share a room, lobby, role, or coordinated task.

**Add:** session properties (`room_id`, `room_type`, `player_count`, `role`), events (`room_joined`, `room_left`, `ready_state_changed`, `collaboration_started/ended`, `other_user_joined/left`).

**Why it matters:** without room context, collaborative behavior is very hard to interpret.

### Overlay: Live tuning and experimentation

**Use when:** comparing variants, tuning balance, or running A/B tests.

**Add:** explicit `variant`/`condition` field, assignment to condition, comparable outcome events, remote controls.

**Why it matters:** if the condition is not recorded, the experiment did not happen.

### Overlay: Voice, AI, and conversational interaction

**Use when:** the app includes voice input, AI agents, guided conversations, or assistants.

**Start with:** `assistant_invoked`, `assistant_response_shown`, `assistant_action_taken`, `voice_interaction_started/ended`. Properties: `interaction_category`, `resolution_outcome`, `duration_seconds`, `turn_count`.

**Add transcript capture only if:** it answers a real question, consent and governance are clear, the project explicitly wants that detail.

### Overlay: Input mode and mixed reality

**Use when:** interpretation changes depending on device, mode, or input system.

**Add:** session properties (`input_mode`, `movement_style`, `device_choice`, `is_mr`), events (`input_mode_changed`, `movement_style_changed`).

**Why it matters:** a project may perform differently for hands vs controllers, VR vs WebGL, or MR vs VR.
