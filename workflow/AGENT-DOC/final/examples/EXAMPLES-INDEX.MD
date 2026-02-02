# AGENT-DOC — Exemplos Concretos

> **Quando usar:** Referência durante geração de artefatos para garantir consistência.
> **Carregue com:** `@examples/EXAMPLES-INDEX.MD` + arquivo específico

---

## Projeto de Exemplo: TaskFlow

**Contexto:** Sistema de gerenciamento de tarefas para equipes pequenas.
- **Perfil:** INTERNO (2-50 usuários, rede corporativa)
- **Stack:** Node.js + TypeScript + PostgreSQL + React

---

## Arquivos de Exemplo

| Arquivo | Conteúdo | Linhas |
|---------|----------|--------|
| `@examples/EXAMPLES-PRD.MD` | PRD.md completo (user stories, Gherkin, regras) | ~250 |
| `@examples/EXAMPLES-TECH.MD` | TECH_SPECS.md completo (API contracts, modelos, testes) | ~450 |
| `@examples/EXAMPLES-SECURITY.MD` | SECURITY_IMPLEMENTATION.md (threat model, RBAC, OWASP) | ~200 |
| `@examples/EXAMPLES-INFRA.MD` | INFRASTRUCTURE.md (deploy, CI/CD, backup) | ~200 |
| `@examples/EXAMPLES-ADR.MD` | ADR.md (4 decisões arquiteturais) | ~150 |

---

## Quando carregar cada exemplo

| Fase | Exemplo a carregar |
|------|--------------------|
| FASE 3 (PRD) | `@examples/EXAMPLES-PRD.MD` |
| FASE 3 (Tech Specs) | `@examples/EXAMPLES-TECH.MD` |
| FASE 4 (Security) | `@examples/EXAMPLES-SECURITY.MD` |
| FASE 4 (Infra) | `@examples/EXAMPLES-INFRA.MD` |
| FASE 2-4 (ADRs) | `@examples/EXAMPLES-ADR.MD` |

---

## Exemplo: `.cursor/rules/project-context.md`

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

## Estrutura Completa de Artefatos

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
