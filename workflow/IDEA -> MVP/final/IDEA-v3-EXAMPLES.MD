# IDEA-v3 — Exemplos Concretos de Artefatos

> Este documento contém exemplos completos de cada artefato definido no IDEA-v3.MD.
> Use como referência para garantir consistência e completude na geração de documentação.

---

## Exemplo de Projeto: TaskFlow

**Contexto:** Sistema de gerenciamento de tarefas para equipes pequenas.
- **Perfil:** INTERNO (2-50 usuários, rede corporativa)
- **Stack:** Node.js + TypeScript + PostgreSQL + React

---

## 1. Exemplo: `.cursor/rules/project-context.md`

```markdown
---
description: Contexto geral do projeto TaskFlow
alwaysApply: true
---

# Contexto do Projeto — TaskFlow

## Resumo
Sistema de gerenciamento de tarefas para equipes pequenas (2-50 usuários). 
Perfil: **INTERNO** — uso corporativo, sem exposição à internet pública.

## Stack Aprovada

| Camada | Tecnologia | Versão |
|--------|------------|--------|
| Runtime | Node.js | 20+ LTS |
| Linguagem | TypeScript | 5.3+ |
| Backend | Fastify | 4.x |
| ORM | Prisma | 5.x |
| Validação | Zod | 3.x |
| Database | PostgreSQL | 15+ |
| Frontend | Next.js | 14+ (App Router) |
| UI | Tailwind + shadcn/ui | 3.x |

## Estrutura de Diretórios

```
taskflow/
├── src/
│   ├── api/           # Rotas Fastify
│   ├── services/      # Lógica de negócio
│   ├── models/        # Schemas Zod
│   ├── db/            # Prisma
│   └── lib/           # Utilitários
├── web/               # Frontend Next.js
├── prisma/            # Schema e migrations
└── tests/             # Testes
```

## ADRs Vigentes
- ADR-001: Fastify ao invés de Express (performance)
- ADR-002: Prisma ao invés de TypeORM (DX)
- ADR-003: Zustand ao invés de Redux (simplicidade)
```

---

## 2. Exemplo: `PRD.md`

```markdown
# PRD — TaskFlow

> **Resumo executivo:** Sistema de gerenciamento de tarefas para equipes de 2-50 pessoas. 
> Permite criar projetos, atribuir tarefas, acompanhar progresso e gerar relatórios simples. 
> Perfil INTERNO, sem exposição pública, autenticação via SSO corporativo.

## 1. Visão Geral

TaskFlow é uma ferramenta interna de gerenciamento de tarefas que permite equipes 
organizarem seu trabalho em projetos e tarefas. O sistema oferece visão kanban, 
atribuição de responsáveis, datas de entrega e relatórios de progresso.

## 2. Classificação do Projeto

- **Perfil:** INTERNO
- **Justificativa:**
  - Usuários: 2-50 (equipe corporativa)
  - Exposição: Rede interna apenas
  - PII: Apenas de colaboradores (nome, email corporativo)
  - SLA: Não formal, mas disponibilidade em horário comercial
  - Impacto de falha: Operacional (atraso em entregas)

## 3. Objetivos e Não-Objetivos

### Objetivos (MVP)
- Permitir criação de projetos e tarefas
- Permitir atribuição de tarefas a membros da equipe
- Visualização kanban de tarefas por status
- Notificações por email em atribuições
- Relatório simples de tarefas por período

### Não-Objetivos (fora do escopo)
- Integração com ferramentas externas (Slack, Jira)
- App mobile nativo
- Funcionalidades de chat/comentários em tempo real
- Gestão de horas/timesheet

## 4. Personas/Papéis

| Persona | Descrição | Permissões |
|---------|-----------|------------|
| Admin | Administrador do sistema | CRUD projetos, CRUD usuários, todas as tarefas |
| Manager | Gestor de projetos | CRUD tarefas em seus projetos, visualizar relatórios |
| Member | Membro da equipe | CRUD próprias tarefas, visualizar projetos atribuídos |

## 5. User Stories

### FEAT-001: Criar Projeto

**Como** Admin ou Manager, **quero** criar um novo projeto, **para** organizar as tarefas da equipe.

#### Critérios de Aceite (Gherkin)

```gherkin
# Happy path 1: Criação com dados mínimos
Given um usuário autenticado com role "admin" ou "manager"
When ele envia POST /api/v1/projects com { "name": "Projeto Alpha", "description": "Descrição" }
Then o sistema retorna 201 Created
And o response contém o projeto criado com id, name, description, createdAt
And o usuário é automaticamente adicionado como owner do projeto

# Happy path 2: Criação com membros iniciais
Given um usuário autenticado com role "admin"
When ele envia POST /api/v1/projects com { "name": "Projeto Beta", "memberIds": ["uuid1", "uuid2"] }
Then o sistema retorna 201 Created
And os usuários uuid1 e uuid2 são adicionados como members do projeto

# Unhappy path 1: Nome duplicado
Given um projeto com nome "Projeto Alpha" já existe
When um usuário tenta criar outro projeto com nome "Projeto Alpha"
Then o sistema retorna 409 Conflict
And o response contém { "code": "PROJECT.NAME_TAKEN", "severity": "WARNING" }

# Unhappy path 2: Nome vazio
Given um usuário autenticado
When ele envia POST /api/v1/projects com { "name": "", "description": "Desc" }
Then o sistema retorna 400 Bad Request
And o response contém { "code": "VALIDATION.INVALID_INPUT", "details": [{ "field": "name" }] }

# Unhappy path 3: Sem permissão
Given um usuário autenticado com role "member"
When ele tenta criar um projeto
Then o sistema retorna 403 Forbidden
And o response contém { "code": "AUTH.FORBIDDEN" }
```

#### Regras de Negócio
- RULE-001: Nome do projeto DEVE ter entre 3 e 100 caracteres
- RULE-002: Nome do projeto DEVE ser único no sistema
- RULE-003: Criador do projeto é automaticamente owner

#### Testes Associados
- TEST-001: Criação de projeto com dados válidos (201)
- TEST-002: Criação de projeto com membros iniciais (201)
- TEST-003: Erro ao criar projeto com nome duplicado (409)
- TEST-004: Erro ao criar projeto com nome vazio (400)
- TEST-005: Erro ao criar projeto sem permissão (403)

---

### FEAT-002: Criar Tarefa

**Como** membro do projeto, **quero** criar uma tarefa, **para** registrar trabalho a ser feito.

#### Critérios de Aceite (Gherkin)

```gherkin
# Happy path 1: Tarefa simples
Given um usuário membro do projeto "proj-123"
When ele envia POST /api/v1/projects/proj-123/tasks com { "title": "Implementar login", "status": "TODO" }
Then o sistema retorna 201 Created
And a tarefa é criada com status "TODO" e sem assignee

# Happy path 2: Tarefa com assignee e deadline
Given um usuário membro do projeto "proj-123"
And existe um usuário "user-456" membro do mesmo projeto
When ele envia POST com { "title": "Review PR", "assigneeId": "user-456", "deadline": "2024-02-01" }
Then o sistema retorna 201 Created
And um email de notificação é enviado para "user-456"

# Unhappy path 1: Assignee não é membro do projeto
Given um usuário membro do projeto "proj-123"
And existe um usuário "user-789" que NÃO é membro do projeto
When ele tenta criar tarefa com assigneeId "user-789"
Then o sistema retorna 422 Unprocessable Entity
And o response contém { "code": "TASK.ASSIGNEE_NOT_MEMBER" }

# Unhappy path 2: Deadline no passado
Given um usuário membro do projeto
When ele tenta criar tarefa com deadline "2020-01-01"
Then o sistema retorna 400 Bad Request
And o response contém { "code": "VALIDATION.INVALID_INPUT", "details": [{ "field": "deadline", "code": "DATE_IN_PAST" }] }

# Unhappy path 3: Projeto não existe
Given um usuário autenticado
When ele tenta criar tarefa em projeto "inexistente-123"
Then o sistema retorna 404 Not Found
And o response contém { "code": "PROJECT.NOT_FOUND" }
```

#### Regras de Negócio
- RULE-004: Título da tarefa DEVE ter entre 5 e 200 caracteres
- RULE-005: Status inicial DEVE ser um de: TODO, IN_PROGRESS, REVIEW, DONE
- RULE-006: Assignee DEVE ser membro do projeto
- RULE-007: Deadline, se informado, DEVE ser data futura

#### Testes Associados
- TEST-006: Criação de tarefa simples (201)
- TEST-007: Criação de tarefa com assignee (201 + email)
- TEST-008: Erro assignee não membro (422)
- TEST-009: Erro deadline no passado (400)
- TEST-010: Erro projeto não encontrado (404)

---

### FEAT-003: Visualizar Kanban

**Como** membro do projeto, **quero** ver as tarefas em formato kanban, **para** acompanhar o progresso.

#### Critérios de Aceite (Gherkin)

```gherkin
# Happy path 1: Visualização padrão
Given um usuário membro do projeto "proj-123"
And o projeto tem 10 tarefas em diferentes status
When ele acessa GET /api/v1/projects/proj-123/tasks?view=kanban
Then o sistema retorna 200 OK
And as tarefas são agrupadas por status (TODO, IN_PROGRESS, REVIEW, DONE)
And cada grupo contém as tarefas ordenadas por deadline (mais urgente primeiro)

# Happy path 2: Filtro por assignee
Given um usuário membro do projeto
When ele acessa GET /api/v1/projects/proj-123/tasks?view=kanban&assigneeId=user-456
Then apenas tarefas atribuídas a "user-456" são retornadas

# Unhappy path 1: Projeto não encontrado
Given um usuário autenticado
When ele acessa GET /api/v1/projects/inexistente/tasks?view=kanban
Then o sistema retorna 404 Not Found

# Unhappy path 2: Usuário não é membro
Given um usuário autenticado que NÃO é membro do projeto "proj-123"
When ele tenta acessar o kanban do projeto
Then o sistema retorna 403 Forbidden
And o response contém { "code": "AUTH.FORBIDDEN" }
```

#### Regras de Negócio
- RULE-008: Usuário só pode ver kanban de projetos onde é membro
- RULE-009: Tarefas são ordenadas por deadline dentro de cada coluna

#### Testes Associados
- TEST-011: Visualização kanban com tarefas (200)
- TEST-012: Filtro por assignee (200)
- TEST-013: Erro projeto não encontrado (404)
- TEST-014: Erro usuário não membro (403)

## 6. Grafo de Dependências

```mermaid
graph LR
    FEAT-001[FEAT-001: Criar Projeto] --> FEAT-002[FEAT-002: Criar Tarefa]
    FEAT-001 --> FEAT-003[FEAT-003: Kanban]
    FEAT-002 --> FEAT-003
    FEAT-002 --> FEAT-004[FEAT-004: Relatórios]
    FEAT-003 --> FEAT-004
```

**Ordem de implementação:** FEAT-001 → FEAT-002 → FEAT-003 → FEAT-004

## 7. Regras de Negócio Consolidadas

| ID | Regra | Features | Erro se violada |
|----|-------|----------|-----------------|
| RULE-001 | Nome projeto: 3-100 chars | FEAT-001 | VALIDATION.INVALID_INPUT |
| RULE-002 | Nome projeto único | FEAT-001 | PROJECT.NAME_TAKEN |
| RULE-003 | Criador é owner | FEAT-001 | - |
| RULE-004 | Título tarefa: 5-200 chars | FEAT-002 | VALIDATION.INVALID_INPUT |
| RULE-005 | Status válido | FEAT-002 | VALIDATION.INVALID_INPUT |
| RULE-006 | Assignee é membro | FEAT-002 | TASK.ASSIGNEE_NOT_MEMBER |
| RULE-007 | Deadline futura | FEAT-002 | VALIDATION.INVALID_INPUT |
| RULE-008 | Acesso só para membros | FEAT-003 | AUTH.FORBIDDEN |
| RULE-009 | Ordenação por deadline | FEAT-003 | - |

## 8. Mapa de Erros

| Code | Severity | Quando ocorre | HTTP Status | i18nKey |
|------|----------|---------------|-------------|---------|
| AUTH.UNAUTHORIZED | ERROR | Token ausente/inválido | 401 | errors.auth.unauthorized |
| AUTH.FORBIDDEN | WARNING | Sem permissão para ação | 403 | errors.auth.forbidden |
| VALIDATION.INVALID_INPUT | WARNING | Dados de entrada inválidos | 400 | errors.validation.invalid_input |
| PROJECT.NOT_FOUND | WARNING | Projeto não existe | 404 | errors.project.not_found |
| PROJECT.NAME_TAKEN | WARNING | Nome de projeto duplicado | 409 | errors.project.name_taken |
| TASK.NOT_FOUND | WARNING | Tarefa não existe | 404 | errors.task.not_found |
| TASK.ASSIGNEE_NOT_MEMBER | WARNING | Assignee não é membro | 422 | errors.task.assignee_not_member |
| INTERNAL.SERVER_ERROR | CRITICAL | Erro interno | 500 | errors.internal.server_error |

## 9. Métricas de Sucesso

| Métrica | Baseline | Target | Como medir |
|---------|----------|--------|------------|
| Tarefas criadas/semana | 0 | 50+ | Query no banco |
| Tempo médio de conclusão | N/A | < 5 dias | Analytics |
| Adoção (usuários ativos) | 0 | 80% do time | Login tracking |
| NPS | N/A | > 7 | Survey mensal |
```

---

## 3. Exemplo: `TECH_SPECS.md`

```markdown
# Tech Specs — TaskFlow

> **Resumo executivo:** API REST em Node.js/Fastify com PostgreSQL. 
> Autenticação via JWT integrado com SSO corporativo. 
> Frontend em Next.js com shadcn/ui. Deploy em containers Docker.

## 1. Arquitetura

### Diagrama

```mermaid
graph TB
    subgraph Cliente
        WEB[Next.js Web App]
    end
    
    subgraph Backend
        API[Fastify API]
        AUTH[Auth Service]
        NOTIFY[Notification Service]
    end
    
    subgraph Data
        PG[(PostgreSQL)]
        REDIS[(Redis Cache)]
    end
    
    subgraph External
        SSO[Corporate SSO]
        SMTP[Email SMTP]
    end
    
    WEB --> API
    API --> AUTH
    API --> NOTIFY
    AUTH --> SSO
    AUTH --> REDIS
    NOTIFY --> SMTP
    API --> PG
```

### Componentes

| Componente | Responsabilidade | Tecnologia |
|------------|------------------|------------|
| Web App | Interface do usuário | Next.js 14 + shadcn/ui |
| API | Lógica de negócio, endpoints REST | Fastify 4 + TypeScript |
| Auth Service | Autenticação, tokens | JWT + Redis |
| Notification Service | Emails de notificação | Nodemailer |
| PostgreSQL | Persistência principal | PostgreSQL 15 |
| Redis | Cache de sessões | Redis 7 |

## 2. Modelo de Dados

### Entidade: User

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "email": { 
      "type": "string", 
      "format": "email", 
      "maxLength": 255,
      "x-pii": true,
      "x-normalize": ["trim", "lowercase"]
    },
    "name": {
      "type": "string",
      "minLength": 2,
      "maxLength": 100,
      "x-pii": true,
      "x-normalize": ["trim"]
    },
    "role": {
      "type": "string",
      "enum": ["admin", "manager", "member"]
    },
    "status": {
      "type": "string",
      "enum": ["active", "inactive"]
    },
    "createdAt": { "type": "string", "format": "date-time" },
    "updatedAt": { "type": "string", "format": "date-time" }
  },
  "required": ["id", "email", "name", "role", "status"]
}
```

### Entidade: Project

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "name": {
      "type": "string",
      "minLength": 3,
      "maxLength": 100,
      "x-normalize": ["trim"]
    },
    "description": {
      "type": "string",
      "maxLength": 1000
    },
    "ownerId": { "type": "string", "format": "uuid" },
    "status": {
      "type": "string",
      "enum": ["active", "archived"]
    },
    "createdAt": { "type": "string", "format": "date-time" },
    "updatedAt": { "type": "string", "format": "date-time" }
  },
  "required": ["id", "name", "ownerId", "status"]
}
```

### Entidade: Task

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "projectId": { "type": "string", "format": "uuid" },
    "title": {
      "type": "string",
      "minLength": 5,
      "maxLength": 200,
      "x-normalize": ["trim"]
    },
    "description": {
      "type": "string",
      "maxLength": 5000
    },
    "status": {
      "type": "string",
      "enum": ["TODO", "IN_PROGRESS", "REVIEW", "DONE"]
    },
    "assigneeId": { 
      "type": "string", 
      "format": "uuid",
      "nullable": true
    },
    "deadline": {
      "type": "string",
      "format": "date",
      "nullable": true
    },
    "createdById": { "type": "string", "format": "uuid" },
    "createdAt": { "type": "string", "format": "date-time" },
    "updatedAt": { "type": "string", "format": "date-time" }
  },
  "required": ["id", "projectId", "title", "status", "createdById"]
}
```

### Diagrama ER

```mermaid
erDiagram
    User ||--o{ Project : owns
    User ||--o{ Task : creates
    User ||--o{ Task : assigned_to
    User }|--o{ ProjectMember : member_of
    Project ||--|{ ProjectMember : has_members
    Project ||--o{ Task : contains
    
    User {
        uuid id PK
        string email UK
        string name
        enum role
        enum status
        datetime createdAt
        datetime updatedAt
    }
    
    Project {
        uuid id PK
        string name UK
        text description
        uuid ownerId FK
        enum status
        datetime createdAt
        datetime updatedAt
    }
    
    ProjectMember {
        uuid projectId PK,FK
        uuid userId PK,FK
        enum role
        datetime joinedAt
    }
    
    Task {
        uuid id PK
        uuid projectId FK
        string title
        text description
        enum status
        uuid assigneeId FK
        date deadline
        uuid createdById FK
        datetime createdAt
        datetime updatedAt
    }
```

## 3. Contrato de API

### API-001: Criar Projeto

- **FEAT:** FEAT-001
- **Método:** POST
- **Path:** `/api/v1/projects`
- **Auth:** Bearer JWT (roles: admin, manager)

#### Request

```json
{
  "name": "Projeto Alpha",
  "description": "Descrição opcional do projeto",
  "memberIds": ["uuid-1", "uuid-2"]
}
```

#### Validações

| Campo | Tipo | Obrigatório | Limites | Normalização | Erro |
|-------|------|-------------|---------|--------------|------|
| name | string | sim | 3-100 chars | trim | VALIDATION.INVALID_INPUT |
| description | string | não | max 1000 | trim | VALIDATION.INVALID_INPUT |
| memberIds | uuid[] | não | max 50 items | - | VALIDATION.INVALID_INPUT |

#### Response (201 Created)

```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Projeto Alpha",
    "description": "Descrição opcional do projeto",
    "ownerId": "owner-uuid",
    "status": "active",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z",
    "members": [
      { "userId": "uuid-1", "role": "member" },
      { "userId": "uuid-2", "role": "member" }
    ]
  }
}
```

#### Response (409 Conflict)

```json
{
  "code": "PROJECT.NAME_TAKEN",
  "message": "Já existe um projeto com este nome",
  "severity": "WARNING",
  "traceId": "trace-uuid"
}
```

---

### API-002: Criar Tarefa

- **FEAT:** FEAT-002
- **Método:** POST
- **Path:** `/api/v1/projects/:projectId/tasks`
- **Auth:** Bearer JWT (membro do projeto)

#### Request

```json
{
  "title": "Implementar autenticação",
  "description": "Integrar com SSO corporativo",
  "status": "TODO",
  "assigneeId": "user-uuid",
  "deadline": "2024-02-01"
}
```

#### Validações

| Campo | Tipo | Obrigatório | Limites | Normalização | Erro |
|-------|------|-------------|---------|--------------|------|
| title | string | sim | 5-200 chars | trim | VALIDATION.INVALID_INPUT |
| description | string | não | max 5000 | trim | VALIDATION.INVALID_INPUT |
| status | enum | não | TODO,IN_PROGRESS,REVIEW,DONE | - | VALIDATION.INVALID_INPUT |
| assigneeId | uuid | não | membro do projeto | - | TASK.ASSIGNEE_NOT_MEMBER |
| deadline | date | não | data futura | - | VALIDATION.INVALID_INPUT |

#### Response (201 Created)

```json
{
  "data": {
    "id": "task-uuid",
    "projectId": "project-uuid",
    "title": "Implementar autenticação",
    "description": "Integrar com SSO corporativo",
    "status": "TODO",
    "assigneeId": "user-uuid",
    "deadline": "2024-02-01",
    "createdById": "creator-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

#### Response (422 Unprocessable Entity)

```json
{
  "code": "TASK.ASSIGNEE_NOT_MEMBER",
  "message": "O usuário atribuído não é membro do projeto",
  "severity": "WARNING",
  "traceId": "trace-uuid"
}
```

---

### API-003: Listar Tarefas (Kanban)

- **FEAT:** FEAT-003
- **Método:** GET
- **Path:** `/api/v1/projects/:projectId/tasks`
- **Auth:** Bearer JWT (membro do projeto)

#### Query Parameters

| Param | Tipo | Default | Descrição |
|-------|------|---------|-----------|
| view | string | list | "kanban" agrupa por status |
| assigneeId | uuid | - | Filtrar por assignee |
| status | enum | - | Filtrar por status |
| page | number | 1 | Página (se view=list) |
| perPage | number | 20 | Items por página |

#### Response (200 OK - view=kanban)

```json
{
  "data": {
    "TODO": [
      {
        "id": "task-1",
        "title": "Tarefa 1",
        "assigneeId": "user-1",
        "deadline": "2024-02-01"
      }
    ],
    "IN_PROGRESS": [
      {
        "id": "task-2",
        "title": "Tarefa 2",
        "assigneeId": "user-2",
        "deadline": "2024-01-30"
      }
    ],
    "REVIEW": [],
    "DONE": []
  },
  "meta": {
    "total": 2,
    "byStatus": {
      "TODO": 1,
      "IN_PROGRESS": 1,
      "REVIEW": 0,
      "DONE": 0
    }
  }
}
```

## 4. Integrações Externas

| Integração | Tipo | Timeout | Retries | Idempotência | Fallback |
|------------|------|---------|---------|--------------|----------|
| Corporate SSO | OAuth2 | 5s | 2 | N/A | Cache de token válido |
| SMTP (Email) | SMTP | 10s | 3 | Via messageId | Queue para retry |

## 5. Requisitos Não-Funcionais

| Requisito | Valor | Perfil | Como validar |
|-----------|-------|--------|--------------|
| Latência p95 | < 300ms | INTERNO | Load test |
| Disponibilidade | 99% (horário comercial) | INTERNO | Monitoring |
| Usuários simultâneos | 50 | INTERNO | Load test |
| Backup | Diário | INTERNO | Cron job |

## 6. Estratégia de Testes

### Cobertura por Feature

| FEAT | TEST IDs | Tipo | Cenários |
|------|----------|------|----------|
| FEAT-001 | TEST-001 a TEST-005 | unit, integration | criar projeto válido/inválido/duplicado |
| FEAT-002 | TEST-006 a TEST-010 | unit, integration | criar tarefa válida/inválida/sem permissão |
| FEAT-003 | TEST-011 a TEST-014 | integration | kanban view, filtros, permissões |

### Exemplo: TEST-001

```gherkin
# TEST-001: Criação de projeto com dados válidos
Given um usuário "admin@corp.com" autenticado com role "admin"
And não existe projeto com nome "Projeto Teste"
When POST /api/v1/projects com body { "name": "Projeto Teste" }
Then status code é 201
And response.data.name é "Projeto Teste"
And response.data.ownerId é o id do usuário admin
And o projeto existe no banco de dados
```

## 7. Matriz de Rastreabilidade Completa

| FEAT | RULE | API | TEST | ADR |
|------|------|-----|------|-----|
| FEAT-001 | RULE-001, RULE-002, RULE-003 | API-001 | TEST-001 a TEST-005 | ADR-001, ADR-002 |
| FEAT-002 | RULE-004, RULE-005, RULE-006, RULE-007 | API-002 | TEST-006 a TEST-010 | - |
| FEAT-003 | RULE-008, RULE-009 | API-003 | TEST-011 a TEST-014 | - |

## 8. Mapa de Erros Consolidado

| Code | Severity | HTTP | Quando | Exemplo Response |
|------|----------|------|--------|------------------|
| AUTH.UNAUTHORIZED | ERROR | 401 | Token ausente/expirado | `{"code":"AUTH.UNAUTHORIZED","message":"Token inválido","severity":"ERROR","traceId":"..."}` |
| AUTH.FORBIDDEN | WARNING | 403 | Sem permissão | `{"code":"AUTH.FORBIDDEN","message":"Acesso negado","severity":"WARNING","traceId":"..."}` |
| VALIDATION.INVALID_INPUT | WARNING | 400 | Dados inválidos | `{"code":"VALIDATION.INVALID_INPUT","message":"Dados inválidos","severity":"WARNING","details":[...],"traceId":"..."}` |
| PROJECT.NOT_FOUND | WARNING | 404 | Projeto não existe | `{"code":"PROJECT.NOT_FOUND","message":"Projeto não encontrado","severity":"WARNING","traceId":"..."}` |
| PROJECT.NAME_TAKEN | WARNING | 409 | Nome duplicado | `{"code":"PROJECT.NAME_TAKEN","message":"Nome já em uso","severity":"WARNING","traceId":"..."}` |
| TASK.ASSIGNEE_NOT_MEMBER | WARNING | 422 | Assignee inválido | `{"code":"TASK.ASSIGNEE_NOT_MEMBER","message":"Usuário não é membro","severity":"WARNING","traceId":"..."}` |
| INTERNAL.SERVER_ERROR | CRITICAL | 500 | Erro interno | `{"code":"INTERNAL.SERVER_ERROR","message":"Erro interno","severity":"CRITICAL","traceId":"..."}` |
```

---

## 4. Exemplo: `SECURITY_IMPLEMENTATION.md`

```markdown
# Security Implementation — TaskFlow

> **Resumo executivo:** Sistema interno com autenticação via SSO corporativo. 
> PII limitada a dados de colaboradores (nome, email). 
> Foco em autorização (RBAC) e logging para auditoria.

## 1. Threat Model

### Ativos

| Ativo | Classificação | Impacto se comprometido |
|-------|---------------|-------------------------|
| Credenciais de usuário | Sensível | Acesso não autorizado |
| Dados de projetos/tarefas | Interno | Vazamento de informação |
| Email/nome de colaboradores | PII | Compliance LGPD |
| Tokens JWT | Sensível | Impersonation |

### Atacantes

| Tipo | Motivação | Capacidade |
|------|-----------|------------|
| Colaborador malicioso | Acesso a dados de outros | Acesso interno |
| Ex-colaborador | Vingança, espionagem | Credenciais antigas |
| Atacante externo | Dados corporativos | Limitado (rede interna) |

### Vetores de Ataque

| Vetor | Ativo alvo | Mitigação | FEAT relacionada |
|-------|------------|-----------|------------------|
| Escalação de privilégios | Dados | RBAC rigoroso | FEAT-001, FEAT-002 |
| Token roubado | Sessão | Expiração curta, refresh rotation | Auth |
| Injeção SQL | Banco | Prisma ORM | FEAT-001, FEAT-002 |
| IDOR | Dados | Ownership check | FEAT-002, FEAT-003 |

## 2. Classificação de Dados

| Dado | Classificação | Retenção | Minimização | Criptografia |
|------|---------------|----------|-------------|--------------|
| email | PII | Enquanto ativo + 90d | Não | TLS em trânsito |
| nome | PII | Enquanto ativo + 90d | Não | TLS em trânsito |
| senha | N/A (SSO) | - | - | - |
| projetos | Interno | Indefinido | - | - |
| tarefas | Interno | Indefinido | - | - |
| audit_logs | Interno | 2 anos | - | - |

## 3. AuthN (Autenticação)

### Fluxo

```mermaid
sequenceDiagram
    participant U as Usuário
    participant A as TaskFlow API
    participant S as Corporate SSO
    participant R as Redis
    
    U->>A: GET /auth/login
    A->>S: Redirect to SSO
    S->>U: Login page
    U->>S: Credentials
    S->>A: Callback with code
    A->>S: Exchange code for token
    S->>A: SSO token + user info
    A->>A: Generate JWT
    A->>R: Store refresh token
    A->>U: JWT + Refresh token
```

### Configurações

| Parâmetro | Valor | Justificativa |
|-----------|-------|---------------|
| JWT expiry | 1h | Balance segurança/UX interno |
| Refresh expiry | 8h | Dia de trabalho |
| JWT algorithm | RS256 | Assimétrico, validação sem secret |
| Refresh rotation | Sim | Mitiga token roubado |

## 4. AuthZ (Autorização)

### Matriz de Permissões (RBAC)

| Role | Criar projeto | Ver todos projetos | Gerenciar membros | Criar tarefa | Ver tarefas | Deletar tarefa |
|------|---------------|--------------------|--------------------|--------------|-------------|----------------|
| admin | ✓ | ✓ | ✓ | ✓ (todos) | ✓ (todos) | ✓ (todos) |
| manager | ✓ | Só seus | ✓ (seus projetos) | ✓ (seus) | ✓ (seus) | ✓ (seus) |
| member | - | Só membros | - | ✓ (membros) | ✓ (membros) | Só próprias |

### Ownership Check

```typescript
// Toda operação em recurso DEVE verificar ownership
async function checkTaskAccess(userId: string, taskId: string) {
  const task = await prisma.task.findUnique({
    where: { id: taskId },
    include: { project: { include: { members: true } } },
  });
  
  if (!task) throw new NotFoundError('TASK.NOT_FOUND');
  
  const isMember = task.project.members.some(m => m.userId === userId);
  if (!isMember) throw new ForbiddenError('AUTH.FORBIDDEN');
  
  return task;
}
```

## 5. OWASP Top 10 — Aplicação Contextual

| # | Vulnerabilidade | Aplica? | Mitigação | Onde |
|---|-----------------|---------|-----------|------|
| A01 | Broken Access Control | ✓ | RBAC + ownership check em toda rota | Middleware |
| A02 | Cryptographic Failures | ✓ | TLS 1.3, JWT RS256 | Infra + Auth |
| A03 | Injection | ✓ | Prisma ORM (prepared statements) | DB layer |
| A04 | Insecure Design | ✓ | Threat model, security review | Design phase |
| A05 | Security Misconfiguration | ✓ | Helmet, CORS restrito, headers | Config |
| A06 | Vulnerable Components | ✓ | npm audit, Dependabot | CI/CD |
| A07 | Auth Failures | ✓ | SSO corporativo, JWT curto | Auth |
| A08 | Data Integrity Failures | ✓ | Validação Zod em todo input | API layer |
| A09 | Logging Failures | ✓ | Structured logging, audit trail | Logging |
| A10 | SSRF | Baixo risco | Sem chamadas externas arbitrárias | - |

## 6. Rate Limiting

| Endpoint | Limite | Janela | Ação se exceder |
|----------|--------|--------|-----------------|
| /auth/* | 10 req | 1min | 429 + block 5min |
| /api/* (autenticado) | 200 req | 1min | 429 + Retry-After |

## 7. Logging Seguro

### O que logar

| Evento | Dados logados | Dados NUNCA logados |
|--------|---------------|---------------------|
| Login/Logout | userId, timestamp, IP | tokens, senhas |
| CRUD projeto | userId, projectId, action | - |
| CRUD tarefa | userId, taskId, action | - |
| Erro authZ | userId, resource, attemptedAction | - |

### Formato

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "traceId": "uuid",
  "level": "INFO",
  "event": "task.created",
  "userId": "user-uuid",
  "projectId": "project-uuid",
  "taskId": "task-uuid",
  "ip": "10.0.0.50",
  "userAgent": "Mozilla/5.0..."
}
```

## 8. Gestão de Segredos

| Segredo | Onde armazenar | Rotação | Acesso |
|---------|----------------|---------|--------|
| JWT private key | Vault | 30 dias | Auth service |
| Database URL | Environment | 90 dias | API |
| SSO client secret | Vault | 90 dias | Auth service |

## 9. Resposta a Incidentes

### Classificação

| Severidade | Critério | Tempo resposta | Quem acionar |
|------------|----------|----------------|--------------|
| P1 | Vazamento de dados, acesso não autorizado | 30min | TI + Gestão |
| P2 | Sistema indisponível | 1h | TI |
| P3 | Bug de segurança sem exploração | 4h | Dev |

### Runbook

1. **Detectar:** Alerta de monitoring ou report de usuário
2. **Conter:** Revogar tokens suspeitos, bloquear usuário se necessário
3. **Investigar:** Analisar logs, identificar escopo
4. **Comunicar:** Informar gestão se P1/P2
5. **Corrigir:** Deploy de fix
6. **Documentar:** Post-mortem em 48h
```

---

## 5. Exemplo: `INFRASTRUCTURE.md`

```markdown
# Infrastructure — TaskFlow

> **Resumo executivo:** Deploy em containers Docker. 
> Ambiente de desenvolvimento local via Docker Compose. 
> Staging/Produção em servidor corporativo com Docker.

## 1. Ambientes

| Ambiente | URL | Propósito | Quem acessa |
|----------|-----|-----------|-------------|
| local | localhost:3000 | Desenvolvimento | Devs |
| staging | taskflow-staging.corp.local | Testes | Time |
| production | taskflow.corp.local | Uso real | Colaboradores |

## 2. Variáveis de Ambiente

| Variável | Obrigatória | Exemplo | Onde obter |
|----------|-------------|---------|------------|
| DATABASE_URL | Sim | postgres://user:pass@host:5432/db | Vault |
| REDIS_URL | Sim | redis://host:6379 | Config |
| JWT_PRIVATE_KEY | Sim | [gerado] | Vault |
| JWT_PUBLIC_KEY | Sim | [gerado] | Vault |
| SSO_CLIENT_ID | Sim | taskflow | SSO Admin |
| SSO_CLIENT_SECRET | Sim | [secret] | Vault |
| SSO_ISSUER_URL | Sim | https://sso.corp.local | SSO Admin |
| NODE_ENV | Sim | production | Deploy config |
| LOG_LEVEL | Não | info | Deploy config |

## 3. Dependências de Infraestrutura

| Serviço | Versão | Propósito | Fallback |
|---------|--------|-----------|----------|
| PostgreSQL | 15+ | Banco principal | - |
| Redis | 7+ | Cache/Sessions | Graceful (sessões em DB) |
| Corporate SSO | - | Autenticação | - (crítico) |
| SMTP Relay | - | Notificações | Queue para retry |

## 4. CI/CD Pipeline

```mermaid
flowchart LR
    A[Push] --> B[Lint + Test]
    B --> C{Pass?}
    C -->|No| D[Notify + Stop]
    C -->|Yes| E[Build Docker]
    E --> F{Branch?}
    F -->|main| G[Deploy Staging]
    G --> H[Smoke Tests]
    H --> I{Pass?}
    I -->|No| D
    I -->|Yes| J[Manual Approval]
    J --> K[Deploy Production]
    F -->|feature| L[End]
```

### Stages

| Stage | Trigger | Ações | Tempo |
|-------|---------|-------|-------|
| Lint + Test | Push | ESLint, TypeScript, Vitest | ~3min |
| Build | Test pass | docker build | ~5min |
| Deploy Staging | main branch | docker-compose up | ~2min |
| Smoke Tests | Auto | Healthcheck + basic API test | ~1min |
| Deploy Prod | Manual | docker-compose up | ~2min |

## 5. Deploy

### Docker Compose (Staging/Production)

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  api:
    image: taskflow-api:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=taskflow
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

### Comandos

```bash
# Build
docker build -t taskflow-api:latest .

# Deploy staging
cd /opt/taskflow/staging
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Deploy production
cd /opt/taskflow/production
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Ver logs
docker-compose logs -f api

# Rollback
docker-compose -f docker-compose.prod.yml up -d --no-deps api
# (usando imagem anterior)
```

### Checklist Pré-Deploy

- [ ] Testes passando em CI
- [ ] Migrations executadas em staging
- [ ] Smoke tests passando em staging
- [ ] Variáveis de ambiente verificadas
- [ ] Backup do banco realizado (se breaking change)
- [ ] Rollback plan documentado
- [ ] Horário de baixo uso (se possível)

## 6. Rollback

### Procedimento

1. Identificar versão anterior: `docker images taskflow-api`
2. Atualizar tag em docker-compose ou fazer pull da versão anterior
3. `docker-compose up -d --no-deps api`
4. Verificar healthcheck: `curl http://localhost:3000/health`
5. Se migrations: executar rollback script

### Tempo máximo: 5 minutos

## 7. Monitoramento e Alertas

| Métrica | Threshold | Alerta | Ação |
|---------|-----------|--------|------|
| API Health | Down > 1min | Email TI | Verificar containers |
| Error rate | > 5% | Email TI | Investigar logs |
| Response time p95 | > 1s | Log | Investigar |
| Disk usage | > 80% | Email TI | Limpar/expandir |
| Memory usage | > 90% | Email TI | Restart/investigar |

### Healthcheck Endpoint

```
GET /health
Response: { "status": "ok", "db": "ok", "redis": "ok", "version": "1.0.0" }
```

## 8. Backup e Recovery

| Dado | Frequência | Retenção | RTO | RPO |
|------|------------|----------|-----|-----|
| PostgreSQL | Diário 2h | 30 dias | 2h | 24h |
| Redis | - | - | - | 0 (cache) |
| Uploads (se houver) | Diário | 30 dias | 2h | 24h |

### Comandos de Backup

```bash
# Backup manual
docker exec taskflow_db pg_dump -U postgres taskflow > backup_$(date +%Y%m%d).sql

# Restore
docker exec -i taskflow_db psql -U postgres taskflow < backup_20240115.sql
```

### Cron (servidor)

```cron
# Backup diário às 2h
0 2 * * * /opt/taskflow/scripts/backup.sh >> /var/log/taskflow-backup.log 2>&1
```
```

---

## 6. Exemplo: `ADR.md`

```markdown
# ADR — TaskFlow

## ADR-001: Fastify ao invés de Express

**Status:** Aceito  
**Data:** 2024-01-10  
**Decisores:** Equipe de Desenvolvimento

### Contexto
Precisamos de um framework web para a API REST. O time tem experiência com Express, mas estamos avaliando alternativas mais modernas.

### Decisão
Adotar **Fastify 4.x** como framework web.

### Alternativas Consideradas

| Opção | Prós | Contras |
|-------|------|---------|
| Express 4 | Familiar, ecossistema | Performance inferior, sem types nativos |
| Fastify 4 | Performance, TypeScript, schema validation | Curva de aprendizado |
| NestJS | Estruturado, DI | Overhead, complexidade |

### Consequências

**Ganhos:**
- Performance 2-3x melhor que Express
- Suporte nativo a TypeScript e JSON Schema
- Plugins oficiais (cors, helmet, swagger)

**Perdas:**
- Time precisa aprender nova API
- Alguns middlewares Express não são compatíveis

---

## ADR-002: Prisma como ORM

**Status:** Aceito  
**Data:** 2024-01-10  
**Decisores:** Equipe de Desenvolvimento

### Contexto
Precisamos de uma forma segura e produtiva de interagir com PostgreSQL.

### Decisão
Adotar **Prisma 5.x** como ORM.

### Alternativas Consideradas

| Opção | Prós | Contras |
|-------|------|---------|
| Prisma | DX excelente, migrations, tipos | Overhead em queries complexas |
| TypeORM | Familiar (Active Record/Data Mapper) | Tipos menos precisos, bugs |
| Knex | Flexível, queries brutas | Mais código manual |

### Consequências

**Ganhos:**
- Tipos gerados automaticamente do schema
- Migrations versionadas
- Prepared statements automáticos (segurança)

**Perdas:**
- Queries muito complexas podem precisar de $queryRaw

---

## ADR-003: Zustand para state management (Frontend)

**Status:** Aceito  
**Data:** 2024-01-10  
**Decisores:** Equipe de Desenvolvimento

### Contexto
Precisamos gerenciar estado global no frontend (auth, preferências).

### Decisão
Adotar **Zustand 4.x** para state management.

### Alternativas Consideradas

| Opção | Prós | Contras |
|-------|------|---------|
| Zustand | Simples, pequeno, hooks | Menos ferramentas de debug |
| Redux Toolkit | Padrão da indústria, DevTools | Boilerplate, complexidade |
| Jotai | Atômico, simples | Menos familiar |

### Consequências

**Ganhos:**
- API simples baseada em hooks
- Bundle pequeno (~1KB)
- Fácil de testar

**Perdas:**
- Menos ferramentas de debug que Redux
- Pode precisar de middleware para casos complexos

---

## ADR-004: Autenticação via SSO Corporativo

**Status:** Aceito  
**Data:** 2024-01-12  
**Decisores:** Equipe + TI

### Contexto
Usuários são colaboradores da empresa. Já existe SSO corporativo.

### Decisão
Integrar com **SSO Corporativo** via OAuth2/OIDC.

### Alternativas Consideradas

| Opção | Prós | Contras |
|-------|------|---------|
| SSO Corporativo | Uma senha, provisioning automático | Dependência externa |
| Auth próprio | Controle total | Manutenção de senhas, mais código |

### Consequências

**Ganhos:**
- Usuários não precisam de nova senha
- Provisioning/deprovisioning automático
- MFA já configurado no SSO

**Perdas:**
- Dependência do SSO (se cair, app não autentica)
- Configuração inicial mais complexa

**Mitigação:**
- Cache de tokens válidos por 1h para degradação graceful
```

---

## Resumo da Estrutura Completa

```
projeto/
├── .cursor/
│   └── rules/
│       ├── project-context.md     # alwaysApply: true
│       ├── security.md            # alwaysApply: true
│       ├── api-conventions.mdc    # globs: src/api/**
│       ├── database.mdc           # globs: src/db/**
│       ├── testing.mdc            # globs: **/*.test.*
│       └── frontend.mdc           # globs: src/app/**
├── docs/
│   ├── PRD.md
│   ├── TECH_SPECS.md
│   ├── SECURITY_IMPLEMENTATION.md
│   ├── INFRASTRUCTURE.md
│   └── ADR.md
└── ... (código fonte)
```
