# Data Model: Authentication

## Ownership

Better Auth owns the native `user`, `account` and `session` records through its official MongoDB
adapter. Application code must not mirror those records in Mongoose or directly mutate native
credential and session fields. The API owns the `isActive` user extension and the separate
`AuthAttemptState` records needed for lockout, anti-enumeration and distributed serialization.

## User

Identity managed by Better Auth.

| Field | Type | Rules |
| --- | --- | --- |
| `id` | string | Stable Better Auth identifier; unique and immutable |
| `email` | string | Trimmed, lowercase, valid email; unique |
| `emailVerified` | boolean | `false` in this MVP; verification is out of scope |
| `name` | string | Internal normalized email; not exposed by the feature contract |
| `isActive` | boolean | Application extension; defaults to `true`; never client-writable |
| `createdAt` | date | Set by Better Auth |
| `updatedAt` | date | Set by Better Auth |

**Indexes**: unique index on `email`.

**Derived access state**: `ACTIVE` when `isActive=true`; `INACTIVE` otherwise. Login and every private
operation require `ACTIVE`.

## Credential Account

Password credential managed exclusively by Better Auth.

| Field | Type | Rules |
| --- | --- | --- |
| `id` | string | Unique account identifier |
| `userId` | string | References `User.id`; required |
| `providerId` | string | Fixed to `credential` for this feature |
| `accountId` | string | Better Auth credential account identifier |
| `password` | string | Versioned pepper identifier plus encoded Argon2id hash; never returned |
| `createdAt` | date | Set by Better Auth |
| `updatedAt` | date | Set by Better Auth |

**Indexes**: unique compound index on (`providerId`, `accountId`) and index on `userId`.

**Relationship**: one `User` has exactly one credential account in the MVP.

## Session

Opaque, revocable session managed by Better Auth.

| Field | Type | Rules |
| --- | --- | --- |
| `id` | string | Unique internal identifier; never exposed |
| `token` | string | Unique opaque secret; cookie only; never returned or logged |
| `userId` | string | References `User.id`; required |
| `expiresAt` | date | Seven days after creation or eligible activity renewal |
| `ipAddress` | string/null | Better Auth security metadata; restricted from public responses |
| `userAgent` | string/null | Better Auth security metadata; restricted from public responses |
| `createdAt` | date | Set by Better Auth |
| `updatedAt` | date | Last persisted update |

**Indexes**: unique index on `token`, index on `userId`, and index on `expiresAt`. Expiration is always
checked logically; a TTL index may remove stale records asynchronously but is not authorization.

**Transitions**:

| Event | Result |
| --- | --- |
| Valid registration or login | Create session and set opaque cookie |
| Eligible activity after 24 hours | Set `expiresAt` to activity time plus seven days |
| Less than 24 hours since update | Keep current expiration |
| Seven days without renewal | Treat as expired regardless of physical record |
| Logout | Revoke/delete current session and expire cookie immediately |
| User deactivation | Deny session on next server validation; revoke all sessions operationally |

When eligible activity renews `expiresAt`, the API session interceptor forwards the corresponding
`Set-Cookie` header on the same private response so browser and database expiration remain aligned.

## AuthAttemptState

Application-owned state for every attempted normalized identity, including identities with no user.
It prevents account enumeration and serializes password verification across API replicas.

| Field | Type | Rules |
| --- | --- | --- |
| `subjectKey` | string | HMAC of normalized email; unique; raw unknown email is not retained |
| `failedAttempts` | integer | From 0 through 5; defaults to 0 |
| `blockedUntil` | date/null | Fixed at fifth failure time plus 15 minutes |
| `leaseId` | string/null | Random owner for one in-flight attempt; never exposed |
| `leaseUntil` | date/null | Short crash-recovery deadline; never used as account block |
| `updatedAt` | date | Server timestamp of latest state mutation |

**Indexes**: unique index on `subjectKey`; index on `blockedUntil`; index on `leaseUntil` for cleanup
support. Expired states may be removed only after their block and lease are no longer active.

**Login state machine**:

| Current state | Event | Next state | Observable result |
| --- | --- | --- | --- |
| Available, attempts 0-3 | Invalid credential | Attempts + 1 | `401 INVALID_CREDENTIALS` |
| Available, attempts 4 | Invalid credential | Attempts 5, blocked for 15 minutes | `401 INVALID_CREDENTIALS` |
| Blocked | Any credential before deadline | Unchanged | `429 AUTH_TEMPORARILY_UNAVAILABLE` |
| Block expired | Valid credential | Attempts 0, no block | `200`, create session |
| Available | Valid credential | Attempts 0, no block | `200`, create session |
| Available | Inactive or unknown identity | Same failure transitions | Generic `401` or `429` |

Lease acquisition, password verification, state transition and session issuance form one serialized
logical operation per `subjectKey`. A request that cannot acquire the lease waits or fails closed with a
generic temporary response; it never bypasses the state check.

## OriginRateLimitState

MongoDB-backed storage used by NestJS Throttler.

| Field | Type | Rules |
| --- | --- | --- |
| `trackerKey` | string | HMAC of trusted client IP; unique |
| `windowStartedAt` | date | Start of current 60-second window |
| `requestCount` | integer | Number of login attempts in the window |
| `blockedUntil` | date/null | One minute after the excess attempt |
| `updatedAt` | date | Server timestamp |

**Indexes**: unique index on `trackerKey`; TTL cleanup index on a separate computed retention deadline.

The first 20 attempts in the window continue to identity checks. The 21st and subsequent attempts are
rejected before Argon2 work. A blocked request never extends `blockedUntil`.

## Registration transaction

Registration normalizes and validates input, then creates the Better Auth user, credential account and
initial session in one adapter transaction. The unique `User.email` index is the final authority under
concurrency. Any duplicate-key or partial failure maps to the generic registration error; no orphan user,
credential or session remains. Duplicate paths perform equivalent dummy Argon2id work when needed, and
both successful and duplicate responses wait for the common 750 ms minimum response duration.

## Validation rules

- Email is trimmed, lowercased and validated before lookup or persistence.
- Password length is 8 through 128 characters; no composition rule is applied.
- Password confirmation must equal password and is never passed to persistence.
- Client input cannot set `isActive`, counters, deadlines, IDs, names or session metadata.
- Server time is authoritative for windows, leases, blocking and session expiration.
- Passwords, confirmations, peppers, hashes, session tokens and raw unknown emails are forbidden in logs.
