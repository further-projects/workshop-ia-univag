# Implementation Plan: Rich Text Editor

**Branch**: `004-rich-text-editor` | **Date**: 2026-08-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/004-rich-text-editor/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Implementar edição Rich Text privada com Tiptap em um Client Component Next.js, usando uma allowlist
compartilhada de JSON ProseMirror para texto, H1-H3, listas, alinhamento, cores e tamanhos semânticos. A
web deriva toolbar e sumário do estado vivo do editor, salva revisões serialmente após 1,5 segundo de
inatividade e reutiliza ETags e endpoints da gestão de documentos. A API continua autoridade de sessão,
propriedade, limites e validação estrutural; conflitos preservam o rascunho local sem mesclagem.

## Technical Context

**Language/Version**: TypeScript 5.9.3 em Node.js 22.13 ou superior dentro da linha Node.js 22 LTS

**Primary Dependencies**: Next.js 16.3.0, React 19.2.8, Tiptap 3.30.0 (`core`, `pm`, `react`,
StarterKit, TextStyle, TextAlign, Underline, UniqueID e TableOfContents), shadcn/ui, Tailwind CSS 4.3.3,
NestJS 11.1.29 com Fastify 5.11.3, Mongoose 9.9.2 e Zod 4.4.3

**Storage**: Reutiliza a coleção MongoDB `documents` de `003-document-management`; JSON ProseMirror
allowlisted permanece canônico, headings recebem IDs UUID persistidos e nenhum HTML, Markdown,
rascunho auxiliar ou estado de seleção é persistido

**Testing**: Vitest 4.1.10 para schemas, estado de autosave e componentes; Playwright 1.62.1 para
seleção, colagem, teclado, responsividade, hidratação e conflitos em dois contextos; axe-core 4.13.0
para verificações automatizadas de acessibilidade; MongoDB real nos testes de integração da API

**Target Platform**: Navegadores modernos em desktop e mobile; frontend Next.js e API Node.js em
contêiner Linux; desenvolvimento local em macOS e Linux

**Project Type**: Aplicação web em monorepo, com frontend Next.js, API NestJS e schemas compartilhados

**Performance Goals**: Mudanças de título atualizam o sumário no próximo ciclo de renderização; acionar
um item torna o heading visível em até 1 segundo; ações locais da toolbar respondem em até 100 ms p95;
autosave inicia 1,5 segundo após a última alteração e preserva as metas de mutação da feature 003

**Constraints**: TDD e 100% de statements, branches, functions e lines em documentos, autorização e
schemas compartilhados; conteúdo com até 1.000.000 bytes, 100 níveis e 10.000 nós; um save em voo;
sem execução de HTML ativo, armazenamento paralelo de rascunhos, merge automático ou perda silenciosa;
API valida sessão, propriedade, origem, ETag e allowlist; toolbar e painéis operáveis por teclado

**Scale/Scope**: Um editor privado, uma toolbar sticky, sidebar com paginação incremental sobre até 500
documentos no perfil validado, sumário H1-H3, seis tokens de cor, quatro tamanhos, dois painéis
responsivos abaixo de 1024 CSS pixels e reutilização dos quatro endpoints necessários da feature 003

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Security by Default: PASS.** A API mantém sessão, propriedade e origem como autoridades, valida o
  JSON allowlisted antes de persistir e nunca renderiza ou executa conteúdo ativo. Conteúdo, título,
  seleção, clipboard, ETag e identificadores privados não entram em logs.
- **Test-First in Critical Domains: PASS.** O schema Rich Text compartilhado e as alterações no domínio
  de documentos seguem Red-Green-Refactor e mantêm 100% nas quatro métricas; integração usa MongoDB real
  e E2E cobre conflitos, paste hostil e jornadas essenciais.
- **Modular Monorepo and Simplicity: PASS.** Componentes do editor ficam em `apps/web`, validação
  autoritativa permanece no módulo de documentos em `apps/api` e somente o contrato reutilizável fica em
  `packages/schemas`. Não há novo serviço, coleção ou API.
- **Shared and Validated Contracts: PASS.** `richTextContentSchema` especializa as escritas da feature
  003 e é consumido pela web e API com tipos inferidos. Um contrato OpenAPI complementar estreita POST e
  PATCH; leituras permanecem compatíveis com conteúdo legado e são verificadas antes de inicializar o
  editor. O schema ProseMirror executável vive em um pacote reutilizável único.
- **Governed Dependencies: PASS.** APIs atuais de Tiptap e Next.js foram verificadas no Context7; versões
  Tiptap 3.30.0 e axe-core 4.13.0 foram confirmadas no registro e serão centralizadas no catálogo pnpm.
- **Technical Constraints: PASS.** TypeScript permanece estrito, aliases `@/` são locais e Biome mantém
  o formato constitucional. Toolbar, menus e painéis avaliam/reutilizam primitivas shadcn/ui, preservando
  linguagem visual e temas claro/escuro.
- **Workflow and Quality Gates: PASS.** Implementação permanece bloqueada até `/speckit.tasks` e
  `/speckit.analyze`; schema, unitários, componentes, integração, contrato, E2E, acessibilidade, lint,
  type-check e build são gates obrigatórios.

Post-design re-check: **PASS**. O modelo separa conteúdo persistido de estado transitório, o contrato
estreita a API existente e o quickstart prova segurança, limites, acessibilidade e recuperação sem
introduzir autoridade no cliente ou dependências não governadas.

## Project Structure

### Documentation (this feature)

```text
specs/004-rich-text-editor/
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
│   │   ├── app/(private)/documents/[documentId]/page.tsx
│   │   ├── components/editor/
│   │   │   ├── rich-text-editor.tsx
│   │   │   ├── editor-toolbar.tsx
│   │   │   ├── editor-shell.tsx
│   │   │   ├── document-sidebar.tsx
│   │   │   └── table-of-contents.tsx
│   │   └── lib/editor/
│   │       ├── editor-content.ts
│   │       ├── heading-index.ts
│   │       ├── document-sidebar-state.ts
│   │       └── autosave-machine.ts
│   └── tests/editor/
└── api/
    ├── src/modules/documents/
    │   ├── document-content.ts
    │   ├── documents.controller.ts
    │   └── documents.service.ts
    └── test/documents/

packages/
├── editor-schema/
│   ├── src/index.ts
│   └── tests/editor-schema.test.ts
└── schemas/
    ├── src/
    │   ├── documents.ts
    │   └── rich-text.ts
    └── tests/rich-text.test.ts

tests/
└── e2e/
    ├── editor-formatting.spec.ts
    ├── editor-navigation.spec.ts
    ├── editor-autosave.spec.ts
    ├── editor-conflict.spec.ts
    └── editor-accessibility.spec.ts
```

**Structure Decision**: Preservar os destinos definidos pelas features 001-003. A rota privada carrega
o snapshot e entrega o editor a um Client Component. Componentes separam shell, toolbar, navegação e
sumário por responsabilidade concreta; `lib/editor` concentra transformação segura, índice de headings,
paginação da sidebar e máquina de autosave testáveis sem UI. `packages/editor-schema` é o único
proprietário da configuração ProseMirror/Tiptap consumida por web e API; `packages/schemas` continua
fonte dos contratos Zod. A API apenas endurece escritas no módulo já owner-scoped, enquanto leituras
genéricas permitem que o editor detecte conteúdo legado sem o sobrescrever.

## Complexity Tracking

Nenhuma violação constitucional requer justificativa.
