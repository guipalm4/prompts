# AGENT-DOC — Gestão de Memória e Contexto

> **Quando usar:** FASE 1 e FASE 4 para evitar retrabalho e manter contexto persistente.
> **Carregue com:** `@AGENT-DOC-MEMORY.MD`

---

## 1. Problema: Retrabalho e Perda de Contexto

### 1.1 Cenários problemáticos

**Erros repetidos:**
- Agente comete mesmo erro em projetos diferentes
- Solução já encontrada não é reutilizada
- Padrões que funcionam não são documentados

**Contexto perdido:**
- Decisões tomadas durante sessão são "esquecidas"
- Agente repete perguntas já respondidas
- Informações importantes não persistem entre sessões

**Pesquisas repetidas:**
- Mesma documentação consultada múltiplas vezes
- APIs já validadas são pesquisadas novamente
- Libs já verificadas são consultadas repetidamente

### 1.2 Solução: Memória Persistente

Criar **artefatos persistentes** que mantêm contexto entre sessões e projetos.

---

## 2. Tipos de Memória

| Tipo | Onde Persistir | Quando Atualizar | Formato |
|------|----------------|------------------|---------|
| **Decisões arquiteturais** | `ADR.md` | Ao tomar decisão | ADR-xxx |
| **Erros comuns** | `.cursor/rules/learnings.md` | Ao resolver bug | Tabela |
| **Libs/versões usadas** | `.cursor/rules/project-context.md` | Ao adicionar dep | Lista |
| **Consultas de docs** | `.cursor/rules/learnings.md` | Ao consultar API | Cache |
| **Padrões que funcionam** | `.cursor/rules/learnings.md` | Ao validar padrão | Lista |
| **Tarefas e progresso** | `.taskmaster/tasks/tasks.json` | Continuamente | JSON |

---

## 3. Artefato: `.cursor/rules/learnings.md`

### 3.1 Propósito

Arquivo persistente que armazena:
- Erros conhecidos e suas soluções
- Padrões que funcionam bem
- Cache de consultas de documentação
- Lições aprendidas do projeto

### 3.2 Estrutura

```markdown
---
description: Lições aprendidas e erros a evitar
alwaysApply: true
---

# Learnings — [Nome do Projeto]

## Erros Conhecidos

| Data | Erro | Solução | Contexto | FEAT/API |
|------|------|---------|----------|----------|
| 2024-01-15 | Prisma N+1 query | Usar `include` ao invés de queries separadas | API-001 | FEAT-001 |
| 2024-01-16 | Validação Zod não normaliza | Adicionar `.transform()` antes de `.parse()` | API-002 | FEAT-002 |
| 2024-01-17 | CORS bloqueando requests | Configurar `origin` explícito no Fastify | - | Setup |

## Padrões que Funcionam

### Padrão: Validação de Input
**Quando usar:** Sempre que receber input do usuário
**Como:** Usar Zod schema com `.transform()` para normalização
**Exemplo:**
```typescript
const schema = z.object({
  email: z.string().email().transform(v => v.trim().toLowerCase()),
});
```
**Funciona bem para:** Normalização de dados antes de validação

### Padrão: Tratamento de Erros
**Quando usar:** Em todos os endpoints
**Como:** Usar classe de erro customizada com código
**Exemplo:**
```typescript
throw new AppError('PROJECT.NOT_FOUND', 'Projeto não encontrado', 404);
```
**Funciona bem para:** Consistência de respostas de erro

## Libs Consultadas (Cache)

| Lib | Versão | Doc URL | Última Consulta | Notas |
|-----|--------|---------|-----------------|-------|
| Fastify | 4.x | https://fastify.dev | 2024-01-15 | API estável, compatível Node 20+ |
| Prisma | 5.x | https://prisma.io/docs | 2024-01-15 | Migrations funcionam bem |
| Zod | 3.x | https://zod.dev | 2024-01-16 | Validação + transform útil |

## Consultas de API Externa

| API | Endpoint | Método | Última Consulta | Resposta Esperada |
|-----|----------|--------|-----------------|-------------------|
| GitHub API | /repos/{owner}/{repo} | GET | 2024-01-17 | 200 OK com dados do repo |
| SSO Corporativo | /oauth/token | POST | 2024-01-17 | 200 OK com access_token |

## Decisões de Design

| Data | Decisão | Justificativa | ADR |
|------|---------|---------------|-----|
| 2024-01-15 | Usar Fastify ao invés de Express | Performance 2x melhor | ADR-001 |
| 2024-01-16 | Validação com Zod | Type-safety + runtime validation | ADR-002 |

## Comandos Úteis

| Comando | Quando usar | Resultado |
|---------|-------------|-----------|
| `npm run test:watch` | Durante desenvolvimento | Testes em modo watch |
| `npx prisma migrate dev` | Ao alterar schema | Cria migration + aplica |
| `npm run lint:fix` | Antes de commit | Corrige problemas de lint |

## Notas de Debug

| Data | Problema | Solução | Ferramenta |
|------|----------|---------|------------|
| 2024-01-18 | Testes lentos | Usar `--run` ao invés de watch em CI | Vitest |
| 2024-01-19 | Build falhando | Limpar `.next` antes de rebuild | Next.js |
```

### 3.3 Regras de uso

**REGRA 1:** Ao encontrar erro, DEVE registrar em learnings.md
- Incluir data, erro, solução, contexto
- Vincular com FEAT/API quando aplicável

**REGRA 2:** Ao consultar Context7, registrar lib/versão em learnings.md
- Evitar consultas repetidas
- Manter cache de documentação consultada

**REGRA 3:** Antes de pesquisar, verificar se já existe em learnings.md
- Buscar por palavra-chave
- Reutilizar soluções conhecidas

**REGRA 4:** Ao validar padrão que funciona, documentar em "Padrões que Funcionam"
- Incluir quando usar, como usar, exemplo
- Facilitar reutilização em outros projetos

---

## 4. Integração com Task-Master-AI

### 4.1 Parsear PRD

**Quando:** Após gerar PRD.md na FASE 4

**Como:**
```bash
# Via MCP Task-Master
parse_prd:
  input: "docs/PRD.md"
  projectRoot: "/caminho/absoluto/projeto"
  numTasks: 0  # Deixa decidir automaticamente
  force: true
```

**Resultado:** `.taskmaster/tasks/tasks.json` com todas as tarefas estruturadas

**Benefício:** 
- Contexto persistente das tarefas
- Progresso rastreável
- Não precisa reexplicar o projeto a cada sessão

### 4.2 Gerenciar Progresso

**Durante implementação:**

```bash
# Obter próxima tarefa
next_task  # Retorna próxima tarefa priorizada

# Ver detalhes de tarefa específica
get_task: { taskId: "task-123" }

# Atualizar status
set_task_status: { 
  taskId: "task-123", 
  status: "completed" 
}
```

**Benefício:**
- Agente sabe exatamente onde parou
- Contexto da tarefa atual mantido
- Progresso visível

### 4.3 Expandir Tarefas Complexas

```bash
# Quebrar tarefa grande em subtarefas
expand_task: { taskId: "task-123" }
```

**Benefício:**
- Tarefas grandes viram subtarefas gerenciáveis
- Progresso granular
- Menos sobrecarga cognitiva

---

## 5. Integração com project-context.md

### 5.1 Atualizar Stack

**Quando:** Ao adicionar nova dependência ou decidir stack

**Como:** Atualizar `.cursor/rules/project-context.md`:

```markdown
## Stack Aprovada

| Camada | Tecnologia | Versão | Validação |
|--------|------------|--------|-----------|
| Runtime | Node.js | 20+ LTS | ✅ Consultado Context7 |
| Framework | Fastify | 4.x | ✅ Consultado Context7 - API estável |
| ORM | Prisma | 5.x | ✅ Consultado Context7 - compatível |
```

**Benefício:**
- Stack documentada e validada
- Versões fixadas evitam breaking changes
- Referência rápida para agente

### 5.2 Registrar Decisões

**Quando:** Ao tomar decisão arquitetural

**Como:** Criar ADR em `ADR.md`:

```markdown
## ADR-001: Fastify ao invés de Express

**Status:** Aceito
**Data:** 2024-01-15
**Decisores:** Equipe

### Contexto
[Por que essa decisão]

### Decisão
[O que foi decidido]

### Consequências
**Ganhos:**
- Performance 2x melhor
- TypeScript nativo

**Perdas:**
- Curva de aprendizado
```

**Benefício:**
- Decisões documentadas e justificadas
- Evita reabrir discussões já resolvidas
- Contexto histórico preservado

---

## 6. Regras de Contexto (Inquebráveis)

### 6.1 Antes de pesquisar

**REGRA:** Verificar se informação já existe em:
1. `learnings.md` (erros, padrões, cache)
2. `project-context.md` (stack, versões)
3. `ADR.md` (decisões arquiteturais)
4. `TECH_SPECS.md` (contratos, APIs)

**Só pesquisar se:** Não encontrado nos artefatos acima

### 6.2 Ao encontrar erro

**REGRA:** DEVE registrar imediatamente em `learnings.md`:
- Data do erro
- Descrição do erro
- Solução encontrada
- Contexto (FEAT/API relacionada)

**Formato:**
```markdown
| Data | Erro | Solução | Contexto | FEAT/API |
|------|------|---------|----------|----------|
| YYYY-MM-DD | [Descrição] | [Solução] | [Contexto] | FEAT-xxx |
```

### 6.3 Ao consultar Context7

**REGRA:** DEVE registrar em `learnings.md`:
- Nome da lib
- Versão consultada
- URL da documentação
- Data da consulta
- Notas relevantes

**Formato:**
```markdown
| Lib | Versão | Doc URL | Última Consulta | Notas |
|-----|--------|---------|----------------|-------|
| [Nome] | [Versão] | [URL] | YYYY-MM-DD | [Notas] |
```

### 6.4 Ao validar padrão

**REGRA:** Se padrão funcionou bem, DEVE documentar em "Padrões que Funcionam":
- Quando usar
- Como usar
- Exemplo de código
- Contexto onde funciona

---

## 7. Fluxo de Memória por Fase

### 7.1 FASE 1 — Discovery

**Ações:**
1. Ler `learnings.md` se existir (projeto existente)
2. Consultar Context7 para validar stack proposta
3. Registrar libs consultadas em `learnings.md`
4. Verificar `ADR.md` para decisões anteriores

**Resultado:**
- Stack validada e registrada
- Evita consultas repetidas
- Contexto de decisões anteriores preservado

### 7.2 FASE 2 — Arquitetura

**Ações:**
1. Registrar decisões em `ADR.md`
2. Atualizar `project-context.md` com stack escolhida
3. Consultar Context7 para confirmar versões
4. Registrar consultas em `learnings.md`

**Resultado:**
- Decisões documentadas
- Stack persistida
- Versões validadas

### 7.3 FASE 3 — Blueprint

**Ações:**
1. Consultar `learnings.md` antes de definir padrões
2. Reutilizar padrões que funcionam
3. Evitar erros conhecidos

**Resultado:**
- Padrões consistentes
- Menos erros repetidos

### 7.4 FASE 4 — Entrega

**Ações:**
1. Criar `learnings.md` se não existir
2. Parsear PRD com Task-Master → gerar `tasks.json`
3. Durante implementação:
   - Registrar erros em `learnings.md`
   - Atualizar progresso no Task-Master
   - Consultar `learnings.md` antes de pesquisar

**Resultado:**
- Contexto persistente
- Progresso rastreável
- Erros evitados

---

## 8. Checklist de Memória

### 8.1 Por projeto novo

- [ ] `learnings.md` criado em `.cursor/rules/`
- [ ] `project-context.md` atualizado com stack
- [ ] Libs consultadas registradas em `learnings.md`
- [ ] Task-Master configurado (se projeto >= INTERNO)

### 8.2 Por sessão

- [ ] Consultado `learnings.md` antes de pesquisar
- [ ] Verificado `project-context.md` para stack
- [ ] Verificado `ADR.md` para decisões anteriores
- [ ] Task-Master consultado para progresso atual

### 8.3 Por erro resolvido

- [ ] Erro registrado em `learnings.md`
- [ ] Solução documentada
- [ ] Contexto (FEAT/API) vinculado
- [ ] Data registrada

### 8.4 Por consulta de lib

- [ ] Lib registrada em `learnings.md` (cache)
- [ ] Versão documentada
- [ ] URL da documentação salva
- [ ] Notas relevantes adicionadas

---

## 9. Exemplos Práticos

### 9.1 Erro comum: Prisma N+1

**Antes (sem memória):**
- Agente pesquisa "Prisma N+1" toda vez
- Solução não é reutilizada
- Erro se repete

**Depois (com learnings.md):**
```markdown
| Data | Erro | Solução | Contexto | FEAT/API |
|------|------|---------|----------|----------|
| 2024-01-15 | Prisma N+1 query | Usar `include` ao invés de queries separadas | API-001 | FEAT-001 |
```

Agente consulta `learnings.md` → encontra solução → aplica diretamente

### 9.2 Consulta repetida de lib

**Antes (sem memória):**
- Context7 consultado múltiplas vezes para mesma lib
- Tokens desperdiçados
- Tempo perdido

**Depois (com learnings.md):**
```markdown
| Lib | Versão | Doc URL | Última Consulta | Notas |
|-----|--------|---------|-----------------|-------|
| Fastify | 4.x | https://fastify.dev | 2024-01-15 | API estável |
```

Agente consulta `learnings.md` → encontra cache → não precisa consultar Context7 novamente

### 9.3 Progresso perdido

**Antes (sem Task-Master):**
- Agente não sabe onde parou
- Precisa reexplicar contexto
- Tarefas duplicadas

**Depois (com Task-Master):**
```bash
next_task  # Retorna: "Implementar API-001: Criar Projeto"
get_task: { taskId: "task-123" }  # Retorna contexto completo
```

Agente sabe exatamente o que fazer → continua de onde parou

---

## 10. Referências rápidas

### Arquivos de memória

| Arquivo | O que armazena | Quando atualizar |
|---------|----------------|------------------|
| `.cursor/rules/learnings.md` | Erros, padrões, cache | Continuamente |
| `.cursor/rules/project-context.md` | Stack, versões | FASE 1-2 |
| `ADR.md` | Decisões arquiteturais | FASE 2 |
| `.taskmaster/tasks/tasks.json` | Tarefas e progresso | FASE 4+ |

### Comandos Task-Master

| Comando | Quando usar |
|---------|-------------|
| `parse_prd` | Após gerar PRD.md |
| `get_tasks` | Ver todas as tarefas |
| `next_task` | Obter próxima tarefa |
| `get_task` | Ver detalhes de tarefa |
| `set_task_status` | Atualizar progresso |
| `expand_task` | Quebrar tarefa grande |

---

## Arquivos relacionados

- `@AGENT-DOC-TOOLS.MD` — Integração com Task-Master e Context7
- `@AGENT-DOC-TEMPLATES.MD` — Template de `learnings.md`
- `@AGENT-DOC.MD` — Fluxo principal do workflow
