# cognitive3d-sdk-implementation Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides implementation strategy guidance for the [Cognitive3D](https://cognitive3d.com) SDKs — an XR/VR/AR/MR analytics platform.

Supports **Unity**, **Unreal Engine**, **native Apple Vision Pro** (visionOS), **native Android XR** (Jetpack XR and Meta Spatial SDK), and **WebXR**. Capability varies sharply between targets, and the skill screens every plan against what the project's SDK can actually do.

## What it does

Activates the `cognitive3d-sdk-implementation` skill, which guides Claude through the full workflow for implementing Cognitive3D analytics:

- **SDK identification** — detects whether the project is Unity, Unreal, visionOS, Android XR or WebXR, plus the WebXR framework, Android XR platform or Unreal authoring surface, then loads only the matching technical reference. Device names are not toolchains: a Vision Pro app may be native Swift or Unity, and the skill settles which before answering anything
- **Discovery** — asks the right questions before recommending instrumentation
- **Data strategy** — classifies projects by archetype, maps business questions to SDK primitives
- **Phased implementation** — builds prioritized track plans (custom events, dynamic objects, exit polls, session properties)
- **Capability screening** — keeps plans inside what the target SDK and platform can actually do: WebXR capability varies by adapter, several Unreal metrics come from opt-in components rather than automatic capture, Android XR caps events at ten properties with no documented ExitPoll, and Apple Vision Pro exposes no eye tracking to any app
- **Technical routing** — points to the correct SDK APIs and dashboard docs

The strategy layer is SDK-neutral. Only the technical reference changes between targets.

## Installation

### From the Cognitive3D Public Marketplace (Recommended)

```
/plugin marketplace add CognitiveVR/c3d-marketplace-public
/plugin install cognitive3d-sdk-implementation@c3d-marketplace-public
```

### From GitHub

Add to your Claude Code settings:

```json
{
  "plugins": [
    {
      "source": "github",
      "repo": "CognitiveVR/c3d-marketplace-public"
    }
  ]
}
```

### Local (development)

```bash
claude --plugin-dir ./plugins/cognitive3d-sdk-implementation
```

## Usage

The skill activates automatically when you ask about Cognitive3D SDK integration, tracking plans, or XR analytics instrumentation. You can also ask for it by name ("use the cognitive3d-sdk-implementation skill").

It will ask which SDK the project targets if it cannot determine that from the project itself, then load only the matching technical reference.

## Bundled References

| File | Contents | Scope |
|------|----------|-------|
| `references/data_strategy.md` | Strategy layer — primitives, phasing, overlays, naming conventions, anti-patterns | SDK-neutral |
| `references/playbooks.md` | Detailed archetype playbooks and overlay cards | SDK-neutral |
| `references/field_notes.md` | Practitioner observations from real integrations, tagged by SDK | Mixed |
| `references/sdk_capability_matrix.md` | Cross-SDK feature availability, effort differences, and plan screening rules | Cross-SDK |
| `references/unity_sdk_reference.md` | Unity SDK technical knowledge and doc routing | Unity |
| `references/unreal_sdk_reference.md` | Unreal SDK technical knowledge, Blueprint and C++ API, built-in components, doc routing | Unreal |
| `references/visionos_sdk_reference.md` | Native visionOS SDK knowledge, Swift API, gaze limitation, ExitPoll SwiftUI stack, doc routing | visionOS |
| `references/androidxr_sdk_reference.md` | Native Android XR SDK knowledge, Kotlin API, config and upload workflow, doc routing | Android XR |
| `references/webxr_sdk_reference.md` | WebXR SDK technical knowledge, framework matrix, API surface, doc routing | WebXR |
| `references/track_plan_template.md` | Output template with quick plan and full plan formats | SDK-neutral |
| `references/example_plans.md` | Example track plans for common project shapes | SDK-neutral |
| `references/queryable_data.md` | What the platform captures automatically and how custom data becomes queryable | Platform |
| `c3d-progress-tracker-template.md` | Worksheet for tracking implementation progress per client | SDK-neutral |

## Supported SDKs

| SDK | Package / source | Docs |
|---|---|---|
| Unity | `https://github.com/CognitiveVR/cvr-sdk-unity.git` (UPM) | https://docs.cognitive3d.com/unity/minimal-setup-guide/ |
| Unreal Engine | https://github.com/CognitiveVR/cvr-sdk-unreal/releases | https://docs.cognitive3d.com/unreal/get-started/ |
| Apple Vision Pro (visionOS) | https://github.com/CognitiveVR/c3d-sdk-visionOS | https://docs.cognitive3d.com/visionos/get-started/ |
| Android XR (Jetpack XR) | `com.cognitive3d:android-xr-sdk` | https://docs.cognitive3d.com/android-xr/get-started/ |
| Android XR (Meta Spatial) | `com.cognitive3d:meta-spatial-sdk` | https://docs.cognitive3d.com/android-xr/get-started/ |
| WebXR | `npm install @cognitive3d/analytics` | https://docs.cognitive3d.com/webxr/get-started/ |
| WebXR / Mattercraft | `@cognitive3d/three-mattercraft` | https://docs.cognitive3d.com/webxr/mattercraft/ |

Capability is not uniform across targets, which is why the skill screens every plan:

- **WebXR** varies by framework. Dynamic objects are available on Three.js and Mattercraft only, and per-object gaze follows them. See https://docs.cognitive3d.com/webxr/framework-support/.
- **Unreal** requires a C++ based project, and several metrics that are automatic on Unity (framerate, HMD orientation, room size, battery, boundary events) come from opt-in built-in components. Its Blueprint custom event variant also stringifies property values, so numeric properties need the C++ path.
- **visionOS** is the native Swift SDK for Apple Vision Pro. A Vision Pro app built in Unity is a Unity project instead. The platform exposes no eye-tracking rays to applications, so gaze is a head-direction signal and attention findings have to be described accordingly; custom event properties are string-only, and the SDK ships SwiftUI ExitPoll views with offline question-set caching.
- **Android XR** is the native Kotlin SDK for apps with no game engine, and is not the External Android Plugin that Unity and Unreal use on Android headsets. Custom events are capped at ten properties, FPS is the only automatic sensor, and ExitPoll has no documented support.
- **Unity** is the most feature-complete target and the only one that excludes in-editor sessions from dashboards automatically.

## Prerequisites

You'll need a Cognitive3D account and access to a Unity, Unreal, visionOS, Android XR or WebXR project. Accounts and scene setup are managed from the [Cognitive3D Dashboard](https://app.cognitive3d.com). Unity and Unreal upload scene and object geometry through in-engine tooling; visionOS, Android XR and WebXR use the [Upload Web App](https://upload.cognitive3d.com), which accepts glTF Separate rather than GLB.

## Related

- `cognitive3d-public-api` (same marketplace) — REST API query construction and data pipelines
- [Cognitive3D MCP server](https://docs.cognitive3d.com/mcp-server/getting-started/) — live project reads plus objective and ExitPoll configuration

## Contributing

Found something missing or incorrect? Pull requests and issues are welcome.

## License

MIT
