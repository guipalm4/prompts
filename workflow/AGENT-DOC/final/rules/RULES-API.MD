# AGENT-DOC — Regras de API

> **Quando usar:** FASE 2-3 ao definir contratos de API.
> **Carregue com:** `@rules/RULES-API.MD`
> **Escopo:** Filtros, paginação, versionamento, concorrência, deleção.

---

## 2.8 Filtros de Data em APIs (obrigatório para endpoints com filtros temporais)

Ao definir filtros de data em APIs, DEVE especificar:

### Campos obrigatórios por filtro

| Atributo | Descrição | Exemplo |
|----------|-----------|---------|
| **Campo alvo** | Qual campo do banco o filtro aplica | `created_at`, `updated_at`, `due_date` |
| **Formato** | ISO 8601 com timezone | `2024-01-15T00:00:00Z` |
| **Operadores** | Quais comparações suportadas | `gte`, `lte`, `between`, `eq` |
| **Default** | Comportamento sem filtro | "Últimos 30 dias" ou "Todos" |
| **Timezone** | Como tratar timezone | "UTC no backend, conversão no front" |

### Template obrigatório

```markdown
### Filtro de data: `[nome_do_filtro]`

| Atributo | Valor |
|----------|-------|
| Campo alvo | `orders.created_at` |
| Parâmetros | `?date_from=ISO8601&date_to=ISO8601` |
| Operadores | `gte` (from), `lte` (to) |
| Default | Últimos 30 dias se omitido |
| Timezone | UTC (converter no cliente) |
| Validação | `date_from` ≤ `date_to`, max range: 1 ano |
```

**Anti-pattern:** "Filtrar pedidos por data" sem especificar se é `created_at`, `updated_at` ou `shipped_at`.

---

## 2.11 Paginação Padronizada (obrigatório para endpoints que listam recursos)

Todo endpoint que retorna **listas de recursos** DEVE seguir um padrão de paginação.

### Escolha da estratégia

| Estratégia | Quando usar | Prós | Contras |
|------------|-------------|------|---------|
| **Offset-based** | Listas pequenas (<10k items), UI com "página 1, 2, 3" | Simples, permite pular páginas | Performance ruim em offsets altos |
| **Cursor-based** | Listas grandes, feeds infinitos, dados mutáveis | Performance constante, consistente | Não permite pular páginas |

**Regra:** Se não souber qual usar, **prefira cursor-based** para PRODUCAO.

### Template obrigatório

#### Request (query params)

| Estratégia | Parâmetros | Exemplo |
|------------|------------|---------|
| Offset | `page`, `perPage` | `?page=2&perPage=20` |
| Cursor | `cursor`, `limit` | `?cursor=abc123&limit=20` |

**Limites obrigatórios:**

| Parâmetro | Mínimo | Máximo | Default |
|-----------|--------|--------|---------|
| `perPage` / `limit` | 1 | 100 | 20 |
| `page` | 1 | - | 1 |

#### Response (formato padrão)

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 2,
    "perPage": 20,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true,
    "nextCursor": "eyJpZCI6MTIzfQ==",
    "prevCursor": "eyJpZCI6MTAwfQ=="
  }
}
```

| Campo | Offset | Cursor | Descrição |
|-------|--------|--------|-----------|
| `total` | ✓ | Opcional* | Total de items |
| `page` | ✓ | - | Página atual |
| `perPage` | ✓ | - | Items por página |
| `totalPages` | ✓ | - | Total de páginas |
| `hasNext` | ✓ | ✓ | Existe próxima página? |
| `hasPrev` | ✓ | ✓ | Existe página anterior? |
| `nextCursor` | - | ✓ | Cursor para próxima página |
| `prevCursor` | - | ✓ | Cursor para página anterior |

*Para cursor-based, `total` pode ser omitido se count for custoso.

### Ordenação

| Parâmetro | Formato | Exemplo |
|-----------|---------|---------|
| `sortBy` | Campo único | `?sortBy=createdAt` |
| `sortOrder` | `asc` ou `desc` | `?sortOrder=desc` |
| `sort` | Composto (alternativa) | `?sort=-createdAt,+name` |

**Default obrigatório:** Sempre especificar ordenação default (ex: `createdAt desc`).

**Anti-pattern:** Endpoint de listagem sem limite máximo de items por página.

---

## 2.12 Versionamento de API (obrigatório para perfil INTERNO/PRODUCAO)

### Estratégia de versionamento

| Método | Formato | Quando usar |
|--------|---------|-------------|
| **URL Path** (recomendado) | `/api/v1/resource` | APIs públicas, clareza |
| Header | `Accept: application/vnd.api+json;version=1` | APIs internas, flexibilidade |
| Query param | `?version=1` | Evitar (dificulta cache) |

**Regra:** Para novos projetos, usar **URL Path**.

### Ciclo de vida de versões

```
v1 (current) ──────────────────────────────────►
                   v2 (beta) ─────► v2 (current)
                                        v1 (deprecated) ──► v1 (sunset)
```

| Estado | Descrição | Headers |
|--------|-----------|---------|
| **current** | Versão principal, suportada | - |
| **beta** | Em teste, pode mudar | `X-API-Beta: true` |
| **deprecated** | Funciona mas será removida | `Deprecation: true`, `Sunset: <date>` |
| **sunset** | Removida | 410 Gone |

### Template obrigatório

```markdown
## Versionamento de API

### Versão atual
- **Versão:** v1
- **Base URL:** `/api/v1/`
- **Status:** current

### Política de deprecação
- **Aviso mínimo:** [30 dias | 90 dias | 6 meses] antes do sunset
- **Comunicação:** [Changelog | Email | Dashboard]
- **Headers:** `Deprecation: true`, `Sunset: <ISO8601 date>`

### Breaking changes (o que incrementa versão major)
- Remover campo obrigatório do response
- Adicionar campo obrigatório no request
- Mudar tipo de campo existente
- Remover endpoint
- Mudar comportamento de erro

### Non-breaking changes (não incrementa versão)
- Adicionar campo opcional no request
- Adicionar campo no response
- Adicionar novo endpoint
- Adicionar novo valor em enum (com cuidado)
```

### Erros de versão

| Situação | Status | Code |
|----------|--------|------|
| Versão não existe | 404 | `API.VERSION_NOT_FOUND` |
| Versão deprecated | 200 + headers | - |
| Versão sunset | 410 | `API.VERSION_SUNSET` |

**Anti-pattern:** Breaking change sem incrementar versão major.

---

## 2.13 Concorrência e Locking (obrigatório para recursos editáveis por múltiplos usuários)

### Problema

```
Usuário A: GET /tasks/123 → version: 1
Usuário B: GET /tasks/123 → version: 1
Usuário A: PUT /tasks/123 → salva (version: 2) ✓
Usuário B: PUT /tasks/123 → sobrescreve A! ✗
```

### Estratégia: Optimistic Locking

**Como funciona:**
1. Recurso tem campo `version` (integer) ou `updatedAt` (timestamp)
2. Cliente envia versão atual no update
3. Servidor verifica se versão ainda é a mesma
4. Se diferente → erro 409 Conflict

### Implementação

#### Opção 1: Campo `version` no body

```json
// GET /tasks/123
{ "id": "123", "title": "Task", "version": 5 }

// PUT /tasks/123
{ "title": "Updated", "version": 5 }

// Se version != 5 no banco → 409 Conflict
```

#### Opção 2: Header `If-Match` (recomendado para REST puro)

```http
GET /tasks/123
ETag: "v5"

PUT /tasks/123
If-Match: "v5"

# Se ETag mudou → 412 Precondition Failed
```

### Template obrigatório

```markdown
## Concorrência: `[nome_do_recurso]`

### Estratégia
- **Método:** [version field | ETag/If-Match]
- **Campo:** `version` (integer, incrementa a cada update)

### Comportamento em conflito
- **Status:** 409 Conflict
- **Code:** `RESOURCE.CONFLICT`
- **Response:**
  ```json
  {
    "code": "TASK.CONFLICT",
    "message": "Recurso foi modificado por outro usuário",
    "severity": "WARNING",
    "currentVersion": 6,
    "yourVersion": 5
  }
  ```

### Resolução de conflito
- **Cliente:** Buscar versão atual, mostrar diff, permitir merge
- **Ou:** Last-write-wins (se dados não críticos)
```

### Quando aplicar

| Cenário | Locking necessário? |
|---------|---------------------|
| Documento editável por múltiplos usuários | ✓ Sim |
| Configurações do sistema | ✓ Sim |
| Transações financeiras | ✓ Sim (considerar pessimistic) |
| Logs, métricas (append-only) | Não |
| Recursos só editáveis pelo owner | Opcional |

**Anti-pattern:** Permitir PUT/PATCH em recursos compartilhados sem verificar versão.

---

## 2.14 Soft Delete vs Hard Delete (obrigatório definir estratégia)

### Definições

| Tipo | O que acontece | Dados |
|------|----------------|-------|
| **Soft Delete** | Marca como deletado (`deleted_at`) | Mantém no banco |
| **Hard Delete** | Remove do banco | Perdido permanentemente |

### Quando usar cada um

| Critério | Soft Delete | Hard Delete |
|----------|-------------|-------------|
| Auditoria necessária | ✓ | - |
| Possível "desfazer" | ✓ | - |
| GDPR "right to be forgotten" | - | ✓ |
| Economia de espaço | - | ✓ |
| Dados PII sensíveis | Depende | ✓ (após período legal) |
| Relacionamentos complexos | ✓ (evita orphans) | Cuidado (cascade) |

### Template obrigatório

```markdown
## Estratégia de Deleção: `[nome_do_recurso]`

### Tipo
- **Método:** [Soft Delete | Hard Delete | Híbrido]

### Se Soft Delete
- **Campo:** `deletedAt` (timestamp, nullable)
- **Índice:** Incluir `WHERE deleted_at IS NULL` em queries
- **Cascade:** [Soft delete em filhos também | Bloquear se tiver filhos]
- **Restauração:** [Permitido via admin | Não permitido]
- **Retenção:** Manter por [90 dias | 1 ano | indefinido]
- **Purge:** [Automático após retenção | Manual | Nunca]

### Se Hard Delete
- **Cascade:** [ON DELETE CASCADE | SET NULL | RESTRICT]
- **Backup:** [Snapshot antes de deletar | Não]
- **Audit log:** [Registrar quem deletou e quando]

### GDPR/LGPD (se PII)
- **Anonimização:** [Após X dias, substituir PII por hash]
- **Right to erasure:** [Hard delete sob solicitação | Processo documentado]
```

### Implementação de Soft Delete

```sql
-- Tabela
ALTER TABLE tasks ADD COLUMN deleted_at TIMESTAMP NULL;

-- Query padrão (sempre filtrar)
SELECT * FROM tasks WHERE deleted_at IS NULL;

-- Índice parcial (performance)
CREATE INDEX idx_tasks_active ON tasks(id) WHERE deleted_at IS NULL;
```

### Erros relacionados

| Situação | Status | Code |
|----------|--------|------|
| Tentar acessar recurso soft-deleted | 404 | `RESOURCE.NOT_FOUND` |
| Tentar deletar recurso com dependências | 409 | `RESOURCE.HAS_DEPENDENCIES` |
| Tentar restaurar recurso hard-deleted | 404 | `RESOURCE.NOT_FOUND` |

**Anti-patterns:**
- ❌ Soft delete sem filtrar em queries (dados "fantasma" aparecem)
- ❌ Hard delete de PII sem audit log
- ❌ Soft delete indefinido sem política de purge (banco cresce infinitamente)

---

## Arquivos relacionados

- `@rules/RULES-CORE.MD` — Regras fundamentais (erros, validações, IDs)
- `@rules/RULES-AUTH.MD` — Regras de autenticação e usuários
- `@AGENT-DOC.MD` — Fluxo principal do workflow
