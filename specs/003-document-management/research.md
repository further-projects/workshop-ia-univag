# Research: Document Management

## Persistence boundary

**Decision**: Use Mongoose 9.9.2 through `@nestjs/mongoose` 11.0.4 for one `documents` collection.
Persist one record per document with server-owned `ownerId`, canonical structured content, derived
search text, explicit version and timestamps. Provision indexes explicitly and disable automatic index
creation in production.

**Rationale**: Mongoose is the approved ODM for application-domain models and its NestJS module exposes
connection and model providers without mixing these records with Better Auth collections.

**Alternatives considered**: The native driver was rejected because domain persistence is assigned to
Mongoose. Embedding documents in users was rejected by the global data decision. A generic repository
framework was rejected as unnecessary.

## Structured content validation

**Decision**: Store the ProseMirror-compatible JSON object as the source of truth. The shared schema
requires a `doc` root and JSON-safe recursive nodes, with at most 100 levels, 10.000 nodes and 1 MB of
canonical UTF-8 JSON. The empty document is `{ "type": "doc", "content": [] }`. Feature 004 may narrow
allowed node, mark and attribute names without changing storage ownership.

The root counts as node 1 at level 1. Each object in a node's `content` array adds one node and one level
relative to its parent; marks and attribute values count toward bytes but not document node/depth limits.
Prototype-sensitive property names are rejected recursively in all attribute objects.

**Rationale**: Structural and resource limits make the boundary safe before the full editor vocabulary
exists. Replacing the whole content object avoids unreliable deep change tracking for mixed values.

**Alternatives considered**: HTML was rejected as non-canonical and unsafe. An unrestricted object was
rejected because size alone does not bound recursion or node count. Final Tiptap extensions are deferred
to the editor spec rather than guessed here.

## Title and size semantics

**Decision**: Trim title input; an empty result becomes `Sem título`. Count a maximum of 200 Unicode
code points after trim, before the separate search-text normalization. Define the content limit as 1 MB (1.000.000 bytes from canonical
`JSON.stringify` UTF-8), and set the route transport limit to exactly 1.250.000 bytes to accommodate the envelope.

**Rationale**: Code-point counting handles supplementary characters more predictably than JavaScript
UTF-16 length. Binary bytes give one testable persistence limit while leaving room for request metadata.

**Alternatives considered**: Grapheme counting was rejected because it requires another segmentation
policy and dependency for little MVP benefit. A 1 MB transport limit was rejected because valid content
plus its envelope could be refused before domain validation.

## Search representation and matching

**Decision**: Derive `searchText` on every successful create or update from normalized title and every
string-valued `text` field in document order. Concatenate adjacent text-node siblings and insert a space
around nested non-text containers, so formatting marks do not split words while structural containers
cannot join them. Insert one space between title and extracted content. Apply Unicode NFKD, remove
combining marks, lowercase and collapse
whitespace to both stored text and trimmed query. Search for the escaped query as a literal substring,
always combined with `ownerId`; an empty normalized query restores the unfiltered list.

**Rationale**: This exactly provides partial, case-insensitive and accent-insensitive matching without
accepting executable regex syntax. The field is derived server-side and cannot diverge by client input.

**Alternatives considered**: MongoDB `$text` was rejected because it searches tokens rather than
arbitrary substrings. Case-insensitive collation does not accelerate `$regex`. Atlas Search, n-grams and
an external engine were rejected as operational complexity until measured scale requires them.

## Search scale and performance gate

**Decision**: Validate the MVP at up to 500 documents per user and 50 kB average searchable text. Run
at least 100 warmed list/search operations with up to 10 concurrent requests and fail when p95 exceeds
one second. Include absent and low-selectivity terms and inspect MongoDB execution statistics. Document
the scan-based limitation and revisit the search engine before raising either bound.

**Rationale**: Non-anchored substring search cannot use a text range index. A measurable owner-bounded
profile makes SC-003 honest rather than implying unbounded scalability.

**Alternatives considered**: Claiming performance near the 1 MB maximum for all 500 documents was
rejected without evidence. Adding a search service preemptively was rejected by the simplicity gate.

## Indexes, ordering and cursor pagination

**Decision**: Create `{ ownerId: 1, updatedAt: -1, _id: -1 }`. Sort list and search results by the same
order. Use keyset pagination with default 20 and maximum 50 items. A signed, versioned opaque cursor
contains the last `updatedAt`, `_id`, owner fingerprint, normalized-query fingerprint and page limit; changed criteria or
invalid signatures return `INVALID_CURSOR`. Do not return total counts.

**Rationale**: Equality by owner followed by range/order fields serves private stable pages and avoids
offset cost. `_id` breaks timestamp ties. Query binding prevents accidental cursor reuse.

**Alternatives considered**: Offset pagination was rejected because updates cause skips and duplicates.
Snapshot pagination was rejected as disproportionate. A separate search-text index was rejected because
non-anchored regex cannot use it efficiently.

## Optimistic concurrency and HTTP preconditions

**Decision**: Start `version` at 1 and expose an ETag derived only from document ID and version. Require
`If-Match` for PATCH and reject it as `INVALID_PRECONDITION` unless its embedded ID equals the path ID.
Atomically filter by `_id`, `ownerId` and expected version, replace supplied
editable fields, regenerate derivatives, increment version and update the timestamp. Return `412
DOCUMENT_VERSION_CONFLICT` if an owner-scoped projection confirms the document still exists; otherwise
return owner-safe `404`.

**Rationale**: MongoDB single-document updates are atomic. Explicit compare-and-swap works with query
updates, whereas Mongoose optimistic concurrency primarily governs `save()` and default versioning is
not a complete concurrency solution.

**Alternatives considered**: Last-write-wins violates the spec. Automatic merge and edit locks are out
of scope. A body-only expected version was rejected in favor of standard HTTP preconditions.

## Owner-safe authorization and errors

**Decision**: Obtain `ownerId` only from the authoritative session and include it in the first filter of
every operation. Invalid, missing, deleted and foreign IDs all return the same `404 DOCUMENT_NOT_FOUND`
shape. Use stable `application/problem+json` codes; never include database messages, owner data or
current content in errors.

**Rationale**: Filtering before access makes ownership a database invariant and prevents content or
identity disclosure. Stable codes permit localized UI without parsing prose.

**Alternatives considered**: Loading by ID and checking ownership afterward was rejected because it
materializes foreign data. Returning `403` for foreign documents was rejected as an existence oracle.

## Deletion policy

**Decision**: Physically delete synchronously with an owner-scoped operation and return `204` only after
MongoDB confirms deletion. Subsequent access and repeated deletion return owner-safe `404`. Cancellation
is entirely client-side and makes no API request.

**Rationale**: This gives the required immediate observable result without retention, restoration or
pervasive soft-delete filters.

**Alternatives considered**: Soft delete and asynchronous purge were rejected because trash, retention
and recovery are explicitly outside current requirements.

## Web state and accessible deletion

**Decision**: Keep the last confirmed snapshot and current local draft separate. Validation,
precondition, network and persistence failures leave the draft untouched; only confirmed success
replaces snapshot and version. Use shadcn/ui Alert Dialog for deletion, default focus to Cancel, identify
the document, prevent duplicate submission, announce errors and restore a meaningful focus target.

**Rationale**: Explicit state prevents silent loss and the approved component provides the required
keyboard and assistive-technology dialog semantics.

**Alternatives considered**: Resetting from server state after any failure violates FR-012/FR-018.
Custom modal primitives duplicate established accessible behavior. Autosave remains feature 004 scope.

## Cache and observability

**Decision**: Return `Cache-Control: private, no-store`, `Vary: Cookie` and `X-Request-Id` on document
responses. Log route template, operation, outcome, status, duration and request ID; record low-cardinality
latency, conflict, limit and storage-failure metrics. Never log title, content, query, cookie, raw user ID
or raw document ID. Trace session validation, MongoDB operation and serialization without payloads.

**Rationale**: Private mutable content must not survive in shared or browser caches. Metadata-only
telemetry supports diagnosis and performance gates without collecting document data.

**Alternatives considered**: ETag-based response caching was rejected because ETag exists only for
write preconditions in this MVP. Payload logging was rejected as a privacy and secret-leak risk.

## Validation strategy

**Decision**: Follow Red-Green-Refactor and enforce 100% coverage in document API/web domain code,
authorization and shared schemas. Use unit tests for schemas, normalization, extraction, cursors and UI
state; real-MongoDB integration tests for owner filters, indexes, paging, physical deletion and atomic
conflicts; OpenAPI contract tests for every response; component accessibility tests and Playwright for
create, search, conflict, limits and deletion journeys.

**Rationale**: Numerical coverage is constitutional, while MongoDB and browser behaviors require tests
beyond mocks.

**Alternatives considered**: Mock-only persistence tests cannot prove indexes or compare-and-swap.
E2E-only tests make validation and authorization branches difficult to diagnose.

## Sources reviewed

- Context7 `/automattic/mongoose/9.0.1`: timestamps, `versionKey`, `optimisticConcurrency`, compound
  indexes, lean reads and update-validator limitations.
- Context7 `/nestjs/mongoose`: connection lifecycle and model registration with `forRoot`/`forFeature`.
- Context7 `/websites/mongodb_manual`: regex/collation limitations, compound-index ordering and atomic
  conditional single-document updates.
- Package registry metadata queried on 2026-08-12: Mongoose 9.9.2 requires Node.js 20.19 or newer;
  `@nestjs/mongoose` 11.0.4 explicitly accepts NestJS 10/11 and Mongoose 7/8/9 as peers. Current
  Mongoose security guidance recommends strict schemas/queries and sanitized filters; document filters
  are built only from validated scalar IDs and server-owned operators. Registry audit was unavailable
  because the foundation lockfile does not yet exist, so implementation must run `pnpm audit` before
  catalog adoption and block unresolved applicable high/critical advisories.

No `NEEDS CLARIFICATION` items remain.
