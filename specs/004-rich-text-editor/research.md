# Research: Rich Text Editor

## Tiptap integration and SSR

- **Decision**: Use Tiptap 3.30.0 in a Next.js Client Component with `immediatelyRender: false`. The
  document route may load the owner-scoped snapshot on the server, but editor construction and all
  selection-dependent UI remain inside the client boundary.
- **Rationale**: Current Tiptap React guidance disables immediate rendering in SSR applications to
  avoid hydration mismatches. The App Router supports browser-only Client Components; a separate
  `next/dynamic({ ssr: false })` boundary is reserved for an actual browser API incompatibility rather
  than used by default.
- **Alternatives considered**: Server-rendering `EditorContent` risks hydration and DOM access errors.
  Disabling SSR for the complete page would discard useful authenticated loading and loading states.

## Explicit extension allowlist

- **Decision**: Configure Tiptap with a closed persisted vocabulary: `doc`, `paragraph`, `text`,
  `heading`, `bulletList`, `orderedList` and `listItem`; marks `bold`, `italic`, `underline` and
  `textStyle`; attributes `heading.level`, `heading.id`, block `textAlign`, and `textStyle.color` plus
  `textStyle.fontSize`. Configure headings as H1-H3. Undo/redo may be enabled because it does not add
  persisted node or mark types.
- **Rationale**: A closed allowlist prevents Tiptap defaults or future extensions from silently
  widening the shared storage contract. It exactly matches FR-002 through FR-005 and allows deterministic
  schema, paste and round-trip tests.
- **Alternatives considered**: Unrestricted StarterKit admits unsupported blockquotes, code blocks,
  horizontal rules and strike marks. Render-time sanitization leaves invalid canonical data stored.

## Predefined color and font-size values

- **Decision**: Persist colors as six semantic tokens in the `textStyle.color` attribute: `default`,
  `muted`, `red`, `orange`, `green` and `blue`. Persist font size as four semantic tokens in
  `textStyle.fontSize`: `small`, `default`, `large` and `x-large`. Tiptap rendering maps tokens to theme
  CSS variables/classes; raw CSS colors, units and arbitrary style declarations are never persisted.
- **Rationale**: Semantic tokens satisfy the clarified predefined sets, remain stable across light and
  dark themes, and avoid persisting presentation values that may become unreadable or unsafe.
- **Alternatives considered**: Hex/RGB colors and arbitrary CSS lengths make contrast, validation and
  migrations harder. A larger palette adds combinations without a stated user need.

## Canonical content validation

- **Decision**: Define `richTextContentSchema` once in `packages/schemas/src/rich-text.ts` and compose it
  into document create/update request schemas. Export the single executable ProseMirror/Tiptap schema
  from `packages/editor-schema` for both web and API. Enforce the existing 1,000,000-byte, 100-level and
  10,000-node limits before constructing a ProseMirror node. Client validation is only early feedback.
- **Rationale**: Shared Zod contracts satisfy the constitution, while ProseMirror validation catches
  structural relationships that a generic recursive object schema cannot safely express. The API stays
  authoritative and feature 004 narrows, rather than replaces, feature 003's canonical content contract.
- **Alternatives considered**: Zod-only recursive validation can drift from editor semantics. Tiptap's
  `enableContentCheck` protects editor initialization but cannot authorize persistence. HTML sanitization
  is irrelevant because HTML is not canonical.

## Empty and invalid stored documents

- **Decision**: Keep `{ "type": "doc", "content": [] }` as the canonical empty document and configure
  the ProseMirror root to permit zero or more blocks. Read responses retain feature 003's generic safe
  JSON schema for legacy compatibility; the client applies `richTextContentSchema` plus ProseMirror
  checking before initialization. If content fails, do not initialize it as editable or autosave a
  repaired copy; show a safe recovery state and retain the received JSON only in memory until the user
  leaves or explicitly reloads. POST/PATCH accept only the strict feature 004 shape.
- **Rationale**: This preserves the established document contract and prevents a silent cleanup from
  destroying legacy or unknown content on the next autosave.
- **Alternatives considered**: Requiring one paragraph conflicts with feature 003. Automatic repair on
  load creates an implicit destructive migration.

## Paste normalization

- **Decision**: Let ProseMirror parse pasted text/HTML through the configured allowlist, then normalize
  the resulting slice before insertion. Preserve accepted paragraphs, H1-H3, lists and marks; flatten
  unsupported containers to accepted text blocks; convert unsupported headings to paragraphs; remove
  active/embed nodes; and remove color or size attributes not in the predefined sets. Plain text remains
  plain paragraphs. The API rejects nonconforming canonical JSON instead of normalizing it.
- **Rationale**: Paste is the only boundary where user-friendly cleanup is appropriate. Server-side
  rejection prevents alternate clients from using normalization differences to expand the contract.
- **Alternatives considered**: Preserving arbitrary HTML violates FR-011/FR-012. Converting H4-H6 to H3
  invents hierarchy. Server-side silent normalization can make the persisted response differ from the
  submitted draft.

## Heading identity and table of contents

- **Decision**: Persist an opaque UUID heading `id`; preserve it when text, level or position changes,
  and generate a new value for new, pasted-without-ID or duplicate headings. Validate UUID shape and
  uniqueness within the document. Derive the table of contents from the live editor state in document
  order and include only H1-H3.
- **Rationale**: IDs remain stable when titles are repeated or renamed. Current Tiptap documentation
  provides UniqueID/TableOfContents patterns and `generateTocIds`, but the shared schema remains the
  authority for accepted IDs.
- **Alternatives considered**: Text slugs collide and change on rename. Document positions shift after
  preceding edits. Session-only IDs break navigation stability after reopen.

## Autosave and navigation protection

- **Decision**: Debounce autosave for 1.5 seconds after the latest change, allow one PATCH in flight,
  and track `confirmedSnapshot`, ETag, local revision and acknowledged revision separately. A response
  acknowledges only the submitted revision; edits made during the request remain dirty and trigger the
  next save. Retry only network and `503 DOCUMENT_STORE_UNAVAILABLE` failures at 2, 5 and 15 seconds.
  Internal navigation performs an immediate save attempt and requires explicit discard confirmation if
  changes remain unconfirmed. Register `beforeunload` only while dirty/saving/failed.
- **Rationale**: Revision tracking avoids marking newer edits saved when an older request finishes.
  Serial writes prevent avoidable self-conflicts and reuse the feature 003 ETag contract.
- **Alternatives considered**: Parallel saves can reorder. A single `isDirty` boolean loses edits made
  in flight. `sendBeacon` cannot send the existing conditional PATCH, and unload-time delivery of a
  payload near 1 MB cannot be guaranteed.

## Version conflicts and recovery

- **Decision**: On `412 DOCUMENT_VERSION_CONFLICT`, stop automatic saves, retain the draft and stale
  ETag in memory, expose a conflict state, and offer explicit reload-latest and continue-reviewing/copy
  actions. Reload requires a second discard confirmation because it destroys the local draft. Never
  merge, overwrite or retry the conflict automatically.
- **Rationale**: This implements FR-022 and the compare-and-swap contract from feature 003 without
  claiming collaborative merge behavior.
- **Alternatives considered**: Last-write-wins loses remote work. Automatic merge is out of scope.
  Browser storage would introduce a recovery lifecycle and privacy surface not required by the spec.

## Toolbar, accessibility and responsive panels

- **Decision**: Implement a named toolbar with roving focus and arrow-key navigation. Toggle controls
  expose active/mixed state through accessible pressed semantics and unavailable actions are disabled.
  Color and font-size use labeled shadcn-compatible menu controls. The table of contents is a named
  navigation region. At Tailwind's `lg` breakpoint (1024 CSS pixels), document navigation and TOC move
  into separate shadcn/ui Sheet-based panels with labeled triggers, focus trapping and trigger focus
  restoration.
- **Rationale**: The design makes keyboard acceptance deterministic and uses established accessible
  primitives instead of custom drawers. A 1024-pixel breakpoint preserves a practical writing column
  while both side regions are present.
- **Alternatives considered**: Every control in the tab order makes the toolbar cumbersome. Tooltips are
  not accessible names. Custom drawers duplicate focus management. Hiding either panel violates FR-021.

## API contract reuse

- **Decision**: Add no editor-specific endpoint. Reuse `GET /documents`, `POST /documents`,
  `GET /documents/{documentId}` and conditional `PATCH /documents/{documentId}` from feature 003. The
  feature 004 OpenAPI companion specializes POST/PATCH content as the intersection of the existing
  request envelope and strict Rich Text Content. GET remains generic enough to return legacy safe JSON,
  which the editor validates before use. Status codes, problem bodies, ETags, cookies, trusted-origin
  checks and private cache headers remain unchanged.
- **Rationale**: Rich text is the document's canonical content, not a separate resource. Reuse avoids
  divergent ownership and concurrency behavior.
- **Alternatives considered**: A dedicated save endpoint duplicates document authorization and version
  semantics. A client-only contract would leave the API accepting unsupported editor content.

## Sidebar pagination

- **Decision**: Consume `GET /documents` in cursor pages of at most 50 and append pages as the user
  scrolls or activates an explicit load-more control. Reset accumulated items when search criteria
  change, deduplicate by document ID, and preserve the API's `updatedAt DESC, id DESC` order. Creation
  opens immediately; it prepends the returned projection only when there is no active search, otherwise
  the filtered list remains unchanged.
- **Rationale**: The validated profile permits 500 documents, while the existing contract deliberately
  caps each page at 50. Incremental loading keeps the sidebar responsive without inventing a new endpoint.
- **Alternatives considered**: Fetching all cursors eagerly increases startup work. Showing only the
  first page makes older documents unreachable. Client-side sorting can conflict with cursor order.

## Testing and observability

- **Decision**: Use Vitest 4.1.10 for shared schema, autosave state machine and component tests;
  Playwright 1.62.1 for native selection, clipboard, keyboard, responsive panels, hydration and
  two-context conflicts; and `axe-core` 4.13.0 for automated accessibility checks supplemented by
  explicit keyboard assertions. Document/shared-schema code maintains 100% statements, branches,
  functions and lines. Log only request ID, operation, outcome, latency and safe error code; never log
  content, title, selection, clipboard data, ETag, user ID or document ID.
- **Rationale**: Browser editing behavior is not faithfully testable in a DOM mock, while state-machine
  edge cases are faster and clearer in unit tests. Metadata-only telemetry preserves private content.
- **Alternatives considered**: E2E-only testing is slow and poor for exhaustive transitions. Unit-only
  testing misses selection, focus, paste and browser navigation behavior.

## Dependency versions

- **Decision**: Add matching Tiptap 3.30.0 versions for `@tiptap/core`, `@tiptap/pm`, `@tiptap/react`,
  `@tiptap/starter-kit`, `@tiptap/extension-text-style`, `@tiptap/extension-text-align`,
  `@tiptap/extension-underline`, `@tiptap/extension-unique-id` and
  `@tiptap/extension-table-of-contents`; add `axe-core` 4.13.0 for accessibility validation. Declare all
  direct versions in the pnpm catalog.
- **Rationale**: Matching Tiptap package versions reduce ProseMirror peer/version drift. Versions were
  checked against current official Context7 documentation and the registry on 2026-08-12.
- **Alternatives considered**: Mixed Tiptap versions create avoidable compatibility risk. An all-in-one
  editor wrapper adds UI and schema behavior outside the approved Tiptap/shadcn baseline.
