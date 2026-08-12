# Arquitetura do Projeto

Este documento reúne as decisões globais de stack e a arquitetura de alto nível do Mini Notion. O
desenho detalhado de cada feature pertence ao respectivo `plan.md` em [`specs/`](../specs/README.md).

## Stack aprovada

### Monorepo e qualidade

- pnpm e Turborepo.
- TypeScript.
- Biome.
- Vitest e uma ferramenta de testes end-to-end a ser definida durante o plano da foundation.
- Zod para schemas e contratos compartilhados.
- Commitlint e Lefthook.
- Docker para ambientes das aplicações e do MongoDB.

### Web

- Next.js e React.
- Tailwind CSS.
- shadcn/ui.
- Tiptap.

### API e dados

- NestJS com Fastify.
- MongoDB.
- Better Auth com seu adaptador oficial do MongoDB.
- Mongoose para modelos pertencentes ao domínio da aplicação.

As versões serão escolhidas somente durante o planejamento técnico, após consulta ao Context7, e
centralizadas no `catalog` de `pnpm-workspace.yml`.

## Arquitetura de alto nível

A estrutura abaixo representa o destino planejado. As pastas serão criadas conforme as features forem
planejadas e implementadas.

```text
apps/
├── web/
│   └── src/
│       ├── app/
│       ├── components/
│       └── styles/
└── api/
    └── src/
        ├── modules/
        ├── shared/
        └── infra/

packages/
├── typescript-config/
├── schemas/
├── biome/
└── vitest-config/
```

Aplicações executáveis ficam em `apps/`; contratos e configurações realmente reutilizáveis ficam em
`packages/`. A estrutura detalhada será definida no plano de cada feature. Novas pastas ou camadas
devem existir por responsabilidade concreta, não apenas para reproduzir um padrão.

## Responsabilidades gerais

- `apps/web` oferece a interface, mas não é autoridade para autenticação ou autorização;
- `apps/api` valida sessões, propriedade dos recursos, contratos e regras de negócio no servidor;
- `packages/schemas` concentra contratos reutilizáveis definidos com Zod;
- pacotes de configuração mantêm regras consistentes entre os workspaces sem duplicação.

## Documentos relacionados

- [Visão do produto](product-overview.md)
- [Decisões técnicas](technical-decisions.md)
- [Constituição do projeto](../.specify/memory/constitution.md)
