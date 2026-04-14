# Cognitive3D Track Plan Template

Use this template **after discovery** to turn recommendations into a clear, reviewable plan.

The goal is to produce something a developer, product owner, researcher, or data stakeholder can actually review and act on.

## Choosing the right format

### Quick plan (default)

Use the quick plan format for first engagements, focused questions, and when the developer needs a starting point rather than a comprehensive roadmap. Most conversations should produce a quick plan.

A quick plan has five sections:

1. **Project readback** — what the experience is, who it's for, what the team wants to learn, plus the selected business motion, archetype(s), and overlays.
2. **Top questions to answer now** — the 3–5 most important questions from discovery.
3. **Phase 1 priorities** — foundation instrumentation only, with a brief note on why this phase matters and what it unlocks.
4. **Event catalog** — a compact table of Phase 1 events (typically 4–8 events) with properties and first analysis use.
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
- Route implementation detail to `unity_sdk_reference.md` and current docs.

---

## 1. Project readback

**Experience summary**
- What the experience is
- Who uses it
- What the team wants to learn or decide

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

## 7. Dynamic object plan

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
- custom shaders export correctly, if applicable
- offline or delayed uploads work, if relevant
- the team can name the first objective or query they will build

## 13. Implementation routes

- Custom events: Unity Custom Events → https://docs.cognitive3d.com/unity/customevents/
- Dynamic objects: Unity Dynamic Objects → https://docs.cognitive3d.com/unity/dynamic-objects/
- Participant properties: Unity Participants → https://docs.cognitive3d.com/unity/participants/
- Session properties: Unity Comprehensive Setup → https://docs.cognitive3d.com/unity/comprehensive-setup-guide/
- Exit polls: Unity ExitPoll → https://docs.cognitive3d.com/unity/exitpoll/
- Objectives: Dashboard Creating Objectives → https://docs.cognitive3d.com/dashboard/creating-objectives/
- LMS/xAPI: Dashboard LMS → https://docs.cognitive3d.com/dashboard/lms/
- Remote controls: Unity Remote Controls → https://docs.cognitive3d.com/unity/remote-controls/

## 14. Open questions and assumptions

End with any assumptions or unresolved items.

Examples:
- Whether identity is anonymous or employee-based
- Whether a score metric already exists in code
- Whether raw transcript capture is approved
- Whether the team wants one project or separate dev and prod projects
- Whether condition assignment should live on the session or participant
- Whether the team is ready for the full roadmap or just needs Phase 1 now
