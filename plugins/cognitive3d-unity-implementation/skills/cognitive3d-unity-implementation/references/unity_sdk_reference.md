# Cognitive3D Unity SDK Technical Reference

This file is a routing layer for Unity SDK implementation questions. It helps find the right live docs page quickly.

**Primary docs root:** https://docs.cognitive3d.com/

## How to use this file

This file is advisory, not authoritative. Treat the linked docs pages as the source of truth.

### Operating rules

1. **Use the deepest relevant page first.** Do not answer from the docs root when a feature page exists.
2. **Treat this file as a fast-lookup layer.** Treat the linked docs page as authoritative.
3. **Escalate to live docs when freshness matters.** If the question involves latest versions, compatibility, recently changed setup, release notes, hardware support, dashboard navigation paths, or API/MCP auth details — verify from the live page.
4. **If browsing is unavailable, answer with clear caveats.** Give the best likely page(s) to confirm.
5. **Stay at the user's level.** Do not dump low-level implementation detail unless asked.
6. **Prefer stable concepts over brittle UI narration.** If UI labels may have changed, anchor to page names and current docs URLs.

### Trust hierarchy

Use this order of trust when answering:

1. **Exact live feature page** — e.g., Unity Dynamic Objects, Unity Custom Events
2. **Engine landing page** — e.g., Unity minimal setup guide
3. **Release pages / repositories** — for latest versions, release notes, compatibility
4. **Docs portal root** — when the question is broad or needs routing
5. **API/Data and MCP docs** — for programmatic reads, and for objective and ExitPoll configuration writes

### Change-watch anchors (verify live before quoting)

- Unity latest releases: https://github.com/CognitiveVR/cvr-sdk-unity/releases/latest
- Docs root: https://docs.cognitive3d.com/
- SDK downloads: https://docs.cognitive3d.com/download/
- Supported hardware: https://docs.cognitive3d.com/hardware/

---

## Common implementation mental model

Across all Cognitive3D integrations, the recurring model is:

1. **Create/choose a project** in the dashboard
2. **Get the right keys** (Developer Key retrieves Application Key)
3. **Install the SDK** via Unity Package Manager
4. **Associate the app** with the project and configure scene handling
5. **Start and end sessions** correctly
6. **Attach session metadata** — participant info, tags, session properties
7. **Record telemetry** — gaze/fixations, custom events, sensors, dynamic objects, exit polls, remote controls, media, local cache
8. **Upload scenes/meshes/object geometry**
9. **Validate in dashboard** — replay, scene/object views, analysis, performance
10. **Troubleshoot** if data is missing

### Canonical nouns

Organization, Project, Scene, Scene Version, Session, Participant, Dynamic Object, Custom Event, Sensor, ExitPoll, Remote Controls

Dashboard Concepts page: https://docs.cognitive3d.com/dashboard/concepts/

---

## Fast route by question type

### "How do I install the SDK?"

- Unity minimal setup: https://docs.cognitive3d.com/unity/minimal-setup-guide/
- Installation via UPM git URL: `https://github.com/CognitiveVR/cvr-sdk-unity.git`
- Release watch: https://github.com/CognitiveVR/cvr-sdk-unity/releases/latest

### "How do I configure keys/auth?"

- Unity setup page: https://docs.cognitive3d.com/unity/minimal-setup-guide/
- Uses Developer Key to retrieve Application Key from dashboard

### "How do I upload scenes?"

- Unity Scenes: https://docs.cognitive3d.com/unity/scenes/

**This is an Editor workflow.** Scene uploads are done entirely through the Cognitive3D Unity Editor tools (Scene Manager in SDK 2.3+, or Project Setup/Preferences in earlier versions). No code is involved. Refer to the docs page for current steps.

### "How do I track dynamic objects?"

- Unity Dynamic Objects: https://docs.cognitive3d.com/unity/dynamic-objects/
- Important sections: component setup, ID pools, controllers, engagements, uploading meshes
- Note: Unsupported visualization for material/texture changes, particle systems, skeletal animations, mesh deformation

**This is primarily an Editor workflow.** Adding components and uploading meshes should be done through the **Feature Builder > Dynamic Objects** window (Cognitive3D menu). This handles component setup, mesh export, and upload in batch. Refer to the docs page for current steps — especially: https://docs.cognitive3d.com/unity/dynamic-objects/#uploading-dynamic-meshes-window

**Only use code (ID Pools) when** objects are spawned at runtime and don't exist in the scene at build time (e.g., projectiles, procedurally generated items, networked avatars). If the project already has `DynamicObject` components on similar objects, follow that pattern.

### "How do I record custom events?"

- Unity Custom Events: https://docs.cognitive3d.com/unity/customevents/
- Key sections: Setup, Duration, Dynamic Object Custom Events, Dynamic Object Properties, Event Sensors, Optimized Implementation, Trigger Areas

### "How do I track gaze and fixations?"

- Unity Gaze/Fixations: https://docs.cognitive3d.com/unity/gaze-fixations/
- Fixations explainer: https://docs.cognitive3d.com/fixations/
- Supported hardware: https://docs.cognitive3d.com/hardware/

### "How do I record sensors/performance?"

- Unity Sensors: https://docs.cognitive3d.com/unity/sensors/
- Unity Performance: https://docs.cognitive3d.com/unity/performance/
- Dashboard App Performance: https://docs.cognitive3d.com/dashboard/app-performance/

### "How do I add session/participant metadata?"

- Unity comprehensive setup (Sessions, Properties, Tags, Participants): https://docs.cognitive3d.com/unity/comprehensive-setup-guide/
- Unity Participants: https://docs.cognitive3d.com/unity/participants/
- Session properties can be set during the experience and overwrite prior values with the same key
- Manual session start supported if consent is needed before recording

### "How do I ask users questions in-app?"

- Unity ExitPoll: https://docs.cognitive3d.com/unity/exitpoll/
- Dashboard ExitPoll Results: https://docs.cognitive3d.com/dashboard/exitpoll-results/
- MCP ExitPoll tools: https://docs.cognitive3d.com/mcp-server/exitpoll/

**This is a hybrid task.** The in-app trigger is code (or an Editor-placed component); the question set and hook live on the platform. Question sets are configurable **either** on the dashboard **or** via MCP — `create_exitpoll_question_set`, `archive_exitpoll_question_set`, `create_exitpoll_hook`, `update_exitpoll_hook` for writes, `get_exitpoll_configuration` and `get_exitpoll_question_set` for reads. Writes need a write-enabled organization key with an org- or project-admin role. Ask which route the team wants rather than assuming.

Two failure modes worth naming up front: question set versions are immutable, so an edit creates a new version and **existing hooks stay on the old one** until reassigned with `update_exitpoll_hook`; and a hook with no question set assigned is skipped silently at runtime rather than erroring. See Step 7 in SKILL.md.

### "How do I control runtime behavior remotely?"

- Dashboard Remote Controls: https://docs.cognitive3d.com/dashboard/remote-controls/
- Unity Remote Controls: https://docs.cognitive3d.com/unity/remote-controls/

### "Is this supported on our device?"

- Supported hardware: https://docs.cognitive3d.com/hardware/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Privacy language: https://docs.cognitive3d.com/legal/

### "How do I create or change objectives?"

- MCP objective tools: https://docs.cognitive3d.com/mcp-server/objectives/
- Objective concepts and step types: https://docs.cognitive3d.com/dashboard/creating-objectives/

**This is a platform task, not a code task, and there are two routes.** The dashboard needs no API key and suits teams where a non-developer owns objectives, or who would rather not have a write-enabled key in circulation. The MCP server (`create_objective`, `update_objective`, `delete_objective`) needs a write-enabled organization key and suits teams who want definitions version-controlled or staging and production kept in exact parity. Ask which the team wants rather than assuming.

See Step 7 in SKILL.md for route selection and for the platform constraints that apply either way — `sequential` immutability, the ~30-day re-scoring window, the 32-character name cap — plus the MCP-specific dry-run step. ExitPoll question sets have the same two routes; see "How do I ask users questions in-app?" above.

### "How do I access data programmatically?"

- API/Data get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/

### "How do I expose Cognitive3D to an AI client or MCP?"

- MCP getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- Note: MCP is a data and configuration layer, not the SDK instrumentation layer — it cannot instrument the app, but it can read project data and **write objectives and ExitPoll configuration**. Config details are highly freshness-sensitive.

---

## Unity SDK directory

### First-stop pages

- Getting Started / minimal setup: https://docs.cognitive3d.com/unity/minimal-setup-guide/
- Comprehensive setup: https://docs.cognitive3d.com/unity/comprehensive-setup-guide/
- Feature Builder: https://docs.cognitive3d.com/unity/feature-builder/
- Pre-launch checklist: https://docs.cognitive3d.com/unity/prelaunch-checklist/
- Project validation: https://docs.cognitive3d.com/unity/project-validation/
- Terminology: https://docs.cognitive3d.com/unity/terminology/
- Runtimes: https://docs.cognitive3d.com/unity/runtimes/
- Updating the SDK: https://docs.cognitive3d.com/unity/updates/

### Core feature pages

- Scenes: https://docs.cognitive3d.com/unity/scenes/
- Custom Events: https://docs.cognitive3d.com/unity/customevents/
- Dynamic Objects: https://docs.cognitive3d.com/unity/dynamic-objects/
- Gaze/Fixations: https://docs.cognitive3d.com/unity/gaze-fixations/
- ExitPoll Survey: https://docs.cognitive3d.com/unity/exitpoll/
- Sensors: https://docs.cognitive3d.com/unity/sensors/
- Participants: https://docs.cognitive3d.com/unity/participants/
- External Android Plugin: https://docs.cognitive3d.com/unity/android-plugin/
- Remote Controls: https://docs.cognitive3d.com/unity/remote-controls/
- Audio Recording: https://docs.cognitive3d.com/unity/audio-recording/

### Extra feature pages

- Ready Room: https://docs.cognitive3d.com/unity/ready-room/
- Active Session View: https://docs.cognitive3d.com/unity/active-session-view/
- Built-In Components: https://docs.cognitive3d.com/unity/components/
- Media & 360: https://docs.cognitive3d.com/unity/media/
- Multiplayer: https://docs.cognitive3d.com/unity/multiplayer/
- Local Cache: https://docs.cognitive3d.com/unity/local-cache/
- SDK Data Connector: https://docs.cognitive3d.com/unity/sdk-data-connector/

### Advanced/ops pages

- Preferences: https://docs.cognitive3d.com/unity/preferences/
- Data Uploader: https://docs.cognitive3d.com/unity/data-uploader/
- HMD Specific Information: https://docs.cognitive3d.com/unity/hmd-specific-info/
- Troubleshooting: https://docs.cognitive3d.com/unity/troubleshooting/
- Performance: https://docs.cognitive3d.com/unity/performance/

### High-value sections inside pages

**Comprehensive setup** sections:

- Begin and End Sessions
- Session Name
- Session Property
- Session Tags
- Gaze and Fixations
- Scenes
- Dynamic Objects
- ExitPoll
- Custom Events
- Sensors
- Participants
- Local Cache
- Attributions

**Custom Events** sections:

- Setup
- Duration
- Dynamic Object Custom Events
- Dynamic Object Properties
- Event Sensors
- Optimized Implementation
- Trigger Areas

**Built-In Components** page includes:

- Boundary (SDK 1.6.5+, Meta and OpenXR platforms)
- Multiplayer integrations: Unity Netcode for GameObjects, Normcore

### Best-answer routing

- **installation/setup** → minimal setup first, then comprehensive setup
- **consent/session lifecycle** → comprehensive setup → Begin and End Sessions
- **metadata/tags/properties** → comprehensive setup → Session Name / Session Property / Session Tags
- **events** → Custom Events
- **interactables/controllers/hands** → Dynamic Objects
- **dashboard views** → Session Details, Analysis Tool, Objectives, Scene/Object views
- **missing data** → Project Validation, Troubleshooting, Data Uploader, Performance

---

## Dashboard directory

### Concepts and framing

- Concepts: https://docs.cognitive3d.com/dashboard/concepts/

### Replay and behavior review

- Session Replay: https://docs.cognitive3d.com/dashboard/session-replay/
- Embeddable Session Replay: https://docs.cognitive3d.com/dashboard/embeddable-session-replay/

### Dashboard summaries

- Project Overview: https://docs.cognitive3d.com/dashboard/project-overview/
- App Performance: https://docs.cognitive3d.com/dashboard/app-performance/
- Live Operations: https://docs.cognitive3d.com/dashboard/live-operations/
- Demographics: https://docs.cognitive3d.com/dashboard/demographics/
- Spatial Optimization: https://docs.cognitive3d.com/dashboard/spatial-optimization/
- ExitPoll Results: https://docs.cognitive3d.com/dashboard/exitpoll-results/

### Scene-centric analysis

- Scene Viewer: https://docs.cognitive3d.com/dashboard/scene-viewer/
- Session Details: https://docs.cognitive3d.com/dashboard/session-details/
- Object Explorer: https://docs.cognitive3d.com/dashboard/object-explorer/
- Object Details: https://docs.cognitive3d.com/dashboard/object-details/

### Objectives

- Objectives Summary: https://docs.cognitive3d.com/dashboard/objectives-summary/
- Objective Details: https://docs.cognitive3d.com/dashboard/objective-details/
- Creating Objectives: https://docs.cognitive3d.com/dashboard/creating-objectives/

### Participants

- Participant Summary: https://docs.cognitive3d.com/dashboard/participants-summary/
- Participant Details: https://docs.cognitive3d.com/dashboard/participant-details/

### Analysis

- Simple Analysis: https://docs.cognitive3d.com/dashboard/simple-analysis/
- Advanced Analysis: https://docs.cognitive3d.com/dashboard/advanced-analysis/

### Settings

- Organization Settings: https://docs.cognitive3d.com/dashboard/organization-settings/
- Project Settings: https://docs.cognitive3d.com/dashboard/project-settings/
- Remote Controls: https://docs.cognitive3d.com/dashboard/remote-controls/
- Personal Settings: https://docs.cognitive3d.com/dashboard/personal-settings/

### Upload, export, and integrations

- Scene Uploads: https://docs.cognitive3d.com/unity/scenes/
- Object Uploads: https://docs.cognitive3d.com/unity/dynamic-objects/
- LMS Integration: https://docs.cognitive3d.com/dashboard/lms/
- Filters: https://docs.cognitive3d.com/dashboard/filters/
- Data Export: https://docs.cognitive3d.com/dashboard/data-export/
- Crash Reports: https://docs.cognitive3d.com/dashboard/crash-reports/

---

## API and MCP directory

### API/Data

- Get started: https://docs.cognitive3d.com/api/get-started/
- Postman docs: https://docs.api.cognitive3d.com/
- Lobby System: https://docs.cognitive3d.com/api/lobbies/
- Attributions: https://docs.cognitive3d.com/api/attributions/
- R package: https://docs.cognitive3d.com/api/r/getting-started/

### MCP Server

- Getting started: https://docs.cognitive3d.com/mcp-server/getting-started/
- Organization queries: https://docs.cognitive3d.com/mcp-server/organization/
- Project queries: https://docs.cognitive3d.com/mcp-server/projects/
- Session queries: https://docs.cognitive3d.com/mcp-server/sessions/
- Objective queries: https://docs.cognitive3d.com/mcp-server/objectives/

---

## General reference pages

- Docs root: https://docs.cognitive3d.com/
- SDK downloads: https://docs.cognitive3d.com/download/
- Supported hardware: https://docs.cognitive3d.com/hardware/
- Fixations explainer: https://docs.cognitive3d.com/fixations/
- Metrics glossary: https://docs.cognitive3d.com/metrics-glossary/
- Sample privacy language: https://docs.cognitive3d.com/legal/
- Firewall settings: https://docs.cognitive3d.com/firewall/
- Mixing Unreal and Unity: https://docs.cognitive3d.com/scenarios/mixing-unreal-unity/

### High-staleness surfaces (always verify live)

- Latest SDK versions
- Release notes
- Supported engine versions
- Supported hardware / eye tracking
- Package install coordinates
- Dashboard navigation paths
- API key formats / auth examples
- MCP server config examples
- Device/platform feature support
