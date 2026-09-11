# Cross-SDK capability matrix

_Load this before finalizing any plan (SKILL.md Step 5) so the plan only contains things the target SDK can actually do._

This file answers one question: **is this recommendation possible on the SDK this project uses?** It does not explain how to implement anything. For that, load the SDK reference for the project's target: `unity_sdk_reference.md`, `unreal_sdk_reference.md`, `visionos_sdk_reference.md`, `androidxr_sdk_reference.md`, or `webxr_sdk_reference.md`.

The strategy in `data_strategy.md` and the archetype playbooks in `playbooks.md` are deliberately SDK-neutral, which means they will happily suggest something the target SDK does not support. This file is the screen that catches it.

**Two column headings name a toolchain, not a device.** "visionOS" means a native Swift app; a Unity app shipped to Vision Pro is in the Unity column. "Android XR" means the native Kotlin SDK; a Unity or Unreal app on an Android headset is in its engine's column.

---

## The one hard platform limit

**visionOS exposes no eye tracking to applications.** Apple does not provide eye-tracking rays to apps, so the visionOS SDK records HMD forward direction as the gaze signal. Everything downstream of that — heatmaps, dwell, gaze objectives, "what did they look at" — measures head orientation on this platform and eye attention on eye-tracked hardware elsewhere.

No SDK release changes this. When a plan's core question is about eye attention and the target is Vision Pro, the answer is to reframe the question, not to instrument harder. This is the only entry in this file that is a property of the hardware platform rather than of an SDK's maturity.

---

## Primitive availability

| Primitive | Unity | Unreal | visionOS | Android XR | WebXR |
| --- | --- | --- | --- | --- | --- |
| Custom events | yes | yes | yes | yes | yes |
| Typed event properties | yes | **C++ only** | **no, string-only** | yes | yes |
| Event property count | unlimited | unlimited | unlimited | **10 per event** | unlimited |
| Event duration | yes | yes | yes, by deferring send | verify live | yes |
| Session properties | yes | yes | yes, typed | yes | yes |
| Participant properties | yes | yes | yes | yes | yes |
| Session tags | yes | yes | **not documented** | **not documented** | yes |
| Session name | yes | yes | **not documented** | **not documented** | yes |
| Sensors (custom) | yes | yes, float only | verify live | yes, float only | yes |
| Sensors (automatic) | broad | **opt-in components** | narrow, see below | **FPS only** | moderate |
| Dynamic objects | yes | yes | yes | yes | **Three.js and Mattercraft only** |
| Dynamic object linked to an event | parameter | parameter | parameter | **by property convention** | parameter |
| ID pools for spawned objects | yes | yes | no | no | no |
| Gaze stream | yes | yes | yes, **head direction only** | yes, when `enable_gaze` | yes |
| Per-object gaze and heatmaps | yes | yes | yes, head-direction based | yes | **Three.js and Mattercraft only** |
| Eye-tracked fixations | yes, on supported hardware | yes, via Fixation Recorder | **never, platform restriction** | verify live | hardware dependent |
| Objectives | yes | yes | yes | yes | yes |
| ExitPoll | yes, UI shipped | yes, UMG widgets shipped | **yes, SwiftUI views shipped** | **not documented** | yes, but no UI provided |
| ExitPoll offline | verify live | verify live | **yes, question sets cached** | n/a | no |
| Scene upload | in-engine tools | in-engine tools | Upload Web App | Upload Web App | Upload Web App |
| Dynamic object mesh upload | in-engine tools | Dynamic Object Manager | Upload Web App | Upload Web App | Upload Web App |
| Local cache / offline upload | yes | yes | **yes, well specified** | **not documented** | **not documented** |
| Remote controls | yes | yes | **not documented** | **not documented** | **not documented** |
| Media and 360 | yes | yes | **not documented** | **not documented** | **not documented** |
| Multiplayer | yes, components | yes, Lobby IDs | **not documented** | **not documented** | `setLobbyId` only |
| Audio recording | yes | **not documented** | ExitPoll voice only | **not documented** | **not documented** |
| Active Session View | yes | yes | **not documented** | **not documented** | no |
| Ready Room | yes | no | no | no | no |
| LMS forwarding | yes | yes | yes | yes | yes |

"Not documented" means the feature has no page in that SDK's docs section. Treat it as unavailable for planning purposes, and verify live before telling a team either way. Absence is weakest evidence on the newest SDKs, visionOS and Android XR.

---

## Automatic capture differences

All five SDKs capture some base layer with no instrumentation (see `queryable_data.md`, which is written from Unity's behaviour). Where they differ:

| Automatic capture | Unity | Unreal | visionOS | Android XR | WebXR |
| --- | --- | --- | --- | --- | --- |
| Session lifecycle events | yes | yes | yes | yes | yes |
| Head pose / position | yes | yes | yes | yes | yes |
| Head pitch | yes | **component** | yes | verify live | yes |
| Head yaw | yes | **component** | **v1.0.1+, off by default** | verify live | yes |
| Head roll | yes | verify live | **not recorded** | verify live | verify live |
| Gaze | yes | yes, via Player Tracker | yes, **head direction** | yes, when enabled | yes |
| Hands | yes | yes | yes, when `isHandTrackingRequired` | yes | yes |
| Controllers | yes | yes | n/a, no controllers | yes | yes |
| FPS | yes | **component** | yes | yes | yes |
| Battery | yes | **component** | yes | verify live | no |
| Participant height | verify live | **component** | **yes, `c3d.participant.height`** | verify live | no |
| Boundary / room size | yes | **component** | n/a on this platform | verify live | `bounded-floor` only |
| Draw calls, memory, main thread | yes | no | verify live | no | renderer passed in only |
| Biometrics | yes, on supported hardware | integration dependent | no | no | no |
| Editor sessions excluded | yes | **no, behind a toggle** | **no equivalent** | **no equivalent** | **no equivalent** |

Three rows change a Phase 1 recommendation:

- **"Component" is not "automatic."** On Unreal, most comfort, performance and boundary metrics come from built-in components the team adds. Name the component.
- **Android XR documents only FPS as automatic**, and visionOS is narrow too, though it gives participant height for free, which nothing else does.
- **Only Unity filters developer sessions.** On every other target, explicit `development_mode` separation is required rather than advisory.

---

## Effort differences for the same recommendation

| Plan item | Unity | Unreal | visionOS | Android XR | WebXR |
| --- | --- | --- | --- | --- | --- |
| Exit poll at end of module | low: place a prefab hook | low to medium: Blueprint nodes plus widgets and a Widget Interaction component | low: view model plus six shipped SwiftUI views, offline-capable | **verify support first** | **medium to high**: build the UI yourself |
| Track ten dynamic objects | low to medium | low to medium | medium: register each, upload each via web app | medium | medium |
| Track spawned objects | ID Pool | ID Pool Asset | register per instance | register per instance | register per instance |
| Comfort / FPS / device metrics | free | add the components | mostly free, narrow set | FPS free, rest is your own sampling | partial |
| Numeric event properties | free | free in C++, **broken in Blueprint** | **broken; put them on the session** | free, within 10 | free |
| Eye-attention analysis | free on eye-tracked hardware | free on eye-tracked hardware | **impossible** | verify live | hardware dependent |
| Offline operation | supported | supported | **supported and specified** | not documented | not documented |
| Install prerequisites | UPM | **project must be C++ based** | local Swift package, not a remote URL | Gradle plus AndroidX XR alpha alignment | npm, Node 20+ |

---

## Screening rules

Apply these before presenting any plan:

1. **If the project is WebXR and not Three.js or Mattercraft, remove every dynamic object row.** Replace object-level gaze and interaction questions with custom events carrying an object identifier property, and state what that cannot answer.
2. **If the project is plain JS or any WebXR framework without dynamic objects, remove every per-object attention recommendation.**
3. **If the plan contains an exit poll, check the target supports one.** WebXR: scope the UI work. Unreal: scope the Widget Interaction component. **Android XR: verify ExitPoll exists at all.** visionOS and Unity: it is cheap, so place hooks early.
4. **If the plan contains remote controls, local cache, media, audio recording or multiplayer and the project is not Unity or Unreal, verify live before including them.** visionOS local cache is the exception: it is documented and solid.
5. **If the plan relies on biometric or device-level sensors, confirm the SDK and the hardware both support them.**
6. **If the project is anything but Unity, dev/prod separation is Phase 1 and non-negotiable.**
7. **If an objective uses a gaze or fixation step, confirm dynamic objects exist on that SDK.** On visionOS the objective will build and score, but it scores head direction: name it accordingly ("faced the notice", not "read the notice").
8. **Do not let `queryable_data.md` over-promise.** Its lists are written from the Unity SDK; where it and the table above disagree, this file wins.
9. **If the project is Unreal, name the built-in component behind every comfort, performance, boundary or input metric**, and check the three that need hand dynamic objects.
10. **If the project is Unreal and any plan row carries a numeric property, state which authoring surface sends it.** Blueprint stringifies values.
11. **If the project is Android XR, count the properties on every event.** Ten is the cap and the surplus is dropped silently.
12. **If the project is visionOS or Android XR, confirm the asset pipeline can emit glTF Separate.** The Upload Web App rejects GLB, which is what most pipelines produce by default.
13. **If the project is visionOS and any plan row carries a numeric event property, move it to the session or accept the loss.** Custom event properties are string-only with no typed alternative.
14. **If the project is visionOS, restate every attention claim in the plan as head direction.** Then check whether the business question survives that restatement. If it does not, raise it during discovery rather than after the dashboard is full.

---

## When a team runs more than one SDK

Some organizations ship one experience to several targets: an engine build for headsets, a native build per platform, a WebXR build for the browser. Two things matter:

- **Event names, property keys and units should be identical across all of them.** They land in the same project and the same queries, and divergence splits every series permanently. Write the conventions document once and apply it everywhere (SKILL.md rule 3). Watch property **types** and **counts** especially: an event that is numeric on one SDK, stringified on another and capped on a third has effectively diverged three ways while looking identical in the plan. Design to the tightest constraint in the set.
- **Scene IDs and dynamic object IDs are per-project, not per-SDK.** If multiple builds report to one project, decide deliberately whether they share a scene or upload separate ones.

**One caution specific to mixed Vision Pro estates.** A team running a native visionOS build alongside a Unity build on eye-tracked hardware will have gaze data that means two different things under one property name. Either separate them with a session property that records the platform, or keep the analyses apart. Merging them silently produces an attention metric that is an average of two incompatible measurements.

Cross-engine background: https://docs.cognitive3d.com/scenarios/mixing-unreal-unity/
