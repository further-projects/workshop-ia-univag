# Quickstart: Authentication Validation

This guide validates the authentication design after the foundation and feature tasks are implemented.
It is not an implementation script. Run commands from the repository root.

## Prerequisites

- Node.js 22.13 or newer within the Node.js 22 LTS line
- pnpm 11.21.0
- Docker with Compose
- A clean test environment using placeholder-only local secrets
- The MongoDB container configured as a single-node replica set

Never place real peppers, cookie secrets or HMAC keys in committed files. Copy the documented example
environment locally and provide test-only values through the shell or ignored environment files.

## Install and start

```bash
pnpm install --frozen-lockfile
docker compose up --build --wait
```

Expected result: web, API and MongoDB become healthy; MongoDB reports an initialized replica set. The
browser reaches the web application through the same-origin API proxy.

## Run quality gates

```bash
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test -- --coverage
pnpm build
pnpm test:e2e
```

Expected result: every command exits successfully. Authentication, authorization and shared schemas
report 100% statements, branches, functions and lines.

## Validate registration

1. Open the registration page.
2. Submit an invalid email, mismatched confirmation, seven-character password and 129-character
   password separately.
3. Register with a normalized unique email and an 8-to-128-character password.
4. Confirm the browser reaches the private home page without another login.
5. Inspect the response and browser storage.

Expected result: invalid fields are identified without persistence. Valid registration returns `201`,
sets an opaque `HttpOnly` cookie and exposes no token to JavaScript or JSON. The confirmation is absent
from persistence and logs. See [auth.openapi.yaml](contracts/auth.openapi.yaml).

## Validate duplicate and concurrent registration

Submit the same email again with case and surrounding-space differences, then run the integration test
that sends simultaneous registrations for the normalized identity.

Expected result: all duplicate failures use `REGISTRATION_FAILED` without identifying the email as
existing. Exactly one user and credential account exist, and no orphan records remain. Successful and
duplicate requests both respect the 750 ms response floor, and repeated measurements show no stable
fast-path distinction beyond the integration test's documented tolerance.

## Validate login lockout

1. Log out.
2. Submit four invalid passwords for the registered email.
3. Submit the fifth invalid password and record the server-controlled time.
4. Try the correct password before 15 minutes elapse.
5. Repeat attempts during the block.
6. Advance the injected integration-test clock past the original deadline and use the correct password.

Expected result: failures are generic; the fifth failure establishes one fixed deadline; blocked
attempts return `429` with `Retry-After` and never extend it. After expiry, valid login succeeds and
resets the counter. Run the equivalent test for an unknown email and verify matching response shapes.

## Validate origin limiting

Run the integration scenario that sends 21 login attempts from one trusted tracker within 60 seconds,
then requests again during and after the block. Include forged forwarding headers from an untrusted
peer.

Expected result: the first 20 requests reach credential handling, the 21st returns `429`, and requests
remain rejected for one minute without extending the deadline. Forged headers do not change the tracker.

## Validate session lifecycle

Use the injectable test clock to exercise the lifecycle in [data-model.md](data-model.md):

1. Authenticate and access a private API operation.
2. Advance less than 24 hours and access it again.
3. Advance past the 24-hour update interval but before seven inactive days and access it again.
4. Advance seven days without activity.
5. Authenticate again, log out, and replay the old cookie.
6. Deactivate the user and replay a previously valid session.

Expected result: renewal occurs only when eligible; seven inactive days expire the session; logout and
deactivation deny the next operation immediately. Missing, altered, expired and revoked cookies all
produce `401` without leaking their cause. An eligible ordinary private operation forwards the renewed
cookie; renewal does not require a separate request to `/auth/session`.

## Validate authorization and CSRF controls

Request a private operation with another user's resource identifier, then send each mutating auth
operation with an untrusted `Origin`.

Expected result: ownership failure is denied on the server even with a valid session. Untrusted origins
receive `403 REQUEST_ORIGIN_REJECTED`; UI visibility has no effect on authorization.

## Validate observability safety

Capture structured logs for successful and failed registration, login, lockout and logout scenarios.

Expected result: events contain an event name, outcome, request ID, pseudonymous subject/tracker where
needed and timestamp. They contain no password, confirmation, pepper, hash, cookie, token, raw unknown
email or exact lockout counter.

## Clean up

```bash
docker compose down --volumes
```

Expected result: containers, networks and test volumes created for this validation are removed.
