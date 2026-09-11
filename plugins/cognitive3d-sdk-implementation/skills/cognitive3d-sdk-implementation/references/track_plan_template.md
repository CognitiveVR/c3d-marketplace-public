# Cognitive3D Track Plan Template

Use this template **after discovery** to turn recommendations into a clear, reviewable plan.

The goal is to produce something a developer, product owner, researcher, or data stakeholder can actually review and act on.

## Choosing the right format

### Quick plan (default)

Use the quick plan format for first engagements, focused questions, and when the developer needs a starting point rather than a comprehensive roadmap. Most conversations should produce a quick plan.

A quick plan has five sections:

1. **Project readback** — what the experience is, who it's for, what the team wants to learn, the target SDK and framework, plus the selected business motion, archetype(s), and overlays.
2. **Top questions to answer now** — the 3–5 most important questions from discovery.
3. **Phase 1 priorities** — foundation instrumentation only, with a brief note on why this phase matters and what it unlocks.
4. **Event catalog** — a compact table of Phase 1 events (typically 4–8 events) with properties and first analysis use. For brownfield projects, include the Status column (see section 6) — the audit's keep/amend/revive/replace decisions belong in the quick plan too, not only in a full plan.
5. **Validation checklist** — how to confirm the data is flowing correctly.

A quick plan deliberately omits Phase 2/3 recommendations, separate dynamic object tables, participant property tables, objectives sections, and exit poll question design. Those come later when the developer says "Phase 1 is validated, what's next?"

### Full plan

Use the full plan format when the developer explicitly asks for a complete integration roadmap, when the project is complex enough to need all three phases up front, or when the conversation has progressed past Phase 1.

A full plan uses all 14 sections below.

---

## Instructions

- Fill in only the sections that matter for the project.
- Choose one primary business motion and one or two archetypes.
- Keep the plan phased.
- Prefer the smallest useful set of recommendations.
- Explain why each recommendation matters.
- Screen every row against `sdk_capability_matrix.md` before presenting. A plan containing something the target SDK cannot do is worse than a smaller plan.
- Route implementation detail to the SDK reference for the project's target — `unity_sdk_reference.md`, `unreal_sdk_reference.md`, `androidxr_sdk_reference.md` or `webxr_sdk_reference.md` — and to current docs.

---

## 1. Project readback

**Experience summary**
- What the experience is
- Who uses it
- What the team wants to learn or decide

**Target SDK**
- Unity, Unreal, Android XR, or WebXR
- For Unreal, the authoring surface: Blueprint, C++, or both
- For Android XR, the platform: Jetpack XR or Meta Spatial SDK
- For WebXR, the framework: Three.js, Mattercraft, Wonderland, PlayCanvas, Babylon, or plain WebXR
- Note any capability limits this imposes on the plan, so the reader understands why something obvious is absent

**Primary business motion**
- performance and compliance
- progression and retention
- content engagement and habit formation
- exploration and evaluation
- structured research and experimentation

**Selected archetypes**
- archetype 1
- archetype 2, if truly needed

**Overlays**
- shared device or SSO
- multiplayer
- experimentation
- AI guide
- mixed reality
- privacy-sensitive capture
- content catalog
- multi-SDK delivery
- other relevant overlays

## 2. Top questions to answer now

List the 3 to 5 most important questions.

Example format:

1. Where do users fail in onboarding?
2. Which content gets completed versus abandoned?
3. Which prototype do users prefer after handling it?
4. Does movement style affect success?
5. Which cohort needs extra support?

## 3. Phase 1: foundation

State the smallest instrumentation set that should exist first.

### Phase 1 priorities
- lifecycle events
- FTUE stages
- key dynamic objects
- essential session context
- participant identity, if needed
- exit poll hooks
- validation sessions

### Why Phase 1 matters
Explain what questions this phase unlocks and what it deliberately does not try to answer yet. After Phase 1 is validated, the team should be able to open session replay and see the core user journey.

### What comes next
Briefly note what Phase 2 would add when the team is ready, so the developer knows this is a starting point, not the whole picture.

## 4. Phase 2: decision-grade instrumentation

_Include this section only in a full plan or when the developer has validated Phase 1 and asks for the next step._

Add the recommendations that directly answer the core business questions.

Suggested structure:
- key step or milestone events
- recurrent behavior events
- outcome metrics
- participant or cohort context
- objectives
- deeper content or prototype metadata

### Why Phase 2 matters
Explain which business questions become answerable after this phase.

## 5. Phase 3: optimization and experimentation

_Include this section only in a full plan or when the developer has validated Phase 2._

Suggested structure:
- UI instrumentation
- variants or conditions
- remote controls or live tuning
- AI or voice interaction detail
- extra survey logic
- advanced cohorting

### Why Phase 3 matters
Explain what tuning or experiment work this phase enables.

## 6. Event catalog

Use a table like this.

| Event name | When it fires | Properties | Why it matters | First analysis use |
| --- | --- | --- | --- | --- |
| `example_started` | when the activity begins | activity_id, activity_type | anchors the lifecycle | duration by activity |
| `example_ended` | when the activity ends | activity_id, duration_seconds, outcome | supports completion and outcome analysis | completion and drop-off |

Guidelines:
- Use stable event names
- Put variation into properties
- Include `_seconds` on time fields
- Keep only events that answer a real question
- For a quick plan, include only Phase 1 events (typically 4–8)
- For brownfield projects, add the **Status** column described below

### Event status column (brownfield projects)

When the project has existing instrumentation, add a **Status** column to the event catalog so the reviewer can see at a glance how much is genuinely new versus repaired. Greenfield plans omit the column (every row would be `New`). Use ONLY this closed vocabulary — do not invent per-project values, or plans stop being comparable across projects:

| Value | Meaning |
| --- | --- |
| `New` | Did not exist before this plan |
| `Keep` | Exists, fires correctly, no change |
| `Amend` | Keeps its name, gains or changes properties |
| `Revive` | Code exists but fires nowhere — not attached to any live scene or execution path (Unity: no GameObject or prefab; Unreal: an unreachable Blueprint graph or a level with no `BP_Cognitive3DActor`; Android XR: a cancelled scope or an activity never reached; WebXR: never imported, or the call site is never reached) |
| `Replaces: a, b, c` | Subsumes the listed existing events |
| `Retire` | Should stop being sent |

`Revive` matters because instrumentation routinely diverges between code presence and execution: a Unity event script that exists in the codebase but is attached to nothing in the scene, an Unreal Blueprint node sitting in a graph nothing calls, an Android XR sampling coroutine cancelled with its scope, or a WebXR module that is imported but never reached because the adapter update or the session hook was never wired. None of those is `New` and none is `Keep`.

### Migration map

When the plan has **three or more** `Replaces`/`Retire` rows, add a migration map subsection — old event → new event → break-risk decision — because multi-event `Replaces` cells become unreadable. Below that threshold, the Status cells and their notes carry it.

**A break-risk decision is required for every replaced or retired event, at any volume.** Renaming an event breaks every dashboard query, saved segment, and objective built on the old name, and permanently splits the historical series — old sessions keep the old name. State one of three options per event:

- **Cut over** — accept the split; rebuild queries and objectives on the new name.
- **Dual-send for one release** — send both names during a transition build, then retire the old one.
- **Leave it alone** — keep the existing name; consistency is not worth the break.

## 7. Dynamic object plan

Dynamic objects are available on Unity, Unreal and Android XR, and on WebXR only for the Three.js and Mattercraft adapters. If the project cannot support them, replace this section with a one-line statement of that fact plus the custom-event substitute, and say what it cannot answer.

List the objects that should become dynamic objects and why.

| Object | Why it should be dynamic | Related question | Related objective or replay use |
| --- | --- | --- | --- |
| instruction_panel | need to know whether users look at it before acting | are users missing instructions? | gaze step in objective |
| prototype_component_a | need object-level gaze and interaction context | which component drew attention? | replay and object analysis |

Do not list every object in the scene. Only include decision-relevant objects.

## 8. Session properties

| Property | Type | Example values | Why it belongs on the session |
| --- | --- | --- | --- |
| `app_mode` | enum | practice, test | differs by session |
| `movement_style` | enum | teleport, free_move | may affect interpretation |
| `variant` | enum | control, variant_b | needed for comparison |

## 9. Participant properties

| Property | Type | Example values | Why it belongs on the participant |
| --- | --- | --- | --- |
| `role` | enum | manager, associate | stable across sessions |
| `cohort` | enum | pilot_group_a | used for cohort comparison |
| `experience_level` | enum | beginner, advanced | helps explain outcomes |

If identity is not needed, say so explicitly.

## 10. Objectives and dashboard reads

List the first objectives or dashboard reads the team should create.

| Objective or readout | What it measures | Depends on | Why it matters |
| --- | --- | --- | --- |
| FTUE completion | whether users complete onboarding | ftue events | reveals early drop-off |
| module completion objective | whether required steps were completed | step events, objectives | supports assessment and reporting |

The goal is to name the first useful analysis surfaces, not just the raw data.

## 11. Exit poll plan

| Hook location | Purpose | Question type | Notes |
| --- | --- | --- | --- |
| end_of_module | capture confidence and feedback | scale + optional free response | can be configured later |
| end_of_content | capture satisfaction | recommendation or sentiment | useful for content ranking |

## 12. Validation checklist

Every plan should end with a test plan.

Suggested checklist:
- scenes uploaded and visible
- session replay visible
- key events appear with correct properties
- dynamic objects show up and register gaze
- session properties appear as expected
- participant properties appear as expected
- controller and boundary tracking active
- scene geometry exports with correct materials, or the check is marked not applicable because the toolchain cannot export them
- numeric properties arrive as numbers, not strings (check the authoring surface on Unreal)
- any built-in components the plan depends on are present (Unreal)
- no event exceeds the property cap where one applies (Android XR allows ten)
- dev and production traffic are separated
- offline or delayed uploads work, if relevant and supported on this SDK
- the team can name the first objective or query they will build

Add the SDK-specific checks from the relevant reference file: the Unreal, Android XR and WebXR references each carry a troubleshooting quick table, and the Unity reference routes to the Unity troubleshooting and project validation doc pages.

## 13. Implementation routes

Include only the column for the project's target SDK.

| Topic | Unity | Unreal | Android XR | WebXR |
| --- | --- | --- | --- | --- |
| Setup | https://docs.cognitive3d.com/unity/minimal-setup-guide/ | https://docs.cognitive3d.com/unreal/get-started/ | https://docs.cognitive3d.com/android-xr/installation-integration/ | https://docs.cognitive3d.com/webxr/get-started/ |
| Custom events | https://docs.cognitive3d.com/unity/customevents/ | https://docs.cognitive3d.com/unreal/customevents/ | https://docs.cognitive3d.com/android-xr/custom-events/ | https://docs.cognitive3d.com/webxr/events/ |
| Dynamic objects | https://docs.cognitive3d.com/unity/dynamic-objects/ | https://docs.cognitive3d.com/unreal/dynamic-objects/ | https://docs.cognitive3d.com/android-xr/dynamic-objects/ | https://docs.cognitive3d.com/webxr/dynamic-objects/ |
| Session and participant properties | https://docs.cognitive3d.com/unity/comprehensive-setup-guide/, https://docs.cognitive3d.com/unity/participants/ | https://docs.cognitive3d.com/unreal/sessions/, https://docs.cognitive3d.com/unreal/participants/ | https://docs.cognitive3d.com/android-xr/custom-session-properties/ | https://docs.cognitive3d.com/webxr/properties/ |
| Sensors | https://docs.cognitive3d.com/unity/sensors/ | https://docs.cognitive3d.com/unreal/sensors/ | https://docs.cognitive3d.com/android-xr/custom-sensors/ | https://docs.cognitive3d.com/webxr/sensors/ |
| Scenes and uploads | https://docs.cognitive3d.com/unity/scenes/ | https://docs.cognitive3d.com/unreal/scenes/ | https://docs.cognitive3d.com/android-xr/scene-object-uploads/ | https://docs.cognitive3d.com/webxr/scenes/ |
| Exit polls | https://docs.cognitive3d.com/unity/exitpoll/ | https://docs.cognitive3d.com/unreal/exitpoll/ | not documented; verify live | https://docs.cognitive3d.com/webxr/exitpoll/ |
| Remote controls | https://docs.cognitive3d.com/unity/remote-controls/ | https://docs.cognitive3d.com/unreal/remote-controls/ | not documented; verify live | not documented; verify live |
| Built-in components | https://docs.cognitive3d.com/unity/components/ | https://docs.cognitive3d.com/unreal/built-in-components/ | n/a | n/a |
| Platform support | n/a | n/a | https://docs.cognitive3d.com/android-xr/get-started/ | https://docs.cognitive3d.com/webxr/framework-support/ |

The Upload Web App (https://docs.cognitive3d.com/dashboard/upload-webapp/) serves Android XR and WebXR, and any other target without in-engine tooling.

SDK-independent:

- Objectives: Dashboard Creating Objectives → https://docs.cognitive3d.com/dashboard/creating-objectives/ (or via MCP server — see the SDK reference for routes and constraints)
- LMS/xAPI: Dashboard LMS → https://docs.cognitive3d.com/dashboard/lms/
- Data export and API: https://docs.cognitive3d.com/api/get-started/

## 14. Open questions and assumptions

End with any assumptions or unresolved items.

Examples:
- Whether identity is anonymous or employee-based
- Whether a score metric already exists in code
- Whether raw transcript capture is approved
- Whether the team wants one project or separate dev and prod projects
- Whether condition assignment should live on the session or participant
- Whether the team is ready for the full roadmap or just needs Phase 1 now
- Whether the experience will also ship on another SDK, and whether the builds should report into one project
- Which authoring surface carries the instrumentation, on Unreal, and whether numeric properties need a C++ path
- Whether ExitPoll is available on the target SDK, and what replaces it if not
