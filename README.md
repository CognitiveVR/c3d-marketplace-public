# c3d-marketplace-public

Open-source [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugins from [Cognitive3D](https://cognitive3d.com) — the XR/VR/AR analytics platform.

This marketplace provides skills that help Claude work with Cognitive3D APIs and data. Install a plugin and Claude gains deep knowledge of endpoint schemas, query construction, and response parsing — no manual lookup required.

## Getting Started

Add the marketplace to Claude Code:

```
/plugin marketplace add CognitiveVR/c3d-marketplace-public
```

Then install the plugin you want:

```
/plugin install cognitive3d-public-api@c3d-marketplace-public
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| **cognitive3d-public-api** | Expert guide for the Cognitive3D REST API — constructing requests, choosing endpoints, building slicer queries, and parsing responses for XR session analytics |

## Usage

Once installed, the skill activates automatically when you ask Claude about Cognitive3D data. You can also invoke it directly:

```
/cognitive3d-public-api
```

### What the API skill helps with

- Choosing the right endpoint for any C3D data query (sessions, gaze, events, sensors, objectives, ExitPoll)
- Constructing slicer queries with session filters, event filters, aggregations, and slice-bys
- Parsing and interpreting API responses
- Building Python, JavaScript, and C# scripts for data pipelines and dashboards
- Looking up authentication, field names, and property paths

## Prerequisites

You'll need a Cognitive3D account and an API key. Keys are organization-scoped and issued from the [Cognitive3D Dashboard](https://app.cognitive3d.com). See the plugin's bundled reference docs for authentication details.

## Directory Structure

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json        # Plugin metadata
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md        # Skill definition
│       └── references/     # Bundled reference docs
└── README.md               # Plugin documentation
```

## Contributing

Contributions are welcome! To suggest improvements to an existing plugin or propose a new one, open an issue or pull request.

## License

MIT
