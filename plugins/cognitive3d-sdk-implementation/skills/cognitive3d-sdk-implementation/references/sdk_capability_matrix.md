# Cross-SDK capability matrix

_Load this before finalizing any plan (SKILL.md Step 5) so the plan only contains things the target SDK can actually do._

This file answers one question: **is this recommendation possible on the SDK this project uses?** It does not explain how to implement anything. For that, load the SDK reference for the project's target: `unity_sdk_reference.md`, `unreal_sdk_reference.md`, or `webxr_sdk_reference.md`.

The strategy in `data_strategy.md` and the archetype playbooks in `playbooks.md` are deliberately SDK-neutral, which means they will happily suggest something the target SDK does not support. This file is the screen that catches it.

---

## Primitive availability

| Primitive | Unity | Unreal | WebXR | Notes |
| --- | --- | --- | --- | --- |
| Custom events | yes | yes | yes | WebXR requires an explicit `[x, y, z]` position argument; Unity and Unreal infer it |
| Typed event properties | yes | **C++ only** | yes | Unreal's Blueprint `Send` variant stringifies every value |
| Session properties | yes | yes | yes | equivalent |
| Participant properties | yes | yes | yes | equivalent |
| Session tags | yes | yes | yes | equivalent |
| Sensors (custom) | yes | yes, float only | yes | Unreal caps at 10 Hz by default and takes floats only |
| Sensors (automatic) | broad | **opt-in components** | narrower | see below |
| Dynamic objects | yes | yes | **Three.js and Mattercraft only** | the single biggest planning constraint on WebXR |
| ID pools for spawned objects | yes | yes | no | WebXR registers each instance at spawn |
| Gaze stream | yes | yes | yes | |
| Per-object gaze, heatmaps, fixation objectives | yes | yes | **Three.js and Mattercraft only** | needs dynamic objects, so it follows their availability |
| Eye-tracked fixations | yes, on supported hardware | yes, via Fixation Recorder plus an eye tracking SDK | hardware and browser dependent | verify the integration list live |
| Objectives | yes | yes | yes | platform resource, SDK-independent |
| ExitPoll | yes, UI shipped | yes, UMG widgets shipped | yes, **but no UI is provided** | WebXR apps must render the survey themselves |
| Scene upload | in-engine tools | in-engine tools | Upload Web App | different workflow, same result |
| Dynamic object mesh upload | in-engine tools | Dynamic Object Manager | Upload Web App | separate from scene upload on all three |
| Remote controls | yes | yes | **not documented** | verify live before promising on WebXR |
| Local cache / offline upload | yes | yes | **not documented** | verify live before promising on WebXR |
| Media and 360 | yes | yes | **not documented** | verify live before promising on WebXR |
| Multiplayer | yes, components | yes, Lobby IDs | `setLobbyId` only | Unity ships Netcode and Normcore integrations |
| Audio recording | yes | **not documented** | **not documented** | Unreal ExitPoll voice panels are a separate, narrower feature |
| Active Session View | yes | yes | no | |
| Ready Room | yes | no | no | Unity-only onboarding scene |
| Attributions | yes | yes | verify live | |
| LMS forwarding | yes | yes | yes | configured on the dashboard, SDK-independent |

"Not documented" means the feature has no page in that SDK's docs section. Treat it as unavailable for planning purposes, and verify live before telling a team either way.

---

## Automatic capture differences

All three SDKs capture a base layer with no instrumentation (see `queryable_data.md`, which is written from Unity's behaviour). Where they differ:

| Automatic capture | Unity | Unreal | WebXR |
| --- | --- | --- | --- |
| Session lifecycle events | yes | yes | yes |
| Head and controller pose | yes | yes | yes |
| Gaze | yes | yes, via Player Tracker | yes |
| HMD orientation | yes | **component** | yes |
| FPS and 1% low | yes | **component** | yes |
| Boundary / room size | yes | **component** | only in a `bounded-floor` session |
| Controller tracking loss | yes | **component** | yes |
| Battery | yes | **component** | no |
| Hand and arm ergonomics | yes | **component**, plus hand dynamic objects | partial, controller height only |
| Controller button input | yes | **component**, plus input definitions | no |
| Draw calls, memory, main thread time | yes | no | only when the renderer is passed to the `C3D` constructor |
| CPU/GPU level, passthrough, wifi | yes (Android plugin) | partial, see Android plugin page | no |
| Biometrics (HP Omnicept, HarmonEyes) | yes, on supported hardware | sensor API, integration dependent | no |
| Multiplayer ping and variance | yes, via components | via Lobby IDs, verify | no |
| Editor sessions excluded from dashboards | yes | **no, recorded and shown behind a toggle** | **no equivalent** |

Two rows change a Phase 1 recommendation:

- **"Component" is not "automatic."** On Unreal, most comfort, performance and boundary metrics come from built-in Blueprint Script Macro components the team adds. A plan that assumes they are free will produce empty dashboard widgets. Name the component.
- **Only Unity filters developer sessions.** Unreal records editor sessions and shows them behind a toggle; WebXR has no concept of an editor session at all. On both, explicit `development_mode` separation is required rather than advisory.

---

## Effort differences for the same recommendation

Same plan item, materially different cost depending on the SDK. Say so when it applies, because a team reading another SDK's docs will have the wrong expectation.

| Plan item | Unity | Unreal | WebXR |
| --- | --- | --- | --- |
| Exit poll at end of module | low: place a prefab hook | low to medium: Blueprint nodes plus shipped UMG widgets, and a Widget Interaction component on the player | **medium to high**: fetch the set, build the in-scene UI, submit answers |
| Track ten dynamic objects | low to medium: components plus batch mesh upload | low to medium: components plus Dynamic Object Manager upload | medium: per-object tagging, registration and individual upload |
| Track spawned objects | ID Pool | ID Pool Asset, sized to concurrent spawns | manual registration per spawn |
| Comfort / FPS / room metrics | free | add the built-in components | partial, and profiler needs the renderer passed in |
| Numeric event properties | free | free in C++, **broken in Blueprint** | free |
| Scene upload | in-engine, one action | in-engine; multi-level setups need a combine strategy | export, then upload four files to the web app |
| Dev/prod separation | session property; editor already excluded | session property; editor sessions are visible | session property, and it is the only thing separating dev traffic |
| Install prerequisites | UPM, no project changes | **project must be C++ based**; Blueprint-only projects convert first | npm, Node 20+ |

---

## Screening rules

Apply these before presenting any plan:

1. **If the project is WebXR and not Three.js or Mattercraft, remove every dynamic object row.** Replace object-level gaze and interaction questions with custom events carrying an object identifier property, and state plainly what that cannot answer: dwell before action, attention without interaction, object heatmaps.
2. **If the project is plain JS or any WebXR framework without dynamic objects, remove every per-object attention recommendation.** A gaze stream still exists, but heatmaps, object attention analysis and fixation-based objectives do not. Confirm the framework during discovery rather than after the plan is written.
3. **If the plan contains an exit poll and the project is WebXR, scope the UI work as a line item.** It is not free. On Unreal, scope the Widget Interaction component instead: small, but the survey is unanswerable without it.
4. **If the plan contains remote controls, local cache, media, audio recording or multiplayer components and the project is WebXR, verify live before including them.** Do not carry them over from an engine-shaped plan.
5. **If the plan relies on biometric or device-level sensors, confirm the SDK and the hardware both support them.** Most of that list is Unity-first.
6. **If the project is Unreal or WebXR, dev/prod separation is Phase 1 and non-negotiable.** Neither filters developer sessions the way Unity does.
7. **If an objective uses a gaze or fixation step, confirm dynamic objects exist on that SDK and framework first.** Gaze steps reference dynamic object IDs, so without dynamic objects the objective cannot be built.
8. **Do not let `queryable_data.md` over-promise.** Its automatic-sensor and device-field lists are written from the Unity SDK. The automatic capture table above is the cross-SDK version; where the two disagree, this file wins.
9. **If the project is Unreal, name the built-in component behind every comfort, performance, boundary or input metric in the plan**, and check that the three components depending on hand dynamic objects (Arm Length, Hand Elevation, Input Tracker) have that prerequisite met.
10. **If the project is Unreal and any plan row carries a numeric property, state which authoring surface sends it.** Blueprint stringifies values. A `duration_seconds` that arrives as text cannot be averaged or charted, and nothing about the dashboard makes that obvious.

---

## When a team runs more than one SDK

Some organizations ship the same experience to two targets: a Unity or Unreal build for headsets and a WebXR build for the browser, or an Unreal flagship alongside a Unity pilot. Two things matter:

- **Event names, property keys and units should be identical across all of them.** They land in the same project and the same queries. A divergence here is the one mistake that is expensive to undo later, because it splits every series permanently. Write the conventions document once and apply it to every build (SKILL.md rule 3). Pay particular attention to property **types**: an event that is numeric on one SDK and stringified on another is a divergence even when the names match perfectly.
- **Scene IDs and dynamic object IDs are per-project, not per-SDK.** If multiple builds report to one project, decide deliberately whether they share a scene or upload separate ones. Sharing is right when it is the same environment and you want combined replay and heatmaps; separate scenes are right when the geometry genuinely differs, since mismatched geometry makes replay misleading rather than merely imperfect.

Cross-engine background: https://docs.cognitive3d.com/scenarios/mixing-unreal-unity/
