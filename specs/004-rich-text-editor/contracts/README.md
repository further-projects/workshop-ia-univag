# Rich Text Editor Contracts

- [rich-text-content.md](rich-text-content.md) is the canonical node, mark, attribute and normalization
  contract for editor content.
- [editor-writes.openapi.yaml](editor-writes.openapi.yaml) is the OpenAPI 3.1 companion that narrows the
  existing document POST/PATCH request bodies to strict Rich Text Content.
- This feature adds no HTTP endpoint. It reuses
  [`003-document-management/contracts/documents.openapi.yaml`](../../003-document-management/contracts/documents.openapi.yaml)
  for list, create, read and conditional update operations.
- `richTextContentSchema` in `packages/schemas/src/rich-text.ts` is the contract source shared by web and
  API; `packages/editor-schema` is the single executable ProseMirror/Tiptap schema. Feature 003 request
  schemas compose the strict schema instead of maintaining a second recursive shape.
- Existing session cookie, owner isolation, trusted-origin checks, private cache headers, request IDs,
  ETags and problem codes remain unchanged.
- Clients depend on HTTP status and stable problem `code`; localized prose is not control flow.
- `INVALID_CONTENT` covers any unknown or structurally invalid rich text node, mark or attribute. Existing
  dedicated size, depth and node-count violations retain precedence.
- Persisted content is JSON only. HTML, Markdown, DOM classes, CSS values, selections and editor plugin
  state are not API fields.
- Read responses keep feature 003's generic safe JSON content schema so pre-004 records can be returned.
  The editor must validate the stricter contract before initialization and enter recovery mode on failure.
