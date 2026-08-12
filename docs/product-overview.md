# Visão do Produto

## Mini Notion

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

## Roadmap documental

1. Foundation do monorepo e quality gates.
2. Cadastro, login, sessão e autorização.
3. Gerenciamento e pesquisa de documentos.
4. Editor Rich Text e navegação pelo conteúdo.

Cada etapa deve ter spec aprovada, clarificação quando necessária, plano, tarefas e análise de
consistência antes da implementação. O estado de cada etapa está no [índice de specs](../specs/README.md).

## Decisões futuras

Os seguintes temas permanecem fora das specs iniciais ou precisam ser definidos no plano adequado:

- autosave, indicador de estado e resolução de falhas ao salvar;
- paginação e limites de documentos e conteúdo;
- exclusão lógica, lixeira e política de retenção;
- recuperação de senha e confirmação de e-mail;
- exportação para HTML ou Markdown;
- estratégia de observabilidade e implantação;
- ferramenta de teste end-to-end.

## Documentos relacionados

- [Arquitetura](architecture.md)
- [Decisões técnicas](technical-decisions.md)
- [Especificações das features](../specs/README.md)
