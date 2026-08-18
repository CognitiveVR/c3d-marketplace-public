# Cognitive3D Implementation Progress Tracker

> **Instructions:** Copy this template for each project. Update status fields as work progresses. This file is designed for both human review and LLM context loading — keep entries concise and structured.

---

## Project Info

| Field | Value |
|---|---|
| **Client / Project Name** | |
| **Unity Version** | |
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
| What is the experience about? | `[ ] Not asked` `[ ] Asked` `[x] Answered` | |
| Who are the target users? | `[ ] Not asked` `[ ] Asked` `[x] Answered` | |
| What decisions/insights to support? | `[ ] Not asked` `[ ] Asked` `[x] Answered` | |
| Key interactions/activities? | `[ ] Not asked` `[ ] Asked` `[x] Answered` | |
| KPIs/success metrics? | `[ ] Not asked` `[ ] Asked` `[x] Answered` | |
| Follow-up questions needed? | `[ ] None` `[ ] Pending` `[x] Completed` | |

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
| Custom shader check | `Not started` / `N/A` / `Done` | |
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

Mesh upload is a separate step from scene upload. Adding the `DynamicObject` component does not upload the mesh — use **Feature Builder > Dynamic Objects** to export and upload meshes.

| Object | Component Added | Mesh Uploaded (Feature Builder) | Visible in Replay | Gaze Verified | Notes |
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

| Hook Location | Code Placed | Questions Configured | Tested | Notes |
|---|---|---|---|---|
| | `Yes` / `No` | `Yes` / `No` | `Yes` / `No` | |

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
| Custom shaders correct | `Pass` / `Fail` / `N/A` | | |
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
Stage: [Discovery / Phase 1 / Phase 2 / Phase 3 / Validation / Complete]
Last action: [what was done last]
Next action: [what needs to happen next]
Blockers: [none / describe]
```
