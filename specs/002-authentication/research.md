# Research: Authentication

## Better Auth boundary

**Decision**: Use Better Auth 1.6.27 with `@better-auth/mongo-adapter` 1.6.27 behind four application
endpoints. Do not expose Better Auth's native routes or response bodies directly to the browser.

**Rationale**: Better Auth owns credential verification, password hashes, sessions and cookie creation,
but its native email sign-in response includes a session token and does not implement account lockout.
The facade preserves the library's security-sensitive behavior while enforcing the product contract and
removing tokens before serialization.

**Alternatives considered**: Exposing native routes was rejected because it leaks a token-shaped value
to browser code and couples the product contract to the library. Reimplementing authentication was
rejected because it duplicates mature password and session behavior.

## MongoDB adapter and transactions

**Decision**: Give the official adapter a native MongoDB 7 client connected to MongoDB 8. Run local and
test MongoDB as a single-node replica set and enable adapter transactions. Provision indexes explicitly
because the adapter does not own application deployment indexes.

**Rationale**: Better Auth documents the native adapter and can use transactions when it receives a
Mongo client. A single-node replica set keeps the foundation's one-container topology while supporting
atomic multi-document registration and realistic integration tests.

**Alternatives considered**: Mongoose integration was rejected by the global persistence decision.
Standalone MongoDB was rejected because it cannot provide transactions. A second database was rejected
as unnecessary.

## Password policy and hashing

**Decision**: Configure email/password authentication with automatic sign-in, no email verification,
and lengths from 8 through 128 characters without composition rules. Override hashing with Argon2id via
`@node-rs/argon2` 2.0.2, using 64 MiB memory, three iterations, four lanes and 32-byte output. Preprocess
the password with a server-side pepper before hash and verification.

**Rationale**: Better Auth supports custom hash/verify functions and explicit minimum and maximum
lengths. The Argon2id settings match the current Better Auth example and the approved project decision.
The library creates a unique salt for each encoded hash.

**Alternatives considered**: Better Auth's default scrypt was rejected because Argon2id is already an
approved project decision. Composition rules were rejected by clarification. Client-side hashing was
rejected because it would turn the derived value into a reusable password equivalent.

## Pepper lifecycle

**Decision**: Prefix each stored password hash with a non-secret pepper key identifier. New hashes use
the current pepper; verification resolves the identifier against current and explicitly retained prior
peppers supplied by the secret manager. Rotation changes the current identifier while retaining old
values until affected credentials are migrated or reset. Deployment must fail if a referenced pepper is
missing; no pepper value or derived password may be logged.

**Rationale**: A versioned key ring permits controlled rotation and rollback without storing peppers in
MongoDB. Recovery consists of restoring the matching secret version, not weakening verification.

**Alternatives considered**: One unversioned pepper was rejected because rotation would invalidate all
credentials. Storing the pepper with hashes was rejected because it removes the separation benefit.
Automatic silent retirement was rejected because this MVP has no password-recovery flow.

## Email normalization and duplicate registration

**Decision**: Shared Zod input schemas trim and lowercase email before either operation. A unique index
on normalized `user.email` is the final concurrency authority. Duplicate registration maps to the same
generic `REGISTRATION_FAILED` response regardless of whether detected before insertion or by the index.
The facade performs dummy Argon2id work when a duplicate can be rejected before Better Auth hashes a
password, then delays both successful and duplicate responses until a 750 ms minimum duration measured
from validated request receipt. The internal Better Auth `name` field receives the normalized email
because display names are outside the MVP and the field is never part of the public contract.

**Rationale**: Normalization and a database constraint prevent case, whitespace and concurrent
duplicates. Generic mapping satisfies the clarified behavior without pretending that duplicate
registration succeeded. Equivalent password work and a common response floor reduce the timing gap.
Integration tests compare latency distributions with a generous environment-independent tolerance.

**Alternatives considered**: A pre-insert lookup alone was rejected because concurrent requests race.
Returning conflict details was rejected because it enumerates accounts. Simulating success was rejected
by clarification. Adding a display-name field was rejected as out of scope.

## Account lockout and enumeration resistance

**Decision**: Store one `AuthAttemptState` document keyed by an HMAC of the normalized email for both
existing and nonexistent identities. Acquire a short MongoDB lease on that document before login,
serialize verification for the identity, increment failures atomically, and set a fixed 15-minute
`blockedUntil` on the fifth failure. Attempts while blocked do not mutate the deadline. Successful login
resets the state. Unknown identities execute a dummy Argon2id verification and follow the same state
transitions and public `401`/`429` shapes.

**Rationale**: Better Auth's native sign-in has no lockout point. The persisted lease closes the race in
which a valid concurrent request could create a session after the fifth failure, works across API
replicas, and prevents a five-request status oracle for account existence. HMAC avoids retaining raw
unknown email addresses.

**Alternatives considered**: Hooks alone were rejected because no public hook sits atomically between
password verification and session creation. Fields only on existing users were rejected because their
different lockout behavior would reveal account existence. An in-memory mutex was rejected because it
does not survive restarts or coordinate replicas.

## Rate limiting by origin

**Decision**: Use `@nestjs/throttler` 6.5.0 with a MongoDB-backed storage implementation and a tracker
derived only from Fastify's trusted client IP. The login route allows 20 attempts in 60 seconds and
blocks the tracker for 60 seconds after the excess attempt. Store an HMAC tracker rather than a raw IP.
Honor forwarded addresses only from explicitly configured trusted proxies.

**Rationale**: NestJS Throttler supports route overrides, block duration and proxy-aware trackers. A
shared store keeps the requirement correct across replicas and restarts; trusted-proxy configuration
prevents spoofed forwarding headers.

**Alternatives considered**: In-memory storage was rejected for distributed inconsistency. Rate
limiting by user was rejected because authentication has not happened. Blindly trusting `X-Forwarded-For`
was rejected because clients can forge it.

## Stateful session and cookie policy

**Decision**: Persist revocable sessions in MongoDB with `expiresIn` of seven days and `updateAge` of
24 hours. Disable cookie session cache so every private operation observes revocation and `isActive`.
Use an opaque `mini_notion.session` host-only cookie with `HttpOnly`, `Path=/`, no `Domain`, `Secure` in
production and `SameSite=Lax` for the approved same-origin/proxy deployment. The session guard resolves
the Better Auth session and an interceptor forwards any renewal `Set-Cookie` header on every private
response, not only `/auth/session`. All auth responses use `Cache-Control: no-store`.

**Rationale**: Better Auth documents sliding expiration with these settings. Avoiding cookie cache is
required for immediate logout and user deactivation. A host-only cookie preserves local HTTP
development compatibility; the interceptor keeps browser and persisted expiration synchronized during
ordinary private activity.

**Alternatives considered**: Browser bearer tokens and refresh tokens were rejected by project
decisions. Cookie cache was rejected because a revoked session could remain authoritative temporarily.
Cross-site cookies were rejected because the architecture prefers one origin.

## CSRF, CORS and private authorization

**Decision**: Proxy browser requests through the web origin whenever possible. Configure exact trusted
origins and credentialed CORS only where needed. Use `@fastify/cors` 11.3.0 with Fastify 5; never use
`origin: true` with credentials because it reflects arbitrary request origins. Keep strict preflight
validation enabled. Better Auth origin checks remain enabled; the facade also rejects mutating requests
whose `Origin` is absent or untrusted. A NestJS guard resolves the authoritative session from MongoDB,
verifies `user.isActive`, and attaches only the authenticated user identity. Resource ownership remains
the responsibility of each domain operation.

**Rationale**: SameSite cookies reduce but do not replace CSRF validation. Server-side session and
ownership checks satisfy the constitution; hiding UI is not authorization.

**Alternatives considered**: Wildcard credentialed CORS was rejected as invalid and unsafe. Next.js
middleware as the sole authorization layer was rejected because it cannot authorize API resources.

## Fastify cookie integration

**Decision**: Use `@fastify/cookie` 11.1.2 with Fastify 5 and register parsing on `onRequest` before any
hook that depends on cookies. Better Auth remains responsible for the opaque session value; the Fastify
bridge preserves `HttpOnly`, `Path=/`, `SameSite=Lax`, host-only scope and `Secure` outside local HTTP
development. Cookie values and signing material are never logged.

**Rationale**: The official plugin documentation defines early hook registration and secure serialization
attributes. Explicit options keep parsing available before authentication while preserving the session
policy selected for the same-origin deployment.

**Alternatives considered**: Manual Cookie header parsing was rejected because the maintained Fastify
plugin provides framework-compatible parsing and serialization. `SameSite=None` was rejected because
the deployment does not require cross-site cookies and that mode requires `Secure` in every environment.

## Error and audit semantics

**Decision**: Expose stable codes from the OpenAPI contract. Duplicate registration is generic; unknown
email, wrong password and inactive account share `INVALID_CREDENTIALS`; account and origin blocks share
`AUTH_TEMPORARILY_UNAVAILABLE` plus `Retry-After`. Audit structured event names, outcome, request ID,
HMAC subject/tracker and timestamp, never password, cookie, token, raw hash or raw unknown email.

**Rationale**: Stable generic errors are testable and reduce enumeration. Pseudonymous audit fields
support incident correlation with less personal data exposure.

**Alternatives considered**: Detailed authentication errors were rejected for information leakage.
Logging request bodies was rejected because they contain credentials.

## Validation strategy

**Decision**: Follow Red-Green-Refactor and enforce 100% coverage on auth, authorization and shared
schemas. Use unit tests for schemas, hashing, state transitions and error mapping; MongoDB integration
tests for indexes, transactions, leases and concurrent requests; contract tests for every OpenAPI
response; Playwright for registration, login, private redirect, lockout, expiry and logout. Include
tests that private operations propagate eligible renewal cookies and that successful and duplicate
registration meet the common 750 ms response floor without a stable timing separation.

**Rationale**: Coverage is constitutionally mandatory in these critical domains, while concurrency and
cookie behavior require tests beyond isolated units.

**Alternatives considered**: Mock-only persistence tests were rejected because they cannot prove unique
indexes, transactions or atomic updates. E2E-only coverage was rejected because security branches would
remain hard to diagnose.

## Sources reviewed

- Context7 `/better-auth/better-auth/v1.6.23`: email/password options, custom Argon2 hashing, Fastify
  bridge, MongoDB adapter, session expiration/update age and authoritative stateful session behavior.
- Context7 `/nestjs/throttler`: route-specific limits, block duration and Fastify trusted-proxy tracker.
- Context7 `/fastify/fastify-cors`: Fastify 5 compatibility for version 11.3.0, exact allowlist matching,
  credentialed responses, strict preflight behavior and the risk of reflecting arbitrary origins.
- Context7 `/fastify/fastify-cookie`: `onRequest` parsing, secret rotation support and secure cookie
  attributes including `HttpOnly`, `Secure`, `SameSite`, `Path` and host scoping.
- npm registry metadata queried on 2026-08-12: Better Auth and MongoDB adapter 1.6.27,
  `@node-rs/argon2` 2.0.2, NestJS Throttler 6.5.0, Fastify CORS 11.3.0 and Fastify Cookie 11.1.2.

No `NEEDS CLARIFICATION` items remain.
