# Mini Notion — Visão Geral do Projeto

## Visão

O Mini Notion é uma aplicação web simplificada para criação e edição de documentos pessoais. Pessoas
autenticadas poderão criar uma conta, entrar, pesquisar e gerenciar seus próprios documentos, além de
editar conteúdo Rich Text.

O produto não pretende reproduzir integralmente o Notion. O MVP prioriza autenticação segura,
isolamento dos dados por usuário e uma experiência direta de escrita e organização.

## Escopo do MVP

- Cadastro e login com e-mail e senha.
- Sessão persistente e proteção de páginas e operações privadas.
- Bloqueio temporário após tentativas repetidas de login.
- Criação, listagem, pesquisa, edição e exclusão de documentos.
- Editor Rich Text com negrito, itálico, sublinhado, cores, tamanho de fonte, alinhamento e listas.
- Sidebar para documentos e conta.
- Sumário navegável derivado dos títulos do documento.
- Temas claro e escuro.

## Fora do escopo inicial

- Cópia completa dos recursos do Notion.
- Colaboração simultânea, compartilhamento público e permissões entre usuários.
- Aplicações móveis nativas.
- Login social, autenticação multifator e confirmação de e-mail.
- Recuperação de senha, histórico de versões e exportação, até que recebam specs próprias.

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

A estrutura detalhada será definida no plano de cada feature. Novas pastas ou camadas devem existir
por responsabilidade concreta, não apenas para reproduzir um padrão.

## Decisões de persistência

### Autenticação

O Better Auth será executado na API NestJS e será responsável pelos dados de usuários, contas,
credenciais, sessões e verificações. Seu adaptador oficial usará o driver nativo do MongoDB. O
Mongoose continuará restrito aos modelos do domínio, evitando uma integração não oficial entre
Better Auth e Mongoose.

A senha não será criptografada: será armazenada como hash Argon2id. A implementação deverá:

- gerar salt único por senha por meio da biblioteca Argon2;
- obter o pepper de um segredo de ambiente, nunca do banco ou repositório;
- documentar estratégia de rotação e recuperação antes de habilitar pepper em produção;
- nunca persistir senha ou confirmação em texto puro.

### Documentos

Documentos ficarão em coleção própria e serão relacionados ao proprietário por `userId`. Não serão
embutidos como strings no registro do usuário.

O conteúdo canônico será o JSON estruturado do Tiptap/ProseMirror. HTML e Markdown poderão ser
gerados para renderização ou exportação, mas não serão a fonte de verdade. A pesquisa poderá usar um
texto simples derivado e indexável, definido no plano da feature. O sumário será derivado dos headings
presentes no JSON.

## Decisões de autenticação e sessão

- A API é a autoridade de autenticação e autorização.
- O navegador receberá uma sessão opaca em cookie `HttpOnly`, `Secure` em produção e com política
  `SameSite` compatível com a implantação.
- O MVP não implementará manualmente um par `accessToken`/`refreshToken` no navegador.
- A sessão permanecerá válida por até sete dias e poderá ser renovada por atividade com intervalo de
  atualização de 24 horas; não haverá refresh a cada minuto.
- Web e API devem ser publicados sob a mesma origem ou por proxy sempre que possível.
- Todas as operações privadas validarão sessão e propriedade do recurso no servidor.

## Regras de cadastro e login

- E-mail será normalizado, validado por schema compartilhado e protegido por unicidade no banco.
- A confirmação de senha existirá apenas no formulário e na validação de entrada.
- O usuário iniciará uma sessão após cadastro válido.
- Erros de credenciais serão genéricos para reduzir enumeração de contas.
- Após cinco tentativas inválidas, a conta será bloqueada por 15 minutos.
- `loginAttempts` será zerado após autenticação válida e `lockedUntil` representará o bloqueio.
- O bloqueio por conta será combinado com rate limiting da rota e origem da requisição.

## Estratégia de qualidade

- Autenticação, autorização, documentos e schemas compartilhados exigem 100% de cobertura em
  statements, branches, functions e lines.
- Não existe percentual mínimo global. Outras áreas recebem testes proporcionais ao risco.
- Cada feature começa por testes e segue Red-Green-Refactor.
- Contratos e integrações exigem testes de integração; jornadas essenciais exigem testes end-to-end.
- Cobertura numérica não substitui testes de comportamento, falhas, limites e segurança.

## Roadmap documental

1. Foundation do monorepo e quality gates.
2. Cadastro, login, sessão e autorização.
3. Gerenciamento e pesquisa de documentos.
4. Editor Rich Text e navegação pelo conteúdo.

Cada etapa deve ter spec aprovada, clarificação quando necessária, plano, tarefas e análise de
consistência antes da implementação.

## Decisões futuras

Os seguintes temas permanecem fora das specs iniciais ou precisam ser definidos no plano adequado:

- autosave, indicador de estado e resolução de falhas ao salvar;
- paginação e limites de documentos e conteúdo;
- exclusão lógica, lixeira e política de retenção;
- recuperação de senha e confirmação de e-mail;
- exportação para HTML ou Markdown;
- estratégia de observabilidade e implantação;
- ferramenta de teste end-to-end.
