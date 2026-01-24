# ADR - Architecture Decision Records

## ADR-001: Stack Tecnológica - Node.js/TypeScript + PostgreSQL

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento

### Contexto

O Sistema BIOSIMA precisa de uma stack moderna, escalável e adequada para processamento de dados sensíveis de saúde. O sistema deve suportar:
- API REST para múltiplos clientes (web, futuramente mobile)
- Integração com equipamentos via HL7
- Processamento de dados transacionais com alta integridade
- Conformidade com LGPD e normas de laboratórios

### Decisão

Adotar a seguinte stack:
- **Backend:** Node.js 20+ com TypeScript
- **Framework:** Express 4+ (ou Fastify como alternativa)
- **Database:** PostgreSQL 15+
- **Cache:** Redis 7+
- **Frontend:** React 18+ / Next.js 14+ com TypeScript

### Alternativas Consideradas

#### Alternativa 1: Java/Spring Boot
- **Prós:** Maturidade, forte tipagem, ampla adoção em saúde
- **Contras:** Mais verboso, curva de aprendizado maior, deploy mais complexo
- **Rejeitada porque:** Time mais familiarizado com Node.js, desenvolvimento mais rápido

#### Alternativa 2: Python/Django
- **Prós:** Simplicidade, bibliotecas científicas (se necessário)
- **Contras:** Performance inferior para APIs, GIL limita concorrência
- **Rejeitada porque:** Node.js oferece melhor performance para I/O intensivo (APIs, integrações)

#### Alternativa 3: .NET Core
- **Prós:** Forte tipagem, performance, suporte Microsoft
- **Contras:** Ecossistema menor no Brasil, custo de licenciamento (se Windows)
- **Rejeitada porque:** Menor familiaridade do time, preferência por stack open-source

### Consequências

**Ganhos:**
- Desenvolvimento rápido com TypeScript (tipagem + produtividade)
- Ecossistema rico (npm) para integrações (HL7, PDF, etc.)
- Escalabilidade horizontal fácil (API stateless)
- PostgreSQL oferece ACID completo e integridade referencial
- Redis para cache e sessões melhora performance

**Perdas:**
- Node.js single-threaded pode ser limitante para processamento CPU-intensivo (não é o caso aqui)
- TypeScript adiciona passo de compilação (mitigado por tooling moderno)

**Riscos:**
- Dependências npm podem ter vulnerabilidades (mitigado por scans regulares)
- PostgreSQL requer conhecimento de SQL e otimização (mitigado por ORM/Query Builder)

---

## ADR-002: Estratégia de Autenticação - JWT Stateless

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento + Segurança

### Contexto

O sistema precisa autenticar múltiplos tipos de usuários (pacientes, médicos, funcionários) com diferentes níveis de acesso. Requisitos:
- Suporte a múltiplos clientes (web, futuramente mobile/API)
- Escalabilidade horizontal (stateless)
- Segurança robusta (LGPD, dados de saúde)
- Experiência do usuário (sem re-login frequente)

### Decisão

Adotar autenticação baseada em **JWT (JSON Web Tokens)** com:
- **Access Token:** Expiração de 1 hora, assinado com RS256 (ou HS256 se não houver infraestrutura de chaves)
- **Refresh Token:** Expiração de 7 dias, armazenado no banco de dados
- **Rotação de Refresh Token:** Novo token gerado a cada uso (refresh token rotation)

### Alternativas Consideradas

#### Alternativa 1: Session-Based (Cookies HttpOnly)
- **Prós:** Mais seguro contra XSS (HttpOnly), revogação imediata possível
- **Contras:** Requer armazenamento de sessão (Redis/DB), não funciona bem para APIs mobile
- **Rejeitada porque:** JWT oferece melhor escalabilidade e suporte a múltiplos clientes

#### Alternativa 2: OAuth2 / OpenID Connect
- **Prós:** Padrão da indústria, suporte a SSO, terceiros
- **Contras:** Complexidade maior, overkill para sistema interno
- **Rejeitada porque:** Complexidade desnecessária para MVP, pode ser adicionado depois se necessário

#### Alternativa 3: JWT sem Refresh Token
- **Prós:** Mais simples, totalmente stateless
- **Contras:** Tokens longos são risco de segurança, tokens curtos degradam UX
- **Rejeitada porque:** Refresh tokens oferecem melhor equilíbrio segurança/UX

### Consequências

**Ganhos:**
- Stateless permite escalabilidade horizontal sem compartilhamento de sessão
- Suporte nativo a múltiplos clientes (web, mobile, API)
- Refresh tokens permitem tokens de acesso curtos (mais seguro) sem degradar UX
- Rotação de refresh tokens reduz impacto de vazamento

**Perdas:**
- Revogação imediata de tokens não é trivial (requer blacklist ou esperar expiração)
- Refresh tokens requerem armazenamento (banco de dados)

**Riscos:**
- Vazamento de JWT: mitigado por expiração curta (1h) e HTTPS obrigatório
- Refresh token comprometido: mitigado por rotação e armazenamento seguro

**Mitigações:**
- Blacklist de tokens revogados (Redis) se necessário
- Monitoramento de uso anômalo de tokens
- Rate limiting em endpoints de refresh

---

## ADR-003: Estratégia de Dados - PostgreSQL Relacional

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento + Arquitetura

### Contexto

O sistema precisa armazenar dados altamente relacionais:
- Pacientes com múltiplas amostras
- Amostras com múltiplos exames
- Exames com múltiplos resultados/parâmetros
- Rastreabilidade completa (auditoria)
- Integridade referencial crítica (dados de saúde)

Requisitos:
- ACID completo
- Integridade referencial
- Queries complexas (relatórios)
- Conformidade e auditoria

### Decisão

Adotar **PostgreSQL 15+** como banco de dados principal:
- Modelo relacional normalizado
- Constraints de integridade (FK, UNIQUE, CHECK)
- Transações ACID
- Suporte a JSONB para dados flexíveis (se necessário)

### Alternativas Consideradas

#### Alternativa 1: MongoDB (NoSQL)
- **Prós:** Flexibilidade de schema, escalabilidade horizontal mais fácil
- **Contras:** Sem integridade referencial nativa, queries complexas mais difíceis
- **Rejeitada porque:** Dados altamente relacionais se beneficiam de modelo relacional, integridade é crítica

#### Alternativa 2: MySQL/MariaDB
- **Prós:** Amplamente usado, maduro
- **Contras:** Menos recursos avançados que PostgreSQL, licenciamento (MySQL Enterprise)
- **Rejeitada porque:** PostgreSQL oferece melhor suporte a JSON, tipos avançados, e é totalmente open-source

#### Alternativa 3: SQL Server
- **Prós:** Integração com ecossistema Microsoft, ferramentas robustas
- **Contras:** Custo de licenciamento, menos comum em ambientes Linux
- **Rejeitada porque:** Preferência por stack open-source, PostgreSQL atende todas as necessidades

### Consequências

**Ganhos:**
- Integridade referencial garantida pelo banco (FK constraints)
- Queries complexas eficientes (JOINs, agregações)
- Suporte a transações ACID (crítico para dados de saúde)
- JSONB permite flexibilidade quando necessário (ex: changes em audit_logs)
- Ferramentas maduras de backup/restore

**Perdas:**
- Escalabilidade horizontal mais complexa (sharding requerido)
- Schema rígido (mudanças requerem migrações)

**Riscos:**
- Escalabilidade futura: mitigado por otimizações (índices, particionamento se necessário)
- Migrações: mitigado por ferramentas de migração (Knex/TypeORM)

**Estratégias Futuras:**
- Read replicas para relatórios (se necessário)
- Particionamento de tabelas grandes (ex: audit_logs por data)
- Cache (Redis) para consultas frequentes

---

## ADR-004: Integração HL7 - Parser Customizado vs Biblioteca

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento

### Contexto

O sistema precisa receber e processar mensagens HL7 de equipamentos laboratoriais. Requisitos:
- Parse de mensagens HL7 v2.5 (ou superior)
- Validação de estrutura e dados
- Idempotência (evitar processamento duplicado)
- Tratamento de erros robusto

### Decisão

Adotar **biblioteca HL7 existente** (ex: `hl7` para Node.js ou similar) com:
- Parser de mensagens HL7
- Validação customizada de negócio (código de amostra, CPF, etc.)
- Idempotência via `MSH-10` (Message Control ID) armazenado no banco

**INCERTO:** Biblioteca específica a ser escolhida após pesquisa. Opções:
- `node-hl7` (se existir e mantida)
- `hl7-parser` (npm)
- Parser customizado se nenhuma biblioteca adequada for encontrada

### Alternativas Consideradas

#### Alternativa 1: Parser Customizado do Zero
- **Prós:** Controle total, sem dependências externas
- **Contras:** Desenvolvimento demorado, manutenção de código complexo, risco de bugs
- **Rejeitada porque:** Bibliotecas existentes reduzem tempo de desenvolvimento e risco

#### Alternativa 2: Integração via API REST (Equipamentos Enviam JSON)
- **Prós:** Mais simples, sem parser HL7
- **Contras:** Requer mudança nos equipamentos (pode não ser viável)
- **Rejeitada porque:** Equipamentos existentes já usam HL7, mudança não é viável

### Consequências

**Ganhos:**
- Desenvolvimento mais rápido usando biblioteca
- Menos código customizado para manter
- Biblioteca testada pela comunidade

**Perdas:**
- Dependência externa (requer manutenção e atualizações)
- Possível necessidade de adaptações para casos específicos

**Riscos:**
- Biblioteca descontinuada: mitigado por escolha de biblioteca ativa
- Bugs na biblioteca: mitigado por testes e possível fallback para parser customizado

**Próximos Passos:**
- Pesquisar bibliotecas HL7 disponíveis para Node.js
- Avaliar manutenção, documentação e suporte
- Escolher biblioteca ou decidir por parser customizado

---

## ADR-005: Estratégia de Observabilidade - Logs Estruturados + Métricas Básicas

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento + DevOps

### Contexto

Sistema de produção crítico precisa de:
- Rastreabilidade de erros e ações
- Monitoramento de performance
- Alertas para problemas
- Compliance (auditoria)

**INCERTO:** Ferramentas específicas de observabilidade (Datadog, New Relic, ELK, etc.)

### Decisão

Adotar estratégia de observabilidade em camadas:
1. **Logging:** Logs estruturados (JSON) com traceId
2. **Métricas:** Métricas básicas (request rate, error rate, latency) via middleware
3. **Tracing:** traceId para correlação (distributed tracing opcional no futuro)
4. **Alertas:** Alertas básicos (erro rate, latency, disponibilidade)

**Ferramentas:** A definir (baseado em orçamento e preferência)
- **Opção 1 (Cloud):** Datadog, New Relic, AWS CloudWatch
- **Opção 2 (Self-hosted):** ELK Stack (Elasticsearch, Logstash, Kibana) + Prometheus + Grafana

### Alternativas Consideradas

#### Alternativa 1: Observabilidade Completa desde o Início
- **Prós:** Visibilidade total, ferramentas avançadas
- **Contras:** Custo alto, complexidade, overkill para MVP
- **Rejeitada porque:** MVP pode começar com observabilidade básica e evoluir

#### Alternativa 2: Sem Observabilidade Estruturada
- **Prós:** Mais simples, menos custo
- **Contras:** Debugging difícil, sem visibilidade de problemas
- **Rejeitada porque:** Sistema crítico requer observabilidade mínima

### Consequências

**Ganhos:**
- Rastreabilidade via traceId permite debug eficiente
- Logs estruturados facilitam análise e busca
- Métricas básicas permitem detectar problemas rapidamente

**Perdas:**
- Distributed tracing completo requer ferramentas adicionais (pode ser adicionado depois)
- Análise avançada requer ferramentas especializadas (pode ser adicionado depois)

**Riscos:**
- Falta de visibilidade: mitigado por logs estruturados e traceId
- Custo de ferramentas: mitigado por começar com soluções básicas/self-hosted

**Evolução Futura:**
- Adicionar distributed tracing (Jaeger, Zipkin) se necessário
- Adicionar APM (Application Performance Monitoring) se necessário
- Escalar observabilidade conforme necessidade

---

## ADR-006: Estratégia de Deploy - Containerização com Docker

**Status:** Proposta  
**Data:** 2024-01-XX  
**Decisores:** Equipe de Desenvolvimento + DevOps

### Contexto

Sistema precisa ser:
- Deployável de forma consistente
- Escalável
- Fácil de manter
- Compatível com diferentes ambientes (dev, staging, produção)

**INCERTO:** Ambiente específico (cloud provider, on-prem, etc.)

### Decisão

Adotar **containerização com Docker**:
- Aplicação em containers Docker
- Docker Compose para ambiente local/desenvolvimento
- Preparado para orquestração (Kubernetes, Docker Swarm, ou serviços gerenciados como ECS/EKS)

**Estratégia de Deploy:**
- **Desenvolvimento:** Docker Compose (app + PostgreSQL + Redis)
- **Staging/Produção:** Containers em orquestrador (Kubernetes ou serviço gerenciado)

### Alternativas Consideradas

#### Alternativa 1: Deploy Tradicional (Servidor Físico/Virtual)
- **Prós:** Simples, sem complexidade de containers
- **Contras:** Difícil de escalar, inconsistência entre ambientes
- **Rejeitada porque:** Containers oferecem melhor portabilidade e escalabilidade

#### Alternativa 2: Serverless (AWS Lambda, etc.)
- **Prós:** Escalabilidade automática, sem gerenciamento de servidores
- **Contras:** Cold starts, limites de tempo, custo para workloads constantes
- **Rejeitada porque:** Sistema tem carga constante, serverless não é ideal

### Consequências

**Ganhos:**
- Consistência entre ambientes (dev, staging, produção)
- Escalabilidade horizontal fácil
- Isolamento de dependências
- Facilita CI/CD

**Perdas:**
- Complexidade adicional (Docker, orquestração)
- Curva de aprendizado

**Riscos:**
- Complexidade de orquestração: mitigado por começar com Docker Compose e evoluir
- Overhead de containers: mínimo, aceitável

**Próximos Passos:**
- Definir ambiente de produção (cloud/on-prem)
- Escolher orquestrador se necessário
- Configurar CI/CD para build e deploy de containers
