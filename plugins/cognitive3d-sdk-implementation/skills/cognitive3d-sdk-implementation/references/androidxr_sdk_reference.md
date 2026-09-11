# Cognitive3D Android XR SDK Technical Reference

This file is the routing and API layer for **native Android XR** implementation questions: apps built in Kotlin or Java against **Android XR (Jetpack XR)** or the **Meta Spatial SDK**.

For Unity load `unity_sdk_reference.md`; for Unreal Engine load `unreal_sdk_reference.md`; for native Apple Vision Pro load `visionos_sdk_reference.md`; for browser-based XR load `webxr_sdk_reference.md`. For cross-SDK feature parity at planning time, see `sdk_capability_matrix.md`.

**Primary docs root:** https://docs.cognitive3d.com/
**Android XR docs root:** https://docs.cognitive3d.com/android-xr/get-started/
**Repository:** https://github.com/CognitiveVR/cvr-sdk-android
**Maven artifacts:** `com.cognitive3d:android-xr-sdk`, `com.cognitive3d:meta-spatial-sdk`

## Do not confuse this with the External Android Plugin

The Unity and Unreal references both mention an **External Android Plugin** (https://docs.cognitive3d.com/unity/android-plugin/, https://docs.cognitive3d.com/unreal/android-plugin/). That is a companion that adds Android-level device sensors to a Unity or Unreal app running on an Android headset. It is **not** this SDK.

This file covers the **native Android SDK**: a Kotlin library for apps that have no game engine at all. If the project is a Unity or Unreal app that happens to ship to Quest or Android, it is a Unity or Unreal project and you want that reference instead.

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative.
3. **Escalate to live docs aggressively.** This is the youngest SDK in the skill and its documentation set is the smallest. Several features that exist on the engine SDKs have no Android XR page at all, and the feature list is still growing. Verify before promising anything not documented below.
4. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
5. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.
6. **Establish the platform.** Android XR (Jetpack XR) and Meta Spatial SDK are separate Maven artifacts from the same SDK family. The API surface documented here is shared, but confirm which artifact the project uses before answering anything build- or entity-related.

### Trust hierarchy

1. **Exact live feature page**: e.g. Android XR Custom Events, Dynamic Objects
2. **Installation and integration page**, for config shape and version alignment
3. **Repository and Maven Central**, for current coordinates and version compatibility
4. **Docs portal root** when the question is broad or needs routing
5. **API/Data and MCP docs** for programmatic reads, and for objective configuration writes

### Change-watch anchors (verify live before quoting)

- Android XR docs: https://docs.cognitive3d.com/android-xr/get-started/
- Installation and integration: https://docs.cognitive3d.com/android-xr/installation-integration/
- Repository: https://github.com/CognitiveVR/cvr-sdk-android
- Releases, for AAR builds pinned to specific AndroidX XR alphas: https://github.com/CognitiveVR/cvr-sdk-android/releases
- Supported hardware: https://docs.cognitive3d.com/hardware/

---

## Structural differences worth knowing up front

This is the leanest integration in the skill. It behaves like WebXR in shape (no editor, web app uploads, everything in code) but with a narrower documented feature set and a few hard constraints the other SDKs do not have.

**Five differences that change a plan or an answer:**

1. **ExitPoll has no Android XR documentation.** It is documented for Unity, Unreal, WebXR, visionOS and C++, and absent here. The skill's universal baseline treats "place exit poll hooks early" as a Phase 1 item; on Android XR that item cannot be taken at face value. Verify live before including it, and if it is genuinely unavailable, plan self-report through the app's own UI plus a custom event, and say plainly what is lost: no dashboard-side question management, no changing questions without a release.

2. **Custom events are capped at 10 properties.** No other SDK in this skill documents a limit. The "one event plus properties" advice in `data_strategy.md` still holds, but the budget is finite, so the properties on an event have to earn their place. Flag it whenever a plan row approaches the cap.

3. **Dynamic object association is a property, not a parameter.** The engine SDKs pass a dynamic object ID into the event call. Here the documented approach is a property such as `target_object_id` on an ordinary event. It works, and it costs one of the ten property slots, and the naming has to be consistent by convention because nothing enforces it.

4. **Only FPS is captured automatically.** Controllers, hands and gaze are tracked, but the broad automatic sensor layer Unity has, and the built-in component set Unreal has, do not exist here. Anything else is `recordSensor` and your own sampling loop.

5. **No editor, and no engine-side session lifecycle actor.** Configuration lives in a JSON asset, not a settings window. Scene and object geometry go through the Upload Web App, exactly as on visionOS and WebXR.

| | Unity | Unreal | visionOS | Android XR | WebXR |
| --- | --- | --- | --- | --- | --- |
| Authoring surface | Editor + C# | Editor + Blueprint/C++ | Swift | Kotlin or Java | JS/TS |
| Config | Editor window | Project Settings + `.ini` | code + Info.plist | `assets/cognitive3d.json` | `settings.js` object |
| Scene upload | in-engine | in-engine | Upload Web App | Upload Web App | Upload Web App |
| Automatic sensors | broad | opt-in components | narrow | **FPS only** | moderate |
| ExitPoll | shipped UI | shipped UMG widgets | shipped SwiftUI views | **not documented** | API only, no UI |
| Event property limit | none documented | none documented | none documented | **10 per event** | none documented |

---

## Common implementation mental model

1. **Create/choose a project** in the dashboard
2. **Get the Application Key** from the Project Keys dialog for the runtime config, and the **Developer Key** separately for the Upload Web App
3. **Add the Gradle dependency** for the right platform artifact
4. **Generate `cognitive3d.json`** with the provided Gradle task and fill it in
5. **Import and use `Cognitive3DManager`** throughout the app
6. **Attach session metadata**: participant info, session properties
7. **Record telemetry**: custom events, sensors, dynamic object registrations
8. **Upload scene and dynamic object geometry** through the Upload Web App
9. **Validate in dashboard**: replay, scene/object views, analysis
10. **Troubleshoot** if data is missing

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Custom Event, Sensor, Entity

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Fast route by question type

### "How do I install the SDK?"

- Installation and integration: https://docs.cognitive3d.com/android-xr/installation-integration/

Two artifacts, one per platform. Pick by what the app is built on:

```kotlin
// Android XR (Jetpack XR)
implementation("com.cognitive3d:android-xr-sdk:1.1.0")

// Meta Spatial SDK
implementation("com.cognitive3d:meta-spatial-sdk:1.1.0")
```

Groovy DSL equivalents are documented. **Do not use a dynamic `+` version in production builds**; the docs say so directly, and an XR SDK pinned to specific AndroidX XR alphas is exactly where a floating version bites.

Documented toolchain requirements, all of which move and should be confirmed live rather than quoted from here:

| Requirement | Documented value |
| --- | --- |
| Android Studio | Narwhal (2025.1.1) or later, with AGP 8.11 |
| compileSdk | 36 |
| minSdk | 29 |
| Language | Kotlin 1.9+ or Java 11+ |

**AndroidX XR alpha alignment is the sharp edge.** SDK versions track specific AndroidX XR alpha releases, and Maven Central and the GitHub releases page have carried different alignments (alpha09 via Maven Central, alpha10 as an AAR from GitHub, at time of writing). If a team reports build or runtime incompatibility, this is the first thing to check, and the answer is version-specific enough that it must come from the live releases page rather than from here.

### "How do I configure keys and settings?"

Configuration is a JSON asset at `src/main/assets/cognitive3d.json`, generated from a template by a Gradle task.

| Field | Purpose |
| --- | --- |
| `api_key` | the **Application** API key from the Project Keys dialog |
| `scene_settings.id` | scene ID from the dashboard |
| `scene_settings.version` | scene version number |
| `scene_settings.path` | main activity path, e.g. `com.example.MainActivity` |
| `enable_gaze` | gaze tracking on or off |
| `automatic_send_timer` | seconds between automatic sends |

Plus data cache and snapshot settings; check the live page for the full list.

- The runtime key is the **Application Key**. The **Developer Key** is a separate credential used only by the Upload Web App, and must not go in this file.
- `cognitive3d.json` ships inside the APK, so the Application Key is extractable from a built app. That is inherent to a client-side SDK. Keep the file out of public source control where practical and treat the key as identifying rather than secret. Follow SKILL.md rule 4: never read, echo or log the value.
- **A blank or stale `scene_settings.id` is the most common cause of "sessions exist but replay is empty."** Re-uploading a changed scene increments the version, so `scene_settings.version` has to be updated too or replay silently falls back to old geometry.

### "How do I start and end a session?"

**The docs do not state the session lifecycle explicitly**, which is a real gap rather than an omission in this file. Two documented facts sit near it: `cognitive3d.json` carries a `scene_settings.path` naming the main activity, and the performance page advises initializing the session "when the participant is least likely to notice it, immediately when the application begins or during a scene load." Those are suggestive of an activity-scoped lifecycle, and they are not a specification.

Do not invent an API here. Confirm the exact call against https://docs.cognitive3d.com/android-xr/installation-integration/ or the repository before handing a team code, and if a session-start question comes up, say that this is the one part of the integration worth reading from the live page.

Everything else in this file goes through the same entry point:

```kotlin
import com.cognitive3d.android.Cognitive3DManager
```

### "How do I record custom events?"

- Custom Events: https://docs.cognitive3d.com/android-xr/custom-events/

```kotlin
Cognitive3DManager.sendCustomEvent("file_browser_opened")

val properties = mapOf(
    "model_name" to "nice_home_3d",
    "file_type" to "glb",
    "load_status" to "success",
    "file_size_mb" to 24.5,
    "load_duration_ms" to 3200
)
Cognitive3DManager.sendCustomEvent("model_loaded", properties)
```

Java uses a `HashMap<String, Object>` and is otherwise identical.

Constraints that shape a plan, not just an implementation:

- **Maximum 10 key-value pairs per event.** This is the one hard budget in the skill. When a plan row lists more than a handful of properties, trim it here rather than letting the eleventh silently vanish.
- Property values may be strings, numbers or booleans. Send numbers as numbers, per `queryable_data.md`.
- Keys should be ASCII, and should carry units (`duration_seconds`, `size_mb`), which matches the naming conventions in `data_strategy.md`.
- Events are timestamped automatically and associated with the participant's position, so there is no position argument to pass, unlike WebXR.
- **Dynamic object association is by convention**: put the object's ID in a property such as `target_object_id`. Decide the property name once and write it into the project's conventions document, because nothing in the SDK enforces it and a typo produces an event that simply is not linked.

### "How do I track dynamic objects?"

- Dynamic Objects: https://docs.cognitive3d.com/android-xr/dynamic-objects/

```kotlin
Cognitive3DManager.registerDynamicObject(name, meshName, entity)
```

| Parameter | Meaning |
| --- | --- |
| `name` | this instance's identifier |
| `meshName` | the grouping category, shared by instances of the same object, and **the field that must match the uploaded model name** |
| `entity` | the Jetpack XR `Entity` being tracked |

The docs gloss both fields in terms of the exported model, which reads ambiguously. The behaviour that matters is unambiguous: **`meshName` is the key that groups instances and correlates them with uploaded geometry**, while `name` identifies the individual instance. If aggregation or the dashboard visual is wrong, check `meshName` first.

Once registered, position, rotation, gaze direction and engagement metrics are tracked automatically.

- **`meshName` is what makes aggregate analysis work.** Getting it wrong gives you tracking with no visual, which looks like a failed upload and is not. Instances that should aggregate must share a `meshName`.
- **Controllers and hands are tracked automatically and separately.** Do not register them.
- **No unregister method is documented.** For content that is loaded and unloaded repeatedly, confirm the intended lifecycle against the live page before designing around long-lived registration.
- Good candidates, per the docs: user-loaded content (models, images, documents), interactive tools (pointers, markers, measurement tools), persistent spatial content (annotations, pins, labels). Avoid static environment geometry, transient effects, and non-spatial UI.

Mesh geometry still has to be uploaded separately; see below.

### "How do I upload scenes and object meshes?"

- Scene and object uploads: https://docs.cognitive3d.com/android-xr/scene-object-uploads/
- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/, at https://upload.cognitive3d.com

**Android XR has no visual editor, so both scene and dynamic object geometry go through the Upload Web App**, authenticated with the **Developer Key**. Note the docs' explicit warning: do not use an organization API key (one starting `orgkey-`).

| Upload | Files |
| --- | --- |
| Scene | `scene.gltf` and `scene.bin` required; `screenshot.png` recommended; PNG/JPG textures optional |
| Dynamic object | `{name}.gltf` and `{name}.bin` required; `screenshot.png` or `cvr_object_thumbnail.png` recommended; PNG/JPG/WEBP textures optional |

**The app accepts glTF Separate only (`.gltf` plus `.bin`), not GLB.** This trips up teams whose asset pipeline outputs GLB by default, which on Android XR is most of them. Raise it before they try.

As on every SDK, dynamic object mesh upload is **separate from scene upload** and is the step most often missed.

### "How do I add session and participant metadata?"

- Custom session properties: https://docs.cognitive3d.com/android-xr/custom-session-properties/

```kotlin
Cognitive3DManager.setParticipantId("user_12345")
Cognitive3DManager.setParticipantFullName("John Smith")
Cognitive3DManager.setParticipantProperty("role", "architect")

Cognitive3DManager.setSessionProperty("app_mode", "full_space")
Cognitive3DManager.setSessionProperty("collaboration_enabled", true)
Cognitive3DManager.setSessionProperty("max_participants", 8)
```

The docs draw the split the same way this skill does: participant properties describe the person; session properties describe the environment, app configuration, workspace settings and device state. The guidance in `data_strategy.md` on choosing between them applies unchanged.

**Session tags and a session-name setter are not documented for this SDK.** Session tags are in the skill's primitive table and are a cheap cohorting tool elsewhere. Before putting one in an Android XR plan, either verify the API live or use a session property instead and note the substitution. Analyst-applied tags on the dashboard are unaffected, since those are added after the fact and need no SDK support.

### "How do I record sensors?"

- Custom sensors: https://docs.cognitive3d.com/android-xr/custom-sensors/

```kotlin
Cognitive3DManager.recordSensor("loaded_model_count", 5.0f)

sensorJob = lifecycleScope.launch {
    while (isActive) {
        Cognitive3DManager.recordSensor("loaded_model_count", workspaceManager.loadedModels.size.toFloat())
        delay(5000)
    }
}
```

Java uses a `ScheduledExecutorService` for the same pattern.

- **Float values only**, timestamped by the SDK.
- **FPS is the only documented automatic sensor.** Everything else is yours to sample.
- Documented sampling guidance: about 5 seconds for standard metrics, about 2 seconds for interaction tracking, and no more often than needed. The performance page's worst case used every-frame sensors, which is explicitly not the recommended pattern.
- Naming: lowercase with underscores, units in the name (`latency_ms`, `memory_mb`), and specific rather than general (`left_hand_confidence`, not `hand_confidence`).

### "How do I ask users questions in-app?"

**ExitPoll is not documented for Android XR.** It has pages for Unity, Unreal, WebXR, visionOS and C++, and none here.

Do not assume it is available, and do not port the API from another SDK. Verify live at https://docs.cognitive3d.com/android-xr/get-started/ before including it in a plan. If it is genuinely unsupported, the honest substitute is the app's own UI writing answers as a custom event, and the trade-off should be stated rather than glossed: question wording is then baked into the release, dashboard-side question management and ExitPoll Results reporting are unavailable, and changing a question needs a new build.

This matters more than it sounds, because "place exit poll hooks early even if the questions are not ready" is one of the twelve universal baseline items in `data_strategy.md`. On Android XR that item needs checking rather than assuming.

### "How do I create or change objectives?"

Objectives are a platform resource and behave identically regardless of SDK, so they work here even though the SDK has no objective API of its own.

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

Two routes, both fully supported: the dashboard (no API key, right default when a non-developer owns definitions) or the MCP server (`create_objective`, `update_objective`, `delete_objective`, needs a write-enabled organization key). Ask which the team wants; always dry-run first.

**Objective platform constraints, identical across SDKs:**

- `sequential` cannot be changed after save.
- Writing steps asynchronously re-scores roughly the last 30 days of sessions and nothing older.
- `name` is capped at 32 characters.
- Gaze and fixation steps reference dynamic object IDs, which are per-project, so such objectives cannot be copied between projects verbatim.
- `delete_objective` is a **permanent cascade** with no restore path. Confirm intent explicitly.

Objectives built on ExitPoll answers are the exception: without ExitPoll, those steps have no source on this SDK.

### "What about performance?"

- Performance: https://docs.cognitive3d.com/android-xr/performance/

Measured on a Samsung Galaxy XR (Snapdragon XR2+ Gen 2). Note the published figures were taken on SDK 1.0.2 while the current release is newer, so treat them as indicative of the SDK's general weight rather than as a measurement of the version a team will ship: about **-0.28% framerate** (71.58 to 71.38 fps) and about **+5.73% CPU** (60.54% to 64.01%). That was a deliberate worst case, with custom sensors every frame and events every second; typical integrations are described as materially lighter.

Two documented levers:

- **Increase `automatic_send_timer`** to send fewer, larger requests when network cost shows up in a profile.
- **Initialize the session when the participant will not notice**, at app start or during a scene load.

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/

Two platform families share this SDK: **Android XR (Jetpack XR)** devices, and **Meta Spatial SDK** apps. Confirm which artifact the project uses; the surrounding platform APIs differ even where the Cognitive3D API does not.

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

Identical across SDKs. Route API query construction to the `cognitive3d-public-api` skill.

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- MCP is a data and configuration layer, not an instrumentation layer. It is SDK-agnostic, so everything in SKILL.md about MCP-based validation applies unchanged, and it is especially useful here: with the thinnest tooling of any target, programmatic verification that events and properties actually arrived saves the most time.

---

## Android XR SDK directory

- Get started: https://docs.cognitive3d.com/android-xr/get-started/
- Installation and integration: https://docs.cognitive3d.com/android-xr/installation-integration/
- Custom session properties: https://docs.cognitive3d.com/android-xr/custom-session-properties/
- Custom events: https://docs.cognitive3d.com/android-xr/custom-events/
- Custom sensors: https://docs.cognitive3d.com/android-xr/custom-sensors/
- Dynamic objects: https://docs.cognitive3d.com/android-xr/dynamic-objects/
- Scene and object uploads: https://docs.cognitive3d.com/android-xr/scene-object-uploads/
- Performance: https://docs.cognitive3d.com/android-xr/performance/
- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/

**Features with no Android XR page at time of writing** — verify live before promising any of them: ExitPoll, remote controls, local cache, media and 360, multiplayer, session tags, session name, audio recording, Active Session View.

---

## Dashboard directory

Dashboard surfaces are SDK-agnostic. Listed in full so an Android XR engagement never needs another reference file.

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

| Symptom | Most likely cause |
| --- | --- |
| No sessions at all | `api_key` missing or wrong in `cognitive3d.json`, or the session never initialized |
| Sessions exist but replay has no geometry | `scene_settings.id` blank, or the scene was never uploaded |
| Replay shows old geometry after a re-upload | `scene_settings.version` not bumped in the config |
| Upload Web App rejects the model | GLB supplied; it accepts glTF Separate only (`.gltf` plus `.bin`) |
| Upload Web App rejects the key | organization key used (`orgkey-`); it needs the Developer Key |
| Dynamic object tracked but invisible in replay | mesh never uploaded, or `meshName` does not match the uploaded model name |
| Dynamic object instances do not aggregate | `meshName` differs per instance; the grouping key is `meshName`, not `name` |
| Event property silently missing | more than 10 key-value pairs on the event |
| Event not linked to an object | the dynamic object ID property name does not match the project's convention |
| Numeric property cannot be charted | sent as a string; the map takes typed values, so pass numbers as numbers |
| No gaze data | `enable_gaze` false in the config |
| No sensor data beyond FPS | only FPS is automatic; everything else needs `recordSensor` plus a sampling loop |
| Data arrives in large delayed bursts during testing | `automatic_send_timer` set high; lower it for validation, restore afterwards |
| Build or runtime incompatibility | SDK version not aligned with the project's AndroidX XR alpha; check the releases page |
| Dev traffic polluting dashboards | no editor-session concept exists here, so nothing is filtered; the plan's dev/prod session property was never set |

---

## High-staleness surfaces (always verify live)

- Whether ExitPoll, session tags, remote controls, local cache, media or multiplayer have landed for this SDK
- Maven coordinates and current version
- AndroidX XR alpha alignment, and which builds are on Maven Central versus GitHub releases
- Android Studio, AGP, compileSdk and minSdk floors
- Session lifecycle API
- Dynamic object unregistration behaviour
- Dashboard navigation paths
- API key formats and auth examples
- Device and platform feature support
