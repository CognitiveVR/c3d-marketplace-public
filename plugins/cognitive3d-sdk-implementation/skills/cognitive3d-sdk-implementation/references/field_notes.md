# Cognitive3D Field Notes

_Lessons from real integrations. Use these alongside the playbooks to avoid common pitfalls and catch high-leverage opportunities._

This file is organized by topic, not by archetype. Some notes apply universally; others are tagged with the archetypes or overlays where they matter most.

Each note carries an **Applies to** line covering archetypes and, where it differs, **SDKs**. A note with no SDK qualifier applies to every target. Skip notes tagged for an SDK the project does not use.

## How to use this file

After choosing your archetypes and pulling the relevant playbooks, scan the topics below for anything that applies. The playbooks tell you what to recommend. This file tells you what to watch out for and where the easy wins are.

AI assistants should read this file after `playbooks.md` and before filling out `track_plan_template.md`.

---

## Controller and boundary tracking verification

**Applies to:** all projects, all SDKs

This is the single most common issue found during integration reviews.

Controllers should be tracked. If they are not, many built-in dashboard metrics — hand/controller height, ergonomics scoring, input tracking — will be missing or incorrect.

- **Unity:** the Project Setup flow detects and tracks the HMD, tracking space and controllers automatically by default, but SteamVR rigs require assigning the controller GameObjects manually in the Project Setup window, and older SDK versions or unusual configurations may also need manual setup.
- **visionOS:** there are no controllers; hands are the input, tracked when `isHandTrackingRequired` is set and registered via `registerHand(id:isRightHand:)` with the `hand_left` / `hand_right` tags. There is no boundary or room-size concept on this platform either, so drop both from an Apple Vision Pro validation checklist rather than marking them failed.
- **Android XR:** controllers and hands are tracked automatically and separately, so do not register them as dynamic objects. Gaze depends on `enable_gaze` in `cognitive3d.json`, which is the first thing to check when a session has pose data but no gaze.
- **Unreal:** controllers need Dynamic Object components on the motion controller hand meshes, with Controller Type set and the Set Left Hand / Set Right Hand buttons used; spawned controllers assign the hand value in the Constructor graph. Three built-in components (Arm Length, Hand Elevation, Input Tracker) silently produce nothing until hand dynamics are configured, so verify the dynamics before blaming the components.
- **WebXR:** controller tracking follows the browser's reported input sources, so there is no rig to configure, but it fails silently when the app never enters an immersive session with controllers present, or when the adapter's per-frame `update()` is not running.

Boundary tracking is equally important and equally easy to miss. You can verify it on any session detail page: look at the top of the page for the purple icons showing all active data streams. If boundary is missing, physical-versus-virtual movement metrics will not work.

Three SDK-specific prerequisites: on **Unreal**, boundary events and room size come from built-in components that must be added, and Room Size additionally needs UE 4.27+ (or 4.26+ on Oculus/Meta). On **WebXR**, room-size data requires a `bounded-floor` session, so an app that requests only `local-floor` will never produce it. On **Android XR**, boundary capture is not clearly documented, so verify live rather than putting it in a validation checklist that will fail for a reason nobody can diagnose.

**Recommendation:** Always include a controller and boundary verification step in the validation checklist, regardless of project type.

---

## Exit poll hooks are cheap on most SDKs, expensive on WebXR, and unconfirmed on Android XR

**Applies to:** all projects, all SDKs

Exit poll hooks should be placed at the beginning and end of the experience even if the team does not have survey questions ready yet. Questions are configured on the platform — on the dashboard or programmatically via the MCP server — and can be added, changed, or removed without shipping a new build. (One caveat when changing them: editing a question set creates a new version, and existing hooks stay pointed at the old version until explicitly reassigned — see the ExitPoll platform constraints in the SDK reference.) The hook itself must exist in the app.

Stakeholders inevitably ask "can we survey users about X?" weeks or months after launch. If hooks are already in place, a survey can be live in minutes. Without them, it requires a code change, a new build, and a store submission.

**First: confirm the SDK has ExitPoll.** It is documented for Unity, Unreal, visionOS, WebXR and C++, and has no Android XR page. If a team on Android XR needs self-report and ExitPoll is genuinely unavailable, the substitute is the app's own UI writing answers as a custom event. That works, and it gives up the thing that makes this note worth writing: questions can no longer be added or changed without a release, so the "hooks are nearly free insurance" argument does not apply and the survey has to be designed before ship rather than after.

**The in-app cost is not the same across SDKs.** Unity ships survey UI you place in the scene. visionOS ships six SwiftUI question views and a view model, and caches question sets locally so a survey still works with no connectivity, which is worth naming for field, travel and clinical deployments. Unreal ships UMG widgets and actors under Show Plugin Content, driven by three Blueprint nodes, with one easily-missed prerequisite: a **Widget Interaction component** on the player or motion controller, without which the panel renders and cannot be answered. That presents as an unresponsive survey rather than an error, and it is worth naming up front. The WebXR SDK fetches the question set and submits answers but renders nothing, so a WebXR team builds the survey UI themselves. The argument for placing hooks early is the same everywhere, but on WebXR the work is real and belongs in the plan as its own line item rather than as a footnote to a hook.

**Recommendation:** Treat exit poll hooks as part of the Phase 1 foundation, not as a Phase 2 or Phase 3 addition, on any SDK that supports them. On WebXR, scope the survey UI alongside them. On Unreal, check the Widget Interaction component during validation. On visionOS, check `NSMicrophoneUsageDescription` if voice questions are in scope. On Android XR, verify support before the plan commits to it.

---

## First-time user experience and the two-minute window

**Applies to:** progression and mechanics, content and wellness, exploration and evaluation. All SDKs

Users often decide within the first two minutes whether to abandon an app entirely. Getting to the spatial payoff — the moment the experience feels worth continuing — within that window is critical for long-term retention.

This makes FTUE tracking one of the highest-value instrumentation investments across almost all project types. Duration per stage, not just completion, is what reveals whether onboarding is fast enough.

**Recommendation:** Always track FTUE stages with durations. If the team is debating onboarding approaches, flag that this is a strong candidate for A/B testing: with remote controls on Unity or Unreal, or with the app's own feature-flag mechanism on visionOS, Android XR and WebXR, where remote controls are not documented. Either way the requirement is that the assigned variant is recorded as a session property.

---

## Dev/prod separation, and what the SDK does or does not do for you

**Applies to:** all projects, all SDKs

**Unity:** sessions run in the Editor are automatically excluded from major analytics on the dashboard, and can be toggled back on with a button in the bottom left corner. For builds deployed to devices during development, a `development_mode` session property or a "Development" session tag is still necessary, or test sessions from sideloaded builds will pollute production dashboards.

**Unreal: editor sessions are recorded, not excluded.** Pressing Play creates a session that appears on the dashboard behind an "Editor Mode" toggle. That is visibility, not filtering, so every iteration cycle adds real sessions to the project.

**visionOS, Android XR and WebXR have no editor-session concept at all.** A developer running a debug build on a headset, or the app on localhost, produces sessions indistinguishable from production traffic. Nothing filters them, and by the time anyone notices, the dashboard already holds weeks of them and the fix is not retroactive. Setting `development_mode` from the build variant costs one line and solves it permanently.

Some teams use a separate dashboard project for heavy testing. A session property is the simplest approach if the team wants everything in one project.

**Recommendation:** Always include dev/prod separation in Phase 1. On Unity, mention the editor auto-exclusion so developers do not duplicate effort. On every other SDK, state plainly that nothing is excluded for them.

---

## Gaze is not the same measurement on every platform

**Applies to:** all archetypes that lean on attention, especially exploration and evaluation. Critical on visionOS

On eye-tracked hardware, gaze means where the eyes went. On **Apple Vision Pro it means where the head was pointed**, because visionOS exposes no eye-tracking rays to applications. That is an Apple platform privacy restriction, not an SDK gap, and no future release lifts it. The docs are explicit: "this is not eye tracking."

The practical difference is larger than it sounds on a device designed around eye input. A Vision Pro user can read a whole panel, compare two objects, or study a label with their head almost still. Head-direction gaze misses all of that, and it credits attention to whatever happens to be centred while the eyes are elsewhere.

What this changes:

- **Heatmaps and dwell still work and still mean something real** — orientation, body positioning, what people turned toward. Describe them that way in the plan and in the dashboard readout, not as eye attention.
- **Gaze-step objectives still build and score.** Name them for what they measure: "faced the safety notice", not "read the safety notice".
- **An exploration or evaluation plan needs a second look.** Those archetypes lean hardest on "what drew attention without interaction", which is exactly the question this platform cannot answer well.
- **If eye attention is the actual business question**, say so during discovery. No vendor can answer it on Vision Pro, and the question has to be reframed around interaction, dwell-by-orientation and self-report instead. Finding this out after the dashboard fills up is a bad conversation.

A mixed estate makes it worse: a team running a native Vision Pro build alongside an eye-tracked Unity build will have two incompatible measurements under one property name. Record the platform as a session property and keep the analyses apart, or accept an attention metric that averages two different things.

**Recommendation:** On any Vision Pro project, establish during discovery whether the team assumes eye tracking. Most do. Correct it before the plan is written, not after.

---

## Dynamic objects: quality over quantity

**Applies to:** all projects. Unity, Unreal, visionOS and Android XR, and WebXR on Three.js and Mattercraft only

Do not recommend dynamic objects on every interactable in the scene. Only add them where object-level attention or interaction data answers a real question.

Objects that spawn repeatedly — individual bullets, individual hit targets, particle effects — should generally not be tracked as dynamic objects. This pollutes the object list on the dashboard and makes it difficult to find meaningful objects.

Good candidates are objects where gaze, fixation, or interaction context changes how you interpret the session: tools, equipment, instruction panels, prototypes, focal environmental features.

If the project needs to track spawned objects with meaningful identity, Unity and Unreal use ID Pools while visionOS, Android XR and WebXR register each instance explicitly at spawn time. On Unreal the Id Pool Asset holds a fixed array of GUIDs, so a pool smaller than the real concurrent spawn count silently drops tracking for the overflow. Ask what the actual maximum is rather than accepting a round number.

On visionOS, Android XR and WebXR the economics are harsher than on the engine SDKs. Every object needs explicit registration and its own mesh upload through the web app, with no batch tooling. A twenty-object list that is merely tedious in Unity is a genuine sprint there, so the "only where it answers a question" rule does more work.

One Android XR specific: the grouping key is `meshName`, not `name`. Instances that should aggregate must share a `meshName`, and it must match the uploaded model name or the dashboard cannot correlate geometry with data. Getting that wrong produces tracking with no visual, which looks like a failed upload and is not.

**Recommendation:** When listing dynamic objects in the track plan, always state why each object is included and what question it supports.

---

## Platform identity: Oculus Social and Steam

**Applies to:** progression and mechanics, content and wellness (consumer apps). Store identity is Unity-centric

For Meta Quest apps, the Unity SDK's OculusSocial component captures platform identity (Oculus username and ID). For Steam apps, saving the Steam username as participant name and Steam ID as participant ID achieves the same thing.

Platform identity enables review attribution — understanding why specific users left positive or negative reviews — and correlation with store-level metrics.

WebXR apps have no store identity to draw on, so the equivalent is whatever account system the site already has: set `setParticipantId` and `setParticipantFullName` from the existing login. Where the experience is genuinely anonymous, say so in the plan and drop cross-session analysis rather than inventing a fragile browser-local identifier.

visionOS, Android XR and Unreal all expose participant ID and name setters but no packaged store integration. visionOS adds one free participant metric nothing else has: `c3d.participant.height`, estimated automatically from sampled headset Y-position, which is useful for ergonomics and reach analysis without asking anyone anything.

The Unreal docs suggest a useful alternative where no login exists: isolate the participant in an empty scene before the session starts and collect identifying properties through an ExitPoll, rather than hardcoding them. **That route depends on ExitPoll, so it does not transfer to Android XR**, where ExitPoll has no documented support. On Android XR the options are the app's own identification UI writing `setParticipantId` and `setParticipantProperty`, or accepting anonymous sessions and saying so in the plan.

**Recommendation:** Flag this for any consumer app. It pairs well with participant tracking and is easy to miss.

---

## Content duration: planned versus actual

**Applies to:** content and wellness, performance and assessment

For content-based experiences (classes, meditations, workouts, training modules), recording both the planned content duration and the actual participation duration is important. A user who completes 4 minutes of a 30-minute class is a very different signal from a user who completes 28 minutes. Without both values, you cannot distinguish the two.

**Recommendation:** Include `content_duration_seconds` (planned length) and `duration_seconds` (actual time) on content lifecycle events.

---

## Scene export fidelity

**Applies to:** all projects, all SDKs. The specific failure differs by SDK

Replay is only as useful as the geometry behind it, and export problems are far cheaper to fix before the scene is uploaded than after. Check early.

**Unity — custom shaders.** When a project uses custom shaders, materials appear white on the dashboard because the GLTF exporter does not know how to map custom shader properties to standard PBR. If custom shaders are present, a shader-properties export script needs to be created: an Editor class inheriting `GLTFSceneExporter.ShaderPropertyCollection` (namespace `Cognitive3D.UnityGLTF`), placed under `Editor/GLTF`, mapping the shader's property names to PBR. The SDK ships examples to copy from — `URPShaderProperties`, `HDRPShaderProperties`, `StandardShaderProperties` — and discovers subclasses automatically via reflection; no registration step is needed. See the Custom Shaders section of https://docs.cognitive3d.com/unity/troubleshooting/.

**Unreal — materials and exporter quirks.** Complex materials often do not translate, so make the diffuse output representative on its own rather than expecting the full material graph to survive. Four specific traps from the troubleshooting page: Forward Shading can crash the glTF export (disable it for the export, re-enable afterwards), TextRenderers do not export at all, skeletal meshes need UE 4.26+ for correct materials, and Metahumans need Level of Detail 0 with hair and skeletal animation unsupported. "Only Export Selected" is the fix when far too much geometry comes through.

**visionOS — no exporter either.** Geometry comes from the team's own pipeline and goes up through the Upload Web App in glTF Separate, not GLB. One extra trap here: the first upload generates a Scene ID, and later uploads of the same scene must reuse it or the dashboard fills with duplicate scenes.

**Android XR — no exporter at all.** There is no engine to export from, so scene and object geometry comes from the team's own asset pipeline and goes up through the Upload Web App. The trap is format: the app accepts glTF Separate (`.gltf` plus `.bin`) and rejects GLB, which is what most pipelines emit by default. Confirm the pipeline can produce the right format before the plan assumes replay geometry will exist.

**WebXR — adapter export coverage.** Export is an adapter capability, not a given. Three.js and Mattercraft export scenes and objects; PlayCanvas and Babylon do not; Wonderland exports geometry only, with no materials or textures, so replay will be untextured. Where the adapter cannot export, the team produces the glTF another way and uploads it manually, or accepts replay without geometry. Establish which of those it is before the team sees an empty scene viewer and concludes the integration is broken.

**Recommendation:** Include a scene export fidelity check in the validation checklist on every project, and name the specific failure mode for the SDK in play.

---

## Shared devices: device ID is not enough

**Applies to:** shared devices and SSO overlay

In shared-device environments (enterprise training, classroom deployments, lab studies, kiosk setups), device ID alone will make every session from the same headset look like the same person.

Participant ID must be set explicitly at session start. IDs typically come from SSO, a device management platform, or a manual login screen. Participant properties (role, department, site, cohort, training level) should be set at the same time.

Do not store participant properties on the session. They belong on the participant profile so they follow the person across sessions.

**Recommendation:** For any project where shared devices are likely, make participant identification a Phase 1 priority.

---

## Recurring behavior versus milestones

**Applies to:** progression and mechanics, content and wellness

A common mistake is only tracking one-time milestones (first achievement, first completion, first purchase) when the real question depends on repeated behavior. Mechanic usage frequency, content repeat rate, practice cadence, and tool adoption are all recurring patterns that milestones alone cannot capture.

**Recommendation:** When reviewing a track plan, check whether the instrumentation captures both milestones and recurring behavior. If only milestones are present and the business questions involve frequency or habit, flag the gap.

---

## LMS and objective forwarding

**Applies to:** performance and assessment

Objectives can send completion data to external LMS platforms. Set up an LMS configuration in Organization Settings, then attach it to completion objectives.

If the team mentions LMS, xAPI, or external reporting, make sure the module completion objective is designed to carry the right data for forwarding. A vague completion event without score or attempt metadata will not satisfy most LMS requirements.

**Recommendation:** When LMS integration is a requirement, design the completion objective and its triggering events together so the forwarded data is useful from day one.

---

## Audio recording: high value, high sensitivity

**Applies to:** performance and assessment (training only). Unity; not documented for Unreal, Android XR or WebXR. visionOS records audio only inside an ExitPoll voice question, which is a narrower feature with its own Info.plist permission (Unreal's ExitPoll voice panels are a narrower, separate feature that needs `DefaultEngine.ini` configuration)

Audio recording captures in-session audio aligned to the session timeline. It is valuable for training scenarios where verbal communication matters — trainees explaining procedures, giving verbal responses, or communicating with virtual patients.

Audio recording requires explicit runtime permission and may require additional privacy disclosures. Do not recommend this casually.

**Recommendation:** Only recommend audio capture for training use cases where verbal output is directly relevant to the assessment. Always flag the consent and governance requirements.

---

## Multi-scene applications and scene timing

**Applies to:** all projects with multiple scenes, all SDKs

If the experience transitions between scenes (common in fitness apps, training apps with multiple modules, games with level loading), each scene should be uploaded to the dashboard.

On **WebXR** every scene also needs an entry in `allSceneData` with a real scene ID and the current version number, and the transition needs an explicit `setScene()` call; a scene that is uploaded but not declared in config produces sessions with no geometry.

On **visionOS** each scene needs its `sceneId` and `versionNumber` in `SceneData`, and the first upload generates the Scene ID that later uploads must reuse or duplicate scenes appear.

On **Android XR** every scene needs its ID and version in `cognitive3d.json`, and there is no in-engine exporter, so each scene's geometry comes from the team's own pipeline as glTF Separate and goes up through the web app individually.

On **Unreal** with level streaming, only the last-loaded level holding a valid Scene Id records session data, which surprises teams whose persistent level is not the one they uploaded. Make sublevels visible during export so geometry culling does not silently drop them, and consider the documented combine strategies when several levels make one visual space.

Scene transition timing provides natural engagement metrics — time in menu versus time in activity — without any custom instrumentation. The "Duration by Scene" dashboard widget surfaces this automatically.

**Recommendation:** For multi-scene apps, include scene upload verification in the validation checklist and mention the Duration by Scene widget as a free analysis surface.

---

## Remote controls for live tuning

**Applies to:** progression and mechanics, content and wellness (Phase 3). Unity and Unreal; not documented for visionOS, Android XR or WebXR

Games and content apps benefit from remote controls for live tuning without new builds. Common uses include difficulty scaling, spawn rates, feature flags, balance parameters, and content surfacing logic.

Remote controls are a Phase 3 recommendation for most projects, but they are worth flagging early so the team designs events and properties that can be meaningfully compared across control states.

**Remote controls have no page in the visionOS, Android XR or WebXR docs.** Verify current support before promising them to a team on any of those. In the meantime the experiment still works: a browser or native app almost always has its own config endpoint, remote config service or feature-flag system, so let that assign the condition and record the assigned value as a session property. What matters analytically is that the condition is recoverable per session, not which system handed it out.

**Recommendation:** If the team mentions A/B testing, live tuning, or balance iteration, flag remote controls early even if implementation is deferred to Phase 3. On visionOS, Android XR and WebXR, flag the recording requirement instead.
