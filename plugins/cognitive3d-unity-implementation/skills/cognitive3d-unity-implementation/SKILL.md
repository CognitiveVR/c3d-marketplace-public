---
name: cognitive3d-unity-implementation
description: "Cognitive3D Unity SDK implementation strategy for client projects. Use this skill whenever someone asks about integrating Cognitive3D analytics, planning what to track, setting up the SDK in Unity, creating a tracking plan, or implementing custom events/dynamic objects/exit polls/session properties. Also use when the user mentions Cognitive3D, C3D, XR analytics, spatial analytics, or wants to instrument a VR/AR/MR application for behavioral data collection. This covers the full workflow: discovery, data strategy, phased implementation, and technical routing."
---

# Cognitive3D Unity SDK Implementation Strategy

This skill guides the full workflow for implementing Cognitive3D analytics in Unity client projects — from discovery through validated instrumentation.

## How this skill is organized

- **This file**: Core workflow, discovery, classification, key rules, output format, and routing
- **references/data_strategy.md**: Stable strategy layer — primitives, phasing, overlays, naming, anti-patterns, business question map
- **references/playbooks.md**: Detailed archetype playbooks and overlay cards
- **references/field_notes.md**: Practitioner observations from real integrations, organized by topic
- **references/unity_sdk_reference.md**: Unity SDK technical knowledge and doc routing
- **references/track_plan_template.md**: Output template with quick plan and full plan formats
- **references/example_plans.md**: Example track plans for common project shapes
- **c3d-progress-tracker-template.md**: Worksheet template for tracking implementation progress per client

**Read this file first.** Load reference files progressively — only when needed for the current step. Do not load all reference files at once.

---

## Critical rules

### 1. Discovery before implementation

When a developer or client asks to integrate, set up, or improve Cognitive3D analytics, you must ask the **Core Discovery Questions** and wait for answers before recommending instrumentation, writing code, or editing files.

Do not infer business questions from project structure alone. You may inspect the project for technical context, but that is not a substitute for discovery.

### 2. Project conventions outrank generic advice

Before recommending instrumentation or writing code, check whether the project defines its own Cognitive3D conventions — an implementation guide, data design document, telemetry standard, project configuration, or a relevant section of `CLAUDE.md` or `CONTRIBUTING.md`. Look in the repo root, in `docs/`, and at any markdown file whose name mentions telemetry, analytics, instrumentation, data design, or Cognitive3D.

If such a document exists, **it wins**. Match its event naming, property casing, required-property lists, environment layout and session-tag vocabulary, even where this skill's generic recommendations differ. A team with a house standard cares more about conformance than about optimality, and a parallel scheme is worse than an imperfect one applied consistently.

Where the document labels which items are Editor work and which are code, honour those labels rather than re-deriving them, and never offer to perform an Editor-only step (see rule 8).

If the project has no such document, offer to draft one. A short conventions file is the cheapest way to keep later instrumentation consistent, and it makes every future session on the project more useful.

### 3. Never read or expose credentials

Never read, log, store, or output API keys, developer keys, SSO secrets, or credentials from `Cognitive3D_Preferences`, environment variables, config files, build scripts, or any other file that may contain secrets. If the SDK needs keys, instruct the developer to enter them in the project setup flow themselves.

### 4. Strategy before code

The strategy docs answer **what to track and why**. The SDK reference answers **how to implement it**. Keep them separate. Do not jump to code until the tracking plan is clear.

### 5. Smallest useful instrumentation

Do not recommend every available feature by default. A good plan answers the developer's questions with the smallest decision-grade set of custom events, dynamic objects, session properties, participant properties, objectives, exit polls, and tags.

### 6. Properties over event-name proliferation

Do not create separate events for every variation. Use one event plus properties when that answers the question better. Use explicit units in property names — at minimum, durations should use `_seconds`.

### 7. Verify live docs when freshness matters

Do not hard-code SDK versions, release notes, supported hardware, dashboard UI paths, or API auth details into recommendations. When freshness matters, route to live docs.

### 8. Editor workflows vs code tasks

Not all implementation steps are code tasks. Many Cognitive3D features — especially dynamic objects, scene setup, exit poll hook placement, and component configuration — are primarily Unity Editor workflows (add component, configure in Inspector, drag references). Before writing a runtime script for any SDK feature, first check whether the project already has an Editor-based pattern for that feature. If it does, describe the Editor steps instead of generating code. Only write code when the feature genuinely requires runtime logic (e.g., custom events fired from game logic, session properties set from runtime state, ID Pools for spawned objects).

### 9. Default to a concise plan

Unless the developer explicitly asks for a full integration roadmap, produce a **quick plan** (Phase 1 only). The full template exists for comprehensive plans, but most first conversations should produce a focused, approachable starting point. See `references/track_plan_template.md` for the quick plan format.

---

## Core workflow

### Step 1: Discovery

Ask these questions first and wait for answers.

1. **What is the experience about?**
   Examples: training simulation, VR game, content library, meditation app, product evaluation, academic study, virtual showroom

2. **Who are the target end users or participants?**
   Examples: trainees, employees, players, consumers, researchers, students, patients

3. **What decisions or insights are you trying to support?**
   Examples: completion rates, retention, errors, time to proficiency, content engagement, attention, comparison between variants

4. **What are the key interactions, activities, or content units?**
   Examples: tool use, puzzle steps, missions, classes, meditation sessions, prototype handling, menu use

5. **What KPIs or success metrics matter most right now?**
   Examples: completion, duration, accuracy, return rate, score, rating, preference, compliance, comfort

#### Adaptive follow-up questions

Only ask these if needed for the project type:

**Training / assessment / education:**

- Practice flow, formal assessment, or both?
- Shared devices?
- LMS, xAPI, or internal reporting needed?
- Critical incorrect actions or compliance failures?

**Games / progression / interactive entertainment:**

- What counts as progression?
- Which mechanics matter to learn or balance?
- Tutorials, chapters, missions, or repeatable loops?
- Live tuning, variants, or A/B tests planned?

**Content / wellness / practice apps:**

- Primary content unit (class, course, session, track, chapter, meditation, drill)?
- Do pause, resume, bookmark, repeat, or recommendations matter?
- Need to compare content types, teachers, programs, difficulty?

**Evaluation / consumer research / studies:**

- Conditions, variants, or prototypes to compare?
- Does interaction order matter?
- Ratings, surveys, or cohort tags needed?
- Anonymous, identified, or shared-device usage?

**Multiplayer / collaboration / social flows:**

- What identifies a room, lobby, session, or role?
- Do join, leave, ready, invite, or match outcomes matter?
- Per-person or per-room analysis?

**AI guides / agents / conversational features:**

- Category-level interaction data, or approved transcript-level data?
- Privacy or consent constraints?
- What outcome should agent usage influence?

#### Technical context to infer from the project

Inspect the project to determine (confirm only if unclear):

- Target engine and platform
- Render pipeline
- Scene structure
- Whether SDK is installed
- Whether scenes are uploaded
- Whether controllers, hands, gaze, dynamics, events, exit poll hooks are in use
- Whether shared-device or SSO-enabled
- Whether a naming convention exists for custom events and properties
- **Execution architecture** — how scripts execute: standard MonoBehaviour callbacks, custom event system, visual scripting, timeline, state machine, coroutine sequencer, or a mix. This determines where analytics code needs to hook in.
- **Whether instrumentation already exists** — if so, audit it against the universal baseline in `references/data_strategy.md` before recommending additions
- **Whether the project defines its own C3D conventions** — an implementation guide, data design document, telemetry standard or project configuration. See rule 2: if one exists, it governs naming, required properties and conventions, and this skill's generic advice becomes the fallback

### Step 2: Classify the project

After discovery, classify before recommending. Do not jump straight from discovery to a feature list.

**Choose one primary business motion:**

#### A. Performance and compliance

Use when the team needs to prove completion, competency, accuracy, or adherence to a process.

Typical examples: training simulation, SOP walkthrough, certification flow, education module with pass/fail logic.

#### B. Progression and retention

Use when the team needs to improve onboarding, progression, mechanic use, or repeat play.

Typical examples: games, narrative experiences, puzzle flows, sandbox progression.

#### C. Content engagement and habit formation

Use when the team needs to understand what content is chosen, completed, repeated, paused, bookmarked, or abandoned.

Typical examples: fitness and wellness apps, meditation libraries, music practice or instrument trainers, reading or media experiences.

#### D. Exploration and evaluation

Use when the team needs to understand what people noticed, touched, compared, or preferred in an environment.

Typical examples: showrooms, architecture and design reviews, retail or product evaluation, prototype comparison.

#### E. Structured research and experimentation

Use when the team needs to compare conditions, cohorts, blocks, trials, or repeated measures.

Typical examples: academic studies, UX research studies, controlled experiments, pilot programs with explicit conditions.

**Choose one or two archetypes:**

Business motion tells you the decision. Archetype tells you the shape of the experience.

1. **Procedural assessment** — step order matters, pass/fail matters, errors matter, shared devices may matter
2. **Progression loop** — time to fun matters, progression blockers matter, mechanic adoption matters, recurring actions matter
3. **Session-based content loop** — start/end of content matters, content metadata matters, pause/repeat/bookmark/browsing matter
4. **Exploration or object-interaction loop** — order of interaction matters, object gaze matters, comparison matters, ratings or preferences matter
5. **Trial or condition loop** — condition assignment matters, trial order matters, cohort analysis matters, protocol adherence matters

**Add overlays** only where they truly apply:

- Shared devices / SSO
- Multiplayer / collaborative sessions
- Content catalog / subscription
- Live ops / A/B testing / remote controls
- Mixed reality / input-mode changes
- AI guides / agents / conversational
- Privacy-sensitive capture
- Multi-scene flows

#### Example classifications

| Example project | Primary motion | Archetype | Common overlays |
| --- | --- | --- | --- |
| Fire safety simulation | Performance and compliance | Procedural assessment | shared device, LMS |
| VR puzzle adventure | Progression and retention | Progression loop | live tuning |
| Meditation library | Content engagement | Session-based content loop | content catalog |
| Product comparison study | Exploration and evaluation | Exploration loop | survey, cohort tags |
| Academic experiment | Structured research | Trial or condition loop | condition assignment |

### Step 3: Create or update progress tracker

Before proceeding to recommendations, create a progress tracker file for this client using `c3d-progress-tracker-template.md` — or update the existing one if this is a returning project. Save it in the client's project directory, not in the skill folder. Populate it with discovery answers and classification from Steps 1–2. This is required. Do not proceed to Step 4 until the tracker exists and reflects current discovery and classification status.

### Step 4: Choose the right primitive

| Need | Best primitive |
|---|---|
| Something happened at a moment in time | Custom event |
| A value describes the whole session | Session property |
| A value describes the person across sessions | Participant property |
| Object seen, used, moved, or fixated | Dynamic object |
| Completion logic or sequences | Objective |
| Self-reported feedback or cohort questions | Exit poll |
| Analyst-added grouping after the fact | Session tag |

For detailed guidance on event vs property and objective vs event decisions, see `references/data_strategy.md`.

### Step 5: Build a phased recommendation

Read `references/data_strategy.md` for strategy guidance. Then read only the relevant sections of `references/playbooks.md` for the chosen archetype(s). Do not load all playbooks — pull only the ones that fit.

Check `references/field_notes.md` for any topics relevant to the chosen archetypes and overlays.

**Phase 1 — Foundation:** Make the project observable.

- Primary activity start/end, FTUE stages, key dynamic objects, essential session context, dev/prod separation, exit poll hooks, validation sessions

**Phase 2 — Decision-grade instrumentation:** Answer the real questions.

- Step/milestone events, recurrent behavior, error events, participant identity, success metrics, objectives, content/prototype metadata

**Phase 3 — Optimization and experimentation:** Tune and compare.

- UI detail, variant/condition tracking, remote controls, catalog attributes, agent metrics, deeper surveys, social/multiplayer logic

### Step 6: Structure the output

Use `references/track_plan_template.md` to produce the actual recommendation.

**Default to the quick plan format** unless the developer explicitly asks for a comprehensive roadmap. A quick plan includes only:

- Project readback
- Top questions to answer now
- Phase 1 priorities (with a brief note on what Phase 2 would add later)
- Event catalog (Phase 1 events only, typically 4–8)
- Validation checklist

**Do not include code snippets in the quick plan.** The plan should describe *what* to track and *why*, not *how* to implement it. Code examples belong in Step 6 (technical routing) when the developer asks you to implement a specific item.

**Use the full plan format** when:

- The developer asks for a complete integration roadmap
- The project is complex enough to need all three phases up front
- The conversation has progressed past Phase 1 validation

See `references/example_plans.md` for pattern references.

### Step 7: Route technical implementation

When the developer asks **how** to implement something, use `references/unity_sdk_reference.md` to route to the correct documentation page.

When routing implementation, identify whether the step is:

- **Editor workflow** — add/configure components in Inspector, upload scenes, upload dynamic object meshes; also dashboard configuration when the team configures objectives or ExitPoll question sets there. Describe the steps; do not generate code, and do not offer to perform the step yourself.
- **Code task** — custom events, session/participant properties, sensor recording, runtime lifecycle logic. Write or modify scripts, following the project's own conventions where they exist (rule 2).
- **Platform task** — objectives and ExitPoll question sets live on the platform, not in the build. Both can be configured **either** on the dashboard **or** programmatically through the MCP server. Ask which route the team wants before doing either; never assume a write-enabled key exists, or that the team wants one.
- **Hybrid** — e.g., exit polls need both a code trigger in the app and a question set configured on the platform. The hook name in the Unity call must match the platform-side hook exactly.

#### Choosing a route for objectives and ExitPoll question sets

Objectives and ExitPoll question sets are each configurable both ways, and both routes are fully supported. This is a team preference, not a best practice with one right answer.

**Objectives:**

- **Dashboard** — https://docs.cognitive3d.com/dashboard/creating-objectives/. No API key needed. The right default when objectives are owned by an analyst, researcher or instructional designer rather than a developer; when they change rarely; or when the team would rather not have a write-enabled key in circulation. Describe the steps; do not generate code.
- **MCP server** — `create_objective`, `update_objective`, `delete_objective`. Worth it when there are many objectives, when staging and production must match exactly, or when the team wants definitions version-controlled and reviewed like code.

**ExitPoll question sets and hooks:**

- **Dashboard** — question set and hook creation is documented alongside the Unity integration at https://docs.cognitive3d.com/unity/exitpoll/. No API key needed. The right default when a researcher or designer owns question wording, which is common — survey copy is usually not a developer's call.
- **MCP server** — `create_exitpoll_question_set`, `archive_exitpoll_question_set`, `create_exitpoll_hook`, `update_exitpoll_hook`, plus `get_exitpoll_configuration` and `get_exitpoll_question_set` for reads. Worth it when question sets must be identical across projects, or when survey definitions should live in version control.

Both MCP routes need an organization API key with write access and an org- or project-admin role; read-only keys can view but not change either resource. Organization API keys are scoped to the entire organization — C3D does not currently offer per-project granularity — so a write-enabled key grants write access across every project in the org. That is a legitimate reason for a team to decline and stay on the dashboard. Do not push back on it.

**Always dry-run first.** Every MCP write for both resources previews what will happen and changes nothing until confirmed. Do not skip the preview because a change looks small.

Effects to flag to the developer before executing a write:

- **Objectives** — `sequential` cannot be changed after save; writing steps asynchronously re-scores roughly the last 30 days of sessions and nothing older; `name` is capped at 32 characters and truncates silently. Gaze and fixation steps reference dynamic object IDs, which are per-project, so those cannot be copied between projects verbatim.
- **ExitPoll** — question set versions are immutable, so editing a question creates a new version rather than changing the existing one. Removal is archival only; there is no hard delete. **A new version does not move existing hooks.** Hooks stay pointed at whatever version they were assigned until explicitly reassigned with `update_exitpoll_hook`, so publishing v2 and expecting the app to pick it up is a silent no-op. Hooks themselves cannot be deleted, only unassigned — and a hook with no question set assigned is skipped silently at runtime, which looks identical to a working hook from inside the app.

These constraints are properties of the platform, not of the MCP — they apply to dashboard-created objectives and question sets too, and are worth stating either way.

Before writing a runtime script, verify which execution path it needs to be on. If the project uses a custom event system, state machine, or visual scripting, analytics code must integrate with that system — not bypass it with a standalone MonoBehaviour.

Before writing code for dynamic objects: check whether the project already manages them via Editor-attached components. Most projects do. The default recommendation should be to add the DynamicObject component in the Editor and follow the existing project pattern.

Key Unity doc entry points:

- Installation: Unity minimal setup guide
- Custom Events: Unity Custom Events page
- Dynamic Objects: Unity Dynamic Objects page
- Session/participant metadata: Unity comprehensive setup guide
- Exit Polls: Unity ExitPoll page
- Scenes: Unity Scenes page
- Remote Controls: Unity Remote Controls page

### Step 8: Validate

Every plan should end with a validation plan confirming:

- Scenes uploaded and visible
- Session replay working
- Key events fire with correct properties
- Dynamic object meshes uploaded via Feature Builder (separate from scene upload)
- Dynamic objects visible in replay with gaze data
- Session and participant properties populated
- Controller and boundary tracking active
- Custom shaders export correctly (if applicable)
- Offline/delayed upload behavior works (if needed)

---

## Brownfield projects: when instrumentation already exists

If the SDK is already partially integrated, do not start from scratch. Instead:

1. **Audit the existing instrumentation** against the universal baseline in `references/data_strategy.md`. Which of the 12 baseline items are already in place? Which are missing?
2. **Map the execution architecture (Unity projects).** Before recommending where to add analytics, understand how existing scripts get executed. Unity projects often have multiple execution paths (MonoBehaviour callbacks, custom event systems, visual scripting, timelines, coroutine-based sequencers). Analytics must be placed on paths that are actually live in the scene.
   - Search the scene file by script GUID to confirm which scripts are attached to GameObjects. Code existence does not imply scene presence — Unity projects have a dual nature: code (readable from files) and scene state (configured in the Editor, only partially readable from YAML). Confirm with the developer which scripts are active in the scene.
   - If a custom sequencing system exists (event/action framework, state machine, visual scripting graph, etc.), map its structure before assuming analytics scripts within it are firing.
   - Do not proceed to implementation until you can answer: "how does each analytics-relevant script get called at runtime?"
3. **Run discovery anyway.** Existing events tell you what the team tried to track. Discovery tells you what they actually need to know. These are often different.
4. **Identify gaps, not a full rewrite.** The recommendation should focus on what to add, adjust, or remove — not a from-scratch plan that ignores existing work.
5. **Check naming consistency.** If existing events follow a convention, match it. If they don't, recommend a migration path rather than a parallel naming scheme.

**Important: stage the output in two steps.** First, present only the audit — what's working, what's missing, and the project classification. Then **stop and ask** the developer if they'd like you to propose improvements before presenting a plan. Do not combine the audit and the plan into a single response. This keeps the audit digestible and gives the developer a chance to correct misunderstandings before you build on them.

**Progress tracker required here too.** After completing the audit and before presenting improvement recommendations, create or update the progress tracker using `c3d-progress-tracker-template.md`. Save it in the client's project directory. Pre-populate it with the audit findings so the existing instrumentation status is captured.

---

## Plan evolution: revisiting and extending

A tracking plan is not a one-time deliverable. Teams should revisit it when:

- Phase 1 is validated and the team is ready for deeper instrumentation
- Business questions change (new stakeholders, new KPIs, pivot in product direction)
- New features are added to the app (multiplayer, AI guide, new content types)
- The team wants to run experiments or compare variants

When revisiting, re-run the discovery questions with updated context. The business motion and archetype may stay the same, but the top questions and phase priorities will evolve.

---

## Universal baseline (always recommend)

These apply to almost every Cognitive3D project. See `references/data_strategy.md` for full details on each:

1. Primary activity lifecycle (start/end pair)
2. Onboarding / FTUE stages with durations
3. Key dynamic objects (only where they answer a question)
4. Session context properties
5. Participant identity (when cross-session analysis matters)
6. Dev vs production separation
7. Input and environment verification
8. Exit poll hooks (even if questions aren't ready)
9. Controller and boundary tracking verification
10. Custom shader check (Unity)
11. Validation sessions
12. At least one analysis surface (objective, query, replay, or comparison)

---

## Quality bar

A strong recommendation should be:

- **Tailored** to the developer's actual goals, not generic
- **Small enough** to implement without feeling overwhelming
- **Rich enough** to answer real questions after validation
- **Consistent** in naming and property conventions
- **Explicit** about assumptions and open questions
- **Honest** about privacy-sensitive areas
- **Concrete** about what the team will see in the dashboard after implementation — name the first replay path, objective, or query

---

## Anti-patterns to avoid

1. Recommending before discovery
2. Encoding properties into event names
3. Using session properties for person-level data
4. Using participant properties for one-session values
5. Tracking everything as a dynamic object
6. Missing duration fields
7. Omitting units from property names
8. Only tracking one-time milestones when recurring behavior matters
9. No dev vs prod separation
10. No validation plan
11. Casual privacy-sensitive collection
12. Recommending features without naming the first analysis use

---

## Progress tracking

See **Step 3** in the core workflow. The progress tracker is a required gate — create or update it before moving to recommendations. For brownfield projects, create it after the audit. Update it as work progresses so the next session can pick up where you left off.

---

## Naming conventions

- Use stable event names: `module_started`, `step_completed`, `content_bookmarked`
- Put variation in properties, not event names
- Use `_seconds` for time fields, consider `_percent`, `_count`, `_meters`
- Use both stable IDs and readable labels when names can change
- Record recurrence and order when sequence matters (`attempt_count`, `interaction_order`)
- Match the app's domain language (games use `mission`, training uses `module`, research uses `trial`)

---

## Reference file loading guide

Load files in this order as needed. Do not load them all at once.

1. **This file (SKILL.md)** — always read first
2. **references/data_strategy.md** — when you need strategy depth (primitives, phasing, overlays, anti-patterns, business question map)
3. **references/playbooks.md** — when you need archetype-specific recommendations (pull only the relevant playbook, not all of them)
4. **references/field_notes.md** — when you need practitioner observations for the chosen archetypes and overlays
5. **references/example_plans.md** — when a concrete pattern example would help
6. **references/track_plan_template.md** — when you're ready to structure the output
7. **references/unity_sdk_reference.md** — when the developer asks how to implement a recommendation
