# AGENT-DOC — Checklist de Validação

> **Quando usar:** Final da FASE 4 (artefatos) e durante implementação (validação da regra implementation.md).
> **Carregue com:** `@AGENT-DOC-CHECKLIST.MD`
> **Meta de qualidade:** 95%+ de precisão em decisões

---

## Checklist de Documentação (FASE 4) — Critérios verificáveis

### Estrutura
- [ ] `.cursor/rules/` gerada com pelo menos `project-context.md` e `security.md`
- [ ] `.cursor/rules/learnings.md` criado (obrigatório para INTERNO/PRODUCAO)
- [ ] Todos os artefatos obrigatórios para o perfil estão presentes
- [ ] Cada artefato tem resumo executivo (≤150 tokens)

### Rastreabilidade
- [ ] Todo `FEAT-xxx` aparece em PRD.md E TECH_SPECS.md
- [ ] Todo `FEAT-xxx` tem pelo menos 1 `TEST-xxx` associado
- [ ] Todo `API-xxx` tem request, response e pelo menos 1 erro documentado
- [ ] Matriz de rastreabilidade FEAT→RULE→API→TEST→ADR está completa
- [ ] Grafo de dependências entre features está presente

### Qualidade dos requisitos
- [ ] Cada `FEAT-xxx` tem ≥2 cenários happy path em Gherkin
- [ ] Cada `FEAT-xxx` tem ≥3 cenários unhappy path em Gherkin (ou justificativa)
- [ ] Nenhum termo vago sem métrica: "rápido", "seguro", "escalável"

### Validações
- [ ] Todos os campos de input têm validação definida (JSON Schema ou tabela)
- [ ] Cada validação tem `errorCode` mapeado para o padrão global de erros
- [ ] Normalização (trim, lowercase, etc.) definida por campo

### Erros
- [ ] Mapa de erros consolidado presente
- [ ] Todo erro tem: `code`, `severity`, `quando ocorre`, `status HTTP`
- [ ] Campo `i18nKey` presente (se múltiplos idiomas)

### Segurança
- [ ] AuthN/AuthZ especificados quando necessário
- [ ] Matriz de permissões (RBAC) presente quando há múltiplos roles
- [ ] Dados classificados (PII/sensível) com retenção/minimização
- [ ] OWASP Top 10 aplicado contextualmente

### Gestão de Usuários (se houver autenticação)
- [ ] Método de bootstrap definido (CRUD | Seed | CLI | Convite | OAuth)
- [ ] Comando/endpoint de criação do primeiro usuário documentado
- [ ] Dados iniciais especificados (email, senha default, role)
- [ ] Pós-bootstrap definido (trocar senha obrigatório?)
- [ ] CRUD de usuários documentado (se não for OAuth puro)

### Filtros de Data (se houver endpoints com filtros temporais)
- [ ] Cada filtro de data especifica o campo alvo (`created_at`, `updated_at`, etc.)
- [ ] Formato de data definido (ISO 8601 recomendado)
- [ ] Operadores suportados listados (`gte`, `lte`, `between`)
- [ ] Comportamento default documentado (ex: "últimos 30 dias")
- [ ] Tratamento de timezone especificado

### Transições de Estado (se houver campos enum de status)
- [ ] Tipo de transição definido (livre ou restrita/state machine)
- [ ] Se restrita: tabela de transições permitidas presente
- [ ] Permissões por transição documentadas (quem pode fazer cada uma)
- [ ] Transições automáticas identificadas (se houver)
- [ ] Reversibilidade de cada transição especificada
- [ ] Diagrama de estado presente (Mermaid) para perfil ≥ INTERNO

### Recuperação de Acesso (condicional por perfil)
- [ ] **PESSOAL:** Mencionado método de reset (pode ser "manual no banco")
- [ ] **INTERNO:** Admin-reset documentado com quem pode executar
- [ ] **PRODUÇÃO:** Self-service completo (endpoint, token, rate limit)
- [ ] **PRODUÇÃO:** Fluxo por tipo de usuário documentado
- [ ] **PRODUÇÃO:** Bloqueio após tentativas falhas definido
- [ ] **PRODUÇÃO:** Audit log de resets obrigatório

### Paginação (se houver endpoints de listagem)
- [ ] Estratégia definida (offset-based ou cursor-based)
- [ ] Limites de `perPage`/`limit` especificados (min: 1, max: 100, default: 20)
- [ ] Formato de response padronizado (`data` + `pagination`)
- [ ] Ordenação default especificada para cada endpoint de listagem
- [ ] Parâmetros de ordenação documentados (`sortBy`, `sortOrder`)

### Versionamento de API (obrigatório para INTERNO/PRODUCAO)
- [ ] Estratégia de versionamento definida (URL path recomendado)
- [ ] Versão atual documentada (`v1`, `v2`, etc.)
- [ ] Política de deprecação especificada (aviso mínimo, comunicação)
- [ ] Lista de breaking changes que incrementam versão
- [ ] Headers de deprecação documentados (`Deprecation`, `Sunset`)

### Concorrência/Locking (se recursos editáveis por múltiplos usuários)
- [ ] Estratégia de locking definida (optimistic com `version` ou `ETag`)
- [ ] Campo de versão presente no modelo de dados
- [ ] Comportamento em conflito documentado (409 Conflict)
- [ ] Response de conflito inclui `currentVersion` e `yourVersion`
- [ ] Estratégia de resolução de conflito definida (client-side merge ou last-write-wins)

### Soft Delete vs Hard Delete (para cada recurso deletável)
- [ ] Estratégia definida para cada entidade principal
- [ ] Se soft delete: campo `deletedAt` no modelo
- [ ] Se soft delete: queries filtram `deleted_at IS NULL`
- [ ] Se soft delete: política de retenção e purge documentada
- [ ] Se hard delete: cascade behavior definido (CASCADE, SET NULL, RESTRICT)
- [ ] GDPR/LGPD: estratégia de anonimização ou erasure para PII

### Consistência
- [ ] Todo ID referenciado existe em pelo menos um artefato
- [ ] Versões de libs/frameworks são consistentes entre artefatos
- [ ] Estrutura de diretórios corresponde aos globs nas rules

### Incertezas
- [ ] Toda incerteza está marcada como **INCERTO**
- [ ] Todo **INCERTO** tem pergunta associada

### Testes (expandido)
- [ ] Cada `FEAT-xxx` tem pelo menos 1 teste definido em TECH_SPECS.md
- [ ] Cada `API-xxx` tem teste de contrato (request/response/erro) definido
- [ ] Cobertura mínima definida conforme perfil (40% PESSOAL / 60% INTERNO / 80% PRODUCAO)
- [ ] Tipo de teste especificado para cada TEST-xxx (unit/integration/e2e)
- [ ] Prioridade definida para cada TEST-xxx (P0/P1/P2)
- [ ] Script de smoke test documentado (se perfil ≥ INTERNO)
- [ ] CI/CD configurado para bloquear merge em falha de testes (se perfil ≥ INTERNO)
- [ ] Testes de regressão definidos para features existentes
- [ ] Fixtures/mocks padronizados documentados (se perfil ≥ INTERNO)
- [ ] Arquivo `TESTING_STRATEGY.md` gerado (se perfil ≥ INTERNO)

### Ferramentas
- [ ] Context7 usado para validar libs recomendadas (verificado em project-context.md)
- [ ] `learnings.md` criado em `.cursor/rules/` com estrutura inicial
- [ ] `project-context.md` contém stack com versões fixadas
- [ ] Libs consultadas registradas em `learnings.md` (cache)
- [ ] Task-Master configurado e PRD parseado (se perfil ≥ INTERNO)
- [ ] GitHub issues criadas para FEATs principais (se usando GitHub MCP)

### Memória e Contexto
- [ ] `learnings.md` criado e estruturado
- [ ] Erros conhecidos registrados (se houver)
- [ ] Padrões que funcionam documentados (se houver)
- [ ] Libs consultadas registradas em cache
- [ ] Decisões arquiteturais registradas em ADR.md
- [ ] Consultado `learnings.md` antes de pesquisar (evitou retrabalho)

---

## Checklist de Implementação (regra implementation.md) — Por Task

> **Detalhes completos:** regra `.cursor/rules/implementation.md` (template em `@AGENT-DOC-TEMPLATES.MD`).

### Git (PRIMEIRO - obrigatório)
- [ ] Branch dedicada criada (`feat/FEAT-xxx`, `fix/BUG-xxx`, etc.)
- [ ] NÃO está na branch main/master/develop
- [ ] Commits seguem padrão convencional (`feat:`, `fix:`, etc.)

### Antes de começar
- [ ] Li FEAT-xxx no PRD.md
- [ ] Li API-xxx no TECH_SPECS.md
- [ ] Identifiquei RULE-xxx aplicáveis
- [ ] Identifiquei TEST-xxx que devo implementar
- [ ] Usei PLAN mode para planejar
- [ ] Fiz Clarification Questions se necessário

### Durante implementação
- [ ] Consultei Context7 para cada lib nova
- [ ] Versões conferem com TECH_SPECS.md
- [ ] Segui contrato de API exatamente
- [ ] Implementei validações conforme documentado
- [ ] Usei códigos de erro corretos

### Definition of Done (DOD) — Antes de marcar DONE

**DOD Básico (todas as tasks):**
- [ ] Código compila sem erros
- [ ] Lint passa sem erros críticos
- [ ] Formatação aplicada
- [ ] Tipos verificados

**DOD Funcional (features):**
- [ ] Teste unitário existe e passa (mínimo happy path)
- [ ] Teste de integração existe (se API)
- [ ] Versões das libs conferem com TECH_SPECS.md
- [ ] Comando funciona localmente

**DOD Infra (setup/deploy):**
- [ ] Container builda sem erro
- [ ] Container inicia sem erro
- [ ] Health check responde
- [ ] Logs não mostram erros

**DOD Métricas:**
- [ ] Clarification Questions feitas quando necessário
- [ ] Precisão da task registrada em learnings.md
- [ ] Se houve retrabalho, causa registrada

### Comando de validação (executar antes de DONE)

```bash
# Verificar branch
git branch --show-current  # Deve NÃO ser main/master/develop

# Python
make lint && make test && docker compose up -d && docker compose logs

# Node
npm run lint && npm test && docker compose up -d && docker compose logs
```

---

## Métricas de Qualidade do Processo

### Clarification Questions — Meta: 95%+ de precisão

**Definição de precisão:**
- Precisão = (Decisões corretas na primeira tentativa) / (Total de decisões)
- Uma decisão é "correta" se NÃO precisou de retrabalho posterior

**Quando Clarification Questions DEVE ser usado:**

| Situação | Obrigatório? | Se não perguntar |
|----------|--------------|------------------|
| Versão de lib não especificada exatamente | ✅ Sim | Risco de incompatibilidade |
| Múltiplas formas de implementar | ✅ Sim | Risco de refatoração |
| Comportamento de erro não definido | ✅ Sim | Risco de comportamento inesperado |
| Estrutura de diretórios não definida | ✅ Sim | Risco de reorganização |
| Dependência externa não especificada | ✅ Sim | Risco de escolha errada |
| Detalhes de UI/UX não especificados | ⚠️ Recomendado | Pode assumir padrão |
| Ordem de implementação | ⚠️ Recomendado | Pode seguir grafo de deps |

### Como registrar métricas

**Ao final de cada FEAT, registrar em `learnings.md`:**

```markdown
## Métricas FEAT-xxx (YYYY-MM-DD)

| Métrica | Valor |
|---------|-------|
| Clarification Questions feitas | X |
| Decisões que precisaram retrabalho | Y |
| Precisão | (X-Y)/X * 100% |
| Causa dos retrabalhos | [lista] |
```

### Relatório de Qualidade (ao final do projeto)

```markdown
## Relatório de Qualidade — [Projeto]

### Métricas Gerais
| Métrica | Valor | Meta |
|---------|-------|------|
| Total de Clarification Questions | X | - |
| Precisão média | Y% | 95%+ |
| Tasks sem retrabalho | Z% | 90%+ |

### Por FEAT
| FEAT | Questions | Retrabalhos | Precisão |
|------|-----------|-------------|----------|
| FEAT-001 | 3 | 0 | 100% |
| FEAT-002 | 2 | 1 | 50% |

### Lições Aprendidas
- [O que funcionou bem]
- [O que melhorar]
```

---

## Artefatos por perfil

| Artefato | PESSOAL | INTERNO | PRODUÇÃO |
|----------|---------|---------|----------|
| `.cursor/rules/` (mínimo) | ✓ | ✓ | ✓ |
| `.cursor/rules/learnings.md` | Opcional | ✓ | ✓ |
| `PRD.md` | ✓ | ✓ | ✓ |
| `TECH_SPECS.md` | ✓ | ✓ | ✓ |
| `TESTING_STRATEGY.md` | - | ✓ | ✓ |
| `SECURITY_IMPLEMENTATION.md` | Simplificado | ✓ | ✓ Completo |
| `ADR.md` | ✓ | ✓ | ✓ |
| `INFRASTRUCTURE.md` | - | Simplificado | ✓ Completo |

## Seções obrigatórias em TECH_SPECS.md (por perfil)

| Seção | PESSOAL | INTERNO | PRODUÇÃO |
|-------|---------|---------|----------|
| Gestão de Usuários (bootstrap) | Mencionar | ✓ | ✓ CRUD completo |
| Filtros de Data (se houver) | ✓ | ✓ | ✓ |
| State Machine (se houver enum status) | Tabela simples | ✓ Tabela | ✓ + Diagrama Mermaid |
| Recuperação de Acesso | - | Admin-reset | ✓ Self-service + Admin |
| **Paginação** (se listagens) | Básico | ✓ | ✓ + cursor-based |
| **Versionamento de API** | - | ✓ URL path | ✓ + política deprecação |
| **Concorrência** (se edição multi-user) | - | ✓ version field | ✓ + ETag |
| **Soft/Hard Delete** | Mencionar | ✓ | ✓ + GDPR compliance |

---

## Referência rápida de IDs

| Prefixo | Significado | Onde aparece |
|---------|-------------|--------------|
| FEAT-xxx | Feature/User Story | PRD.md, TECH_SPECS.md |
| RULE-xxx | Regra de negócio | PRD.md, TECH_SPECS.md |
| API-xxx | Endpoint/Ação | TECH_SPECS.md |
| TEST-xxx | Cenário de teste | TECH_SPECS.md |
| ADR-xxx | Decisão arquitetural | ADR.md |

---

## Arquivos relacionados

- `@AGENT-DOC.MD` — Fluxo principal (4 fases)
- `@AGENT-DOC-TEMPLATES.MD` — Templates (inclui regra implementation.md — DOD, Git, Métricas)
- `@AGENT-DOC-TOOLS.MD` — Ferramentas e integração Task-Master
- `@AGENT-DOC-TESTING.MD` — Estratégia de testes
