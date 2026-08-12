# Quickstart: Rich Text Editor Validation

This guide validates the completed feature. It does not replace implementation tasks or automated tests.
Run commands from the repository root after features 001, 002 and 003 are implemented.

## Prerequisites

- Node.js 22.13 or newer within Node.js 22 LTS and the repository-pinned pnpm version.
- Docker with Compose support and a local environment derived from committed examples.
- Two active authenticated test users and document indexes provisioned by feature 003.
- Tiptap dependencies pinned through the pnpm catalog at the versions in [plan.md](plan.md).

## Start a Clean Environment

```bash
pnpm install --frozen-lockfile
docker compose up --build --wait
```

Expected result: web, API and MongoDB become healthy; private document/editor routes require the opaque
session cookie and reveal no private content before authentication.

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
for shared rich text/document schemas, document validation and modified document authorization/domain
code. Browser tests report no critical automated accessibility violations.

## Validate Contract and Canonical JSON

Use [contracts/rich-text-content.md](contracts/rich-text-content.md) as the content authority and reuse
the HTTP facade linked by [contracts/README.md](contracts/README.md).

Automated contract tests must prove:

1. Every accepted node, mark and attribute round-trips editor -> PATCH -> GET -> editor without loss.
2. Empty `{ "type": "doc", "content": [] }` remains valid and editable.
3. Unknown nodes/marks/attributes, H4-H6, duplicate heading IDs, arbitrary CSS colors/sizes and malformed
   structures return `422 VALIDATION_FAILED` plus `INVALID_CONTENT` without a persistence change.
4. Existing exact byte, depth, node-count and transport limits retain their documented status and
   precedence.
5. Content, title, clipboard values, ETags, cookies and raw user/document IDs never appear in logs,
   traces or error responses.
6. A generic legacy document can still be read, but strict client validation blocks editor initialization
   and autosave until the user explicitly leaves or reloads; no silent repair is persisted.

## Validate Editing and Formatting

1. Sign in as user A, create a document and open it in the editor.
2. Enter paragraphs and H1, H2 and H3 headings; add ordered and unordered lists.
3. Apply and remove bold, italic, underline, every alignment, six color tokens and four font-size tokens.
4. Exercise collapsed selections, text selections and mixed-format selections.
5. Wait for autosave, reload the route and compare semantic content and visual state.

Expected result: toolbar actions update the editor within 100 ms p95, communicate active/inactive/mixed/
disabled state, and every accepted value returns unchanged after persistence. No H4-H6 action exists.

## Validate Paste Safety

Paste HTML containing accepted formatting together with scripts, event handlers, iframes, images, links,
H4, blockquotes, arbitrary colors, pixel sizes and duplicate heading IDs. Also paste plain text and text
that resembles JSON or HTML.

Expected result: accepted structure remains; H4 becomes a paragraph; unsupported containers are safely
flattened; active/embed content, links and arbitrary styles are removed; heading IDs are unique; plain
text remains literal. Saved JSON contains only the contract vocabulary and executes no active content.

## Validate Table of Contents

1. Create repeated heading text at H1-H3 and verify each entry has a distinct destination.
2. Rename, move and change the level of one heading; verify its ID remains stable and hierarchy/order
   update without a page reload.
3. Insert and delete headings and verify the list updates in the same editor update cycle.
4. Activate each item by pointer and keyboard.

Expected result: the target becomes visible and identifiable within one second. Empty documents show no
invalid TOC entries, and repeated titles never navigate to the wrong heading.

## Validate Autosave and Recovery

Use network inspection and deterministic fake timers for unit/component coverage, then repeat critical
behavior in a real browser:

1. Make several changes less than 1.5 seconds apart; verify no PATCH occurs until 1.5 seconds after the
   final change.
2. Edit while one PATCH is delayed; verify only one request is in flight and its success does not mark
   the newer revision clean. A second save follows.
3. Simulate network and `503` failures; verify retries at 2, 5 and 15 seconds and retained local content.
4. Simulate `422`; verify no automatic retry and correction returns the state to dirty/saveable.
5. Navigate to another document while dirty; verify an immediate save attempt. Fail it and confirm that
   cancel preserves the editor while explicit discard allows navigation.
6. Close/reload while dirty; verify the browser warning is active only while changes are unconfirmed.

Expected result: state always distinguishes pending, saving, saved and failed work. No response for an
older revision clears newer edits. Browser shutdown is treated as best-effort plus warning, not as a
guaranteed final PATCH.

## Validate Concurrent Editing Conflict

1. Open one document for user A in two independent browser contexts with the same initial ETag.
2. Save a change in context one.
3. Let autosave submit different content from context two with the stale ETag.
4. Continue reviewing the retained local content, then choose reload latest.

Expected result: context two receives `412 DOCUMENT_VERSION_CONFLICT`, stops autosave, keeps local edits
visible and offers explicit recovery. Reload asks before discarding, then obtains the latest owner-scoped
snapshot. No automatic merge, overwrite or blind retry occurs.

## Validate Keyboard and Assistive Technology

1. Reach the named toolbar by keyboard and use arrow keys to move through controls.
2. Apply every essential format without a pointer; verify accessible names, pressed/mixed semantics and
   disabled actions.
3. Navigate the named table-of-contents region and verify target focus/identification.
4. Trigger save, validation, persistence and conflict failures; verify status changes are announced
   without repeatedly stealing focus.

Expected result: every essential toolbar/sidebar action and TOC navigation works by keyboard. Tooltips
are supplementary and never the only accessible name.

## Validate Responsive Panels and Themes

Run the same document at widths immediately above and below 1024 CSS pixels in light and dark themes.

1. At or above 1024 pixels, verify sidebar, editor and TOC preserve a usable writing column.
2. Below 1024 pixels, verify sidebar and TOC become separate labeled panel triggers.
3. Open each panel by keyboard; verify focus enters, stays contained, closes with Escape and returns to
   its trigger. Opening one closes the other.
4. Verify semantic colors remain readable and editor/toolbar remain available in both themes.

Expected result: no writing or essential navigation capability disappears at narrow widths, and closed
panels neither obscure nor trap interaction.

## Validate Sidebar Pagination

Seed user A with at least 55 documents and use a page size of 50.

1. Open the editor and verify the first page follows API order without loading document content.
2. Scroll to the end or activate load more; verify the second cursor page appends without duplicates.
3. Fail one next-page request; verify loaded items remain usable and retry uses the retained cursor.
4. Change search criteria; verify previous items/cursor clear and matching pages load independently.
5. Without a search, create a document and verify it opens immediately and appears first. Repeat with a
   nonmatching active search and verify it opens without entering the filtered list.

Expected result: every owned document remains reachable through incremental pagination; foreign
documents never appear and the client does not reorder cursor pages.

## Validate Ownership and Observability

1. As user B, attempt to open and update user A's document through the editor route and API.
2. Compare malformed, absent, deleted and foreign document behavior.
3. Inspect structured logs for successful saves, retries, validation failures and conflicts.

Expected result: every owner-scoped miss remains `404 DOCUMENT_NOT_FOUND`; the UI exposes no foreign
metadata. Logs contain only request ID, operation, outcome, latency and safe error code.

## Stop the Environment

```bash
docker compose down
```
