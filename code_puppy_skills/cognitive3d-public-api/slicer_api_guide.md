# Slicer API Workflow Guide

This guide is for AI agents or anyone else building a program that calls the slicer API directly. It covers the discovery routes used to learn what fields, properties, and values are available before building queries.

For the slicer query schema itself (aggregations, filters, field references, output types), see [`slicer_query_doc.md`](slicer_query_doc.md). For common field metadata (descriptions, units, display names), see [`slicer_fields.yaml`](slicer_fields.yaml). But bear in mind that custom properties unique to your project are not listed there, and must be discovered via the process described in this document. 

## Discovery Routes

Before building a slicer query, a program typically needs to learn what fields, properties, and values are available. Three routes support this. The first is static and the latter two are dynamic per-project.

### Schema Metadata: `GET /v0/datasets/sessions/slicerOptions`
This static endpoint returns metadata about available operation types, value types, and built-in fields. No request body is needed. The response includes `valueTypes`, `aggregationOps`, `sessionFields`, and `eventFields`. These are documented inline in the schema reference doc under the [Glossary](slicer_query_doc.md#glossary-common-concepts) section.

At this time, this endpoint does not cover newer provisional features like user filters or objective filters — for those, refer to the schema doc directly.

### Discovering Properties: `POST /v0/datasets/sessions/slicerPropertyNameQueries`
Properties are free-form key/value pairs that vary per project, SDK version, and application. To discover what properties exist, call this route.

**Request:** Requires `entityFilters` with at minimum a `projectId`. Supports all the same optional filters as slicer queries (`sessionFilters`, `eventFilters`, `userFilters`, `objectiveFilters`) to narrow down which sessions are considered. For example, you can pass a date filter to see only properties from recent sessions, or filter by app version to see properties from a specific build.

```json
{
  "sessionType": "project",
  "entityFilters": { "projectId": 541 }
}
```

**Response:** An object keyed by the six property field names, each containing an array of property name strings:
```json
{
  "textualSessionProp": ["c3d.app.version", "c3d.device.type", ...],
  "numericalSessionProp": ["c3d.metrics.average_fps", ...],
  "booleanSessionProp": ["c3d.session_tag.test", "c3d.session_tag.junk", ...],
  "textualEventProp": ["Reason", "description", ...],
  "numericalEventProp": ["duration", "timestamp", ...],
  "booleanEventProp": []
}
```

These keys correspond directly to the `nestedFieldName` (or `unnestedFieldName`) you use in a [Field Reference](slicer_query_doc.md#Field-Reference) when building filters, operations, or slice-bys. For example, seeing `"c3d.app.version"` under `textualSessionProp` tells you to reference it as:
```json
{ "nestedFieldName": "textualSessionProp", "path": "c3d.app.version" }
```

Some common properties sent by the Cognitive3D SDK that you'll encounter across most projects:

| Property Name             | Field Name             | Description                     |
|---------------------------|------------------------|---------------------------------|
| `c3d.app.version`        | `textualSessionProp`   | Application version string      |
| `c3d.device.type`        | `textualSessionProp`   | Device type (e.g. VR headset)   |
| `c3d.device.hmd.type`    | `textualSessionProp`   | HMD model name                  |
| `c3d.geo.country`        | `textualSessionProp`   | Country name from geo lookup    |
| `c3d.geo.countryIso`     | `textualSessionProp`   | Country ISO code                |
| `c3d.device.os`          | `textualSessionProp`   | Operating system                |
| `c3d.session_tag.test`   | `booleanSessionProp`   | Whether session is tagged test  |
| `c3d.session_tag.junk`   | `booleanSessionProp`   | Whether session is tagged junk  |
| `c3d.metrics.average_fps`| `numericalSessionProp` | Average FPS for the session     |
| `c3d.device.memory`      | `numericalSessionProp` | Device memory                   |

For the full list of common SDK properties with descriptions, units, and display metadata, see [`slicer_fields.yaml`](slicer_fields.yaml).

### Discovering Field Values: `POST /v0/datasets/sessions/slicerFieldValuesQueries`
Once you know what fields and properties exist, you may need to discover the actual values a field has — for populating dropdowns, validating filter values, or understanding the data.

**Request:** Requires `entityFilters` and a `field` (a [Field Reference](slicer_query_doc.md#Field-Reference)). Supports all the same optional filters as slicer queries. Optionally set `maxTerms` (default 256) to control how many distinct values are returned.

```json
{
  "sessionType": "project",
  "entityFilters": { "projectId": 541 },
  "field": {
    "unnestedFieldName": "textualSessionProp",
    "path": "c3d.app.version"
  }
}
```

**Response:** A JSON array of distinct values:
```json
["0.6", "1.0", "0.9.1", "0.8.1", "0.9", "0.8", "0.7", ...]
```

This is useful for any discrete field. Some common examples:
- App versions: `field: { "unnestedFieldName": "textualSessionProp", "path": "c3d.app.version" }`
- Country codes: `field: { "unnestedFieldName": "textualSessionProp", "path": "c3d.geo.countryIso" }`
- Event names: `field: { "fieldName": "eventName", "fieldParent": "event" }`
- Device types: `field: { "unnestedFieldName": "textualSessionProp", "path": "c3d.device.type" }`

## Putting the Workflow Together
A program building slicer queries would typically:
1. **Learn the schema** — read the [schema doc](slicer_query_doc.md) or call `GET /v0/datasets/sessions/slicerOptions` to understand what operation types, value types, and built-in fields exist
2. **Discover properties** — call `POST /v0/datasets/sessions/slicerPropertyNameQueries` for the target project to find what custom properties are available
3. **Discover values** — call `POST /v0/datasets/sessions/slicerFieldValuesQueries` for specific fields of interest to see what values exist (e.g. what app versions are in use)
4. **Build and execute** — construct a `POST /v0/datasets/sessions/slicerQueries` request using the fields, properties, and values discovered above
