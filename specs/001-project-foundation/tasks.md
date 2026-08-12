---

description: "Tarefas de implementação ordenadas por dependência para a Project Foundation"
---

# Tarefas: Project Foundation

**Input**: Design documents from `/specs/001-project-foundation/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Tests are required because the specification explicitly requires automated integrated
validation and the constitution requires Red-Green-Refactor with 100% coverage for shared Zod schemas.

**Organization**: Tasks are grouped by user story so each increment has an explicit independent test.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it affects different files and has no dependency on unfinished
  tasks in the same phase.
- **[Story]**: Maps implementation work to `US1`, `US2`, or `US3` from `spec.md`.
- Every task includes the exact path or paths it changes.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish root metadata, workspace discovery, and the directories required by later work.

- [ ] T001 Criar metadados raiz para Node.js 22 e pnpm 11 e centralizar todas as versões diretas no catálogo de `pnpm-workspace.yml`, com scripts canônicos em `package.json` e versão em `.node-version`
- [ ] T002 Configurar descoberta dos workspaces e tarefas ordenadas por dependência em `turbo.json`
- [ ] T003 [P] Criar manifests usando exclusivamente referências `catalog:` para dependências diretas em `apps/web/package.json`, `apps/api/package.json`, `packages/typescript-config/package.json`, `packages/biome/package.json`, `packages/vitest-config/package.json` e `packages/schemas/package.json`
- [ ] T004 [P] Adicionar regras para Node.js, Next.js, Turborepo, cobertura, Playwright, Docker e ambientes locais em `.gitignore` e `.dockerignore`

**Checkpoint**: pnpm discovers both applications and all four shared packages from the repository root.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Provide shared strict TypeScript, formatting, testing, and contract foundations required by
all user stories.

**CRITICAL**: No user story implementation begins until this phase passes.

- [ ] T005 [P] Definir presets TypeScript estritos em `packages/typescript-config/base.json`, `packages/typescript-config/nextjs.json` e `packages/typescript-config/nestjs.json`
- [ ] T006 [P] Definir o preset Biome com ponto e vírgula, aspas duplas, quatro espaços e 130 colunas em `packages/biome/biome.json`
- [ ] T007 [P] Definir presets reutilizáveis de base e cobertura do Vitest em `packages/vitest-config/base.ts`, `packages/vitest-config/coverage.ts` e `packages/vitest-config/index.ts`
- [ ] T008 Escrever testes falhando para payloads válidos, indisponíveis, duplicados, com vazamento de segredo e equivalência com `specs/001-project-foundation/contracts/health.openapi.yaml` em `packages/schemas/src/health.test.ts`
- [ ] T009 Implementar e exportar schemas Zod de saúde e tipos inferidos em `packages/schemas/src/health.ts` e `packages/schemas/src/index.ts`
- [ ] T010 Configurar 100% de statements, branches, functions e lines em `packages/schemas/vitest.config.ts` e verificar os testes dos schemas

**Checkpoint**: Shared configuration packages resolve, and `packages/schemas` passes its tests and 100%
coverage gate.

---

## Phase 3: User Story 1 - Prepare the Local Environment (Priority: P1) MVP

**Goal**: A developer on macOS or Linux can validate prerequisites, install the complete workspace from
the root, and receive an immediate actionable failure for missing or invalid configuration.

**Independent Test**: In a clean clone, copy `.env.example`, run the preflight, and install with the
frozen lockfile; then repeat with an unsupported tool version, invalid port, conflicting ports, and
missing required variable and confirm each fails before installation or service startup.

### Tests for User Story 1

> Write these tests first and confirm they fail for the expected missing behavior.

- [ ] T011 [US1] Escrever testes falhando de preflight para versões, ferramentas ausentes, ambiente inválido, conflito de portas e erros sem valores em `tests/integration/check-prerequisites.test.sh`

### Implementation for User Story 1

- [ ] T012 [US1] Implementar validação fail-fast de pré-requisitos, ambiente e portas sem registrar valores em `scripts/check-prerequisites.sh`
- [ ] T013 [P] [US1] Documentar nomes e padrões locais não sensíveis em `.env.example`
- [ ] T014 [P] [US1] Gerar e verificar o lockfile reproduzível do workspace em `pnpm-lock.yaml`
- [ ] T015 [US1] Adicionar comandos raiz de setup e preflight e incluir `tests/integration/check-prerequisites.test.sh` no gate `test` em `package.json`
- [ ] T016 [US1] Documentar pré-requisitos macOS/Linux, clone limpo, falhas e meta de 15 minutos em `README.md`

**Checkpoint**: User Story 1 is independently usable without starting the applications or MongoDB.

---

## Phase 4: User Story 2 - Run Quality Checks (Priority: P2)

**Goal**: Root commands consistently run format, lint, type-check, test, and build across all applicable
workspaces, and commit hooks validate changed files and Conventional Commit messages.

**Independent Test**: Run each root quality command successfully, introduce one isolated formatting,
type, and test failure in turn, and verify the corresponding command fails with the originating
workspace; verify invalid commit messages and changed-file violations are rejected.

### Tests for User Story 2

> Write these tests first and confirm they fail before completing orchestration and hooks.

- [ ] T017 [P] [US2] Escrever testes falhando do grafo para cobertura, propagação de falha e execução única em `tests/integration/quality-gates.test.ts`
- [ ] T018 [P] [US2] Escrever testes falhando dos hooks para Biome em arquivos alterados e Conventional Commits em `tests/integration/commit-hooks.test.ts`

### Implementation for User Story 2

- [ ] T019 [US2] Configurar `format:check`, `lint`, `typecheck`, `test`, `build` e `check` com `dependsOn` e outputs de cache, além de `dev` persistente não cacheável, em `turbo.json`
- [ ] T020 [US2] Adicionar scripts canônicos e garantir que `pnpm test` e `pnpm check` executem testes Vitest e `tests/integration/check-prerequisites.test.sh` em `package.json` e `vitest.config.ts`
- [ ] T021 [P] [US2] Configurar aliases locais `@/` e presets em `apps/web/tsconfig.json`, `apps/api/tsconfig.json`, `apps/web/vitest.config.ts` e `apps/api/vitest.config.ts`
- [ ] T022 [P] [US2] Configurar Biome em arquivos alterados e regras Conventional Commits em `lefthook.yml` e `commitlint.config.mjs`
- [ ] T023 [US2] Adicionar scripts de qualidade aos manifests em `apps/web/package.json`, `apps/api/package.json`, `packages/typescript-config/package.json`, `packages/biome/package.json`, `packages/vitest-config/package.json` e `packages/schemas/package.json`
- [ ] T024 [US2] Documentar quality gates, falhas esperadas, hooks e reúso por automação em `README.md`

**Checkpoint**: User Story 2 independently detects violations in every applicable workspace and blocks
invalid local commits.

---

## Phase 5: User Story 3 - Start the Integrated Environment (Priority: P3)

**Goal**: Start the minimal web application, API, and MongoDB reproducibly, expose contract-validated
health endpoints, prove all components are healthy within two minutes, and tear everything down cleanly.

**Independent Test**: From a prepared clone with no running Compose services, run `pnpm test:e2e` in an
isolated Compose project with a fresh volume; verify the browser page in light and dark modes, web health
contract, API health contract, and MongoDB readiness pass within two minutes, then verify managed
services and the acceptance volume are removed on both success and forced failure.

### Tests for User Story 3

> Write contract and integration tests first and confirm each fails for the expected missing behavior.

- [ ] T025 [P] [US3] Escrever testes falhando do contrato da API e MongoDB indisponível em `apps/api/test/health.e2e-spec.ts`
- [ ] T026 [P] [US3] Escrever testes falhando da página, provider em modos claro/escuro e rota de saúde web em `apps/web/tests/health.test.ts`
- [ ] T027 [P] [US3] Escrever jornada Playwright falhando para navegador, HTTP e teardown em `tests/e2e/health.spec.ts`

### Implementation for User Story 3

- [ ] T028 [P] [US3] Inicializar NestJS com Fastify e ambiente validado em `apps/api/src/main.ts`, `apps/api/src/app.module.ts` e `apps/api/src/config/environment.ts`
- [ ] T029 [US3] Implementar saúde da API e MongoDB produzindo e validando respostas com `packages/schemas` conforme `specs/001-project-foundation/contracts/health.openapi.yaml` em `apps/api/src/modules/health/health.module.ts` e `apps/api/src/modules/health/health.controller.ts`
- [ ] T030 [P] [US3] Configurar Tailwind CSS 4, Button copiado pelo shadcn CLI e temas claro/escuro em `apps/web/postcss.config.mjs`, `apps/web/components.json`, `apps/web/src/app/globals.css`, `apps/web/src/app/layout.tsx`, `apps/web/src/app/page.tsx`, `apps/web/src/components/theme-provider.tsx`, `apps/web/src/components/mode-toggle.tsx`, `apps/web/src/components/ui/button.tsx` e `apps/web/src/lib/utils.ts`
- [ ] T031 [US3] Implementar rota web de saúde validada por contrato em `apps/web/src/app/api/health/route.ts`
- [ ] T032 [P] [US3] Criar containers de produção com usuários não root em `Dockerfile.api` e `Dockerfile.web`
- [ ] T033 [US3] Orquestrar ordem, saúde, portas, volume persistente de desenvolvimento e encerramento de MongoDB, API e web em `compose.yml`
- [ ] T034 [US3] Configurar Playwright, timeout e Compose isolado com volume novo e teardown garantido de serviços e volume em `playwright.config.ts` e `scripts/test-e2e.sh`
- [ ] T035 [US3] Adicionar scripts de desenvolvimento, saúde, E2E e encerramento e um `pnpm clean` não destrutivo para caches de build/teste sem remover volumes em `package.json` e `scripts/clean.sh`
- [ ] T036 [US3] Documentar inicialização, saúde, validação, encerramento, limpeza não destrutiva de caches e remoção explícita de volumes em `README.md`

**Checkpoint**: All three user stories work together, and User Story 3 satisfies the automated integrated
health requirement.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Validate the completed foundation against all documented acceptance and governance gates.

- [ ] T037 [P] Executar a verificação já definida de equivalência entre Zod e OpenAPI, sem adicionar comportamento após a implementação, em `packages/schemas/src/health.test.ts` e `specs/001-project-foundation/contracts/health.openapi.yaml`
- [ ] T038 [P] Auditar segredos e exceções de ambiente em `.gitignore`, `.dockerignore` e `.env.example`
- [ ] T039 Cronometrar em um clone limpo no macOS a preparação após os pré-requisitos, validar o limite de 15 minutos, executar todos os quality gates e registrar comandos, duração e resultados em `specs/001-project-foundation/quickstart.md`
- [ ] T040 Executar em um host Linux um clone limpo, preflight, instalação e todos os quality gates e registrar os resultados em `specs/001-project-foundation/quickstart.md`
- [ ] T041 Verificar quality gates, cobertura, build, E2E, startup e teardown e registrar evidências em `specs/001-project-foundation/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 - Setup**: Starts immediately.
- **Phase 2 - Foundational**: Depends on Phase 1 and blocks all user stories.
- **Phase 3 - US1**: Depends on Phase 2 and delivers the suggested MVP.
- **Phase 4 - US2**: Depends on Phase 2 and on the workspace/install interface finalized by US1.
- **Phase 5 - US3**: Depends on Phase 2, uses US1 preflight/configuration, and must pass US2 root gates.
- **Phase 6 - Polish**: Depends on all selected user stories.

### User Story Dependency Graph

```text
Setup -> Foundational -> US1 -> US2 -> US3 -> Polish
                       \--------------> US3
```

- **US1 (P1)**: No dependency on another story after Foundational.
- **US2 (P2)**: Uses the root workspace and install contract completed by US1; its behavior remains
  independently testable without running the integrated environment.
- **US3 (P3)**: Uses US1 configuration/preflight and is validated through US2 gates; it remains
  independently testable through `pnpm test:e2e` once those prerequisites exist.

### Within Each User Story

- Write tests first and verify they fail for the expected reason.
- Implement only enough behavior to pass the tests.
- Refactor while preserving passing tests and required coverage.
- Complete the independent test before advancing to the next story.

## Parallel Opportunities

### Setup and Foundational

- T003 and T004 can run in parallel after T001 establishes root metadata.
- T005, T006, and T007 affect independent configuration packages and can run in parallel.

### User Story 1

```text
After T012 defines accepted configuration:
Task T013: Document `.env.example`.
Task T014: Generate and verify `pnpm-lock.yaml`.
```

### User Story 2

```text
Task T017: Test task graph behavior in `tests/integration/quality-gates.test.ts`.
Task T018: Test hook behavior in `tests/integration/commit-hooks.test.ts`.

After root task names are fixed:
Task T021: Configure application TypeScript and Vitest aliases.
Task T022: Configure Lefthook and Commitlint.
```

### User Story 3

```text
Task T025: Test API and MongoDB health.
Task T026: Test web page and web health.
Task T027: Test the integrated Playwright journey.

After shared health schemas are available:
Task T028: Bootstrap the API.
Task T030: Bootstrap the web application.
Task T032: Add both container builds.
```

### Polish

- T037 and T038 can run in parallel because contract reconciliation and secret auditing touch different
  files.

## Implementation Strategy

### MVP First: User Story 1

1. Complete Setup and Foundational.
2. Complete US1 with its failing preflight tests first.
3. Validate a clean-clone preparation and every documented fail-fast case on macOS or Linux.
4. Stop for review before adding repository-wide quality automation.

### Incremental Delivery

1. **US1**: Reproducible workspace preparation and actionable preflight failures.
2. **US2**: Root quality gates and fast commit-time protection.
3. **US3**: Executable web/API/MongoDB environment with automated health validation.
4. **Polish**: Cross-platform evidence, security audit, full quickstart validation.

### Execution Discipline

- Respect task IDs as the default sequential order.
- Run only tasks marked `[P]` concurrently and only after their stated prerequisite.
- Tasks that modify the same file, especially `package.json`, `README.md`, and `quickstart.md`, remain
  sequential even when conceptually independent.
- Mark each task `[X]` only after its tests or checkpoint pass.
