# Cognitive3D Unreal SDK Technical Reference

This file is the routing and API layer for **Unreal Engine** implementation questions.

For Unity projects load `unity_sdk_reference.md`; for browser-based XR load `webxr_sdk_reference.md`. For cross-SDK feature parity at planning time, see `sdk_capability_matrix.md`.

**Primary docs root:** https://docs.cognitive3d.com/
**Unreal docs root:** https://docs.cognitive3d.com/unreal/get-started/
**Repository:** https://github.com/CognitiveVR/cvr-sdk-unreal
**Releases:** https://github.com/CognitiveVR/cvr-sdk-unreal/releases

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative.
3. **Escalate to live docs when freshness matters.** Engine version support, eye-tracking integrations, release packaging and World Partition behaviour all move.
4. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
5. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.
6. **Establish Blueprint or C++ before writing anything.** Both are supported and the API shape differs between them, including in one way that affects data quality (see the Blueprint property-typing warning below).

### Trust hierarchy

1. **Exact live feature page**: e.g. Unreal Dynamic Objects, Unreal Custom Events
2. **Engine landing page**: Unreal get started, feature overview
3. **Releases page** for versions, packaging and engine compatibility
4. **Docs portal root** when the question is broad or needs routing
5. **API/Data and MCP docs** for programmatic reads, and for objective and ExitPoll configuration writes

### Change-watch anchors (verify live before quoting)

- Unreal releases: https://github.com/CognitiveVR/cvr-sdk-unreal/releases
- Docs root: https://docs.cognitive3d.com/
- Supported hardware: https://docs.cognitive3d.com/hardware/
- Troubleshooting (eye tracking SDK list, export fixes): https://docs.cognitive3d.com/unreal/troubleshooting/

---

## Structural differences worth knowing up front

Unreal sits between Unity and WebXR in shape. Like Unity it is editor-first, with in-engine tooling for scene and mesh upload. Unlike Unity it has two authoring surfaces, and several things Unity does automatically are opt-in here.

**Four differences that change a plan or an answer:**

1. **The project must be C++ based.** A pure Blueprint project cannot host the plugin. Converting is a documented and quick step (Tools > New C++ Class, finish the wizard, delete the temporary class, which generates the project structure), but it is a real prerequisite to state during discovery rather than discover at install time.

2. **Blueprint custom event properties are converted to strings.** The Blueprint `Send` variant takes `TArray<AnalyticsEventAttr>` and stringifies every value; the C++ variant takes a `TSharedPtr<FJsonObject>` and preserves types. This is the single most consequential Unreal-specific trap in this whole skill: a `duration_seconds` sent from Blueprint arrives as text and **cannot be averaged, charted, or used in a numeric filter**. `queryable_data.md`'s "send numbers as numbers" rule has a concrete failure mode here. Flag it whenever a Blueprint-authored plan contains a numeric property, and route numeric events through C++ or a C++ helper node.

3. **Much of what Unity records automatically is an opt-in component here.** Framerate, HMD orientation, room size, battery, boundary events, controller tracking loss, HMD height and arm length are **Blueprint Script Macro components you add**, not free capture. Do not tell an Unreal team that FPS or comfort data is automatic; tell them which component to add. See "Built-in components" below.

4. **Editor sessions are recorded and shown, not excluded.** Unity hides in-editor sessions from major dashboard analytics by default. In Unreal, pressing Play creates a session that appears on the dashboard behind an "Editor Mode" toggle. It is visible rather than filtered out, so dev/prod separation still needs an explicit session property or tag.

| | Unity | Unreal | WebXR |
| --- | --- | --- | --- |
| Authoring surface | Editor + C# | Editor + Blueprint **and** C++ | Code (Mattercraft excepted) |
| Install | UPM git URL | download release, extract to `Plugins/`, run Project Setup | `npm install` |
| Scene upload | in-engine tooling | in-engine, via Project Setup / Scene Export | Upload Web App |
| Mesh upload | Feature Builder | Dynamic Object Manager window | Upload Web App |
| Property typing | typed | **typed in C++, stringified in Blueprint** | typed |
| Editor sessions | auto-excluded | recorded, shown behind a toggle | no concept, all sessions real |

---

## Common implementation mental model

1. **Create/choose a project** in the dashboard
2. **Get the Developer Key** (gear icon > Manage Developer Key), enter it in Project Setup, then **Validate Developer Key** to retrieve the Application Key
3. **Install the plugin**: download the release, extract into the project directory, rename the extracted folder to `Plugins`, run Project Setup from the Cognitive3D menu, restart the editor
4. **Place `BP_Cognitive3DActor`** in the level; its presence is what makes a session record
5. **Export and upload the level** through the Scene Export flow
6. **Attach session metadata**: participant info, tags, session properties
7. **Record telemetry**: custom events, sensors, dynamic objects, exit polls
8. **Export and upload dynamic object meshes** through the Dynamic Object Manager
9. **Add the built-in components** the plan relies on
10. **Validate in dashboard**: replay, scene/object views, analysis
11. **Troubleshoot** if data is missing

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Dynamic Object Id Pool, Custom Event, Sensor, ExitPoll, Hook, Question Set, Remote Controls

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Fast route by question type

### "How do I install the SDK?"

- Get started: https://docs.cognitive3d.com/unreal/get-started/
- Releases: https://github.com/CognitiveVR/cvr-sdk-unreal/releases

Requires **Unreal 4.26.2 or newer**; older engine versions have limited support with version-specific packages on the releases page. Verify the current floor live rather than quoting one, since it moves.

Download the release, extract into the project directory, rename the extracted folder to `Plugins`, complete Project Setup from the Cognitive3D menu, then restart the editor.

**To update the SDK:** close Unreal, delete `Plugins/Cognitive3D`, download the latest release, reopen the editor so binaries rebuild. Do not merge a new release over an old folder.

### "How do I configure keys and auth?"

Developer Key comes from the dashboard (gear icon > **Manage Developer Key**). Enter it in Project Setup and choose **Validate Developer Key** to retrieve the Application Key, which is what records session data.

Two config files matter, and both must be closed-editor edits:

| File | Holds |
| --- | --- |
| `Project/c3dlocal/Cognitive3DKeys.ini` | Developer Key |
| `Project/Config/c3dlocal/Cognitive3DSettings.ini` | Application Key and scene data |

**`Cognitive3DKeys.ini` must not be committed to source control.** The docs say so explicitly. If the project is in git, check that it is ignored, and say so without reading the file: SKILL.md rule 4 applies, so never open, echo or log either file's contents.

### "How do I start and end a session?"

- Sessions: https://docs.cognitive3d.com/unreal/sessions/

A session begins automatically when the application runs with **`BP_Cognitive3DActor` present in the level**. That actor's presence, not a line of code, is the switch. `StartSession()` (with or without properties) and `EndSession()` exist for manual control.

- Manual `EndSession()` is the pattern for multiple participant attempts without closing the app, which is common in training and lab research setups.
- For consent-gated capture the docs are direct: if you do not want to record, **do not call `StartSession`**. That is the privacy-clean route, and it is worth naming when a project has a consent step.
- Sessions end automatically when the application closes or the editor stops playing.

C++ access pattern, used by nearly every API on this page:

```cpp
TWeakPtr<FAnalyticsProviderCognitive3D> cognitive =
    FAnalyticsCognitive3D::Get().GetCognitive3DProvider();
if (cognitive.IsValid())
{
    cognitive.Pin()->SetSessionName("Onboarding - Variant B");
}
```

Always guard with `IsValid()` before pinning. Equivalent Blueprint nodes exist for all of these.

### "How do I upload scenes?"

- Scenes: https://docs.cognitive3d.com/unreal/scenes/

**This is an editor workflow.** Export and upload the level through the Scene Export flow; uploading both creates the data container (generating a Scene Id) and uploads geometry for replay context.

- **No game logic is uploaded.** Colliders, trigger volumes, NPC paths and audio sources are not represented in the exported geometry. Say this when a team expects to see their gameplay structure in the scene viewer.
- **Create a new Scene Version** when base level geometry changes substantially, when you want clean separation between testing and production data, or when dynamic objects are significantly restructured or removed. **Merely repositioning existing dynamic objects does not need a re-upload.**
- **Level streaming:** only the last-loaded level with a valid Scene Id records session data. For geometry culling, make all sublevels visible during export.
- **Multiple levels**, three documented options: export only the art-focused level if it gives enough spatial context; combine everything into a temporary level, export, then rename the export folder to the production level name before uploading; or export levels separately, combine in external software such as Blender, and save the combined glTF back to the export directory.
- **Manual scene control:** uncheck **Automatically Set Tracking Scene** in Project Settings, then set the tracking scene from Blueprint or C++.
- **World Partition landscapes (UE 5.4+):** `LandscapeStreamingProxy` actors are handled by exporting the associated `WorldPartitionHLOD` actors instead. If HLODs are not visible, enable **Allow Showing HLODs in Editor** in World Settings, or build them from the World Partition window.

### "How do I track dynamic objects?"

- Dynamic Objects: https://docs.cognitive3d.com/unreal/dynamic-objects/

**Primarily an editor workflow.** Add the **Dynamic Object Component** to the actor, then export and upload meshes through the **Dynamic Object Manager** (Cognitive3D menu > Feature Builder > Dynamic Object), which lists level actors and blueprint dynamics with export and upload status.

Key component properties:

| Property | Purpose |
| --- | --- |
| Mesh Name | which uploaded mesh represents this object in Session Replay |
| Id Source Type | Custom Id, or pool-based |
| Custom Id | GUID identifying an individual object |
| Controller settings | mark as left/right hand and set controller type |

- **Mesh upload is separate from scene upload**, as on every SDK, and is the most commonly missed step. The component alone gives you tracking with no visual in replay.
- **Spawned blueprints use a Dynamic Object Id Pool Asset** rather than a Custom Id: create the asset, add GUIDs matching the expected concurrent spawn count, then reference it from the component's Id Source Type. A spawned blueprint takes an unused value from the pool. Sizing the pool too small is a silent data loss, so ask what the real concurrent maximum is.
- **Controllers** need Dynamic Object components on the motion controller hand meshes, with Controller Type set for the platform and the **Set Left Hand** / **Set Right Hand** buttons used. For spawned controllers, assign the hand value in the Constructor graph. Button input is recorded through the Enhanced Input plugin, or `DefaultInput.ini` mappings on the legacy input system.
- **Engagements** record manipulation states: `UDynamicObject::BeginEngagement(dynamic, "Grab")` and `UDynamicObject::EndEngagement(dynamic, "Grab")`, available in Blueprint too.
- **Proxy Meshes** substitute a simpler stand-in for visuals that cannot export, such as particle systems and deformed meshes. This is the Unreal answer to "our object looks wrong in replay."
- **Multiple dynamics per actor** are supported: individual mesh components can each carry their own tracking.
- **Cross-session aggregation** of fixation, gaze and custom event data requires the scene to hold the complete list of dynamic object IDs, uploaded through the Dynamic Object Manager. Partial ID lists give partial aggregation, which reads as missing data rather than as a configuration gap.

**Not supported for visualization:** material and texture changes, particle systems, skeletal animations, mesh deformation. Same list as Unity.

### "How do I record custom events?"

- Custom Events: https://docs.cognitive3d.com/unreal/customevents/

C++, reached through the provider (same pattern as every other API here, so derive it rather than assuming a loose `customEventRecorder` is in scope):

```cpp
TWeakPtr<FAnalyticsProviderCognitive3D> cognitive =
    FAnalyticsCognitive3D::Get().GetCognitive3DProvider();
if (cognitive.IsValid())
{
    auto customEventRecorder = cognitive.Pin()->customEventRecorder;

    customEventRecorder->Send(TEXT("step_completed"));
    customEventRecorder->Send(TEXT("step_completed"), position);
    customEventRecorder->Send(TEXT("step_completed"), properties);            // TSharedPtr<FJsonObject>
    customEventRecorder->Send(TEXT("step_completed"), properties, dynamicId);
}
```

For events with a duration, use a persistent `UCustomEvent` object, which times itself between construction and `Send()`:

| Method | Purpose |
| --- | --- |
| `SetCategory(FString)` | event category |
| `SetDynamicObject(UDynamicObjectComponent*)` | associate with a dynamic object |
| `SetProperty(FString key, value)` | set a property |
| `SetPosition(FVector)` | explicit position |
| `AppendAllSensors()` | attach current sensor values to the event |
| `Send()` | record and transmit |

Position and HMD position are recorded automatically when not supplied, so unlike WebXR the position argument is optional.

**The Blueprint typing trap, restated because it matters:** the Blueprint `Send` variant takes `TArray<AnalyticsEventAttr>` and **converts every property value to a string**. The C++ `FJsonObject` variant preserves types. Any numeric property that must be averaged, charted, bucketed or compared numerically has to go through the C++ path. When auditing an existing Unreal integration, check which variant is in use before concluding the properties are fine; this failure is invisible until someone tries to chart a duration.

`AppendAllSensors()` is a genuinely useful Unreal-specific move for biometric work: it snapshots sensor values onto the event, so an outcome event carries the physiological state at that moment without a separate join.

**Trigger Areas** are the documented pattern for region enter/exit events, built on `ActorBeginOverlap`.

### "How do I add session and participant metadata?"

- Sessions: https://docs.cognitive3d.com/unreal/sessions/
- Participants: https://docs.cognitive3d.com/unreal/participants/

```cpp
cognitive.Pin()->SetSessionName("Onboarding Tutorial - Version B");
cognitive.Pin()->SetSessionProperty("app_mode", "practice");
cognitive.Pin()->SetSessionTag("beta");

cognitive.Pin()->SetParticipantId("UniqueID_0001");
cognitive.Pin()->SetParticipantName("Jane Doe");
cognitive.Pin()->SetParticipantProperty("Years Employed", 3);
cognitive.Pin()->SetParticipantProperty("Handedness", "Left");
```

- Session name can be set before or after the session starts; it falls back to the participant name, then to an auto-generated name.
- Session properties can be set at any time and overwrite duplicates by key.
- Session tags are for isolating metrics, and can also be applied after the fact on the dashboard, which makes them the cheap route for ad hoc cohorting.
- The docs recommend pairing a unique ID (employee number, for instance) with a friendly full name.
- **The docs advise against hardcoding participant data.** The suggested pattern is to isolate the user in an empty scene before the session starts and collect participant properties through an ExitPoll. That is a useful answer when a team asks how to identify people without a login, and it pairs well with the shared-device field note.

### "How do I record sensors?"

- Sensors: https://docs.cognitive3d.com/unreal/sensors/

```cpp
TWeakPtr<FAnalyticsProviderCognitive3D> cognitive =
    FAnalyticsCognitive3D::Get().GetCognitive3DProvider();
if (cognitive.IsValid())
{
    auto provider = cognitive.Pin();
    provider->sensors->RecordSensor("Heart.Blood Oxygen", oxygen);
    provider->sensors->InitializeSensor("Heart.My 100hz Sensor", 100, initialValue);
}
```

Exact member names and overloads move between releases; confirm against the plugin source or the live page before handing a team code to paste.

Three constraints that shape plans:

- **Float values only.** Booleans and strings are not sensor data here; use properties or events.
- **Default cap is 10 Hz.** Data recorded faster is discarded unless the sensor is initialized with a custom frequency, and high-frequency data will not appear in SceneViewer even when it is captured. Do not plan a 60 Hz stream expecting to see it in replay.
- **Period-separated names group sensors** in Session Replay (`Heart.Blood Oxygen`), which is worth using from the start since renaming later splits the series.

**There is no automatic sensor list the way Unity has one.** Framerate, HMD orientation, room size and battery come from built-in components you add. See the next section.

### "What are the built-in components?"

- Built-In Components: https://docs.cognitive3d.com/unreal/built-in-components/

Eleven Blueprint Script Macro components. **These are opt-in**, and several of the metrics a stakeholder will assume are automatic live here. The engine and SDK version requirements below are as documented at time of writing and are exactly the kind of thing rule 8 says not to harden: confirm them live before making a commitment that depends on one.

| Component | Records | Requires |
| --- | --- | --- |
| Room Size | configured room size in metres | UE 4.27+, or 4.26+ on Oculus/Meta |
| Arm Length | approximate arm length from HMD-to-hand distance | dynamic objects on hands |
| HMD Height | approximate player height from median HMD elevation | none |
| HMD Recenter | event on participant recenter | none |
| Boundary Event | event on boundary crossing | none |
| Battery Level | battery percentage and charging status | none |
| Framerate | sensor for average and 1% low framerate | SDK v1.8.3+ for Meta SpaceWarp detection |
| Hand Elevation | hand position relative to HMD | dynamic objects on hands |
| HMD Orientation | sensor for HMD pitch and yaw | none |
| Controller Tracking Events | event on controller tracking loss | none |
| Input Tracker | controller button states over time | dynamic objects and custom input definitions |

**Planning consequence:** whenever a plan leans on comfort, ergonomics, performance or boundary data in an Unreal project, name the specific component that produces it. Three of them (Arm Length, Hand Elevation, Input Tracker) additionally depend on hand dynamic objects being configured, so they inherit that prerequisite.

### "How do I ask users questions in-app?"

- ExitPoll: https://docs.cognitive3d.com/unreal/exitpoll/
- Dashboard ExitPoll Results: https://docs.cognitive3d.com/dashboard/exitpoll-results/
- MCP ExitPoll tools: https://docs.cognitive3d.com/mcp-server/exitpoll/

**Hybrid task.** Hooks and Question Sets are created on the platform; the in-app side is Blueprint.

Unreal ships the survey UI, so the app-side cost is closer to Unity than to WebXR. Pre-built UMG widgets and actors live under **Show Plugin Content > Cognitive3D Content**, including a Scale Panel and a Multiple Choice Panel with replaceable button textures.

Three Blueprint nodes carry the basic implementation: **Get Question Set** (takes the Hook), **Begin ExitPoll Actor**, and a completion output for whatever follows the survey.

Presentation and interaction details worth stating before a team builds around it:

- The panel appears **400 cm in front of the participant** by default and continuously rotates to face them.
- **A Widget Interaction component is mandatory** on the player or motion controller; it draws a line forward to hit the buttons. Without it the survey renders and cannot be answered, which presents as an unresponsive panel rather than an error.
- Buttons support a configurable **Gaze Duration** for dwell activation, which is the accessible route when controllers are not guaranteed.
- **Voice panels need specific `DefaultEngine.ini` entries.** Check that before promising voice responses.
- Customising the provided panels is supported through texture overrides; removing or renaming core elements inside them breaks text and button display. Prefer texture overrides to surgery.

**ExitPoll platform constraints are identical across SDKs**, because they belong to the platform. Flag these before any write:

- Question set versions are immutable: editing creates a new version. Removal is archival only, with no hard delete.
- **A new version does not move existing hooks.** Hooks stay on the version they were assigned until reassigned with `update_exitpoll_hook`, so publishing v2 and expecting the app to pick it up is a silent no-op.
- Hooks cannot be deleted, only unassigned, and a hook with no question set assigned is skipped silently at runtime.

Question sets are configurable **either** on the dashboard **or** via MCP (`create_exitpoll_question_set`, `archive_exitpoll_question_set`, `create_exitpoll_hook`, `update_exitpoll_hook` for writes; `get_exitpoll_configuration` and `get_exitpoll_question_set` for reads). MCP writes need a write-enabled organization key with an org- or project-admin role. Ask which route the team wants (route-selection gates are in SKILL.md Step 7).

### "How do I create or change objectives?"

Objectives are a platform resource and behave identically regardless of SDK.

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

Two routes, both fully supported: the dashboard (no API key, right default when a non-developer owns definitions) or the MCP server (`create_objective`, `update_objective`, `delete_objective`, needs a write-enabled organization key). Ask which the team wants; always dry-run first.

**Objective platform constraints, identical to the other SDKs:**

- `sequential` cannot be changed after save.
- Writing steps asynchronously re-scores roughly the last 30 days of sessions and nothing older.
- `name` is capped at 32 characters.
- Gaze and fixation steps reference dynamic object IDs, which are per-project, so such objectives cannot be copied between projects verbatim.
- `delete_objective` is a **permanent cascade** with no restore path. Confirm intent explicitly.

### "How do I track gaze and fixations?"

- Gaze and Fixations: https://docs.cognitive3d.com/unreal/gaze-fixations/
- Fixations explainer: https://docs.cognitive3d.com/fixations/
- Supported hardware: https://docs.cognitive3d.com/hardware/

Gaze is recorded from the participant's viewpoint via the **Player Tracker** component, which the standard Project Setup places as part of the player setup rather than leaving to the team. It is therefore "automatic" in a way the eleven built-in components in the previous section are not, but it is still a component: if gaze is missing, confirm Player Tracker is actually present before looking further.

True eye-tracked fixations additionally need the **Fixation Recorder** plus an eye tracking SDK integration.

Documented eye tracking integrations: TobiiEyeTracking, SRanipal (v1.2 and v1.3), Varjo, PicoMobile, HPGlia. Verify the current list live. **Varjo needs build dependencies added** to `Cognitive3D.Build.cs`:

```
PublicDependencyModuleNames.Add("VarjoHMD");
PublicDependencyModuleNames.Add("VarjoEyeTracker");
```

### "How do I control runtime behavior remotely?"

- Unreal Remote Controls: https://docs.cognitive3d.com/unreal/remote-controls/
- Dashboard Remote Controls: https://docs.cognitive3d.com/dashboard/remote-controls/

Supported, unlike WebXR. This makes Unreal a viable target for live tuning and variant experiments without new builds.

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/
- Android plugin: https://docs.cognitive3d.com/unreal/android-plugin/
- Meta XR platform implementation: https://docs.cognitive3d.com/unreal/metaxr-platform-implementation/

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

Identical across SDKs. Route API query construction to the `cognitive3d-public-api` skill.

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- MCP is a data and configuration layer, not an instrumentation layer. It reads project data and writes objectives and ExitPoll configuration. It is SDK-agnostic, so everything in SKILL.md about MCP-based validation applies unchanged to Unreal projects.

---

## Unreal SDK directory

### First-stop pages

- Get started: https://docs.cognitive3d.com/unreal/get-started/
- Feature overview: https://docs.cognitive3d.com/unreal/feature-overview/
- Feature Builder: https://docs.cognitive3d.com/unreal/feature-builder/
- Terminology: https://docs.cognitive3d.com/unreal/terminology/
- Settings: https://docs.cognitive3d.com/unreal/settings/
- Troubleshooting: https://docs.cognitive3d.com/unreal/troubleshooting/

### Core feature pages

- Sessions: https://docs.cognitive3d.com/unreal/sessions/
- Scenes: https://docs.cognitive3d.com/unreal/scenes/
- Custom Events: https://docs.cognitive3d.com/unreal/customevents/
- Dynamic Objects: https://docs.cognitive3d.com/unreal/dynamic-objects/
- Gaze and Fixations: https://docs.cognitive3d.com/unreal/gaze-fixations/
- Boundary: https://docs.cognitive3d.com/unreal/boundary/
- ExitPoll: https://docs.cognitive3d.com/unreal/exitpoll/
- Sensors: https://docs.cognitive3d.com/unreal/sensors/
- Participants: https://docs.cognitive3d.com/unreal/participants/
- Built-In Components: https://docs.cognitive3d.com/unreal/built-in-components/
- Remote Controls: https://docs.cognitive3d.com/unreal/remote-controls/
- External Android Plugin: https://docs.cognitive3d.com/unreal/android-plugin/

### Extra feature pages

- Active Session View: https://docs.cognitive3d.com/unreal/active-session-view/
- Multiplayer: https://docs.cognitive3d.com/unreal/multiplayer/
- Local Cache: https://docs.cognitive3d.com/unreal/local-cache/
- Media: https://docs.cognitive3d.com/unreal/media/
- Additional SDK features: https://docs.cognitive3d.com/unreal/additional-sdk-features/
- Meta XR platform implementation: https://docs.cognitive3d.com/unreal/metaxr-platform-implementation/

### Project settings reference

From https://docs.cognitive3d.com/unreal/settings/. The batching values are the ones worth changing during validation and changing back afterwards.

**Account:** Developer Key, Application Key, Export Folder Path, Attribution Key

**Cognitive3D:** Enable Logging, Enable Dev Logging, Enable Local Cache, Local Cache Size, Gateway, Automatically Set Tracking Scene

**Exported Scene Data:** Get Latest Scene Version Data, Scene Data Table

**Batching and timing:** Gaze Snapshot Batch Size, Custom Event Batch Size, Custom Event Auto Timer, Dynamic Data Limit, Dynamic Auto Timer, Sensor Data Limit, Sensor Auto Timer, Fixation Batch Size, Fixation Auto Timer

`Enable Logging` plus the Output Log is the first debugging step for almost any missing-data report.

### Best-answer routing

- **installation/setup** → get started, then feature overview
- **session lifecycle/consent** → Sessions
- **metadata/tags/properties** → Sessions and Participants
- **events** → Custom Events
- **interactables/controllers/hands** → Dynamic Objects
- **comfort/performance/room/battery metrics** → Built-In Components, not Sensors
- **dashboard views** → Session Details, Analysis Tool, Objectives, Scene/Object views
- **missing data** → Troubleshooting, Settings (Enable Logging), Scenes

---

## Dashboard directory

Dashboard surfaces are SDK-agnostic. Listed in full so an Unreal engagement never needs another reference file.

**Concepts and framing**

- Concepts: https://docs.cognitive3d.com/dashboard/concepts/
- Metrics glossary: https://docs.cognitive3d.com/metrics-glossary/
- Fixations explainer: https://docs.cognitive3d.com/fixations/

**Replay and behavior review**

- Session Replay: https://docs.cognitive3d.com/dashboard/session-replay/
- Embeddable Session Replay: https://docs.cognitive3d.com/dashboard/embeddable-session-replay/

**Summaries**

- Project Overview: https://docs.cognitive3d.com/dashboard/project-overview/
- App Performance: https://docs.cognitive3d.com/dashboard/app-performance/
- Live Operations: https://docs.cognitive3d.com/dashboard/live-operations/
- Demographics: https://docs.cognitive3d.com/dashboard/demographics/
- Spatial Optimization: https://docs.cognitive3d.com/dashboard/spatial-optimization/
- ExitPoll Results: https://docs.cognitive3d.com/dashboard/exitpoll-results/

**Scene-centric analysis**

- Scene Viewer: https://docs.cognitive3d.com/dashboard/scene-viewer/
- Session Details: https://docs.cognitive3d.com/dashboard/session-details/
- Object Explorer: https://docs.cognitive3d.com/dashboard/object-explorer/
- Object Details: https://docs.cognitive3d.com/dashboard/object-details/

**Objectives and participants**

- Objectives Summary: https://docs.cognitive3d.com/dashboard/objectives-summary/
- Objective Details: https://docs.cognitive3d.com/dashboard/objective-details/
- Creating Objectives: https://docs.cognitive3d.com/dashboard/creating-objectives/
- Participant Summary: https://docs.cognitive3d.com/dashboard/participants-summary/
- Participant Details: https://docs.cognitive3d.com/dashboard/participant-details/

**Analysis, settings and export**

- Simple Analysis: https://docs.cognitive3d.com/dashboard/simple-analysis/
- Advanced Analysis: https://docs.cognitive3d.com/dashboard/advanced-analysis/
- Filters: https://docs.cognitive3d.com/dashboard/filters/
- Organization Settings: https://docs.cognitive3d.com/dashboard/organization-settings/
- Project Settings: https://docs.cognitive3d.com/dashboard/project-settings/
- Data Export: https://docs.cognitive3d.com/dashboard/data-export/
- Crash Reports: https://docs.cognitive3d.com/dashboard/crash-reports/
- LMS Integration: https://docs.cognitive3d.com/dashboard/lms/

---

## Troubleshooting quick table

Sourced from https://docs.cognitive3d.com/unreal/troubleshooting/ and the pages above. Start with **Enable Logging** in Project Settings and the Output Log for anything not listed here.

| Symptom | Most likely cause |
| --- | --- |
| No sessions at all | no `BP_Cognitive3DActor` in the level |
| No sessions, actor present | scene has no Scene Id, or was never uploaded |
| Sessions exist but replay has no geometry | level not exported and uploaded, or uploaded under a different name |
| Numeric properties cannot be charted or averaged | events sent through the Blueprint variant, which stringifies all values |
| GLTF export crashes | Forward Shading enabled; disable it in Engine > Rendering for the export, then re-enable |
| Textures wrong in replay | complex materials do not translate; make the diffuse output representative |
| Meshes missing from export | TextRenderers are unsupported; skeletal meshes need UE 4.26+ for materials |
| Too much geometry exported | enable "Only Export Selected" and pick objects manually |
| Metahuman exports badly | set Level of Detail to 0; hair and skeletal animation are unsupported |
| Dynamic object has no visual in replay | mesh never exported and uploaded through the Dynamic Object Manager |
| Spawned objects stop being tracked | Id Pool exhausted; the pool has fewer GUIDs than concurrent spawns |
| Gaze and event data not aggregating across sessions | scene does not hold the complete dynamic object ID list |
| Only one level's data recorded in a streaming setup | only the last-loaded level with a valid Scene Id records |
| No comfort, FPS, room size or battery data | the corresponding built-in component was never added |
| Arm length or input data missing | built-in component added but hand dynamic objects are not configured |
| ExitPoll renders but buttons do nothing | no Widget Interaction component on the player or controller |
| ExitPoll voice panel silent | `DefaultEngine.ini` entries for voice not configured |
| Eye tracking absent on Varjo | `VarjoHMD` and `VarjoEyeTracker` not added to `Cognitive3D.Build.cs` |
| Sensor stream missing above 10 Hz | default 10 Hz cap; needs `InitializeSensor` with a frequency, and still will not show in SceneViewer |
| Plugin will not load after an update | folder merged instead of replaced; delete `Plugins/Cognitive3D` and reinstall |
| Dev traffic polluting dashboards | editor sessions are recorded and shown behind a toggle, not excluded; set `development_mode` |

---

## High-staleness surfaces (always verify live)

- Minimum and maximum supported Unreal Engine versions
- Release packaging and per-engine-version packages
- Eye tracking SDK integration list and build requirements
- Built-in component requirements and SDK version floors
- World Partition and HLOD export behaviour
- Dashboard navigation paths
- API key formats and auth examples
- MCP server config examples
- Device and platform feature support
