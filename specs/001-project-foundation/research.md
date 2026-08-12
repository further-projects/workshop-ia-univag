# Research: Project Foundation

## Runtime and package manager baseline

**Decision**: Use Node.js 22.13 or newer within the Node.js 22 LTS line, pnpm 11.21.0, and TypeScript
5.9.3. Record the package manager in the root manifest and centralize all direct dependency versions in
the `catalog` section of `pnpm-workspace.yml`.

**Rationale**: pnpm 11 requires Node.js 22.13 or newer. This baseline also satisfies Next.js 16
(`>=20.9`), NestJS 11 (`>=20`), Vite 8 (`>=22.12` on Node.js 22), Vitest 4, Playwright 1.62, and
Commitlint 21 (`>=22.12`). TypeScript 5.9 is the latest stable 5.x release and avoids adopting the newly
released TypeScript 7 before every selected framework explicitly confirms support.

**Alternatives considered**: Node.js 20 was rejected because current pnpm and Commitlint releases do
not support it. Node.js 24 was rejected as an unnecessary runtime jump for the workshop baseline.
TypeScript 7 was deferred because foundation stability is more valuable than its new compiler behavior.

## Monorepo orchestration

**Decision**: Use pnpm workspaces for `apps/*` and `packages/*`, and Turborepo 2.10.9 for dependency-
aware root tasks. Define `build`, `typecheck`, `lint`, `test`, and `check` as cacheable where outputs are
stable; define `dev` as persistent and non-cacheable. Root scripts are the canonical local and future
automation entry points.

**Rationale**: Turborepo documents pnpm workspace discovery, dependency ordering with `dependsOn`, and
persistent non-cacheable development tasks. This gives one failure-propagating command per quality gate
without duplicating orchestration logic in future CI.

**Alternatives considered**: Plain recursive pnpm scripts were rejected because they provide weaker
task dependency and cache modeling. Nx was rejected because the constitution already selects
Turborepo and a second orchestrator adds no value.

## Web application baseline

**Decision**: Use Next.js 16.3.0 with React 19.2.8, App Router, TypeScript, a local `@/*` alias, and a
minimal `/api/health` route handler. Configure Tailwind CSS 4.3.3 through `@tailwindcss/postcss` 4.3.3
and `@import "tailwindcss"`. Use Biome rather than adding ESLint.

**Rationale**: Next.js 16 supports App Router route handlers and requires Node.js 20.9 or newer. The
App Router and TypeScript are current defaults, and a route handler supplies a machine-readable web
health contract without introducing product UI behavior. Tailwind CSS 4 officially documents its
PostCSS plugin and CSS import as the Next.js integration path.

**Alternatives considered**: A static page alone was rejected because it cannot expose a stable JSON
health contract. Adding ESLint was rejected because Biome is the approved formatter and linter and two
overlapping lint stacks would increase configuration drift. Tailwind CSS 3 configuration was rejected
because the approved stack can use the current CSS-first Tailwind CSS 4 setup.

## shadcn/ui and theme baseline

**Decision**: Use the `shadcn` 4.17.0 CLI as a pinned development tool to copy owned component source
into `apps/web/src/components/ui/`; do not treat shadcn/ui as a runtime component package. Configure the
Next.js application through `apps/web/components.json`, use CSS variables in
`apps/web/src/app/globals.css`, and manage light, dark, and system modes with `next-themes` 0.4.6 through
`apps/web/src/components/theme-provider.tsx`. Copy only the shadcn/ui Button to
`apps/web/src/components/ui/button.tsx` and use it from `apps/web/src/components/mode-toggle.tsx` for
the foundation theme control; evaluate shadcn/ui before writing any other reusable UI.

**Rationale**: The official shadcn/ui documentation describes a code distribution model in which the
CLI copies component source into the application. `components.json` records style, aliases, global CSS,
React Server Component support, and CSS-variable choices. The official Next.js dark-mode guide uses
`next-themes`, a root theme provider, class-based themes, and `suppressHydrationWarning`. This preserves
the default shadcn/ui visual language while keeping the generated component source reviewable.

**Alternatives considered**: Installing a fictional shadcn/ui runtime component library was rejected
because shadcn/ui distributes source code. Hand-writing an equivalent reusable component was rejected
because the constitution requires evaluating shadcn/ui first. A custom theme context was rejected
because the official Next.js integration already uses `next-themes` and supports system preference.

## API and MongoDB health

**Decision**: Use NestJS 11.1.29 with Fastify 5.11.3, Terminus 11.1.1, Mongoose 9.9.2, and MongoDB 8.
Expose `GET /health`; report healthy only when the API process and its MongoDB connection are ready.

**Rationale**: NestJS officially documents the Fastify adapter, Fastify injection for end-to-end
tests, and Terminus health aggregation. Terminus provides a Mongoose health indicator, allowing the API
endpoint to prove the required database integration while keeping database knowledge out of the web
application.

**Alternatives considered**: A custom MongoDB ping was rejected because Terminus already provides the
framework-supported abstraction. Connecting the web directly to MongoDB was rejected because the API
is the server authority. Adding authentication or domain collections was rejected as out of scope.

## Automated integrated validation

**Decision**: Use Playwright 1.62.1 for a repository-level health journey. The validation starts the
Docker Compose environment, waits for Compose health checks, verifies the web page and web health
contract, verifies the API health contract includes MongoDB readiness, and always tears the environment
down.

**Rationale**: Playwright can verify browser and HTTP behavior in one supported test runner. The API
health response makes MongoDB state externally testable, and Compose health dependencies bound startup
to the two-minute requirement.

**Alternatives considered**: Cypress was not selected because Playwright covers browser and API
requests without another request library. A shell-only smoke test was rejected because it would not
exercise the user-facing page in a real browser. Testcontainers was deferred because this feature must
also validate the delivered Compose environment itself.

## Local commit gates

**Decision**: Use Lefthook 2.1.10. Run Biome checks on changed supported files in `pre-commit`, and run
Commitlint 21.2.1 with the Conventional Commits configuration in `commit-msg`. Keep full root quality
gates separate from commit-time checks.

**Rationale**: This matches the clarified requirement for message validation plus changed-file checks
while preserving fast commits. Lefthook provides one cross-platform hook configuration and avoids an
additional staged-file dependency with a stricter Node.js requirement.

**Alternatives considered**: Running every quality gate on every commit was rejected due to feedback
time. `lint-staged` was rejected as redundant with Lefthook's changed-file support. Message-only
validation was rejected by clarification.

## Configuration and secret handling

**Decision**: Commit only `.env.example` with non-secret local defaults and required variable names.
Validate required values before application bootstrap and before integrated startup; ignore `.env*`
except explicit examples. Turborepo task hashes list only environment variables that affect outputs.

**Rationale**: Early validation prevents partial startup, meets FR-015, and keeps real secrets outside
source control. Explicit Turborepo environment inputs prevent stale cache reuse without logging values.

**Alternatives considered**: Implicit framework validation was rejected because it fails after work has
already started. Committing a usable `.env` was rejected because local convenience does not justify
normalizing secret files in version control.

## Sources reviewed

- Context7 `/vercel/next.js/v16.2.9`: Node.js engine, TypeScript setup, App Router route handlers, and
  lint script behavior. The package registry confirmed Next.js 16.3.0 as the current release.
- Context7 `/nestjs/docs.nestjs.com`: Fastify adapter, Fastify test injection, and Terminus indicators.
- Context7 `/vercel/turborepo`: pnpm workspace setup, package-manager declaration, task dependencies,
  persistent development tasks, caching, and environment hashing.
- Context7 `/tailwindlabs/tailwindcss.com`: Tailwind CSS 4 Next.js setup with `@tailwindcss/postcss`,
  `@import "tailwindcss"`, and class-based dark variants.
- Context7 `/shadcn-ui/ui/shadcn_3.5.0`: copied-source distribution, `components.json`, CSS-variable
  themes, and the Next.js `next-themes` integration.
- Package registry metadata queried on 2026-08-12 for every direct version recorded in this plan.
