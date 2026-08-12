# Feature Specification: Rich Text Editor

**Feature Branch**: `004-rich-text-editor`

**Created**: 2026-08-11

**Status**: Approved

**Input**: Oferecer edição Rich Text, navegação entre documentos e sumário baseado em títulos.

## Clarifications

### Session 2026-08-12

- Q: Como as alterações do editor devem ser salvas e protegidas contra perda ao trocar de documento
  ou fechar a página? → A: Autosave após inatividade; tentar salvar ao sair e avisar antes de
  descartar se houver falha.
- Q: Quais níveis de título o editor deve oferecer e incluir no sumário? → A: H1, H2 e H3.
- Q: Como o usuário deve escolher cor e tamanho da fonte? → A: Paleta e tamanhos predefinidos.
- Q: Como a sidebar de documentos e o sumário devem funcionar em telas estreitas? → A: Ambos
  recolhidos em painéis acionáveis.
- Q: O que deve acontecer quando o autosave detectar que outra sessão já salvou uma versão mais
  recente do mesmo documento? → A: Rejeitar, preservar alterações locais e solicitar recarga.

## User Scenarios & Testing

### User Story 1 - Escrever e formatar conteúdo (Priority: P1)

Como proprietário de um documento, quero escrever e aplicar formatação Rich Text para organizar e
destacar minhas ideias.

**Why this priority**: A experiência de escrita formatada diferencia o produto de um campo de texto
simples.

**Independent Test**: O usuário escreve conteúdo, aplica cada formato suportado, persiste o documento
e encontra a mesma estrutura visual ao reabri-lo.

**Acceptance Scenarios**:

1. **Given** um documento próprio aberto, **When** o usuário inserir e editar texto, **Then** o
   conteúdo será refletido no editor e poderá ser persistido.
2. **Given** texto selecionado ou um ponto de inserção, **When** uma ação de formatação suportada for
   aplicada, **Then** o estado visual e semântico correspondente será atualizado.
3. **Given** um documento formatado e salvo, **When** for reaberto, **Then** texto, estrutura e
   formatação suportada serão restaurados.

---

### User Story 2 - Usar a barra de formatação (Priority: P1)

Como usuário escrevendo, quero acessar as ações Rich Text acima do documento e entender quais estão
ativas para formatar sem memorizar comandos.

**Why this priority**: Os recursos só são úteis quando estão visíveis, compreensíveis e operáveis.

**Independent Test**: Cada ação suportada pode ser aplicada pela toolbar e comunica seu estado,
inclusive por teclado e tecnologia assistiva.

**Acceptance Scenarios**:

1. **Given** o foco no editor, **When** a seleção mudar, **Then** a toolbar refletirá os formatos
   ativos nessa seleção.
2. **Given** uma ação incompatível com o contexto atual, **When** a toolbar for exibida, **Then** a
   ação ficará indisponível sem alterar o documento.
3. **Given** navegação por teclado, **When** o usuário percorrer e acionar a toolbar, **Then** todas as
   ações essenciais estarão disponíveis com nome acessível.

---

### User Story 3 - Navegar pelo documento (Priority: P2)

Como usuário em um documento longo, quero usar um sumário lateral para navegar diretamente aos
títulos existentes.

**Why this priority**: A navegação por estrutura torna documentos longos utilizáveis.

**Independent Test**: Títulos suportados aparecem no sumário na ordem do documento e cada item leva
ao trecho correspondente.

**Acceptance Scenarios**:

1. **Given** um documento com títulos, **When** o conteúdo for exibido, **Then** o sumário à direita
   listará os títulos na mesma ordem e hierarquia.
2. **Given** um item do sumário, **When** ele for acionado, **Then** o título correspondente ficará
   visível e identificável.
3. **Given** alteração, inclusão ou remoção de um título, **When** o conteúdo mudar, **Then** o sumário
   será atualizado sem recarregar a página.
4. **Given** um documento sem títulos, **When** o editor for exibido, **Then** o espaço do sumário não
   mostrará itens inválidos ou vazios.

---

### User Story 4 - Alternar contexto pelo editor (Priority: P2)

Como usuário autenticado, quero visualizar meus outros documentos e ações de conta em uma sidebar
para navegar sem retornar à tela principal.

**Why this priority**: Preserva o contexto de escrita e reduz passos entre documentos.

**Independent Test**: A sidebar permite abrir outro documento próprio, criar um novo e acessar a área
de conta sem expor documentos de outros usuários.

**Acceptance Scenarios**:

1. **Given** o editor aberto, **When** o usuário usar a sidebar, **Then** verá seus documentos, uma ação
   de criação e acesso às informações da conta.
2. **Given** alterações ainda não confirmadas como persistidas, **When** o usuário tentar trocar de
   documento, **Then** a interface evitará perda silenciosa.
3. **Given** a criação de um documento pela sidebar, **When** ela for concluída, **Then** o novo
   documento será aberto no editor.

### Edge Cases

- Seleções contendo formatos mistos devem comunicar um estado coerente na toolbar.
- Aplicar e remover repetidamente um formato não deve corromper a estrutura.
- Colar conteúdo externo deve preservar somente estruturas e formatos aceitos.
- Conteúdo vazio deve continuar sendo um documento válido e editável.
- Títulos repetidos devem produzir destinos de navegação distintos e estáveis durante a sessão.
- Formatação desconhecida ou conteúdo legado deve falhar de forma segura, sem executar HTML ativo.
- Falha de persistência ou troca de documento não deve descartar mudanças silenciosamente.
- Autosave de uma versão desatualizada deve ser rejeitado sem mesclagem automática, preservar as
  alterações locais e solicitar que o usuário recarregue a versão mais recente.

## Requirements

### Functional Requirements

- **FR-001**: Proprietários MUST poder inserir, selecionar, substituir e remover conteúdo textual.
- **FR-002**: O editor MUST oferecer negrito, itálico, sublinhado, cor por uma paleta predefinida e
  tamanho de fonte por um conjunto predefinido.
- **FR-003**: O editor MUST oferecer alinhamento à esquerda, centralizado, à direita e justificado.
- **FR-004**: O editor MUST oferecer listas ordenadas e não ordenadas.
- **FR-005**: O editor MUST oferecer títulos H1, H2 e H3, e somente esses níveis MUST compor o
  sumário.
- **FR-006**: A toolbar MUST permanecer acima da área de conteúdo durante a edição.
- **FR-007**: A toolbar MUST comunicar o estado ativo, inativo, misto ou indisponível das ações.
- **FR-008**: Ações essenciais MUST ser operáveis por teclado e possuir nomes acessíveis.
- **FR-009**: Conteúdo persistido MUST preservar texto, estrutura e formatos suportados ao reabrir.
- **FR-010**: O conteúdo canônico MUST permanecer estruturado e permitir representações derivadas sem
  perda da fonte original.
- **FR-011**: Conteúdo inserido externamente MUST ser limitado a estruturas e formatos aceitos; cores
  e tamanhos fora dos conjuntos predefinidos MUST ser normalizados para um valor aceito ou removidos.
- **FR-012**: O editor MUST NOT executar scripts ou conteúdo ativo proveniente do documento.
- **FR-013**: O sumário MUST ser derivado dos títulos do conteúdo e respeitar ordem e hierarquia.
- **FR-014**: Cada item do sumário MUST navegar para um título único no documento atual.
- **FR-015**: O sumário MUST acompanhar mudanças de títulos durante a sessão de edição.
- **FR-016**: A sidebar MUST listar somente documentos do usuário autenticado.
- **FR-017**: A sidebar MUST oferecer criação de documento, troca de documento e acesso à conta.
- **FR-018**: O editor MUST iniciar autosave após um período configurado sem alterações.
- **FR-019**: Ao trocar de documento ou fechar a página com alterações pendentes, o editor MUST tentar
  persistir as alterações e, se não conseguir confirmá-las, MUST exigir confirmação explícita antes
  de permitir seu descarte.
- **FR-020**: O estado de salvamento MUST distinguir alterações pendentes, persistidas e falhas.
- **FR-021**: Em telas sem largura suficiente para a apresentação lateral, a sidebar e o sumário MUST
  permanecer recolhidos em painéis acionáveis, mantendo escrita, navegação e ações essenciais
  acessíveis por teclado e tecnologia assistiva.
- **FR-022**: Ao detectar que outra sessão persistiu uma versão mais recente, o autosave MUST rejeitar
  a versão desatualizada, preservar as alterações locais e solicitar recarga, sem mesclagem automática.

### Key Entities

- **Rich Text Content**: Árvore estruturada de blocos, texto, marcas e atributos suportados.
- **Heading**: Bloco de título com nível, texto e destino único usado pelo sumário.
- **Editor State**: Conteúdo atual, seleção, formatos ativos e estado de persistência da sessão.

## Success Criteria

### Measurable Outcomes

- **SC-001**: 100% dos formatos declarados podem ser aplicados, removidos, salvos e restaurados nos
  cenários automatizados.
- **SC-002**: 100% dos títulos suportados aparecem no sumário na ordem e hierarquia corretas.
- **SC-003**: Acionar qualquer item do sumário torna o título correspondente visível em até um
  segundo.
- **SC-004**: Todas as ações essenciais da toolbar e sidebar são concluídas somente por teclado nos
  testes de acessibilidade.
- **SC-005**: Nenhum cenário de colagem, renderização ou reabertura executa conteúdo ativo armazenado.
- **SC-006**: Nenhum cenário testado de falha ou troca de contexto perde alterações sem aviso.
- **SC-007**: 100% dos cenários automatizados de conflito entre sessões rejeitam a versão
  desatualizada e preservam as alterações locais para recuperação pelo usuário.

## Assumptions

- Edição colaborativa em tempo real está fora do escopo.
- HTML e Markdown são formatos futuros de exportação, não o formato canônico.
- O intervalo de inatividade do autosave e a política de novas tentativas serão definidos no
  planejamento técnico, preservando o comportamento de proteção contra descarte estabelecido nesta
  spec.
- A largura exata para recolher os painéis laterais será definida no planejamento técnico.
