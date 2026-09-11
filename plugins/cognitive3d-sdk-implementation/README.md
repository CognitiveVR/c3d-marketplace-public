# cognitive3d-sdk-implementation Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides implementation strategy guidance for the [Cognitive3D](https://cognitive3d.com) SDKs — an XR/VR/AR/MR analytics platform.

Supports **Unity** and **WebXR**. WebXR coverage is deepest for Three.js and Mattercraft; Wonderland, PlayCanvas, Babylon.js and plain WebXR/WebGL are supported at varying capability levels, and the skill screens plans against what each can actually do.

## What it does

Activates the `cognitive3d-sdk-implementation` skill, which guides Claude through the full workflow for implementing Cognitive3D analytics:

- **SDK identification** — detects whether the project is Unity or WebXR, and for WebXR which framework, then loads only the matching technical reference
- **Discovery** — asks the right questions before recommending instrumentation
- **Data strategy** — classifies projects by archetype, maps business questions to SDK primitives
- **Phased implementation** — builds prioritized track plans (custom events, dynamic objects, exit polls, session properties)
- **Capability screening** — keeps plans inside what the target SDK and framework can actually do, which matters because WebXR capability varies sharply by adapter
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
| `references/webxr_sdk_reference.md` | WebXR SDK technical knowledge, framework matrix, API surface, doc routing | WebXR |
| `references/track_plan_template.md` | Output template with quick plan and full plan formats | SDK-neutral |
| `references/example_plans.md` | Example track plans for common project shapes | SDK-neutral |
| `references/queryable_data.md` | What the platform captures automatically and how custom data becomes queryable | Platform |
| `c3d-progress-tracker-template.md` | Worksheet for tracking implementation progress per client | SDK-neutral |

## Supported SDKs

| SDK | Package / source | Docs |
|---|---|---|
| Unity | `https://github.com/CognitiveVR/cvr-sdk-unity.git` (UPM) | https://docs.cognitive3d.com/unity/minimal-setup-guide/ |
| WebXR | `npm install @cognitive3d/analytics` | https://docs.cognitive3d.com/webxr/get-started/ |
| WebXR / Mattercraft | `@cognitive3d/three-mattercraft` | https://docs.cognitive3d.com/webxr/mattercraft/ |

WebXR capability varies by framework. Dynamic objects are available on Three.js and Mattercraft only, and plain JavaScript integrations have no gaze tracking. See https://docs.cognitive3d.com/webxr/framework-support/ and the bundled capability matrix.

## Prerequisites

You'll need a Cognitive3D account and access to a Unity or WebXR project. Accounts and scene setup are managed from the [Cognitive3D Dashboard](https://app.cognitive3d.com). WebXR scene and object geometry is uploaded through the [Upload Web App](https://upload.cognitive3d.com).

## Related

- `cognitive3d-public-api` (same marketplace) — REST API query construction and data pipelines
- [Cognitive3D MCP server](https://docs.cognitive3d.com/mcp-server/getting-started/) — live project reads plus objective and ExitPoll configuration

## Contributing

Found something missing or incorrect? Pull requests and issues are welcome.

## License

MIT
