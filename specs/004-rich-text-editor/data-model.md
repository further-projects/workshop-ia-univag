# Data Model: Rich Text Editor

## Rich Text Content

Canonical structured content stored in the existing document `content` field. This feature narrows the
generic JSON boundary from feature 003 to a closed ProseMirror-compatible vocabulary.

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| `type` | Literal `doc` | Yes | Root object; exactly one root |
| `content` | Rich Text Block[] | Yes | May be empty; document order is significant |

The complete canonical serialization remains limited to 1,000,000 UTF-8 bytes, 100 levels and 10,000
nodes using the counting rules in `003-document-management/data-model.md`. JSON must not contain
executable values, non-JSON values or prototype-related keys.

### Rich Text Block

A root or list container accepts only these node types:

| Node | Allowed children | Attributes |
|------|------------------|------------|
| `paragraph` | Zero or more `text` nodes | Optional `textAlign` |
| `heading` | Zero or more `text` nodes | Required `level` and `id`; optional `textAlign` |
| `bulletList` | One or more `listItem` nodes | None |
| `orderedList` | One or more `listItem` nodes | None |
| `listItem` | One or more blocks; first block is a paragraph | None |

Nested lists are accepted only inside a `listItem` after its first paragraph. Paragraph and heading
nodes do not directly contain block nodes.

### Text Node

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| `type` | Literal `text` | Yes | Accepted only in paragraph or heading content |
| `text` | String | Yes | Non-empty; plain Unicode text, never interpreted as HTML |
| `marks` | Rich Text Mark[] | No | No duplicate mark type; canonical mark order is deterministic |

### Rich Text Mark

| Mark | Attributes | Rules |
|------|------------|-------|
| `bold` | None | Boolean semantic mark |
| `italic` | None | Boolean semantic mark |
| `underline` | None | Boolean semantic mark |
| `textStyle` | Optional `color`, optional `fontSize` | At least one accepted attribute; no other style attributes |

Canonical mark order is `bold`, `italic`, `underline`, `textStyle`. Equivalent content is normalized to
this order before an editor-originated save to keep serialization and tests stable. The API accepts any
non-duplicate mark order because JSON array order is not semantic for marks; it does not rewrite a valid
alternate-client payload.

### Allowed Attributes

| Attribute | Values |
|-----------|--------|
| `heading.level` | Integer `1`, `2` or `3` |
| `heading.id` | Lowercase UUID string, unique among headings in the document |
| `textAlign` | `left`, `center`, `right` or `justify` |
| `textStyle.color` | `default`, `muted`, `red`, `orange`, `green` or `blue` |
| `textStyle.fontSize` | `small`, `default`, `large` or `x-large` |

All node, mark and attribute objects are strict. Unknown keys, node types, mark types, levels, IDs,
alignment values, colors and sizes fail API validation with the existing `422 VALIDATION_FAILED` and
`INVALID_CONTENT` violation.

Block nodes always include `content`, including an empty array for paragraph/heading; text nodes omit
`content`. Nodes and marks omit `attrs` when no attributes exist. Before validation/save, editor output
removes absent/default attributes emitted as `null`, including `textAlign: null`; accepted alignment is
persisted only as a non-null allowlisted token. Object key order has no semantic meaning. These shape
rules make omission and empty-array behavior deterministic without treating JSON object serialization
order as domain data.

### Invariants

- The API validates limits, shared Zod shape and ProseMirror structural validity before persistence.
- HTML, Markdown and rendered CSS are derived representations and never canonical fields.
- Heading IDs are persisted presentation-independent identities, not user-facing slugs.
- Heading IDs remain unchanged when heading text, level or position changes.
- Missing or duplicate IDs from interactive creation/paste are replaced before insertion; alternate API
  clients receive validation errors instead of server-side silent repair.
- Empty content remains `{ "type": "doc", "content": [] }`.
- Invalid stored content is not initialized as editable and cannot be overwritten by autosave.
- Read responses retain the generic safe JSON shape from feature 003 so legacy content can be returned;
  strict Rich Text validation is mandatory before editor initialization and for every new write.

## Heading Index Entry

Transient projection derived from the live Rich Text Content for table-of-contents rendering.

| Field | Type | Rules |
|-------|------|-------|
| `id` | UUID string | Matches exactly one live heading |
| `level` | Integer | `1`, `2` or `3` |
| `text` | String | Visible heading text; empty text uses an accessible fallback label |
| `position` | Integer | Current ProseMirror document position; transient and never persisted |
| `active` | Boolean | Whether the heading is the current navigation context |

Entries appear in document order. Repeated heading text remains distinct through `id`. Editing,
inserting, deleting or moving a heading rebuilds the projection in the same editor update cycle.

## Editor Session

Transient browser state for one open document. It is not a second persistence source.

| Field | Type | Rules |
|-------|------|-------|
| `documentId` | String | Current owner-scoped document ID |
| `confirmedSnapshot` | Document response | Last successful GET/POST/PATCH response |
| `confirmedEtag` | Strong ETag | Sent in the next PATCH `If-Match` header |
| `draftContent` | Rich Text Content | Current editor JSON, including edits not yet confirmed |
| `localRevision` | Non-negative integer | Increments for each local content change |
| `acknowledgedRevision` | Non-negative integer | Highest local revision confirmed by the API |
| `submittedRevision` | Integer or null | Revision represented by the one request currently in flight |
| `saveState` | Save State | Current persistence state |
| `retryAttempt` | Integer | `0` through `3`; reset after success or a new local change |
| `problem` | Problem or null | Last actionable API/network failure without private payloads |

The title may continue to use the feature 003 draft state, but title and rich text content share one
revision and one conditional PATCH when either changes.

### Save State

| State | Meaning | Permitted transitions |
|-------|---------|-----------------------|
| `clean` | Draft equals confirmed revision | `dirty` |
| `dirty` | Unconfirmed local changes exist | `debouncing`, `saving` |
| `debouncing` | Waiting 1.5 seconds since latest change | `dirty`, `saving` |
| `saving` | One PATCH is in flight | `clean`, `dirty`, `retry-wait`, `validation-error`, `conflict` |
| `retry-wait` | Network/503 retry is scheduled | `dirty`, `saving`, `save-error` |
| `save-error` | Retry budget exhausted or non-retryable persistence error | `dirty`, `saving` |
| `validation-error` | API rejected draft content | `dirty` after user edit |
| `conflict` | ETag is stale because another session saved | `conflict` until explicit reload/discard |

```text
clean -> dirty -> debouncing -> saving -> clean
                    ^            |  |
                    |            |  +-> dirty (newer local revision exists)
                    |            +----> retry-wait -> saving | save-error
                    +------------------ user edits

saving -> validation-error -> dirty after correction
saving -> conflict -> explicit reload/discard only
```

### Save Invariants

- At most one PATCH is in flight for an editor session.
- A successful response acknowledges only `submittedRevision` and replaces the confirmed snapshot/ETag.
- If `localRevision > submittedRevision` after success, the session remains dirty and schedules another
  save.
- Network and `503` failures retry after 2, 5 and 15 seconds; validation, auth, not-found, origin,
  precondition and conflict failures do not retry automatically.
- Internal context changes attempt immediate save. If changes remain unconfirmed, navigation requires an
  explicit discard action.
- `beforeunload` is active only while unconfirmed changes exist; browser close persistence is best effort,
  not guaranteed delivery.
- Conflict retains draft and stale ETag in memory and never performs merge or blind overwrite.

## Responsive Panel State

Transient presentation state for document navigation and table of contents.

| Field | Type | Rules |
|-------|------|-------|
| `viewportMode` | `wide` or `narrow` | Narrow below 1024 CSS pixels |
| `documentPanelOpen` | Boolean | Sheet state in narrow mode |
| `tocPanelOpen` | Boolean | Sheet state in narrow mode |
| `lastTrigger` | Element reference or null | Used only to restore focus on close |

In wide mode both side regions may remain visible. In narrow mode they are separate, labeled modal
panels; opening one closes the other, traps focus while open and restores focus to its trigger on close.

## Document Sidebar Page State

Transient cursor-pagination state backed by the feature 003 document list endpoint.

| Field | Type | Rules |
|-------|------|-------|
| `items` | Document List Item[] | Deduplicated by ID in API order |
| `nextCursor` | String or null | Opaque; `null` means all reachable items are loaded |
| `search` | String | Current normalized search input |
| `loadState` | Enum | `idle`, `loading`, `failed` |

The first and subsequent requests use at most 50 items per page. Changing search clears items and cursor
before loading its first page. A failed next-page request retains loaded items and cursor for retry. A
newly created projection is prepended only when `search` is empty; with an active filter, creation opens
the new document but leaves the filtered collection unchanged until a matching query retrieves it.
