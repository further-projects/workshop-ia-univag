# Implementation Plan: Document Management

**Branch**: `003-document-management` | **Date**: 2026-08-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/003-document-management/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Implementar criação, listagem, pesquisa, leitura, atualização concorrente e exclusão física de documentos
privados em um módulo NestJS com Mongoose. A API filtra toda operação pela identidade da sessão, valida
contratos Zod compartilhados, persiste JSON estruturado como fonte canônica e deriva texto normalizado
para busca parcial. Paginação por cursor usa `updatedAt` e `_id`; atualizações usam precondição HTTP e
comparação atômica de versão para preservar rascunhos locais diante de conflitos.

## Technical Context

**Language/Version**: TypeScript 5.9.3 em Node.js 22.13 ou superior dentro da linha Node.js 22 LTS

**Primary Dependencies**: Next.js 16.3.0, React 19.2.8, NestJS 11.1.29 com Fastify 5.11.3, Mongoose
9.9.2, `@nestjs/mongoose` 11.0.4, MongoDB 8 e Zod 4.4.3; autenticação e cookies conforme
`002-authentication`

**Storage**: Coleção MongoDB `documents` gerenciada por Mongoose, com conteúdo JSON canônico, texto
pesquisável derivado, versão explícita e índice composto por proprietário e ordem de atualização;
exclusão física síncrona no MVP

**Testing**: Vitest 4.1.10 para testes unitários, de integração, contrato e componentes; Playwright
1.62.1 para jornadas essenciais; MongoDB real para índices, isolamento, busca, paginação e concorrência

**Target Platform**: Navegadores modernos no frontend; API Node.js em contêiner Linux; desenvolvimento
local em macOS e Linux

**Project Type**: Aplicação web em monorepo, com frontend Next.js, API NestJS e schemas compartilhados

**Performance Goals**: Pelo menos 95% das listagens e pesquisas em até 1 segundo, medido como p95 em
100 operações aquecidas, com até 10 requisições concorrentes, sobre até 500 documentos por usuário e
50 kB médios de texto pesquisável; leituras e mutações pontuais em até 200 ms p95 sob o mesmo perfil

**Constraints**: TDD e 100% de statements, branches, functions e lines em documentos, autorização e
schemas compartilhados; API como autoridade de sessão e propriedade; título com até 200 pontos de
código Unicode; conteúdo canônico com até 1 MB (1.000.000 bytes UTF-8), 100 níveis e 10.000 nós;
corpos HTTP de mutação limitados a 1.250.000 bytes; respostas privadas sem cache; conflitos nunca descartam o
rascunho local; busca não executa expressão regular fornecida pelo usuário

**Scale/Scope**: Um proprietário por documento, até 500 documentos por usuário no perfil validado,
páginas de 20 itens por padrão e 50 no máximo, cinco endpoints privados, uma tela principal e fluxo de
edição simples; busca maior ou mais sofisticada requer mecanismo indexado próprio

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Security by Default: PASS.** A API obtém o proprietário somente da sessão e inclui `ownerId` em
  toda consulta ou mutação. Recurso inexistente, excluído ou alheio compartilha `404`; conteúdo, título,
  pesquisa, cookies e identificadores não entram em logs. Mutações mantêm a validação de origem.
- **Test-First in Critical Domains: PASS.** Documentos, autorização e schemas compartilhados seguem
  Red-Green-Refactor e mantêm 100% nas quatro métricas, com MongoDB real para concorrência e índices.
- **Modular Monorepo and Simplicity: PASS.** UI fica em `apps/web`, autoridade e persistência em
  `apps/api`, e somente contratos Zod reutilizáveis ficam em `packages/schemas`. Uma coleção e um módulo
  atendem o domínio sem camadas ou serviços externos especulativos.
- **Shared and Validated Contracts: PASS.** Entrada, saída, cursores e erros são definidos uma vez com
  Zod, inferidos em TypeScript e refletidos no contrato OpenAPI.
- **Governed Dependencies: PASS.** Mongoose e sua integração NestJS foram revisados no Context7;
  versões diretas aprovadas permanecem no catálogo central e não há novo mecanismo de busca externo.
- **Technical Constraints: PASS.** TypeScript permanece estrito, aliases `@/` são locais, Biome mantém
  o formato constitucional e a confirmação reutiliza o Alert Dialog do shadcn/ui em claro e escuro.
- **Workflow and Quality Gates: PASS.** Implementação permanece bloqueada até `/speckit.tasks` e
  `/speckit.analyze`; unitários, integração, contrato, componentes, E2E, lint, type-check e build são
  obrigatórios.

Post-design re-check: **PASS**. Modelo, contrato e quickstart preservam isolamento por proprietário,
validação compartilhada, atualização atômica e os gates de cobertura. O texto pesquisável duplicado é
justificado pela busca sem caixa ou acentos; permanece derivado e nunca substitui o conteúdo canônico.

## Project Structure

### Documentation (this feature)

```text
specs/003-document-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md             # Phase 2 output; not created by /speckit.plan
```

### Source Code (repository root)

```text
apps/
├── web/
│   ├── src/
│   │   ├── app/(private)/documents/
│   │   │   ├── [documentId]/page.tsx
│   │   │   └── page.tsx
│   │   ├── components/documents/
│   │   │   ├── delete-document-dialog.tsx
│   │   │   ├── document-editor.tsx
│   │   │   ├── document-list.tsx
│   │   │   └── document-search.tsx
│   │   └── lib/documents/
│   │       ├── document-draft.ts
│   │       └── documents-api.ts
│   └── tests/documents/
└── api/
    ├── src/modules/documents/
    │   ├── documents.controller.ts
    │   ├── documents.module.ts
    │   ├── documents.service.ts
    │   ├── documents.repository.ts
    │   ├── document.schema.ts
    │   ├── document-search.ts
    │   ├── document-cursor.ts
    │   ├── document-security.ts
    │   └── document-observability.ts
    ├── src/infra/mongodb/mongoose.config.ts
    ├── scripts/provision-document-indexes.ts
    └── test/documents/

packages/
└── schemas/
    ├── src/documents.ts
    └── tests/documents.test.ts

tests/
└── e2e/
    ├── documents-create-list.spec.ts
    ├── documents-search.spec.ts
    ├── documents-update.spec.ts
    ├── documents-delete.spec.ts
    └── documents-security.spec.ts
```

**Structure Decision**: Preservar a divisão definida pela foundation e autenticação. O controller
aplica sessão, origem e contratos; o serviço concentra regras; o repositório encapsula apenas operações
Mongoose owner-scoped. A web mantém snapshot confirmado e rascunho local separados. O conteúdo completo
é carregado somente ao abrir um documento; listagem e pesquisa retornam projeções pequenas.

US1 estabelece a lista e a criação como incremento-base. US2, US3 e US4 dependem dessa fundação de
navegação, mas cada uma possui fixtures, contratos e jornadas próprias para validar pesquisa, atualização
e exclusão isoladamente sem depender da execução de outra história posterior.

## Complexity Tracking

Nenhuma violação constitucional requer justificativa.
