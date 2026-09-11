# Cognitive3D Data Strategy

_Stable strategy layer for deciding what to track and why._

Read this file after the main SKILL.md when you need deeper guidance on primitives, phasing, overlays, naming, or anti-patterns.

**This file is SDK-neutral.** Everything here describes what to track and why, and applies equally to Unity, Unreal and WebXR projects. It will occasionally suggest something a particular SDK or framework cannot do, so screen the finished plan against `sdk_capability_matrix.md` before presenting it, and route implementation detail to the SDK reference for the project's target.

## Table of contents

1. [Operating principle](#operating-principle)
2. [Cognitive3D mental model](#cognitive3d-mental-model)
3. [Choosing the right primitive](#choosing-the-right-primitive)
4. [Universal baseline instrumentation](#universal-baseline-instrumentation)
5. [Naming and property conventions](#naming-and-property-conventions)
6. [Phased implementation](#phased-implementation)
7. [Cross-cut overlays](#cross-cut-overlays)
8. [Business question map](#business-question-map)
9. [Anti-patterns](#anti-patterns)

---

## Operating principle

The goal is not to "use more Cognitive3D features."

The goal is to make the app measurable enough that a team can answer a real question and act on the answer.

A strong recommendation follows this chain:

1. **Business question**
   - Example: Where do learners fail?
   - Example: Which classes get repeated?
   - Example: Which prototype wins?
   - Example: Which tutorial step causes drop-off?

2. **Evidence needed**
   - sequence
   - duration
   - outcome
   - object attention
   - self-reported feedback
   - cohort or variant

3. **Right data primitive**
   - custom event
   - dynamic object
   - session property
   - participant property
   - objective
   - exit poll
   - tag

4. **Usable analysis surface**
   - replay
   - objective completion
   - session details
   - participant directory
   - dashboard query
   - cohort comparison
   - export or downstream reporting

If a recommendation does not clearly support a decision, it is probably noise.

---

## Cognitive3D mental model

Cognitive3D is strongest when instrumentation supports the platform's three layers of value.

| Product layer | Strategy question | Typical instrumentation |
| --- | --- | --- |
| Track XR Experiences | What happened in the app? | session lifecycle, custom events, scenes, session properties, exit poll hooks |
| Explore Individual Behaviour | What did one person do in context? | replay, dynamic objects, gaze, input-mode data, objectives |
| Analyze Aggregate Insights | What patterns matter across cohorts, variants, or builds? | participant properties, session properties, tags, objectives, dashboards, queries |

By default, Cognitive3D already captures core spatial, device, and session context. The main job of a data strategy is to add the **app-specific context** that makes that base data useful.

---

## Choosing the right primitive

A large percentage of bad instrumentation comes from using the wrong primitive.

| Need | Best primitive | Why |
| --- | --- | --- |
| Something happened at a moment in time | Custom event | Events are time-aligned on the session timeline and replay |
| A value describes the whole session | Session property | Best for filtering, grouping, and cohort comparison |
| A value describes the person across sessions | Participant property | Belongs on the participant profile, not inside every session |
| Need to know what object was seen, used, moved, or fixated | Dynamic object | Adds object-level context for replay, gaze, and objectives. **Not available on every SDK and framework** — check `sdk_capability_matrix.md` before planning one |
| Measure completion logic or sequences | Objective | Turns events, gaze, and survey responses into success logic. Created on the dashboard or programmatically via the MCP server with a write-enabled organization key |
| Self-reported feedback or cohort questions | Exit poll | Best for sentiment, preference, confidence, and study questions. Question sets are created on the dashboard or programmatically via the MCP server with a write-enabled organization key |
| Continuous value sampled over time | Sensor | Time-series charted on the session timeline; aggregates (avg/min/max) queryable. The SDK records many automatically (HMD orientation, controller ergonomics, performance, biometrics on supported hardware); custom sensors capture app-specific continuous values |
| Analyst-added grouping after the fact | Session tag | Flexible for cohorting and ad hoc study grouping |

### Event or property?

If the value changes over time during a session, it probably belongs in an event.

If the value is one stable descriptor for the session, it probably belongs in a session property.

If the value should follow the person across sessions, it probably belongs in a participant property.

### Objective or event?

Use an event when you need to record a building block.

Use an objective when you need to score or assess a sequence, threshold, completion, or condition.

Example:
- `step_completed` is an event
- "completed all required steps in order" is an objective

---

## Universal baseline instrumentation

These recommendations apply to almost every Cognitive3D project.

### 1. Primary activity lifecycle

Track the main activity with a start and end pair. At minimum, the pair should make it possible to answer: what started, when it ended, how long it lasted, what outcome it had.

Common properties: `activity_id`, `activity_name`, `activity_type`, `duration_seconds`, `outcome`, `score_percent`

### 2. Onboarding and first-time use

Track onboarding in stages, not as one vague blob. Duration per stage, not just completion, is what reveals whether onboarding is fast enough.

> **Field note:** See `field_notes.md` → _First-time user experience and the two-minute window_ for more.

Recommended pattern: `ftue_started`, `ftue_stage_started`, `ftue_stage_completed`, `ftue_completed`

Suggested properties: `stage_name`, `stage_order`, `duration_seconds`, `help_used`, `input_mode`

### 3. Key dynamic objects

Add dynamic objects only where object-level attention or interaction answers a question. Good candidates: equipment, tools, instruction surfaces, targets, prototypes, guide objects, objects used in objectives. Do not track everything — spawned objects like bullets pollute the object list.

**Important:** registering an object in the app is only half the job. Dynamic object meshes must also be exported and uploaded **separately from the scene** (Unity's Feature Builder, Unreal's Dynamic Object Manager, or the WebXR Upload Web App). Without that step, objects have no visual representation in dashboard replay. Scene upload and dynamic object mesh upload are two distinct workflows on every SDK, and this is the single most commonly missed step in an integration.

**Availability:** dynamic objects exist on Unity and Unreal, and on WebXR only for the Three.js and Mattercraft adapters. Where they are unavailable, substitute custom events carrying an object identifier property and state in the plan what that cannot answer: dwell before action, attention without interaction, and object heatmaps. See `sdk_capability_matrix.md`.

**Objects spawned at runtime** need an identity strategy: Unity and Unreal use ID Pools (Unreal's is an Id Pool Asset that must be sized to the concurrent spawn count); WebXR registers each instance explicitly at spawn time. Route the mechanics to the SDK reference.

> **Field note:** See `field_notes.md` → _Dynamic objects: quality over quantity_ for more.

### 4. Session context

Common examples: `app_mode`, `build_channel`, `development_mode`, `device_choice`, `movement_style`, `input_mode`, `scenario_type`, `variant`, `condition`

### 5. Participant identity, when it matters

Set when cross-session analysis matters: shared-device training, employee progression, consumer research cohorts, longitudinal studies. Do not force identity where it is not needed.

### 6. Dev versus production separation

For builds and deployments used during development, use a `development_mode` session property, session tag, or separate project. Without this, dashboards become noisy fast.

How much this matters depends on the SDK, and only Unity does any of it for you. Unity excludes in-editor sessions from major dashboard analytics automatically, so the property mainly covers sideloaded device builds. Unreal records editor sessions and shows them behind a dashboard toggle rather than filtering them out. WebXR has no equivalent concept at all: local development produces sessions indistinguishable from real ones. On Unreal and WebXR, explicit separation is mandatory rather than advisory. See the field note.

> **Field note:** See `field_notes.md` → _Dev/prod separation, and what the SDK does or does not do for you_ for more.

### 7. Input and environment verification

Record when these affect interpretation: hands vs controllers, VR vs WebGL, practice vs test, seated vs standing, MR vs VR.

### 8. Exit poll hooks early

Add hooks even without final questions. Questions are configured on the platform — dashboard or MCP — with no new build. Stakeholders inevitably ask "can we survey users?" weeks after launch — hooks eliminate that bottleneck.

Scope the in-app side honestly. Unity ships survey UI you can place, and Unreal ships UMG widgets and actors plus three Blueprint nodes (though the panel is unanswerable without a Widget Interaction component on the player or controller). The WebXR SDK fetches the question set and submits answers but renders nothing, so a WebXR exit poll includes building the survey UI. Plan that as real work rather than a hook placement.

> **Field note:** See `field_notes.md` → _Exit poll hooks are cheap on the engine SDKs, less so on WebXR — place them early either way_ for more.

Good default positions: beginning of experience, end of module/session/content unit, major milestones.

### 9. Controller and boundary tracking verification

Single most common issue in integration reviews — verify both in every validation pass, regardless of project type.

> **Field note:** See `field_notes.md` → _Controller and boundary tracking verification_ for more.

### 10. Scene export fidelity check

Geometry that exports badly makes replay misleading rather than merely imperfect, and the fix has to land before the scene is uploaded. Check early, and check what the project's specific toolchain does:

- **Unity:** custom shaders render as white materials on the dashboard, because the GLTF exporter cannot map custom properties to PBR. A shader-properties export class is needed.
- **Unreal:** complex materials may not translate, so the diffuse output has to be representative on its own. Forward Shading can crash the glTF export, TextRenderers do not export, and Metahumans need LOD 0 with hair and skeletal animation unsupported.
- **WebXR:** export capability varies by adapter. Some frameworks have no scene export at all, and Wonderland exports geometry only, with no materials or textures.

> **Field note:** See `field_notes.md` → _Scene export fidelity_ for more.

### 11. Validation sessions

Instrumentation is not done when the code compiles. It is done when the data is usable.

Run validation sessions that confirm: scenes uploaded and correctly referenced in project config, timeline tags appear, key events fire with correct properties, dynamic objects visible with gaze, session/participant properties populated, controller/boundary tracking active, scene geometry exports with correct materials, dev and production traffic separated, offline upload works if needed and supported.

### 12. At least one analysis surface

Name the first objective, query, replay path, or cohort comparison. If the plan cannot name the first useful analysis view, it is not ready.

---

## Naming and property conventions

Consistency matters more than cleverness.

### Event naming

Use stable event names that describe what happened.

Good: `module_started`, `step_completed`, `mission_ended`, `content_bookmarked`, `prototype_rating_submitted`

Bad: `module_completed_easy_path_controller_user_02`, `menu_trackbrowser_clicked_round3`, `prototype1_q1_rating5`

The bad versions bury analyzable information inside the event name.

### Use properties for variation

Properties should carry the variation.

Examples:
- event: `mission_started` — properties: `mission_name`, `difficulty`, `weapon_loadout`
- event: `menu_selected` — properties: `menu_name`, `selected_item`
- event: `prototype_interaction_started` — properties: `prototype_id`, `interaction_order`

### Units belong in names

At minimum, time fields should end with `_seconds`.

Good: `duration_seconds`, `time_to_complete_seconds`, `pause_duration_seconds`

Also consider: `_percent`, `_count`, `_meters`, `_bpm`

### Prefer stable IDs plus readable labels

If names can change over time, use both an id and a label.

Example: `content_id = course_023`, `content_name = Evening Reset`

### Record recurrence and order when sequence matters

Examples: `attempt_count`, `repeat_count`, `interaction_order`, `step_order`, `visit_number`

### Use booleans and enums carefully

Good booleans: `is_mr`, `is_completed`, `used_hint`

Good enums: `movement_style = teleport`, `condition = variant_b`, `service_type = windows`

Avoid overloading booleans for things that really have many states.

### Match the app's domain language

Games may use `mission`, `chapter`, or `quest`. Training may use `module`, `procedure`, or `step`. Content apps may use `class`, `track`, `course`. Research may use `trial`, `block`, `stimulus`.

Use the developer's domain language, but keep the logic clean.

---

## Phased implementation

Do not hand a team a sixty-line instrumentation plan if they only need a decision-grade start.

### Phase 1: foundation

Goal: make the project observable.

Usually includes: primary activity start/end, FTUE stages, key dynamic objects, essential session context, dev vs prod separation, exit poll hooks, validation sessions.

Questions Phase 1 can answer: Are people reaching the core activity? How long does it take? Where does onboarding fail? Is the data pipeline healthy?

After Phase 1 is validated, you should be able to open session replay and see exactly where users drop out of onboarding — that alone often changes decisions.

### Phase 2: decision-grade instrumentation

Goal: support the team's real product or research questions.

Usually includes: step/milestone events, recurrent behavior events, error/incorrect action events, participant identity/cohort, success metrics, objectives, content/prototype metadata.

Questions Phase 2 can answer: Where do users fail? Which content wins? What behavior predicts success or churn? Which cohort behaves differently?

### Phase 3: optimization and experimentation

Goal: tune and compare.

Usually includes: UI interaction detail, variant/condition tracking, remote controls / A/B support, catalog attributes, agent/conversational metrics, deeper surveys, social/multiplayer logic.

Several of these are engine-only or unconfirmed on WebXR (remote controls, multiplayer components, media, local cache). Screen against `sdk_capability_matrix.md` before promising them.

Questions Phase 3 can answer: Which variant performs better? Which settings correlate with better outcomes? Which curation increases repeat use?

---

## Cross-cut overlays

These patterns can apply on top of almost any archetype.

### Shared devices and SSO

If people share headsets, device id is not enough. Use participant id, participant properties (role, department, store, cohort, training level), and session tags for cohorting.

Typical fits: enterprise training, classrooms, labs, kiosks.

### Multiplayer and collaborative sessions

Useful session properties: `room_id`, `session_role`, `player_count`, `match_type`

Useful events: `room_joined`, `room_left`, `ready_state_changed`, `collaboration_started/ended`, `other_user_joined/left`

Without room context, collaborative behavior is hard to interpret.

### Content catalog and subscriptions

Useful properties: `content_id`, `content_name`, `content_category`, `difficulty`, `instructor`, `program`, `is_premium`

Useful events: `content_selected`, `content_started/ended`, `content_bookmarked`, `content_paused`, `content_recommended`

### UI and tool-surface instrumentation

Useful patterns: `ui_opened`, `ui_closed`, `ui_selected`, `tool_activated`, `tool_closed`

Useful properties: `ui_name`, `selected_item`, `duration_seconds`, `context_surface`

### Recurring versus one-time behavior

Do not only track milestone events if recurring behavior is the thing that answers the question.

Good one-time events: tutorial completed, first purchase, first success.

Good recurring events: item added, item moved, step repeated, target hit, portal engaged, track loaded, hint used.

If behavior happens many times and you care about frequency, order, or rate, it should be instrumented as recurring behavior.

### Input mode, device choice, and mixed reality

Useful session properties: `input_mode`, `movement_style`, `device_choice`, `is_mr`

Useful events: `input_mode_changed`, `movement_style_changed`

Do not assume hardware metadata alone captures the product distinction the team cares about.

### Live ops, variants, and experimentation

Record the condition via session property, participant property, session tag, or remote control state. The important part is that the assigned condition is recoverable later. If the condition is not recorded, the experiment effectively did not happen.

Unity and Unreal both support remote controls. Where the SDK does not, the app's own config or feature-flag system assigns the condition and the plan simply records it as a session property. The requirement is on the recording, not on where the assignment came from.

### Privacy-sensitive capture

Use special care with: audio, voice, transcripts, biometrics, open-text, demographics. Only recommend when it answers a real decision. Collect minimum needed. Make consent explicit. Prefer category-level metadata before full-content capture.

---

## Business question map

Use this table when translating discovery answers into tracking recommendations.

| Business question | What to add | Why it works |
| --- | --- | --- |
| Where do users drop out during onboarding? | FTUE stage start/completion events with durations | Reveals the exact stage where people stop |
| Which step causes learners to fail? | Step start, completion, skipped, incorrect action events | Supports sequential objectives and duration analysis |
| Which content gets completed, repeated, or abandoned? | Content start/end, pause, bookmark, repeat count | Content-level cohort analysis and replay |
| Which objects attract attention but not action? | Dynamic objects plus events | Replay connects gaze to outcomes |
| Are users getting lost? | Portal/waypoint events, dynamic objects, movement context | Compare pathing, attention, progression blockers |
| Which mechanic is actually being used? | Mechanic-specific events with properties | Usage frequency and adoption measurable |
| Which settings affect success? | Session properties for movement, device, input, plus outcomes | Segmented comparison without event explosion |
| Which prototype or item wins? | Interaction start/end, rating, interaction order, cohort tags | Comparative evaluation and study analysis |
| What predicts completion, churn, or return? | Lifecycle, participant identity, repeated usage, session context | Cohort and behavior comparisons |
| Which UI surfaces are confusing? | UI opened/closed, selection events with duration | Behavioral friction connected to replay |
| Which variant performs better? | Condition/variant property plus outcome events | Clean split testing or pilot comparison |
| What do users say? | Exit poll hooks and question sets | Self-report without new build |
| Do shared-device users complete progression? | Participant ID and properties | Separates people from devices |
| How to prove compliance externally? | Module completion events and objectives | Bridge to LMS/downstream reporting |
| What happened before quit/failure/crash? | Outcome event, context properties, base data flow | Session replay and timeline review |

---

## Anti-patterns

Avoid these unless you have a very specific reason.

### 1. Recommending before discovery
If discovery is skipped, recommendations will sound plausible but generic. They will not answer the team's real questions.

### 2. Encoding properties into event names
This makes grouping and querying harder. Use stable event names and carry variation in properties.

### 3. Using session properties for person-level data
Role, cohort, department, or user segment usually belong on the participant, not the session.

### 4. Using participant properties for one-session values
Session-specific accuracy or one-session movement style usually belongs on the session, not the participant.

### 5. Tracking everything as a dynamic object
Only track objects that support an actual analysis question. Spawned objects pollute the dashboard.

### 6. Missing duration fields
Without durations, many lifecycle events are much less useful.

### 7. Omitting units
`duration = 120` is ambiguous. `duration_seconds = 120` is not.

### 8. Only tracking one-time milestones
Many important questions depend on repeated behavior, not just firsts or lasts.

### 9. No dev versus prod separation
Test traffic will pollute dashboards quickly.

### 10. No validation plan
Instrumentation that never gets checked is likely wrong.

### 11. Casual privacy-sensitive collection
Do not recommend audio, transcript, or demographic capture just because it is technically possible.

### 12. Recommending features without naming the first analysis use
If you cannot describe the first objective, query, replay view, or dashboard use, the recommendation is not grounded enough.

### 13. Silently renaming or replacing existing events
Renaming an event breaks every dashboard query, saved segment, and objective built on the old name, and permanently splits the historical series — old sessions keep the old name. Every replaced or retired event needs an explicit break-risk decision: cut over, dual-send for one release, or leave it alone. See the event status column and migration map in `track_plan_template.md`.

### 14. Letting naming diverge between SDKs

When a team ships the same experience on more than one SDK, every build reports into the same project and the same queries. Divergent event names, property keys or units split every series permanently. Write the conventions once and apply them everywhere. Types count as well as names: a property that is numeric on one SDK and stringified on another has diverged even when the key matches.

### 15. Sending numbers as strings

A numeric property that arrives as text cannot be averaged, charted, bucketed or filtered numerically, and nothing on the dashboard flags it. The most common cause is Unreal's Blueprint custom event variant, which stringifies every value; the C++ variant preserves types. Check the authoring surface before assuming a numeric plan row will be queryable.
