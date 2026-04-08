# cognitive3d-public-api Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides expert guidance for the [Cognitive3D](https://cognitive3d.com) REST API — an XR/VR/AR analytics platform.

## What it does

Activates the `cognitive3d-public-api` skill, which helps Claude:

- Choose the right endpoint for any C3D data query
- Construct slicer queries with session filters, event filters, user filters, and aggregations
- Parse and interpret API responses (gaze metrics, session properties, objectives, ExitPoll)
- Build Python/JS/C# scripts for data pipelines and dashboards

## Installation

### From the Cognitive3D Public Marketplace (Recommended)

```
/plugin marketplace add CognitiveVR/c3d-marketplace-public
/plugin install cognitive3d-public-api@c3d-marketplace-public
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
claude --plugin-dir ./plugins/cognitive3d-public-api
```

## Usage

The skill activates automatically when you ask about Cognitive3D data. You can also invoke it directly:

```
/cognitive3d-public-api
```

## Bundled References

| File | Contents |
|------|----------|
| `references/endpoints.md` | Full endpoint catalog with real response examples |
| `references/slicer_query_doc.md` | Complete slicer filter and aggregation syntax |
| `references/slicer_api_guide.md` | Slicer API usage guide |
| `references/slicer_fields.yaml` | Slicer field definitions |
| `references/output_types.md` | Output type reference |
| `references/endpoint-coverage.md` | Coverage table — which endpoints have example responses |
| `references/code-examples.md` | Python, JavaScript, and C# request examples |

## Prerequisites

You'll need a Cognitive3D account and an API key. Keys are organization-scoped and issued from the [Cognitive3D Dashboard](https://app.cognitive3d.com).

## Contributing

Found a missing endpoint or incorrect example? Pull requests and issues are welcome.

## License

MIT
