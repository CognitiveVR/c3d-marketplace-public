# Cognitive3D visionOS SDK Technical Reference

This file is the routing and API layer for **native Apple Vision Pro** implementation questions: apps written in Swift against visionOS, RealityKit and SwiftUI.

For Unity load `unity_sdk_reference.md`; for Unreal Engine load `unreal_sdk_reference.md`; for native Android XR load `androidxr_sdk_reference.md`; for browser-based XR load `webxr_sdk_reference.md`. For cross-SDK feature parity at planning time, see `sdk_capability_matrix.md`.

**Primary docs root:** https://docs.cognitive3d.com/
**visionOS docs root:** https://docs.cognitive3d.com/visionos/get-started/
**Repository:** https://github.com/CognitiveVR/c3d-sdk-visionOS

## Apple Vision Pro is not always a visionOS project

Vision Pro apps are built two ways, and only one of them uses this SDK.

- **Native visionOS**, in Swift with RealityKit and SwiftUI. That is this file.
- **Unity**, using Unity's PolySpatial/visionOS support. That is a **Unity project** and uses `unity_sdk_reference.md`, even though it ships to a Vision Pro.

"We're building for Vision Pro" therefore does not settle which reference to load. Establish the toolchain first: an `.xcodeproj`/`Package.swift` with Swift sources means native; an `Assets/` folder and a Unity project means Unity. If both are present, it is a Unity project with an Xcode build output, and Unity wins. Step 0 in SKILL.md carries the same tiebreak.

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative.
3. **Escalate to live docs when freshness matters.** The SDK is young and its version notes matter: some behaviour is documented as arriving in specific point releases and defaulting off.
4. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
5. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.
6. **Lead with the gaze limitation whenever attention comes up.** See immediately below. It is the fact most likely to make a plan wrong on this platform, and it is not obvious from anything else in the skill.

### Change-watch anchors (verify live before quoting)

- visionOS docs: https://docs.cognitive3d.com/visionos/get-started/
- Integrating the SDK: https://docs.cognitive3d.com/visionos/integrating-sdk/
- Repository: https://github.com/CognitiveVR/c3d-sdk-visionOS
- Supported hardware: https://docs.cognitive3d.com/hardware/

---

## The one thing to get right: there is no eye tracking

**visionOS does not expose eye-tracking rays to applications.** This is an Apple platform privacy restriction, not a Cognitive3D limitation and not something a future SDK release can lift. The docs state it plainly: "this is not eye tracking."

What the SDK records instead is the **HMD forward direction as the gaze signal**. That is a head-direction proxy. It is genuinely useful, and it is not the same measurement the other SDKs produce on eye-tracked hardware.

The consequences, which belong in the plan rather than in a footnote:

- **"What did they look at" becomes "what were they facing."** On Vision Pro a person can read an entire panel, or study an object, with their head still. Head-direction gaze misses that entirely, and it also reports attention on whatever is centred while the eyes are elsewhere.
- **Fixation-based analysis does not mean what it means elsewhere.** Do not carry a fixation recommendation across from a Unity or Unreal plan without restating what it measures here.
- **Dwell and heatmaps still work**, and still tell you something real about orientation and body positioning. Describe them as head-direction attention, not eye attention, in the plan and in the dashboard readout the team will look at.
- **If the team's core question is genuinely about eye attention**, say so early. The honest answer is that Vision Pro cannot answer it for any app, from any vendor, and the question has to be reframed around interaction, dwell-by-orientation and self-report.

This is the single most important thing to establish during discovery on a visionOS project, because an exploration- or evaluation-archetype plan built on the usual gaze assumptions will look fine and quietly measure something else.

---

## Other structural differences

1. **Custom event properties are `[String: String]`.** Session properties are typed (String, Bool, Numeric); event properties are not. A numeric event property arrives as text and cannot be averaged, charted or filtered numerically. This is the same failure mode as Unreal's Blueprint variant, and it has no C++-style escape hatch here: if a number must be queryable, put it on the session, or accept the loss and say so.
2. **The SDK ships the ExitPoll UI.** Six SwiftUI question views plus a view model, and question sets cache locally so surveys work offline. This is the strongest ExitPoll story outside Unity, and a real reason to place hooks early.
3. **Installation is a local Swift package**, not a remote package URL: the framework and `Package.swift` are copied into a subfolder and added as a local dependency.
4. **Session start and end are async and return a result.** Unlike the engine SDKs there is no actor or component whose mere presence starts a session.
5. **No editor.** Scene and dynamic object geometry go through the Upload Web App, as on Android XR and WebXR.

| | Unity | Unreal | visionOS | Android XR | WebXR |
| --- | --- | --- | --- | --- | --- |
| Authoring | Editor + C# | Editor + Blueprint/C++ | Swift, SwiftUI, RealityKit | Kotlin or Java | JS/TS |
| Install | UPM git URL | release into `Plugins/` | local Swift package | Gradle dependency | `npm install` |
| Event property typing | typed | typed in C++, stringified in Blueprint | **string-only** | typed | typed |
| Eye tracking | on supported hardware | on supported hardware | **never, platform restriction** | verify live | hardware dependent |
| ExitPoll UI | shipped | shipped (UMG) | **shipped (SwiftUI)** | not documented | none, build it yourself |
| Local cache | yes | yes | yes | not documented | not documented |

---

## Common implementation mental model

1. **Create/choose a project** in the dashboard
2. **Get the Application Key** for the runtime config, and the **Developer Key** separately for the Upload Web App
3. **Add the SDK** as a local Swift package
4. **Configure `CoreSettings` and `SceneData`**, then initialize at app startup
5. **Start the session** (async) and observe lifecycle
6. **Attach session metadata**: participant info, session properties
7. **Record telemetry**: custom events, dynamic object registration, sensors
8. **Upload scene and dynamic object geometry** through the Upload Web App
9. **Validate in dashboard**: replay, scene/object views, analysis
10. **Troubleshoot** if data is missing

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Custom Event, ExitPoll, Hook, Question Set, Entity, Immersive Root

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Fast route by question type

### "How do I install the SDK?"

- Integrating the SDK: https://docs.cognitive3d.com/visionos/integrating-sdk/
- Repository: https://github.com/CognitiveVR/c3d-sdk-visionOS

Distributed as a **local Swift package**: copy the framework and `Package.swift` into a subfolder of the project, then add it in Xcode as a local Swift package dependency. There is no remote package URL to paste, which surprises Swift developers who expect one.

The repository builds an xcframework, with scripts for framework and debug-symbol builds, so a team that wants to track SDK source rather than a drop-in binary has that route.

Minimum visionOS and Xcode versions are not stated in the docs. Do not invent them; check the repository's `Package.swift` or ask.

### "How do I configure and initialize?"

```swift
import Cognitive3DAnalytics

@main
struct MyApp: App {
    init() {
        cognitiveSDKInit()
    }
}
```

Configuration builds a `SceneData` and a `CoreSettings`:

```swift
let sceneData = SceneData(
    sceneName: "your-scene-name",
    sceneId: "your-scene-id",
    versionNumber: 1,
    versionId: 1234
)

let core = Cognitive3DAnalyticsCore.shared
let settings = CoreSettings()
settings.defaultSceneName = sceneData.sceneName
settings.allSceneData = [sceneData]
settings.apiKey = Bundle.main.object(forInfoDictionaryKey: "APPLICATION_API_KEY") as? String ?? ""
```

Settings worth knowing during planning, not just implementation:

| Setting | Why it matters |
| --- | --- |
| `loggingLevel`, `isDebugVerbose` | first stop for any missing-data report |
| `sensorAutoSendInterval` | batching; lower it for validation, restore afterwards |
| `isHandTrackingRequired` | gates hand tracking availability |
| `shouldEndSessionOnBackground` | decides whether backgrounding ends the session, which changes what a "session" means in the data |

- The runtime key is the **Application Key**, read here from an Info.plist entry. The **Developer Key** is separate and belongs only to the Upload Web App.
- `scene_settings` equivalents live in `SceneData`: a blank or stale `sceneId`, or a `versionNumber` not bumped after re-upload, is the usual cause of sessions with no replay geometry.
- Follow SKILL.md rule 4: never read, echo or log the key value, whatever file it sits in.

### "How do I start and end a session?"

```swift
let didStart = await Cognitive3DAnalyticsCore.shared.startSession()
let didEnd   = await Cognitive3DAnalyticsCore.shared.endSession()
```

Both are **async and return a Bool**, so a failed start is observable rather than silent. Check it; a swallowed `startSession()` result is the visionOS equivalent of a missing session actor.

From v1.0.1 there are **Combine publishers** over a `SessionEvent` enum with `.started(sessionId)` and `.ended(sessionId, state)` cases, which is the clean way to react to lifecycle without polling. A `.observeCognitive3DScenePhase()` modifier wires SwiftUI scene phase to the session.

`shouldEndSessionOnBackground` decides whether backgrounding ends a session. Settle it deliberately during planning: it changes session counts, session duration distributions, and whether a multi-sitting workflow reads as one session or several. Whichever way the team chooses, record the choice in the conventions document so the numbers are interpretable later.

### "How do I record custom events?"

- Custom Events: https://docs.cognitive3d.com/visionos/custom-events/

```swift
let core = Cognitive3DAnalyticsCore.shared

let event = CustomEvent(
    name: "step_completed",
    properties: ["step_name": "seal_valve"],
    core: core
)
let success = event.send()
```

Full initializer:

```swift
CustomEvent(
    name: String,
    properties: [String: String]?,
    dynamicObjectId: String?,
    core: Cognitive3DAnalyticsCore
)
```

- `send()` records and transmits; `send(position)` supplies explicit coordinates. **Position is optional**: events record `x, y, z` automatically, falling back to the HMD position.
- **Duration comes free from deferring the send.** Construct the event when the activity starts, call `send()` when it ends, and duration is appended to the payload. This is the idiomatic way to get durations here and is easy to miss.
- **`dynamicObjectId` is a first-class parameter**, unlike Android XR where the link is a property convention.

**The property-typing trap.** `properties` is `[String: String]`. Numbers and booleans become text. A `duration_seconds` sent this way cannot be averaged, charted or numerically filtered, and nothing on the dashboard flags it. Two mitigations, in order of preference:

1. Put values that must be numeric on the **session** instead (`setSessionProperty` is typed), where the question is a per-session one.
2. Accept the loss for genuinely per-event values, and say so in the plan rather than letting an analyst discover it.

When auditing an existing visionOS integration, check this before concluding the instrumentation is sound.

### "How do I track dynamic objects?"

- Dynamic Objects: https://docs.cognitive3d.com/visionos/dynamic-objects/
- Uploading dynamics: https://docs.cognitive3d.com/visionos/uploading-dynamics/

Three source files carry the integration: `DynamicObjectSystem.swift`, `DynamicComponent.swift`, `ImmersiveView+DynamicObject.swift`.

Registration is a RealityKit ECS pattern:

```swift
DynamicComponent.registerComponent()
DynamicObjectSystem.registerSystem()

core.entity = immersiveRoot          // required: the root to traverse
registerDynamicObject(id:name:mesh:)
registerHand(id:isRightHand:)        // tags: hand_left, hand_right
```

`DynamicObjectSystem` runs each frame, finds entities carrying `DynamicComponent`, and records updates only when thresholds and rates are met. Position, orientation, scale and object properties are tracked, and enabled/disabled states are emitted when an entity is removed or deactivated.

Two gotchas worth raising before a team builds around them:

- **SwiftUI windows are not in the RealityKit hierarchy**, so they need a separate `PositionTrackerView` with the transform mirrored manually. A plan that treats a 2D window panel as just another dynamic object underestimates it.
- **`performMaintenanceCleanup()`** prunes stale state and is recommended for extended sessions. Long-session projects (training, productivity, anything multi-hour) should have it in the plan rather than discovering drift later.

Mesh geometry still has to be uploaded separately; see below.

### "How do I upload scenes and object meshes?"

- Uploading scenes: https://docs.cognitive3d.com/visionos/uploading-scenes/
- Uploading dynamics: https://docs.cognitive3d.com/visionos/uploading-dynamics/
- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/, at https://upload.cognitive3d.com

**visionOS has no visual editor**, so both scene and dynamic object geometry go through the Upload Web App with the **Developer Key**. Not an organization key (one starting `orgkey-`).

- The first upload generates a **Scene ID**; reuse it for later uploads of the same scene, or you create duplicate scenes. Re-upload is only needed when assets actually change.
- **A scene must exist before dynamic objects can be uploaded**, since objects attach to it.
- Per the Upload Web App docs, it accepts **glTF Separate** (`.gltf` plus `.bin`) and not GLB. The visionOS pages do not restate the format, so confirm live if the team's pipeline is GLB-first, which most are.

As on every SDK, dynamic object mesh upload is **separate from scene upload** and is the step most often missed.

### "How do I add session and participant metadata?"

- Session property: https://docs.cognitive3d.com/visionos/session-property/

```swift
Cognitive3DAnalyticsCore.shared.setSessionProperty(key: "testSession", value: true)

Cognitive3DAnalyticsCore.shared.setParticipantId("11112222")
Cognitive3DAnalyticsCore.shared.setParticipantFullName("Jack Jones")
Cognitive3DAnalyticsCore.shared.setParticipantProperty(keySuffix: "someAttribute", value: "someValue")
```

- **Session properties are typed**: String, Bool and Numeric all work, and only the latest value per key is kept. This is the escape hatch for numbers that custom event properties cannot carry.
- Session properties are **cleared when a session ends**, so anything that must describe a later session has to be set again, or belongs on the participant.
- Custom participant properties are transmitted with keys prefixed `c3d.participant.`, which is why the parameter is `keySuffix` rather than a full key.
- **Session tags are not documented for this SDK.** Use a session property instead, and note the substitution. Analyst-applied dashboard tags are unaffected.

### "What is tracked automatically?"

- Tracking the HMD: https://docs.cognitive3d.com/visionos/tracking-hmd/
- HMD height: https://docs.cognitive3d.com/visionos/hmd-height/

| Captured | Notes |
| --- | --- |
| Position | device pose, from a shared ARKit world-tracking session via `ARSessionManager` |
| Pitch | head orientation |
| FPS | frame rate |
| Battery level | device power |
| HMD height | sampled Y-position, median plus a forehead offset, stored as `c3d.participant.height` in centimetres. No setup needed |
| Gaze | **HMD forward direction only.** Not eye tracking. See the section above |

- **Yaw is supported from v1.0.1 but disabled by default.** If a plan depends on horizontal head orientation, enabling it is an explicit step, not an assumption.
- **Roll is not recorded** as a built-in sensor.
- `ARSessionManager` keeps a single shared ARKit session, so the SDK does not fight the app's own world tracking.

HMD height is a small, free win worth mentioning: participant stature arrives with no instrumentation, which is useful for ergonomics and reach analysis in training and productivity contexts.

### "How do I ask users questions in-app?"

- ExitPoll: https://docs.cognitive3d.com/visionos/exitpoll/
- ExitPoll view model: https://docs.cognitive3d.com/visionos/exitpoll-view-model/
- ExitPoll SwiftUI views: https://docs.cognitive3d.com/visionos/exitpoll-swiftui-views/
- MCP ExitPoll tools: https://docs.cognitive3d.com/mcp-server/exitpoll/

**Hybrid task**, and the app side is well supported. Hooks and question sets live on the platform; the SDK provides both the logic and the UI.

`ExitPollSurveyViewModel`, used as a `@StateObject`:

| Method | Purpose |
| --- | --- |
| `fetchSurvey(hook:)` | fetch by hook; returns `Result<Void, ExitPollSurveyError>` |
| `setAnswer(_:forQuestionAt:)` | record one response by question index |
| `submitSurvey()` | submit all answers; returns `Result<Void, ExitPollSurveyError>` |

`surveyQuestions` holds the fetched questions; `isLoading` and `errorMessage` carry state. `ExitPollSurveyError` covers network failure, invalid response and empty survey.

Six question types, each with a shipped SwiftUI view:

| Type | View |
| --- | --- |
| Boolean | `BooleanQuestionView` |
| Happy/Sad | `HappySadQuestionView` |
| Thumbs up/down | `ThumbsQuestionView` |
| Multiple choice (2 to 4 options) | `MultipleChoiceQuestionView` |
| Scale | `ScaleQuestionView` |
| Voice | `VoiceQuestionView` |

- **Voice needs `NSMicrophoneUsageDescription`** in Info.plist and records base64 audio with a configurable time limit. Voice capture is privacy-sensitive: apply the usual `data_strategy.md` guidance rather than enabling it because it is available.
- The views are reference implementations, and the docs invite building custom ones from them, so a team with its own design system is not stuck.
- **Question sets cache locally**, so surveys work offline: the most recent set is shown and answers are cached until reconnection. That is a genuine advantage for field, training and travel deployments.

**ExitPoll platform constraints are identical across SDKs**, because they belong to the platform. Flag these before any write:

- Question set versions are immutable: editing creates a new version. Removal is archival only.
- **A new version does not move existing hooks.** Hooks stay on the version they were assigned until reassigned with `update_exitpoll_hook`.
- Hooks cannot be deleted, only unassigned, and a hook with no question set assigned is skipped silently at runtime.

Question sets are configurable **either** on the dashboard **or** via MCP. MCP writes need a write-enabled organization key with an org- or project-admin role. Ask which route the team wants (route-selection gates are in SKILL.md Step 7).

### "How do I create or change objectives?"

Objectives are a platform resource and behave identically regardless of SDK.

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

Two routes, both fully supported: the dashboard, or the MCP server (`create_objective`, `update_objective`, `delete_objective`, needs a write-enabled organization key). Ask which the team wants; always dry-run first.

**Objective platform constraints, identical across SDKs:**

- `sequential` cannot be changed after save.
- Writing steps asynchronously re-scores roughly the last 30 days of sessions and nothing older.
- `name` is capped at 32 characters.
- Gaze and fixation steps reference dynamic object IDs, which are per-project, so such objectives cannot be copied between projects verbatim.
- `delete_objective` is a **permanent cascade** with no restore path. Confirm intent explicitly.

**Gaze-step objectives deserve a second look on this platform.** They will build and score, but they score head direction, not eye attention. An objective named "looked at the safety notice" means "faced the safety notice" here. Name it accordingly.

### "What happens when the device is offline?"

- Local cache: https://docs.cognitive3d.com/visionos/local-cache/

Supported and well specified, which is not true of every SDK in this skill. `DataCacheSystem` and `DualFileCache` hold requests when upload is not possible and send them when connectivity returns.

- Cache cap **100 MB** by default.
- Two files, `data_read` and `data_write`, in the app's Documents directory.
- Retry uses exponential backoff: 10s, 20s, 40s, 80s, 160s, 300s maximum, resetting after a success.
- If file-based cache initialization fails, an in-memory fallback covers the current run.

Worth naming in any plan involving field use, travel, hospitals, factory floors or anywhere wifi is unreliable.

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

Identical across SDKs. Route API query construction to the `cognitive3d-public-api` skill.

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- MCP is a data and configuration layer, not an instrumentation layer. It is SDK-agnostic, so everything in SKILL.md about MCP-based validation applies unchanged.

---

## visionOS SDK directory

- Get started: https://docs.cognitive3d.com/visionos/get-started/
- Integrating the SDK: https://docs.cognitive3d.com/visionos/integrating-sdk/
- Tracking the HMD: https://docs.cognitive3d.com/visionos/tracking-hmd/
- HMD height: https://docs.cognitive3d.com/visionos/hmd-height/
- Custom events: https://docs.cognitive3d.com/visionos/custom-events/
- Session property: https://docs.cognitive3d.com/visionos/session-property/
- Dynamic objects: https://docs.cognitive3d.com/visionos/dynamic-objects/
- ExitPoll: https://docs.cognitive3d.com/visionos/exitpoll/
- ExitPoll view model: https://docs.cognitive3d.com/visionos/exitpoll-view-model/
- ExitPoll SwiftUI views: https://docs.cognitive3d.com/visionos/exitpoll-swiftui-views/
- Uploading scenes: https://docs.cognitive3d.com/visionos/uploading-scenes/
- Uploading dynamics: https://docs.cognitive3d.com/visionos/uploading-dynamics/
- Local cache: https://docs.cognitive3d.com/visionos/local-cache/
- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/

**Features with no visionOS page at time of writing** — verify live before promising any of them: eye-tracked fixations (platform-blocked, not a documentation gap), session tags, session name, remote controls, media and 360, multiplayer, Active Session View, custom sensors as a first-class API.

---

## Dashboard directory

Dashboard surfaces are SDK-agnostic. Listed in full so a visionOS engagement never needs another reference file.

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
| No sessions at all | `startSession()` returned false and the result was not checked, or the API key is missing from Info.plist |
| Sessions exist but replay has no geometry | `sceneId` blank in `SceneData`, or the scene was never uploaded |
| Replay shows old geometry after a re-upload | `versionNumber` not bumped in `SceneData` |
| Duplicate scenes on the dashboard | a new Scene ID generated instead of reusing the existing one |
| Upload Web App rejects the model | GLB supplied; it accepts glTF Separate (`.gltf` plus `.bin`) |
| Upload Web App rejects the key | organization key used (`orgkey-`); it needs the Developer Key |
| Numeric property cannot be charted | sent as a custom event property, which is string-only; put it on the session instead |
| Dynamic object tracked but invisible in replay | mesh never uploaded through the web app |
| Dynamic object never tracked | `core.entity` not set to the immersive root, or the component/system never registered |
| SwiftUI window content not tracked | windows are outside the RealityKit hierarchy; needs `PositionTrackerView` with mirrored transforms |
| Tracking drifts or bloats in long sessions | `performMaintenanceCleanup()` never called |
| Hands not tracked | `isHandTrackingRequired` not set, or `registerHand` never called |
| No yaw data | supported from v1.0.1 but disabled by default |
| Gaze data looks wrong or attention seems off | it is head direction, not eye tracking; the measurement is working as designed |
| Sessions end unexpectedly, or never split | `shouldEndSessionOnBackground` set opposite to what the team assumed |
| ExitPoll voice question fails | `NSMicrophoneUsageDescription` missing from Info.plist |
| Data arrives late in bursts | `sensorAutoSendInterval`, or the cache backing off after a network failure |
| Dev traffic polluting dashboards | no editor-session concept exists here; the plan's dev/prod session property was never set |

---

## High-staleness surfaces (always verify live)

- Minimum visionOS and Xcode versions, and the deployment target
- Whether yaw has become enabled by default, and whether roll has landed
- Whether session tags, remote controls, media or multiplayer have arrived
- Custom sensor API availability
- Package distribution method, if it moves from local package to a remote one
- Scene and object export formats accepted by the Upload Web App
- Dashboard navigation paths
- API key formats and auth examples
- Device and platform feature support
