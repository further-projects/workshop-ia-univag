# Quickstart: Project Foundation Validation

This guide defines the runnable acceptance flow for the completed foundation. Commands describe the
planned interface and become executable during `/speckit.implement`.

## Prerequisites

- macOS or Linux
- Node.js 22.13 or newer within the Node.js 22 LTS line
- Corepack enabled with pnpm 11.21.0
- Docker Engine with Docker Compose v2
- Git

Docker access is required for the integrated scenario. The repository sandbox used by an AI agent may
block Docker even when it is available to the developer host.

## Prepare a clean clone

```bash
cp .env.example .env
./scripts/check-prerequisites.sh
corepack pnpm install --frozen-lockfile
```

Expected outcomes:

- Preflight validates supported tool versions, required configuration, and port conflicts before
  dependency installation or service startup.
- Invalid or missing configuration produces a non-zero exit and identifies only the affected field.
- Installation discovers every workspace under `apps/*` and `packages/*`.
- No real secret is required.

## Run root quality gates

```bash
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm check
```

Expected outcomes:

- Each command covers every applicable workspace exactly once through Turborepo.
- `pnpm check` provides the consolidated reusable gate.
- A deliberate format, type, or test violation causes its corresponding root command to fail and name
  the affected workspace.

## Start the integrated environment

```bash
docker compose up --build --wait --wait-timeout 120
```

Expected outcomes within two minutes:

- MongoDB is healthy.
- The API health response is HTTP 200 and reports `api: up` and `mongodb: up`.
- The web health response is HTTP 200 and reports `web: up`.
- The web page is reachable at the URL documented in `.env.example`.

The exact response schemas are defined in [contracts/health.openapi.yaml](contracts/health.openapi.yaml),
and operational states are defined in [data-model.md](data-model.md).

## Run automated integrated validation

```bash
pnpm test:e2e
```

Expected outcomes:

- Playwright verifies the web page in a browser.
- Playwright validates both health payloads against the shared Zod contract.
- The API result proves MongoDB readiness.
- The command fails when any required service or check is unavailable.
- Test orchestration tears down managed services on success or failure.

## Validate commit hooks

```bash
pnpm exec lefthook run pre-commit
pnpm exec commitlint --from HEAD~1 --to HEAD --verbose
```

Expected outcomes:

- Pre-commit applies Biome checks only to supported changed files.
- Commit messages that do not follow Conventional Commits are rejected.
- Full root gates remain separate and can be run with `pnpm check`.

## Stop and clean up

```bash
docker compose down --remove-orphans
```

This stops managed services while retaining the named development database volume. To explicitly
discard local temporary database data when diagnosing an invalid cache or state:

```bash
docker compose down --volumes --remove-orphans
pnpm clean
```

Volume removal is intentionally explicit because it deletes local MongoDB data. Neither cleanup path
reads, displays, or commits environment secrets.
