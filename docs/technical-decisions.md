# Decisões Técnicas

Este documento reúne decisões globais que afetam mais de uma feature do Mini Notion. Requisitos de
comportamento permanecem nas [specs](../specs/README.md), e detalhes de implementação devem ser
confirmados nos planos correspondentes.

## Persistência

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

## Autenticação e sessão

- A API é a autoridade de autenticação e autorização.
- O navegador receberá uma sessão opaca em cookie `HttpOnly`, `Secure` em produção e com política
  `SameSite` compatível com a implantação.
- O MVP não implementará manualmente um par `accessToken`/`refreshToken` no navegador.
- A sessão permanecerá válida por até sete dias e poderá ser renovada por atividade com intervalo de
  atualização de 24 horas; não haverá refresh a cada minuto.
- Web e API devem ser publicados sob a mesma origem ou por proxy sempre que possível.
- Todas as operações privadas validarão sessão e propriedade do recurso no servidor.

## Cadastro e login

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

Os quality gates permanentes e as regras para dependências estão definidos na
[constituição](../.specify/memory/constitution.md).

## Documentos relacionados

- [Visão do produto](product-overview.md)
- [Arquitetura](architecture.md)
- [Especificação de autenticação](../specs/002-authentication/spec.md)
- [Especificação de documentos](../specs/003-document-management/spec.md)
- [Especificação do editor](../specs/004-rich-text-editor/spec.md)
