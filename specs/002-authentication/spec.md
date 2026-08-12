# Feature Specification: Authentication

**Feature Branch**: `002-authentication`

**Created**: 2026-08-11

**Status**: Draft

**Input**: Permitir cadastro e login seguros, manter sessão e impedir acesso não autorizado.

## User Scenarios & Testing

### User Story 1 - Criar uma conta (Priority: P1)

Como visitante, quero criar uma conta com e-mail, senha e confirmação para acessar meu espaço privado.

**Why this priority**: Sem uma conta válida não existe acesso às funcionalidades pessoais do produto.

**Independent Test**: Um visitante fornece dados válidos, recebe uma sessão e chega à tela principal
sem precisar entrar novamente.

**Acceptance Scenarios**:

1. **Given** um e-mail ainda não cadastrado e senhas válidas e iguais, **When** o visitante confirmar
   o cadastro, **Then** a conta será criada, uma sessão será iniciada e a tela principal será exibida.
2. **Given** um e-mail inválido, senhas diferentes ou senha fora da política, **When** o cadastro for
   enviado, **Then** nenhuma conta será criada e os campos inválidos serão indicados.
3. **Given** um e-mail já utilizado, **When** um novo cadastro for solicitado, **Then** nenhuma conta
   duplicada será criada e uma resposta segura será apresentada.

---

### User Story 2 - Entrar na conta (Priority: P1)

Como usuário cadastrado, quero entrar com e-mail e senha para acessar meus documentos.

**Why this priority**: É o caminho recorrente de acesso ao produto.

**Independent Test**: Um usuário ativo entra com credenciais válidas e passa a acessar recursos
privados; dados inválidos não iniciam sessão.

**Acceptance Scenarios**:

1. **Given** uma conta ativa, **When** forem fornecidas credenciais válidas, **Then** uma sessão será
   iniciada, tentativas anteriores serão zeradas e a tela principal será exibida.
2. **Given** credenciais inválidas, **When** o login for solicitado, **Then** nenhuma sessão será
   iniciada e a mensagem não identificará qual credencial falhou.
3. **Given** uma conta inativa, **When** o login for solicitado, **Then** nenhuma sessão será iniciada.

---

### User Story 3 - Conter tentativas repetidas (Priority: P1)

Como proprietário de uma conta, quero que tentativas repetidas sejam limitadas para reduzir ataques
de força bruta.

**Why this priority**: A proteção de credenciais é requisito de segurança, não melhoria opcional.

**Independent Test**: Cinco falhas consecutivas acionam bloqueio de 15 minutos; durante o período,
novas tentativas não autenticam, mesmo com a senha correta.

**Acceptance Scenarios**:

1. **Given** quatro falhas consecutivas, **When** ocorrer a quinta falha, **Then** a conta será
   bloqueada por 15 minutos.
2. **Given** uma conta bloqueada, **When** houver nova tentativa antes do prazo, **Then** o acesso será
   negado com mensagem genérica de indisponibilidade temporária.
3. **Given** que o prazo terminou, **When** credenciais válidas forem fornecidas, **Then** o login será
   permitido e o contador será zerado.

---

### User Story 4 - Manter e encerrar a sessão (Priority: P2)

Como usuário autenticado, quero permanecer conectado durante meu uso e poder sair com segurança.

**Why this priority**: Sessões previsíveis evitam logins excessivos e acesso residual indesejado.

**Independent Test**: A sessão permanece utilizável por atividade dentro do limite de sete dias, é
invalidada ao sair e deixa de autorizar acesso após expirar.

**Acceptance Scenarios**:

1. **Given** uma sessão válida, **When** o usuário acessar um recurso privado, **Then** a operação será
   autorizada sem expor credenciais ao cliente.
2. **Given** uma sessão expirada ou inválida, **When** um recurso privado for solicitado, **Then** a
   operação será negada e a interface direcionará o usuário ao login.
3. **Given** uma sessão válida, **When** o usuário sair, **Then** a sessão será invalidada e não poderá
   ser reutilizada.

### Edge Cases

- E-mails com diferenças de caixa ou espaços não devem criar identidades duplicadas.
- Requisições simultâneas de cadastro para o mesmo e-mail devem produzir no máximo uma conta.
- Tentativas simultâneas não devem ultrapassar o limite sem acionar o bloqueio.
- Sessão de usuário desativado deve deixar de autorizar novas operações.
- Cookies ausentes, alterados, expirados ou revogados devem ser tratados como sessão inválida.
- Mensagens e tempos de resposta não devem facilitar descoberta de contas existentes.

## Requirements

### Functional Requirements

- **FR-001**: Visitantes MUST poder se cadastrar com e-mail, senha e confirmação de senha.
- **FR-002**: O sistema MUST normalizar e validar o e-mail antes do cadastro ou login.
- **FR-003**: O sistema MUST validar igualdade das senhas e uma política de senha definida no plano.
- **FR-004**: A confirmação de senha MUST NOT ser persistida.
- **FR-005**: Cada e-mail normalizado MUST identificar no máximo uma conta.
- **FR-006**: Um cadastro válido MUST iniciar a sessão e direcionar à tela principal.
- **FR-007**: Usuários ativos MUST poder entrar com e-mail e senha válidos.
- **FR-008**: Falhas de login MUST usar mensagens que não revelem se e-mail ou senha estão incorretos.
- **FR-009**: O sistema MUST contabilizar falhas consecutivas por conta sem expor o contador.
- **FR-010**: A quinta falha consecutiva MUST bloquear a conta por 15 minutos.
- **FR-011**: Um login válido após o bloqueio expirar MUST zerar tentativas e remover o bloqueio.
- **FR-012**: A rota de login MUST aplicar limitação adicional de requisições por origem.
- **FR-013**: Sessões MUST permanecer válidas por no máximo sete dias e renovar-se somente por uso
  ativo conforme a política global.
- **FR-014**: Credenciais e identificadores de sessão MUST NOT ficar acessíveis ao código executado no
  navegador.
- **FR-015**: Logout MUST invalidar a sessão utilizada.
- **FR-016**: Páginas privadas MUST redirecionar visitantes para o login.
- **FR-017**: Operações privadas MUST validar sessão e estado ativo da conta no servidor.
- **FR-018**: O sistema MUST negar recursos pertencentes a outro usuário, mesmo com sessão válida.
- **FR-019**: Senhas MUST ser armazenadas somente como hash resistente, com segredo operacional
  separado dos dados persistidos.
- **FR-020**: Eventos de bloqueio, login e logout MUST ser registráveis sem incluir senhas, tokens ou
  outros segredos.

### Key Entities

- **User**: Identidade do usuário, com e-mail normalizado, estado ativo, tentativas, bloqueio e
  timestamps.
- **Credential Account**: Credencial de senha associada ao usuário, sem armazenar texto puro.
- **Session**: Sessão revogável associada ao usuário, com criação, expiração e metadados de segurança.

## Success Criteria

### Measurable Outcomes

- **SC-001**: 100% dos cadastros válidos iniciam uma sessão e chegam à tela principal em uma única
  jornada.
- **SC-002**: 100% dos cadastros inválidos ou duplicados deixam o banco sem contas duplicadas.
- **SC-003**: Nenhum cenário de erro de login informa se o e-mail ou a senha isoladamente está errado.
- **SC-004**: Cinco falhas consecutivas bloqueiam a conta por exatamente 15 minutos em todos os
  cenários automatizados.
- **SC-005**: 100% das operações privadas testadas rejeitam sessões ausentes, expiradas, revogadas ou
  pertencentes a usuário sem acesso ao recurso.
- **SC-006**: Uma sessão encerrada deixa de autorizar novas operações imediatamente.

## Assumptions

- O MVP possui um único tipo de usuário autenticado.
- Confirmação de e-mail, recuperação de senha, login social e autenticação multifator estão fora do
  escopo desta feature.
- O produto será servido de forma que o navegador possa usar sessão segura sem armazenar tokens.
- Política detalhada de senha, cookies, auditoria e implantação será fechada no plano técnico.
