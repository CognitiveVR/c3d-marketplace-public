# Cognitive3D C++ SDK Technical Reference

This file is the routing and API layer for **C++ SDK** implementation questions: custom engines and native applications that integrate Cognitive3D directly in C++ with no Unity, Unreal, Swift, Kotlin or browser layer in between.

For Unity load `unity_sdk_reference.md`; for Unreal Engine load `unreal_sdk_reference.md`; for native Apple Vision Pro load `visionos_sdk_reference.md`; for native Android XR load `androidxr_sdk_reference.md`; for browser-based XR load `webxr_sdk_reference.md`. For cross-SDK feature parity at planning time, see `sdk_capability_matrix.md`.

**Primary docs root:** https://docs.cognitive3d.com/
**C++ docs root:** https://docs.cognitive3d.com/cpp/get-started/
**Repository:** https://github.com/CognitiveVR/cvr-sdk-cpp (default branch `development`)

## Unreal C++ is not the C++ SDK

The Unreal reference talks about a "C++ authoring surface". That is C++ written against the **Unreal plugin**, and it is an Unreal project. This file covers the standalone **C++ SDK**: a source library (`namespace cognitive`, class `CognitiveVRAnalyticsCore`) for engines and applications that have no Cognitive3D plugin of their own. If the project has a `.uproject`, it is an Unreal project and you want that reference, however much C++ it contains.

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth, and the public headers in the repository as the tiebreak when the docs are silent: the SDK is small enough that the headers are readable in one sitting, and they document more than the docs site does.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists. There are seven: get started, gaze, sensors, ExitPoll, dynamic objects, events, advanced.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative, and the header as authoritative where the page is silent.
3. **Know what kind of SDK this is before planning.** It is a **recording and batching library, not a capture library**. It captures nothing on its own: every gaze sample, every sensor value, every dynamic object snapshot and every HMD pose is pushed in by the host application, and even the HTTPS transport is a function pointer the host supplies. Plans that assume an automatic layer are wrong here from the first row.
4. **Treat it as maintenance-mode.** The repository's last activity predates every other SDK's current release by a wide margin, it carries no release tags, and its settings enum still lists Vive, Rift and Gear as the HMD choices. Nothing here should be quoted as current without checking the repository's recent history. The cross-engine freshness gate in cvr-cortex tracks it by commit sha rather than by version for that reason.
5. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
6. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.

### Trust hierarchy

1. **Exact live feature page**: e.g. C++ Dynamic Objects, C++ Events
2. **Public headers in the repository**, for signatures, overloads, defaults and the behaviours the docs do not mention
3. **Get started page**, for the construction and session shape
4. **Docs portal root** when the question is broad or needs routing
5. **API/Data and MCP docs** for programmatic reads, and for objective configuration writes

### Change-watch anchors (verify live before quoting)

- C++ docs: https://docs.cognitive3d.com/cpp/get-started/
- Repository, `development` branch: https://github.com/CognitiveVR/cvr-sdk-cpp
- Core header, which is the real API index: `CognitiveVRAnalytics/CognitiveVRAnalytics/CognitiveVRAnalytics.h`
- Settings header: `CognitiveVRAnalytics/CognitiveVRAnalytics/coresettings.h`
- Supported hardware: https://docs.cognitive3d.com/hardware/

---

## Structural differences worth knowing up front

This is the thinnest integration in the skill, thinner than Android XR. Android XR at least captures FPS, hands, controllers and gaze on its own; the C++ SDK captures nothing, and expects the host to drive it every frame.

**Six differences that change a plan or an answer:**

1. **There is no automatic layer at all.** The only things the SDK sends without being told are the session begin and end records and four identity fields: device id, user name, session name and HMD type. Head pose, gaze, controllers, hands, FPS, battery, boundary, biometrics: all of it is `RecordGaze`, `RecordSensor` or `RecordDynamic` calls the host makes. Every "captured automatically" row in `queryable_data.md` is a row the host has to produce here, or go without.

2. **The host owns the network.** `CoreSettings.webRequest` is a function pointer. The SDK builds URLs, headers and JSON bodies and hands them to that function; the host sends them with whatever HTTPS stack it has and calls the response callback. No request leaves the process otherwise. This makes "data not arriving" a two-sided debugging problem, and it means there is **no local cache or offline queue**: if the host's request fails, the batch is gone.

3. **The host owns the clock and the loop.** Gaze is expected at ten samples per second (`GazeInterval` must stay at `0.1f` unless agreed with Cognitive3D), and the SDK does not poll anything. The host calls `RecordGaze` from its own update loop at that cadence, decides when to snapshot dynamic objects, and calls `SendData` when it wants outstanding batches flushed.

4. **Gaze is whatever the host passes in.** `RecordGaze` takes an HMD position, an HMD rotation and an optional gaze point. Whether that gaze point came from eye tracking or from a forward ray is the host's business and invisible to the SDK. The same holds for fixations: the SDK has a `RecordFixation` call, but the host computes the fixation. In planning terms the eye-attention question is answered by the host's hardware and code, not by the SDK.

5. **Custom events need a position and accept typed JSON.** As on WebXR, `RecordEvent` requires a position vector. Properties are an `nlohmann::json` object, so numbers stay numbers and there is no property cap. Dynamic object association is a parameter, not a convention.

6. **ExitPoll is API-only, with voice answers.** The SDK fetches a question set by hook, lets the host add answers and sends them. It renders nothing. Question types include a voice type whose answer is a base64 `.wav` string, which no other SDK exposes as a plain API.

| | Unity | Unreal | visionOS | Android XR | WebXR | C++ |
| --- | --- | --- | --- | --- | --- | --- |
| Authoring surface | Editor + C# | Editor + Blueprint/C++ | Swift | Kotlin or Java | JS/TS | C++ 11+ |
| Config | Editor window | Project Settings + `.ini` | code + Info.plist | `assets/cognitive3d.json` | `settings.js` object | `CoreSettings` in code |
| Transport | SDK | SDK | SDK | SDK | SDK | **host-supplied function pointer** |
| Scene upload | in-engine | in-engine | Upload Web App | Upload Web App | Upload Web App | Upload Web App |
| Automatic sensors | broad | opt-in components | narrow | FPS only | moderate | **none** |
| ExitPoll | shipped UI | shipped UMG widgets | shipped SwiftUI views | not documented | API only, no UI | API only, no UI, voice supported |
| Local cache | yes | yes | yes | not documented | not documented | **none** |

---

## Common implementation mental model

1. **Create/choose a project** in the dashboard
2. **Get the Application Key** from the Project Keys dialog, and the **Developer Key** separately for the Upload Web App
3. **Add the source** to the project (or build the static library with CMake) and include `CognitiveVRAnalytics.h`
4. **Implement the web request callback** against the host's HTTPS stack
5. **Fill a `CoreSettings`**: callback, key, scene list, HMD type, batch sizes
6. **Construct `CognitiveVRAnalyticsCore`**, set user and device names, set session properties, `StartSession()`
7. **Drive it from the update loop**: `RecordGaze` at 10 Hz, `RecordDynamic` on movement past a threshold, `RecordSensor` on your own schedule, `RecordEvent` from game logic
8. **Flush and end**: `SendData()` when batches should go, `EndSession()` on exit
9. **Upload scene and dynamic object geometry** through the Upload Web App
10. **Validate in dashboard**: replay, scene/object views, analysis

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Custom Event, Sensor, Fixation, Engagement, Hook, Question Set

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Fast route by question type

### "How do I install the SDK?"

- Get started: https://docs.cognitive3d.com/cpp/get-started/

Two documented routes: add the source files to the project directly, or build a static library with the repository's `CMakeLists.txt`. Either way:

```cpp
#include "cognitive/CognitiveVRAnalytics.h"
```

| Requirement | Documented value |
| --- | --- |
| Language standard | C++11 or newer (`make_unique_cognitive` is provided for C++11; use `std::make_unique` on C++14+) |
| Bundled dependency | nlohmann json, vendored as `json.hpp` |
| Host-provided dependency | an HTTPS client. The docs' pseudocode uses curl; the SDK itself does not link one |
| Platforms | not stated. The repository carries a Visual Studio solution and a CMake file |

There are **no release tags and no package registry entry**. Teams pin a commit. Say so during discovery, because it shapes how they will track updates.

### "How do I configure keys and settings?"

Configuration is a `cognitive::CoreSettings` object filled in code before construction. The header is the full reference; the fields that matter:

| Field | Purpose | Default |
| --- | --- | --- |
| `webRequest` | **the host's HTTPS function**; the SDK cannot send without it | `nullptr` |
| `APIKey` | the **Application** API key from the Project Keys dialog | placeholder |
| `AllSceneData` | `std::vector<SceneData>` of every scene the app can be in: name, scene id, version | empty, **must contain at least one** |
| `DefaultSceneName` | which of those to start in | `""` |
| `HMDType` | `ECognitiveHMDType`: `kUnknown`, `kVive`, `kRift`, `kGear`, `kMobile`; picks the replay player mesh | `kUnknown` |
| `GazeInterval` | how often the host sends gaze; **tags the data, does not drive it**; the header says it must stay `0.1f` unless agreed | `0.1f` |
| `GazeBatchSize`, `CustomEventBatchSize`, `SensorDataLimit`, `DynamicDataLimit`, `FixationBatchSize` | points batched before a send | 64 each |
| `CustomGateway` | private-cloud gateway host | `data.cognitive3d.com` |
| `DynamicObjectFileType` | mesh format name sent with object registrations | `"obj"` in the header; the docs example sets `"gltf"` |
| `loggingLevel` | `LoggingLevel::kAll` by default | all |

```cpp
cognitive::CoreSettings settings;
settings.webRequest = &MakeWebRequest;
settings.APIKey = "<application key, entered by the developer>";
std::vector<cognitive::SceneData> scenes;
scenes.emplace_back(cognitive::SceneData("tutorial", "<scene id>", "1"));
settings.AllSceneData = scenes;
settings.DefaultSceneName = "tutorial";
settings.DynamicObjectFileType = "gltf";
settings.HMDType = cognitive::ECognitiveHMDType::kUnknown;

auto cog = cognitive::make_unique_cognitive<cognitive::CognitiveVRAnalyticsCore>(settings);
```

- The runtime key is the **Application Key**. The **Developer Key** is a separate credential used only by the Upload Web App and never belongs in `CoreSettings`.
- The key is compiled into the binary and is extractable from it. That is inherent to a client-side SDK. Recommend sourcing it from a build-time define or config file that is not committed, and follow SKILL.md rule 4: never read, echo or log the value.
- **The HMD type enum is a fossil.** It predates every current headset. `kUnknown` is the honest default; do not let a team pick `kRift` for a Quest because it is the closest word.
- **A scene id that does not match an uploaded scene is the most common cause of "sessions exist but replay is empty."** Re-uploading a changed scene increments the version, which has to be updated in `SceneData` too.

### "How do I start and end a session?"

```cpp
cog->SetDeviceName("<stable device identifier>");
cog->SetUserName("<participant identifier>");
cog->SetSessionProperty("age", 21);               // int, float and string overloads
cog->SetSessionName("<optional session name>");
cog->StartSession();                                // returns false if already started

// ... per-frame recording ...

cog->SendData();                                    // flush outstanding batches; must be after StartSession
cog->EndSession();                                  // sends final data
```

- **Either `SetDeviceName` or `SetUserName` is mandatory.** The docs say so directly; the SDK derives its user id from whichever is set. Prefer a stable device id plus a participant-supplied user name, which is the shared-device pattern in `data_strategy.md`.
- There is no participant property API. The C++ SDK predates the participant namespace: it sends `c3d.username` and `c3d.deviceid` rather than `c3d.participant.*`. Anything about the person that needs to be queryable goes in a **session property**, and the plan should say so rather than listing participant properties it cannot set.
- `Instance()` is a static accessor and **may return null** before construction. Guard it.
- Threading is undocumented. Treat the object as single-threaded and call it from the loop that owns it.

### "How do I record custom events?"

- Events: https://docs.cognitive3d.com/cpp/events/

```cpp
std::vector<float> pos = { 0, 0, 0 };                      // required: world position of the event
cog->GetCustomEvent()->RecordEvent("Toggle Music", pos);

cognitive::nlohmann::json props;
props["volume"] = -21;                                     // typed: numbers stay numbers
props["duration_seconds"] = 4.5f;
cog->GetCustomEvent()->RecordEvent("Set Music Volume", pos, props);

cog->GetCustomEvent()->RecordEvent("Grab", pos, props, dynamicObjectId);   // associate with a dynamic object
```

Overloads also take an explicit `double timestamp` for events reconstructed after the fact.

- **A position vector is required**, as on WebXR. Pass the HMD or the interaction point; `{0,0,0}` works but places every event at the origin in replay.
- Properties are JSON, so types are preserved and there is **no documented property cap**. The naming conventions in `data_strategy.md` apply unchanged.
- Dynamic object association is a **string id parameter**, matching the engine SDKs rather than Android XR's property convention.
- Events recorded before `StartSession()` are queued and sent once the session begins. Events batch to `CustomEventBatchSize` and go on the next `SendData()` or when the batch fills.

### "How do I track gaze?"

- Gaze: https://docs.cognitive3d.com/cpp/gaze/

The host calls one of these from its update loop every `GazeInterval` seconds:

```cpp
auto gaze = cog->GetGazeTracker();
gaze->RecordGaze(hmdPosition, hmdRotation);                          // looking at nothing: sky, void
gaze->RecordGaze(hmdPosition, hmdRotation, worldGazePoint);          // hit a surface in the scene
gaze->RecordGaze(hmdPosition, hmdRotation, localGazePoint, objectId);// hit a dynamic object; point in its local space
gaze->RecordGaze(hmdPosition, hmdRotation, localGazePoint, objectId, mediaId, mediaTimeMs, uvs); // hit uploaded media
```

Positions are `std::vector<float>` of three, rotations a quaternion of four, world space. Each overload also has a timestamped variant.

- **The SDK does not raycast.** The host decides what was hit and in which object's local space, then reports it. Per-object gaze and heatmaps therefore depend entirely on the host's hit test being correct and the object ids matching registrations.
- **Eye tracking versus head direction is the host's choice.** If the host passes a forward ray from the HMD, every attention metric on the dashboard is head direction, exactly as on visionOS, and the plan must say so. If the host has eye-tracking data and passes the eye gaze point, the metrics are eye attention. Ask which during discovery; the SDK gives no hint either way.
- Keep `GazeInterval` at `0.1f`. The header is unusually blunt that the value tags the data for processing rather than throttling it, so sending at a different cadence while leaving the tag alone corrupts dwell maths downstream.
- `SendData()` on the gaze tracker also carries any session properties set since the last send.

### "How do I record fixations?"

Undocumented on the site; present in `fixation.h`.

```cpp
cog->GetFixation()->RecordFixation(startTime, durationMs, maxRadius, worldPosition);
cog->GetFixation()->RecordFixation(startTime, durationMs, maxRadius, objectId, localPosition);
```

- **The host computes the fixation**: start time, duration in milliseconds, maximum radius, and whether it landed on a dynamic object. The SDK only batches and sends. Unity and Unreal ship fixation recorders; here the detection algorithm is the team's.
- Only recommend fixation-based analysis to a team that already has eye tracking and a fixation detector, or is prepared to build one. Otherwise plan on gaze.

### "How do I track dynamic objects?"

- Dynamic Objects: https://docs.cognitive3d.com/cpp/dynamic-objects/

```cpp
auto dyn = cog->GetDynamicObject();

// Register, either with your own stable id (needed for cross-session aggregation) ...
dyn->RegisterObjectCustomId("Drill", "drill_mesh", "drill_01", position, rotation);
// ... or let the SDK generate one for a transient instance
std::string id = dyn->RegisterObject("Spark", "spark_mesh", position, rotation, scale);

// Snapshot when the object has moved past a threshold, not every frame
dyn->RecordDynamic("drill_01", position, rotation);
dyn->RecordDynamic("drill_01", position, rotation, scale, properties);

// Interactions
dyn->BeginEngagement("drill_01", "Grab", controllerId);
dyn->EndEngagement("drill_01", "Grab", controllerId);

// Removal
dyn->RemoveObject("drill_01", position, rotation);
```

| Parameter | Meaning |
| --- | --- |
| `name` | this instance's display name |
| `meshname` | **the field that must match the uploaded model name**; groups instances for aggregation |
| `customid` | your stable id. Use it for anything that should aggregate across sessions |
| `controllerType`, `isRight` | overloads that register the object as a controller, enabling input-state snapshots |

- **There are no ID pools.** `RegisterObject` hands back a generated id per call, which is per-instance registration like visionOS, Android XR and WebXR. For spawned objects whose identity matters across sessions, generate and pass a custom id yourself.
- **Controllers are dynamic objects you register**, with the controller overloads, and their button and axis state goes in via the `RecordDynamic` overloads that take `inputName`/`inputValue` or a `std::vector<ControllerInputState>`. Nothing tracks controllers unless the host does this.
- **Snapshot on movement, not on frame.** The docs say it explicitly; the batch limit is 64 and a per-frame snapshot of a few objects fills it in a second.
- Object ids are refreshed on scene change so reused objects land in the new scene's manifest; re-registration after `SetScene` is handled, but verify the behaviour live if objects persist across scenes.
- Mesh geometry still has to be uploaded separately; see below.

### "How do I upload scenes and object meshes?"

- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/, at https://upload.cognitive3d.com

**There is no exporter.** The host engine's own pipeline produces the geometry, and it goes up through the Upload Web App with the **Developer Key**, which accepts **glTF Separate** (`.gltf` plus `.bin`), not GLB. `DynamicObjectFileType` in `CoreSettings` should match what was uploaded; the docs example uses `"gltf"`, and the header default of `"obj"` reflects an older pipeline.

As on every SDK, dynamic object mesh upload is **separate from scene upload** and is the step most often missed.

### "How do I add session metadata?"

```cpp
cog->SetSessionProperty("build", "1.4.2");         // string
cog->SetSessionProperty("difficulty", 3);          // int
cog->SetSessionProperty("start_latency_s", 0.42f); // float
cog->SetSessionName("lab-run-17");
cog->SetLobbyId(lobbyId);                           // multiplayer: links sessions sharing a lobby
```

- Session properties set before `StartSession()` go with the first send; ones set later ride on the next gaze send.
- **No participant properties, no session tags.** Both are in the skill's primitive table and both are absent from this API. Use session properties for the person's attributes, and leave cohort tagging to the dashboard, where analysts can tag sessions after the fact without SDK support.
- `EDeviceProperty` in the core header enumerates app and device metadata keys (app name, version, engine, device model, GPU and so on), and `HMDType` from settings is sent as the HMD type. Whether the enum is reachable from the public API is not clear from the headers; verify before promising device metadata beyond HMD type.

### "How do I record sensors?"

- Sensors: https://docs.cognitive3d.com/cpp/sensors/

```cpp
cog->GetSensor()->RecordSensor("heart_rate_bpm", 79.0f);
```

- **Float values only**, timestamped by the SDK at the call.
- **Nothing is automatic.** FPS, battery, memory, anything: the host samples it and calls `RecordSensor`. The sampling guidance from the Android XR reference transfers directly: a few seconds for standard metrics, faster only for interaction, never per frame.
- Batches to `SensorDataLimit` per sensor before sending.

### "How do I ask users questions in-app?"

- ExitPoll: https://docs.cognitive3d.com/cpp/exitpoll/

```cpp
auto poll = cog->GetExitPoll();
poll->RequestQuestionSet("scene_complete");          // hook name from the dashboard
// ... later, once HasQuestionSet() ...
nlohmann::json qs = poll->GetQuestionSet();          // render it yourself
poll->AddAnswer(cognitive::ExitPollAnswer(cognitive::EQuestionType::kBoolean, 1));
poll->AddAnswer(cognitive::ExitPollAnswer(cognitive::EQuestionType::kScale, 7));
poll->AddAnswer(cognitive::ExitPollAnswer(cognitive::EQuestionType::kVoice, base64Wav));
poll->SendAllAnswers(position);                      // position overload places it in replay
poll->ClearQuestionSet();
```

Question types: `kBoolean`, `kHappySad`, `kThumbs`, `kMultiple` (answer is the option index), `kScale`, `kVoice` (answer is a base64-encoded `.wav`). A negative number skips a question.

- **No UI is shipped.** Like WebXR, the hook placement is cheap and the survey rendering is real work. Scope it as such in the plan.
- **No offline question-set cache.** The request goes through the host's web callback; if it fails, `HasQuestionSet()` stays false and the host must handle it.
- The "place hooks early" baseline from `data_strategy.md` holds: the hook is one line, and questions are managed on the dashboard without a rebuild.

### "How do I create or change objectives?"

Objectives are a platform resource and behave identically regardless of SDK, so they work here even though the SDK has no objective API of its own.

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

Two routes, both fully supported: the dashboard, or the MCP server (`create_objective`, `update_objective`, `delete_objective`, needs a write-enabled organization key). Ask which the team wants; always dry-run first.

**Objective platform constraints, identical across SDKs:** `sequential` cannot be changed after save; writing steps re-scores roughly the last 30 days; `name` is capped at 32 characters; gaze and fixation steps reference per-project dynamic object ids; `delete_objective` is a permanent cascade.

Gaze and fixation steps on this SDK score whatever the host passed to `RecordGaze` and `RecordFixation`. If that was head direction, name the objective step accordingly.

### "What happens when the device is offline?"

Nothing is kept. There is no local cache, no retry and no queue in the SDK: a batch handed to the host's web callback that fails to send is lost unless the host's own HTTPS layer retries it. For kiosk, field or flaky-network deployments, the host has to add persistence in its `webRequest` implementation, and the plan should treat that as a real work item rather than a setting.

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/

Device support is the host engine's concern; the SDK only needs a C++11 compiler and an HTTPS client. The `HMDType` enum does not list any current headset, so `kUnknown` is the correct value for all of them.

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

Identical across SDKs. Route API query construction to the `cognitive3d-public-api` skill.

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- MCP is a data and configuration layer, not an instrumentation layer. It is SDK-agnostic, so everything in SKILL.md about MCP-based validation applies unchanged. It matters most here: with no editor, no automatic layer and a host-owned transport, confirming programmatically that gaze, events and objects actually arrived is the only fast feedback loop there is.

---

## C++ SDK directory

- Get started: https://docs.cognitive3d.com/cpp/get-started/
- Gaze: https://docs.cognitive3d.com/cpp/gaze/
- Sensors: https://docs.cognitive3d.com/cpp/sensors/
- ExitPoll: https://docs.cognitive3d.com/cpp/exitpoll/
- Dynamic objects: https://docs.cognitive3d.com/cpp/dynamic-objects/
- Events: https://docs.cognitive3d.com/cpp/events/
- Advanced (scene ids, custom gateway, lobby id): https://docs.cognitive3d.com/cpp/advanced/
- Upload Web App: https://docs.cognitive3d.com/dashboard/upload-webapp/

**In the headers but not on the docs site:** fixations (`fixation.h`), media gaze (`RecordGaze` overload with `mediaId`), controller registration and input states (`dynamicobject.h`), session name (`SetSessionName`), explicit timestamps on most record calls.

**Absent from both:** participant properties, session tags, local cache, remote controls, audio recording outside ExitPoll voice, Active Session View, Ready Room, any automatic sensor.

---

## Dashboard directory

Dashboard surfaces are SDK-agnostic. Listed in full so a C++ engagement never needs another reference file.

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
| Nothing arrives at all | `webRequest` never set, or the host's implementation never calls the response callback; the SDK has no transport of its own |
| Session never starts | neither `SetDeviceName` nor `SetUserName` called before `StartSession()` |
| `Instance()` returns null | accessed before `CognitiveVRAnalyticsCore` was constructed |
| Sessions exist but replay has no geometry | scene id in `AllSceneData` does not match an uploaded scene, or the scene was never uploaded |
| Replay shows old geometry after a re-upload | scene version not bumped in `SceneData` |
| Data stops partway through a session | a batch failed in the host's HTTPS layer; there is no retry or cache, so the batch is gone |
| Data only appears at session end | `SendData()` never called; batches below their size limit sit until end or flush |
| Dwell and attention numbers look wrong | gaze sent at a cadence other than `GazeInterval`, or the host raycast reports the wrong hit |
| Attention metrics are really head direction | host passed a forward ray as the gaze point; the SDK cannot tell, so the plan must say which it is |
| Dynamic object tracked but invisible in replay | mesh never uploaded, `meshname` does not match the uploaded model, or `DynamicObjectFileType` does not match the uploaded format |
| Dynamic object instances do not aggregate | registered with `RegisterObject` (generated ids) instead of a stable custom id |
| Dynamic batch fills instantly | snapshots recorded every frame instead of on movement past a threshold |
| Controllers absent from replay | never registered with the controller overloads; nothing registers them automatically |
| ExitPoll question set never arrives | `RequestQuestionSet` went through a failing `webRequest`; check `HasQuestionSet()` before rendering |
| Upload Web App rejects the model | GLB supplied; it accepts glTF Separate only (`.gltf` plus `.bin`) |
| Upload Web App rejects the key | organization key used (`orgkey-`); it needs the Developer Key |
| Dev traffic polluting dashboards | no editor-session concept exists; the plan's dev/prod session property was never set |
| Build fails on `make_unique` | C++11 toolchain; use `cognitive::make_unique_cognitive` |

---

## High-staleness surfaces (always verify live)

- Whether the repository has had any activity since the last check, and whether a tagged release exists yet
- Any API not shown on the seven docs pages, since the headers may move ahead of or behind them
- `DynamicObjectFileType` default and the Upload Web App's accepted formats
- Whether `EDeviceProperty` is reachable from the public API
- Threading guarantees, which are undocumented
- Dashboard navigation paths
- API key formats and auth examples
