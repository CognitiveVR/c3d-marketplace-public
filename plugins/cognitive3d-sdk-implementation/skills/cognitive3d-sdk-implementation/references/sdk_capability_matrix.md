# Cross-SDK capability matrix

_Load this before finalizing any plan (SKILL.md Step 5) so the plan only contains things the target SDK can actually do._

This file answers one question: **is this recommendation possible on the SDK this project uses?** It does not explain how to implement anything. For that, load the SDK reference for the project's target: `unity_sdk_reference.md`, `unreal_sdk_reference.md`, `androidxr_sdk_reference.md`, or `webxr_sdk_reference.md`.

The strategy in `data_strategy.md` and the archetype playbooks in `playbooks.md` are deliberately SDK-neutral, which means they will happily suggest something the target SDK does not support. This file is the screen that catches it.

**Android XR here means the native Kotlin SDK** for Jetpack XR and Meta Spatial apps, not the External Android Plugin that Unity and Unreal projects use on Android headsets.

---

## Primitive availability

| Primitive | Unity | Unreal | Android XR | WebXR | Notes |
| --- | --- | --- | --- | --- | --- |
| Custom events | yes | yes | yes | yes | WebXR requires an explicit position argument; Android XR caps properties at 10 |
| Typed event properties | yes | **C++ only** | yes | yes | Unreal's Blueprint `Send` variant stringifies every value |
| Event property count | unlimited | unlimited | **10 per event** | unlimited | the only documented hard cap in the skill |
| Session properties | yes | yes | yes | yes | |
| Participant properties | yes | yes | yes | yes | |
| Session tags | yes | yes | **not documented** | yes | analyst-applied dashboard tags work everywhere regardless |
| Session name | yes | yes | **not documented** | yes | |
| Sensors (custom) | yes | yes, float only | yes, float only | yes | Unreal caps at 10 Hz by default |
| Sensors (automatic) | broad | **opt-in components** | **FPS only** | moderate | see below |
| Dynamic objects | yes | yes | yes | **Three.js and Mattercraft only** | the biggest planning constraint on WebXR |
| Dynamic object linked to an event | parameter | parameter | **by property convention** | parameter | Android XR spends one of its ten property slots on it |
| ID pools for spawned objects | yes | yes | no | no | non-engine SDKs register per instance |
| Gaze stream | yes | yes | yes, when `enable_gaze` | yes | |
| Per-object gaze and heatmaps | yes | yes | yes | **Three.js and Mattercraft only** | follows dynamic object availability |
| Eye-tracked fixations | yes, on supported hardware | yes, via Fixation Recorder | verify live | hardware dependent | |
| Objectives | yes | yes | yes | yes | platform resource, SDK-independent |
| ExitPoll | yes, UI shipped | yes, UMG widgets shipped | **not documented** | yes, but no UI provided | verify live on Android XR before planning one |
| Scene upload | in-engine tools | in-engine tools | Upload Web App | Upload Web App | |
| Dynamic object mesh upload | in-engine tools | Dynamic Object Manager | Upload Web App | Upload Web App | separate from scene upload on all four |
| Remote controls | yes | yes | **not documented** | **not documented** | verify live before promising |
| Local cache / offline upload | yes | yes | **not documented** | **not documented** | |
| Media and 360 | yes | yes | **not documented** | **not documented** | |
| Multiplayer | yes, components | yes, Lobby IDs | **not documented** | `setLobbyId` only | Unity ships Netcode and Normcore integrations |
| Audio recording | yes | **not documented** | **not documented** | **not documented** | |
| Active Session View | yes | yes | **not documented** | no | |
| Ready Room | yes | no | no | no | Unity-only onboarding scene |
| LMS forwarding | yes | yes | yes | yes | configured on the dashboard, SDK-independent |

"Not documented" means the feature has no page in that SDK's docs section. Treat it as unavailable for planning purposes, and verify live before telling a team either way. This matters most on Android XR, which is the newest SDK and has the smallest documentation set, so absence there is weaker evidence than absence elsewhere.

---

## Automatic capture differences

All four SDKs capture some base layer with no instrumentation (see `queryable_data.md`, which is written from Unity's behaviour). Where they differ:

| Automatic capture | Unity | Unreal | Android XR | WebXR |
| --- | --- | --- | --- | --- |
| Session lifecycle events | yes | yes | yes | yes |
| Head and controller pose | yes | yes | yes | yes |
| Hands | yes | yes | yes | yes |
| Gaze | yes | yes, via Player Tracker | yes, when `enable_gaze` | yes |
| HMD orientation | yes | **component** | verify live | yes |
| FPS | yes | **component** | yes | yes |
| Boundary / room size | yes | **component** | verify live | only in a `bounded-floor` session |
| Controller tracking loss | yes | **component** | verify live | yes |
| Battery | yes | **component** | verify live | no |
| Hand and arm ergonomics | yes | **component**, plus hand dynamics | verify live | partial |
| Draw calls, memory, main thread time | yes | no | no | only when the renderer is passed to the `C3D` constructor |
| Biometrics (HP Omnicept, HarmonEyes) | yes, on supported hardware | sensor API, integration dependent | no | no |
| Editor sessions excluded from dashboards | yes | **no, recorded behind a toggle** | **no equivalent** | **no equivalent** |

Three rows change a Phase 1 recommendation:

- **"Component" is not "automatic."** On Unreal, most comfort, performance and boundary metrics come from built-in components the team adds. Name the component.
- **Android XR documents only FPS as automatic.** Anything else the plan wants as a continuous value is a `recordSensor` call plus a sampling loop the team writes. Budget for it rather than assuming it is free.
- **Only Unity filters developer sessions.** Unreal records editor sessions behind a toggle; Android XR and WebXR have no editor-session concept at all. On the other three, explicit `development_mode` separation is required rather than advisory.

---

## Effort differences for the same recommendation

Same plan item, materially different cost depending on the SDK. Say so when it applies, because a team reading another SDK's docs will have the wrong expectation.

| Plan item | Unity | Unreal | Android XR | WebXR |
| --- | --- | --- | --- | --- |
| Exit poll at end of module | low: place a prefab hook | low to medium: Blueprint nodes, shipped widgets, plus a Widget Interaction component | **verify support first**; if absent, own UI plus a custom event | **medium to high**: fetch the set, build the UI, submit answers |
| Track ten dynamic objects | low to medium: components plus batch upload | low to medium: components plus Dynamic Object Manager | medium: register each, upload each through the web app | medium: per-object tagging, registration and upload |
| Track spawned objects | ID Pool | ID Pool Asset sized to concurrent spawns | register per instance | register per instance |
| Comfort / FPS / room metrics | free | add the built-in components | FPS free, the rest is your own sampling | partial; profiler needs the renderer passed in |
| Numeric event properties | free | free in C++, **broken in Blueprint** | free, within the 10-property cap | free |
| Scene upload | in-engine, one action | in-engine; multi-level needs a combine strategy | export glTF Separate, upload four files | export glTF Separate, upload four files |
| Dev/prod separation | session property; editor already excluded | session property; editor sessions visible | session property; nothing is excluded | session property; nothing is excluded |
| Install prerequisites | UPM, no project changes | **project must be C++ based** | Gradle, plus AndroidX XR alpha alignment | npm, Node 20+ |

---

## Screening rules

Apply these before presenting any plan:

1. **If the project is WebXR and not Three.js or Mattercraft, remove every dynamic object row.** Replace object-level gaze and interaction questions with custom events carrying an object identifier property, and state plainly what that cannot answer: dwell before action, attention without interaction, object heatmaps.
2. **If the project is plain JS or any WebXR framework without dynamic objects, remove every per-object attention recommendation.** A gaze stream still exists, but heatmaps, object attention analysis and fixation-based objectives do not.
3. **If the plan contains an exit poll, check the target supports one.** WebXR: scope the UI work as a line item. Unreal: scope the Widget Interaction component. **Android XR: verify ExitPoll exists at all before including it**, and if it does not, replace it with the app's own UI plus a custom event and say what is lost.
4. **If the plan contains remote controls, local cache, media, audio recording or multiplayer and the project is Android XR or WebXR, verify live before including them.** Do not carry them over from an engine-shaped plan.
5. **If the plan relies on biometric or device-level sensors, confirm the SDK and the hardware both support them.** Most of that list is Unity-first.
6. **If the project is anything but Unity, dev/prod separation is Phase 1 and non-negotiable.** Only Unity filters developer sessions.
7. **If an objective uses a gaze or fixation step, confirm dynamic objects exist on that SDK and framework first.** If it uses an ExitPoll answer, confirm ExitPoll exists.
8. **Do not let `queryable_data.md` over-promise.** Its automatic-sensor and device-field lists are written from the Unity SDK. The automatic capture table above is the cross-SDK version; where the two disagree, this file wins.
9. **If the project is Unreal, name the built-in component behind every comfort, performance, boundary or input metric**, and check that the three components depending on hand dynamic objects (Arm Length, Hand Elevation, Input Tracker) have that prerequisite met.
10. **If the project is Unreal and any plan row carries a numeric property, state which authoring surface sends it.** Blueprint stringifies values.
11. **If the project is Android XR, count the properties on every event.** Ten is the cap and the surplus is dropped silently. If an event needs a dynamic object link, that costs a slot too.
12. **If the project is Android XR or WebXR, confirm the asset pipeline can emit glTF Separate.** The Upload Web App takes `.gltf` plus `.bin` and rejects GLB, which is what most pipelines produce by default.

---

## When a team runs more than one SDK

Some organizations ship the same experience to several targets: an engine build for headsets, a WebXR build for the browser, a native Android XR build for a specific device. Two things matter:

- **Event names, property keys and units should be identical across all of them.** They land in the same project and the same queries. A divergence here is the one mistake that is expensive to undo later, because it splits every series permanently. Write the conventions document once and apply it to every build (SKILL.md rule 3). Pay particular attention to property **types** and **counts**: an event that is numeric on one SDK and stringified on another has diverged even when the names match, and an event designed for an unconstrained SDK may not fit Android XR's ten-property cap, which forces a choice between trimming everywhere or accepting divergence. Trim everywhere.
- **Scene IDs and dynamic object IDs are per-project, not per-SDK.** If multiple builds report to one project, decide deliberately whether they share a scene or upload separate ones. Sharing is right when it is the same environment and you want combined replay and heatmaps; separate scenes are right when the geometry genuinely differs, since mismatched geometry makes replay misleading rather than merely imperfect.

Cross-engine background: https://docs.cognitive3d.com/scenarios/mixing-unreal-unity/
