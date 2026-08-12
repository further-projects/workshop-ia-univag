# Tasks: Document Management

**Input**: Design documents from `/specs/003-document-management/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Document management, authorization and shared schemas follow Red-Green-Refactor and require
100% coverage for statements, branches, functions and lines. Every test task must fail for the intended
missing behavior before its production task begins.

**Organization**: Tasks are grouped by user story. US1 is the navigation foundation; US2–US4 build on
it but retain independent fixtures and acceptance tests for their own behavior.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files and has no dependency on incomplete tasks
- **[Story]**: Maps a task to its user story (`US1` through `US4`)
- Every task includes exact file paths

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add governed document dependencies, runtime configuration and mandatory quality gates.

- [ ] T001 Record exact-version Context7 and registry compatibility/security evidence in specs/003-document-management/research.md, add Mongoose 9.9.2 and @nestjs/mongoose 11.0.4 to pnpm-workspace.yml and apps/api/package.json, update pnpm-lock.yaml, then run pnpm audit --audit-level high against the resulting graph and block or document approved scoped exceptions for applicable high/critical advisories
- [ ] T002 [P] Write failing tests for the required cursor HMAC secret and exact 1,250,000-byte document body-limit environment contract in apps/api/src/config/environment.spec.ts
- [ ] T003 Add document integration, contract, performance, component, E2E, all-files coverage and security-audit scripts to apps/api/package.json, apps/web/package.json and package.json after T001 updates apps/api/package.json
- [ ] T004 [P] Configure explicit all-files include globs and 100% statements, branches, functions and lines for document API/web domain files, document authorization and shared document schemas in apps/api/vitest.config.ts, apps/web/vitest.config.ts and packages/schemas/vitest.config.ts
- [ ] T005 Write failing tests for the @nestjs/mongoose connection plus strict-query and filter-sanitization settings in apps/api/src/infra/mongodb/mongoose.config.spec.ts after T001 installs the governed dependencies

**Checkpoint**: Dependencies, configuration and mandatory test gates are available without introducing product behavior.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared validation, persistence, authorization and HTTP behavior required by every story.

**CRITICAL**: No user story implementation begins until this phase is complete.

### Tests First

- [ ] T006 [P] Write failing shared-schema boundary tests for safe recursive JSON, root/node/depth counting, prototype-key rejection, 200 Unicode code points after trim, exactly 1,000,000 UTF-8 bytes, deterministic field/code/limit/unit violations, strict DTOs, pages, problems and ETags in packages/schemas/tests/documents.test.ts
- [ ] T007 [P] Write failing OpenAPI parse, internal-reference and Zod-drift tests for every document request, response, status, media type, stable code and header in apps/api/test/documents/documents-openapi.contract.spec.ts
- [ ] T008 [P] Write failing unit tests for structured-text extraction, NFKD case/accent normalization, whitespace, literal regex escaping and search derivative generation from shared-Zod-normalized title/query inputs in apps/api/src/modules/documents/document-search.spec.ts
- [ ] T009 [P] Write failing unit tests for signed versioned cursors bound to owner, normalized query and limit plus malformed/tampered criteria in apps/api/src/modules/documents/document-cursor.spec.ts
- [ ] T010 [P] Write failing MongoDB integration tests for server-owned fields, timestamps, version initialization, strict schema behavior and the exact owner/order index in apps/api/test/documents/document-schema.integration.spec.ts
- [ ] T011 [P] Write failing unit/integration tests for authoritative session attachment, trusted-origin mutation checks, owner-safe ID normalization, owner-bound cursors, private no-store headers, request IDs and problem mapping without requiring document endpoints in apps/api/src/modules/documents/document-security.spec.ts and apps/api/test/documents/documents-security.integration.spec.ts
- [ ] T012 [P] Write failing configuration tests for missing cursor secrets, exact normal/chunked 1,250,000-byte acceptance and 1,250,001-byte rejection in apps/api/test/documents/documents-bootstrap.integration.spec.ts

### Shared Implementation

- [ ] T013 Implement and export recursive content, create/update, list/page, ETag, violation and discriminated problem Zod schemas in packages/schemas/src/documents.ts and packages/schemas/src/index.ts
- [ ] T014 Implement text extraction, Unicode search normalization and escaped literal matching helpers while consuming shared Zod-normalized document inputs in apps/api/src/modules/documents/document-search.ts
- [ ] T015 Implement HMAC-signed opaque cursor encoding/decoding with owner, query and limit binding in apps/api/src/modules/documents/document-cursor.ts
- [ ] T016 Implement the strict Mongoose Document schema, explicit version/timestamps, private projections and compound owner/order index declaration in apps/api/src/modules/documents/document.schema.ts
- [ ] T017 Implement stable problem mapping, owner-safe identifier parsing and private response headers in apps/api/src/modules/documents/document-security.ts
- [ ] T018 Implement the tested cursor-secret/body-limit environment contract and Mongoose security settings, configure exact route body limits plus auth/origin protection, and register the document module in apps/api/src/config/environment.ts, apps/api/src/infra/mongodb/mongoose.config.ts, apps/api/src/modules/documents/documents.module.ts and apps/api/src/app.module.ts

**Checkpoint**: Shared schemas, storage model, cursors and security behavior pass tests with mandatory coverage.

---

## Phase 3: User Story 1 - View and create documents (Priority: P1) MVP

**Goal**: Show only the authenticated user's documents in recent-modification order, distinguish the
empty state and create/open a private `Sem título` document in one action.

**Independent Test**: A user with no documents sees the exact empty state, creates once and immediately
opens a canonical empty document; another user's records never appear and failures do not report success.

### Tests for User Story 1

- [ ] T019 [P] [US1] Write failing deployed contract tests for GET /api/documents statuses 200/400/401/422/503 and POST /api/documents statuses 201/400/401/403/413/422/503, asserting JSON media types on body responses, stable codes on problems, private headers on all responses and Location/ETag only on POST 201 in apps/api/test/documents/documents-create-list.contract.spec.ts
- [ ] T020 [P] [US1] Write failing MongoDB integration tests for owner-derived creation, canonical defaults, shared-Zod POST title/content/node/depth boundaries with no persistence on rejection, private list projection, recent-first ordering, pagination, owner isolation and list/create p95 acceptance in apps/api/test/documents/documents-create-list.integration.spec.ts
- [ ] T021 [P] [US1] Write failing component tests for exact empty copy, create action, loading/error recovery, ordered items and edit navigation in apps/web/tests/documents/document-list.spec.tsx
- [ ] T022 [P] [US1] Write the failing browser journey for empty state, one-action creation, immediate editor opening and cross-account list isolation in tests/e2e/documents-create-list.spec.ts

### Implementation for User Story 1

- [ ] T023 [US1] Implement owner-scoped create/list repository operations with lean private projections, keyset ordering and physical persistence defaults in apps/api/src/modules/documents/documents.repository.ts
- [ ] T024 [US1] Implement create/list orchestration by consuming shared Zod-normalized title/content and limit results, applying persistence defaults and derivative generation, and returning ETag/Location or stable persistence problems in apps/api/src/modules/documents/documents.service.ts
- [ ] T025 [US1] Expose GET and POST /api/documents with Zod validation, session/origin checks and private observability headers in apps/api/src/modules/documents/documents.controller.ts
- [ ] T026 [P] [US1] Implement typed create/list document requests and stable problem mapping in apps/web/src/lib/documents/documents-api.ts
- [ ] T027 [US1] Build the accessible empty/list/loading/error states and per-item edit navigation using approved shadcn/ui primitives in apps/web/src/components/documents/document-list.tsx
- [ ] T028 [US1] Add the private documents page and immediate navigation to a newly created document in apps/web/src/app/(private)/documents/page.tsx

**Checkpoint**: User Story 1 is independently usable as the MVP and passes its API, MongoDB, component and browser tests.

---

## Phase 4: User Story 2 - Find a document (Priority: P1)

**Goal**: Search private titles and structured content by literal partial text without case or accent
distinction, with distinct no-result state and stable cursor pagination.

**Independent Test**: Searches for title/content fragments, alternate case/accents and regex characters
return all and only matching owner documents across all pages; clearing the term restores the full list.

### Tests for User Story 2

- [ ] T029 [P] [US2] Write failing contract tests for the 200-code-point trimmed search limit with search/SEARCH_TOO_LONG/200/code_points violation, search/limit/cursor validation, INVALID_CURSOR and paged list output on GET /api/documents in apps/api/test/documents/documents-search.contract.spec.ts
- [ ] T030 [P] [US2] Write failing real-MongoDB tests for title/content substring search, case/accent normalization, literal metacharacters, blank search, owner isolation, stable timestamp ties, complete multi-page retrieval and common/absent search p95 acceptance in apps/api/test/documents/documents-search.integration.spec.ts
- [ ] T031 [P] [US2] Write failing component tests for pending search, preserving current results after an over-limit query, no-match versus no-documents states, URL criteria, cursor continuation and clearing in apps/web/tests/documents/document-search.spec.tsx
- [ ] T032 [P] [US2] Write the failing browser journey for partial title/content search, accents, metacharacters, no results, all pages and clearing in tests/e2e/documents-search.spec.ts

### Implementation for User Story 2

- [ ] T033 [US2] Extend owner-scoped list persistence with escaped normalized substring filtering, limit-plus-one pagination and deterministic updatedAt/id cursor predicates in apps/api/src/modules/documents/documents.repository.ts
- [ ] T034 [US2] Consume the shared Zod 200-code-point trimmed query result and add search orchestration, owner-bound cursor validation and next-cursor generation in apps/api/src/modules/documents/documents.service.ts
- [ ] T035 [US2] Complete GET /api/documents search, limit and cursor contract handling in apps/api/src/modules/documents/documents.controller.ts
- [ ] T036 [P] [US2] Extend the typed document client with normalized URL search criteria and cursor continuation in apps/web/src/lib/documents/documents-api.ts
- [ ] T037 [US2] Build the accessible search control, pending/no-match states and incremental results in apps/web/src/components/documents/document-search.tsx and apps/web/src/components/documents/document-list.tsx
- [ ] T038 [US2] Synchronize search criteria with the private documents page URL and reset cursor chains after criteria changes in apps/web/src/app/(private)/documents/page.tsx

**Checkpoint**: User Story 2 retrieves every matching private document in a stable, independently testable search flow.

---

## Phase 5: User Story 3 - Update a document (Priority: P1)

**Goal**: Load and update a private document with strict limits and atomic optimistic concurrency while
preserving the local draft after validation, conflict, network or persistence failure.

**Independent Test**: The owner persists title/content and receives an advanced version/time; a stale
session receives 412 and retains its draft; invalid, missing and foreign IDs reveal nothing.

### Tests for User Story 3

- [ ] T039 [P] [US3] Write failing deployed contract tests for GET /api/documents/{documentId} statuses 200/401/404/503 and PATCH statuses 200/400/401/403/404/412/413/422/428/503, asserting JSON media types on body responses, stable codes on problems, private headers on all responses, ETag on successful snapshots and If-Match requirements on PATCH in apps/api/test/documents/documents-read-update.contract.spec.ts
- [ ] T040 [P] [US3] Write failing MongoDB integration tests for owner-safe reads, atomic version compare-and-swap, exactly one concurrent winner, timestamp/version advancement, ETag path binding, no mutation on failure and read/update p95 acceptance in apps/api/test/documents/documents-update.integration.spec.ts
- [ ] T041 [P] [US3] Write failing PATCH exact-boundary/no-mutation tests with deterministic title/content violation codes and code_points/bytes/levels/nodes units for title trim, 1,000,000-byte content, 100 levels, 10,000 nodes, safe attributes and derived search replacement in apps/api/test/documents/documents-limits.integration.spec.ts
- [ ] T042 [P] [US3] Write failing client-draft state tests for confirmed snapshots and preservation across validation, stale ETag, network and 503 failures in apps/web/tests/documents/document-draft.spec.ts
- [ ] T043 [P] [US3] Write failing editor component/route tests for load/save states, Sem título normalization, limit messages, retry and explicit conflict reload warning in apps/web/tests/documents/document-editor.spec.tsx
- [ ] T044 [P] [US3] Write the failing two-context browser journey for persistence, cross-account 404, exact limits and stale-update draft preservation in tests/e2e/documents-update.spec.ts

### Implementation for User Story 3

- [ ] T045 [US3] Implement owner-scoped read and atomic update-by-id/owner/version with derivative replacement, version increment and conflict-versus-not-found discrimination in apps/api/src/modules/documents/documents.repository.ts
- [ ] T046 [US3] Implement read/update orchestration by consuming shared Zod-normalized boundary results, validating the ETag path and returning stable conflict/storage problems in apps/api/src/modules/documents/documents.service.ts
- [ ] T047 [US3] Expose GET and PATCH /api/documents/{documentId} with If-Match requirements and owner-safe contract responses in apps/api/src/modules/documents/documents.controller.ts
- [ ] T048 [P] [US3] Extend the typed document client with owner-safe reads, ETag-aware updates and retryable problem handling in apps/web/src/lib/documents/documents-api.ts
- [ ] T049 [P] [US3] Implement confirmed-snapshot/local-draft transitions without autosave in apps/web/src/lib/documents/document-draft.ts
- [ ] T050 [US3] Build the accessible document editor shell with save state, retained draft, retry and conflict reload warning in apps/web/src/components/documents/document-editor.tsx
- [ ] T051 [US3] Add the private document route with owner-safe not-found handling and editor data loading in apps/web/src/app/(private)/documents/[documentId]/page.tsx

**Checkpoint**: User Story 3 saves safely without last-write-wins or silent local-content loss.

---

## Phase 6: User Story 4 - Delete a document (Priority: P2)

**Goal**: Require accessible explicit confirmation, preserve the document on cancellation/failure and
physically remove it before reporting success.

**Independent Test**: Cancel and Escape send no request; successful confirmation removes and makes the
document inaccessible; foreign/missing IDs share 404 and a storage failure leaves a retryable dialog.

### Tests for User Story 4

- [ ] T052 [P] [US4] Write failing deployed contract and MongoDB tests for DELETE /api/documents/{documentId} statuses 204/401/403/404/503, asserting no body/media type on 204, problem media types/codes on errors, private headers, owner isolation, immediate physical deletion, repeated owner-safe 404 and delete p95 acceptance in apps/api/test/documents/documents-delete.integration.spec.ts
- [ ] T053 [P] [US4] Write failing Alert Dialog component tests for document identification, Cancel focus, Escape/cancel without request, duplicate-submit prevention, announced failure and focus restoration in apps/web/tests/documents/delete-document-dialog.spec.tsx
- [ ] T054 [P] [US4] Write the failing keyboard browser journey for cancel, failed retry, confirmed deletion, immediate disappearance and later 404 in tests/e2e/documents-delete.spec.ts

### Implementation for User Story 4

- [ ] T055 [US4] Implement owner-scoped synchronous physical deletion with not-found and storage-failure outcomes in apps/api/src/modules/documents/documents.repository.ts and apps/api/src/modules/documents/documents.service.ts
- [ ] T056 [US4] Expose DELETE /api/documents/{documentId} with session/origin validation and 204 only after confirmed persistence in apps/api/src/modules/documents/documents.controller.ts
- [ ] T057 [P] [US4] Extend the typed document client with delete and stable retryable problem handling in apps/web/src/lib/documents/documents-api.ts
- [ ] T058 [US4] Build the accessible shadcn/ui Alert Dialog deletion flow in apps/web/src/components/documents/delete-document-dialog.tsx
- [ ] T059 [US4] Integrate cancellation, deletion success, list refresh and meaningful post-delete focus into apps/web/src/components/documents/document-list.tsx and apps/web/src/app/(private)/documents/page.tsx

**Checkpoint**: User Story 4 deletes only after explicit accessible confirmation and never exposes ownership.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verify deployment, privacy, performance and complete-system behavior across all stories.

- [ ] T060 [P] Write failing idempotent index-provisioning tests, then add and implement the apps/api/package.json provision:document-indexes command with exact-index verification and deployment-blocking conflict failure in apps/api/test/documents/document-indexes.integration.spec.ts and apps/api/scripts/provision-document-indexes.ts
- [ ] T061 [P] Write failing observability tests for every success/400/401/403/404/412/413/422/428/503 path, bounded labels, required route-template/operation/outcome/status/duration/request-ID log fields, session/MongoDB/serialization spans and exclusion of payloads/raw IDs in apps/api/test/documents/documents-observability.integration.spec.ts
- [ ] T062 Implement and register bounded request-duration, conflict, limit and storage-failure metrics, required structured log fields and payload-free session/MongoDB/serialization spans across apps/api/src/modules/documents/document-observability.ts, apps/api/src/modules/documents/documents.module.ts, apps/api/src/modules/documents/documents.controller.ts, apps/api/src/modules/documents/documents.repository.ts and apps/api/src/modules/auth/auth-session.guard.ts
- [ ] T063 Consolidate the prewritten story performance assertions into the deterministic seven-workload gate and make it Green by tuning apps/api/src/modules/documents/documents.repository.ts and apps/api/src/modules/documents/documents.service.ts, preserving separate p95 assertions, 10-request concurrency and explain("executionStats") evidence in apps/api/test/documents/documents-performance.integration.spec.ts
- [ ] T064 [P] Run and reconcile the OpenAPI-versus-Zod drift suite for every document operation, status, stable code, media type and header in apps/api/test/documents/documents-openapi.contract.spec.ts
- [ ] T065 Consolidate and run the prewritten story security scenarios for foreign IDs, owner-bound cursors and failure leakage in tests/e2e/documents-security.spec.ts and apps/api/test/documents/documents-security.integration.spec.ts, fixing any failure only after adding it to the owning story test in apps/api/test/documents/ or tests/e2e/
- [ ] T066 Run the full validation sequence from specs/003-document-management/quickstart.md, fix discovered document behavior only with a failing regression in the relevant apps/api/test/documents/, apps/web/tests/documents/ or tests/e2e/ suite before changing apps/api/src/modules/documents/ or apps/web/src/, and record required command corrections in specs/003-document-management/quickstart.md
- [ ] T067 Run the final frozen-lockfile security audit plus format, lint, type-check, all-files 100% document/authz/schema coverage, contract, integration, performance, build and E2E gates using the root scripts in package.json

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Starts immediately and establishes dependencies, configuration and test commands.
- **Foundational (Phase 2)**: Depends on Setup and blocks every user story.
- **User Story 1 (Phase 3)**: Depends on Foundational and is the recommended MVP.
- **User Story 2 (Phase 4)**: Depends on the US1 list endpoint/component and uses its own seeded search fixtures.
- **User Story 3 (Phase 5)**: Depends on US1 navigation and shared schemas, but uses directly seeded owner documents and does not depend on search.
- **User Story 4 (Phase 6)**: Depends on the US1 list and shared owner-safe persistence, introduces the delete action itself and does not depend on search or update.
- **Polish (Phase 7)**: Depends on all selected stories.

### User Story Dependency Graph

```text
Setup -> Foundational -> US1 -> US2
                          |--> US3
                          |--> US4
US1 + US2 + US3 + US4 -> Polish
```

### Within Each User Story

- Write tests first and confirm they fail for the expected missing behavior.
- Implement repository behavior before service and controller exposure.
- Implement typed web clients before connecting components and routes.
- Pass story tests and 100% critical-domain coverage before its checkpoint.

### Requirement Traceability

| Requirement | Primary tasks |
|-------------|---------------|
| FR-001–FR-006, FR-016, FR-018–FR-022; SC-001–SC-002, SC-007–SC-008 | T019–T028, T041 |
| FR-007–FR-009, FR-016, FR-019, FR-023; SC-002–SC-004, SC-009 | T029–T038, T063 |
| FR-010–FR-012, FR-016–FR-022; SC-006–SC-008 | T039–T051 |
| FR-006, FR-013–FR-018; SC-005–SC-006 | T052–T059 |
| Constitution coverage, security and operational gates | T001–T018, T060–T067 |

### Parallel Opportunities

- T002 and T004 can run in parallel while T003 and T005 wait for T001 dependency adoption.
- T006 through T012 can be authored in parallel before shared implementation.
- After Foundational and US1, US2, US3 and US4 test work can proceed in parallel in different files.
- Contract, integration, component and E2E tests marked `[P]` within each story can be authored concurrently.
- Web client/state tasks marked `[P]` can proceed against the approved contract while API behavior is implemented.
- T060, T061 and T064 can run in parallel after story behavior exists; T063 waits for T060 to verify the production index.

---

## Parallel Examples

### User Story 1

```text
Task T019: GET/POST document contract tests in apps/api/test/documents/documents-create-list.contract.spec.ts
Task T020: Owner-scoped create/list integration tests in apps/api/test/documents/documents-create-list.integration.spec.ts
Task T021: Empty/list component tests in apps/web/tests/documents/document-list.spec.tsx
Task T022: Create/list browser journey in tests/e2e/documents-create-list.spec.ts
```

### User Story 2

```text
Task T029: Search/cursor contract tests in apps/api/test/documents/documents-search.contract.spec.ts
Task T030: Search/pagination MongoDB tests in apps/api/test/documents/documents-search.integration.spec.ts
Task T031: Search component tests in apps/web/tests/documents/document-search.spec.tsx
Task T032: Search browser journey in tests/e2e/documents-search.spec.ts
```

### User Story 3

```text
Task T039: Read/update contract tests in apps/api/test/documents/documents-read-update.contract.spec.ts
Task T040: Concurrent update integration tests in apps/api/test/documents/documents-update.integration.spec.ts
Task T042: Local draft state tests in apps/web/tests/documents/document-draft.spec.ts
Task T044: Two-context update browser journey in tests/e2e/documents-update.spec.ts
```

### User Story 4

```text
Task T052: Delete contract/persistence tests in apps/api/test/documents/documents-delete.integration.spec.ts
Task T053: Alert Dialog accessibility tests in apps/web/tests/documents/delete-document-dialog.spec.tsx
Task T054: Keyboard deletion browser journey in tests/e2e/documents-delete.spec.ts
```

---

## Implementation Strategy

### MVP First: User Story 1

1. Complete Setup and Foundational phases.
2. Complete User Story 1 using Red-Green-Refactor.
3. Stop and validate empty state, private ordered list and one-action creation independently.
4. Demonstrate a new private `Sem título` document opening immediately.

### Incremental Delivery

1. Setup + Foundational establish shared contracts, storage and security.
2. US1 delivers private listing and creation as the MVP.
3. US2 adds complete owner-scoped partial search and cursor pagination.
4. US3 adds safe editing with optimistic concurrency and local-draft preservation.
5. US4 adds accessible confirmed physical deletion.
6. Polish verifies deployment, observability, performance and all repository gates.

### Parallel Team Strategy

1. Complete Setup and Foundational together.
2. Deliver US1 before splitting work because US2–US4 extend its navigation foundation.
3. After US1, assign US2, US3 and US4 tests/clients to separate owners while serializing shared repository/service/controller edits.
4. Merge only after each story checkpoint passes independent tests and mandatory coverage.

## Notes

- `[P]` means file-level parallelism without an incomplete dependency, not merely work that could overlap.
- Never accept `ownerId`, `searchText`, version or timestamps from the client.
- Never log title, content, query, cookie or raw user/document identifiers.
- Keep OpenAPI, shared Zod schemas, web behavior and API behavior synchronized.
- Do not introduce autosave, soft deletion or an external search service in this feature.
- Commit only after a test-first logical group passes its applicable gates.
