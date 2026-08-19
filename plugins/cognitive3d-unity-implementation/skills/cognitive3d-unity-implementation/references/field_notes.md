# Cognitive3D Field Notes

_Lessons from real integrations. Use these alongside the playbooks to avoid common pitfalls and catch high-leverage opportunities._

This file is organized by topic, not by archetype. Some notes apply universally; others are tagged with the archetypes or overlays where they matter most.

## How to use this file

After choosing your archetypes and pulling the relevant playbooks, scan the topics below for anything that applies. The playbooks tell you what to recommend. This file tells you what to watch out for and where the easy wins are.

AI assistants should read this file after `playbooks.md` and before filling out `track_plan_template.md`.

---

## Controller and boundary tracking verification

**Applies to:** all projects

This is the single most common issue found during integration reviews.

Controllers should be tracked as dynamic objects. The SDK's Project Setup flow detects and tracks the HMD, tracking space, and controllers automatically by default, but SteamVR rigs require assigning the controller GameObjects manually in the Project Setup window, and older SDK versions or unusual configurations may also need manual setup. If controllers are not tracked, many built-in dashboard metrics — hand/controller height, ergonomics scoring, input tracking — will be missing or incorrect.

Boundary tracking is equally important and equally easy to miss. You can verify it on any session detail page: look at the top of the page for the purple icons showing all active data streams. If boundary is missing, physical-versus-virtual movement metrics will not work.

**Recommendation:** Always include a controller and boundary verification step in the validation checklist, regardless of project type.

---

## Exit poll hooks are nearly free — place them early

**Applies to:** all projects

Exit poll hooks should be placed at the beginning and end of the experience even if the team does not have survey questions ready yet. Questions are configured on the platform — on the dashboard or programmatically via the MCP server — and can be added, changed, or removed without shipping a new build. (One caveat when changing them: editing a question set creates a new version, and existing hooks stay pointed at the old version until explicitly reassigned — see the ExitPoll platform constraints in `unity_sdk_reference.md`.) The hooks must exist in the app code.

Stakeholders inevitably ask "can we survey users about X?" weeks or months after launch. If hooks are already in place, a survey can be live in minutes. Without them, it requires a code change, a new build, and a store submission.

**Recommendation:** Treat exit poll hooks as part of the Phase 1 foundation, not as a Phase 2 or Phase 3 addition.

---

## First-time user experience and the two-minute window

**Applies to:** progression and mechanics, content and wellness, exploration and evaluation

Users often decide within the first two minutes whether to abandon an app entirely. Getting to the spatial payoff — the moment the experience feels worth continuing — within that window is critical for long-term retention.

This makes FTUE tracking one of the highest-value instrumentation investments across almost all project types. Duration per stage, not just completion, is what reveals whether onboarding is fast enough.

**Recommendation:** Always track FTUE stages with durations. If the team is debating onboarding approaches, flag that this is a strong candidate for A/B testing with remote controls.

---

## Editor sessions and dev/prod separation

**Applies to:** all projects

Sessions run in the Unity Editor are automatically excluded from major analytics on the dashboard. They can be toggled back on with a button in the bottom left corner.

However, for builds deployed to devices during development, a `development_mode` session property or a "Development" session tag is still necessary. Without it, test sessions from sideloaded builds will pollute production dashboards.

Some teams use a separate dashboard project for heavy testing. A session property is the simplest approach if the team wants everything in one project.

**Recommendation:** Always include dev/prod separation in Phase 1. Mention the editor auto-exclusion so developers do not duplicate effort.

---

## Dynamic objects: quality over quantity

**Applies to:** all projects

Do not recommend dynamic objects on every interactable in the scene. Only add them where object-level attention or interaction data answers a real question.

Objects that spawn repeatedly — individual bullets, individual hit targets, particle effects — should generally not be tracked as dynamic objects. This pollutes the object list on the dashboard and makes it difficult to find meaningful objects.

Good candidates are objects where gaze, fixation, or interaction context changes how you interpret the session: tools, equipment, instruction panels, prototypes, focal environmental features.

If the project needs to track spawned objects with meaningful identity, use ID Pools.

**Recommendation:** When listing dynamic objects in the track plan, always state why each object is included and what question it supports.

---

## Platform identity: Oculus Social and Steam

**Applies to:** progression and mechanics, content and wellness (consumer apps)

For Meta Quest apps, the OculusSocial component captures platform identity (Oculus username and ID). For Steam apps, saving the Steam username as participant name and Steam ID as participant ID achieves the same thing.

Platform identity enables review attribution — understanding why specific users left positive or negative reviews — and correlation with store-level metrics.

**Recommendation:** Flag this for any consumer app. It pairs well with participant tracking and is easy to miss.

---

## Content duration: planned versus actual

**Applies to:** content and wellness, performance and assessment

For content-based experiences (classes, meditations, workouts, training modules), recording both the planned content duration and the actual participation duration is important. A user who completes 4 minutes of a 30-minute class is a very different signal from a user who completes 28 minutes. Without both values, you cannot distinguish the two.

**Recommendation:** Include `content_duration_seconds` (planned length) and `duration_seconds` (actual time) on content lifecycle events.

---

## Custom shaders and scene upload

**Applies to:** all projects (Unity)

When a project uses custom shaders, materials will appear white on the Cognitive3D dashboard because the GLTF exporter does not know how to map custom shader properties to standard PBR. This must be handled before the scene is uploaded.

Check for custom shaders early. If they are present, a shader-properties export script needs to be created: an Editor class inheriting `GLTFSceneExporter.ShaderPropertyCollection` (namespace `Cognitive3D.UnityGLTF`), placed under `Editor/GLTF`, mapping the shader's property names to PBR. The SDK ships examples to copy from — `URPShaderProperties`, `HDRPShaderProperties`, `StandardShaderProperties` — and discovers subclasses automatically via reflection; no registration step is needed. See the Custom Shaders section of https://docs.cognitive3d.com/unity/troubleshooting/.

**Recommendation:** Include a custom shader check in the validation checklist for Unity projects.

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

**Applies to:** performance and assessment (training only)

Audio recording captures in-session audio aligned to the session timeline. It is valuable for training scenarios where verbal communication matters — trainees explaining procedures, giving verbal responses, or communicating with virtual patients.

Audio recording requires explicit runtime permission and may require additional privacy disclosures. Do not recommend this casually.

**Recommendation:** Only recommend audio capture for training use cases where verbal output is directly relevant to the assessment. Always flag the consent and governance requirements.

---

## Multi-scene applications and scene timing

**Applies to:** all projects with multiple scenes

If the experience transitions between scenes (common in fitness apps, training apps with multiple modules, games with level loading), each scene should be uploaded to the dashboard.

Scene transition timing provides natural engagement metrics — time in menu versus time in activity — without any custom instrumentation. The "Duration by Scene" dashboard widget surfaces this automatically.

**Recommendation:** For multi-scene apps, include scene upload verification in the validation checklist and mention the Duration by Scene widget as a free analysis surface.

---

## Remote controls for live tuning

**Applies to:** progression and mechanics, content and wellness (Phase 3)

Games and content apps benefit from remote controls for live tuning without new builds. Common uses include difficulty scaling, spawn rates, feature flags, balance parameters, and content surfacing logic.

Remote controls are a Phase 3 recommendation for most projects, but they are worth flagging early so the team designs events and properties that can be meaningfully compared across control states.

**Recommendation:** If the team mentions A/B testing, live tuning, or balance iteration, flag remote controls early even if implementation is deferred to Phase 3.
