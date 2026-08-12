# Implementation Plan: Project Foundation

**Branch**: `001-project-foundation` | **Date**: 2026-08-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-project-foundation/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Establish a reproducible pnpm and Turborepo workspace with minimal Next.js and NestJS applications,
shared TypeScript, Biome, Vitest, and Zod packages, and a Docker Compose environment for the web, API,
and MongoDB. Root commands orchestrate formatting, linting, type checking, tests, and builds; Lefthook
and Commitlint protect commits; Playwright verifies the integrated health contract on macOS and Linux.

## Technical Context

**Language/Version**: TypeScript 5.9.3 on Node.js 22.13 or newer within the Node.js 22 LTS line

**Primary Dependencies**: pnpm 11.21.0, Turborepo 2.10.9, Next.js 16.3.0, React 19.2.8, Tailwind CSS
4.3.3, `@tailwindcss/postcss` 4.3.3, PostCSS 8.5.26, shadcn CLI 4.17.0, next-themes 0.4.6, NestJS
11.1.29, Fastify 5.11.3, MongoDB 8, Mongoose 9.9.2, Zod 4.4.3, Biome 2.5.8, Lefthook 2.1.10,
Commitlint 21.2.1

**Storage**: MongoDB 8 container with a named local development volume; no domain collections are
introduced by this feature. Normal development startup preserves the volume, while the automated
clean-state acceptance scenario uses an isolated Compose project and fresh volume that it removes on
success or failure.

**Testing**: Vitest 4.1.10 for unit and integration tests; Playwright 1.62.1 for the automated
cross-application health scenario; Docker Compose health checks for service readiness

**Target Platform**: Local development on macOS and Linux; Linux containers through Docker Compose

**Project Type**: Web application monorepo with a Next.js frontend, NestJS API, and shared packages

**Performance Goals**: A clean environment reaches healthy state within two minutes; a prepared clone
completes setup within 15 minutes on a supported development machine

**Constraints**: Strict TypeScript; `@/` resolves to each application's `src/`; Biome uses semicolons,
double quotes, four spaces, and 130 columns; direct versions live in the pnpm catalog; no real secrets;
configuration validation fails before installation or service startup; root gates fail on any workspace;
the minimal web shell follows the default shadcn/ui visual language and renders correctly in light and
dark modes

**Scale/Scope**: Two minimal executable applications, four reusable configuration/contract packages,
one MongoDB service, one public health endpoint per application, and one integrated health journey

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Security by Default: PASS.** Environment examples contain placeholders only, `.env*` remains
  ignored except explicit examples, health responses expose no credentials, and required configuration
  is validated before startup.
- **Test-First in Critical Domains: PASS.** Foundation code is outside the mandatory 100% domains, but
  configuration validation and health contracts receive risk-based tests. Shared Zod health schemas
  follow Red-Green-Refactor and the constitution's 100% coverage requirement.
- **Modular Monorepo and Simplicity: PASS.** Executables live in `apps/`; only configurations and the
  reusable health contract live in `packages/`. No speculative domain layers are introduced.
- **Shared and Validated Contracts: PASS.** Web and API health payloads use one Zod schema from
  `packages/schemas`; TypeScript types are inferred from that schema.
- **Governed Dependencies: PASS.** Current documentation and package metadata were reviewed, every
  direct dependency version is centralized in `pnpm-workspace.yml`, and workspace manifests reference
  those versions exclusively through `catalog:`.
- **Technical Constraints: PASS.** TypeScript is strict, application aliases are local, and Biome
  formatting matches the constitution. The minimal web shell uses the default shadcn/ui visual language
  and supports both light and dark modes; reusable UI is evaluated against shadcn/ui before custom code.
- **Workflow and Quality Gates: PASS.** Tasks and implementation remain gated behind `/speckit.tasks`
  and `/speckit.analyze`; the plan provides format, lint, type-check, test, build, and end-to-end gates.

Post-design re-check: **PASS**. The contracts, operational model, and quickstart add no constitutional
violations or unjustified complexity.

## Project Structure

### Documentation (this feature)

```text
specs/001-project-foundation/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
```text
apps/
├── web/
│   ├── src/
│   │   ├── app/
│   │   │   ├── api/health/route.ts
│   │   │   ├── globals.css
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── components/
│   │   │   ├── ui/button.tsx
│   │   │   ├── mode-toggle.tsx
│   │   │   └── theme-provider.tsx
│   │   └── lib/utils.ts
│   ├── components.json
│   ├── postcss.config.mjs
│   └── tests/
└── api/
    ├── src/
    │   ├── modules/health/
    │   ├── app.module.ts
    │   └── main.ts
    └── test/

packages/
├── biome/
├── schemas/
│   └── src/health.ts
├── typescript-config/
└── vitest-config/

tests/
└── e2e/health.spec.ts

scripts/
├── check-prerequisites.sh
└── clean.sh

compose.yml
Dockerfile.web
Dockerfile.api
lefthook.yml
pnpm-workspace.yml
turbo.json
```

**Structure Decision**: Use the constitution's `apps/` and `packages/` split. Keep health behavior in
each application's existing framework boundary and share only its Zod response contract. Keep the
single cross-application journey at repository level because it owns Docker Compose lifecycle and
verifies the whole system rather than one workspace. Tailwind CSS is configured through PostCSS and
global CSS; the pinned shadcn CLI copies Button source into `apps/web/src/components/ui/button.tsx`
according to `apps/web/components.json`, while `next-themes` and `mode-toggle.tsx` own theme selection.

## Complexity Tracking

No constitutional violations require justification.
