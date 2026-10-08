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

| Plugin                               | Description                                                                                                                                                    |
|--------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **cognitive3d-public-api**           | Expert guide for the Cognitive3D REST API — constructing requests, choosing endpoints, building slicer queries, and parsing responses for XR session analytics |
| **cognitive3d-sdk-implementation**   | SDK implementation strategy for Unity, Unreal, visionOS, Android XR and WebXR — guides SDK identification, discovery, data strategy, phased instrumentation, and technical routing |

## Usage

Once installed, skills activate automatically when you ask Claude about relevant topics. You can also invoke them directly:

```
/cognitive3d-public-api
/cognitive3d-sdk-implementation
```

### What the API skill helps with

- Choosing the right endpoint for any C3D data query (sessions, gaze, events, sensors, objectives, ExitPoll)
- Constructing slicer queries with session filters, event filters, aggregations, and slice-bys
- Parsing and interpreting API responses
- Building Python, JavaScript, and C# scripts for data pipelines and dashboards
- Looking up authentication, field names, and property paths

### What the SDK implementation skill helps with

- Identifying which SDK the project targets (Unity, Unreal, native visionOS, native Android XR, or WebXR) and loading only that reference
- Discovery — asking the right questions before recommending instrumentation
- Classifying projects by business motion and archetype
- Building phased tracking plans (custom events, dynamic objects, exit polls, session properties)
- Screening plans against what the target SDK can actually do
- Routing to the correct SDK APIs and dashboard docs

### Migrating from cognitive3d-unity-implementation

`cognitive3d-unity-implementation` has been replaced by `cognitive3d-sdk-implementation`. All of the Unity guidance carries over; the new plugin adds Unreal, visionOS, Android XR and WebXR alongside it.

If you had the Unity plugin installed, run these two commands in a Claude Code session:

```
/plugin marketplace update c3d-marketplace-public
/plugin install cognitive3d-sdk-implementation@c3d-marketplace-public
```

Claude Code migrates your settings to the new name automatically after the first command and shows a one-time "Renamed to cognitive3d-sdk-implementation" notice. Until you run the second command, sessions report `Plugin "cognitive3d-sdk-implementation" not cached` — that's the cue to install it. No uninstall is needed.

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
