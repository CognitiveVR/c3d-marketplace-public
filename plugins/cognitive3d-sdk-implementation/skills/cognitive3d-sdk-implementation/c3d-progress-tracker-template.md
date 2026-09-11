# Cognitive3D Implementation Progress Tracker

> **Instructions:** Copy this template for each project. Update status fields as work progresses. This file is designed for both human review and LLM context loading — keep entries concise and structured.

---

## Project Info

| Field | Value |
|---|---|
| **Client / Project Name** | |
| **Target SDK** | `Unity` / `Unreal` / `Android XR` / `WebXR` |
| **Engine / Framework / Platform** | Unity or Unreal version; Jetpack XR or Meta Spatial; or Three.js / Mattercraft / Wonderland / PlayCanvas / Babylon / plain WebXR |
| **Authoring Surface** (Unreal) | `Blueprint` / `C++` / `Both` |
| **Target Platform(s)** | |
| **SDK Version** | |
| **Dashboard Project URL** | |
| **Start Date** | |
| **Last Updated** | |
| **Updated By** | |

---

## Discovery Status

| Question | Status | Answer Summary |
|---|---|---|
| What is the experience about? | `Not asked` / `Asked` / `Answered` | |
| Who are the target users? | `Not asked` / `Asked` / `Answered` | |
| What decisions/insights to support? | `Not asked` / `Asked` / `Answered` | |
| Key interactions/activities? | `Not asked` / `Asked` / `Answered` | |
| KPIs/success metrics? | `Not asked` / `Asked` / `Answered` | |
| Follow-up questions needed? | `None` / `Pending` / `Completed` | |

**Discovery notes:**
_Capture anything unusual, constraints, or context that doesn't fit the questions above._

---

## Classification

| Field | Value |
|---|---|
| **Primary Business Motion** | |
| **Archetype 1** | |
| **Archetype 2** (if any) | |
| **Active Overlays** | |

---

## Implementation Phases

### Phase 1: Foundation

| Item | Status | Notes |
|---|---|---|
| SDK installed | `Not started` / `In progress` / `Done` | |
| Scenes uploaded | `Not started` / `In progress` / `Done` | |
| Primary activity lifecycle events | `Not started` / `In progress` / `Done` | |
| FTUE stage events | `Not started` / `In progress` / `Done` | |
| Key dynamic objects registered | `Not started` / `In progress` / `Done` | |
| Session context properties set | `Not started` / `In progress` / `Done` | |
| Dev vs prod separation | `Not started` / `In progress` / `Done` | |
| Exit poll hooks placed | `Not started` / `In progress` / `Done` | |
| Controller tracking verified | `Not started` / `In progress` / `Done` | |
| Boundary tracking verified | `Not started` / `In progress` / `Done` | |
| Scene export fidelity check | `Not started` / `N/A` / `Done` | Unity: custom shaders. Unreal: materials, Forward Shading, Metahumans. Android XR/WebXR: glTF Separate, not GLB |
| Built-in components added (Unreal) | `Not started` / `N/A` / `Done` | Framerate, HMD orientation, room size, battery, boundary as the plan requires |
| Numeric properties verified as numbers | `Not started` / `In progress` / `Done` | Unreal Blueprint events stringify values |
| Event property counts within cap | `Not started` / `N/A` / `Done` | Android XR allows 10 per event; surplus is dropped silently |
| ExitPoll support confirmed for this SDK | `Not started` / `N/A` / `Done` | Not documented for Android XR; verify before relying on it |
| Validation session run | `Not started` / `In progress` / `Done` | |

**Phase 1 blockers / notes:**

---

### Phase 2: Decision-Grade Instrumentation

| Item | Status | Notes |
|---|---|---|
| Step/milestone events | `Not started` / `In progress` / `Done` | |
| Recurrent behavior events | `Not started` / `In progress` / `Done` | |
| Error/incorrect action events | `Not started` / `In progress` / `Done` | |
| Participant identity set up | `Not started` / `N/A` / `Done` | |
| Participant properties | `Not started` / `N/A` / `Done` | |
| Success metrics / outcome events | `Not started` / `In progress` / `Done` | |
| Objectives created | `Not started` / `In progress` / `Done` | |
| Content/prototype metadata | `Not started` / `N/A` / `Done` | |
| Validation session run | `Not started` / `In progress` / `Done` | |

**Phase 2 blockers / notes:**

---

### Phase 3: Optimization and Experimentation

| Item | Status | Notes |
|---|---|---|
| UI interaction events | `Not started` / `N/A` / `Done` | |
| Variant/condition tracking | `Not started` / `N/A` / `Done` | |
| Remote controls | `Not started` / `N/A` / `Done` | |
| AI/voice interaction events | `Not started` / `N/A` / `Done` | |
| Deeper survey questions | `Not started` / `N/A` / `Done` | |
| Advanced cohorting | `Not started` / `N/A` / `Done` | |

**Phase 3 blockers / notes:**

---

## Event Catalog Status

| Event Name | Implemented | Validated | Notes |
|---|---|---|---|
| | `Yes` / `No` | `Yes` / `No` | |

---

## Dynamic Objects Status

Mesh upload is a separate step from scene upload on every SDK, and it is the step most often missed. Unity: add the `DynamicObject` component, then export and upload via **Feature Builder > Dynamic Objects**. Unreal: add the Dynamic Object Component, then export and upload via the **Dynamic Object Manager** (Cognitive3D menu > Feature Builder > Dynamic Object). Android XR: call `registerDynamicObject(name, meshName, entity)` and upload the mesh through the Upload Web App, making sure `meshName` matches the uploaded model name. WebXR: tag with `userData.isDynamic` and `userData.c3dId`, call `registerObjectCustomId`, add to the tracked set, then upload the mesh through the Upload Web App.

Dynamic objects are not available on every WebXR framework. If the project is Wonderland, PlayCanvas, Babylon or plain JS, mark this whole section `N/A` and note what was used instead.

| Object | Registered in App | Mesh Uploaded | Visible in Replay | Gaze Verified | Notes |
|---|---|---|---|---|---|
| | `Yes` / `No` | `Yes` / `No` | `Yes` / `No` | `Yes` / `No` | |

---

## Objectives Status

Objectives can be created on the dashboard or via the MCP server — record which route the team chose.

| Objective | Route | Created | Tested | Notes |
|---|---|---|---|---|
| | `Dashboard` / `MCP` | `Yes` / `No` | `Yes` / `No` | |

---

## Exit Polls Status

ExitPoll is not documented for Android XR; confirm support before filling this section in, and mark it `N/A` with the substitute noted if it is unavailable. On WebXR the SDK fetches the question set but renders nothing, so the in-app survey UI is the team's own work. On Unreal the widgets ship, but a Widget Interaction component on the player or controller is required or the panel cannot be answered. Track the app-side work as its own column rather than assuming the hook is the whole job.

| Hook Location | Hook Placed in App | Survey UI / Interaction Ready | Questions Configured | Tested | Notes |
|---|---|---|---|---|---|
| | `Yes` / `No` | `Yes` / `No` / `N/A` | `Yes` / `No` | `Yes` / `No` | WebXR: UI built. Unreal: Widget Interaction component present |

---

## Validation Summary

| Check | Status | Date | Notes |
|---|---|---|---|
| Scenes visible on dashboard | `Pass` / `Fail` / `Not tested` | | |
| Session replay working | `Pass` / `Fail` / `Not tested` | | |
| Events fire with correct properties | `Pass` / `Fail` / `Not tested` | | |
| Dynamic objects visible + gaze | `Pass` / `Fail` / `Not tested` | | |
| Session properties populated | `Pass` / `Fail` / `Not tested` | | |
| Participant properties populated | `Pass` / `Fail` / `Not tested` | | |
| Controller tracking active | `Pass` / `Fail` / `Not tested` | | |
| Boundary tracking active | `Pass` / `Fail` / `Not tested` | | |
| Scene geometry exports with correct materials | `Pass` / `Fail` / `N/A` | | |
| Numeric properties queryable as numbers | `Pass` / `Fail` / `Not tested` | | |
| Event property counts within SDK cap | `Pass` / `Fail` / `N/A` | | |
| Required built-in components present (Unreal) | `Pass` / `Fail` / `N/A` | | |
| Dev and production traffic separated | `Pass` / `Fail` / `Not tested` | | |
| Offline upload works | `Pass` / `Fail` / `N/A` | | |
| First analysis surface usable | `Pass` / `Fail` / `Not tested` | | |

---

## Open Questions

| # | Question | Status | Resolution |
|---|---|---|---|
| 1 | | `Open` / `Resolved` | |
| 2 | | `Open` / `Resolved` | |

---

## Session Log

_Record key decisions, changes, and progress per work session for continuity._

| Date | Who | What was done | What's next |
|---|---|---|---|
| | | | |

---

## Quick Summary (for LLM context loading)

_Keep this section updated as a 3-5 line summary of current state. This is the first thing an LLM should read to get oriented._

```
Project: [name]
SDK: [Unity | Unreal + Blueprint/C++ | Android XR + Jetpack/Meta Spatial | WebXR + framework]
Stage: [Discovery / Phase 1 / Phase 2 / Phase 3 / Validation / Complete]
Last action: [what was done last]
Next action: [what needs to happen next]
Blockers: [none / describe]
```
