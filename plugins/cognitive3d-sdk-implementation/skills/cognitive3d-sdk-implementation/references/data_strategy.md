# Cognitive3D Data Strategy

_Stable strategy layer for deciding what to track and why._

Read this file after the main SKILL.md when you need deeper guidance on primitives, phasing, overlays, naming, or anti-patterns.

**This file is SDK-neutral.** Everything here describes what to track and why, and applies equally to Unity, Unreal, visionOS, Android XR and WebXR projects. It will occasionally suggest something a particular SDK or framework cannot do, so screen the finished plan against `sdk_capability_matrix.md` before presenting it, and route implementation detail to the SDK reference for the project's target.

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
| Presence, attention or dwell time in a place | Zone | Named shapes drawn on a scene from the dashboard, or created through the MCP server. The platform records when each session enters and leaves and how long it stays, from the head position and gaze it already captures, so a zone needs no code and no new build. Counts visits only in sessions recorded after it is saved. See [Zone or event?](#zone-or-event) |
| Measure completion logic or sequences | Objective | Turns events, gaze, and survey responses into success logic. Created on the dashboard or programmatically via the MCP server with a write-enabled organization key |
| Self-reported feedback or cohort questions | Exit poll | Best for sentiment, preference, confidence, and study questions. Question sets are created on the dashboard or programmatically via the MCP server with a write-enabled organization key |
| Continuous value sampled over time | Sensor | Time-series charted on the session timeline; aggregates (avg/min/max) queryable. **How much arrives automatically varies sharply by SDK** — broad on Unity, opt-in components on Unreal, narrow on visionOS, FPS only on Android XR — so check `sdk_capability_matrix.md` before assuming a stream is free. Custom sensors capture app-specific continuous values |
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

### Zone or event?

A zone is a named area drawn on a scene from the dashboard (Scenes, then the scene version, then the Zones tab) or created through the MCP server (see _Creating zones programmatically_ below). Once saved, the platform records a visit each time a session enters and leaves it, with entry time, exit time and duration, from the head position and gaze data it already captures. No code, no build, no SDK dependency: zones behave the same on every target that sends head position, and the gaze-based visit types on every target that sends gaze.

**If the question is about a place, use a zone, not an event.** "Did they reach the inspection point", "how long did they spend at the workbench", "which bay did they visit first", "did they pass through the doorway" and "which areas of the store draw attention" are zone questions. Emitting `area_entered`, `area_exited`, `doorway_passed` or `zone_dwell` events for them duplicates something the platform computes for free, costs a build to change, and cannot be redrawn after the fact. Sending the scene's own layout as events (`scene_area` and `scene_object` inventories at session start) is the same mistake one level up: the scene upload and the dynamic object manifest already describe the scene.

What a zone records:

- Three visit types, all tracked at once. **Head position** counts a visit while the participant's head is inside the zone. **Gazed in** counts a visit while the point they are looking at is inside it. **Gazed through** counts a visit while their line of sight passes through it. Head position is presence; the gaze types are attention from a distance.
- Visits shorter than 0.5 seconds are ignored and visits separated by less than 0.5 seconds are merged, so a zone is not a frame-accurate trigger.
- A zone is one or more shapes (rectangle, circle or sphere, height-limited or infinitely tall), and its shapes can sit on different scenes, so a visit continues across a scene change as long as the new scene contains one of the zone's shapes.

Where a zone surfaces:

- The **Zones report** dashboard template: sessions visiting a zone, sessions by zone, zone map, zone heatmap, share of time by zone, time in zone, first zone entered, and visiting sessions over time. The same widgets go on custom dashboards from the Spatial category. The report reads head-position visits.
- **Zone objectives.** A zone step names a zone, a visit type, an occurrence count and an optional minimum dwell per visit ("reach the safety area at least twice and stay 10 seconds each time"). Zone steps combine with event, gaze and ExitPoll steps in one objective, and can be created on the dashboard or through the MCP server's `create_objective` (below).
- **Session Replay** shows zones as labelled volumes and each visit as a bar on the timeline.
- **Analyze**, on the Zones tab, counts gaze, head position or custom events inside any shape across sessions already recorded, so an area can be explored and its boundary refined before a zone is saved.
- **Queries.** Visits are the `spaceVisit` entity in slicer queries and the `spaceVisits` array on the session document, with zone id, start and end time and duration per visit.

What a zone does not record, so an event still earns its place:

- **Order and direction inside the zone.** A zone knows entered, left and how long, not whether the person reversed along their own path, circled, or which way they faced. Backtracking, path efficiency and "went the wrong way" stay as events or as analysis over the position stream.
- **What they did there.** Grasps, tool use, UI choices and errors inside a zone are still events; the zone gives them a place.
- **History.** A zone counts visits only in sessions recorded after it is saved, and the zones table shows a "Collecting since" date per zone. Analyze answers the historical question, with limits: it cannot yet count saved zones that contain circles or rotated rectangles, and it reads at most the first 65,536 sessions for gaze and head position and the first 10,000 for events.

Planning rules:

- Put zones in the plan as their own table next to dynamic objects: zone name, scene or scenes, shape, visit type, the question it answers, and the report or objective that reads it. The template has the table.
- **Draw and save zones before the validation sessions run.** This is the one operational gotcha. A zone saved after the first real sessions has no data for them, and a zone objective evaluates only from its zone's save date.
- Saved zones are frozen. Name and description can change; shapes cannot. To move a boundary, duplicate the shape, edit the copy and delete the old zone, and expect the "Collecting since" clock to restart.
- Organization Admins and Project Admins create and delete zones; Observers can view them. A zone that an objective uses cannot be deleted until the objective step is removed. Name in the plan who will draw them.
- Scenes generated by a pipeline multiply the work: a zone's shapes live on a scene, so forty generated layouts need forty sets of shapes. Drawing them is a dashboard task per scene; creating them is one MCP call per scene (below), so on generated-scene projects keep the zone definitions beside the scene generator and create them from there.

#### Creating zones programmatically

With a Cognitive3D MCP server connected, zones can be read and created without the dashboard.

Read tools, in the default `analytics` toolset:

- `list_zones(project_id)`: every zone in the project with its shapes, each resolved to a scene name and a one-line geometry summary. Call it first on any brownfield project so the plan amends the zones that exist rather than duplicating them.
- `list_shapes(project_id, scene_id)`: the shapes on one scene and which zone each belongs to.
- `get_zone_visits(project_id, zone_id?, start_date?, end_date?)`: visits, visiting sessions, share of sessions, mean visit and total dwell per zone, with saved-but-unvisited zones listed against their saved date. This is the validation check for "did the zone collect anything".

Write tools, in the opt-in `write` toolset alongside objectives and ExitPoll, so they need an organization key with `CHANGE_SPACES_CONFIGURATION`:

- `create_zone_from_shapes(project_id, name, shapes=[{scene_id, shape_type, name, x, y, z, width?, length?, height?, radius?, yaw_degrees?}], description?, publish?)` is the one to reach for. It validates every shape and creates them in one call, and removes anything it created if a step fails. **By default it creates a draft, not a zone**: the shapes land as one draft group named after the zone on the scene's Zones page, the reply links that page, and a person reviews the geometry and publishes it. Pass `publish=true` to create the zone immediately with no review; that is the scripted path, and the only path for a zone whose shapes sit on different scenes, because draft groups are per scene.
- `create_zone(project_id, name, shape_ids)` publishes existing shapes as a zone. It is the MCP equivalent of the dashboard's Save changes, scoped to exactly those shapes. Prefer it over the dashboard button for publishing a reviewed draft: Save changes publishes every draft on that scene, including ones nobody meant to keep.
- `create_shape` (one draft shape; `y` is required for BOX, SPHERE and CYLINDER and ignored by the infinitely tall types), `update_zone` (name and description only; shapes are frozen once saved), `delete_shape` and `delete_zone` cover the rest.
- Every create and delete is a dry run until `confirm` is passed, and the dry run names any blocker before anything is written: a shape already in a zone, a zone an objective references, a key without the capability.
- `create_objective` accepts a zone step: `{type: "zone", name, zone_id, occurrence, duration_seconds?, visit_type?}` with `visit_type` one of `hmd` (head position, the default), `gazed_in` or `gazed_through`. Existing zone steps read back through `get_project_objectives`. The zone must be published first; a draft has no id to point at.

Shape contract: `shape_type` is `BOX` (width, height, length), `INF_TALL_BOX` (width, length), `SPHERE` (radius), `CYLINDER` (radius, height) or `INF_TALL_CYLINDER` (radius). Positions are metres, Y up, origin at the shape centre; the infinitely tall types sit on the floor and ignore `y`. Rotation is `yaw_degrees`; the tool builds the quaternion, so never write one by hand. The infinitely tall rectangle is what the dashboard's top-down editor draws, and is the right default for floor areas.

**Before asking an agent to define zones on a scene.** An agent has no view of the scene's geometry, so give it anchors first. The scene needs recorded sessions: the agent reads where participants stand from the position heatmap (`query_spatial_heatmap`), which is in the same coordinates zones use. Then draw one or two reference zones yourself on the dashboard's Zones page, at landmarks you can name ("the counter", "the exit door"), and save them; they tell the agent which coordinates mean what, and it reads them back with `list_zones`. With both in place, ask for zones as drafts, open the linked Zones page, check the shapes against the scene mesh, and click Save changes to publish. If the scene has no sessions yet and no reference zones, the only way to get coordinates is from inside the app's own project, where object transforms can be read directly (the editor on Unity and Unreal; the scene code on the other targets). One caution: Save changes publishes every draft on that scene, so clear out stale drafts first. The MCP server's own zones resource carries the agent-side workflow, so the plan only needs to say which anchors exist and who reviews the drafts.

Rules for using them in a plan:

- The save-date rule is unchanged, and it is the publish that starts the clock. A draft records nothing; a zone counts visits from the moment it is published, whether by `publish=true`, `create_zone` or Save changes. So publish zones before the validation sessions run, in the same working session as the plan when the key allows writes, and check `list_zones` for drafts left unpublished.
- For generated or pipeline-built scenes, keep the zone definitions next to the scene generator and create them per scene with `create_zone_from_shapes`. The plan's zone table is the input to that script.
- Read before writing. `list_zones` first; a brownfield audit that finds area events should also find whether zones already cover the same places.
- Zone objectives can be created in the same pass as the zones, with `create_objective` zone steps, once the zones are published. Validate them the same way as any objective: against one session where someone knows the ground truth.

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

### 3. Key dynamic objects and zones

Add dynamic objects only where object-level attention or interaction answers a question. Good candidates: equipment, tools, instruction surfaces, targets, prototypes, guide objects, objects used in objectives. Do not track everything — spawned objects like bullets pollute the object list.

**Important:** registering an object in the app is only half the job. Dynamic object meshes must also be exported and uploaded **separately from the scene** (Unity's Feature Builder, Unreal's Dynamic Object Manager, or the Upload Web App on visionOS, Android XR, WebXR and C++). Without that step, objects have no visual representation in dashboard replay. Scene upload and dynamic object mesh upload are two distinct workflows on every SDK, and this is the single most commonly missed step in an integration.

**Availability:** dynamic objects exist on Unity, Unreal, visionOS and Android XR, and on WebXR only for the Three.js and Mattercraft adapters. Where they are unavailable, substitute custom events carrying an object identifier property and state in the plan what that cannot answer: dwell before action, attention without interaction, and object heatmaps. See `sdk_capability_matrix.md`.

**Objects spawned at runtime** need an identity strategy: Unity and Unreal use ID Pools (Unreal's is an Id Pool Asset that must be sized to the concurrent spawn count); visionOS, Android XR and WebXR register each instance explicitly at spawn time. Route the mechanics to the SDK reference.

> **Field note:** See `field_notes.md` → _Dynamic objects: quality over quantity_ for more.

Zones are the place-shaped counterpart. Wherever a top question is about where someone was, how long they stayed there, or which area they looked at, add a zone rather than an area event: the platform computes the visits from data it already has, and the zone can be redrawn without a build. List them in the plan's zone table, draw them before the validation sessions, and keep events only for what the zone cannot see (direction, backtracking, what happened inside). See [Zone or event?](#zone-or-event).

> **Field note:** See `field_notes.md` → _Zones replace place events, and the save date is the clock_ for more.

### 4. Session context

Common examples: `app_mode`, `build_channel`, `development_mode`, `device_choice`, `movement_style`, `input_mode`, `scenario_type`, `variant`, `condition`

### 5. Participant identity, when it matters

Set when cross-session analysis matters: shared-device training, employee progression, consumer research cohorts, longitudinal studies. Do not force identity where it is not needed.

### 6. Dev versus production separation

For builds and deployments used during development, use a `development_mode` session property, session tag, or separate project. Without this, dashboards become noisy fast. A session property is the portable choice: SDK-applied session tags are not documented on every target, so prefer the property when the SDK is visionOS or Android XR, or when you have not confirmed tag support.

How much this matters depends on the SDK, and only Unity does any of it for you. Unity excludes in-editor sessions from major dashboard analytics automatically, so the property mainly covers sideloaded device builds. Unreal records editor sessions and shows them behind a dashboard toggle rather than filtering them out. visionOS, Android XR and WebXR have no equivalent concept at all: local development produces sessions indistinguishable from real ones. On every SDK except Unity, explicit separation is mandatory rather than advisory. See the field note.

> **Field note:** See `field_notes.md` → _Dev/prod separation, and what the SDK does or does not do for you_ for more.

### 7. Input and environment verification

Record when these affect interpretation: hands vs controllers, VR vs WebGL, practice vs test, seated vs standing, MR vs VR.

### 8. Exit poll hooks early

Add hooks even without final questions. Questions are configured on the platform — dashboard or MCP — with no new build. Stakeholders inevitably ask "can we survey users?" weeks after launch — hooks eliminate that bottleneck.

**First confirm the SDK has ExitPoll at all.** It is documented for Unity, Unreal, visionOS, WebXR and C++, and has no Android XR page; verify live before this baseline item goes into an Android XR plan, and substitute the app's own UI writing a custom event if it is genuinely unavailable, naming what that costs: questions baked into the release, and no dashboard-side question management.

Then scope the in-app side honestly. Unity ships survey UI you can place; Unreal ships UMG widgets and actors plus three Blueprint nodes (though the panel is unanswerable without a Widget Interaction component on the player or controller); visionOS ships six SwiftUI question views plus a view model, and caches question sets so surveys work offline, which makes it one of the cheaper places to honour this recommendation. The WebXR SDK fetches the question set and submits answers but renders nothing, so a WebXR exit poll includes building the survey UI. Plan that as real work rather than a hook placement.

> **Field note:** See `field_notes.md` → _Exit poll hooks are cheap on most SDKs, expensive on WebXR, and unconfirmed on Android XR_ for more.

Good default positions: beginning of experience, end of module/session/content unit, major milestones.

### 9. Controller and boundary tracking verification

Single most common issue in integration reviews — verify both in every validation pass on any platform that has them. Apple Vision Pro has neither controllers nor a boundary concept, so mark both not applicable there rather than failed.

> **Field note:** See `field_notes.md` → _Controller and boundary tracking verification_ for more.

### 10. Scene export fidelity check

Geometry that exports badly makes replay misleading rather than merely imperfect, and the fix has to land before the scene is uploaded. Check early, and check what the project's specific toolchain does:

- **Unity:** custom shaders render as white materials on the dashboard, because the GLTF exporter cannot map custom properties to PBR. A shader-properties export class is needed.
- **Unreal:** complex materials may not translate, so the diffuse output has to be representative on its own. Forward Shading can crash the glTF export, TextRenderers do not export, and Metahumans need LOD 0 with hair and skeletal animation unsupported.
- **visionOS:** there is no exporter; geometry comes from the team's own pipeline and goes up through the web app, in glTF Separate rather than GLB. Reuse the Scene ID on later uploads or you create duplicate scenes.
- **Android XR:** there is no exporter at all; geometry is produced by the team's own asset pipeline and uploaded through the web app, which accepts glTF Separate (`.gltf` plus `.bin`) and rejects GLB. Confirm the pipeline can emit that before the plan assumes replay geometry exists.
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

Use stable snake_case event names that describe what happened.

Good: `module_started`, `step_completed`, `mission_ended`, `content_bookmarked`, `prototype_rating_submitted`

Bad: `module_completed_easy_path_controller_user_02`, `menu_trackbrowser_clicked_round3`, `prototype1_q1_rating5`

The bad versions bury analyzable information inside the event name.

**No app or vendor prefix.** `acme.step.completed` is wrong twice: the dotted style mimics the SDK's own built-in events (`c3d.sessionEnd`, `c3d.SceneChange`), so a reader cannot tell app events from platform events, and the prefix hard-codes the app into every name. If several apps report into one project, tell them apart with a session property (`app_name`), never with the event name. Variation goes in properties; the app is a variation.

**Identical across sibling apps.** When a team ships more than one app or build, the same thing has the same name in every one of them. A dashboard built on `step_completed` has to work for all of them without a mapping table.

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

### One label, one ID, nothing else

Each noun (module, step, tool, menu, order) gets one readable label, and one ID only where records must link: an attempt ID that joins every event in an attempt, a step ID that an objective matches on, a tracked-object ID that joins to a dynamic object. The label is what the dashboard shows; the ID exists to join. A third identity is noise.

Example: `step_name = Close the inlet valve`, `step_id = pump_inspection_050`. Not also `step_index`, `step_visit_id`, `step_asset_name` and `step_asset_path`.

If the label can change over time and nothing downstream joins on the noun, the label alone is enough. Before dropping an identity, check that the ones you keep are 1-to-1 with it across the whole content catalogue, not just the sessions seen so far: an asset name that also encodes a variant (`PumpInspection_LargeUnitStory`) carries information the display label may not.

### Record recurrence and order when sequence matters

Ordinals are 1-based and named for what they count. Use `_number` for a position in a fixed sequence (`step_number`, `attempt_number`, `visit_number`) and `_order` for the order in which a person did things (`interaction_order`, `trial_order`). Never `_index`, never 0-based: a 0-based `step_index` next to a 1-based `step_count` reads as an off-by-one on every dashboard.

Every counter states its scope. `attempt_number` that resets each session means "nth attempt of this module in this session"; a trainee's nth attempt ever is a different field and usually comes from the participant view, not the app. Write the scope into the plan so the implementer and the analyst agree.

### Use booleans and enums carefully

Good booleans: `is_mr`, `is_completed`, `used_hint`

Good enums: `movement_style = teleport`, `condition = variant_b`, `service_type = windows`

Avoid overloading booleans for things that really have many states.

### Match the app's domain language

Games may use `mission`, `chapter`, or `quest`. Training may use `module`, `procedure`, or `step`. Content apps may use `class`, `track`, `course`. Research may use `trial`, `block`, `stimulus`.

Use the developer's domain language, but keep the logic clean. One word per level: if the code says `story`, `scenario` and `module` for the same thing, the events say one of them.

### Property hygiene

Properties have a budget even where the SDK sets no cap. Each one must earn its slot, and the test for each is mechanical. Run these over a real session's payloads before sign-off (see the validation step in `SKILL.md`):

- **If a property has one value across every session, drop it.** `attempt_scope = story`, `is_forward = true`, `controller_name = MotionController` on every event are not data. Keep it only if the other value is possible and matters.
- **If the platform records it, do not send it.** Scene, timestamp, session length, device, engine, app version and participant identity are captured without instrumentation (`queryable_data.md`). A `scene_id` or `recorded_at` property duplicates a free field and spends the reader's attention.
- **If it is an engine path, class name or function name, it is not a label.** `/Game/Maintenance/Data/TRN_PumpInspection_DA.TRN_PumpInspection_DA`, `BP_PumpHousing_C`, `OnStepCompleted` describe the code, not what happened. Send the display string the app's own UI shows.
- **If it restates the event name, drop it.** `completed = true` on `step_completed`, `state = ended` on `module_ended`. The event is the statement.
- **One field says how something ended, and it is `outcome`.** Values are lower-case enums: `completed`, `abandoned`, `failed`. Not `reason`, `state`, `source` and `result` on different events for the same meaning. Menus keep `close_reason`, because a menu closing is not an outcome.
- **Enum values are lower-case** (`tutorial`, not `TutorialMode`), and a mode has one field (`mode`), not `mode` and `mode_id`.
- **Context properties apply where they mean something.** Attempt and module fields ride on events inside a module attempt and on nothing else. A lobby click with `module_name` set to the lobby scene name is a context bag, not context. The usual cause is one shared serializer that every event calls; curate per event instead.
- **Counts go on the event that summarises.** `step_count`, `completed_step_count` and `skipped_step_count` belong on `module_ended`, not on every step event.

### Session end is synthetic when the SDK never sent one

If an app is closed from the headset without a clean shutdown, the SDK never sends its end event. The platform then synthesises a `c3d.sessionEnd` at the last head-position sample, with the session length measured to that point; the dashboard shows it as "Application stopped sending data". A late upload of cached data can then add a second, real end event after the session was already processed. On any app where users remove the headset or kill the app to finish, session length and `c3d.sessionEnd` are not a completion measure. Build completion on the app's own lifecycle event (`module_ended` with `outcome`), and treat the synthetic end as a data-quality signal rather than a behaviour.

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

Several of these are engine-only or unconfirmed on the native and browser SDKs (remote controls, multiplayer components, media). Local cache is the exception worth knowing: documented on Unity, Unreal and visionOS; present in the Android XR SDK (a `local_data_cache_size` setting) with no docs page; absent on WebXR. Screen against `sdk_capability_matrix.md` before promising any of them.

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

Useful properties: `ui_name`, `selected_item_name`, `selection_type`, `interaction_order`, `close_reason`, `selection_count`, `duration_seconds`, `location`

Two patterns worth copying for menus. `ui_closed.duration_seconds` (menu opened to menu closed) is time to decide, which is the usual question about a selector. And a menu that is still open when the session ends gets a `ui_closed` with `close_reason = session_ended`, so the open and close counts match and abandonment is visible. Instrument key interfaces only; a tutorial close button or a leaderboard filter rarely answers a question. Each `ui_selected` carries only what was picked and where in the sequence; the final choice and the summary counts go on `ui_closed`.

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

Record the condition via session property, participant property, session tag, or remote control state, using whichever of those the target SDK actually supports. The important part is that the assigned condition is recoverable later. If the condition is not recorded, the experiment effectively did not happen.

Unity and Unreal both support remote controls; visionOS, Android XR and WebXR do not document them. Where the SDK does not, the app's own config or feature-flag system assigns the condition and the plan simply records it as a session property. The requirement is on the recording, not on where the assignment came from.

### Privacy-sensitive capture

Use special care with: audio, voice, transcripts, biometrics, open-text, demographics. Only recommend when it answers a real decision. Collect minimum needed. Make consent explicit. Prefer category-level metadata before full-content capture.

**Never emit events from credential entry, and never per keystroke anywhere.** A login keyboard that fires an event per key press reveals password length and typing rhythm even when no key value is sent. Participant identity is still set at login (that is a participant property, not UI tracking), but the login screen itself gets no UI events. Note that a session which starts in the login scene still records head and gaze data there; if the screen must be untracked entirely, start the session on the first scene after login.

---

## Business question map

Use this table when translating discovery answers into tracking recommendations.

| Business question | What to add | Why it works |
| --- | --- | --- |
| Where do users drop out during onboarding? | FTUE stage start/completion events with durations | Reveals the exact stage where people stop |
| Which step causes learners to fail? | Step start, completion, skipped, incorrect action events | Supports sequential objectives and duration analysis |
| Which content gets completed, repeated, or abandoned? | Content start/end, pause, bookmark, repeat count | Content-level cohort analysis and replay |
| Which objects attract attention but not action? | Dynamic objects plus events | Replay connects gaze to outcomes |
| Are users getting lost? | Zones on waypoints and decision points, dynamic objects, and events only for what a zone cannot see (direction, backtracking) | Zone visits give reach, order and dwell per area with no build; events cover the rest |
| How long do people spend at each station, and which do they visit first? | A zone per station, read through the Zones report | Visits, dwell and first zone entered are computed by the platform; nothing to ship |
| Did the trainee reach the right place and stay long enough? | Zone objective with an occurrence count and a minimum dwell | Location becomes pass or fail without a custom event |
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

### 15. Reporting a proxy measurement as the real thing

Gaze is not one measurement across platforms. On eye-tracked hardware it is where the eyes went; on Apple Vision Pro it is where the head was pointed, because visionOS exposes no eye-tracking rays to applications. A plan that says "time spent looking at the safety notice" on Vision Pro is reporting orientation, and a stakeholder reading that line will assume otherwise. Name the measurement, not the intention, wherever the two can differ.

### 16. Exceeding a property budget

Android XR's docs limit custom events to ten key-value pairs. The SDK does not enforce it, so nothing warns when an event goes over and what the platform does with the surplus is unspecified; treat ten as the contract. "Use properties rather than more event names" is still right there, but the budget is finite, so each property has to earn its slot, and linking an event to a dynamic object spends one of them. Count before shipping the plan.

### 17. Sending numbers as strings

A numeric property that arrives as text cannot be averaged, charted, bucketed or filtered numerically, and nothing on the dashboard flags it. One path does this by design: Unreal's one-node Blueprint *Send Custom Event* stringifies every value, while the *Make Custom Event* plus *Set Float/Integer/Bool Property* nodes and the C++ `FJsonObject` variant keep types. Everywhere else the property map is typed and the risk is the developer: visionOS's docs example shows only a string property although the API is `[String: Any]`, and copying the example turns a duration into text. Check the authoring surface before assuming a numeric plan row will be queryable.

### 18. Shipping the same context bag on every event

One shared serializer that attaches attempt, module, mode, step, scene and timestamp to every event produces lobby clicks with step fields and login events with module fields. Nothing in it is wrong on the event that owns it; everything in it is noise on the events that do not. Curate properties per event, and let context properties apply only where they mean something.

### 19. Several identities for one noun

A module that arrives as `module_name`, `module_id`, `story_name` and `story_id`, all 1-to-1, is one noun sent four ways. Engine asset names and paths are the usual extras. Send one readable label and, only where records must link, one ID.

### 20. Events from credential entry, or per keystroke

Per-key events from a login keyboard leak password length and typing rhythm even with no key value attached, and tell the analyst nothing. No events from credential screens; no per-keystroke events anywhere.

### 21. Restating the event name as a property

`completed = true` on `step_completed`, `state = ended` on `module_ended`, or a `source` property naming the function that fired the event. The event name is the statement. Properties carry what the name does not say.

### 22. Place events where a zone would do

Emitting `area_entered`, `doorway_passed` or dwell events for regions of the scene, or sending the scene's area and furniture inventory as events at session start. Zones record entry, exit and dwell for any drawn area from data the platform already has, and can be redrawn without a build; the scene upload and the dynamic object manifest already describe the layout. Keep events for what a zone cannot see: direction, backtracking, and what happened inside.
