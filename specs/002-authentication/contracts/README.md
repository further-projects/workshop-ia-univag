# Authentication Contracts

`auth.openapi.yaml` defines the stable browser-facing authentication facade exposed by the NestJS API.
The implementation may call Better Auth internally, but native Better Auth response bodies and routes
are not public product contracts.

Contract rules:

- Browser authentication uses one opaque host-only `HttpOnly` cookie, never bearer tokens.
- Registration, login and session responses never include session identifiers.
- Duplicate registration, invalid credentials, inactive accounts and temporary blocks use the generic
  error semantics documented in OpenAPI.
- Every authentication response is non-cacheable.
- Any private operation may renew the cookie after the 24-hour update interval.
- Mutating operations require a trusted request origin.
- Shared Zod schemas in `packages/schemas` are the executable source for request and response shapes;
  OpenAPI changes must remain synchronized through contract tests.
