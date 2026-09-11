---
name: cognitive3d-sdk-implementation
description: "Cognitive3D SDK implementation strategy across engines and platforms — Unity, Unreal Engine, native Apple Vision Pro (visionOS), native Android XR (Jetpack XR and Meta Spatial SDK), and WebXR (Three.js, Mattercraft, Wonderland, PlayCanvas, Babylon, plain WebXR). Use this skill whenever someone asks about integrating Cognitive3D analytics, planning what to track, setting up the SDK, creating a tracking plan, or implementing custom events/dynamic objects/exit polls/session properties. Also use when the user mentions Cognitive3D, C3D, @cognitive3d/analytics, cvr-sdk-unreal, BP_Cognitive3DActor, Cognitive3DManager, Cognitive3DAnalyticsCore, com.cognitive3d, Apple Vision Pro, XR analytics, spatial analytics, or wants to instrument a VR/AR/MR/WebXR application for behavioral data collection. This covers the full workflow: SDK identification, discovery, data strategy, phased implementation, and technical routing."
---

# Cognitive3D SDK Implementation Strategy

This skill guides the full workflow for implementing Cognitive3D analytics — from discovery through validated instrumentation — for any supported SDK.

**Supported targets:**

| Target | Reference file | Covers |
| --- | --- | --- |
| Unity | `references/unity_sdk_reference.md` | Unity VR/AR/MR projects, including Unity apps shipped to Vision Pro or Android headsets |
| Unreal Engine | `references/unreal_sdk_reference.md` | Unreal 4.26.2+ VR/AR/MR projects, Blueprint and C++ |
| visionOS | `references/visionos_sdk_reference.md` | Native Apple Vision Pro apps in Swift, RealityKit and SwiftUI |
| Android XR | `references/androidxr_sdk_reference.md` | Native Kotlin/Java apps on Android XR (Jetpack XR) and Meta Spatial SDK |
| WebXR | `references/webxr_sdk_reference.md` | Three.js, Mattercraft, Wonderland Engine, PlayCanvas, Babylon.js, plain WebXR/WebGL |

**Two targets name a device, not a toolchain, and both resolve to an engine more often than people expect:**

- **Apple Vision Pro** apps are built either natively in Swift (the visionOS reference) or in Unity (the Unity reference). "We're building for Vision Pro" does not settle it.
- **Android headsets** run Unity and Unreal apps as well as native ones. The Android XR reference is only for apps with no game engine; a Unity app on a Quest is a Unity project, possibly using the External Android Plugin.

In both cases the engine wins when engine project files are present. Step 0 carries the tiebreak.

The strategy layer (what to track and why) is shared across every target. Only the technical reference changes.

## How this skill is organized

- **This file**: Core workflow, SDK identification, discovery, classification, key rules, output format, and routing
- **references/data_strategy.md**: Stable strategy layer — primitives, phasing, overlays, naming, anti-patterns, business question map. SDK-neutral
- **references/playbooks.md**: Detailed archetype playbooks and overlay cards. SDK-neutral
- **references/field_notes.md**: Practitioner observations from real integrations, organized by topic. Notes are tagged with the SDKs they apply to
- **references/sdk_capability_matrix.md**: Cross-SDK feature availability and effort differences — the screen that keeps a plan inside what the target SDK can do
- **references/unity_sdk_reference.md**: Unity SDK technical knowledge and doc routing
- **references/unreal_sdk_reference.md**: Unreal SDK technical knowledge, Blueprint and C++ API surface, built-in components, and doc routing
- **references/visionos_sdk_reference.md**: Native visionOS SDK technical knowledge, Swift API surface, the platform gaze limitation, ExitPoll SwiftUI stack, and doc routing
- **references/androidxr_sdk_reference.md**: Native Android XR SDK technical knowledge, Kotlin API surface, config and upload workflow, and doc routing
- **references/webxr_sdk_reference.md**: WebXR SDK technical knowledge, framework matrix, API surface, and doc routing
- **references/track_plan_template.md**: Output template with quick plan and full plan formats
- **references/example_plans.md**: Example track plans for common project shapes
- **references/queryable_data.md**: Generated reference — what the platform captures automatically and how custom data becomes queryable
- **c3d-progress-tracker-template.md**: Worksheet template for tracking implementation progress per client

**Read this file first.** Load reference files progressively — only when needed for the current step. Do not load all reference files at once, and **never load more than one SDK reference file**: load only the one matching the project's target. Each SDK reference is self-contained, including its dashboard routing, so there is never a reason to open a second one.

## Companion tools

This skill works standalone, but two companions make it materially better. Mention them when relevant; never require them.

- **Cognitive3D MCP server** — read tools (`list_slicer_fields`, `get_field_values`, `get_scene_sessions`, `query_data`, and others) turn validation and data questions into live lookups instead of dashboard round-trips, and objectives/ExitPoll configuration can be done programmatically (see Step 7). The MCP is SDK-agnostic: it talks to the platform, not to the app, so everything here applies equally on every target. It earns the most on Android XR and WebXR, which have the least local tooling for confirming that data actually arrived. If the team will use AI tooling against their data, recommend setting it up: https://docs.cognitive3d.com/mcp-server/getting-started/. **The MCP and `references/queryable_data.md` answer different questions — use both.** The reference describes the platform's built-in fields with units and planning implications; `list_slicer_fields` tells you what a *specific* project actually holds, including its own custom properties and events. Confirm a field exists in the target project with the MCP; decide what is worth tracking from the reference.
- **`cognitive3d-public-api` skill** (same marketplace) — for REST API query construction, data pipelines, and programmatic exports. Route there when the developer's question is "how do I pull this data out," not "what should I track."

---

## Critical rules

### 1. Identify the SDK before anything technical

Every technical answer in this skill depends on the target SDK, and several strategy recommendations do too. Establish the target before routing any implementation question, and before finalizing any plan. See **Step 0**.

Never answer an implementation question with generic Cognitive3D knowledge when the SDK is unknown — the APIs differ in shape (WebXR custom events require a position argument, the others do not; Android XR caps events at ten properties and associates dynamic objects by convention rather than by parameter), the workflows differ in kind (in-engine tooling versus web app uploads), and several features exist on one target and not another. Four targets need a second question, and Step 0 covers each: on **Vision Pro**, native Swift or Unity, because they are different SDKs entirely; on **Unreal**, Blueprint or C++, because the Blueprint custom event variant stringifies property values and the C++ variant does not; on **Android XR**, Jetpack XR or Meta Spatial; on **WebXR**, which framework. Ask, or infer from the project, then route.

### 2. Discovery before implementation

When a developer or client asks to integrate, set up, or improve Cognitive3D analytics, you must ask the **Core Discovery Questions** and wait for answers before recommending instrumentation, writing code, or editing files.

Do not infer business questions from project structure alone. You may inspect the project for technical context, but that is not a substitute for discovery.

**Fast path for direct technical questions.** Discovery and the progress tracker apply to planning and integration engagements. When a developer asks a specific how-do-I question ("how do I add a custom event?", "why is my material white in replay?"), skip the workflow and go straight to Step 7 (technical routing) — do not run discovery, and do not create a tracker file. Identifying the SDK (rule 1) is still required, because the answer differs. If the question reveals a broader planning need, offer the full workflow rather than imposing it.

### 3. Project conventions outrank generic advice

Before recommending instrumentation or writing code, check whether the project defines its own Cognitive3D conventions — an implementation guide, data design document, telemetry standard, project configuration, or a relevant section of `CLAUDE.md` or `CONTRIBUTING.md`. Look in the repo root, in `docs/`, and at any markdown file whose name mentions telemetry, analytics, instrumentation, data design, or Cognitive3D.

If such a document exists, **it wins**. Match its event naming, property casing, required-property lists, environment layout and session-tag vocabulary, even where this skill's generic recommendations differ. A team with a house standard cares more about conformance than about optimality, and a parallel scheme is worse than an imperfect one applied consistently.

Where the document labels which items are authoring-tool work and which are code, honour those labels rather than re-deriving them, and never offer to perform a step that has to happen in an editor or a web app (see rule 9). The one exception: if a label contradicts what the platform can actually do — for example, it calls something editor work that can only be achieved in code, or vice versa — treat that as a documentation bug to raise with the team, not a rule to follow.

If the project has no such document, offer to draft one. A short conventions file is the cheapest way to keep later instrumentation consistent, and it makes every future session on the project more useful. **If the team ships to more than one SDK, this document matters more, not less**: event names, property keys and units must be identical across builds or the dashboard series split permanently.

### 4. Never read or expose credentials

Never read, log, store, or output API keys, developer keys, SSO secrets, or credentials from any source — Unity's `Cognitive3D_Preferences`, Unreal's `c3dlocal/Cognitive3DKeys.ini` and `Config/c3dlocal/Cognitive3DSettings.ini`, visionOS's Info.plist `APPLICATION_API_KEY`, Android XR's `assets/cognitive3d.json`, a WebXR `settings.js` or `.env`, environment variables, config files, build scripts, or any other file that may contain secrets. If the SDK needs keys, instruct the developer to enter them themselves.

Note for Unreal: the docs state explicitly that `Cognitive3DKeys.ini` should not be in source control. Checking that it is git-ignored is a legitimate and useful thing to verify, and it needs no reading of the file's contents.

Note for WebXR: the Application Key is necessarily shipped to the browser. That is inherent to a client-side SDK, not a misconfiguration. Recommend sourcing it from an environment variable at build time so it is not committed to the repository, and be clear that this keeps it out of the source tree rather than out of the client. Never read or echo the value itself.

### 5. Strategy before code

The strategy docs answer **what to track and why**. The SDK references answer **how to implement it**. Keep them separate. Do not jump to code until the tracking plan is clear.

### 6. Smallest useful instrumentation

Do not recommend every available feature by default. A good plan answers the developer's questions with the smallest decision-grade set of custom events, dynamic objects, session properties, participant properties, objectives, exit polls, and tags.

### 7. Properties over event-name proliferation

Do not create separate events for every variation. Use one event plus properties when that answers the question better. Use explicit units in property names — at minimum, durations should use `_seconds`.

### 8. Verify live docs when freshness matters

Do not hard-code SDK versions, release notes, supported hardware, dashboard UI paths, or API auth details into recommendations. When freshness matters, route to live docs. This applies with extra force to WebXR, where the framework support matrix is still expanding.

### 9. Authoring-tool workflows vs code tasks

Not all implementation steps are code tasks. Before writing a script for any SDK feature, establish which kind of task it is on this SDK, and check whether the project already has an established pattern for it.

- **Unity**: many features — dynamic objects, scene setup, exit poll hook placement, component configuration — are primarily Editor workflows (add component, configure in Inspector, drag references). Check for an existing Editor-based pattern before generating code, and describe the Editor steps instead when one exists.
- **Unreal**: editor-first in the same way, with two additions. There are **two authoring surfaces**, Blueprint and C++, so establish which the project uses and answer in that one rather than defaulting to C++. And several capabilities that look automatic are **opt-in built-in components** (framerate, HMD orientation, room size, battery, boundary events, controller tracking loss, hand and arm metrics, input tracking), so the right answer is often "add this component," not "write this code."
- **visionOS**: no editor. Everything runtime is Swift against `Cognitive3DAnalyticsCore.shared`, configuration is code plus an Info.plist key, and geometry goes through the Upload Web App. Dynamic objects follow RealityKit's ECS pattern (register a component and a system, set the immersive root), which is closer to an engine idiom than the other code-only SDKs.
- **Android XR**: no editor at all. Everything runtime is Kotlin or Java against `Cognitive3DManager`, configuration is a JSON asset rather than a settings window, and scene and object geometry go through the Upload Web App. The check that replaces the editor check here is **whether the feature is documented for this SDK at all**, since its documented surface is the narrowest of the five.
- **WebXR**: there is no editor, so nearly everything the app does at runtime is code. What replaces the editor check is the **adapter capability check**: confirm the project's framework supports the feature before recommending it (`references/sdk_capability_matrix.md`, and the matrix in `references/webxr_sdk_reference.md`). Scene and dynamic object geometry uploads are web app workflows, not code and not editor work.
- **Mattercraft** is the exception within WebXR: it has editor behaviors, a properties panel, and export hotkeys, so the Unity-style "check for an existing editor pattern first" rule does apply there.

Only write code when the feature genuinely requires runtime logic (custom events fired from game logic, session properties set from runtime state, object registration for spawned objects).

### 10. Default to a concise plan

Unless the developer explicitly asks for a full integration roadmap, produce a **quick plan** (Phase 1 only). The full template exists for comprehensive plans, but most first conversations should produce a focused, approachable starting point. See `references/track_plan_template.md` for the quick plan format.

---

## Core workflow

### Step 0: Identify the target SDK

Do this first, in every engagement, before discovery and before any technical answer.

#### Inspect the project

Look for these signals before asking:

| Signal | Target |
| --- | --- |
| `Assets/`, `ProjectSettings/`, `Packages/manifest.json`, `.unity` scene files, `.cs` scripts | **Unity** |
| `manifest.json` containing `com.cognitive3d.c3d-sdk` or a `cvr-sdk-unity` git URL | **Unity**, SDK installed |
| `.uproject`, `Source/`, `Content/`, `Config/DefaultEngine.ini`, `.uasset` | **Unreal** |
| `Plugins/Cognitive3D/`, `Cognitive3D.Build.cs`, `c3dlocal/Cognitive3DKeys.ini` | **Unreal**, SDK installed |
| `BP_Cognitive3DActor` referenced in a level or Blueprint | **Unreal**, SDK wired into a level |
| `.xcodeproj`/`.xcworkspace` with Swift sources and no engine project files | **visionOS** |
| `import Cognitive3DAnalytics`, or `Cognitive3DAnalyticsCore` in source | **visionOS**, SDK installed |
| `CoreSettings`, `SceneData`, `DynamicComponent` in Swift source | **visionOS**, SDK wired in |
| `APPLICATION_API_KEY` in an Info.plist | **visionOS**, SDK configured |
| `build.gradle(.kts)`, `AndroidManifest.xml`, `src/main/kotlin`, no engine project files | **Android XR** |
| `com.cognitive3d:android-xr-sdk` in a Gradle file | **Android XR / Jetpack XR**, SDK installed |
| `com.cognitive3d:meta-spatial-sdk` in a Gradle file | **Android XR / Meta Spatial**, SDK installed |
| `src/main/assets/cognitive3d.json`, or `Cognitive3DManager` in source | **Android XR**, SDK configured |
| `androidx.xr.*` or `com.meta.spatial.*` imports | **Android XR**, platform named |
| `package.json` containing `@cognitive3d/analytics` | **WebXR**, SDK installed |
| `package.json` containing `@cognitive3d/three-mattercraft` | **WebXR / Mattercraft** |
| `three`, `@babylonjs/core`, `playcanvas`, `@wonderlandengine/api` in `package.json` | **WebXR**, framework named |
| `c3d-bundle-playcanvas.umd.js` in the project | **WebXR / PlayCanvas** |
| `.mattercraft` project files | **WebXR / Mattercraft** |
| `navigator.xr`, `requestSession("immersive-vr")`, `renderer.xr` in source | **WebXR** |
| `settings.js` exporting `config.APIKey` and `allSceneData` | **WebXR**, SDK configured |

#### Device names are not toolchains

Two of the targets are commonly described by hardware rather than by how the app is built, and getting this wrong routes the whole engagement to the wrong reference.

| The team says | It could be | Settle it by |
| --- | --- | --- |
| "we're building for Apple Vision Pro" | native visionOS **or** Unity | `.xcodeproj` with Swift sources, versus an `Assets/` folder and a Unity project |
| "we're on Quest" or "it's an Android headset" | Unity, Unreal **or** native Android XR | engine project files, versus Gradle files with no engine |

**If both engine project files and platform project files are present, the engine wins.** A Unity app built for Vision Pro produces an Xcode project as build output, and an Android build produces Gradle files; neither makes it a native project. Ask if the signals are genuinely mixed rather than guessing, because the SDKs share no API surface at all.

#### For Android XR, identify the platform too

The same SDK family ships two Maven artifacts: `android-xr-sdk` for Android XR (Jetpack XR) and `meta-spatial-sdk` for Meta Spatial SDK. The Cognitive3D API is shared, but the surrounding platform APIs are not, so anything touching entities, build setup or device behaviour depends on which one the project uses. Read it from the Gradle file, or ask.

Beware the ambiguous case: a Unity or Unreal project that ships to an Android headset is **not** an Android XR project. It is a Unity or Unreal project, possibly using the External Android Plugin. Engine project files (`Assets/`, `.uproject`) settle it; if both engine files and Gradle files are present, the engine wins.

#### For Unreal, identify the authoring surface too

"Unreal" alone is not enough. Establish whether the team works in **Blueprint, C++, or both**, because the answer differs and one difference is not cosmetic: the Blueprint custom event `Send` variant converts every property value to a string, while the C++ `FJsonObject` variant preserves types. A Blueprint-authored plan containing numeric properties will produce data that cannot be averaged or charted. Check `Source/` for gameplay code, and ask if it is ambiguous.

Also confirm the project is **C++ based at all**. The plugin requires it; a pure Blueprint project must be converted first, which is quick but is a prerequisite worth surfacing during discovery rather than at install time.

#### For WebXR, identify the framework too

"WebXR" alone is not enough to answer an implementation question. Capability differs sharply by adapter: dynamic objects exist only on Three.js and Mattercraft, and plain JS has no gaze tracking at all. Determine the framework from `package.json`, the imports in the app's entry point, or by asking.

#### If the project is not available or the signals are ambiguous

Ask directly, as part of the same message as the discovery questions rather than as a separate round trip:

> Which Cognitive3D SDK is this project on — Unity, Unreal, native visionOS, native Android XR, or WebXR? If it targets Vision Pro, is it native Swift or Unity? If Unreal, do you work mostly in Blueprint or C++? If Android XR, is it Jetpack XR or the Meta Spatial SDK? If WebXR, which framework: Three.js, Mattercraft, Wonderland, PlayCanvas, Babylon, or plain WebXR?

#### Record it and route on it

Put the SDK, and the framework, platform or authoring surface, at the top of the progress tracker (Step 3). Then:

- Load **only** the matching SDK reference when you reach Step 7. Never load more than one.
- Load `references/sdk_capability_matrix.md` before finalizing any plan (Step 5), so nothing in the plan is impossible on this target.

#### Targets this skill does not cover

Unity, Unreal, visionOS, Android XR and WebXR are covered here. Cognitive3D ships integrations beyond those — a **C++ SDK** among them — and new ones appear. If a project targets something not in the table above, say so plainly, route to https://docs.cognitive3d.com/ for the correct documentation, and offer the parts of this skill that still apply. The uncovered targets have more in common with Android XR and WebXR than with the engine SDKs: no editor, code-only integration, and geometry through the Upload Web App, so that reference is the closest analogue if the team wants a sense of the shape. The strategy layer — discovery, classification, primitives, phasing, naming, the business question map — is SDK-neutral and remains useful; only Step 7 technical routing does not.

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

- **Target SDK and, for WebXR, the framework** — Step 0. Everything below depends on it
- Target platform and runtime (standalone headset, PC-tethered, mobile browser, desktop browser)
- Whether SDK is installed
- Whether scenes are uploaded
- Whether controllers, hands, gaze, dynamics, events, exit poll hooks are in use
- Whether shared-device or SSO-enabled
- Whether a naming convention exists for custom events and properties
- **Engine-specific context:**
  - *Unity*: render pipeline, scene structure, custom shaders
  - *Unreal*: engine version, whether the project is C++ based, Blueprint versus C++ balance, level streaming or World Partition use, Enhanced Input versus legacy input, which built-in components are present
  - *visionOS*: whether the app is native Swift or Unity-built, RealityKit immersive space versus SwiftUI windows, `shouldEndSessionOnBackground` and `isHandTrackingRequired` settings, and whether anyone has assumed eye tracking exists
  - *Android XR*: Jetpack XR or Meta Spatial, the AndroidX XR alpha the project pins, whether `cognitive3d.json` exists and is filled in, `enable_gaze` state, and whether the asset pipeline can emit glTF Separate rather than GLB
  - *WebXR*: which adapter (if any), bundler and build setup, whether `gazeTrackingSource` is `webxr` or `engine`, whether the renderer is passed to the `C3D` constructor
- **Execution architecture** — how the app's logic actually runs, because this determines where analytics code can hook in
  - *Unity*: standard MonoBehaviour callbacks, custom event system, visual scripting, timeline, state machine, coroutine sequencer, or a mix
  - *Unreal*: Blueprint event graphs, C++ gameplay classes, GameMode and GameInstance lifecycle, Level Blueprints, and which level actually contains `BP_Cognitive3DActor`
  - *visionOS*: where `cognitiveSDKInit()` and the async `startSession()` are called, whether the returned Bool is checked, which scene phase the session is tied to, and whether the immersive root is assigned so dynamic objects are traversed at all
  - *Android XR*: the activity and lifecycle the session is scoped to, where `Cognitive3DManager` is initialized, and the coroutine or executor that carries any sensor sampling
  - *WebXR*: the render loop (`setAnimationLoop` or a custom XR frame callback), the app's own event system or state machine, and where XR session lifecycle events are handled
- **Whether instrumentation already exists** — if so, audit it against the universal baseline in `references/data_strategy.md` before recommending additions
- **Whether the project defines its own C3D conventions** — an implementation guide, data design document, telemetry standard or project configuration. See rule 3: if one exists, it governs naming, required properties and conventions, and this skill's generic advice becomes the fallback
- **Whether the team ships to more than one SDK** — a Unity or Unreal build alongside a WebXR one, or an Unreal flagship beside a Unity pilot. Plan them together. See the cross-SDK section of `references/sdk_capability_matrix.md`

### Step 2: Classify the project

After discovery, classify before recommending. Do not jump straight from discovery to a feature list.

Classification is SDK-neutral: business motion and archetype describe the experience, not the engine.

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
- Multi-SDK delivery (the same experience shipped on more than one target)

#### Example classifications

| Example project | Primary motion | Archetype | Common overlays |
| --- | --- | --- | --- |
| Fire safety simulation | Performance and compliance | Procedural assessment | shared device, LMS |
| VR puzzle adventure | Progression and retention | Progression loop | live tuning |
| Meditation library | Content engagement | Session-based content loop | content catalog |
| Product comparison study | Exploration and evaluation | Exploration loop | survey, cohort tags |
| Academic experiment | Structured research | Trial or condition loop | condition assignment |
| Browser-based product configurator | Exploration and evaluation | Exploration loop | multi-SDK delivery |
| Unreal heavy-equipment operator trainer | Performance and compliance | Procedural assessment | shared device, LMS |
| Android XR spatial productivity app | Content engagement and habit formation | Session-based content loop | multi-scene flows |
| Native Vision Pro design review app | Exploration and evaluation | Exploration loop | privacy-sensitive capture |

### Step 3: Create or update progress tracker

Before proceeding to recommendations, create a progress tracker file for this client using `c3d-progress-tracker-template.md` — or update the existing one if this is a returning project. Save it in the client's project directory, not in the skill folder. Populate it with the target SDK from Step 0 and the discovery answers and classification from Steps 1–2. This is required. Do not proceed to Step 4 until the tracker exists and reflects current SDK, discovery and classification status.

### Step 4: Choose the right primitive

| Need | Best primitive |
|---|---|
| Something happened at a moment in time | Custom event |
| A value describes the whole session | Session property |
| A value describes the person across sessions | Participant property |
| Object seen, used, moved, or fixated | Dynamic object. On visionOS "seen" means faced: the platform exposes no eye tracking |
| Completion logic or sequences | Objective |
| Self-reported feedback or cohort questions | Exit poll |
| Continuous value sampled over time (physiological, performance, gameplay telemetry) | Sensor. **How much is recorded for free varies sharply by SDK** — check `sdk_capability_matrix.md` |
| Analyst-added grouping after the fact | Session tag |

For detailed guidance on event vs property and objective vs event decisions, see `references/data_strategy.md`.

**Availability is not uniform across SDKs.** Dynamic objects in particular are unavailable on several WebXR frameworks. Confirm against `references/sdk_capability_matrix.md` before a primitive becomes a plan row.

### Step 5: Build a phased recommendation

Read `references/data_strategy.md` for strategy guidance. Then read only the relevant sections of `references/playbooks.md` for the chosen archetype(s). Do not load all playbooks — pull only the ones that fit.

Check `references/field_notes.md` for any topics relevant to the chosen archetypes and overlays. Notes are tagged with the SDKs they apply to; skip the ones that do not.

**Before finalizing, screen the plan twice:**

1. **Against `references/sdk_capability_matrix.md`** — remove anything the target SDK or framework cannot do, and flag anything whose effort differs sharply from the other targets, and anything the platform measures differently (the WebXR exit poll UI, Unreal's opt-in built-in components, Android XR's ten-property event cap, and visionOS gaze being head direction rather than eye attention are the usual four). A plan containing an impossible row is worse than a smaller plan.
2. **Against `references/queryable_data.md`** (and `list_slicer_fields` when an MCP server is connected, to catch project-specific fields) — drop anything the platform already captures automatically: session duration, scene time, FPS, comfort, device context, geography. Make sure custom properties are typed so they are queryable (numbers as numbers). Spend the instrumentation budget on app-specific context only.

   **Caveat for non-Unity projects:** `queryable_data.md` is generated from the Unity SDK's capture behaviour, so its automatic-sensor and device-field lists over-promise elsewhere. On **WebXR**, battery, CPU/GPU level, passthrough, wifi, biometrics, pupil diameter and multiplayer ping are all Unity-only. On **Unreal**, the data exists but much of it is **opt-in**: framerate, HMD orientation, room size, battery and boundary events come from built-in components the team has to add, so it is capturable rather than automatic. On **visionOS** the platform itself removes a row that exists nowhere else: Apple does not expose eye-tracking rays to apps, so gaze is head direction and no amount of instrumentation recovers eye attention. On **Android XR** the automatic layer is the narrowest of all: FPS is the only documented automatic sensor, so most of what `queryable_data.md` lists as free has to be recorded explicitly or not at all. Note in particular that the file's **"Android Plugin"** sensor category is the *External Android Plugin* used by Unity and Unreal apps on Android headsets, not the native Android XR SDK, and does not apply to an Android XR project. Where `queryable_data.md` and `sdk_capability_matrix.md` disagree, the capability matrix wins. Dropping an instrumentation row because the platform "already captures it" is only safe once you have checked it against the right SDK.

**Phase 1 — Foundation:** Make the project observable.

- Primary activity start/end, FTUE stages, key dynamic objects, essential session context, dev/prod separation, exit poll hooks, validation sessions

**Phase 2 — Decision-grade instrumentation:** Answer the real questions.

- Step/milestone events, recurrent behavior, error events, participant identity, success metrics, objectives, content/prototype metadata

**Phase 3 — Optimization and experimentation:** Tune and compare.

- UI detail, variant/condition tracking, remote controls, catalog attributes, agent metrics, deeper surveys, social/multiplayer logic

### Step 6: Structure the output

Use `references/track_plan_template.md` to produce the actual recommendation.

**Default to the quick plan format** unless the developer explicitly asks for a comprehensive roadmap. A quick plan includes only:

- Project readback (including the target SDK and framework)
- Top questions to answer now
- Phase 1 priorities (with a brief note on what Phase 2 would add later)
- Event catalog (Phase 1 events only, typically 4–8)
- Validation checklist

**Do not include code snippets in the quick plan.** The plan should describe *what* to track and *why*, not *how* to implement it. Code examples belong in Step 7 (technical routing) when the developer asks you to implement a specific item.

**Use the full plan format** when:

- The developer asks for a complete integration roadmap
- The project is complex enough to need all three phases up front
- The conversation has progressed past Phase 1 validation

See `references/example_plans.md` for pattern references.

### Step 7: Route technical implementation

When the developer asks **how** to implement something, load the SDK reference for the target identified in Step 0 — `references/unity_sdk_reference.md`, `references/unreal_sdk_reference.md`, `references/visionos_sdk_reference.md`, `references/androidxr_sdk_reference.md`, **or** `references/webxr_sdk_reference.md`, never more than one — and route to the correct documentation page from there.

If the SDK is still unknown at this point, establish it before answering. A generic answer here is usually a wrong answer: the custom event API requires a position argument on WebXR and not elsewhere, and caps properties at ten on Android XR; dynamic objects are editor components on Unity and Unreal, a RealityKit component-and-system registration plus an assigned immersive root on visionOS, `userData` tags plus explicit registration on WebXR, and a three-argument registration call on Android XR; spawned-object identity uses ID Pools on the engine SDKs and per-instance registration elsewhere; and several features, ExitPoll among them, exist on some targets and not others.

On Unreal, the authoring surface matters as much as the SDK. Answer in Blueprint or C++ to match the project, and raise the property-typing difference whenever a numeric property is involved.

When routing implementation, identify whether the step is:

- **Authoring-tool workflow** — Unity Editor component setup and Inspector configuration, Unreal Dynamic Object Components and built-in component macros, or Mattercraft behaviors and export hotkeys. visionOS and Android XR have none of these. Describe the steps; do not generate code, and do not offer to perform the step yourself.
- **Upload workflow** — scene geometry and dynamic object meshes. Unity and Unreal use in-engine tooling (Feature Builder, and Unreal's Dynamic Object Manager); visionOS, Android XR and WebXR use the Upload Web App at https://upload.cognitive3d.com with the Developer Key, which accepts glTF Separate (`.gltf` plus `.bin`) and not GLB. Either way this is a human step you describe rather than perform, and on **every** SDK the dynamic object mesh upload is **separate from the scene upload** and is routinely missed.
- **Code task** — custom events, session/participant properties, sensor recording, runtime lifecycle logic, and on WebXR most things that would be Editor work in Unity. Write or modify scripts, following the project's own conventions where they exist (rule 3) and the project's actual execution path.
- **Platform task** — objectives and ExitPoll question sets live on the platform, not in the build, and behave identically on every SDK. Both can be configured **either** on the dashboard **or** programmatically through the MCP server. Ask which route the team wants before doing either; never assume a write-enabled key exists, or that the team wants one.
- **Hybrid** — exit polls need both an in-app trigger and a question set configured on the platform. The hook name in the app must match the platform-side hook exactly. On WebXR the app side is larger than developers expect, because the SDK fetches the question set but renders nothing: the survey UI is the team's to build.

#### Choosing a route for objectives and ExitPoll question sets

Both resources are configurable **either** on the dashboard **or** programmatically through the MCP server, and both routes are fully supported — this is a team preference, not a best practice with one right answer. The dashboard needs no API key and is the right default when a non-developer (analyst, researcher, instructional designer) owns the definitions, or when the team would rather not have a write-enabled key in circulation. The MCP route suits teams who want definitions version-controlled, reviewed like code, or kept in exact parity across environments.

The tool names, doc links, and full platform-constraint lists for both resources live in the SDK reference files ("How do I create or change objectives?" and "How do I ask users questions in-app?"), and the constraints are identical in both because they are properties of the platform. **Read the relevant section there before executing any write.**

Behavioral gates — never skip these:

- **Ask which route the team wants before doing either.** Never assume a write-enabled key exists, or that the team wants one.
- MCP writes need an organization API key with write access and an org- or project-admin role. Organization keys are org-wide — C3D offers no per-project granularity — so a write-enabled key grants write access across every project in the org. That is a legitimate reason for a team to decline and stay on the dashboard. Do not push back on it.
- **Always dry-run first.** Every MCP write for both resources previews what will happen and changes nothing until confirmed. Do not skip the preview because a change looks small.
- **Flag the platform constraints before executing a write** — objective immutability and re-scoring behavior, the permanence of `delete_objective`, ExitPoll versioning and hook-reassignment traps.

#### Before writing any runtime code

- **Verify which execution path the code needs to be on.** If the project uses a custom event system, state machine, or visual scripting, analytics code must integrate with that system rather than bypass it with a standalone script. On WebXR, confirm where the XR frame callback lives, because the adapter's per-frame update has to run there.
- **Check for an existing pattern for this feature.** On Unity, Unreal and Mattercraft, dynamic objects are usually managed through editor components; the default recommendation should be to follow the project's existing pattern rather than introduce a parallel code-based one.
- **Confirm the target framework supports the feature** on WebXR, before writing anything. `references/sdk_capability_matrix.md` and the framework matrix in `references/webxr_sdk_reference.md` are the check.
- **On visionOS, check whether the question is really about eye attention** before writing anything that touches gaze. The platform cannot provide it, so the honest move is to reframe rather than implement. Also check the property-typing constraint: custom event properties are string-only, and numbers that must be queryable belong on the session.
- **On Android XR, check whether the feature is documented for this SDK at all** before writing anything. Its documented surface is the narrowest of the five, and several things that exist elsewhere (ExitPoll, session tags, remote controls) have no page here. Do not port an API across from another SDK's docs.
- **On Unreal, check whether a built-in component already does it** before writing anything. Framerate, HMD orientation, room size, battery, boundary events, controller tracking loss, hand elevation, arm length and input tracking are all shipped components; writing a custom equivalent is wasted work and produces a parallel, non-standard series.

### Step 8: Validate

Every plan should end with a validation plan confirming:

- Scenes uploaded and visible, with the right scene ID and version wired into the project
- Session replay working
- Key events fire with correct properties
- Dynamic object meshes uploaded (a separate step from scene upload, on every SDK)
- Dynamic objects visible in replay with gaze data
- Session and participant properties populated
- Scene geometry exports with correct materials, where the toolchain supports materials at all (some WebXR adapters export geometry only, or do not export; mark the check N/A rather than failed)
- Controller and boundary tracking active, where the platform has them: mark both not applicable on visionOS, which has neither controllers nor a boundary concept. Allow for the WebXR prerequisite that room-size data needs a `bounded-floor` session, and for Unreal's requirement that the corresponding built-in components are present
- Any built-in components the plan depends on are actually added (Unreal)
- Numeric properties arrive as numbers rather than strings (Unreal Blueprint events are the usual culprit)
- No event exceeds the property cap where one applies (Android XR allows ten)
- Gaze-derived findings are described as what the platform actually measured (head direction on visionOS, not eye attention)
- Offline/delayed upload behavior works (if needed and supported on this SDK)
- Dev and production traffic are separated

Add the SDK-specific validation items from the relevant reference file. The Unreal, visionOS, Android XR and WebXR references each carry a troubleshooting quick table of the failure modes worth checking first; the Unity reference routes to https://docs.cognitive3d.com/unity/troubleshooting/ and its project validation page.

**If a Cognitive3D MCP server is connected, validate programmatically as well as visually.** MCP read tools can confirm most of the checklist without leaving the conversation: recent sessions arrived for the scene, key custom events appear with the expected properties, session and participant properties are populated, sensor streams are present. Prefer read tools for verification loops — they close the implement → verify cycle directly. Never use write tools during validation.

---

## Brownfield projects: when instrumentation already exists

If the SDK is already partially integrated, do not start from scratch. Instead:

1. **Audit the existing instrumentation** against the universal baseline in `references/data_strategy.md`. Which of the 12 baseline items are already in place? Which are missing?
2. **Map the execution architecture.** Before recommending where to add analytics, understand how existing code actually gets executed. Analytics must be placed on paths that are live at runtime.
   - *Unity*: projects often have multiple execution paths (MonoBehaviour callbacks, custom event systems, visual scripting, timelines, coroutine sequencers). Search the scene file by script GUID to confirm which scripts are attached to GameObjects. Code existence does not imply scene presence — Unity projects have a dual nature: code (readable from files) and scene state (configured in the Editor, only partially readable from YAML). Confirm with the developer which scripts are active in the scene.
   - *Unreal*: confirm which level actually contains `BP_Cognitive3DActor`, since its presence is what makes a session record at all. Then check whether instrumentation lives in Blueprint graphs, C++ classes, or both, and whether the graphs carrying it are reachable at runtime. A Blueprint node sitting in an unreferenced graph is the Unreal form of a script attached to nothing. Audit the Blueprint-versus-C++ split for custom events specifically: Blueprint-sent properties arrive as strings, so an integration can look complete and still be unqueryable for anything numeric.
   - *visionOS*: confirm that `startSession()` is awaited and its Bool result checked, that `core.entity` is set to the immersive root (without it no dynamic object is ever traversed), and that any SwiftUI window content the plan counts on has a `PositionTrackerView`. Check event properties for numbers sent as strings, which the string-only dictionary makes easy to do by accident.
   - *Android XR*: confirm where `Cognitive3DManager` is initialized and which activity lifecycle the session is scoped to, that `cognitive3d.json` carries a real scene ID and the current version, and that any sensor sampling loop is actually running rather than cancelled with its scope. Check event property counts against the ten-property cap while auditing: an over-cap event looks complete in code and arrives truncated.
   - *WebXR*: confirm that `C3D` is constructed with the renderer, that the adapter's `update()` is actually called in the live render loop, and that session start and end are wired to the real XR session events. Imported-but-unused instrumentation modules are the web equivalent of a script attached to nothing, and a missing per-frame `update()` silently disables gaze and dynamic object tracking while leaving events working, which makes the integration look half-broken rather than misconfigured.
   - If a custom sequencing system exists (event/action framework, state machine, visual scripting graph), map its structure before assuming analytics calls within it are firing.
   - Do not proceed to implementation until you can answer: "how does each analytics-relevant call get executed at runtime?"
3. **Run discovery anyway.** Existing events tell you what the team tried to track. Discovery tells you what they actually need to know. These are often different.
4. **Identify gaps, not a full rewrite.** The recommendation should focus on what to add, adjust, or remove — not a from-scratch plan that ignores existing work.
5. **Check naming consistency and mark event status.** If existing events follow a convention, match it. If they don't, recommend a migration path rather than a parallel naming scheme. In the plan's event catalog, mark every event with the closed Status vocabulary from `references/track_plan_template.md` — `New`, `Keep`, `Amend`, `Revive`, `Replaces: …`, `Retire` — and state a break-risk decision (cut over, dual-send for one release, or leave it alone) for every replaced or retired event. Renames break every dashboard query, saved segment, and objective built on the old name, and split the historical series permanently; never propose one silently.

**Important: stage the output in two steps.** First, present only the audit — what's working, what's missing, and the project classification. Then **stop and ask** the developer if they'd like you to propose improvements before presenting a plan. Do not combine the audit and the plan into a single response. This keeps the audit digestible and gives the developer a chance to correct misunderstandings before you build on them.

**Progress tracker required here too.** After completing the audit and before presenting improvement recommendations, create or update the progress tracker using `c3d-progress-tracker-template.md`. Save it in the client's project directory. Pre-populate it with the audit findings so the existing instrumentation status is captured.

---

## Plan evolution: revisiting and extending

A tracking plan is not a one-time deliverable. Teams should revisit it when:

- Phase 1 is validated and the team is ready for deeper instrumentation
- Business questions change (new stakeholders, new KPIs, pivot in product direction)
- New features are added to the app (multiplayer, AI guide, new content types)
- The team wants to run experiments or compare variants
- **The app ships to an additional SDK** — a Unity or Unreal experience gaining a WebXR, visionOS or native Android XR build, or the reverse. Re-run Step 0 and the capability screen; the strategy usually carries over intact, but the plan rows may not

When revisiting, re-run the discovery questions with updated context. The business motion and archetype may stay the same, but the top questions and phase priorities will evolve.

---

## Universal baseline (always recommend)

These apply to almost every Cognitive3D project, on every SDK. See `references/data_strategy.md` for full details on each:

1. Primary activity lifecycle (start/end pair)
2. Onboarding / FTUE stages with durations
3. Key dynamic objects (only where they answer a question, and only where the SDK supports them)
4. Session context properties
5. Participant identity (when cross-session analysis matters)
6. Dev vs production separation
7. Input and environment verification
8. Exit poll hooks (even if questions aren't ready) — confirm the SDK supports ExitPoll first; it is not documented for Android XR, and it is strongest on Unity and visionOS, which both ship survey UI
9. Controller and boundary tracking verification
10. Scene export fidelity check (custom shaders on Unity; material, Metahuman and Forward Shading export issues on Unreal; glTF Separate and asset-pipeline fit on visionOS and Android XR; adapter export limits on WebXR)
11. Validation sessions
12. At least one analysis surface (objective, query, replay, or comparison)

---

## Quality bar

A strong recommendation should be:

- **Tailored** to the developer's actual goals, not generic
- **Possible** on the SDK and framework the project actually uses
- **Small enough** to implement without feeling overwhelming
- **Rich enough** to answer real questions after validation
- **Consistent** in naming and property conventions, across every SDK the team ships to
- **Explicit** about assumptions and open questions
- **Honest** about privacy-sensitive areas
- **Concrete** about what the team will see in the dashboard after implementation — name the first replay path, objective, or query

---

## Anti-patterns to avoid

1. Recommending before discovery
2. Answering an implementation question without establishing the SDK
3. Carrying a plan shaped for one SDK onto another without a capability screen
4. Encoding properties into event names
5. Using session properties for person-level data
6. Using participant properties for one-session values
7. Tracking everything as a dynamic object
8. Missing duration fields
9. Omitting units from property names
10. Only tracking one-time milestones when recurring behavior matters
11. No dev vs prod separation
12. No validation plan
13. Casual privacy-sensitive collection
14. Recommending features without naming the first analysis use
15. Silently renaming or replacing existing events without a break-risk decision
16. Letting event names or property keys diverge between a team's builds on different SDKs
17. Sending numeric properties through Unreal's Blueprint event variant, where they become strings and stop being queryable as numbers
18. Assuming Unreal captures comfort, framerate or room data automatically, when those come from built-in components the team must add
19. Putting ExitPoll in an Android XR plan without first confirming the SDK supports it
20. Designing Android XR events past the ten-property cap, where the surplus is dropped silently
21. Describing visionOS gaze data as eye attention, or carrying a fixation-based recommendation onto Vision Pro unchanged
22. Routing a Vision Pro project to the wrong reference by assuming the device implies the toolchain

---

## Progress tracking

See **Step 3** in the core workflow. The progress tracker is a required gate — create or update it before moving to recommendations. For brownfield projects, create it after the audit. Update it as work progresses so the next session can pick up where you left off.

---

## Naming conventions

These are SDK-neutral and should be identical across every build a team ships.

- Use stable event names: `module_started`, `step_completed`, `content_bookmarked`
- Put variation in properties, not event names
- Use `_seconds` for time fields, consider `_percent`, `_count`, `_meters`
- Use both stable IDs and readable labels when names can change
- Record recurrence and order when sequence matters (`attempt_count`, `interaction_order`)
- Match the app's domain language (games use `mission`, training uses `module`, research uses `trial`)

---

## Reference file loading guide

Load files in this order as needed. Do not load them all at once, and never load more than one SDK reference.

1. **This file (SKILL.md)** — always read first
2. **references/data_strategy.md** — when you need strategy depth (primitives, phasing, overlays, anti-patterns, business question map)
3. **references/playbooks.md** — when you need archetype-specific recommendations (pull only the relevant playbook, not all of them)
4. **references/field_notes.md** — when you need practitioner observations for the chosen archetypes and overlays
5. **references/sdk_capability_matrix.md** — before finalizing any plan, to confirm every recommendation is possible on the target SDK and framework
6. **references/queryable_data.md** — before finalizing any plan, to screen out what the platform already captures (still read it when an MCP server is connected; use `list_slicer_fields` to confirm what the specific project holds)
7. **references/example_plans.md** — when a concrete pattern example would help
8. **references/track_plan_template.md** — when you're ready to structure the output
9. **references/unity_sdk_reference.md**, **references/unreal_sdk_reference.md**, **references/visionos_sdk_reference.md**, **references/androidxr_sdk_reference.md** *or* **references/webxr_sdk_reference.md** — when the developer asks how to implement a recommendation. Load only the one matching the target identified in Step 0
