# Tasks: Authentication

**Input**: Design documents from `/specs/002-authentication/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Authentication, authorization and shared schemas follow Red-Green-Refactor and require 100%
coverage for statements, branches, functions and lines. Every test task must fail for the intended reason
before its production task begins.

**Organization**: Tasks are grouped by user story so each increment remains independently testable.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files and has no dependency on incomplete tasks
- **[Story]**: Maps a task to its user story (`US1` through `US4`)
- Every task includes an exact file path

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add governed dependencies and runtime configuration required by all authentication stories.

- [ ] T001 Record Context7 compatibility and security evidence for every direct authentication dependency in specs/002-authentication/research.md, then add the approved Better Auth, MongoDB adapter, MongoDB driver, Argon2, NestJS Throttler, Fastify CORS and Fastify Cookie versions to pnpm-workspace.yml and apps/api/package.json
- [ ] T002 [P] Add placeholder-only auth, pepper key-ring, HMAC, trusted-origin and trusted-proxy variables to .env.example and extend validation in apps/api/src/config/environment.ts
- [ ] T003 Configure MongoDB 8 as a single-node replica set with health and initialization behavior in compose.yml and scripts/init-mongo-replica-set.sh
- [ ] T004 [P] Add authentication coverage thresholds for statements, branches, functions and lines to apps/api/vitest.config.ts, apps/web/vitest.config.ts and packages/schemas/vitest.config.ts
- [ ] T005 [P] Add authentication integration and browser test scripts to apps/api/package.json, apps/web/package.json and package.json

**Checkpoint**: Dependencies, secrets contract, replica-set storage and mandatory quality gates are ready.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared contracts and security infrastructure that block every user story.

**CRITICAL**: No user story implementation begins until this phase is complete.

### Tests First

- [ ] T006 [P] Write failing shared-schema and OpenAPI drift tests for normalized email, password policy, confirmation equality, strict objects, public session shape and discriminated errors in packages/schemas/tests/auth.test.ts and apps/api/test/auth/auth-openapi.contract.spec.ts
- [ ] T007 [P] Write failing unit tests for Argon2id, versioned peppers, Better Auth email/password settings, automatic sign-in and stateful session configuration in apps/api/src/modules/auth/password.service.spec.ts and apps/api/src/modules/auth/auth.config.spec.ts
- [ ] T008 [P] Write failing integration tests for Better Auth MongoDB transactions, required user/account/session indexes and base AuthAttemptState persistence/reset in apps/api/test/auth/auth-database.integration.spec.ts
- [ ] T009 [P] Define audit event names, fields, outcomes and denylisted secrets in docs/authentication-operations.md, then write failing bootstrap and audit tests in apps/api/test/auth/auth-bootstrap.integration.spec.ts and apps/api/src/modules/auth/auth-audit.service.spec.ts

### Shared Implementation

- [ ] T010 Implement and export Zod registration, login, current-session, validation-issue and error schemas in packages/schemas/src/auth.ts and packages/schemas/src/index.ts
- [ ] T011 Implement versioned-pepper Argon2id hashing and verification without secret logging in apps/api/src/modules/auth/password.service.ts
- [ ] T012 Configure the native MongoDB client, transaction-capable Better Auth adapter, email/password policy, isActive extension and stateful session options in apps/api/src/infra/mongodb/mongo.client.ts and apps/api/src/modules/auth/auth.config.ts
- [ ] T013 Provision Better Auth indexes and implement HMAC-keyed base AuthAttemptState persistence with successful reset in apps/api/src/infra/mongodb/auth-indexes.ts and apps/api/src/modules/auth/auth-attempt.repository.ts
- [ ] T014 Implement exact credentialed CORS, Fastify Cookie, trusted-proxy extraction and request security in apps/api/src/main.ts and implement no-store, trusted-origin and redaction helpers plus pseudonymous audit events in apps/api/src/modules/auth/auth-security.ts and apps/api/src/modules/auth/auth-audit.service.ts
- [ ] T015 Register the authentication module and shared infrastructure in apps/api/src/modules/auth/auth.module.ts and apps/api/src/app.module.ts

**Checkpoint**: Shared schemas, password handling, persistence and request security pass their tests with 100% coverage.

---

## Phase 3: User Story 1 - Create an account (Priority: P1) MVP

**Goal**: Allow a visitor to register with normalized email, password and confirmation, receive a safe
cookie session and reach the private home page without another login.

**Independent Test**: Submit valid and invalid registrations, duplicate normalized emails and concurrent
duplicates; verify exactly one account, automatic session, generic duplicate error, 750 ms response floor,
no token in JSON/browser storage and no confirmation or secret in persistence/logs.

### Tests for User Story 1

- [ ] T016 [P] [US1] Write failing OpenAPI contract tests for POST /api/auth/register and GET /api/auth/session success, validation, generic duplicate, unauthenticated and untrusted-origin responses in apps/api/test/auth/register.contract.spec.ts
- [ ] T017 [P] [US1] Write failing MongoDB integration tests for normalized uniqueness, confirmation non-persistence, rollback, concurrent duplicates, dummy hash, timing equivalence, audit emission and secret redaction in apps/api/test/auth/register.integration.spec.ts
- [ ] T018 [P] [US1] Write failing component and route tests for registration validation, field errors, successful navigation and unauthenticated private-home redirect in apps/web/tests/auth/register-form.spec.tsx and apps/web/tests/auth/private-layout.spec.tsx
- [ ] T019 [P] [US1] Write the failing registration browser journey for automatic session and private-page arrival in tests/e2e/auth-register.spec.ts

### Implementation for User Story 1

- [ ] T020 [US1] Implement transactional registration with confirmation stripped before the adapter boundary, internal name derivation, duplicate mapping, dummy Argon2 work, timing equalization, token-free output and audit emission in apps/api/src/modules/auth/auth.service.ts
- [ ] T021 [US1] Expose POST /api/auth/register and the minimal authoritative GET /api/auth/session needed by the private home, with validation, origin checks, no-store headers and opaque cookie forwarding in apps/api/src/modules/auth/auth.controller.ts
- [ ] T022 [P] [US1] Implement the registration API client and typed error mapping in apps/web/src/lib/auth/register.ts
- [ ] T023 [US1] Build the accessible registration form against the registration client using approved shadcn/ui primitives in apps/web/src/components/auth/register-form.tsx
- [ ] T024 [US1] Add the registration route plus a concrete server-protected private home destination in apps/web/src/app/(auth)/register/page.tsx, apps/web/src/app/(private)/layout.tsx and apps/web/src/app/(private)/page.tsx

**Checkpoint**: User Story 1 is independently usable and all registration tests pass at mandatory coverage.

---

## Phase 4: User Story 2 - Sign in (Priority: P1)

**Goal**: Allow an active registered user to sign in with email and password while giving the same
observable error for unknown email, wrong password and inactive account.

**Independent Test**: Valid credentials start a cookie session and open the private home page; unknown
email, wrong password and inactive account return the same status, code, body shape and token-free response.

### Tests for User Story 2

- [ ] T025 [P] [US2] Write failing OpenAPI contract tests for POST /api/auth/login success, validation, generic credentials and untrusted-origin responses in apps/api/test/auth/login.contract.spec.ts
- [ ] T026 [P] [US2] Write failing integration tests for active login, dummy verification, inactive denial, status/body and latency-distribution equivalence among invalid cases, persisted attempt reset and audit events in apps/api/test/auth/login.integration.spec.ts
- [ ] T027 [P] [US2] Write failing component tests for login validation, generic errors and successful navigation in apps/web/tests/auth/login-form.spec.tsx
- [ ] T028 [P] [US2] Write the failing login browser journey for valid and invalid credentials in tests/e2e/auth-login.spec.ts

### Implementation for User Story 2

- [ ] T029 [US2] Implement token-free login orchestration, active-user enforcement, dummy verification, generic credential mapping, foundational AuthAttemptState reset and audit emission in apps/api/src/modules/auth/auth.service.ts
- [ ] T030 [US2] Expose POST /api/auth/login with Zod validation, origin checks, no-store headers and opaque cookie forwarding in apps/api/src/modules/auth/auth.controller.ts
- [ ] T031 [P] [US2] Implement the login API client and typed generic-error mapping in apps/web/src/lib/auth/login.ts
- [ ] T032 [US2] Build the accessible login form against the login client using approved shadcn/ui primitives in apps/web/src/components/auth/login-form.tsx
- [ ] T033 [US2] Add the public login route and successful private-home navigation in apps/web/src/app/(auth)/login/page.tsx

**Checkpoint**: User Story 2 signs active users in without exposing account existence or session tokens.

---

## Phase 5: User Story 3 - Contain repeated attempts (Priority: P1)

**Goal**: Block an identity for a fixed 15 minutes after five consecutive failures and limit each trusted
origin to 20 login attempts per minute with a one-minute non-extending block.

**Independent Test**: Five failures lock existing and unknown identities equivalently; the correct password
cannot authenticate during the fixed deadline; concurrent attempts cannot bypass it; the 21st origin attempt
is rejected before Argon2 and forged forwarding headers do not change the tracker.

### Tests for User Story 3

- [ ] T034 [P] [US3] Extend failing AuthAttemptState tests for fixed blocking deadlines, lease expiry, concurrent HMAC subjects, Retry-After and block audit events in apps/api/src/modules/auth/auth-attempt.repository.spec.ts
- [ ] T035 [P] [US3] Write failing MongoDB integration tests for atomic increments, cross-replica leases, five simultaneous failures, fifth-failure versus valid-login ordering and nonexistent-identity equivalence in apps/api/test/auth/auth-lockout.integration.spec.ts
- [ ] T036 [P] [US3] Write failing tests for MongoDB-backed 20-per-minute origin tracking, one-minute fixed block, trusted-proxy behavior and proof that the 21st/blocked requests never invoke Argon2 in apps/api/test/auth/auth-rate-limit.integration.spec.ts
- [ ] T037 [P] [US3] Write the failing browser journey for lockout, unchanged deadline and post-expiry login in tests/e2e/auth-lockout.spec.ts

### Implementation for User Story 3

- [ ] T038 [US3] Extend the foundational AuthAttemptState repository with leases, atomic failure transitions, fixed blocking and block audit emission in apps/api/src/modules/auth/auth-attempt.repository.ts
- [ ] T039 [US3] Integrate serialized identity lockout before session creation and return generic audited 401/429 responses with Retry-After in apps/api/src/modules/auth/auth.service.ts
- [ ] T040 [US3] Implement MongoDB-backed NestJS Throttler storage and HMAC trusted-IP tracker in apps/api/src/modules/auth/auth-throttler.storage.ts
- [ ] T041 [US3] Apply the 20-per-minute limit and one-minute non-extending origin block to POST /api/auth/login in apps/api/src/modules/auth/auth.controller.ts
- [ ] T042 [US3] Add generic temporary-unavailability handling without exposing counters or deadlines in apps/web/src/components/auth/login-form.tsx

**Checkpoint**: User Story 3 resists sequential, concurrent and distributed brute-force attempts without an account oracle.

---

## Phase 6: User Story 4 - Maintain and end the session (Priority: P2)

**Goal**: Authorize private operations through an authoritative stateful session, renew active sessions,
redirect unauthenticated visitors and make logout or deactivation effective immediately.

**Independent Test**: Missing, altered, expired, revoked and inactive-user sessions are rejected; eligible
activity forwards a renewed cookie; seven inactive days expire access; logout is idempotent and replay fails;
another user's resource remains forbidden even with a valid session.

### Tests for User Story 4

- [ ] T043 [P] [US4] Write failing contract and authorization tests for GET /api/auth/session, POST /api/auth/logout and reusable resource ownership denial, including optional-cookie logout, renewal headers and generic 401/403 errors in apps/api/test/auth/session.contract.spec.ts and apps/api/src/modules/auth/resource-ownership.service.spec.ts
- [ ] T044 [P] [US4] Write failing unit tests for authoritative session guard, active-user enforcement and renewal-header interceptor in apps/api/src/modules/auth/auth-session.guard.spec.ts and apps/api/src/modules/auth/auth-session.interceptor.spec.ts
- [ ] T045 [P] [US4] Write failing integration tests for 24-hour renewal, seven-day inactivity, altered/expired/revoked cookies, immediate logout, deactivation, replay and session/logout audit events in apps/api/test/auth/session.integration.spec.ts
- [ ] T046 [P] [US4] Extend failing web tests for renewal-aware private layout and logout behavior in apps/web/tests/auth/private-layout.spec.tsx and apps/web/tests/auth/logout-button.spec.tsx
- [ ] T047 [P] [US4] Write the failing browser journey for renewal, logout, replay denial and private redirect in tests/e2e/auth-session.spec.ts

### Implementation for User Story 4

- [ ] T048 [US4] Implement authoritative session resolution, inactive-user denial, authenticated-user attachment and reusable server-side ownership assertion in apps/api/src/modules/auth/auth-session.guard.ts and apps/api/src/modules/auth/resource-ownership.service.ts
- [ ] T049 [P] [US4] Implement eligible Better Auth Set-Cookie renewal propagation on private responses in apps/api/src/modules/auth/auth-session.interceptor.ts
- [ ] T050 [US4] Complete GET /api/auth/session and implement audited idempotent POST /api/auth/logout with no-store, Vary and cookie-clearing semantics in apps/api/src/modules/auth/auth.controller.ts and apps/api/src/modules/auth/auth.service.ts
- [ ] T051 [P] [US4] Implement server-side current-session and logout clients in apps/web/src/lib/auth/session.ts
- [ ] T052 [US4] Build the accessible logout control against the session client using approved shadcn/ui primitives in apps/web/src/components/auth/logout-button.tsx
- [ ] T053 [US4] Extend private-page protection with renewal-aware server-side session resolution and login redirect in apps/web/src/app/(private)/layout.tsx
- [ ] T054 [US4] Write the failing ownership integration test against the planned private resource fixture in apps/api/test/auth/resource-ownership.integration.spec.ts
- [ ] T055 [US4] Implement the test-only private resource fixture with the reusable ownership assertion in apps/api/test/auth/fixtures/private-resource.controller.ts

**Checkpoint**: User Story 4 provides sliding stateful sessions, immediate revocation and server-side authorization.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verify security, observability, contracts and complete-system behavior across all stories.

- [ ] T056 [P] Extend docs/authentication-operations.md with pepper rotation, rollback and retained-key recovery procedures
- [ ] T057 [P] Add failing p95 assertions over at least 100 warmed operations at up to 10 concurrent requests, requiring session and other non-Argon2 responses to remain at or below 200 ms and registration/login at or below 1.5 s in apps/api/test/auth/auth-performance.integration.spec.ts
- [ ] T058 [P] Run and reconcile the existing OpenAPI-versus-Zod drift suite for every authentication request, response and error code in apps/api/test/auth/auth-openapi.contract.spec.ts
- [ ] T059 Run existing story-level leakage assertions and add any newly discovered failing regression before its fix in apps/api/test/auth/auth-secret-leakage.integration.spec.ts and tests/e2e/auth-secret-leakage.spec.ts
- [ ] T060 Run the full validation sequence from specs/002-authentication/quickstart.md and record any required command correction in specs/002-authentication/quickstart.md
- [ ] T061 Run format, lint, type-check, 100% auth/schema coverage, performance, integration, build and E2E gates using the root scripts defined in package.json

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Starts immediately and establishes dependencies, configuration and test commands.
- **Foundational (Phase 2)**: Depends on Phase 1 and blocks every user story.
- **User Story 1 (Phase 3)**: Depends on Phase 2 and is the recommended first deliverable.
- **User Story 2 (Phase 4)**: API tests and clients depend on Phase 2; its private-home browser navigation depends on T024, and shared auth service/controller edits are serialized after US1.
- **User Story 3 (Phase 5)**: Depends on the US2 login orchestration because lockout wraps credential verification and session issuance.
- **User Story 4 (Phase 6)**: Guard/interceptor tests depend on Phase 2; controller completion depends on T021, and browser logout validation needs a session from US1 or US2.
- **Polish (Phase 7)**: Depends on all selected user stories.

### User Story Dependency Graph

```text
Setup -> Foundational -> US1
                    |-> US2 -> US3
                    |-> US4
US1 + US2 + US3 + US4 -> Polish
```

### Within Each User Story

- Write tests first and confirm they fail for the expected missing behavior.
- Implement persistence and service behavior before exposing controller behavior.
- Implement API clients before connecting page-level UI.
- Pass the story-specific tests and 100% critical-domain coverage before its checkpoint.

### Parallel Opportunities

- T002, T004 and T005 can run in parallel after T001 identifies the final manifests.
- T006 through T009 can run in parallel, followed by shared implementation in separate files where marked.
- After Phase 2, non-overlapping US1, US2 and US4 tests/clients can begin in parallel; edits to shared auth service/controller files remain sequential.
- Contract, integration, component and E2E test tasks within each story can be authored in parallel.
- Web clients/components marked `[P]` can proceed while API service behavior is implemented against the contract.
- T056 through T058 can run in parallel after all story behavior exists.

---

## Parallel Examples

### User Story 1

```text
Task T016: Register OpenAPI contract tests in apps/api/test/auth/register.contract.spec.ts
Task T017: Registration transaction and timing tests in apps/api/test/auth/register.integration.spec.ts
Task T018: Registration component tests in apps/web/tests/auth/register-form.spec.tsx
Task T019: Registration browser journey in tests/e2e/auth-register.spec.ts
```

### User Story 2

```text
Task T025: Login OpenAPI contract tests in apps/api/test/auth/login.contract.spec.ts
Task T026: Login integration and generic-error tests in apps/api/test/auth/login.integration.spec.ts
Task T027: Login component tests in apps/web/tests/auth/login-form.spec.tsx
Task T028: Login browser journey in tests/e2e/auth-login.spec.ts
```

### User Story 3

```text
Task T034: Lockout state unit tests in apps/api/src/modules/auth/auth-attempt.repository.spec.ts
Task T035: Concurrent lockout integration tests in apps/api/test/auth/auth-lockout.integration.spec.ts
Task T036: Origin limit integration tests in apps/api/test/auth/auth-rate-limit.integration.spec.ts
Task T037: Lockout browser journey in tests/e2e/auth-lockout.spec.ts
```

### User Story 4

```text
Task T043: Session and logout contract tests in apps/api/test/auth/session.contract.spec.ts
Task T044: Guard and interceptor unit tests in apps/api/src/modules/auth/
Task T045: Session lifecycle integration tests in apps/api/test/auth/session.integration.spec.ts
Task T046: Private-layout and logout web tests in apps/web/tests/auth/
Task T047: Session browser journey in tests/e2e/auth-session.spec.ts
```

---

## Implementation Strategy

### MVP First: User Story 1

1. Complete Setup and Foundational phases.
2. Complete User Story 1 using Red-Green-Refactor.
3. Stop and validate registration independently, including concurrency, timing and session safety.
4. Demonstrate account creation and direct arrival at the private home page.

### Incremental Delivery

1. Setup + Foundational establish shared security and contracts.
2. US1 delivers registration and automatic session.
3. US2 adds recurring login without account enumeration.
4. US3 hardens login against repeated and distributed attempts.
5. US4 completes private access, renewal and logout.
6. Polish verifies cross-story security and all repository gates.

### Parallel Team Strategy

1. Complete Setup and Foundational together.
2. Assign non-overlapping US1, US2 and US4 tests/clients to separate owners while one owner serializes shared service/controller changes.
3. Start US3 after US2's service boundary is stable.
4. Merge only after each story checkpoint passes its independent tests and coverage gates.

## Notes

- `[P]` means file-level parallelism without an incomplete dependency, not merely work that could overlap.
- Better Auth owns credential and session records; do not introduce Mongoose mirrors.
- Never expose or log passwords, confirmations, peppers, hashes, cookies or session tokens.
- Keep OpenAPI, shared Zod schemas, web behavior and API behavior synchronized.
- Commit only after a test-first logical group passes its applicable gates.
