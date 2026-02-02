# Exemplo: INFRASTRUCTURE.md — TaskFlow

> **Carregue com:** `@examples/EXAMPLES-INFRA.MD`

---

```markdown
# Infrastructure — TaskFlow

> **Resumo executivo:** Deploy em containers Docker. 
> Ambiente de desenvolvimento local via Docker Compose. 
> Staging/Produção em servidor corporativo com Docker.

## 1. Ambientes

| Ambiente | URL | Propósito | Quem acessa |
|----------|-----|-----------|-------------|
| local | localhost:3000 | Desenvolvimento | Devs |
| staging | taskflow-staging.corp.local | Testes | Time |
| production | taskflow.corp.local | Uso real | Colaboradores |

## 2. Variáveis de Ambiente

| Variável | Obrigatória | Exemplo | Onde obter |
|----------|-------------|---------|------------|
| DATABASE_URL | Sim | postgres://user:pass@host:5432/db | Vault |
| REDIS_URL | Sim | redis://host:6379 | Config |
| JWT_PRIVATE_KEY | Sim | [gerado] | Vault |
| JWT_PUBLIC_KEY | Sim | [gerado] | Vault |
| SSO_CLIENT_ID | Sim | taskflow | SSO Admin |
| SSO_CLIENT_SECRET | Sim | [secret] | Vault |
| SSO_ISSUER_URL | Sim | https://sso.corp.local | SSO Admin |
| NODE_ENV | Sim | production | Deploy config |
| LOG_LEVEL | Não | info | Deploy config |

## 3. Dependências de Infraestrutura

| Serviço | Versão | Propósito | Fallback |
|---------|--------|-----------|----------|
| PostgreSQL | 15+ | Banco principal | - |
| Redis | 7+ | Cache/Sessions | Graceful (sessões em DB) |
| Corporate SSO | - | Autenticação | - (crítico) |
| SMTP Relay | - | Notificações | Queue para retry |

## 4. CI/CD Pipeline

```mermaid
flowchart LR
    A[Push] --> B[Lint + Test]
    B --> C{Pass?}
    C -->|No| D[Notify + Stop]
    C -->|Yes| E[Build Docker]
    E --> F{Branch?}
    F -->|main| G[Deploy Staging]
    G --> H[Smoke Tests]
    H --> I{Pass?}
    I -->|No| D
    I -->|Yes| J[Manual Approval]
    J --> K[Deploy Production]
    F -->|feature| L[End]
```

### Stages

| Stage | Trigger | Ações | Tempo |
|-------|---------|-------|-------|
| Lint + Test | Push | ESLint, TypeScript, Vitest | ~3min |
| Build | Test pass | docker build | ~5min |
| Deploy Staging | main branch | docker-compose up | ~2min |
| Smoke Tests | Auto | Healthcheck + basic API test | ~1min |
| Deploy Prod | Manual | docker-compose up | ~2min |

## 5. Deploy

### Docker Compose (Staging/Production)

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  api:
    image: taskflow-api:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=taskflow
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

### Comandos

```bash
# Build
docker build -t taskflow-api:latest .

# Deploy staging
cd /opt/taskflow/staging
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Deploy production
cd /opt/taskflow/production
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Ver logs
docker-compose logs -f api

# Rollback
docker-compose -f docker-compose.prod.yml up -d --no-deps api
# (usando imagem anterior)
```

### Checklist Pré-Deploy

- [ ] Testes passando em CI
- [ ] Migrations executadas em staging
- [ ] Smoke tests passando em staging
- [ ] Variáveis de ambiente verificadas
- [ ] Backup do banco realizado (se breaking change)
- [ ] Rollback plan documentado
- [ ] Horário de baixo uso (se possível)

## 6. Rollback

### Procedimento

1. Identificar versão anterior: `docker images taskflow-api`
2. Atualizar tag em docker-compose ou fazer pull da versão anterior
3. `docker-compose up -d --no-deps api`
4. Verificar healthcheck: `curl http://localhost:3000/health`
5. Se migrations: executar rollback script

### Tempo máximo: 5 minutos

## 7. Monitoramento e Alertas

| Métrica | Threshold | Alerta | Ação |
|---------|-----------|--------|------|
| API Health | Down > 1min | Email TI | Verificar containers |
| Error rate | > 5% | Email TI | Investigar logs |
| Response time p95 | > 1s | Log | Investigar |
| Disk usage | > 80% | Email TI | Limpar/expandir |
| Memory usage | > 90% | Email TI | Restart/investigar |

### Healthcheck Endpoint

```
GET /health
Response: { "status": "ok", "db": "ok", "redis": "ok", "version": "1.0.0" }
```

## 8. Backup e Recovery

| Dado | Frequência | Retenção | RTO | RPO |
|------|------------|----------|-----|-----|
| PostgreSQL | Diário 2h | 30 dias | 2h | 24h |
| Redis | - | - | - | 0 (cache) |
| Uploads (se houver) | Diário | 30 dias | 2h | 24h |

### Comandos de Backup

```bash
# Backup manual
docker exec taskflow_db pg_dump -U postgres taskflow > backup_$(date +%Y%m%d).sql

# Restore
docker exec -i taskflow_db psql -U postgres taskflow < backup_20240115.sql
```

### Cron (servidor)

```cron
# Backup diário às 2h
0 2 * * * /opt/taskflow/scripts/backup.sh >> /var/log/taskflow-backup.log 2>&1
```
```
