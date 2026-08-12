# Document Management Contracts

- [documents.openapi.yaml](documents.openapi.yaml) is the stable HTTP facade exposed under `/api`.
- Every operation requires the opaque session cookie from `002-authentication`.
- Mutating operations additionally require the trusted-origin protection defined by authentication.
- API and web must consume Zod schemas from `packages/schemas`; OpenAPI mirrors those schemas.
- Clients depend on HTTP status and stable `code`, never on localized problem prose.
- `ownerId`, search derivatives and persistence field names are never public contract fields.
- ETags are write preconditions only; all responses remain private and non-cacheable.
