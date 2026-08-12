# Feature Specification: Document Management

**Feature Branch**: `003-document-management`

**Created**: 2026-08-11

**Status**: Approved

**Input**: Permitir que usuários autenticados criem, encontrem, editem e excluam seus documentos.

## Clarifications

### Session 2026-08-12

- Q: Quando duas sessões editarem o mesmo documento, o que deverá acontecer quando a segunda tentar salvar uma versão desatualizada? → A: Rejeitar o salvamento desatualizado, preservar as alterações locais e solicitar recarga.
- Q: Quais documentos deverão aparecer primeiro na lista padrão? → A: Os modificados mais recentemente.
- Q: Como um documento com título vazio ou apenas espaços deverá ser exibido e salvo? → A: Salvar e exibir como “Sem título”.
- Q: Como a pesquisa deverá comparar o termo com títulos e conteúdos? → A: Ignorar caixa e acentos e aceitar correspondências parciais.
- Q: Quais limites máximos cada documento deverá aceitar para título e conteúdo estruturado? → A: Título de 200 caracteres e conteúdo estruturado de 1 MB.

## User Scenarios & Testing

### User Story 1 - Visualizar e criar documentos (Priority: P1)

Como usuário autenticado, quero visualizar meus documentos e criar um novo para começar a registrar
minhas ideias.

**Why this priority**: Criar e reencontrar documentos é o valor central do produto.

**Independent Test**: Um usuário sem documentos vê o estado vazio, cria um documento e passa a
visualizá-lo em sua lista sem acessar dados de outra conta.

**Acceptance Scenarios**:

1. **Given** um usuário sem documentos, **When** acessar a tela principal, **Then** verá a mensagem
   “Você ainda não tem documentos, crie e comece a colocar suas ideias.” e uma ação para criar.
2. **Given** um usuário autenticado, **When** criar um documento, **Then** um documento pertencente a
   ele, com o título “Sem título”, será aberto para edição.
3. **Given** um usuário com documentos, **When** acessar a tela principal, **Then** verá somente seus
   próprios documentos, ordenados da modificação mais recente para a mais antiga.

---

### User Story 2 - Encontrar um documento (Priority: P1)

Como usuário com vários documentos, quero pesquisar por título ou palavras do conteúdo para encontrar
rapidamente o que preciso.

**Why this priority**: A lista perde utilidade quando não permite localizar conteúdo conhecido.

**Independent Test**: Uma consulta retorna apenas documentos do usuário que contenham o termo no
título ou conteúdo pesquisável e permite limpar o filtro.

**Acceptance Scenarios**:

1. **Given** documentos com títulos diferentes, **When** pesquisar parte de um título usando caixa ou
   acentuação diferentes, **Then** serão exibidos apenas os documentos correspondentes.
2. **Given** um termo presente no conteúdo, **When** pesquisar esse termo, **Then** o documento
   correspondente será exibido.
3. **Given** uma consulta sem correspondência, **When** a pesquisa terminar, **Then** um estado de
   resultado vazio será mostrado sem sugerir que o usuário não possui documentos.
4. **Given** uma pesquisa ativa, **When** o termo for limpo, **Then** a lista completa será restaurada.
5. **Given** um termo acima de 200 pontos de código após remover espaços nas extremidades, **When** a
   pesquisa for solicitada, **Then** ela será rejeitada com indicação do limite e a lista atual será
   preservada.

---

### User Story 3 - Atualizar um documento (Priority: P1)

Como proprietário, quero alterar o título e o conteúdo de um documento para manter minhas informações
atualizadas.

**Why this priority**: Um documento que não pode evoluir não cumpre a função de editor pessoal.

**Independent Test**: O proprietário abre um documento, altera seus dados e encontra a versão
atualizada em um novo acesso.

**Acceptance Scenarios**:

1. **Given** um documento próprio, **When** o usuário salvar alterações válidas, **Then** título,
   conteúdo e data de atualização refletirão a nova versão.
2. **Given** um documento de outro usuário, **When** alguém tentar acessá-lo ou alterá-lo, **Then** a
   operação será negada sem revelar seu conteúdo.
3. **Given** que outra sessão salvou uma versão mais recente, **When** o usuário tentar salvar sua
   versão desatualizada, **Then** o salvamento será rejeitado, suas alterações locais serão preservadas
   e a interface solicitará a recarga do documento.
4. **Given** um título acima de 200 pontos de código após trim ou conteúdo estruturado acima de
   1.000.000 bytes UTF-8 de JSON canônico, **When** o usuário tentar salvar, **Then** o salvamento será
   rejeitado, o limite excedido será indicado e as alterações locais serão preservadas.
5. **Given** conteúdo estruturado acima de 100 níveis ou 10.000 nós, contando a raiz como nível e nó 1,
   **When** o usuário tentar salvar, **Then** o salvamento será rejeitado, o limite excedido será
   indicado e as alterações locais serão preservadas.

---

### User Story 4 - Excluir um documento (Priority: P2)

Como proprietário, quero excluir um documento com confirmação para evitar remoções acidentais.

**Why this priority**: Exclusão é necessária para gerenciamento, mas deve proteger contra cliques
involuntários.

**Independent Test**: A exclusão só ocorre após confirmação explícita; cancelar preserva o documento.

**Acceptance Scenarios**:

1. **Given** um documento listado, **When** o usuário solicitar exclusão, **Then** um modal de
   confirmação identificará o documento e oferecerá cancelar ou confirmar.
2. **Given** o modal aberto, **When** o usuário cancelar, **Then** o documento permanecerá inalterado.
3. **Given** o modal aberto, **When** o usuário confirmar, **Then** o documento deixará de aparecer e
   não poderá mais ser acessado pelo usuário.

### Edge Cases

- Títulos vazios ou contendo apenas espaços devem ser normalizados e persistidos como “Sem título”.
- Pesquisa deve ignorar diferenças de caixa e acentuação, aceitar correspondências parciais e tratar
  espaços ou caracteres especiais sem erro.
- Um documento excluído ou inexistente não deve revelar dados pelo identificador.
- Requisições concorrentes de alteração devem rejeitar versões desatualizadas sem mesclagem ou
  sobrescrita automática.
- Falha durante criação, salvamento ou exclusão deve manter a interface em estado recuperável.
- A lista não deve misturar “nenhum documento criado” com “nenhum resultado encontrado”.
- Títulos acima de 200 pontos de código após trim ou conteúdos estruturados acima de 1.000.000 bytes
  UTF-8 de JSON canônico devem ser rejeitados sem descartar alterações locais.
- Conteúdo estruturado acima de 100 níveis ou 10.000 nós, contando a raiz como nível e nó 1, deve ser
  rejeitado sem descartar alterações locais.
- Pesquisa acima de 200 pontos de código após trim deve ser rejeitada sem substituir a lista atual.

## Requirements

### Functional Requirements

- **FR-001**: Usuários autenticados MUST visualizar somente documentos dos quais são proprietários.
- **FR-002**: Usuários sem documentos MUST ver a mensagem de estado vazio definida nesta spec.
- **FR-003**: A tela principal MUST oferecer uma ação clara para criar um documento.
- **FR-004**: Cada novo documento MUST ser associado ao usuário autenticado.
- **FR-005**: Um documento recém-criado MUST ser aberto para edição.
- **FR-006**: A lista MUST oferecer ações de editar e excluir para cada documento.
- **FR-007**: Usuários MUST poder pesquisar seus documentos por correspondência parcial no título ou
  conteúdo, sem distinção de caixa ou acentuação.
- **FR-008**: A pesquisa MUST distinguir lista vazia de ausência de correspondências.
- **FR-009**: Limpar a pesquisa MUST restaurar a lista não filtrada.
- **FR-010**: Proprietários MUST poder atualizar título e conteúdo.
- **FR-011**: Cada alteração persistida MUST atualizar a data de modificação.
- **FR-012**: O sistema MUST rejeitar o salvamento de uma versão desatualizada, preservar as alterações
  locais e solicitar que o usuário recarregue o documento; o sistema MUST NOT mesclar ou sobrescrever
  automaticamente a versão mais recente.
- **FR-013**: A exclusão MUST exigir confirmação em modal antes de produzir efeito.
- **FR-014**: Cancelar a confirmação MUST preservar o documento.
- **FR-015**: Após confirmação bem-sucedida, o documento MUST deixar de ser listado e acessível.
- **FR-016**: Toda leitura ou mutação MUST validar sessão e propriedade no servidor.
- **FR-017**: Respostas a identificadores inexistentes ou não pertencentes ao usuário MUST NOT revelar
  conteúdo ou proprietário.
- **FR-018**: Falhas de persistência MUST ser comunicadas sem indicar sucesso ou descartar
  silenciosamente mudanças locais.
- **FR-019**: A lista padrão MUST ordenar documentos pela data de modificação, da mais recente para a
  mais antiga.
- **FR-020**: Na criação ou atualização, um título vazio ou contendo apenas espaços MUST ser
  normalizado, persistido e exibido como “Sem título”.
- **FR-021**: Títulos MUST ter no máximo 200 pontos de código Unicode após trim e o conteúdo estruturado
  MUST ter no máximo 1.000.000 bytes UTF-8 de sua serialização JSON canônica; criação ou atualização
  acima desses limites MUST ser rejeitada com indicação do limite excedido e sem descartar alterações
  locais.
- **FR-022**: O conteúdo estruturado MUST ter no máximo 100 níveis e 10.000 nós, contando a raiz como
  nível e nó 1; criação ou atualização acima desses limites MUST ser rejeitada sem descartar alterações
  locais.
- **FR-023**: O termo de pesquisa MUST ter no máximo 200 pontos de código Unicode após trim; uma
  consulta acima desse limite MUST ser rejeitada com indicação do limite sem substituir a lista atual.

### Key Entities

- **Document**: Documento pessoal com proprietário, título, conteúdo, representação pesquisável,
  versão e timestamps; o título possui no máximo 200 pontos de código após trim, e o conteúdo
  estruturado possui no máximo 1.000.000 bytes UTF-8, 100 níveis e 10.000 nós.
- **Document Search**: Consulta transitória aplicada exclusivamente ao conjunto de documentos do
  usuário autenticado.

## Success Criteria

### Measurable Outcomes

- **SC-001**: 100% dos usuários sem documentos veem o estado vazio correto e conseguem iniciar a
  criação por uma única ação.
- **SC-002**: 100% dos documentos retornados em lista ou pesquisa pertencem ao usuário autenticado.
- **SC-003**: Pelo menos 95% das pesquisas em uma coleção dentro dos limites definidos no plano
  exibem resultados em até um segundo.
- **SC-004**: Título ou palavra conhecida recupera todos os documentos correspondentes nos cenários
  de aceitação.
- **SC-005**: Nenhuma exclusão ocorre sem confirmação explícita nos cenários automatizados.
- **SC-006**: 100% das tentativas de acesso cruzado testadas são negadas sem exposição de conteúdo.
- **SC-007**: 100% das tentativas testadas de persistir título acima de 200 pontos de código após trim
  ou conteúdo estruturado acima de 1.000.000 bytes UTF-8 de JSON canônico são rejeitadas sem perda das
  alterações locais.
- **SC-008**: 100% das tentativas testadas de persistir conteúdo acima de 100 níveis ou 10.000 nós são
  rejeitadas sem perda das alterações locais.
- **SC-009**: 100% das pesquisas testadas acima de 200 pontos de código após trim são rejeitadas sem
  substituir a lista atual.

## Assumptions

- O MVP possui documentos privados e individuais; compartilhamento está fora do escopo.
- Os limites de paginação serão definidos no plano técnico.
- O comportamento observável da exclusão é imediato; retenção ou exclusão física será decisão do
  plano, sem alterar os cenários desta spec.
- Autosave e experiência completa do editor pertencem à feature Rich Text.
