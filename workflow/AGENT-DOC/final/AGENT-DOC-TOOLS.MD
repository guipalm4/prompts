# AGENT-DOC — Ferramentas e Integrações

> **Quando usar:** Referência durante FASE 1-4 para uso correto de MCPs, Skills, SubAgents e Commands.
> **Carregue com:** `@AGENT-DOC-TOOLS.MD`

---

## 1. MCPs Recomendados

### 1.1 Context7 MCP (Obrigatório)

**Propósito:** Consultar documentação oficial de bibliotecas ANTES de recomendar ou usar.

**Regra inquebrável:** 
- **DEVE** usar `resolve-library-id` + `query-docs` antes de propor qualquer lib/API
- **NÃO DEVE** assumir existência ou API de bibliotecas sem consultar
- Integra com regra anti-alucinação (2.5 do `@AGENT-DOC-RULES.MD`)

**Fluxo de uso:**

1. **Antes de recomendar uma lib:**
   ```
   1. resolve-library-id: "nome-da-lib"
   2. query-docs: { libraryId, query: "como usar X" }
   3. Verificar versão compatível com stack
   4. Registrar em project-context.md
   ```

2. **Ao validar stack existente:**
   ```
   1. Ler package.json/requirements.txt
   2. Para cada lib listada, consultar Context7 se necessário
   3. Verificar compatibilidade de versões
   ```

**Quando usar:**
- FASE 1: Validar stack proposta
- FASE 2: Confirmar versões e APIs de libs
- FASE 3: Verificar contratos de APIs externas

**Exemplo de integração:**

```markdown
## Stack Proposta

| Lib | Versão | Validação |
|-----|--------|-----------|
| Fastify | 4.x | ✅ Consultado via Context7 - API estável |
| Prisma | 5.x | ✅ Consultado via Context7 - compatível com Node 20+ |
```

---

### 1.2 GitHub MCP

**Propósito:** Gerenciar issues e PRs durante desenvolvimento.

**Uso recomendado:**

1. **Criar issues para FEATs (FASE 4):**
   - Após gerar PRD.md, criar issue para cada FEAT-xxx
   - Usar template padronizado com labels
   - Vincular issues entre si quando há dependências

2. **Criar PRs automatizados:**
   - Após implementar feature, criar PR com template
   - Incluir referência ao FEAT-xxx e TEST-xxx
   - Adicionar checklist de validação

**Comandos úteis:**

| Comando | Quando usar |
|---------|-------------|
| `create_issue` | Criar issue para FEAT após PRD |
| `create_pull_request` | Após implementar feature |
| `list_issues` | Verificar progresso do projeto |
| `get_issue` | Consultar contexto de FEAT específica |

**Template de Issue:**

```markdown
## FEAT-001: [Título]

**Tipo:** Feature
**Prioridade:** P0 | P1 | P2
**Dependências:** FEAT-002, FEAT-003

### Descrição
[Do PRD.md]

### Critérios de Aceite
- [ ] TEST-001 passa
- [ ] TEST-002 passa
- [ ] Documentação atualizada

### Referências
- PRD.md: FEAT-001
- TECH_SPECS.md: API-001
```

---

### 1.3 Task-Master-AI MCP

**Propósito:** Orquestrar tarefas de implementação a partir do PRD.

**Fluxo de uso:**

1. **Após gerar PRD.md (FASE 4):**
   ```
   parse_prd:
     input: "docs/PRD.md"
     projectRoot: "/caminho/absoluto/projeto"
     numTasks: 0  # Deixa Task-Master decidir
     force: true
   ```

2. **Durante implementação:**
   ```
   get_tasks: # Ver todas as tarefas
   next_task: # Obter próxima tarefa priorizada
   get_task: { taskId } # Ver detalhes de tarefa específica
   set_task_status: { taskId, status } # Atualizar progresso
   ```

3. **Expandir tarefas complexas:**
   ```
   expand_task: { taskId } # Quebrar em subtarefas
   ```

**Integração com workflow:**

- **FASE 4:** Parsear PRD → gerar tasks.json
- **Implementação:** Usar `next_task` para guiar agente
- **Checkpoint:** Atualizar status após cada FEAT completa

**Estrutura esperada:**

```
projeto/
├── .taskmaster/
│   ├── docs/
│   │   └── prd.txt  # Cópia do PRD.md
│   └── tasks/
│       └── tasks.json  # Gerado pelo parse_prd
```

### 1.4 Workflow Task-Master + Documentação

**Antes de `next_task`:**
1. Identificar FEAT-xxx da task
2. Ler seção correspondente em PRD.md
3. Ler seção correspondente em TECH_SPECS.md
4. Ler APIs relacionadas (API-xxx)
5. Verificar regras de negócio (RULE-xxx)
6. Verificar testes esperados (TEST-xxx)

**Durante implementação:**
1. Seguir contrato de API exatamente como documentado
2. Implementar validações conforme JSON Schema
3. Usar códigos de erro conforme Mapa de Erros
4. Consultar Context7 para sintaxe de libs

**Antes de `set_task_status: done`:**
1. Executar DOD completo (ver regra `.cursor/rules/implementation.md`)
2. Verificar que todos TEST-xxx da FEAT passam
3. Validar versões de libs contra TECH_SPECS

**Template de consulta:**
```
Para implementar FEAT-xxx:
1. @PRD.md buscar FEAT-xxx → critérios de aceite
2. @TECH_SPECS.md buscar API-xxx → contrato
3. @TECH_SPECS.md buscar modelo de dados
4. @SECURITY_IMPLEMENTATION.md verificar requisitos de segurança
```

### 1.5 Context7 Durante Implementação

**Regra inquebrável:** Consultar Context7 ANTES de usar qualquer lib/API durante codificação.

**Fluxo obrigatório:**

```
1. Identificar lib a usar (ex: FastAPI, Prisma, etc.)
2. Verificar versão em TECH_SPECS.md
3. resolve-library-id: "nome-da-lib"
4. query-docs: { libraryId, query: "como fazer X" }
5. Validar sintaxe está atualizada para versão do projeto
6. Registrar consulta em learnings.md
```

**Quando usar Context7 durante implementação:**

| Situação | Ação |
|----------|------|
| Primeira vez usando lib no projeto | Consultar setup/configuração |
| Sintaxe não lembrada | Consultar exemplo de uso |
| Dúvida sobre API | Consultar documentação específica |
| Versão diferente da que conhece | Consultar breaking changes |
| Erro estranho | Consultar troubleshooting |

**Anti-pattern:** NÃO assumir sintaxe de memória. SEMPRE verificar via Context7.

---

## 2. Skills do Cursor

**Propósito:** Criar habilidades reutilizáveis para padrões específicos do projeto.

### 2.1 Quando criar Skills

Crie Skills quando:
- Padrão se repete em múltiplos projetos
- Regra específica de domínio (ex: "sempre usar Prisma para DB")
- Workflow customizado (ex: "sempre criar migration antes de alterar schema")

**NÃO crie Skills para:**
- Regras genéricas (use `.cursor/rules/` ao invés)
- Padrões já cobertos por regras existentes

### 2.2 Estrutura recomendada

```markdown
# SKILL.md

## Nome da Skill
[Descrição breve]

## Quando usar
[Contexto específico]

## Como usar
[Passos ou exemplos]

## Exemplos
[Código quando relevante]
```

### 2.3 Integração com `.cursor/rules/`

- Skills são **complementares** às rules
- Rules definem **o quê** fazer
- Skills definem **como** fazer em contexto específico
- Skills podem referenciar rules existentes

**Exemplo:**

```markdown
# Skill: Criar Migration Prisma

## Quando usar
Ao alterar schema do banco de dados.

## Como usar
1. Editar prisma/schema.prisma
2. Executar: npx prisma migrate dev --name descricao
3. Verificar migration gerada
4. Atualizar .cursor/rules/database.mdc se necessário

## Referências
- Regra: @database.mdc (validações de schema)
```

---

## 3. SubAgents do Cursor

**Propósito:** Delegar tarefas especializadas para agentes focados.

### 3.1 Tipos de SubAgents

| Tipo | Uso | Quando usar |
|------|-----|-------------|
| `explore` | Explorar codebase, buscar padrões | FASE 1 (discovery), investigar bugs |
| `generalPurpose` | Tarefas gerais de código | Implementação, refatoração |
| `shell` | Executar comandos, scripts | CI/CD, migrations, builds |

### 3.2 Padrão de delegação por fase

**FASE 1 — Discovery:**
```
- SubAgent explore: Mapear estrutura de diretórios
- SubAgent explore: Identificar padrões existentes
- SubAgent generalPurpose: Analisar dependências
```

**FASE 2 — Arquitetura:**
```
- SubAgent explore: Buscar ADRs similares
- SubAgent generalPurpose: Validar compatibilidade de stack
```

**FASE 3 — Blueprint:**
```
- SubAgent generalPurpose: Gerar exemplos de código
- SubAgent explore: Verificar consistência de contratos
```

**FASE 4 — Entrega:**
```
- SubAgent generalPurpose: Gerar artefatos
- SubAgent shell: Executar validações (lint, test)
```

### 3.3 Limites e boas práticas

- **Máximo:** 4 subagents simultâneos
- **Regra:** Delegar tarefas **independentes** apenas
- **Checkpoint:** Aguardar conclusão antes de depender de resultado

**Exemplo de delegação:**

```markdown
## Tarefa: Validar stack proposta

Delegar para SubAgents:
1. SubAgent explore: Verificar compatibilidade Node.js + libs
2. SubAgent explore: Buscar vulnerabilidades conhecidas
3. SubAgent generalPurpose: Gerar exemplo de uso

Aguardar resultados → Consolidar → Decidir
```

---

## 4. Commands do Cursor

**Propósito:** Criar comandos customizados para workflows repetitivos.

### 4.1 Quando criar Commands

Crie Commands quando:
- Sequência de ações se repete frequentemente
- Workflow específico do projeto (ex: "criar feature completa")
- Automação de validações

### 4.2 Comandos recomendados por tipo de projeto

**Backend (Node.js/TypeScript):**
- `create-api-endpoint`: Gerar endpoint completo (route + service + test)
- `create-migration`: Criar migration Prisma com validação
- `validate-api-contract`: Validar request/response contra schema

**Frontend (React/Next.js):**
- `create-component`: Gerar componente com testes
- `create-page`: Criar página com layout padrão
- `validate-types`: Validar tipos TypeScript

**Full-stack:**
- `create-feature`: Criar feature completa (API + Frontend + Tests)
- `run-smoke-tests`: Executar suite mínima de testes
- `validate-prd`: Validar implementação contra PRD

### 4.3 Estrutura de Command

```json
{
  "name": "create-api-endpoint",
  "description": "Cria endpoint completo seguindo padrões do projeto",
  "prompt": "Criar endpoint POST /api/v1/resource seguindo:\n- Padrão de rota de @api-conventions.mdc\n- Validação Zod\n- Teste de contrato\n- Documentação OpenAPI"
}
```

---

## 5. Integração com Workflow AGENT-DOC

### 5.1 Por fase

| Fase | Ferramentas principais | Ação |
|------|------------------------|------|
| **FASE 1** | Context7, SubAgent explore | Validar stack, explorar contexto |
| **FASE 2** | Context7, SubAgent generalPurpose | Confirmar versões, validar decisões |
| **FASE 3** | Context7, SubAgent explore | Verificar contratos, consistência |
| **FASE 4** | GitHub MCP, Task-Master, SubAgent generalPurpose | Criar issues, parsear PRD, gerar artefatos |

### 5.2 Checklist de uso

**Antes de propor qualquer lib (FASE 1-3):**
- [ ] Consultado Context7 para verificar existência e versão
- [ ] Verificado compatibilidade com stack existente
- [ ] Registrado em project-context.md

**Antes de implementar feature** (regra `.cursor/rules/implementation.md`):
- [ ] Branch dedicada criada (NÃO main/master/develop)
- [ ] PRD parseado com Task-Master (se projeto >= INTERNO)
- [ ] Issue criada no GitHub (se usando GitHub MCP)
- [ ] PLAN mode usado para planejar implementação
- [ ] Documentação consultada (PRD, TECH_SPECS, SECURITY)
- [ ] Context7 consultado para sintaxe de libs

**Durante implementação:**
- [ ] Context7 consultado para cada dúvida de sintaxe
- [ ] Versões de libs conferem com TECH_SPECS.md
- [ ] Clarification Questions feitas quando necessário
- [ ] Decisões registradas em learnings.md

**Após implementar feature:**
- [ ] DOD completo executado (regra implementation.md)
- [ ] Status atualizado no Task-Master
- [ ] PR criado com template padrão
- [ ] Métricas de precisão registradas em learnings.md
- [ ] Learnings atualizados (se houver erro)

---

## 6. Referências rápidas

### Context7
- `resolve-library-id`: Resolver nome da lib para ID
- `query-docs`: Consultar documentação específica

### GitHub MCP
- `create_issue`: Criar issue
- `create_pull_request`: Criar PR
- `list_issues`: Listar issues
- `get_issue`: Obter detalhes de issue

### Task-Master-AI
- `parse_prd`: Gerar tasks do PRD
- `get_tasks`: Listar todas as tarefas
- `next_task`: Obter próxima tarefa
- `get_task`: Obter detalhes de tarefa
- `set_task_status`: Atualizar status
- `expand_task`: Expandir em subtarefas

---

## Arquivos relacionados

- `@AGENT-DOC.MD` — Fluxo principal do workflow (4 fases)
- `@AGENT-DOC-TEMPLATES.MD` — Template da regra implementation.md (DOD, Git, Métricas)
- `@rules/RULES-CORE.MD` — Regra 2.5 (Anti-alucinação) integra com Context7
- `@AGENT-DOC-MEMORY.MD` — Gestão de memória para evitar retrabalho
