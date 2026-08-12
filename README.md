# Desenvolvimento de Software com IA na Prática

Este repositório é o material de apoio de um workshop sobre desenvolvimento de software assistido por
inteligência artificial. O objetivo não é apenas mostrar uma IA gerando código, mas demonstrar como
dar contexto, estabelecer limites, validar resultados e manter uma pessoa responsável pelas decisões.

O produto usado como exemplo é o **Mini Notion**, uma aplicação de documentos pessoais. Ele fornece
um problema realista para aplicar as práticas do workshop, mas os conceitos apresentados aqui podem
ser usados em outros projetos e com outros agentes de IA.

## A ideia central

Uma IA consegue explorar arquivos, propor soluções, editar código e executar ferramentas. Isso não
significa que ela conheça automaticamente o objetivo do produto, as regras da equipe ou o nível de
risco aceitável. Sem contexto e verificações, velocidade apenas permite produzir erros mais rápido.

Neste projeto, a IA trabalha dentro de um processo explícito:

1. O comportamento esperado é especificado antes da implementação.
2. Decisões técnicas são pesquisadas e registradas.
3. O trabalho é dividido em tarefas pequenas e verificáveis.
4. Testes orientam a implementação nos domínios críticos.
5. Ferramentas limitam acesso, reduzem ruído e preservam contexto.
6. Uma pessoa revisa mudanças e autoriza operações sensíveis.

A IA é tratada como uma colaboradora capaz de executar trabalho, não como substituta de engenharia,
revisão ou responsabilidade.

## Práticas adotadas

### Spec-Driven Development (SDD)

Spec-Driven Development coloca a especificação no centro do desenvolvimento. Antes de perguntar
"como implementar?", o processo esclarece "o que precisa acontecer?", "para quem?" e "como saberemos
que funcionou?".

Isso reduz um problema comum no uso de IA: receber uma implementação tecnicamente plausível para um
requisito ambíguo. Neste repositório, cada funcionalidade possui uma pasta em [`specs/`](specs/README.md)
e progride por artefatos com responsabilidades diferentes:

- `spec.md` descreve necessidades, cenários e critérios de sucesso, sem escolher a implementação;
- `plan.md` registra pesquisa, arquitetura e decisões técnicas da feature;
- `tasks.md` transforma o plano em unidades de trabalho ordenadas e testáveis;
- checklists e análises aplicam os critérios obrigatórios de qualidade, chamados de *quality gates*,
  antes de escrever código.

SDD não elimina mudanças. Ele torna explícito o motivo de uma mudança e permite atualizar primeiro a
fonte de verdade correta, em vez de deixar a intenção escondida no código ou em uma conversa com a IA.

### Spec Kit

O [GitHub Spec Kit](https://github.com/github/spec-kit) fornece os comandos e templates usados para
aplicar SDD. O fluxo adotado neste projeto é:

```text
/speckit.constitution
        ↓
/speckit.specify → /speckit.clarify → /speckit.plan → /speckit.tasks
                                                        ↓
                              /speckit.analyze → /speckit.implement
                                                        ↓
                                                /speckit.converge
```

- `/speckit.constitution` estabelece princípios permanentes e quality gates do projeto;
- `/speckit.specify` cria ou atualiza o comportamento esperado de uma feature;
- `/speckit.clarify` resolve ambiguidades que poderiam gerar decisões incorretas;
- `/speckit.plan` pesquisa dependências e define o desenho técnico;
- `/speckit.tasks` produz tarefas ordenadas e verificáveis;
- `/speckit.analyze` compara spec, plano e tarefas antes da implementação;
- `/speckit.implement` executa as tarefas aprovadas;
- `/speckit.converge` identifica lacunas restantes depois de uma rodada de trabalho.

Nem todo projeto precisa de todos os artefatos para qualquer alteração pequena. Aqui, o fluxo completo
é intencional porque faz parte do exercício e porque autenticação, autorização e dados privados têm
alto risco. Os comandos instalados para o OpenCode podem ser vistos em [`.opencode/commands/`](.opencode/commands/).

### Test-Driven Development (TDD)

TDD usa um ciclo curto chamado **Red-Green-Refactor**:

1. **Red:** escrever um teste que descreve o próximo comportamento e confirmar que ele falha pelo
   motivo esperado;
2. **Green:** implementar somente o necessário para o teste passar;
3. **Refactor:** melhorar a estrutura sem alterar o comportamento já protegido.

Com IA, esse ciclo é especialmente útil: o teste fornece um objetivo executável e reduz a chance de a
implementação parecer correta apenas por inspeção. Neste projeto, TDD é obrigatório nos domínios
críticos definidos pela [constituição](.specify/memory/constitution.md), e outros módulos recebem
testes proporcionais ao risco.

Cobertura alta não é prova de qualidade por si só. As asserções ainda precisam representar
comportamentos, falhas, limites e regras de segurança relevantes.

### Engenharia de contexto

Um prompt isolado é uma fonte frágil de contexto: pode ser esquecido, interpretado de maneira
diferente ou desaparecer ao iniciar uma nova sessão. Por isso, as decisões importantes deste projeto
ficam versionadas e distribuídas por responsabilidade:

- [`AGENTS.md`](AGENTS.md) ensina aos agentes como operar no repositório;
- [a constituição](.specify/memory/constitution.md) define princípios permanentes;
- [`docs/`](docs/) concentra a visão e as decisões globais do produto;
- [`specs/`](specs/README.md) define o comportamento de cada feature;
- planos e tarefas registram como uma feature aprovada será implementada.

Essa organização ajuda a IA a recuperar contexto confiável e também permite que pessoas revisem as
mesmas regras. Instruções para agentes fazem parte do projeto e devem receber o mesmo cuidado que
outras configurações versionadas.

### Supervisão humana

Automação não deve significar autorização irrestrita. As regras em [`AGENTS.md`](AGENTS.md) exigem
aprovação explícita antes de criar, editar, mover ou excluir arquivos. O workflow de commits também
separa análise, staging, revisão do diff e autorização final de cada commit.

Esses pontos de controle são úteis porque permitem interromper uma ação antes que ela altere o estado
do projeto. A pessoa continua responsável por validar requisitos, aceitar trade-offs e decidir o que
entra no histórico. Quanto maior o impacto ou a dificuldade de reversão, mais explícita deve ser a
aprovação.

## Ferramentas de apoio

### OpenCode

[OpenCode](https://opencode.ai/) é o agente de desenvolvimento usado neste workshop. Ele lê as
instruções do repositório, consulta arquivos, executa ferramentas e propõe ou realiza alterações. As
práticas deste projeto não dependem exclusivamente dele: SDD, TDD, isolamento e revisão humana
continuam válidos ao trocar o agente.

### AI Jail

[AI Jail](https://github.com/akitaonrails/ai-jail) executa agentes dentro de um sandbox baseado nos
mecanismos do sistema operacional. A configuração [`.ai-jail`](.ai-jail) deste projeto mantém o
workspace gravável, mas bloqueia caminhos comuns de segredos e desabilita recursos desnecessários,
como Docker, GPU, display e SSH.

O sandbox reduz o alcance de erros e comandos indesejados, mas não torna a execução 100% segura. Ele
depende das garantias do sistema operacional, não equivale a uma máquina virtual e não substitui
revisão, backups, controle de versão ou proteção adequada de credenciais. Segurança é aplicada em
camadas.

### Context7

[Context7](https://context7.com/) fornece documentação atual de bibliotecas, frameworks, SDKs e
ferramentas para o agente. Neste projeto, decisões sobre dependências devem consultar essa fonte antes
de adotar ou modificar uma tecnologia.

Isso reduz respostas baseadas apenas no conhecimento de treinamento do modelo, que pode estar
desatualizado. A consulta à documentação também não elimina análise: compatibilidade, segurança e
adequação ao problema ainda precisam ser avaliadas.

### RTK

[RTK](https://github.com/rtk-ai/rtk) funciona como um proxy para comandos de terminal. O agente usa
prefixos como `rtk git status` ou `rtk vitest`, e o RTK remove barras de progresso, repetições e outras
saídas pouco úteis antes de enviá-las ao modelo.

Menos ruído preserva a janela de contexto para informações relevantes e reduz o consumo de tokens.
Como qualquer filtro, ele pode ocultar detalhes; quando necessário, o agente deve consultar a saída
completa ou usar uma ferramenta mais específica. A configuração local fica em [`.rtk/`](.rtk/).

### Git e commits focados

Mudanças pequenas e coerentes são mais fáceis de revisar, testar e reverter. Este projeto usa
[Conventional Commits](https://www.conventionalcommits.org/) e não mistura alterações sem relação no
mesmo commit. Antes de cada commit, o fluxo adotado:

1. inspeciona alterações preparadas para o próximo commit (*staged*) e ainda não preparadas
   (*unstaged*);
2. separa grupos por propósito;
3. sinaliza segredos, arquivos temporários ou artefatos suspeitos;
4. faz staging somente do grupo autorizado;
5. revisa novamente o diff staged;
6. solicita aprovação imediatamente antes do commit.

O histórico Git torna-se parte da explicação do projeto, não apenas um registro de arquivos alterados.

## Como as práticas se conectam

As ferramentas resolvem problemas diferentes e se complementam:

```text
Intenção do produto
        ↓
Spec Kit + SDD          tornam requisitos e decisões explícitos
        ↓
TDD + quality gates     transformam expectativas em verificações executáveis
        ↓
OpenCode                explora e executa o trabalho orientado pelo contexto
        ↓
AI Jail                 limita os recursos acessíveis durante a execução
        ↓
Context7 + RTK          melhoram a qualidade e a eficiência das informações
        ↓
Revisão humana + Git    controlam aceitação, rastreabilidade e reversão
```

Nenhuma ferramenta isolada garante um bom resultado. O valor está no processo completo: contexto
antes da execução, feedback durante o desenvolvimento e revisão antes da integração.

## Estrutura do repositório

```text
.
├── .ai-jail                 # Política do sandbox usado pelo agente
├── .opencode/commands/      # Comandos do Spec Kit e workflows do OpenCode
├── .rtk/                    # Filtros locais de saída do terminal
├── .specify/                # Templates, scripts e constituição do Spec Kit
├── docs/                    # Visão e decisões globais do produto
├── specs/                   # Especificações e planejamento por feature
├── AGENTS.md                # Regras operacionais para agentes de IA
└── README.md                # Guia do workshop (este documento)
```

Durante a implementação do Mini Notion, a estrutura planejada também incluirá:

```text
apps/                        # Aplicações executáveis, como web e API
packages/                    # Contratos e configurações reutilizáveis
```

Pastas não devem ser criadas apenas para reproduzir um padrão. Cada diretório precisa representar uma
responsabilidade concreta do projeto.

## Documentação do exemplo

- [Visão do produto](docs/product-overview.md): objetivo, escopo e roadmap do Mini Notion;
- [Arquitetura](docs/architecture.md): stack aprovada e organização planejada do monorepo;
- [Decisões técnicas](docs/technical-decisions.md): persistência, segurança, sessão e qualidade;
- [Especificações](specs/README.md): comportamento e estado de planejamento de cada feature;
- [Constituição](.specify/memory/constitution.md): princípios permanentes e quality gates.

O README explica **como aprender e trabalhar com IA neste projeto**. Os documentos acima explicam
**o produto que está sendo construído** e funcionam como fontes de verdade para pessoas e agentes.
