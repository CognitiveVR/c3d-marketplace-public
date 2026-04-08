# Cognitive3D API — Code Examples

Language examples for common Cognitive3D API patterns.
Replace `YOUR_API_KEY`, `YOUR_PROJECT_ID`, `YOUR_SCENE_ID`, etc. with real values.

Base URLs:
- Production: `https://api.cognitive3d.com`
- Development: `https://api.c3ddev.com`

---

## Authentication Setup

### Python
```python
import requests

BASE_URL = "https://api.cognitive3d.com/v0"
API_KEY = "orgkey-YOUR_API_KEY"   # or "APIKEY:ORGANIZATION YOUR_KEY" for legacy

headers = {
    "Authorization": API_KEY,
    "Content-Type": "application/json"
}
```

### JavaScript (fetch)
```javascript
const BASE_URL = "https://api.cognitive3d.com/v0";
const API_KEY = "orgkey-YOUR_API_KEY";

const headers = {
  "Authorization": API_KEY,
  "Content-Type": "application/json"
};
```

### C#
```csharp
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

var client = new HttpClient();
client.BaseAddress = new Uri("https://api.cognitive3d.com/v0/");
client.DefaultRequestHeaders.Add("Authorization", "orgkey-YOUR_API_KEY");
```

---

## List Sessions (Paginated)

### Python
```python
def list_sessions(project_id, page=0, limit=20):
    payload = {
        "entityFilters": {"projectId": project_id},
        "sessionFilters": [
            {
                "field": {"fieldParent": "session", "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.test"},
                "op": "eq",
                "value": False
            }
        ],
        "page": page,
        "limit": limit,
        "sort": "desc",
        "orderBy": {"fieldName": "date", "fieldParent": "session"}
    }
    resp = requests.post(f"{BASE_URL}/datasets/sessions/paginatedListQueries", json=payload, headers=headers)
    resp.raise_for_status()
    return resp.json()

sessions = list_sessions(project_id=11)
```

### JavaScript
```javascript
async function listSessions(projectId, page = 0, limit = 20) {
  const body = {
    entityFilters: { projectId },
    sessionFilters: [
      {
        field: { fieldParent: "session", nestedFieldName: "booleanSessionProp", path: "c3d.session_tag.test" },
        op: "eq",
        value: false
      }
    ],
    page,
    limit,
    sort: "desc",
    orderBy: { fieldName: "date", fieldParent: "session" }
  };
  const resp = await fetch(`${BASE_URL}/datasets/sessions/paginatedListQueries`, {
    method: "POST",
    headers,
    body: JSON.stringify(body)
  });
  return resp.json();
}
```

### C#
```csharp
async Task<JsonElement> ListSessionsAsync(int projectId, int page = 0, int limit = 20)
{
    var body = new
    {
        entityFilters = new { projectId },
        sessionFilters = new[]
        {
            new {
                field = new { fieldParent = "session", nestedFieldName = "booleanSessionProp", path = "c3d.session_tag.test" },
                op = "eq",
                value = (object)false
            }
        },
        page,
        limit,
        sort = "desc",
        orderBy = new { fieldName = "date", fieldParent = "session" }
    };
    var json = JsonSerializer.Serialize(body);
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    var response = await client.PostAsync("datasets/sessions/paginatedListQueries", content);
    response.EnsureSuccessStatusCode();
    var result = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<JsonElement>(result);
}
```

---

## Get Raw Session Data

### Python
```python
def get_session_data(project_id, session_id, json_type="ALL"):
    """json_type: GAZE, FIXATION, EVENTS, DYNAMICS, SENSORS, BOUNDARY, ALL"""
    payload = {
        "sessionId": session_id,
        "jsonType": json_type,
        "addGeo": True,
        "addMetadata": True,
        "addProperties": True,
        "sessionRelativeTimestamps": True,
        "convertMillisToSeconds": True
    }
    resp = requests.post(
        f"{BASE_URL}/projects/{project_id}/sessions/{session_id}/jsonRequests",
        json=payload,
        headers=headers
    )
    resp.raise_for_status()
    return resp.json()

data = get_session_data(project_id=11, session_id="abc-123-sdk-session-id")
gaze_points = data["data"]["gaze"]
events = data["data"]["events"]
```

### JavaScript
```javascript
async function getSessionData(projectId, sessionId, jsonType = "ALL") {
  const body = {
    sessionId,
    jsonType,
    addGeo: true,
    addMetadata: true,
    addProperties: true,
    sessionRelativeTimestamps: true,
    convertMillisToSeconds: true
  };
  const resp = await fetch(`${BASE_URL}/projects/${projectId}/sessions/${sessionId}/jsonRequests`, {
    method: "POST",
    headers,
    body: JSON.stringify(body)
  });
  const data = await resp.json();
  return data;
}

const data = await getSessionData(11, "abc-123-sdk-session-id");
const gazePoints = data.data.gaze;
```

### C#
```csharp
async Task<JsonElement> GetSessionDataAsync(int projectId, string sessionId, string jsonType = "ALL")
{
    var body = new { sessionId, jsonType, addGeo = true, addMetadata = true, addProperties = true };
    var json = JsonSerializer.Serialize(body);
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    var response = await client.PostAsync($"projects/{projectId}/sessions/{sessionId}/jsonRequests", content);
    response.EnsureSuccessStatusCode();
    var result = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<JsonElement>(result);
}
```

---

## Analytics Query (Slicer)

### Python — Session count with date filter
```python
from datetime import datetime

def to_ms(dt: datetime) -> int:
    """Convert datetime to milliseconds since epoch."""
    return int(dt.timestamp() * 1000)

def session_count_query(project_id, start_dt, end_dt):
    payload = {
        "entityFilters": {"projectId": project_id},
        "sessionFilters": [
            {
                "field": {"fieldParent": "session", "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.test"},
                "op": "eq",
                "value": False
            },
            {
                "field": {"fieldName": "date", "fieldParent": "session"},
                "op": "gte",
                "value": to_ms(start_dt)
            },
            {
                "field": {"fieldName": "date", "fieldParent": "session"},
                "op": "lte",
                "value": to_ms(end_dt)
            }
        ],
        "sessionType": "project",
        "aggregation": {
            "name": "main",
            "operations": [{"name": "session_count", "type": "sessionCount"}]
        }
    }
    resp = requests.post(f"{BASE_URL}/datasets/sessions/slicerQueries", json=payload, headers=headers)
    resp.raise_for_status()
    data = resp.json()
    # Response: aggregations → <aggregation name> → <operation name> → value
    return data["aggregations"]["main"]["session_count"]["value"]

from datetime import datetime
count = session_count_query(11, datetime(2024, 1, 1), datetime(2024, 1, 31))
print(count)
```

### Python — Filter by session property
```python
def sessions_by_device(project_id, device_type):
    payload = {
        "entityFilters": {"projectId": project_id},
        "sessionFilters": [
            {
                "field": {"fieldParent": "session", "nestedFieldName": "booleanSessionProp", "path": "c3d.session_tag.test"},
                "op": "eq",
                "value": False
            },
            {
                "op": "AND",
                "children": [{"op": "AND", "children": [
                    {
                        "field": {"nestedFieldName": "textualSessionProp", "fieldParent": "session", "path": "c3d.device.type"},
                        "op": "eq",         # use "wildcard" with "*Quest*" for partial match
                        "value": device_type
                    }
                ]}]
            }
        ],
        "sessionType": "project",
        "aggregation": {
            "name": "main",
            "operations": [{"name": "session_count", "type": "sessionCount"}]
        }
    }
    resp = requests.post(f"{BASE_URL}/datasets/sessions/slicerQueries", json=payload, headers=headers)
    resp.raise_for_status()
    return resp.json()
```

### JavaScript — Analytics query
```javascript
async function slicerQuery(projectId, sessionFilters = [], aggregation = null) {
  const body = {
    entityFilters: { projectId },
    sessionFilters: [
      {
        field: { fieldParent: "session", nestedFieldName: "booleanSessionProp", path: "c3d.session_tag.test" },
        op: "eq",
        value: false
      },
      ...sessionFilters
    ],
    sessionType: "project",
    aggregation: aggregation ?? {
      name: "main",
      operations: [{ name: "session_count", type: "sessionCount" }]
    }
  };
  const resp = await fetch(`${BASE_URL}/datasets/sessions/slicerQueries`, {
    method: "POST",
    headers,
    body: JSON.stringify(body)
  });
  return resp.json();
}
```

---

## Gaze Metrics on Dynamic Objects

Objects are identified in gaze queries by their `sdkId` (a UUID string like `"c34cbd8b-4fb9-..."`), not their friendly name. If you only have a friendly name, look up the sdkId first (see "Lookup Object sdkId by Name" below).

### Python
```python
def get_object_gaze(project_id, scene_id, version_id, sdk_ids, gaze_type="gaze"):
    """sdk_ids: list of sdkId UUID strings from the dynamic objects list."""
    payload = {
        "entityFilters": {
            "projectId": project_id,   # integer
            "sceneId": scene_id,
            "versionId": version_id    # integer
        },
        "gazeType": gaze_type,   # "gaze" or "fixation"
        "aggregations": "all",
        "objectIds": sdk_ids
    }
    resp = requests.post(
        f"{BASE_URL}/datasets/sessions/slicerObjectMetricQueries",
        json=payload,
        headers=headers
    )
    resp.raise_for_status()
    data = resp.json()
    # data["metrics"][<sdkId>]["totalGazeLength"] etc.
    # Note: metric values may be null if no data exists — treat null as 0
    for sdk_id, metrics in data["metrics"].items():
        avg_gaze = metrics.get("averageGazeLength") or 0
        total_count = metrics.get("totalGazeCount") or 0
        print(f"{sdk_id}: avg gaze {avg_gaze:.1f}ms, total look count {total_count}")
    return data

result = get_object_gaze(11, "scene-uuid-here", 312, ["c34cbd8b-4fb9-47a2-bd6c-afb01aa5d05c"])
```

### JavaScript
```javascript
async function getObjectGaze(projectId, sceneId, versionId, sdkIds, gazeType = "gaze") {
  const body = {
    entityFilters: {
      projectId,   // integer
      sceneId,
      versionId    // integer
    },
    gazeType,
    aggregations: "all",
    objectIds: sdkIds
  };
  const resp = await fetch(`${BASE_URL}/datasets/sessions/slicerObjectMetricQueries`, {
    method: "POST",
    headers,
    body: JSON.stringify(body)
  });
  const data = await resp.json();
  // Metric values may be null — treat null as 0
  return Object.fromEntries(
    Object.entries(data.metrics).map(([sdkId, m]) => [sdkId, {
      averageGazeLength: m.averageGazeLength ?? 0,
      totalGazeCount: m.totalGazeCount ?? 0,
      totalSessionsWithAnyGaze: m.totalSessionsWithAnyGaze ?? 0,
    }])
  );
}
```

---

## Lookup Object sdkId by Name

Objects have friendly names in the C3D dashboard (e.g. "Cube (4)") but are identified in gaze queries by their `sdkId` UUID. Use this lookup when a user refers to an object by name:

### Python
```python
def get_sdk_id_by_name(version_id, object_name):
    """Look up an object's sdkId given its friendly name and scene version ID."""
    resp = requests.get(
        f"{BASE_URL}/versions/{version_id}/objects",
        headers=headers
    )
    resp.raise_for_status()
    objects = resp.json()
    for obj in objects:
        if obj["name"] == object_name:
            return obj["sdkId"]
    raise ValueError(f"Object '{object_name}' not found in version {version_id}")

# Example: find sdkId for the object named "Cube (4)"
sdk_id = get_sdk_id_by_name(1157, "Cube (4)")
# → "c34cbd8b-4fb9-47a2-bd6c-afb01aa5d05c"
```

---

## Discover Filterable Properties

### Python
```python
def get_available_properties(project_id):
    payload = {"entityFilters": {"projectId": project_id}}
    resp = requests.post(
        f"{BASE_URL}/datasets/sessions/slicerPropertyNameQueries",
        json=payload,
        headers=headers
    )
    resp.raise_for_status()
    props = resp.json()
    print("Text properties:", props["textualSessionProp"])
    print("Number properties:", props["numericalSessionProp"])
    print("Boolean/tag properties:", props["booleanSessionProp"])
    return props
```

---

## Get Project and Scene Info

### Python
```python
def get_project(project_id):
    resp = requests.get(f"{BASE_URL}/projects/{project_id}", headers=headers)
    resp.raise_for_status()
    project = resp.json()
    # project["scenes"] contains scene list with versions
    for scene in project.get("scenes", []):
        print(f"Scene: {scene['sceneName']} ({scene['id']})")
        for v in scene.get("versions", []):
            print(f"  Version {v['versionNumber']} → versionId={v['id']}")
    return project
```

### C#
```csharp
async Task<JsonElement> GetProjectAsync(int projectId)
{
    var response = await client.GetAsync($"projects/{projectId}");
    response.EnsureSuccessStatusCode();
    var result = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<JsonElement>(result);
}
```

---

## Write Operations

The same `Authorization` header used for reads works for write operations.

### Python — Tag a session
```python
def tag_session(project_id, session_id, tag_name, value=True):
    resp = requests.put(
        f"{BASE_URL}/projects/{project_id}/sessions/{session_id}/tags/{tag_name}",
        json={"value": value},
        headers=headers
    )
    resp.raise_for_status()
    return resp.json()

tag_session(11, "sdk-session-id", "junk", True)
```

### JavaScript — Tag a session
```javascript
async function tagSession(projectId, sessionId, tagName, value = true) {
  const resp = await fetch(
    `${BASE_URL}/projects/${projectId}/sessions/${sessionId}/tags/${tagName}`,
    {
      method: "PUT",
      headers,
      body: JSON.stringify({ value })
    }
  );
  return resp.json();
}
```
