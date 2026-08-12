# Quickstart: Document Management Validation

This guide validates the completed feature. It does not replace implementation tasks or automated
tests. Run commands from the repository root after `001-project-foundation` and `002-authentication` are
implemented.

## Prerequisites

- Node.js 22.13 or newer within Node.js 22 LTS and the repository-pinned pnpm version.
- Docker with Compose support.
- Local environment derived from committed examples, with no secrets committed.
- Authentication implemented with two active test users.
- Document cursor-signing secret supplied through the environment.

## Start a Clean Environment

```bash
pnpm install --frozen-lockfile
docker compose up --build --wait
```

Expected result: web, API and MongoDB become healthy; unauthenticated document routes return `401`.

## Run Automated Gates

```bash
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm test:e2e
```

Expected result: all gates pass. Coverage reports show 100% statements, branches, functions and lines
for document domain code in web/API, document authorization and shared document schemas.

## Validate Contracts and Persistence

Use [contracts/documents.openapi.yaml](contracts/documents.openapi.yaml) as the HTTP authority and
[data-model.md](data-model.md) for persistence invariants.

Automated integration and contract tests must prove:

1. The deployed OpenAPI document parses and every status/body/header pair matches shared Zod schemas.
2. The `documents` collection has `{ ownerId: 1, updatedAt: -1, _id: -1 }` and no speculative search
   index.
3. API responses never expose `ownerId`, `searchText`, MongoDB field names or session values.
4. Document responses include `Cache-Control: private, no-store`, `Vary: Cookie` and `X-Request-Id`.
5. Logs and traces contain no title, content, query, cookie or raw user/document ID.

Provisioning must be explicit and idempotent before application rollout:

```bash
pnpm --filter api provision:document-indexes
```

Expected result: the command creates or verifies the exact index and fails deployment on conflict.

## Validate Core Journey

1. Sign in as user A with no documents.
2. Open the main page and verify the specified no-documents message and one create action.
3. Create a document with no title or content.
4. Verify it opens immediately as `Sem título` with the canonical empty structured document.
5. Return to the list and verify only user A's documents appear, newest modification first.
6. Update title and content with the last received ETag in `If-Match`.
7. Reload and verify the content, incremented version and later `updatedAt` persisted.

Expected result: all state comes from owner-scoped API responses; no browser token storage is created.

## Validate Search and Pagination

Seed user A with 55 documents, including titles/content containing `Café`, `Planejamento` and regex
metacharacters such as `[draft]`. Give multiple documents identical `updatedAt` values.

1. Search for `cafe`, `CAFE` and partial `nejamen`; verify matching ignores case and accents.
2. Search literally for `[draft]`; verify metacharacters do not alter query semantics or cause errors.
3. Clear whitespace-only search; verify the complete list returns.
4. Fetch pages of 20 and follow `nextCursor`; verify stable `updatedAt DESC, id DESC` order without
   duplicates in an unchanged dataset.
5. Reuse a cursor with a different query or limit and alter one cursor byte.

Expected result: incompatible and altered cursors return `400 INVALID_CURSOR`; list/search responses do
not include structured content or total counts.

## Validate Ownership

1. Sign in as user B and attempt GET, PATCH and DELETE with user A's document ID.
2. Repeat with a well-formed nonexistent ID, malformed ID and an ID deleted earlier.
3. Compare status, media type and stable code.

Expected result: every request returns the same `404 DOCUMENT_NOT_FOUND` public shape and reveals no
content or owner. User A's document remains unchanged after user B's mutations.

## Validate Concurrent Updates

1. Open the same user A document in two independent browser contexts.
2. Save context one with the shared initial ETag.
3. Attempt to save different changes from context two with the stale ETag.

Expected result: exactly one update succeeds. The second returns `412 DOCUMENT_VERSION_CONFLICT`, does
not expose current content/version, keeps its local draft and prompts for reload. A request without
`If-Match` returns `428 PRECONDITION_REQUIRED`; malformed preconditions and ETags copied from a different
document return `400 INVALID_PRECONDITION`.

## Validate Limits and Recovery

Exercise exactly-at-limit and one-over-limit boundaries:

- 200 and 201 Unicode code points in title.
- 1,000,000 and 1,000,001 canonical UTF-8 content bytes.
- 100 and 101 levels with root as level 1; 10,000 and 10,001 content nodes with root as node 1.
- Transport bodies at 1,250,000 and 1,250,001 bytes.

Expected result: valid boundaries persist; invalid boundaries return `413` or `422` with stable problem
codes and violations. The web retains title and content locally after validation, network and simulated
MongoDB failures and allows retry without blind overwrite.

## Validate Deletion Accessibility and Effect

1. Trigger deletion by keyboard; verify the Alert Dialog names the document and focuses Cancel.
2. Press Escape and repeat using Cancel; verify no DELETE request occurs and focus is restored.
3. Confirm deletion; prevent a duplicate confirmation while pending.
4. Simulate one persistence failure; verify the dialog remains recoverable and announces the error.
5. Confirm successfully, then fetch and delete the same ID again.

Expected result: the successful response is `204` only after physical deletion. The item disappears
immediately, later operations return owner-safe `404`, and focus moves to a meaningful remaining target.

## Validate Performance Profile

Against real MongoDB, seed one owner with 500 documents averaging 50 kB of searchable text. After
warmup, measure at least 100 operations with no more than 10 concurrent requests.

- Report p95 separately for unfiltered list, common search, absent search, point read, create, update
  and delete.
- Fail if list/search p95 exceeds 1 second.
- Fail if point reads or mutations p95 exceeds 200 ms.
- Capture `explain("executionStats")` for representative list and search queries.

Expected result: owner/order index serves list pagination; search remains an explicit owner-bounded scan.
If it misses the target, stop implementation acceptance and plan an indexed search strategy rather than
silently weakening SC-003.

## Stop the Environment

```bash
docker compose down
```
