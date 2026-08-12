# Contracts: Project Foundation

`health.openapi.yaml` defines the two public readiness interfaces delivered by this feature:

- Web origin `GET /api/health`
- API origin `GET /health`

The OpenAPI document combines paths from two local origins for contract review. Implementations and
tests must use the shared Zod schema in `packages/schemas` as the runtime source of truth and keep it
semantically equivalent to this document.

The contracts intentionally exclude diagnostic details. A failed check may identify the stable check
name and `down` status, but must not expose exceptions, credentials, connection strings, internal host
details, or stack traces.
