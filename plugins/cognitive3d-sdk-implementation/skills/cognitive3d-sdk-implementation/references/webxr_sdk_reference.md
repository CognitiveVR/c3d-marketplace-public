# Cognitive3D WebXR SDK Technical Reference

This file is the routing and API layer for **WebXR** implementation questions. Load it when the target project is a browser-based XR app (Three.js, Babylon.js, PlayCanvas, Wonderland Engine, Mattercraft, A-Frame, or plain WebXR/WebGL).

For Unity projects load `unity_sdk_reference.md` instead; for Unreal Engine load `unreal_sdk_reference.md`; for native Apple Vision Pro load `visionos_sdk_reference.md`; for native Android XR load `androidxr_sdk_reference.md`. For cross-SDK feature parity at planning time, see `sdk_capability_matrix.md`.

**Primary docs root:** https://docs.cognitive3d.com/
**WebXR docs root:** https://docs.cognitive3d.com/webxr/get-started/
**Repository:** https://github.com/CognitiveVR/c3d-sdk-webxr
**Sample apps:** https://github.com/CognitiveVR/c3d-webxr-sample-apps
**NPM package:** `@cognitive3d/analytics`

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative.
3. **Escalate to live docs when freshness matters.** The WebXR SDK is younger than the Unity SDK and is explicitly described in the docs as still in active development, so the feature matrix and adapter coverage move more than Unity's. Verify anything version-, adapter-, or capability-dependent against the live page before quoting it.
4. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
5. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.
6. **Check the framework first.** WebXR capability is not uniform. Before answering any "how do I do X" question, establish which framework the project uses, then check the feature matrix below. Several features exist only on the Three.js and Mattercraft adapters.

### Trust hierarchy

1. **Exact live feature page**: e.g. WebXR Dynamic Objects, WebXR Events
2. **Framework page** for the project's engine: Three.js, Mattercraft, Wonderland, PlayCanvas, Plain JS
3. **Repository and sample apps** for current API shape and working reference integrations
4. **Docs portal root** when the question is broad or needs routing
5. **API/Data and MCP docs** for programmatic reads, and for objective and ExitPoll configuration writes

### Change-watch anchors (verify live before quoting)

- WebXR docs: https://docs.cognitive3d.com/webxr/get-started/
- Framework support matrix: https://docs.cognitive3d.com/webxr/framework-support/
- SDK repository: https://github.com/CognitiveVR/c3d-sdk-webxr
- NPM package version: `npm view @cognitive3d/analytics version`
- Supported hardware: https://docs.cognitive3d.com/hardware/

---

## The single most important structural difference from Unity

**There is no editor, with one exception.** In Unity, a large share of Cognitive3D work is Editor workflow: add a component, configure it in the Inspector, run Feature Builder. In almost every WebXR target, everything the app does at runtime is **code**, and the only non-code steps are **web app uploads** and **dashboard configuration**.

**The exception is Mattercraft**, which has an editor with behaviors, a properties panel and export hotkeys. On a Mattercraft project, read the Mattercraft notes further down this file before applying anything in this section; the Unity-style "check for an existing editor pattern first" rule does apply there.

This changes how tasks classify (see SKILL.md Step 7):

| Unity | WebXR equivalent |
| --- | --- |
| Editor workflow (add component, Inspector config) | Code task (constructor args, `userData` tags, adapter calls) |
| Feature Builder scene upload | Upload Web App at https://upload.cognitive3d.com |
| Feature Builder dynamic object mesh upload | Upload Web App, same place, separate artifact |
| Project Setup window (keys, rig detection) | `settings.js` config object plus explicit adapter wiring |
| Editor-session auto-exclusion from dashboards | **No equivalent.** Dev/prod separation must be explicit |

Two consequences worth stating to developers directly:

- **Almost every WebXR instrumentation item is something you can write code for.** The "check whether the project has an editor pattern first" rule in SKILL.md rule 9 is largely inert here. What replaces it is the framework-adapter check: confirm the adapter supports the feature before recommending it.
- **There is no in-editor auto-exclusion of developer sessions.** Local development against `localhost` will produce real sessions on the dashboard unless the team separates them. Make `development_mode` (or a separate project) a hard Phase 1 item for every WebXR project, not a nice-to-have.

---

## Common implementation mental model

1. **Create/choose a project** in the dashboard
2. **Get the right keys**: Application Key for the runtime config, Developer Key for the Upload Web App
3. **Install the SDK**: `npm install @cognitive3d/analytics` (Node 20+)
4. **Construct `C3D`** with a settings object and the renderer, then construct the framework adapter
5. **Declare scene data** in `allSceneData` and call `setScene()`
6. **Start and end sessions** against the browser's `XRSession`
7. **Attach session metadata**: participant info, tags, session properties (`c3d.app.version` is required)
8. **Record telemetry**: custom events, sensors, dynamic objects, exit polls
9. **Upload scene and dynamic object geometry** through the Upload Web App
10. **Validate in dashboard**: replay, scene/object views, analysis
11. **Troubleshoot** if data is missing

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Custom Event, Sensor, ExitPoll, Adapter, Interactable Group

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Framework support matrix

Source of truth: https://docs.cognitive3d.com/webxr/framework-support/. Verify live before making a capability promise, since the WebXR SDK is actively adding adapter coverage.

| Capability | Three.js | Mattercraft | Wonderland | PlayCanvas | Babylon | Plain JS |
| --- | --- | --- | --- | --- | --- | --- |
| Core API (sessions, events, properties, sensors, exit polls) | yes | yes | yes | yes | yes | yes |
| WebXR gaze tracking | yes | yes | yes | yes | yes | see note |
| Performance sensors (profiler) | yes | yes | no | yes | no | no |
| Object export | yes | yes | partial | no | no | no |
| Scene export | yes | yes | partial | no | no | no |
| Dynamic objects | yes | yes | no | no | no | no |
| Sample app available | yes | yes | yes | no | no | no |

Two entries in that table need care, because the upstream docs are not fully consistent about them:

- **Plain JS gaze.** The matrix marks gaze unsupported, while the Plain JS page lists automatic WebXR gaze tracking, HMD orientation, controller tracking and boundary as working. The reconciliation that fits both: the core SDK still captures viewer-pose gaze from the `XRSession`, and what an adapter adds is **engine-driven raycast gaze, per-object gaze analysis and heatmaps**. So a plain-JS project gets a gaze stream but no per-object attention data. Verify against the live pages before promising either reading to a team, and do not plan object-level attention analysis on plain JS.
- **Babylon.js.** The matrix credits Babylon with core API and gaze, but no Babylon adapter, framework page, or setup instructions exist in the docs. Treat Babylon as core-API support and confirm anything beyond that with the live docs before committing to it in a plan.
- **A-Frame** is named as supported in the SDK repository README but has no column in the framework matrix and no framework page. Treat it as unverified: establish what the project actually needs, then confirm capability with the live docs before planning around it.

**Planning implications, and these are the ones that actually change a track plan:**

- **Dynamic objects are Three.js and Mattercraft only.** On Wonderland, PlayCanvas, Babylon, or plain JS you cannot recommend per-object gaze, object heatmaps, or object engagement. Substitute custom events carrying an object identifier property, and say plainly what is lost: you get "the user interacted with the valve" but not "the user looked at the valve for 4 seconds before touching it."
- **Plain JS has no per-object gaze.** Heatmaps, object attention analysis and fixation-based objectives are off the table even though a gaze stream exists. Plan a plain-JS project as an events-and-properties project.
- **Scene export varies.** Where the adapter cannot export the scene, the team must produce the glTF another way and upload it manually, or accept replay without geometry.
- **Wonderland scene export is geometry only**: positions, normals, UVs, with no materials or textures. Replay will be untextured. Set that expectation before the team sees the dashboard.
- **Wonderland's "partial" object export does not imply dynamic object support**, which the same matrix marks unsupported. Exported object geometry has nothing to attach to without dynamic object tracking, so do not read that row as a route to per-object analysis. Confirm live if a team is counting on it.

### Engine identification

With no adapter, the SDK reports `c3d.app.engine` as `"WebXR"`. Adapters set it themselves (for example PlayCanvas sets `AppEngine = "PlayCanvas"` with the engine version). To override, or to identify a framework with no adapter:

```javascript
c3d.setDeviceProperty("AppEngine", "YourEngineName");
c3d.setDeviceProperty("AppEngineVersion", "1.0.0");
```

Plain-JS integrations should set this explicitly, otherwise every project in the org that skips an adapter looks identical in engine-based dashboard filters.

---

## Fast route by question type

### "How do I install the SDK?"

- Get started: https://docs.cognitive3d.com/webxr/get-started/
- `npm install @cognitive3d/analytics`
- Requires Node 20 or higher
- Builds ship as UMD (script tag), ES Module (bundlers), and CommonJS
- From source: clone https://github.com/CognitiveVR/c3d-sdk-webxr, `npm install`, `npm run build`, then `npm install /path/to/c3d-sdk-webxr`
- Mattercraft is the exception: it uses `@cognitive3d/three-mattercraft`, installed through the Mattercraft editor's dependencies panel
- PlayCanvas is the other exception: add `c3d-bundle-playcanvas.umd.js` to the project as an asset

### "How do I configure keys and settings?"

Settings are a plain object, conventionally in `settings.js`:

```javascript
export default {
  config: {
    APIKey: "YOUR_APPLICATION_API_KEY",
    allSceneData: [
      { sceneName: "MyScene", sceneId: "", versionNumber: "1" },
    ],
    // optional
    gazeTrackingSource: "webxr",   // or "engine"
    customEventBatchSize: 256,
    sensorDataLimit: 512,
  },
};
```

- The runtime key is the **Application Key**. The **Developer Key** is a separate credential used only by the Upload Web App. Do not put the Developer Key in client-side config.
- **The Application Key ships to the browser.** This is inherent to a client-side SDK, not a misconfiguration, but it is different from a Unity build and teams occasionally raise it. Recommend sourcing it from an environment variable at build time (`.env` plus a bundler define) so it is not committed to the repo, and note that this is obfuscation of the source tree, not secrecy at runtime. Follow SKILL.md rule 4: never read or echo the key value itself.
- Any config value can be changed at runtime with `c3d.config(key, value)`.

### "How do I start and end a session?"

```javascript
import C3D from "@cognitive3d/analytics";
import C3DThreeAdapter from "@cognitive3d/analytics/adapters/threejs";
import settings from "./settings";

const c3d = new C3D(settings, renderer);
const c3dAdapter = new C3DThreeAdapter(c3d);

c3d.setScene("MyScene");
c3d.setUserProperty("c3d.app.version", "1.0"); // required

renderer.xr.addEventListener("sessionstart", async () => {
  const xrSession = renderer.xr.getSession();
  c3dAdapter.startTracking(renderer, camera, interactableGroup);
  await c3d.startSession(xrSession);
});

renderer.setAnimationLoop((timestamp, frame) => {
  c3dAdapter.update(timestamp, frame);
  renderer.render(scene, camera);
});

renderer.xr.addEventListener("sessionend", () => {
  c3d.endSession();
});
```

Three failure modes account for most "no data on the dashboard" reports, and all three are worth checking before anything else:

1. **`c3d.app.version` was never set.** The docs call this out explicitly: without it you may not get a valid session.
2. **`c3dAdapter.update()` is not called every frame.** Transforms, gaze and dynamic object snapshots all ride on it. If the app uses its own loop rather than `setAnimationLoop`, the call still has to happen there, with the `timestamp` and `frame` the XR frame callback provides.
3. **`endSession()` was not called.** The browser tab can close without an `endSession`, and queued events and sensor readings are lost with it. Wire `endSession` to `sessionend`, and consider `visibilitychange` or `pagehide` as a backstop for users who remove the headset or close the tab without exiting the immersive session.

### "How do I upload scenes?"

- WebXR Scenes: https://docs.cognitive3d.com/webxr/scenes/
- Upload Web App: https://upload.cognitive3d.com (authenticate with the **Developer Key**)

**This is a web app workflow, not code and not an editor step.** A scene upload is four files:

| File | Contents |
| --- | --- |
| `scene.gltf` | node, material and binary manifest |
| `scene.bin` | vertex, normal, UV and index data |
| `settings.json` | scene name, scale, SDK version |
| `screenshot.png` | dashboard thumbnail |

After upload you receive a Scene ID and a version number starting at 1. Both go into `allSceneData`:

```javascript
allSceneData: [
  { sceneName: "MyMainLevel", sceneId: "a1b2c3d4-e5f6-g7h8i9j0", versionNumber: "1" },
]
```

**Data only attaches to geometry when `sceneName` matches an uploaded scene with a real `sceneId`.** A blank or stale `sceneId` is the most common cause of "sessions exist but replay shows nothing." Re-uploading a modified scene increments the version while the ID stays constant, so `versionNumber` has to be updated in config on every re-export or replay silently falls back to the old geometry.

Multi-scene sessions: call `c3d.setScene(newSceneName)` at the transition. The SDK flushes queued data from the previous scene and refreshes the dynamic object manifest.

### "How do I track dynamic objects?"

- WebXR Dynamic Objects: https://docs.cognitive3d.com/webxr/dynamic-objects/
- **Three.js and Mattercraft only.** Confirm the framework before recommending any of this.

The workflow has four parts and skipping any one of them produces a silent partial failure:

1. **Export and upload the mesh** through the Upload Web App, exactly as with scenes and separately from the scene upload. This is the WebXR equivalent of Unity's Feature Builder mesh upload, and it is missed just as often.
2. **Tag the object** via `userData`:

```javascript
object.userData.isDynamic = true;
object.userData.c3dId = "valve_01";
// optional snapshot thresholds
object.userData.positionThreshold = 0.01;  // metres
object.userData.rotationThreshold = 0.5;   // degrees
object.userData.scaleThreshold = 0.05;
```

3. **Register it** so the backend knows the mesh-to-instance mapping:

```javascript
c3d.dynamicObject.registerObjectCustomId(name, meshId, customId, position, quaternion, scale);
```

4. **Put it in the tracked set**: pass it in the `interactableGroup` given to `startTracking(renderer, camera, interactableGroup)`, or, for anything spawned later, `c3dAdapter.addInteractable(object)`.

`startTracking` accepts a `THREE.Scene`, a `THREE.Group`, a single `Object3D`, or an array of them.

**Engagements** record interaction states such as grabbing:

```javascript
c3d.dynamicObject.beginEngagement(objectId, engagementTypeName, parentObjectId);
c3d.dynamicObject.endEngagement(objectId, engagementTypeName, parentObjectId);
```

Notes and differences from Unity:

- **Smart Snapshot Recording**: transforms are only recorded when they move past the thresholds above. Slow drift below the threshold will not appear. Raising a threshold is the first thing to try when an object's payload is too chatty; lowering it is the fix when subtle motion is missing from replay.
- **No ID Pool concept is documented for WebXR.** For objects spawned at runtime, call `registerObjectCustomId` with a generated `customId` at spawn time. Do not port Unity ID Pool advice across.
- Heatmap coordinates are object-local for tagged objects and world-space otherwise.
- The "do not make every interactable a dynamic object" rule from `data_strategy.md` applies unchanged, and matters more here: registration is manual, so every object is real work.

### "How do I record custom events?"

- WebXR Events: https://docs.cognitive3d.com/webxr/events/

```javascript
c3d.customEvent.send(eventName, [x, y, z], properties);

c3d.customEvent.send("player_jumped", [1.5, 1.2, -3.0]);
c3d.customEvent.send("enemy_hit", [10.2, 0.4, 8.1], {
  weapon_used: "Plasma Rifle",
  target_type: "EnemyDrone",
  damage_dealt: 75,
  was_critical_hit: true,
});
```

- **Position is a required positional argument**, an `[x, y, z]` array in world space. This is the main shape difference from the Unity API and the most common porting mistake. If the event has no meaningful location, pass the camera or player position rather than inventing one; the position is what places the event in replay.
- Property values may be strings, numbers, or booleans. Send numbers as numbers, per `queryable_data.md`.
- Batching: `customEventBatchSize` defaults to 256. Lower it during validation (`c3d.config("customEventBatchSize", 32)`) so events appear on the dashboard without waiting for a full batch, then restore it. The docs' profiling page notes that many small requests cost more than few large ones, so do not leave a tiny batch size in production.
- Queued events flush automatically on `endSession()` and `setScene()`.

**Recorded automatically, so do not instrument these yourself:** session start and end (with `sessionlength` and `Reason`), controller tracking loss and recovery, boundary changes and exits, and input source changes.

### "How do I add session and participant metadata?"

- WebXR Properties: https://docs.cognitive3d.com/webxr/properties/

```javascript
// session
c3d.setSessionName("Onboarding Tutorial - Version B");
c3d.setLobbyId("Lobby-Alpha-42");
c3d.setSessionTag("beta");             // sets c3d.sessiontag.beta = true
c3d.setSessionTag("tutorial", false);  // explicit value
c3d.setSessionProperty("difficulty", "hard");
c3d.setSessionProperty({ map_seed: 42, run_mode: "timed" });

// participant
c3d.setParticipantFullName("John Smith");
c3d.setParticipantId("p-alpha-42");
c3d.setParticipantProperty("cohort", "GroupA");
c3d.setParticipantProperties({ experimental_group: "Trained", pre_test_score: 85 });

// device / engine identity
c3d.setDeviceProperty("AppEngine", "None");

// required
c3d.setUserProperty("c3d.app.version", "1.0");
```

- Session properties ride out with the gaze stream and resend only when the value changes, so setting one repeatedly in an update loop is cheap but pointless.
- Custom participant keys are automatically prefixed with `c3d.participant.`.
- Set at least one participant identifier when cross-session analysis matters. The session/participant split rule in `data_strategy.md` applies unchanged.

### "How do I record sensors?"

- WebXR Sensors: https://docs.cognitive3d.com/webxr/sensors/

```javascript
c3d.sensor.recordSensor("heartRate", 85);
c3d.sensor.recordSensor("playerStamina", 92.5);
c3d.sensor.recordSensor("isMoving", true);

setInterval(() => {
  c3d.sensor.recordSensor("activeEnemies", enemyManager.getActiveCount());
}, 1000);
```

Roughly once per second is the documented default cadence. Queued readings flush on `endSession()` and `setScene()`; `sensorDataLimit` defaults to 512.

**Recorded automatically when conditions allow:**

| Sensor | Active when | Metrics |
| --- | --- | --- |
| FPSTracker | always | Average FPS, 1% Low FPS |
| HMDOrientationTracker | active `XRSession` | HMD Pitch, HMD Yaw (degrees) |
| ControllerTracker | tracked controllers present | controller height from HMD |
| BoundaryTracker | `bounded-floor` XR session | RoomSize (m squared) |
| Profiler | renderer/app passed to the `C3D` constructor | draw calls, system memory, main thread time |

Note the Profiler condition: it only runs if the renderer (or `this.app` on PlayCanvas) was passed to `new C3D(settings, renderer)`. A project that constructs `C3D` without it gets no performance data and no error. This is worth checking during any performance-related validation.

The biometric sensors listed in `queryable_data.md` (HP Omnicept, HarmonEyes) are Unity-side integrations. Do not promise them on WebXR.

### "How do I ask users questions in-app?"

- WebXR ExitPoll: https://docs.cognitive3d.com/webxr/exitpoll/
- Dashboard ExitPoll Results: https://docs.cognitive3d.com/dashboard/exitpoll-results/
- MCP ExitPoll tools: https://docs.cognitive3d.com/mcp-server/exitpoll/

```javascript
await c3d.exitpoll.requestQuestionSet("end_of_module");
const questions = c3d.exitpoll.getQuestionSet();     // parsed JSON
// c3d.exitpoll.getQuestionSetString();              // same data as a string

c3d.exitpoll.addAnswer("scale", 4);
c3d.exitpoll.addAnswer("boolean", true);
c3d.exitpoll.sendAllAnswers([0, 1.6, -2]);           // position is optional

c3d.exitpoll.clearQuestionSet();                     // abandon without submitting
```

**The critical difference from Unity: the SDK does not render anything.** It fetches the question set and submits answers; presenting the questions in the scene is entirely the application's job. Unity ships prefab panels; WebXR does not. When an exit poll appears in a WebXR plan, scope the UI work explicitly, because a team reading the Unity or Unreal docs will assume it is free and it is not. Unity, Unreal and visionOS all ship survey UI; WebXR is the only target where the survey is the application's to build, and this is the largest per-feature effort gap in the skill.

Other constraints:

- Answer type strings are **case-sensitive**: `"happySad"`, `"boolean"`, `"thumbs"`, `"scale"`, `"multiple"`, `"voice"`. The uppercase labels shown on the dashboard (`BOOLEAN`) are not valid here and are treated as an unknown type.
- `requestQuestionSet` rejects unless called after `c3d.startSession(...)` has resolved.
- The question set clears automatically after submission.

**ExitPoll platform constraints are identical across SDKs, because they are properties of the platform, not the SDK.** Flag these before any write:

- Question set versions are immutable: editing creates a new version. Removal is archival only, with no hard delete.
- **A new version does not move existing hooks.** Hooks stay on the version they were assigned until reassigned with `update_exitpoll_hook`. Publishing v2 and expecting the app to pick it up is a silent no-op.
- Hooks cannot be deleted, only unassigned, and a hook with no question set assigned is skipped silently at runtime.

Question sets are configurable **either** on the dashboard **or** via MCP (`create_exitpoll_question_set`, `archive_exitpoll_question_set`, `create_exitpoll_hook`, `update_exitpoll_hook` for writes; `get_exitpoll_configuration` and `get_exitpoll_question_set` for reads). MCP writes need a write-enabled organization key with an org- or project-admin role. Ask which route the team wants (route-selection gates are in SKILL.md Step 7).

### "How do I create or change objectives?"

Objectives are a platform resource and behave identically regardless of SDK.

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

Two routes, both fully supported: the dashboard (no API key, right default when a non-developer owns definitions) or the MCP server (`create_objective`, `update_objective`, `delete_objective`, needs a write-enabled organization key). Ask which the team wants; always dry-run first.

**Objective platform constraints, identical across SDKs:**

- `sequential` cannot be changed after save.
- Writing steps asynchronously re-scores roughly the last 30 days of sessions and nothing older.
- `name` is capped at 32 characters.
- Gaze and fixation steps reference dynamic object IDs, which are per-project, so such objectives cannot be copied between projects verbatim. On WebXR this also means **gaze-step objectives are only possible on Three.js and Mattercraft**, since no other adapter supports dynamic objects.
- `delete_objective` is a **permanent cascade** with no restore path. Confirm intent explicitly.

### "How do I control runtime behavior remotely?"

**Remote Controls are not documented in the WebXR section.** Do not promise them. If a WebXR team asks for live tuning or A/B flags, verify current support at https://docs.cognitive3d.com/webxr/get-started/ before answering, and in the meantime plan the experiment around a condition recorded as a session property from the app's own config or feature-flag system. The strategy requirement from `data_strategy.md` still holds: whatever assigns the condition, the assigned condition must be recoverable later.

### "How do I test this during development?"

- Developer setup: https://docs.cognitive3d.com/webxr/developer-setup/

Two routes, and the choice matters for what data you get:

- **Desktop, via the Immersive Web Emulator Chrome extension.** Simulates a headset and controllers without hardware. Fast for iterating on event logic, but emulated poses are not real poses, so do not validate gaze, ergonomics, boundary or comfort data this way.
- **On a Meta Quest over USB**, with Chrome port forwarding to reach the local dev server (the docs use `localhost:5173`). Requires developer mode and Meta Quest Developer Hub. The Quest browser's console is inspectable from desktop Chrome. This is the route for validating anything spatial.

Both produce **real sessions on the dashboard**, since WebXR has no editor-session exclusion. Set `development_mode` before either route is used in anger, not after.

Working from the SDK source rather than the published package (https://docs.cognitive3d.com/webxr/working-with-source-code/): clone, `npm install`, `npm run build` to produce `/lib` and `/types`, then `npm install /localPathTo/c3d-sdk-webxr` in the app. The repo ships a Jest suite; `npm test` is a reasonable smoke check after any local SDK modification.

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/

WebXR adds a layer Unity does not have: the **browser's** WebXR implementation, not just the headset. Immersive VR and immersive AR sessions are both supported; for AR the SDK prioritises the `local` reference space and falls back to `local-floor`. Room-size data requires a `bounded-floor` session, so a project that requests only `local-floor` will have no boundary metrics by construction.

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

Identical across SDKs. Route API query construction to the `cognitive3d-public-api` skill.

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- MCP is a data and configuration layer, not an instrumentation layer. It reads project data and writes objectives and ExitPoll configuration. It is SDK-agnostic, so everything in SKILL.md about MCP-based validation applies unchanged to WebXR projects.

The SDK repository also ships a `validate-c3d-session` coding-agent skill under `.claude/skills/` for verifying that session data arrived. Worth mentioning to teams working directly in the SDK repo.

---

## WebXR SDK directory

### First-stop pages

- Get started: https://docs.cognitive3d.com/webxr/get-started/
- Framework support matrix: https://docs.cognitive3d.com/webxr/framework-support/
- Developer setup: https://docs.cognitive3d.com/webxr/developer-setup/
- Working with source code: https://docs.cognitive3d.com/webxr/working-with-source-code/
- Profiling and SDK overhead: https://docs.cognitive3d.com/webxr/profiling/

### Framework pages

- Three.js: https://docs.cognitive3d.com/webxr/threejs/
- Mattercraft: https://docs.cognitive3d.com/webxr/mattercraft/
- Wonderland Engine: https://docs.cognitive3d.com/webxr/wonderland/
- PlayCanvas: https://docs.cognitive3d.com/webxr/playcanvas/
- Plain JS / no framework: https://docs.cognitive3d.com/webxr/plainjs/

### Core feature pages

- Properties: https://docs.cognitive3d.com/webxr/properties/
- Scenes: https://docs.cognitive3d.com/webxr/scenes/
- Events: https://docs.cognitive3d.com/webxr/events/
- Sensors: https://docs.cognitive3d.com/webxr/sensors/
- Dynamic Objects: https://docs.cognitive3d.com/webxr/dynamic-objects/
- ExitPoll: https://docs.cognitive3d.com/webxr/exitpoll/

### Framework-specific notes

**Three.js** is the reference adapter and the most complete. `C3DThreeAdapter`, imported from `@cognitive3d/analytics/adapters/threejs`. Everything in this file applies as written.

**Mattercraft** uses a separate package, `@cognitive3d/three-mattercraft`, installed from the editor's Add-ons and Dependencies panel. It is the one WebXR target with an editor-like workflow:

- Add the **Cognitive3D Manager** behavior once per scene; its properties panel exposes API Key, Scene Id and Scene Name, so initial setup needs no script
- Mark dynamic objects with the **Cognitive3DDynamicObject** behavior
- Export scene and objects with Shift+E and Shift+D, then upload through the Upload Web App
- ZComponents (lightning-bolt icon) need the behavior attached inside the component definition, not in the scene hierarchy
- Animated sub-components are tracked via AttachmentPoints rather than animation tracking directly
- An "Enable Export" toggle allows exporting geometry at runtime during Live Preview

Because of this, Mattercraft is the one WebXR framework where SKILL.md rule 9 (check for an existing editor pattern before writing code) genuinely applies.

**Wonderland Engine**: `npm install @cognitive3d/analytics`, then a JS component that instantiates `C3DWonderlandAdapter` with the SDK and engine instance, registered in `index.js`. Adapter sets `AppEngine = "Wonderland Engine"`. Limitations: no dynamic objects, geometry-only glTF export (no materials or textures), no profiler.

**PlayCanvas**: add `c3d-bundle-playcanvas.umd.js` as a project asset, create a settings file, then instantiate `C3D` (passing `this.app` so the profiler works) and `C3DPlayCanvasAdapter` in a script component. There is a minimum supported PlayCanvas version; check the live page rather than quoting one, since it moves. Scene export, object export, dynamic objects and per-object heatmaps are not supported.

**Babylon.js**: core API and WebXR gaze only. No profiler, no dynamic objects, no export.

**Plain JS / WebGL**: core API only, and no gaze tracking. Set `AppEngine` manually. Treat these projects as events-and-properties projects and say so during discovery rather than after the plan is written.

---

## Dashboard directory

Dashboard surfaces are SDK-agnostic, so the entry points below are the same ones an engine project uses. They are listed here in full so a WebXR engagement never needs to open another SDK reference. Two exceptions matter for WebXR projects:

- **Object Explorer and Object Details** are only populated where dynamic objects exist, so they are empty on Wonderland, PlayCanvas, Babylon and plain JS projects.
- **App Performance** depends on the profiler, which requires the renderer to have been passed to the `C3D` constructor and is unsupported on Wonderland and Babylon.

Key entry points:

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
| No sessions on the dashboard at all | `c3d.app.version` not set, or `startSession` never awaited |
| Sessions appear but replay has no geometry | `sceneId` blank or `sceneName` mismatched in `allSceneData` |
| Replay shows old geometry after a re-export | `versionNumber` not bumped in config |
| Events missing at the end of a session | `endSession()` not called, or the tab closed without it |
| Events lag far behind the action during testing | `customEventBatchSize` still at the 256 default |
| Dynamic object never appears | mesh not uploaded, `c3dId` mismatch, `registerObjectCustomId` not called, or object not in the tracked set |
| Dynamic object appears but never moves | threshold values too high for the motion involved |
| Object stops updating after a scene change | manifest refreshed on `setScene`; re-register runtime-spawned objects |
| No gaze data | plain JS with no adapter, or `c3dAdapter.update()` not called every frame |
| No performance data | renderer/app not passed to the `C3D` constructor, or an adapter with no profiler support |
| No room size / boundary data | session is not `bounded-floor` |
| ExitPoll answers rejected | answer type string wrong case, or `requestQuestionSet` called before the session resolved |
| Dev traffic polluting dashboards | no editor auto-exclusion exists on WebXR; `development_mode` was never set |

---

## High-staleness surfaces (always verify live)

- Framework support matrix and adapter capabilities
- NPM package version and Node version floor
- Adapter import paths and bundle filenames
- Whether Remote Controls, local cache, media, or multiplayer components have landed for WebXR
- Dashboard navigation paths
- API key formats and auth examples
- MCP server config examples
- Device, browser and runtime feature support
