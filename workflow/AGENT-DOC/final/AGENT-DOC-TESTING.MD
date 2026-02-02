# AGENT-DOC — Estratégia de Testes

> **Quando usar:** FASE 3 (Blueprint) para definir estratégia de testes e FASE 4 para validar implementação.
> **Carregue com:** `@AGENT-DOC-TESTING.MD`

---

## 1. Filosofia: Test-First (TDD)

### 1.1 Regra inquebrável

**NENHUMA feature implementada sem testes escritos ANTES.**

**Fluxo obrigatório:**
1. Definir `TEST-xxx` em TECH_SPECS.md (FASE 3)
2. Implementar teste (FASE 4)
3. Implementar código para fazer teste passar
4. Refatorar se necessário
5. Validar cobertura mínima

### 1.2 Benefícios

- **Segurança:** Garantia de que código funciona antes de escrever
- **Documentação:** Testes servem como especificação executável
- **Refatoração segura:** Testes garantem que nada quebrou
- **Regressão:** Suite completa evita bugs recorrentes

---

## 2. Pirâmide de Testes por Perfil

| Perfil | Unit | Integration | E2E | Cobertura Mínima | Smoke Tests |
|--------|------|-------------|-----|------------------|-------------|
| **PESSOAL** | Críticos apenas | - | - | 40% | Opcional |
| **INTERNO** | Sim | Sim | Smoke apenas | 60% | Obrigatório |
| **PRODUCAO** | Sim | Sim | Sim | 80% | Obrigatório |

### 2.1 Definições

**Unit Tests:**
- Testam função/método isoladamente
- Mocks para dependências externas
- Rápidos (< 100ms cada)
- Foco em lógica de negócio

**Integration Tests:**
- Testam integração entre componentes
- Banco de dados de teste (isolado)
- APIs externas mockadas
- Média velocidade (< 1s cada)

**E2E Tests:**
- Testam fluxo completo do usuário
- Ambiente próximo de produção
- Mais lentos (< 10s cada)
- Foco em happy paths críticos

**Smoke Tests:**
- Subconjunto mínimo de testes críticos
- Executados antes de cada deploy
- Devem passar em < 2 minutos
- Garantem que sistema básico funciona

---

## 3. Padrão de Teste por Camada

### 3.1 API Layer

**Obrigatório para cada `API-xxx`:**

```typescript
// Teste de contrato (request/response/erros)
describe('API-001: Criar Projeto', () => {
  it('TEST-001: Deve criar projeto com dados válidos', async () => {
    // Given: usuário autenticado
    // When: POST /api/v1/projects
    // Then: 201 Created + projeto retornado
  });
  
  it('TEST-002: Deve retornar erro ao criar projeto duplicado', async () => {
    // Given: projeto já existe
    // When: POST com nome duplicado
    // Then: 409 Conflict + código de erro
  });
});
```

**Cobertura mínima:**
- ✅ Happy path (200/201)
- ✅ Unhappy path (400/404/409/422)
- ✅ Validações de input
- ✅ Autenticação/autorização

### 3.2 Services Layer

**Testes unitários com mocks:**

```typescript
describe('ProjectService.create', () => {
  it('TEST-010: Deve criar projeto e adicionar owner como membro', async () => {
    // Mock: prisma.project.create
    // Mock: prisma.projectMember.create
    // Assert: owner adicionado automaticamente
  });
});
```

**Cobertura mínima:**
- ✅ Lógica de negócio (RULE-xxx)
- ✅ Tratamento de erros
- ✅ Validações de regras

### 3.3 Database Layer

**Testes com banco isolado:**

```typescript
describe('ProjectRepository', () => {
  beforeEach(async () => {
    await resetDatabase(); // Limpar banco de teste
  });
  
  it('TEST-020: Deve garantir unicidade de nome', async () => {
    // Given: projeto com nome "Alpha"
    // When: tentar criar outro com mesmo nome
    // Then: erro de constraint único
  });
});
```

**Cobertura mínima:**
- ✅ Constraints de banco
- ✅ Relacionamentos
- ✅ Queries complexas

### 3.4 Frontend Layer

**Testes de componente + E2E:**

```typescript
// Componente
describe('ProjectForm', () => {
  it('TEST-030: Deve validar nome antes de submit', () => {
    // Renderizar componente
    // Preencher nome inválido
    // Assert: mensagem de erro exibida
  });
});

// E2E (fluxo crítico)
describe('E2E: Criar Projeto', () => {
  it('TEST-100: Fluxo completo de criação', async () => {
    // Login → Criar projeto → Verificar na lista
  });
});
```

**Cobertura mínima:**
- ✅ Componentes críticos (forms, modais)
- ✅ Fluxos principais (E2E)
- ✅ Validações de frontend

---

## 4. Testes de Regressão

### 4.1 O que são

Testes que garantem que **features existentes continuam funcionando** após mudanças.

### 4.2 Estratégia

**Antes de implementar nova feature:**
1. ✅ Rodar suite completa de testes
2. ✅ Todos devem passar (baseline)
3. ✅ Registrar tempo de execução

**Após implementar feature:**
1. ✅ Rodar suite completa novamente
2. ✅ Comparar com baseline
3. ✅ Se algum teste falhou → **BUG DE REGRESSÃO**

**CI/CD:**
- ✅ Pipeline DEVE bloquear merge se testes falharem
- ✅ Smoke tests executados em cada PR
- ✅ Suite completa executada antes de merge em main

### 4.3 Script de Smoke Test

**Obrigatório para INTERNO e PRODUCAO:**

```bash
#!/bin/bash
# smoke-test.sh

echo "🧪 Executando smoke tests..."

# Testes críticos apenas
npm test -- --grep "TEST-001|TEST-002|TEST-010|TEST-100"

if [ $? -eq 0 ]; then
  echo "✅ Smoke tests passaram"
  exit 0
else
  echo "❌ Smoke tests falharam - bloqueando deploy"
  exit 1
fi
```

**Integração CI/CD:**

```yaml
# .github/workflows/smoke-tests.yml
on: [pull_request]
jobs:
  smoke:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: ./scripts/smoke-test.sh
```

---

## 5. Template de Cenário de Teste

### 5.1 Formato padrão (Gherkin)

```gherkin
# TEST-xxx: [Título descritivo]
# FEAT: FEAT-xxx
# Tipo: unit | integration | e2e
# Prioridade: P0 (crítico) | P1 (importante) | P2 (nice-to-have)
# Cobertura: [O que este teste cobre]

Given [pre-condição]
And [setup adicional]
When [ação]
Then [resultado esperado]
And [efeito colateral se houver]
```

### 5.2 Exemplo completo

```gherkin
# TEST-001: Criação de projeto com dados válidos
# FEAT: FEAT-001
# Tipo: integration
# Prioridade: P0
# Cobertura: Happy path de criação de projeto

Given um usuário autenticado com role "admin"
And não existe projeto com nome "Projeto Alpha"
When ele envia POST /api/v1/projects com body:
  """
  {
    "name": "Projeto Alpha",
    "description": "Descrição do projeto"
  }
  """
Then o sistema retorna status 201 Created
And o response contém:
  """
  {
    "data": {
      "id": "[uuid]",
      "name": "Projeto Alpha",
      "ownerId": "[user-id]",
      "status": "active"
    }
  }
  """
And o projeto existe no banco de dados
And o usuário criador é automaticamente adicionado como owner
```

### 5.3 Mapeamento TEST → FEAT → API

| TEST | FEAT | API | Tipo | Prioridade |
|------|------|-----|------|------------|
| TEST-001 | FEAT-001 | API-001 | integration | P0 |
| TEST-002 | FEAT-001 | API-001 | integration | P0 |
| TEST-010 | FEAT-001 | - | unit | P1 |
| TEST-100 | FEAT-001 | API-001 | e2e | P0 |

---

## 6. Integração com Workflow AGENT-DOC

### 6.1 FASE 3 — Blueprint

**Ações obrigatórias:**

1. **Para cada `FEAT-xxx`:**
   - Definir pelo menos 1 `TEST-xxx` (happy path)
   - Definir pelo menos 2 `TEST-xxx` (unhappy paths)
   - Especificar tipo (unit/integration/e2e)
   - Definir prioridade (P0/P1/P2)

2. **Documentar em TECH_SPECS.md:**
   ```markdown
   ## 6. Estratégia de Testes
   
   | FEAT | TEST IDs | Tipo | Cenários |
   |------|----------|------|----------|
   | FEAT-001 | TEST-001, TEST-002 | integration | criar válido/inválido |
   ```

3. **Definir cobertura mínima:**
   - PESSOAL: 40% (críticos apenas)
   - INTERNO: 60% (unit + integration)
   - PRODUCAO: 80% (unit + integration + e2e)

### 6.2 FASE 4 — Entrega

**Ações obrigatórias:**

1. **Gerar arquivo `TESTING_STRATEGY.md`** (se perfil >= INTERNO)
   - Ver template em `@AGENT-DOC-TEMPLATES.MD`

2. **Implementar testes ANTES do código:**
   - Escrever teste primeiro
   - Verificar que falha (red)
   - Implementar código mínimo para passar (green)
   - Refatorar se necessário (refactor)

3. **Validar cobertura:**
   ```bash
   npm test -- --coverage
   # Verificar se atinge mínimo do perfil
   ```

4. **Configurar CI/CD:**
   - Pipeline bloqueia merge se testes falharem
   - Smoke tests em cada PR
   - Suite completa antes de merge

---

## 7. Checklist de Testes

### 7.1 Por Feature (FASE 3)

- [ ] Cada `FEAT-xxx` tem pelo menos 1 teste definido
- [ ] Cada `API-xxx` tem teste de contrato (request/response/erro)
- [ ] Testes unhappy path definidos (mínimo 2 por feature)
- [ ] Tipo de teste especificado (unit/integration/e2e)
- [ ] Prioridade definida (P0/P1/P2)
- [ ] Cobertura mínima definida conforme perfil

### 7.2 Por Implementação (FASE 4)

- [ ] Teste escrito ANTES do código (TDD)
- [ ] Teste passa após implementação
- [ ] Cobertura mínima atingida
- [ ] Smoke test atualizado (se aplicável)
- [ ] CI/CD configurado para bloquear merge em falha
- [ ] Suite de regressão executada e passou

### 7.3 Por Deploy

- [ ] Smoke tests executados e passaram
- [ ] Suite completa executada (se mudança crítica)
- [ ] Nenhum teste de regressão falhou
- [ ] Tempo de execução dentro do esperado

---

## 8. Ferramentas Recomendadas

### 8.1 Por Stack

**Node.js/TypeScript:**
- **Framework:** Vitest ou Jest
- **Assertions:** expect (built-in)
- **Mocks:** vi.fn() (Vitest) ou jest.fn()
- **Coverage:** @vitest/coverage-v8 ou jest --coverage

**Python:**
- **Framework:** pytest
- **Assertions:** assert (built-in)
- **Mocks:** unittest.mock
- **Coverage:** pytest-cov

**Frontend (React/Next.js):**
- **Unit:** Vitest + React Testing Library
- **E2E:** Playwright ou Cypress
- **Coverage:** @vitest/coverage-v8

### 8.2 Configuração mínima

**vitest.config.ts:**
```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 60, // Ajustar conforme perfil
        functions: 60,
        branches: 60,
        statements: 60,
      },
    },
  },
});
```

---

## 9. Exemplos Práticos

### 9.1 Teste Unitário (Service)

```typescript
// TEST-010: ProjectService deve validar nome único
describe('ProjectService.create', () => {
  it('deve lançar erro se nome já existe', async () => {
    // Arrange
    const mockRepo = {
      findByName: vi.fn().mockResolvedValue({ id: '1', name: 'Alpha' }),
    };
    const service = new ProjectService(mockRepo);
    
    // Act & Assert
    await expect(
      service.create({ name: 'Alpha', ownerId: 'user-1' })
    ).rejects.toThrow('PROJECT.NAME_TAKEN');
  });
});
```

### 9.2 Teste de Integração (API)

```typescript
// TEST-001: API deve criar projeto válido
describe('POST /api/v1/projects', () => {
  it('deve retornar 201 com projeto criado', async () => {
    const response = await request(app)
      .post('/api/v1/projects')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ name: 'Alpha', description: 'Test' });
    
    expect(response.status).toBe(201);
    expect(response.body.data).toMatchObject({
      name: 'Alpha',
      status: 'active',
    });
    expect(response.body.data.id).toBeDefined();
  });
});
```

### 9.3 Teste E2E (Playwright)

```typescript
// TEST-100: Fluxo completo de criação de projeto
test('usuário pode criar projeto e ver na lista', async ({ page }) => {
  // Login
  await page.goto('/login');
  await page.fill('[name="email"]', 'admin@test.com');
  await page.fill('[name="password"]', 'password');
  await page.click('button[type="submit"]');
  
  // Criar projeto
  await page.click('text=Criar Projeto');
  await page.fill('[name="name"]', 'Projeto E2E');
  await page.click('button[type="submit"]');
  
  // Verificar na lista
  await expect(page.locator('text=Projeto E2E')).toBeVisible();
});
```

---

## 10. Referências rápidas

### IDs de teste
- `TEST-001` a `TEST-099`: Integration tests
- `TEST-100` a `TEST-199`: E2E tests
- `TEST-010`, `TEST-020`, etc.: Unit tests (por camada)

### Prioridades
- **P0:** Crítico — deve passar sempre, bloqueia deploy
- **P1:** Importante — deve passar na maioria dos casos
- **P2:** Nice-to-have — pode falhar ocasionalmente

### Cobertura mínima
- **PESSOAL:** 40% (apenas críticos)
- **INTERNO:** 60% (unit + integration)
- **PRODUCAO:** 80% (unit + integration + e2e)

---

## Arquivos relacionados

- `@AGENT-DOC-TEMPLATES.MD` — Template de `TESTING_STRATEGY.md`
- `@AGENT-DOC-CHECKLIST.MD` — Validações de testes
- `@AGENT-DOC-RULES.MD` — Regra 2.1 (Anti-ambiguidade) aplica a testes
