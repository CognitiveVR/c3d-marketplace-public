# Endpoint Coverage

Tracks all documented endpoints and whether they have example responses, and what filter types they support.
Update this table whenever a route is added, changed, or gets an example response added.

**Filter support key:**
- `—` = no filter support
- `entityFilters` = projectId/sceneId/versionId scoping only
- `sessionFilters` = accepts `sessionFilters` array (same syntax as slicerQueries)
- `sessionFilters + slicer` = full suite: `sessionFilters`, `separableEventFilters`, `eventFilters`, `userFilters`
- `query params` = URL query parameters only (e.g. `excludeTags`, `limit`, `page`)

| Endpoint | Method | Example Response | Filter Support |
|----------|--------|:---------------:|----------------|
| **Sessions** | | | |
| `datasets/sessions/paginatedListQueries` | POST | ✅ | `sessionFilters` |
| `datasets/sessions/singleQueries` | POST | ✅ | `entityFilters` |
| `datasets/sessions/singleProjectSessionQueries` | POST | ✅ | `entityFilters` |
| `projects/:id/sessions/:id/jsonRequests` | POST | ❌ | — |
| `projects/:id/sessions/:id/jsonZipRequest` | GET | — binary ZIP | — |
| `versions/:id/sessions/:id/metadata` | GET | ❌ | — |
| `projects/:id/sessions/metadataLookups` | POST | ❌ | — |
| `datasets/sessions/participantSessionsQueries` | POST | ❌ | `entityFilters` |
| `projects/:id/sessions/:id/tags/:tag` | PUT | ❌ | — |
| `organizations/:id/tags` | GET/POST | ❌ | — |
| `projects/:id/sessions/:id/report.pdf` | GET | — binary PDF | — |
| `versions/:id/sessionsReport.pdf` | POST | — binary PDF | `sessionFilters` |
| `versions/:id/sessions/:id/reportEmails` | POST | ❌ | — |
| **Analytics / Slicer** | | | |
| `datasets/sessions/slicerPropertyNameQueries` | POST | ✅ inline | `entityFilters` |
| `datasets/sessions/slicerQueries` | POST | ✅ | `sessionFilters + slicer` |
| `datasets/sessions/slicerObjectMetricQueries` | POST | ✅ | `entityFilters`, `sessionFilters` |
| `datasets/sessions/slicerSingleObjectPerSessionMetricQueries` | POST | ✅ inline | `entityFilters`, `sessionFilters` |
| **Dynamic Objects** | | | |
| `versions/:id/objects` | GET | ✅ | — |
| `projects/:id/objects` | GET | ✅ | — |
| `versions/:id/objects/:id` | DELETE | — no body | — |
| **Projects & Scenes** | | | |
| `projects/:id` | GET | ✅ | — |
| `projects` | POST | ❌ | — |
| `projects/:id` | PUT | ❌ | — |
| `scenes/:id` | GET | ✅ | — |
| `projects/:id/scenes` | GET | ✅ inline | — |
| **Organization** | | | |
| `organizations/:id` | GET | ✅ | — |
| `organizations/:id/tags` | GET | ✅ | — |
| `organizations/:id/tags/all` | GET | ✅ | — |
| **Objectives** | | | |
| `projects/:id/objectives` | GET | ✅ | — |
| `projects/:id/objectives/:id` | GET | ✅ | — |
| `versions/:id/objectives/:id` | GET | ❌ | — |
| `versions/:id/sessions/:id/objectiveData` | GET | ✅ | — |
| `versions/:id/objectiveVersions/:id/stepResults` | GET | ⚠️ observed 404 (dev, 2026-08-05) — use `results.csv` | — |
| `projects/:id/objectiveVersions/:id/results.csv` | GET | — CSV file | `query params` |
| `datasets/objectives/objectiveResultQueries` | POST | ✅ | `sessionFilters` |
| `datasets/objectives/objectiveStepResultQueries` | POST | ✅ | `sessionFilters` |
| **ExitPoll** | | | |
| `projects/:id/questionSets` | GET | ✅ | — |
| `projects/:id/questionSets/:name` | GET | ✅ | — |
| `projects/:id/questionSets/:name/:ver/responses` | GET | ✅ | `query params` |
| `projects/:id/questionSets/:name/:ver/responseDumps` | POST | ❌ | `query params` |
| `projects/:id/questionSets/:name/:ver/responseCountQueries` | POST | ❌ | `sessionFilters` |
| `projects/:id/questionSets/:name` | DELETE | — no body | — |
| **Remote Config & A/B Tests** | | | |
| `projects/:id/remoteVariables` | GET/POST/PUT | ✅ | — |
| `projects/:id/remoteConfigurations` | GET/POST/PUT/DELETE | ❌ | — |
| `projects/:id/abTests` | GET/POST/PUT/DELETE | ❌ | — |
| **Media** | | | |
| `projects/:id/media` | GET/DELETE | ✅ | — |
| `projects/:id/media/:id/pointOfInterests` | GET | ❌ | — |
| **Widgets & Dashboards** | | | |
| `projects/:id/widgets` | GET/POST/PUT/DELETE | ✅ | — |
| `projects/:id/dashboards` | GET/POST/PUT/DELETE | ❌ | — |
| **App Reviews** | | | |
| `projects/:id/reviews` | POST/GET | ❌ | — |
| **Auth** | | | |
| `sessions` (login) | POST | ❌ | — |
| `sessions/current` | GET | ❌ | — |
