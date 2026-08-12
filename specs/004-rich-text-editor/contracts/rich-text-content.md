# Rich Text Content Contract

## Boundary

The `content` property of document create and update requests is a strict JSON object matching
[data-model.md](../data-model.md). The API validates this contract after the generic byte/depth/node
limits and before every persistence write. Read responses retain feature 003's generic safe JSON shape
for legacy compatibility; the editor applies this strict contract before initialization.

Canonical empty content:

```json
{
    "type": "doc",
    "content": []
}
```

## Vocabulary

| Kind | Accepted values |
|------|-----------------|
| Root | `doc` |
| Blocks | `paragraph`, `heading`, `bulletList`, `orderedList`, `listItem` |
| Inline | `text` |
| Marks | `bold`, `italic`, `underline`, `textStyle` |
| Heading levels | `1`, `2`, `3` |
| Alignment | `left`, `center`, `right`, `justify` |
| Color tokens | `default`, `muted`, `red`, `orange`, `green`, `blue` |
| Font-size tokens | `small`, `default`, `large`, `x-large` |

Heading IDs are lowercase UUID strings and must be unique within one content root. Node, mark and
attribute objects reject additional properties.

## Canonical Example

```json
{
    "type": "doc",
    "content": [
        {
            "type": "heading",
            "attrs": {
                "level": 1,
                "id": "2f144a49-240c-4c82-bd40-28f69ac6ad4e",
                "textAlign": "left"
            },
            "content": [
                {
                    "type": "text",
                    "text": "Planning"
                }
            ]
        },
        {
            "type": "paragraph",
            "attrs": {
                "textAlign": "left"
            },
            "content": [
                {
                    "type": "text",
                    "text": "Important note",
                    "marks": [
                        {
                            "type": "bold"
                        },
                        {
                            "type": "textStyle",
                            "attrs": {
                                "color": "red",
                                "fontSize": "large"
                            }
                        }
                    ]
                }
            ]
        }
    ]
}
```

The toolbar maps semantic tokens to theme-aware classes or CSS variables. These presentation mappings
are not part of the persisted contract. Before validation/save, the editor removes absent/default
attributes serialized as `null`, such as Tiptap TextAlign's default `textAlign: null`; raw
`editor.getJSON()` passes through this canonicalizer before matching the contract.

## Update Mapping

Autosave uses the existing operation:

```http
PATCH /api/documents/{documentId}
If-Match: "document-<documentId>-<version>"
Content-Type: application/json

{
    "content": { ...richTextContent }
}
```

If title and content changed in the same local revision, the same request carries both. A successful
response replaces the client's confirmed document snapshot and ETag. `412 DOCUMENT_VERSION_CONFLICT`
retains the submitted draft and stale ETag but does not return the latest document body.

## Interactive Paste Normalization

Paste into the editor follows these deterministic rules before insertion:

1. Parse input through the configured ProseMirror schema; scripts, event attributes, embeds, images,
   links, stylesheets and unsupported nodes/marks cannot enter the canonical slice.
2. Preserve supported paragraphs, H1-H3, ordered/unordered lists and supported marks.
3. Convert unsupported heading levels to paragraphs rather than changing their hierarchy to H3.
4. Flatten unsupported containers to their accepted textual blocks when safe; otherwise retain only
   their plain visible text.
5. Remove unknown colors, font sizes and arbitrary style properties; do not map them to the nearest token.
6. Generate new UUIDs for headings without IDs and replace duplicate or externally supplied colliding IDs.

The HTTP API does not apply these repairs. Nonconforming JSON from an alternate client returns
`422 VALIDATION_FAILED` with an `INVALID_CONTENT` violation, leaving persistence unchanged.

## Deterministic Validation Precedence

For content-bearing requests, the server reports violations in this order:

1. Transport body over 1,250,000 bytes: `413 REQUEST_BODY_TOO_LARGE`.
2. Canonical content over 1,000,000 UTF-8 bytes: `CONTENT_TOO_LARGE`.
3. Content deeper than 100 levels: `CONTENT_TOO_DEEP`.
4. Content with more than 10,000 nodes: `CONTENT_TOO_MANY_NODES`.
5. Unknown or structurally invalid rich text vocabulary: `INVALID_CONTENT`.

Semantic schema failure returns one public `INVALID_CONTENT` violation because the reused problem
contract intentionally exposes no content path. Internal diagnostics may identify a safe rule name but
must not include document content, private identifiers or serialized field values.

## Round-Trip Requirements

- `editor.getJSON()` for accepted content validates without information loss.
- PATCH followed by GET and editor initialization preserves all accepted nodes, marks, attributes and
  heading IDs.
- Unknown stored content raises the safe non-editable recovery state; initialization must not silently
  remove it and autosave must remain disabled.
- JSON-looking text remains text and is never evaluated as markup or code.
