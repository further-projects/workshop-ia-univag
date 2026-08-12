# Mini Notion — Instruções para Agentes

Este arquivo define como agentes de IA devem trabalhar neste repositório. Ele não é a especificação
completa do produto.

## Fontes de verdade

- [Constituição](.specify/memory/constitution.md): princípios permanentes e quality gates.
- [Visão do projeto](README.md): objetivo, stack, arquitetura e decisões globais.
- [Índice de specs](specs/README.md): funcionalidades, ordem e estado do planejamento.
- `specs/<feature>/spec.md`: comportamento esperado e critérios de aceitação da feature.
- `specs/<feature>/plan.md` e `tasks.md`: decisões e trabalho de implementação, quando existirem.

Em caso de conflito, a constituição prevalece sobre specs, planos e tarefas. Este arquivo prevalece
para o comportamento operacional dos agentes.

## Limites de atuação

- Antes de editar, criar, mover ou excluir arquivos, solicitar autorização explícita ao usuário para
  a operação e informar os caminhos afetados.
- Não implementar funcionalidades enquanto a solicitação estiver restrita a planejamento.
- Preservar alterações locais do usuário e nunca sobrescrever trabalho não relacionado.
- Não criar `plan.md`, `tasks.md` ou código sem solicitação explícita.
- Usar os fluxos oficiais do Spec Kit para alterar artefatos em `.specify/` e `specs/`.

## Fluxo Spec Kit

Cada feature deve seguir esta sequência:

1. `$speckit-specify`: criar ou atualizar a especificação.
2. `$speckit-clarify`: resolver ambiguidades relevantes.
3. `$speckit-plan`: pesquisar e produzir o desenho técnico.
4. `$speckit-tasks`: gerar tarefas ordenadas e testáveis.
5. `$speckit-analyze`: verificar consistência entre spec, plano e tarefas.
6. `$speckit-implement`: implementar somente após os gates anteriores.
7. `$speckit-converge`: identificar trabalho restante quando necessário.

Use `$speckit-constitution` somente para princípios permanentes e `$speckit-checklist` para
checklists específicos de uma feature.

## Idioma

- Planejamento, specs, decisões de produto e instruções do repositório: português.
- Código-fonte, identificadores, comentários de código, contratos de API e documentação voltada ao
  código: inglês.
- Nomes de bibliotecas, comandos e termos técnicos podem manter a grafia oficial.

## Context7

Use o Context7 MCP sempre que a tarefa envolver documentação, configuração, compatibilidade,
migração ou APIs de bibliotecas, frameworks, SDKs, ferramentas CLI ou serviços.

1. Chame `resolve-library-id` com o nome oficial e o assunto pesquisado, salvo quando o usuário
   fornecer um ID exato no formato `/org/project`.
2. Escolha o resultado pela correspondência do nome, relevância, reputação, cobertura e benchmark.
3. Chame `query-docs` com uma pergunta específica e um único conceito por consulta.
4. Baseie a decisão na documentação retornada e declare quando a fonte não estiver indexada.

Não use Context7 para lógica de negócio, refatoração genérica, revisão de código ou scripts sem
dependências externas.

## Arquitetura e qualidade

- Manter o monorepo pnpm + Turborepo com aplicações em `apps/` e pacotes reutilizáveis em
  `packages/`.
- Aplicar SOLID, orientação a objetos e Clean Code de forma pragmática; evitar abstrações sem uso real.
- Usar TDD e Red-Green-Refactor. Autenticação, autorização, documentos e schemas compartilhados
  exigem 100% de statements, branches, functions e lines.
- Cobrir os demais módulos conforme o risco com testes unitários, de integração ou end-to-end; não há
  percentual global mínimo.
- Definir contratos reutilizáveis com Zod e inferir os tipos TypeScript a partir dos schemas.
- Configurar `@/` para apontar ao `src/` de cada aplicação em todas as ferramentas relevantes.
- Configurar Biome com ponto e vírgula, aspas duplas, quatro espaços e largura máxima de 130 colunas.

## Dependências e interface web

- Verificar documentação atual, compatibilidade e segurança antes de escolher ou atualizar
  dependências.
- Declarar todas as versões diretas no `catalog` de `pnpm-workspace.yml`.
- Antes de criar um componente reutilizável, verificar se shadcn/ui já oferece uma base adequada.
- Manter o tema padrão do shadcn/ui, com modos claro e escuro.
- Usar as utilidades aprovadas, como `cn` e `cva`, para composição de classes e variantes.

## RTK

Sempre prefixe comandos de terminal com `rtk`. Em comandos encadeados, cada comando deve receber o
prefixo:

```bash
rtk pnpm install && rtk pnpm test
```

Prefira os filtros específicos, como `rtk read`, `rtk grep`, `rtk git status`, `rtk lint` e
`rtk vitest run`. Se não houver filtro dedicado, o RTK encaminhará o comando.

## Segurança operacional

- Nunca expor ou versionar segredos, tokens, peppers, chaves ou arquivos `.env`.
- Não executar comandos destrutivos sem escopo exato e autorização explícita.
- Autorização e autenticação devem ser verificadas no servidor; ocultar elementos na interface não
  constitui controle de acesso.
- Toda exceção à constituição ou às regras deste arquivo deve ser documentada e aprovada.
