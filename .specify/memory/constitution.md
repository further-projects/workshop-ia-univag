<!--
Sync Impact Report
- Version change: template without version -> 1.0.0
- Added principles:
  - I. Security by Default
  - II. Test-First in Critical Domains
  - III. Modular Monorepo and Simplicity
  - IV. Shared and Validated Contracts
  - V. Governed Dependencies
- Added sections:
  - Technical and Language Constraints
  - Development Workflow and Quality Gates
- Removed sections: none; template placeholders were replaced.
- Deferred TODOs: none.
-->

# Mini Notion Constitution

## Core Principles

### I. Security by Default

Authentication, authorization, session management, password handling, input validation and private
data access MUST use secure defaults. Protected operations MUST validate the authenticated session
and resource ownership on the server. Secrets MUST remain outside source control and persisted user
passwords MUST be one-way hashes. Security-sensitive errors MUST avoid account enumeration and
information leakage. Deviations require a documented threat analysis and explicit approval.

### II. Test-First in Critical Domains

Authentication, authorization, document management and shared validation schemas MUST follow the
Red-Green-Refactor cycle. Tests and required mocks MUST be written before production behavior. These
critical domains MUST maintain 100% coverage for statements, branches, functions and lines. Other
modules have no arbitrary global percentage target, but MUST receive unit, integration or end-to-end
tests proportional to their risk. Coverage does not replace meaningful assertions or behavior tests.

### III. Modular Monorepo and Simplicity

The project MUST remain a pnpm and Turborepo monorepo, with applications under `apps/` and reusable
packages under `packages/`. Modules MUST have focused responsibilities, explicit dependencies and
small public surfaces. SOLID, object-oriented design and reuse MUST be applied when they reduce real
coupling or duplication; speculative abstractions and unnecessary layers MUST NOT be introduced.

### IV. Shared and Validated Contracts

Data crossing application, API or persistence boundaries MUST be validated. Reusable contracts MUST
be defined once in a shared package using Zod and consumed through inferred TypeScript types. API
behavior, schemas and error semantics MUST stay consistent across web and API applications. Breaking
contract changes require updated tests and explicit documentation in the relevant feature artifacts.

### V. Governed Dependencies

Library, framework, SDK, API and CLI decisions MUST be checked against current Context7
documentation before adoption or modification. Direct dependency versions MUST be declared in the
`catalog` of `pnpm-workspace.yml` and referenced from workspace packages. Compatibility and security
MUST be reviewed together; unmaintained, duplicated or unnecessary dependencies MUST NOT be added.

## Technical and Language Constraints

- Source code, identifiers, code comments, API contracts and code-facing documentation MUST be in
  English. Product planning, specifications and repository operating guidance MAY be in Portuguese.
- TypeScript MUST use strict settings. Each application MUST expose `@/` as an alias for its own
  `src/` directory, and all relevant tools MUST resolve the alias consistently.
- Biome formatting MUST use semicolons, double quotes, four-space indentation and a 130-character
  line width.
- Reusable web UI MUST first be evaluated against the approved shadcn/ui component set. Shared
  variants and class composition SHOULD use the established `cn` and `cva` utilities.
- The default shadcn/ui visual language and light/dark themes MUST remain the baseline unless an
  approved feature specification states otherwise.

## Development Workflow and Quality Gates

Every feature MUST progress through an approved specification, clarification when necessary,
implementation plan and dependency-ordered task list before implementation. `$speckit-analyze` MUST
validate complete spec, plan and tasks artifacts before `$speckit-implement` begins. Implementation
MUST preserve the Red-Green-Refactor order and pass applicable unit, integration, end-to-end, lint,
type-check and build gates. Plans MUST justify complexity or deviations from this constitution.

## Governance

This constitution is the authority for permanent engineering principles. `AGENTS.md` defines agent
operating behavior, feature specs define product behavior, and plans and tasks define implementation.
Lower-level artifacts MUST NOT override this constitution. Amendments require a documented reason,
an impact review of active specs and plans, and semantic versioning: MAJOR for incompatible governance
changes, MINOR for new or materially expanded principles, and PATCH for clarifications. Every planning
and review workflow MUST perform a constitution compliance check.

**Version**: 1.0.0 | **Ratified**: 2026-08-11 | **Last Amended**: 2026-08-11
