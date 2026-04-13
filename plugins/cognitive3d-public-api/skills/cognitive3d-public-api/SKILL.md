---
name: cognitive3d-public-api
description: >
  Expert guide for the Cognitive3D (C3D) REST API — constructing requests, suggesting the right
  endpoints, and parsing responses. Use this skill whenever the user wants to query Cognitive3D
  session data, gaze or fixation tracking, sensor readings, dynamic object data, objectives,
  ExitPoll survey results, or wants to build a dashboard or data pipeline from the C3D backend.
  Also trigger for questions about C3D authentication, API key setup, slicer/filter query
  construction, or any request mentioning "cognitive3d", "c3d", "c3ddev", or XR/VR analytics
  data retrieval. If someone is exploring what data is available from a VR session analytics
  platform and it looks like C3D, use this skill without waiting to be asked.
tags:
  - analytics
  - data
  - query
  - api
  - xr
  - vr
  - gaze
  - sessions
  - cognitive3d
  - c3d
---

# Cognitive3D Public API Skill

Cognitive3D is an XR (VR/AR/MR) analytics platform. Its backend stores session recordings —
gaze, movement, events, sensor data, object interactions — that you can query to build dashboards,
reports, and data pipelines.

## Environment & Base URL

| Environment | Base URL |
|-------------|----------|
| Production  | `https://api.cognitive3d.com` |
| Development | `https://api.c3ddev.com` |

All endpoints are under `/v0/`.

---

## Authentication

Every request needs an `Authorization` header.

| Key format | Header value |
|------------|--------------|
| New (issued after May 2025) | `Authorization: orgkey-{your_key}` |
| Legacy (issued before May 2025) | `Authorization: APIKEY:ORGANIZATION {your_key}` |

Keys are organization-scoped — one key unlocks all projects, scenes, and sessions in that org.
The `Authorization` header alone is sufficient for all API calls, including write operations.

### Using a curl config file (recommended)

To avoid exposing API keys in commands, **before making any API calls**, check if a `c3d_curlrc` file exists in the current working directory.

**If `./c3d_curlrc` exists:**
Use it with all curl commands:
```bash
curl --config ./c3d_curlrc https://api.cognitive3d.com/v0/...
```

**If `./c3d_curlrc` does NOT exist:**
Instruct the user to create it before proceeding. They should create a file named `c3d_curlrc` in their working directory with the following content:

```
# Cognitive3D API credentials
header = "Authorization: orgkey-YOUR_API_KEY_HERE"
```

This keeps the API key out of command history and console output. If you ask the user for the API key yourself and add it to curl commands the normal way as a header, it will be visible in console output from the user.

Note well that there cannot be decorative presentation content in this file. A tendency for agents giving this instruction is to decorate it with a bar or header text that is not behind a hashbang #. Make sure that any header text OR other text is behind a hashbang when instructing the user of how to construct this file.

---

## Key IDs to Know

Before constructing any request, identify which IDs you're working with:

| ID | Type | Notes |
|----|------|-------|
| `projectId` | integer | Found in project URLs and responses |
| `sceneId` | UUID string | Identifies a scene (environment) |
| `versionId` | integer | A specific published version of a scene |
| `sessionId` | SDK string | The ID assigned by the C3D SDK at session creation |
| `objectId` / `sdkId` | string | Dynamic object identifier — use `sdkId` field (not `id`) in gaze queries |

**Discovering your organization ID:**
If you don't know your organization or project IDs, call:
```
GET /v0/organizations/apiKeys/whoami
```
This returns just the organization ID of the API key being used, which you can use to fetch project data as well via the other endpoints.

---

## Choosing the Right Endpoint

Use this decision tree when the user describes what they want:

**"List / browse sessions"**
→ `POST /v0/datasets/sessions/paginatedListQueries` (paginated, with filters)

**"All data for one specific session"**
→ `POST /v0/datasets/sessions/singleQueries` (scene-scoped)
→ `POST /v0/datasets/sessions/singleProjectSessionQueries` (project-scoped)

**"Raw gaze / fixation / event / sensor / position data for a session"**
→ `POST /v0/projects/:projectId/sessions/:sessionId/jsonRequests` with `jsonType`
  Values: `GAZE`, `FIXATION`, `EVENTS`, `DYNAMICS`, `SENSORS`, `BOUNDARY`, `ALL`

**"Aggregated gaze stats across many sessions (how long did people look at object X?)"**
→ `POST /v0/datasets/sessions/slicerObjectMetricQueries`

**"Per-session gaze breakdown for a single object"**
→ `POST /v0/datasets/sessions/slicerSingleObjectPerSessionMetricQueries`

**"Analytics query / session counts / aggregations with filters"**
→ `POST /v0/datasets/sessions/slicerQueries`
→ See `references/slicer_query_doc.md` for the full filter/aggregation syntax

**"What filterable properties exist for this project?"**
→ `POST /v0/datasets/sessions/slicerPropertyNameQueries`
  Body: `{ "entityFilters": { "projectId": <int> } }`
  Returns lists of textual, numerical, and boolean session/event properties

**"Project info + scene list"**
→ `GET /v0/projects/:projectId`

**"Training outcomes / pass-fail for a scenario"**
→ `GET /v0/versions/:versionId/sessions/:sessionId/objectiveData` (per-session step results)
→ `GET /v0/projects/:projectId/objectiveVersions/:objectiveVersionId/results.csv` (bulk export)

**"Survey responses (ExitPoll)"**
→ `GET /v0/projects/:projectId/questionSets/:name/:version/responses`

**Full endpoint catalog** → `references/endpoints.md`

---

## Common Request Patterns

### Paginated session list
```
POST /v0/datasets/sessions/paginatedListQueries
{
  "entityFilters": {
    "projectId": "<int>",
    "sceneId": "<uuid>",    // optional
    "versionId": "<int>"    // optional
  },
  "page": 0,
  "limit": 20,
  "sort": "desc",
  "orderBy": { "fieldName": "date", "fieldParent": "session" }
}
```

### Raw data for a session
```
POST /v0/projects/:projectId/sessions/:sessionId/jsonRequests
{
  "sessionId": "<sdkSessionId>",
  "jsonType": "GAZE"   // or FIXATION, EVENTS, DYNAMICS, SENSORS, BOUNDARY, ALL
}
```

### Aggregated gaze on dynamic objects
```
POST /v0/datasets/sessions/slicerObjectMetricQueries
{
  "entityFilters": { "projectId": <int>, "sceneId": "<uuid>", "versionId": <int> },
  "gazeType": "gaze",       // or "fixation"
  "aggregations": "all",
  "objectIds": ["<sdkId_1>", "<sdkId_2>"]   // sdkId is a UUID string like "c34cbd8b-4fb9-..."
}
```

**Object lookup:** `objectIds` takes `sdkId` values (UUID strings), not friendly names. To find the sdkId for a named object, call:
```
GET /v0/versions/:versionId/objects
```
Response array has `name` (friendly) and `sdkId` (UUID) fields. Match on `name`, use `sdkId` in gaze queries.

---

## Critical Gotchas

1. **Timestamps and durations are milliseconds** — `date` field values are Unix ms; `duration` field is also ms (60 seconds = 60000)
2. **Use `sdkId` (UUID string) for objects in gaze queries**, not the integer `id` field. Users often refer to objects by friendly name — look up the sdkId first via `GET /v0/versions/:versionId/objects`
3. **Test sessions are noise** — nearly every analytics query should exclude them:
   ```json
   { "field": { "fieldParent": "session", "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.test" }, "op": "eq", "value": false }
   ```
4. **`sessionType`** — set to `"project"` for project-wide queries, `"scene"` when you have a sceneId
5. **The slicer filter schema is non-obvious** — nested `fieldParent`/`nestedFieldName`/`path` structure for session properties vs flat `fieldName`/`fieldParent` for built-in fields. See the Slicer Query System section below.
6. **Never guess property names** — property paths (e.g. `c3d.app.version`, `c3d.session_tag.test`) are not intuitive enough to infer. Before using a property in a query, either look it up in `references/slicer_fields.yaml` or discover it via `POST /v0/datasets/sessions/slicerPropertyNameQueries` (see `references/slicer_api_guide.md`). The built-in field names listed in this file (like `date`, `duration`, `sessionId`) are safe to use directly — it's the property `path` values that must be verified.

7. **Always make HTTP requests sequentially, never in parallel** — When running queries across multiple projects, sessions, or endpoints, execute requests one at a time. Do not fire parallel requests even if asked to query "all projects" or collate data across many resources. This avoids rate limiting and ensures predictable behavior.

---

## Response Parsing Tips

- **Session list response**: array of session objects with `sessionId`, `date`, `duration`, `userId`, `properties` (nested object of all session properties)
- **jsonRequests ALL response**: `{ "data": { "gaze": [...], "fixations": [...], "events": [...], "dynamics": [...], "dynamics.manifest": {...}, "sensors": [...], "boundary": [...] } }`
- **Slicer query response**: nested under `aggregations → <name> → <op_name> → value`. Example for a `sessionCount` op named `session_count` under aggregation `main`: `response["aggregations"]["main"]["session_count"]["value"]`
- **Gaze metric response**: `{ "sessionCount": N, "metrics": { "<sdkId>": { "averageGazeLength": ..., "totalGazeCount": ..., ... } } }`

---

## Code Examples

For Python, JavaScript, and C# request examples → `references/code-examples.md`

## Full Endpoint Catalog

For all endpoints organized by domain → `references/endpoints.md`

For a coverage table showing which endpoints have example responses → `references/endpoint-coverage.md`
When adding or updating any endpoint or example response in `references/endpoints.md`, update this table to match.

## Slicer Query System

The slicer is the main analytics query engine. Endpoint: `POST /v0/datasets/sessions/slicerQueries`

### Minimal request

```json
{
  "sessionType": "project",
  "entityFilters": { "projectId": 541 },
  "sessionFilters": [],
  "aggregations": [
    {
      "name": "main",
      "operations": [
        { "name": "session_count", "type": "sessionCount" }
      ]
    }
  ]
}
```

Always include `"sessionType": "project"` — the default is scene sessions which is almost never what you want.

### Field references

There are two patterns depending on whether you're referencing a built-in field or a property:

**Built-in fields** (top-level fields like date, duration, sessionId):
```json
{ "fieldName": "date", "fieldParent": "session" }
```

**Properties** (free-form key/value pairs that vary per project):
```json
{ "nestedFieldName": "textualSessionProp", "fieldParent": "session", "path": "c3d.app.version" }
```

**Always use `nestedFieldName` for properties.** Some commonly indexed properties also support `unnestedFieldName`, which is faster — but only if the property is marked `unnested: true` in `references/slicer_fields.yaml`. If you use `unnestedFieldName` on a property that isn't unnested, the query will silently return wrong results. When in doubt, `nestedFieldName` always works.

The six property field names are:

| nestedFieldName        | Type    | Parent    |
|------------------------|---------|-----------|
| `textualSessionProp`   | string  | `session` |
| `numericalSessionProp` | number  | `session` |
| `booleanSessionProp`   | boolean | `session` |
| `textualEventProp`     | string  | `event`   |
| `numericalEventProp`   | number  | `event`   |
| `booleanEventProp`     | boolean | `event`   |

### Session filters

`sessionFilters` is an array of filter objects, ANDed together. Each filter has a field reference, an `op`, and a `value`:

```json
{ "field": { "fieldName": "date", "fieldParent": "session" }, "op": "gte", "value": 1747094400000 }
```

Ops: `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `in` (use `values` array), `notIn` (use `values` array), `range`/`outsideRange` (use `min`/`max`).

Compound filters use `"op": "and"` / `"or"` / `"none"` with a `children` array:
```json
{
  "op": "and",
  "children": [
    { "op": "eq", "field": { "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.test" }, "value": false },
    { "op": "eq", "field": { "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.junk" }, "value": false }
  ]
}
```

### Aggregation operations

| Type           | Requires Field? | Notes                             |
|----------------|-----------------|-----------------------------------|
| `sessionCount` | No              | Count sessions                    |
| `eventCount`   | No              | Count events                      |
| `min`          | Yes             | Requires numerical/integral field |
| `max`          | Yes             | Requires numerical/integral field |
| `average`      | Yes             | Requires numerical/integral field |
| `sum`          | Yes             | Requires numerical/integral field |

### Slice-bys (dimensions)

Operations produce a single number by default. Add `sliceBys` at the aggregation level (not inside the operation) to bucket the data. See `references/slicer_query_doc.md` for full syntax and examples.

**Date histogram** (data over time):
```json
{ "name": "by_week", "timeUnit": "week", "timeValue": 1, "field": { "fieldName": "date", "fieldParent": "session" } }
```

**Discrete field** (split by values of a field):
```json
{ "name": "by_version", "maxTerms": 64, "field": { "unnestedFieldName": "textualSessionProp", "path": "c3d.app.version" } }
```

### When to load reference files

For **complex queries** (objective filters, user segmentation, event filters, compound filter logic, custom groups, histogram slice-bys) → `references/slicer_query_doc.md`

For **discovering what fields/properties/values exist** for a project before building a query → `references/slicer_api_guide.md`

For **looking up a specific field name**, property path, type, or display metadata → `references/slicer_fields.yaml`

For **choosing or parsing output formats** (legacy, json0_keyed, json0_list, json0_key_y) at different dimensionalities → `references/output_types.md`
