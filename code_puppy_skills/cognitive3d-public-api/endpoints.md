# Cognitive3D API — Endpoint Catalog

Base URL: `https://api.cognitive3d.com` (prod) or `https://api.c3ddev.com` (dev)
All paths are under `/v0/`.

---

## Sessions

### List Sessions (paginated)
```
POST /v0/datasets/sessions/paginatedListQueries
```
Body:
```json
{
  "entityFilters": {
    "projectId": "<int>",
    "sceneId": "<uuid>",     // optional
    "versionId": "<int>"     // optional
  },
  "page": 0,
  "limit": 20,
  "sort": "desc",
  "orderBy": { "fieldName": "date", "fieldParent": "session" },
  "sessionFilters": []       // optional — same filter syntax as slicerQueries
}
```

Response:
```json
{
  "count": 3,
  "pages": 1,
  "currentPage": 0,
  "results": [
    {
      "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
      "sdkSessionId": "1773998824_6826c6d546130967474184c118bfbc90",
      "projectId": 341,
      "organizationId": 12,
      "participantId": "00130",
      "friendlyName": "Marco Aiello",
      "date": "2026-03-20T09:27:04.802Z",
      "endDate": "2026-03-20T09:28:25.082Z",
      "duration": 80280,
      "createdAt": 1773998824802,
      "startTime": 1773998824802,
      "gazeInterval": 0.1,
      "hmd": null,
      "user": "6826c6d546130967474184c118bfbc90",
      "deviceId": "6826c6d546130967474184c118bfbc90",
      "userKey": "00130",
      "hasGaze": true,
      "hasFixation": false,
      "hasEvent": true,
      "hasDynamic": true,
      "hasSensor": true,
      "hasBoundary": true,
      "positionLimited": false,
      "gazeLimited": false,
      "fixationsLimited": false,
      "eventsLimited": false,
      "tags": [],
      "properties": {
        "c3d.participant.name": "Marco Aiello",
        "c3d.participant.id": "00130",
        "c3d.participant.armlength": 69.94972229003906,
        "c3d.participant.height": 11,
        "c3d.participant.exitpoll.ExperienceLevel": "Very experienced",
        "c3d.app.name": "trainingdemo",
        "c3d.app.version": "2.0",
        "c3d.app.engine": "Unity",
        "c3d.app.engine.version": "6000.0.28f1",
        "c3d.app.sdktype": "Default",
        "c3d.app.xrplugin": "OpenXRLoader",
        "c3d.app.androidPlugin.version": "2.6.0",
        "c3d.app.handtracking.enabled": true,
        "c3d.app.inEditor": false,
        "c3d.version": "1.8.0",
        "c3d.device.type": "Mobile",
        "c3d.device.model": "Oculus Quest",
        "c3d.device.os": "Android OS 14",
        "c3d.device.memory": 12,
        "c3d.device.gpu": "Adreno (TM) 650",
        "c3d.device.gpu.vendor": "Unknown",
        "c3d.device.cpu": "ARM64 FP ASIMD AES",
        "c3d.device.cpu.vendor": "ARM",
        "c3d.device.hmd.type": "Head Tracking - OpenXR",
        "c3d.device.eyetracking.enabled": false,
        "c3d.device.controllerinputs.enabled": true,
        "c3d.deviceid": "6826c6d546130967474184c118bfbc90",
        "c3d.sessionname": "Marco Aiello",
        "c3d.roomsizeMeters": 1.4450000524520874,
        "c3d.roomsizeDescriptionMeters": "1.2 x 1.2",
        "c3d.geo.country": "United States",
        "c3d.geo.countryIso": "US",
        "c3d.geo.subdivision": "Oregon",
        "c3d.geo.subdivisionIso": "OR",
        "c3d.geo.qualifiedSubdivisionIso": "US-OR",
        "c3d.geo.city": "Portland",
        "c3d.geo.latitude": 45.52349853515625,
        "c3d.geo.longitude": -122.6760025024414,
        "c3d.session_tag.test": false,
        "c3d.session_tag.junk": false,
        "c3d.metrics.immersion_score": 86.79363108433414,
        "c3d.metrics.comfort_score": 67.56187960096001,
        "c3d.metrics.presence_score": 64.22733003068618,
        "c3d.metrics.fps_score": 37.466862644802184,
        "c3d.metrics.average_fps": 45.39032075279638,
        "c3d.metrics.ergonomics_score": 54.666666666666664,
        "c3d.metrics.cyberwellness_score": 30.522108367717028,
        "c3d.metrics.battery_efficiency": 90.01381422365728,
        "c3d.metrics.boundary_score": 90,
        "c3d.metrics.app_performance": 50,
        "c3d.metrics.dynamic_engagement_score": 100,
        "c3d.metrics.controller_engagement_score": 97.79257958507792,
        "c3d.metrics.controller_events_score": 0,
        "c3d.metrics.standing_percentage": 0.4095449500554939,
        "c3d.metrics.orientation_score": 52.54237288135593,
        "c3d.session.sensor.HMD Battery Level.avg": 98.714285714286,
        "c3d.session.sensor.HMD Battery Level.min": 98,
        "c3d.session.sensor.HMD Battery Level.max": 99,
        "c3d.session.sensor.c3d.fps.avg.avg": 45.488466510525,
        "c3d.session.sensor.c3d.fps.avg.min": 23.989830017089844,
        "c3d.session.sensor.c3d.fps.avg.max": 72.05615234375,
        "c3d.session.sensor.c3d.hmd.pitch.avg": -13.5276785900769,
        "c3d.session.sensor.c3d.app.WifiRSSI.avg": -38.478260869565,
        "c3d.debug.info.merge_timestamp_millis": 1773997302636
      }
    }
    // ... additional sessions follow the same structure
  ]
}
```

Key fields to know:
- `count` / `pages` / `currentPage` — pagination info
- `duration` — session length in **milliseconds**
- `date` / `endDate` — ISO 8601 strings; `createdAt` / `startTime` — same moment as Unix ms
- `sessionId` and `sdkSessionId` are the same value — use either when referencing this session in other endpoints
- `hasGaze`, `hasFixation`, `hasDynamic`, etc. — flags indicating which data types were recorded
- `friendlyName` — display name (often same as `c3d.sessionname` / `c3d.participant.name`)
- `tags` — array of tag objects (empty if none applied)
- `properties` — flat key/value map of all session properties. Property name prefixes:
  - `c3d.participant.*` — participant metadata
  - `c3d.device.*` — hardware info as reported by the SDK (free-form strings)
  - `c3d.device.derived.*` — canonical device classification added by backend enrichment: `category`, `family`, `model_family`, `model`, `runtime_host` (lowercase snake_case slugs from a versioned taxonomy — see `slicer_fields.yaml` for the full vocabulary), plus `taxonomy_version`. Absent on sessions recorded before the enrichment pipeline was deployed.
  - `c3d.app.*` — app/SDK info
  - `c3d.geo.*` — geolocation
  - `c3d.metrics.*` — computed XR wellness/performance scores (floats, 0–100 scale)
  - `c3d.metric_components.*` — sub-scores feeding into the metrics above
  - `c3d.session.sensor.<SensorName>.<stat>` — per-session sensor aggregates (avg/min/max)
  - `c3d.session_tag.*` — boolean tag values (e.g. `c3d.session_tag.test`)
  - `c3d.debug.*` — internal debug fields

### Single Session — Scene-scoped
```
POST /v0/datasets/sessions/singleQueries
{
  "projectId": <int>,
  "sceneId": "<uuid>",
  "sessionId": "<sdkSessionId>",
  "versionId": <int>
}
```

Response:
```json
{
  "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
  "sceneVersionId": 141,
  "sceneId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
  "projectId": 341,
  "organizationId": 12,
  "participantId": "00130",
  "date": "2026-03-20T09:27:04.802Z",
  "endDate": "2026-03-20T09:28:25.082Z",
  "duration": 80280,
  "gazeInterval": 0.1,
  "hmd": null,
  "user": "6826c6d546130967474184c118bfbc90",
  "deviceId": "6826c6d546130967474184c118bfbc90",
  "userKey": "00130",
  "hasGaze": true,
  "hasFixation": false,
  "hasEvent": true,
  "hasDynamic": true,
  "hasSensor": true,
  "hasBoundary": true,
  "events": [
    {
      "x": -4.098360061645508,
      "y": 1.6461700201034546,
      "z": -6.356510162353516,
      "name": "c3d.sessionStart",
      "date": "2026-03-20T09:27:04.802Z",
      "object": null,
      "properties": {},
      "parentSceneVersionId": 141,
      "eventHash": 4387432117564403356
    },
    {
      "x": -4.065730094909668,
      "y": 1.6614999771118164,
      "z": -6.447390079498291,
      "name": "cvr.exitpoll",
      "date": "2026-03-20T09:27:16.504Z",
      "object": null,
      "properties": {
        "hook": "assessment_begin",
        "participantId": "00130",
        "questionSetId": "assessment_begin_survey:2",
        "sceneId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
        "Answer0": 1,
        "Answer1": 0,
        "Answer2": 8,
        "duration": 11.240059852600098
      },
      "parentSceneVersionId": 141,
      "eventHash": 841510903072697994
    },
    {
      "x": -4.116869926452637,
      "y": 1.5934300422668457,
      "z": -5.6615800857543945,
      "name": "Equip Hard Hat",
      "date": "2026-03-20T09:27:20.221Z",
      "object": "editor_01331adb-a08a-4fbd-99c1-93ac4b18cd93",
      "properties": {},
      "parentSceneVersionId": 141,
      "eventHash": 4868412283255767355
    },
    {
      "x": 0,
      "y": 0,
      "z": 0,
      "name": "c3d.sessionEnd",
      "date": "2026-03-20T09:28:24.172Z",
      "object": null,
      "properties": {
        "Reason": "Signaled (Quit from within app)",
        "ExitCode": 2,
        "sessionlength": 80.3030014038086
      },
      "parentSceneVersionId": 141,
      "eventHash": 7681580687772162571
    }
    // ... additional events
  ],
  "warnings": {
    "positionLimited": false,
    "gazeLimited": false,
    "fixationsLimited": false,
    "eventsLimited": false
  },
  "properties": {
    "c3d.participant.name": "Marco Aiello",
    "c3d.participant.id": "00130",
    "c3d.app.name": "trainingdemo",
    "c3d.device.model": "Oculus Quest",
    "c3d.metrics.immersion_score": 86.79363108433414
    // ... full properties object (same shape as paginatedListQueries)
  },
  "subscriptions": [],
  "objectiveResults": {},
  "tags": [],
  "friendlyName": "Marco Aiello"
}
```

Key differences from `paginatedListQueries`:
- Includes `events` array with full event records including position (x/y/z), name, properties, and `eventHash`
- ExitPoll survey answers appear as `cvr.exitpoll` events with `hook`, `questionSetId`, and `Answer0`/`Answer1`/... properties
- Includes `warnings` object (same as the `*Limited` flags on list responses)
- Includes `objectiveResults` map (empty `{}` if no objectives recorded)
- Includes `sceneVersionId` identifying which scene version the session belongs to

### Single Session — Project-scoped
```
POST /v0/datasets/sessions/singleProjectSessionQueries
{
  "projectId": "<string>",
  "sessionId": "<string>",
  "attachSceneSessionData": true,
  "includeObjectiveData": true
}
```

Response (same top-level fields as scene-scoped, but `properties` is replaced by `sceneSessionProperties`):
```json
{
  "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
  "projectId": 341,
  "organizationId": 12,
  "participantId": "00130",
  "date": "2026-03-20T09:27:04.802Z",
  "endDate": "2026-03-20T09:28:25.082Z",
  "duration": 80280,
  "hasGaze": true,
  "hasFixation": false,
  "hasEvent": true,
  "hasDynamic": true,
  "hasSensor": true,
  "hasBoundary": true,
  "sceneSessionProperties": {
    "141": {
      "c3d.participant.name": "Marco Aiello",
      "c3d.app.name": "trainingdemo",
      "c3d.device.model": "Oculus Quest",
      "c3d.metrics.immersion_score": 86.79363108433414
      // ... full properties for this scene version
    }
  },
  "tags": [],
  "friendlyName": "Marco Aiello"
}
```

Key difference: `sceneSessionProperties` is a map keyed by `versionId` (as a string), allowing a single session that spans multiple scenes to carry properties for each. Use this endpoint when you don't know the sceneId/versionId upfront.

Note: ExitPoll responses are included in the `events` array of these responses.

### Raw Session Data (JSON)
```
POST /v0/projects/:projectId/sessions/:sessionId/jsonRequests
{
  "sessionId": "<sdkSessionId>",
  "jsonType": "GAZE" | "FIXATION" | "EVENTS" | "DYNAMICS" | "SENSORS" | "BOUNDARY" | "ALL"
}
```
Optional body fields:
- `"addGeo": true` — include geolocation
- `"addMetadata": true` — include session metadata
- `"addProperties": true` — include session properties
- `"separateSceneData": false` — true = split output by versionId
- `"indicateSceneOnDatum": false` — true = add versionId to each data point
- `"sessionRelativeTimestamps": true` — timestamps relative to session start
- `"convertMillisToSeconds": true`
- `"sizeToConvertGapsToMillis": 5000` — gap threshold in ms

Scene-scoped variant:
```
POST /v0/versions/:sceneVersionId/sessions/:sessionId/jsonRequests
// Requires projectId, sceneId, sessionId, jsonType in body
```

### Raw Session Data (ZIP download)
```
GET /v0/projects/:projectId/sessions/:sessionId/jsonZipRequest
GET /v0/versions/:sceneVersionId/sessions/:sessionId/jsonZipRequest?projectId=...&sceneId=...
```
Downloads all jsonTypes as a zip archive.

### Session Metadata
```
GET  /v0/versions/:versionId/sessions/:sessionId/metadata
POST /v0/projects/:projectId/sessions/metadataLookups
     { "sdkIdSceneIds": [{ "sdkId": "<sdkSessionId>", "sceneId": "<uuid>" }] }
```

### Participant Sessions
```
POST /v0/datasets/sessions/participantSessionsQueries
{ "organizationId": "<string>", "participantId": "<string>" }
```

### Session Tags
```
PUT  /v0/projects/:projectId/sessions/:sessionId/tags/:tagName
     { "value": true }
GET  /v0/organizations/:organizationId/tags
POST /v0/organizations/:organizationId/tags
     { "title": "my-tag", "description": "...", "colourHex": "ffffff", "projectIds": ["14"] }
```
Built-in tags: `test`, `junk` — exclude them from analytics with `excludeTags=test,junk` or sessionFilters.

### Session Reports (PDF)
```
GET  /v0/projects/:projectId/sessions/:sessionId/report.pdf
GET  /v0/versions/:versionId/sessions/:sessionId/report.pdf?sceneId=...&projectId=...
POST /v0/versions/:versionId/sessionsReport.pdf?sceneId=...&projectId=...
     { "sessionIds": ["<id1>", "<id2>"] }
POST /v0/versions/:versionId/sessions/:sessionId/reportEmails
     { "sceneId": "...", "projectId": "...", "recipients": ["email@example.com"] }
```

---

## Analytics / Slicer

### Property Discovery
```
POST /v0/datasets/sessions/slicerPropertyNameQueries
{ "entityFilters": { "projectId": <int> } }
```
Returns: `{ textualSessionProp: [...], numericalSessionProp: [...], booleanSessionProp: [...], textualEventProp: [...], numericalEventProp: [...], booleanEventProp: [...] }`

### Analytics Query (aggregations + filters)
```
POST /v0/datasets/sessions/slicerQueries
```
See `slicer_query_doc.md` for full syntax.

Response (sessionCount example):
```json
{
  "aggregations": {
    "main": {
      "session_count": {
        "dimensionless": true,
        "label": "session_count",
        "value": 366
      }
    }
  }
}
```
Access the value with: `response["aggregations"]["<aggregation_name>"]["<operation_name>"]["value"]`

### Object Gaze — Aggregated (multiple objects, across sessions)
```
POST /v0/datasets/sessions/slicerObjectMetricQueries
{
  "entityFilters": { "projectId": <int>, "sceneId": "<uuid>", "versionId": <int> },
  "gazeType": "gaze",      // or "fixation"
  "aggregations": "all",
  "objectIds": ["<sdkId_uuid_1>", "<sdkId_uuid_2>"],   // sdkId is a UUID string, not integer id
  "sessionFilters": []     // optional — same filter syntax
}
```
Response:
```json
{
  "sessionCount": 1,
  "metrics": {
    "<objectSdkId>": {
      "averageGazeInstanceDuration": 120.1,
      "averageGazeCount": 1,
      "averageGazeLength": 120.1,
      "totalGazeCount": 1,
      "totalGazeLength": 120.1,
      "totalSessionsWithAnyGaze": 1,
      "proportionOfSessionsWithAnyGaze": 1,
      "averageTimeToFirstGaze": 765.618,
      "averageGazeSequence": 7
    }
  }
}
```

### Object Gaze — Per Session (single object)
```
POST /v0/datasets/sessions/slicerSingleObjectPerSessionMetricQueries
{
  "projectId": "<string>",
  "sceneId": "<uuid>",
  "versionId": "<string>",
  "gazeType": "gaze",
  "aggregations": "all",
  "objectId": "<sdkId>"
}
```
Response: `{ "metrics": { "<sessionId>": { "averageGazeInstanceDuration": ..., "totalGazeCount": ..., "timeToFirstGaze": ..., "sessionDate": <ms> } } }`

---

## Dynamic Objects

### List Objects by Scene Version
```
GET /v0/versions/:versionId/objects?includeFiles=false
```

Response (array of objects):
```json
[
  {
    "createdAt": 1679331338000,
    "updatedAt": 1679331533000,
    "id": 617,
    "versionId": 141,
    "name": "Instructions Intro",
    "meshName": "instructions_intro",
    "sdkId": "editor_2b28d9d1-7309-4ead-8762-9ad8e2bab7ad",
    "isController": null,
    "initialPositionX": null,
    "initialPositionY": null,
    "initialPositionZ": null,
    "initialRotationX": null,
    "initialRotationY": null,
    "initialRotationZ": null,
    "initialRotationW": null,
    "initialScaleX": null,
    "initialScaleY": null,
    "initialScaleZ": null,
    "scale": null,
    "scaleCustomX": null,
    "scaleCustomY": null,
    "scaleCustomZ": null,
    "dynamicFileType": "gltf",
    "group": null,
    "thumbnailLocation": "objects/instructions_intro/cvr_object_thumbnail.png",
    "externalId": null,
    "externalBaseUrl": null,
    "isOptimized": true
  },
  {
    "createdAt": 1679331338000,
    "updatedAt": 1752457942000,
    "id": 618,
    "versionId": 141,
    "name": "Breaker Main",
    "meshName": "breaker_main",
    "sdkId": "editor_651294a9-c704-46bd-b9dc-d6c87dcad02e",
    "isController": false,
    "initialPositionX": -0.517,
    "initialPositionY": 1.027,
    "initialPositionZ": 1.995,
    "initialRotationX": 0.0,
    "initialRotationY": 0.7071,
    "initialRotationZ": 0.0,
    "initialRotationW": 0.7071,
    "initialScaleX": null,
    "initialScaleY": null,
    "initialScaleZ": null,
    "scale": null,
    "scaleCustomX": 1.0,
    "scaleCustomY": 1.0,
    "scaleCustomZ": 1.0,
    "dynamicFileType": "gltf",
    "group": null,
    "thumbnailLocation": "objects/breaker_main/cvr_object_thumbnail.png",
    "externalId": null,
    "externalBaseUrl": null,
    "isOptimized": true
  }
  // ... additional objects
]
```

Use `sdkId` (not `id`) when referencing objects in gaze queries. `initialPosition*` and `initialRotation*` are null for objects that don't have a fixed spawn position in the scene.

### List Objects by Project
```
GET /v0/projects/:projectId/objects
```
Returns all dynamic objects across all versions in the project. Same shape as the version-scoped list, plus a `versionId` field on each item.
```json
[
  {
    "id": 617,
    "versionId": 141,
    "name": "Instructions Intro",
    "meshName": "instructions_intro",
    "sdkId": "editor_2b28d9d1-7309-4ead-8762-9ad8e2bab7ad",
    "dynamicFileType": "gltf",
    "isOptimized": true,
    "isController": null,
    "scale": null,
    "group": null,
    "externalId": null,
    "thumbnailLocation": "objects/instructions_intro/cvr_object_thumbnail.png"
  }
  // ... more objects
]
```

### Delete Dynamic Object
```
DELETE /v0/versions/:sceneVersionId/objects/:objectId?versionNumber=1&sdkSceneId=<uuid>
```

---

## Gaze & Fixations (Raw)

Use the JSON Request endpoints with the appropriate `jsonType`:
- `GAZE` — raw gaze ray samples
- `FIXATION` — computed fixation points (World, Object, Media fixation types)
- `ALL` — all data types in a single response

---

## Projects & Scenes

### Get Project (with scenes)
```
GET /v0/projects/:projectId?useImageLocations=true&attachStats=false
```

Response:
```json
{
  "createdAt": 1679331121000,
  "updatedAt": 1773175614000,
  "id": 341,
  "organizationId": 12,
  "name": "Sample Electrical Training Demo",
  "description": "",
  "projectType": "vr",
  "enableGeoIp": true,
  "enableAi": true,
  "hidden": false,
  "prefix": "auto_cid_Electrical_Demo_341",
  "imageLocation": null,
  "categoryGroup": {
    "id": 5,
    "name": "Training and Simulation"
  },
  "categories": [],
  "sessionTagTypes": [],
  "scenes": [
    {
      "id": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
      "sceneName": "factory scene",
      "projectId": 341,
      "sdkFacingId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
      "isPublic": false,
      "hidden": false,
      "sceneCreationMethod": "NORMAL",
      "latestScreenshotLocation": "versions/141/files/screenshot_small/screenshot.png",
      "versions": [
        {
          "id": 141,
          "sceneId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
          "versionNumber": 1,
          "scale": 1.0,
          "sdkVersion": "0.21.0",
          "sceneFileType": "gltf",
          "hasFixations": true,
          "isOptimized": true,
          "dynamicsUpdateKey": null
        }
      ]
    }
  ]
}
```

`scenes[].versions[].id` is the `versionId` used in most other endpoints. A scene can have multiple versions if it has been republished.

### Create Project
```
POST /v0/projects
{ "name": "...", "description": "...", "type": "vr", "organizationId": "...", "enableGeoIp": true }
```

### Update Project
```
PUT /v0/projects/:projectId
{ "name": "...", "type": "vr", "categoryIds": [6], "enableGeoIp": true }
```

### Get Scene
```
GET /v0/scenes/:sceneId
```
```json
{
  "id": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
  "sceneName": "factory scene",
  "projectId": 341,
  "sdkFacingId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
  "isPublic": false,
  "hidden": false,
  "sceneCreationMethod": "NORMAL",
  "versions": [
    {
      "id": 141,
      "sceneId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
      "versionNumber": 1,
      "scale": 1.0,
      "sdkVersion": "0.21.0",
      "sceneFileType": "gltf",
      "hasFixations": true,
      "isOptimized": true,
      "dynamicsUpdateKey": null,
      "stats": {
        "latest_session": 1773998824802,
        "total_session_time": 49081137,
        "average_session_duration": 102465.83,
        "session_count": 479,
        "total_gaze_time": 0
      }
    }
  ]
}
```
Note: `scenes/:id` takes the UUID sceneId, not the integer versionId. Do not append a trailing slash — `GET /v0/scenes/:sceneId/` returns 404.

### List Scenes for Project
```
GET /v0/projects/:projectId/scenes?attachStats=false
```
Response per version: `{ id, versionNumber, sdkVersion, sceneFileType, hasFixations, isOptimized }`

---

## Organization

```
GET /v0/organizations/:organizationId?excludeTags=test,junk
GET /v0/organizations/:organizationId/tags
GET /v0/organizations/:organizationId/tags/all
```
Organization level access required.

**GET organizations/:id response** (organization object with full project list):
```json
{
  "id": 12,
  "projects": [
    {
      "id": 341,
      "organizationId": 12,
      "name": "Sample Electrical Training Demo",
      "description": "",
      "projectType": "vr",
      "enableGeoIp": true,
      "enableAi": true,
      "hidden": false,
      "prefix": "auto_cid_Electrical_Demo_341",
      "imageLocation": null,
      "categoryGroup": { "id": 5, "name": "Training and Simulation" },
      "sessionTagTypes": [],
      "categories": [],
      "stats": null
    }
    // ... all projects in the org
  ]
}
```

**GET organizations/:id/tags response** (array of tag definitions):
```json
[
  { "id": 1,   "title": "test",   "colourHex": "ba98fa", "isDefault": true },
  { "id": 2,   "title": "junk",   "colourHex": "eb7a95", "isDefault": true },
  { "id": 260, "title": "Crash",  "colourHex": "a10a0a", "isDefault": true },
  { "id": 261, "title": "LowMemory", "colourHex": "cc8306", "isDefault": true },
  {
    "id": 15,
    "organizationId": 12,
    "title": "exitpoll",
    "colourHex": "330582",
    "isDefault": false,
    "description": ""
  }
  // ... custom tags
]
```
`isDefault: true` tags are system tags (test, junk, Crash, LowMemory). Custom tags have an `organizationId`. `/tags/all` returns the same structure but includes additional inherited entries.

**No org-level aggregate analytics endpoint exists.** `GET /v0/organizations/:organizationId` returns metadata and the project list only — it does not return monthly session counts or other aggregates (a `stats.session_count_by_month` field that once existed is no longer returned; verified absent on both environments 2026-08-05). To aggregate analytics across an organization, run one slicer query per project and combine the results client-side. Run them sequentially, and always set **both** `gte` and `lte` date bounds — open-ended date ranges can 502 on projects with a lot of data.

---

## Objectives & ExitPoll

### Objectives (Training Pass/Fail)
```
GET /v0/projects/:projectId/objectives
```
Returns all objectives defined in the project. Each objective contains its `objectiveVersions` (published snapshots used in sessions) and `objectiveComponents` (the steps).
```json
[
  {
    "id": 269,
    "name": "Lockout Checklist",
    "description": "steps required for a successful lock out tag out",
    "projectId": 341,
    "enabled": true,
    "sceneVersionId": null,
    "sceneId": null,
    "objectiveVersions": [
      {
        "id": 445,
        "isActive": true,
        "saveToSession": false,
        "propertyLabel": null,
        "criteria": null,
        "objectiveComponents": [
          {
            "id": 883,
            "sequenceNumber": 1,
            "type": "eventstep",
            "eventName": "Equip Hard Hat",
            "occurrenceOperator": "eq",
            "occurrenceValue": 1,
            "isStep": true,
            "isStrict": false,
            "isNonSequential": false,
            "propertyConditions": [],
            "dynamicObjectIds": []
          }
          // ... more steps
        ]
      }
    ]
  }
]
```
Key fields: `objectiveVersions[].id` is the `objectiveVersionId` used in session results. Component `type` is `eventstep` (triggered by event name) or `gazestep` (triggered by gaze on a dynamic object). `propertyConditions` optionally filter on event properties.

```
GET /v0/versions/:versionId/objectives/:objectiveId
GET /v0/versions/:versionId/sessions/:sessionId/objectiveData
GET /v0/versions/:versionId/objectiveVersions/:objectiveVersionId/stepResults?excludeJunkAndTest=true
    // Returns: [{ step, succeeded, failed, averageStepCompletionTime, averageStepDuration }]
GET /v0/projects/:projectId/objectiveVersions/:objectiveVersionId/results.csv?excludeJunkAndTestSessions=true&targetNewObjectives=true
```

> ⚠️ `GET /v0/versions/:versionId/objectiveVersions/:objectiveVersionId/results` returns 404 — do not use this endpoint.

> ⚠️ `stepResults` has also been observed returning 404 for **every** valid versionId/objectiveVersionId combination tested (development environment, 2026-08-05, 8+ combinations). If it 404s for you, fall back to `results.csv`, which reliably returns per-session, per-step rows you can aggregate yourself.

Note: Objective results don't include the objective definition — fetch objectives separately and join by ID.

**GET objectiveData response** (map keyed by objectiveVersionId string → array of step results):
```json
{
  "295": [
    { "step": 1, "timestamp": "2026-03-20T09:27:04.802Z", "duration": 0, "result": "succeeded" }
  ],
  "445": [
    { "step": 1, "timestamp": "2026-03-20T09:27:20.221Z", "duration": 15419, "result": "succeeded" },
    { "step": 2, "timestamp": "2026-03-20T09:27:37.974Z", "duration": 17753, "result": "succeeded" },
    { "step": 3, "timestamp": "2026-03-20T09:27:39.771Z", "duration": 1797,  "result": "succeeded" },
    { "step": 4, "timestamp": "2026-03-20T09:27:47.931Z", "duration": 8160,  "result": "succeeded" },
    { "step": 5, "timestamp": "2026-03-20T09:27:53.639Z", "duration": 5708,  "result": "succeeded" },
    { "step": 6, "timestamp": "2026-03-20T09:28:01.844Z", "duration": 8205,  "result": "succeeded" },
    { "step": 7, "timestamp": "2026-03-20T09:28:10.880Z", "duration": 9036,  "result": "succeeded" }
  ],
  "274": [
    { "step": 1, "timestamp": "2026-03-20T09:28:25.082Z", "duration": 80280, "result": "failed" }
  ]
}
```

- Keys are objectiveVersionId strings — cross-reference with `GET /v0/projects/:projectId/objectives` to get names
- `duration` is milliseconds from session start to step completion
- `result` is `"succeeded"` or `"failed"`
- Multi-step objectives (like `"445"` above) have one entry per step

### ExitPoll (In-Session Surveys)
```
GET  /v0/projects/:projectId/questionSets
GET  /v0/projects/:projectId/questionSets/:questionSetName
```
Returns an array of all versions of the named question set. Same structure as the `GET questionSets` list response, but filtered to one name. Useful to find which version numbers exist and compare question definitions across versions.
```json
[
  {
    "id": "assessment_begin_survey:1",
    "name": "assessment_begin_survey",
    "version": 1,
    "status": "active",
    "title": "User Survey",
    "projectId": 341,
    "questions": [
      { "type": "MULTIPLE", "title": "How experienced are you with electrical work?", "saveToSession": true, "propertyLabel": "ExperienceLevel", "answers": [...] },
      { "type": "THUMBS",   "title": "Have you use VR before?", "saveToSession": false, "propertyLabel": null },
      { "type": "SCALE",    "title": "How well do you think you will perform?", "saveToSession": false, "minLabel": "Poorly", "maxLabel": "Very well", "range": { "start": 0, "end": 10 } }
    ]
  },
  {
    "id": "assessment_begin_survey:2",
    "name": "assessment_begin_survey",
    "version": 2,
    "status": "active",
    "title": "User Survey",
    "projectId": 341,
    "questions": [ ... ]
  }
]
```
```
GET  /v0/projects/:projectId/questionSets/:name/:version/responses?excludeTags=test,junk&limit=1000&page=0&orderBy=createdAt&sort=DESC&search=archived:false
POST /v0/projects/:projectId/questionSets/:name/:version/responseDumps?search=archived:false
POST /v0/projects/:projectId/questionSets/:name/:version/responseCountQueries
     // Body: an OBJECT wrapping the filters: {"sessionFilters": [...]}
     // (filters use the same format as slicerQueries). Posting the bare
     // array returns 400 "Invalid json". With no filtering, send
     // {"sessionFilters": []}.
DELETE /v0/projects/:projectId/questionSets/:questionSetName   // archives it
```

**GET questionSets response** (array of question set definitions):
```json
[
  {
    "id": "assessment_begin_survey:2",
    "name": "assessment_begin_survey",
    "title": "User Survey",
    "version": 2,
    "status": "active",
    "projectId": 341,
    "versions": ["assessment_begin_survey:1", "assessment_begin_survey:2"],
    "questions": [
      {
        "type": "MULTIPLE",
        "title": "How experienced are you with electrical work?",
        "saveToSession": true,
        "propertyLabel": "ExperienceLevel",
        "answers": [
          { "answer": "Professional", "icon": null },
          { "answer": "Very experienced", "icon": null },
          { "answer": "Somewhat experienced", "icon": null },
          { "answer": "No experience", "icon": null }
        ]
      },
      {
        "type": "THUMBS",
        "title": "Have you used VR before?",
        "saveToSession": false,
        "propertyLabel": null
      },
      {
        "type": "SCALE",
        "title": "How well do you think you will perform?",
        "saveToSession": false,
        "propertyLabel": null,
        "minLabel": "Poorly",
        "maxLabel": "Very well",
        "range": { "start": 0, "end": 10 }
      }
    ]
  }
  // ... additional question sets
]
```

Question types: `MULTIPLE` (multiple choice), `THUMBS` (thumbs up/down), `SCALE` (numeric range), `BOOLEAN` (yes/no), `HAPPYSAD` (emoji sentiment, no extra fields), `VOICE` (audio response, has `maxResponseLength` integer). When `saveToSession: true`, the answer is stored as a session property under `c3d.participant.exitpoll.<propertyLabel>`.

**GET responses response** (paginated, one record per question per submission):
```json
{
  "count": 465,
  "pages": 10,
  "currentPage": 0,
  "results": [
    {
      "id": 841510903072697994,
      "type": "MULTIPLE",
      "hook": "assessment_begin",
      "questionIndex": 0,
      "timestamp": 1773998836504,
      "userId": "6826c6d546130967474184c118bfbc90",
      "projectId": 341,
      "participantId": "00130",
      "sceneId": "2c2f0603-8f5c-4ff6-94ed-03542a4df34d",
      "sceneVersionNumber": 1,
      "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
      "sessionFriendlyName": "Marco Aiello",
      "sessionSqlId": 948907,
      "questionSetId": "assessment_begin_survey:2",
      "skipped": false,
      "value": 1
    },
    {
      "id": 841510903072697994,
      "type": "THUMBS",
      "hook": "assessment_begin",
      "questionIndex": 1,
      "timestamp": 1773998836504,
      "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
      "questionSetId": "assessment_begin_survey:2",
      "skipped": false,
      "value": 0
    },
    {
      "id": 841510903072697994,
      "type": "SCALE",
      "hook": "assessment_begin",
      "questionIndex": 2,
      "timestamp": 1773998836504,
      "sessionId": "1773998824_6826c6d546130967474184c118bfbc90",
      "questionSetId": "assessment_begin_survey:2",
      "skipped": false,
      "value": 8
    }
  ]
}
```

Each submission generates one record per question. Records from the same submission share the same `id` (the eventHash) and `timestamp`. `questionIndex` tells you which question within the set. `value` is the raw answer index for MULTIPLE/THUMBS, or a numeric value for SCALE. To decode MULTIPLE answers, look up the question definition and index into its `answers` array.

---

## Remote Config & A/B Tests

### Remote Variables
```
GET  /v0/projects/:projectId/remoteVariables
GET  /v0/projects/:projectId/remoteVariables/:remoteVariableId
POST /v0/projects/:projectId/remoteVariables
     { "name": "MyVar", "valueString": "hello" }   // or valueInt / valueBoolean
PUT  /v0/projects/:projectId/remoteVariables/:remoteVariableId
     { "name": "...", "valueBoolean": true }
```
Value types: `valueString`, `valueInt`, `valueBoolean`

**GET remoteVariables response:**
```json
[
  {
    "id": 41,
    "projectId": 341,
    "name": "testing_key",
    "type": "int",
    "valueInt": 4,
    "valid": true
  }
]
```
`type` is `"int"`, `"string"`, or `"boolean"`. The value field matches: `valueInt`, `valueString`, or `valueBoolean`. `valid` indicates whether the variable has a current active configuration.

### Remote Configurations (named overrides)
```
GET    /v0/projects/:projectId/remoteConfigurations
POST   /v0/projects/:projectId/remoteConfigurations
       { "name": "...", "description": "...", "remoteVariableId": "...", "enabled": true, "overrideValueInt": 42 }
PUT    /v0/projects/:projectId/remoteConfigurations/:remoteConfigurationId
       { "name": "...", "overrideValueInt": 1234 }
DELETE /v0/projects/:projectId/remoteConfigurations/:remoteConfigurationId
```

### A/B Tests
```
GET    /v0/projects/:projectId/abTests
GET    /v0/projects/:projectId/abTests/:abTestId
POST   /v0/projects/:projectId/abTests
       {
         "name": "...", "remoteVariableId": "...",
         "remoteVariableProbability": 33,
         "isEnabled": true, "isSticky": "true",
         "abTestVariables": [
           { "valueInt": 1, "probability": 33 },
           { "valueInt": 2, "probability": 33 }
         ]
       }
PUT    /v0/projects/:projectId/abTests/:abTestId
DELETE /v0/projects/:projectId/abTests/:abTestId
```
`isSticky`: same user always gets the same variant across sessions.

---

## Media (360° Video)

```
GET    /v0/projects/:projectId/media?excludeArchived=true
GET    /v0/projects/:projectId/media/:mediaId/files/source_albedo.png
GET    /v0/projects/:projectId/media/:mediaId/pointOfInterests
DELETE /v0/projects/:projectId/media/:mediaId
```
Media fixation data is computed from gaze on 360° media spheres and included in `FIXATION` jsonRequests.

**GET media response:**
```json
[
  {
    "id": 189,
    "name": "colourgrid.png",
    "uploadId": "874150d6-c262-4931-aa29-bff648bfcaf1",
    "staticMedia": true,
    "archived": false,
    "description": "just a testing image",
    "thumbnailLocation": "thumbnail/full.png",
    "fileName": "colourgrid.png",
    "contentType": "image/png",
    "externalUrl": null
  }
]
```
`staticMedia: true` means it's a flat image (not a 360° video). `id` is the `mediaId` used in pointOfInterests and file download endpoints.

---

## Widgets & Dashboards

### Widgets (saved queries)
```
GET    /v0/projects/:projectId/widgets
GET    /v0/projects/:projectId/widgets/:widgetId
POST   /v0/projects/:projectId/widgets
       { "title": "...", "chartType": "DEFAULT", "query": { ... } }
PUT    /v0/projects/:projectId/widgets/:widgetId
DELETE /v0/projects/:projectId/widgets/:widgetId
```
**GET widgets response:**
```json
[
  {
    "id": 342,
    "projectId": 431,
    "userId": 1,
    "title": "averageSessionDuration",
    "isPreset": true,
    "chartType": null,
    "query": {
      "friendlyName": "Average Session Duration",
      "name": "averageSessionDuration",
      "type": "outOfTheBox",
      "aggType": "singular"
    }
  }
]
```
`isPreset: true` means it's a built-in C3D widget (not user-created). `query.type` is `"outOfTheBox"` for presets or `"custom"` for user-defined slicer queries. `chartType` is null for presets; user widgets may have `"DEFAULT"`, `"BAR"`, etc.

### Custom Dashboards
```
GET    /v0/projects/:projectId/dashboards
POST   /v0/projects/:projectId/dashboards
       {
         "title": "...",
         "dateRangeKey": "30D",
         "dateStart": <ms>, "dateEnd": <ms>,
         "widgetMetadataList": [{ "widgetId": 291, "title": "...", "type": "CUSTOM" }]
       }
PUT    /v0/projects/:projectId/dashboards/:dashboardId
DELETE /v0/projects/:projectId/dashboards/:dashboardId
```
`isDefault`: only one custom dashboard can be default at a time.

---

## App Review Importer

```
POST /v0/projects/:projectId/reviews
     // multipart/form-data: reviews = reviews.csv (exported from Meta Quest developer portal)
GET  /v0/projects/:projectId/reviews/session?sdkSessionId=<id>&versionId=<id>
GET  /v0/projects/:projectId/reviews
```

---

## Authentication Endpoints

```
POST /v0/sessions          // Login — returns { userId, csrf }
GET  /v0/sessions/current  // Returns csrf token for current session
```
