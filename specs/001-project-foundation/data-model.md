# Data Model: Project Foundation

This feature introduces no persisted business entities or domain collections. Its data model is
limited to validated configuration and observable service health. MongoDB persistence is provisioned
but remains empty until a domain feature owns a collection.

## Environment Configuration

Represents the validated local settings required before applications or services start.

| Field | Type | Required | Validation | Consumer |
|---|---|---:|---|---|
| `NODE_ENV` | enum | yes | `development`, `test`, or `production` | web, API |
| `WEB_PORT` | integer | yes | 1 through 65535; distinct from `API_PORT` | web, Compose |
| `API_PORT` | integer | yes | 1 through 65535; distinct from `WEB_PORT` | API, Compose |
| `MONGODB_URI` | URL string | yes | `mongodb://` or `mongodb+srv://`; credentials never logged | API |
| `NEXT_PUBLIC_API_URL` | HTTP URL | yes | reachable API origin; no secret values | web |

Rules:

- `.env.example` documents names and non-secret local defaults; `.env` is never committed.
- Preflight validation reports the invalid field without printing its value.
- Validation completes before dependency installation in the documented clean-clone flow and before
  application or Compose startup in all flows.
- Application schemas may be separate runtime adapters, but their accepted values must remain
  consistent with this model.

## Health Response

The reusable response contract returned by each public health endpoint.

| Field | Type | Required | Validation |
|---|---|---:|---|
| `status` | enum | yes | `ok` or `error` |
| `service` | enum | yes | `web` or `api` |
| `checks` | array of Health Check | yes | At least one item; names unique per response |
| `timestamp` | string | yes | RFC 3339 UTC timestamp |

## Health Check

| Field | Type | Required | Validation |
|---|---|---:|---|
| `name` | string | yes | Stable lowercase identifier |
| `status` | enum | yes | `up` or `down` |

Required checks:

- The web response contains `web: up` while its server can process requests.
- The API response contains `api: up` and `mongodb: up` only after the database connection is ready.
- Any required check with `down` produces top-level `status: error` and a non-2xx HTTP status.
- Health payloads contain no exception messages, credentials, connection strings, host internals, or
  stack traces.

## Service Instance

An operational component managed by Docker Compose rather than a persisted application entity.

| Service | Depends on | Ready when | Failure state |
|---|---|---|---|
| `mongodb` | none | MongoDB ping succeeds | ping fails or times out |
| `api` | `mongodb` healthy | `GET /health` satisfies the contract | endpoint is unavailable or reports `error` |
| `web` | `api` healthy | `GET /api/health` satisfies the contract and page responds | endpoint or page is unavailable |

## State Transitions

```text
configured -> starting -> healthy -> stopping -> stopped
                  |          |
                  +-> failed <-+
```

- `configured -> starting` is blocked when preflight validation fails.
- `starting -> healthy` requires the component's health check to pass.
- The integrated environment is healthy only when MongoDB, API, and web are all healthy within two
  minutes.
- A startup failure preserves diagnostic status and can transition to `stopping` for cleanup.
- Normal and failed validation paths must stop managed processes; explicit local volume cleanup is a
  separate documented operation.
