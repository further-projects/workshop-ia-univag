# Implementation Plan: Authentication

**Branch**: `002-authentication` | **Date**: 2026-08-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/002-authentication/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Implementar cadastro, login, bloqueio por tentativas, sessão renovável e logout por meio de uma
fachada HTTP estável no NestJS sobre Better Auth e seu adaptador MongoDB. Contratos Zod compartilhados
validam web e API; Argon2id com pepper protege senhas; estado persistido e atualização atômica contêm
tentativas concorrentes; cookies opacos `HttpOnly` mantêm a sessão fora do JavaScript do navegador.

## Technical Context

**Language/Version**: TypeScript 5.9.3 em Node.js 22.13 ou superior dentro da linha Node.js 22 LTS

**Primary Dependencies**: Next.js 16.3.0, React 19.2.8, NestJS 11.1.29 com Fastify 5.11.3, Better Auth
1.6.27, `@better-auth/mongo-adapter` 1.6.27, MongoDB driver 7, `@node-rs/argon2` 2.0.2,
`@nestjs/throttler` 6.5.0, `@fastify/cors` 11.3.0, `@fastify/cookie` 11.1.2 e Zod 4.4.3

**Storage**: MongoDB 8 em replica set de nó único no desenvolvimento e testes; coleções nativas do
Better Auth para usuário, conta e sessão, mais estado de tentativas de autenticação gerenciado pela API

**Testing**: Vitest 4.1.10 para testes unitários, de integração e contrato; Playwright 1.62.1 para as
jornadas essenciais; MongoDB real em replica set para concorrência, índices, expiração e revogação

**Target Platform**: Navegadores modernos no frontend; API Node.js em contêiner Linux; desenvolvimento
local em macOS e Linux

**Project Type**: Aplicação web em monorepo, com frontend Next.js, API NestJS e schemas compartilhados

**Performance Goals**: Validação de sessão e respostas sem Argon2 em até 200 ms p95; cadastro e login
em até 1,5 s p95 sob a carga do MVP; a vigésima primeira tentativa por origem é rejeitada antes do hash.
Testes de integração medem pelo menos 100 operações após aquecimento em ambiente local isolado, com no
máximo 10 requisições concorrentes, e reportam p95 separadamente para sessão, cadastro e login

**Constraints**: 100% de statements, branches, functions e lines em autenticação, autorização e
schemas compartilhados; TDD; nenhuma resposta ao navegador contém token de sessão; bloqueios por conta
e origem são atômicos e persistem após reinício; cookies seguros; API é autoridade; operações privadas
sempre consultam a sessão stateful e o estado ativo do usuário; cadastro novo e duplicado respeitam o
mesmo piso de 750 ms antes da resposta para reduzir diferenças temporais observáveis. O cookie
`mini_notion.session` é host-only, `HttpOnly`, `Path=/`, `SameSite=Lax`, `Secure` fora do desenvolvimento
local, expira após sete dias de inatividade e é renovado no máximo uma vez a cada 24 horas

**Scale/Scope**: Um tipo de usuário, cadastro/login por e-mail e senha, quatro endpoints públicos de
autenticação, duas telas públicas e proteção das rotas privadas; preparado para múltiplas réplicas da
API por manter coordenação e limites no MongoDB

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Security by Default: PASS.** A API valida sessão e estado ativo, a fachada elimina tokens das
  respostas, cookies são opacos e `HttpOnly`, erros evitam enumeração, senhas usam Argon2id com pepper e
  segredos permanecem no ambiente. Operações mutáveis validam origem confiável.
- **Test-First in Critical Domains: PASS.** Autenticação, autorização e schemas compartilhados seguem
  Red-Green-Refactor e mantêm 100% de cobertura nas quatro métricas, incluindo concorrência e falhas.
- **Modular Monorepo and Simplicity: PASS.** Interface fica em `apps/web`, autoridade em `apps/api` e
  somente contratos Zod reutilizáveis ficam em `packages/schemas`. Better Auth continua responsável por
  credenciais e sessões; não há modelo Mongoose paralelo.
- **Shared and Validated Contracts: PASS.** Cadastro, login, sessão e erros são definidos uma vez com
  Zod, inferidos em TypeScript e refletidos no contrato OpenAPI.
- **Governed Dependencies: PASS.** Documentação atual do Better Auth, NestJS Throttler e integrações
  Fastify foi consultada; versões diretas serão adicionadas ao catálogo central antes do uso.
- **Technical Constraints: PASS.** TypeScript permanece estrito, aliases `@/` são locais, Biome mantém
  o formato constitucional e os formulários reutilizam componentes shadcn/ui adequados com temas claro
  e escuro.
- **Workflow and Quality Gates: PASS.** Implementação permanece bloqueada até `/speckit.tasks` e
  `/speckit.analyze`; unitários, integração, contrato, E2E, lint, type-check e build são obrigatórios.

Post-design re-check: **PASS**. O modelo, a fachada HTTP e os cenários do quickstart preservam os
limites acima. O estado adicional de tentativas é justificado por bloqueio concorrente, rate limiting
distribuído e equivalência observável para identidades inexistentes, capacidades ausentes no login
padrão do Better Auth.

## Project Structure

### Documentation (this feature)

```text
specs/002-authentication/
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
│   │   │   ├── (auth)/login/page.tsx
│   │   │   ├── (auth)/register/page.tsx
│   │   │   └── (private)/layout.tsx
│   │   ├── components/auth/
│   │   └── lib/auth/
│   └── tests/
└── api/
    ├── src/
    │   ├── modules/auth/
    │   │   ├── auth.controller.ts
    │   │   ├── auth.service.ts
    │   │   ├── auth-session.guard.ts
    │   │   ├── auth-session.interceptor.ts
    │   │   ├── auth-attempt.repository.ts
    │   │   └── auth.config.ts
    │   └── infra/mongodb/
    └── test/auth/

packages/
└── schemas/
    ├── src/auth.ts
    └── tests/auth.test.ts

tests/
└── e2e/auth.spec.ts

compose.yml
```

**Structure Decision**: Preservar a divisão `apps/` e `packages/` definida pela foundation. A API
expõe uma fachada pequena e estável em vez dos endpoints nativos do Better Auth, pois estes podem
retornar token no corpo e não implementam o bloqueio por conta. O módulo chama a API server-side do
Better Auth para hash, credenciais e sessões, enquanto um repositório MongoDB dedicado coordena apenas
tentativas e bloqueios. A web consome os contratos compartilhados e acessa a API pela mesma origem ou
proxy para manter cookies host-only.

O incremento US1 entrega `GET /api/auth/session` com leitura stateful autoritativa, validação de usuário
ativo e redirecionamento privado, suficiente para proteger a tela principal criada pelo cadastro. O US4
não substitui essa proteção: adiciona renovação após 24 horas, propagação de `Set-Cookie`, logout
idempotente, expiração, revogação, desativação e a abstração reutilizável de propriedade. Operações de
documentos aplicarão essa abstração em `003-document-management`.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

Nenhuma violação constitucional requer justificativa.
