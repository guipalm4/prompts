# Exemplo: ADR.md — TaskFlow

> **Carregue com:** `@examples/EXAMPLES-ADR.MD`

---

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
