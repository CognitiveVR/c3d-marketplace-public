# c3d-unity-implementation Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides implementation strategy guidance for the [Cognitive3D](https://cognitive3d.com) Unity SDK — an XR/VR/AR/MR analytics platform.

## What it does

Activates the `c3d-unity-implementation` skill, which guides Claude through the full workflow for implementing Cognitive3D analytics in Unity projects:

- **Discovery** — asks the right questions before recommending instrumentation
- **Data strategy** — classifies projects by archetype, maps business questions to SDK primitives
- **Phased implementation** — builds prioritized track plans (custom events, dynamic objects, exit polls, session properties)
- **Technical routing** — points to the correct Unity SDK APIs and dashboard docs

## Installation

### From the Cognitive3D Public Marketplace (Recommended)

```
/plugin marketplace add CognitiveVR/c3d-marketplace-public
/plugin install c3d-unity-implementation@c3d-marketplace-public
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
claude --plugin-dir ./plugins/c3d-unity-implementation
```

## Usage

The skill activates automatically when you ask about Cognitive3D Unity SDK integration, tracking plans, or XR analytics instrumentation. You can also invoke it directly:

```
/c3d-unity-implementation
```

## Bundled References

| File | Contents |
|------|----------|
| `references/data_strategy.md` | Strategy layer — primitives, phasing, overlays, naming conventions, anti-patterns |
| `references/playbooks.md` | Detailed archetype playbooks and overlay cards |
| `references/field_notes.md` | Practitioner observations from real integrations |
| `references/unity_sdk_reference.md` | Unity SDK technical knowledge and doc routing |
| `references/track_plan_template.md` | Output template with quick plan and full plan formats |
| `references/example_plans.md` | Example track plans for common project shapes |
| `c3d-progress-tracker-template.md` | Worksheet for tracking implementation progress per client |

## Prerequisites

You'll need a Cognitive3D account and access to a Unity project. Accounts and scene setup are managed from the [Cognitive3D Dashboard](https://app.cognitive3d.com).

## Contributing

Found something missing or incorrect? Pull requests and issues are welcome.

## License

MIT
