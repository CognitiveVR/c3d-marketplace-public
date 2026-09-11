# Cross-SDK capability matrix

_Load this before finalizing any plan (SKILL.md Step 5) so the plan only contains things the target SDK can actually do._

This file answers one question: **is this recommendation possible on the SDK this project uses?** It does not explain how to implement anything. For that, load the SDK reference for the project's target: `unity_sdk_reference.md` or `webxr_sdk_reference.md`.

The strategy in `data_strategy.md` and the archetype playbooks in `playbooks.md` are deliberately SDK-neutral, which means they will happily suggest something the target SDK does not support. This file is the screen that catches it.

---

## Primitive availability

| Primitive | Unity | WebXR | Notes |
| --- | --- | --- | --- |
| Custom events | yes | yes | WebXR requires an explicit `[x, y, z]` position argument |
| Session properties | yes | yes | equivalent |
| Participant properties | yes | yes | equivalent |
| Session tags | yes | yes | equivalent |
| Sensors (custom) | yes | yes | equivalent |
| Sensors (automatic) | broad | narrower | see below |
| Dynamic objects | yes | **Three.js and Mattercraft only** | the single biggest planning constraint on WebXR |
| Gaze stream | yes | yes | core SDK captures viewer-pose gaze from the `XRSession` |
| Per-object gaze, heatmaps, fixation objectives | yes | **Three.js and Mattercraft only** | needs dynamic objects, so it follows their availability |
| Objectives | yes | yes | platform resource, SDK-independent |
| ExitPoll | yes | yes, **but no UI is provided** | WebXR apps must render the survey themselves |
| Scene upload | in-engine tools | Upload Web App | different workflow, same result |
| Dynamic object mesh upload | in-engine tools | Upload Web App | separate from scene upload in both |
| Remote controls | yes | **not documented** | verify live before promising on WebXR |
| Local cache / offline upload | yes | **not documented** | verify live before promising on WebXR |
| Media and 360 | yes | **not documented** | verify live before promising on WebXR |
| Multiplayer components | yes | `setLobbyId` only | Unity ships Netcode and Normcore integrations |
| Audio recording | yes | **not documented** | verify live before promising on WebXR |
| Ready Room | yes | no | Unity-only onboarding scene |
| Active Session View | yes | no | Unity-only in-editor live view |
| LMS forwarding | yes | yes | configured on the dashboard, SDK-independent |

"Not documented" means the feature has no page in that SDK's docs section. Treat it as unavailable for planning purposes, and verify live before telling a team either way.

---

## Automatic capture differences

Both SDKs capture a large base layer with no instrumentation (see `queryable_data.md`). Where they differ:

| Automatic capture | Unity | WebXR |
| --- | --- | --- |
| Session lifecycle events | yes | yes |
| HMD orientation | yes | yes |
| Controller height / ergonomics | yes | yes, when controllers are tracked |
| FPS and 1% low | yes | yes |
| Boundary / room size | yes | only in a `bounded-floor` session |
| Draw calls, memory, main thread time | yes | only when the renderer is passed to the `C3D` constructor |
| Battery, CPU/GPU level, passthrough, wifi | yes (Android plugin) | no |
| Biometrics (HP Omnicept, HarmonEyes) | yes, on supported hardware | no |
| Eye tracking pupil diameter | yes, on supported hardware | no |
| Multiplayer ping and variance | yes, via components | no |
| Editor sessions auto-excluded from dashboards | yes | **no equivalent** |

The last row changes a Phase 1 recommendation. On Unity you can tell a team that in-editor testing is already filtered out; on WebXR, local development produces indistinguishable real sessions, so `development_mode` separation is mandatory rather than advisory.

---

## Effort differences for the same recommendation

Same plan item, materially different cost depending on the SDK. Say so when it applies, because a team reading the other SDK's docs will have the wrong expectation.

| Plan item | Unity effort | WebXR effort |
| --- | --- | --- |
| Exit poll at end of module | low: place a prefab hook | **medium to high**: fetch the set, build the in-scene UI, submit answers |
| Track ten dynamic objects | low to medium: components plus batch mesh upload | medium: per-object tagging, registration and individual upload |
| Track spawned objects | ID Pool | manual `registerObjectCustomId` per spawn |
| Controller tracking | automatic in most rigs | automatic when the browser reports input sources |
| Scene upload | in-engine, one action | export, then upload four files to the web app |
| Dev/prod separation | session property, editor already excluded | session property, and it is the only thing separating dev traffic |
| Custom shader handling | may need an exporter class | not applicable; export fidelity varies by adapter instead |

---

## Screening rules

Apply these before presenting any plan:

1. **If the project is WebXR and not Three.js or Mattercraft, remove every dynamic object row.** Replace object-level gaze and interaction questions with custom events carrying an object identifier property, and state plainly in the plan what that cannot answer: dwell before action, attention without interaction, object heatmaps.
2. **If the project is plain JS or any WebXR framework without dynamic objects, remove every per-object attention recommendation.** A gaze stream still exists, but heatmaps, object attention analysis and fixation-based objectives do not. Confirm the framework during discovery rather than after the plan is written.
3. **If the plan contains an exit poll and the project is WebXR, scope the UI work as a line item.** It is not free.
4. **If the plan contains remote controls, local cache, media, audio recording or multiplayer components and the project is WebXR, verify live before including them.** Do not carry them over from a Unity-shaped plan.
5. **If the plan relies on biometric or device-level sensors, confirm the SDK and the hardware both support them.** Most of that list is Unity-only.
6. **If the project is WebXR, dev/prod separation is Phase 1 and non-negotiable.** There is no editor exclusion to fall back on.
7. **If an objective uses a gaze or fixation step, confirm dynamic objects exist on that SDK and framework first.** Gaze steps reference dynamic object IDs, so without dynamic objects the objective cannot be built.
8. **Do not let `queryable_data.md` over-promise on WebXR.** Its automatic-sensor and device-field lists are written from the Unity SDK. The automatic capture table above is the WebXR-accurate version; where the two disagree, this file wins for WebXR projects.

---

## When a team runs both

Some organizations ship the same experience to Unity and WebXR. Two things matter:

- **Event names, property keys and units should be identical across both.** They land in the same project and the same queries. A divergence here is the one mistake that is expensive to undo later, because it splits every series permanently. Write the conventions document once and apply it to both (SKILL.md rule 3).
- **Scene IDs and dynamic object IDs are per-project, not per-SDK.** If both builds report to one project, decide deliberately whether they share a scene or upload separate ones. Sharing is right when it is the same environment and you want combined replay and heatmaps; separate scenes are right when the geometry genuinely differs, since mismatched geometry makes replay misleading rather than merely imperfect.

Cross-engine background: https://docs.cognitive3d.com/scenarios/mixing-unreal-unity/
