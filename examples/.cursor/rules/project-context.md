---
description: Contexto geral do projeto - aplicado a todos os arquivos
alwaysApply: true
---

# Contexto do Projeto

## Resumo
Sistema de gerenciamento de laboratório clínico (LIMS) para processamento de amostras e resultados de exames. Perfil: **PRODUÇÃO** (dados de saúde, PII, LGPD).

## Stack Aprovada

### Backend
- **Runtime:** Node.js 20+ (LTS)
- **Linguagem:** TypeScript 5.3+
- **Framework:** Fastify 4.x (NÃO usar Express)
- **ORM:** Prisma 5.x
- **Validação:** Zod 3.x

### Banco de Dados
- **Principal:** PostgreSQL 15+
- **Cache:** Redis 7+

### Frontend
- **Framework:** Next.js 14+ (App Router)
- **UI:** Tailwind CSS 3.x + shadcn/ui
- **State:** Zustand 4.x (NÃO usar Redux)

## Estrutura de Diretórios

```
src/
├── api/           # Rotas e controllers (globs: api-conventions.mdc)
├── services/      # Lógica de negócio
├── models/        # Tipos e schemas Zod
├── db/            # Prisma schema e migrations (globs: database.mdc)
├── lib/           # Utilitários compartilhados
├── middleware/    # Middlewares Fastify
└── tests/         # Testes (globs: testing.mdc)
```

## Convenções Gerais

### Naming
- **Arquivos:** kebab-case (`user-service.ts`)
- **Classes:** PascalCase (`UserService`)
- **Funções/variáveis:** camelCase (`getUserById`)
- **Constantes:** UPPER_SNAKE_CASE (`MAX_RETRY_COUNT`)
- **Tipos/Interfaces:** PascalCase com prefixo (`IUser`, `TUserInput`)

### Imports
- DEVE usar imports absolutos com alias `@/`
- DEVE ordenar: externos → internos → tipos
- NÃO DEVE usar `require()` em TypeScript

### Commits
- Formato: `type(scope): description`
- Tipos: feat, fix, docs, style, refactor, test, chore

## Decisões Arquiteturais (ADRs)
- ADR-001: Node.js/TypeScript + PostgreSQL
- ADR-002: JWT Stateless com Refresh Token
- ADR-003: PostgreSQL Relacional (não NoSQL)
- ADR-004: Integração HL7 via biblioteca

## Links Úteis
- PRD: `/docs/PRD.md`
- Tech Specs: `/docs/TECH_SPECS.md`
- Security: `/docs/SECURITY_IMPLEMENTATION.md`
