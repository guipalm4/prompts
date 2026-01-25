# IDEA-v3 — Regras Inquebráveis

> **Quando usar:** Referência durante FASE 1-3 para garantir consistência na especificação.
> **Carregue com:** `@IDEA-v3-RULES.MD`

---

## 2.1 Anti-ambiguidade (obrigatório)

- Use linguagem normativa: **DEVE**, **NÃO DEVE**, **DEVERIA**, **PODE**.
- Proibido termos vagos sem métrica/definição: "rápido", "intuitivo", "seguro", "escalável".

Todo requisito funcional DEVE ter:
- Critérios de aceite em **Given/When/Then**.
- **Unhappy paths** (erros esperados) com códigos/formatos.
- Exemplos concretos (payloads, telas, fluxos).
- **Mínimo por feature:** ≥ 2 cenários happy e ≥ 3 cenários unhappy (ou justificar por que não se aplica).

---

## 2.2 Padrão global de erros (obrigatório)

Toda resposta de erro DEVE seguir esta estrutura:

```json
{
  "code": "DOMINIO.ERRO_ESPECIFICO",
  "message": "Mensagem segura para exibição (sem PII/secrets)",
  "severity": "CRITICAL | ERROR | WARNING | INFO",
  "details": [
    { "field": "email", "code": "INVALID_FORMAT", "message": "Email inválido" }
  ],
  "traceId": "uuid-v4",
  "i18nKey": "errors.domain.erro_especifico"
}
```

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| `code` | Sim | String estável e única: `AUTH.UNAUTHORIZED`, `USER.EMAIL_TAKEN` |
| `message` | Sim | Segura para exibição; sem PII/secrets |
| `severity` | Sim | `CRITICAL` (plantão), `ERROR` (log+alerta), `WARNING` (log), `INFO` (log) |
| `details` | Não | Lista de erros por campo |
| `traceId` | Sim | Correlação com logs |
| `i18nKey` | Não | Chave para internacionalização |

---

## 2.3 Padrão global de validação (obrigatório)

Para cada campo de input, definir em **JSON Schema** ou tabela:

```json
{
  "field": "email",
  "type": "string",
  "required": true,
  "format": "email",
  "maxLength": 255,
  "normalize": ["trim", "lowercase"],
  "errorCode": "USER.INVALID_EMAIL",
  "example": "usuario@exemplo.com"
}
```

Ou tabela:

| Campo | Tipo | Obrigatório | Limites | Formato | Normalização | Erro | Exemplo |
|-------|------|-------------|---------|---------|--------------|------|---------|
| email | string | sim | max:255 | email | trim, lowercase | USER.INVALID_EMAIL | usuario@exemplo.com |

---

## 2.4 Rastreabilidade com IDs (obrigatório)

IDs estáveis obrigatórios:
- Features: `FEAT-001`, `FEAT-002`, …
- Regras de negócio: `RULE-001`, …
- Endpoints: `API-001`, …
- ADRs: `ADR-001`, …
- Testes: `TEST-001`, `TEST-002`, …

**Garantias:**
- Cada `FEAT-xxx` aparece em PRD.md, TECH_SPECS.md e testes
- Cada `API-xxx` tem request/response, auth, validações e erros
- Cada `FEAT-xxx` tem pelo menos um `TEST-xxx`

**Matriz de rastreabilidade obrigatória:**

| FEAT | RULE | API | TEST | ADR |
|------|------|-----|------|-----|
| FEAT-001 | RULE-001, RULE-002 | API-001 | TEST-001, TEST-002 | ADR-001 |

---

## 2.5 Anti-alucinação (obrigatório)

- **NÃO invente** APIs, libs, endpoints, serviços ou comandos.
- **ANTES** de referenciar qualquer lib/API:
  1. Buscar no codebase existente
  2. Consultar documentação oficial (Context7 MCP ou web search)
  3. Confirmar existência e versão

**Declare Fontes de Verdade:**
- **Locais:** `package.json`, `requirements.txt`, `openapi.yaml`
- **Externas:** OWASP, RFCs, documentação oficial

Se não houver dados: marque **INCERTO** e pergunte.

---

## 2.6 Segurança por design (obrigatório)

Toda feature DEVE incluir requisitos de segurança e casos de abuso.

**Proibido:**
- Segredos/chaves em frontend/logs/repo
- SQL concatenado
- Senha sem hashing forte
- Endpoints sem authZ

**Exigido:**
- Validação/normalização de input
- AuthN + AuthZ (RBAC/ABAC) quando aplicável
- Rate limiting / proteção brute-force
- Logging seguro (sem PII/secrets)
- Classificação de dados (PII/sensível) + retenção/minimização
