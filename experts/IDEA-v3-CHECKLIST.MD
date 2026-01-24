# IDEA-v3 — Checklist de Validação

> **Quando usar:** Final da FASE 4, antes de entregar os artefatos.
> **Carregue com:** `@IDEA-v3-CHECKLIST.MD`

---

## Checklist executável (critérios verificáveis)

### Estrutura
- [ ] `.cursor/rules/` gerada com pelo menos `project-context.md` e `security.md`
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

### Consistência
- [ ] Todo ID referenciado existe em pelo menos um artefato
- [ ] Versões de libs/frameworks são consistentes entre artefatos
- [ ] Estrutura de diretórios corresponde aos globs nas rules

### Incertezas
- [ ] Toda incerteza está marcada como **INCERTO**
- [ ] Todo **INCERTO** tem pergunta associada

---

## Artefatos por perfil

| Artefato | PESSOAL | INTERNO | PRODUÇÃO |
|----------|---------|---------|----------|
| `.cursor/rules/` (mínimo) | ✓ | ✓ | ✓ |
| `PRD.md` | ✓ | ✓ | ✓ |
| `TECH_SPECS.md` | ✓ | ✓ | ✓ |
| `SECURITY_IMPLEMENTATION.md` | Simplificado | ✓ | ✓ Completo |
| `ADR.md` | ✓ | ✓ | ✓ |
| `INFRASTRUCTURE.md` | - | Simplificado | ✓ Completo |

---

## Referência rápida de IDs

| Prefixo | Significado | Onde aparece |
|---------|-------------|--------------|
| FEAT-xxx | Feature/User Story | PRD.md, TECH_SPECS.md |
| RULE-xxx | Regra de negócio | PRD.md, TECH_SPECS.md |
| API-xxx | Endpoint/Ação | TECH_SPECS.md |
| TEST-xxx | Cenário de teste | TECH_SPECS.md |
| ADR-xxx | Decisão arquitetural | ADR.md |
