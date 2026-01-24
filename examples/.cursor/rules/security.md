---
description: Regras de segurança inquebráveis - aplicado a todos os arquivos
alwaysApply: true
---

# Regras de Segurança

## Classificação de Dados

| Tipo | Exemplos | Tratamento |
|------|----------|------------|
| **PII** | nome, CPF, email, telefone | Criptografia em repouso, logs mascarados |
| **Sensível** | senha, token, resultado exame | Nunca logar, hash/encrypt obrigatório |
| **Interno** | IDs, timestamps, status | Logging permitido |
| **Público** | nome do exame, código | Sem restrições |

## Regras Inquebráveis (MUST)

### Autenticação
- DEVE usar JWT com expiração máxima de 1 hora
- DEVE implementar refresh token rotation
- DEVE usar Argon2id para hash de senhas (NÃO bcrypt, NÃO MD5, NÃO SHA)
- DEVE invalidar tokens em logout

### Autorização
- DEVE verificar permissões em TODA rota protegida
- DEVE usar middleware `requireAuth` antes de acessar dados do usuário
- DEVE validar ownership antes de operações em recursos

### Input Validation
- DEVE validar TODO input com Zod antes de processar
- DEVE sanitizar strings (trim, escape HTML quando necessário)
- DEVE limitar tamanho de payloads (max 1MB default)
- NÃO DEVE confiar em headers `X-Forwarded-*` sem proxy confiável

### Database
- DEVE usar Prisma (prepared statements automáticos)
- NÃO DEVE concatenar strings em queries
- NÃO DEVE expor IDs sequenciais (usar UUID)
- DEVE implementar soft delete para dados auditáveis

### Secrets
- NÃO DEVE commitar secrets no repositório
- NÃO DEVE logar tokens, senhas ou chaves
- DEVE usar variáveis de ambiente para configuração sensível
- DEVE rotacionar secrets a cada 90 dias

## Ações Proibidas (MUST NOT)

```typescript
// ❌ NUNCA FAZER
const query = `SELECT * FROM users WHERE id = ${userId}`; // SQL Injection
console.log({ password, token }); // Vazamento de secrets
res.send(error.stack); // Exposição de internals
localStorage.setItem('token', jwt); // XSS vulnerability
```

```typescript
// ✅ SEMPRE FAZER
const user = await prisma.user.findUnique({ where: { id } }); // Prisma seguro
logger.info({ userId, action: 'login' }); // Log seguro
res.send({ code: 'ERROR', message: 'Erro interno' }); // Erro genérico
// JWT em httpOnly cookie com SameSite=Strict
```

## Checkpoints com Aprovação Humana

As seguintes ações DEVEM ter aprovação explícita antes de executar:

| Ação | Risco | Aprovador |
|------|-------|-----------|
| Migration destrutiva (DROP, DELETE) | Alto | Tech Lead |
| Alteração em roles/permissions | Alto | Tech Lead + Security |
| Deploy em produção | Médio | Tech Lead |
| Acesso a dados de produção | Alto | DPO |
| Alteração em configuração de auth | Crítico | Security Team |

## Rate Limiting Obrigatório

| Endpoint Pattern | Limite | Janela |
|------------------|--------|--------|
| `/auth/login` | 5 req | 15 min |
| `/auth/register` | 3 req | 1 hora |
| `/auth/forgot-password` | 3 req | 1 hora |
| `/api/*` (autenticado) | 100 req | 1 min |
| `/api/*` (não autenticado) | 20 req | 1 min |

## Logging Seguro

### O que DEVE logar
- Tentativas de autenticação (sucesso/falha)
- Operações CRUD em dados sensíveis
- Erros de autorização
- Rate limit atingido

### O que NUNCA logar
- Senhas (mesmo hasheadas)
- Tokens JWT completos
- Números de cartão/conta
- Dados médicos detalhados

### Formato obrigatório
```typescript
logger.info({
  traceId: req.traceId,
  userId: user?.id, // Nunca o objeto completo
  action: 'user.login',
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  // metadata adicional segura
});
```

## Headers de Segurança (via Helmet)

```typescript
// DEVE estar configurado em todo app
app.register(helmet, {
  contentSecurityPolicy: true,
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: true,
  crossOriginResourcePolicy: true,
  hsts: { maxAge: 31536000, includeSubDomains: true },
  noSniff: true,
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
});
```
