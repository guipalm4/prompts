# Análise crítica de 3 prompts para geração de documentação agêntica (Cursor IDE)

## Escopo e premissas

Esta análise considera **três prompts existentes no repositório** como “Prompt 1/2/3”, por serem o conjunto coeso de prompts na pasta `prompts/experts/` que descrevem comportamento e entregáveis para geração de documentação consumida por agentes (Cursor IDE e similares):

- **Prompt 1:** `prompts/experts/BASE-GPT.MD`
- **Prompt 2:** `prompts/experts/BASE-SONNET.MD`
- **Prompt 3:** `prompts/experts/BASE-GEMINI.MD`

> Observação: não há ocorrências literais de “Prompt 1/2/3” nos arquivos do workspace; portanto, esta numeração é uma **convenção desta análise** para permitir comparação direta.

---

## Análise do Prompt [Número do Prompt: 1]

*   **Resumo do Prompt:**
    *   O `BASE-GPT.MD` define um **assistente de discovery + especificação técnica** com foco explícito em **documentação pronta para desenvolvimento agêntico no Cursor IDE**, priorizando: anti-ambiguidade, segurança por padrão, anti-alucinação, manutenibilidade e operação segura de agentes. Estrutura o trabalho em **4 fases** (com perguntas guiadas) e exige linguagem normativa (“DEVE/NÃO DEVE”).

*   **Pros:**
    *   **Clareza operacional forte:** explicita prioridades, políticas obrigatórias (anti-ambiguidade, anti-alucinação, segurança by design) e orienta o comportamento do agente (não ser “yes-man”).
    *   **Boa “agent safety”:** inclui “anti-zumbi” com checkpoints, limites de escopo e aprovações humanas para ações destrutivas.
    *   **Orientação para outputs verificáveis:** recomenda critérios mensuráveis, exemplos, formatos e “fontes de verdade”; isso reduz alucinação e facilita validação.
    *   **Segurança integrada ao requisito (não decorativa):** exige casos de abuso e controles, o que tende a melhorar qualidade do backlog e testes.
    *   **Ritmo de descoberta controlado:** limita número de perguntas por vez, ajudando na interação com o usuário.

*   **Contras:**
    *   **Entregáveis finais pouco padronizados:** o prompt descreve princípios/políticas e fases, mas não define com precisão quais **documentos finais** devem ser gerados (ex.: PRD, Tech Specs, ADR, etc.) e seus templates.
    *   **Dependência de “fontes de verdade” genéricas:** cita OWASP/RFCs, mas não obriga mapeamento para artefatos locais (ex.: `openapi.yaml`, estrutura do repo), o que pode gerar documentação “bonita” porém desconectada do projeto.
    *   **4 fases vs necessidades típicas do Cursor:** falta instrução explícita para produzir artefatos específicos para o Cursor (ex.: `.cursorrules`, convenções de edição, checklist de execução no IDE).
    *   **Pode induzir excesso de perguntas:** sem um critério de “suficiência” bem definido por tipo de projeto, pode alongar a Fase 1 e atrasar a geração de artefatos.

*   **Falhas Identificadas e Sugestões de Correção:**
    *   **Falha 1: ausência de contrato de saída (arquivos/artefatos).** O prompt fala em “documentação” mas não fixa quais arquivos produzir, nem estrutura mínima. Sugestão: adicionar uma seção “Artefatos obrigatórios” com nomes, objetivos e tópicos mínimos (ex.: `PRD.md`, `TECH_SPECS.md`, `SECURITY.md`, `ADR.md`, `RUNBOOK.md`, `.cursorrules`).
    *   **Falha 2: falta de integração explícita com o workspace.** Não exige ler arquivos existentes. Sugestão: incluir regra “antes de especificar, listar fontes locais (README, `openapi.yaml`, `package.json`, etc.) e apontar lacunas”.
    *   **Falha 3: critérios de qualidade/testabilidade não estão amarrados a testes.** Há “mensurável”, mas não pede explicitamente casos de teste/gherkin por feature. Sugestão: exigir Gherkin (happy/unhappy paths) e uma matriz de rastreabilidade requisito→teste.
    *   **Falha 4: política de versões existe, mas não é operacionalizada.** Sugestão: impor que toda tecnologia citada venha com versão e alternativa; e um bloco “stack constraints” consumível por agentes.

---

## Análise do Prompt [Número do Prompt: 2]

*   **Resumo do Prompt:**
    *   O `BASE-SONNET.MD` é um prompt mais **extenso e estruturado**, focado em transformar ideias em documentação técnica completa para desenvolvimento agêntico. Define princípios (precisão técnica, segurança, determinismo, observabilidade) e um fluxo em **5 fases** com um conjunto grande de perguntas (inclui segurança/compliance e escala/performance).

*   **Pros:**
    *   **Cobertura de engenharia mais ampla:** além de requisitos e segurança, inclui observabilidade, versionamento, compatibilidade e guidelines de testes.
    *   **Maior detalhamento e rigor:** incentiva schemas, contratos (OpenAPI), tipos (TypeScript/JSON Schema) e exemplos concretos.
    *   **Bom para projetos “de verdade” (não só MVP):** adiciona dimensões comuns que agentes costumam omitir (logs, métricas, tracing, políticas de compatibilidade).
    *   **Perguntas adicionais úteis:** segurança/compliance e performance ajudam a reduzir risco de documentação incompleta.

*   **Contras:**
    *   **Tamanho e complexidade:** por ser longo (654 linhas), aumenta chance de o agente “perder” instruções e gerar saída inconsistente (principalmente em sessões longas no Cursor).
    *   **5 fases diferentes dos outros prompts do repo:** pode causar confusão ao combinar com outros agentes (PM/Arquiteto/Dev/QA) ou “bases” alternativas.
    *   **Ainda não garante entregáveis padronizados para o Cursor:** menciona OpenAPI, schemas, etc., mas não torna obrigatório produzir `.cursorrules` ou um conjunto padrão de arquivos.
    *   **Risco de burocracia:** pode induzir documentação excessiva para problemas simples, aumentando custo de adoção.

*   **Falhas Identificadas e Sugestões de Correção:**
    *   **Falha 1: falta um “modo leve” vs “modo completo”.** Sugestão: introduzir perfis (MVP/Produto Crítico) com checklists e profundidade mínima por perfil.
    *   **Falha 2: ausência de definição explícita de formato de saída (contrato).** Sugestão: adicionar uma seção “Saída obrigatória” com templates e ordem de geração (ex.: primeiro `PRD.md`, depois `TECH_SPECS.md`, etc.).
    *   **Falha 3: integração com Cursor IDE é indireta.** Sugestão: incluir instruções específicas (ex.: gerar `.cursorrules`, convenções de edição, limites de mudanças, checkpoints e como o agente deve navegar pelo codebase).
    *   **Falha 4: risco de inconsistência interna (muitas regras espalhadas).** Sugestão: no topo, adicionar um “Resumo de Regras Inquebráveis” e um “Checklist de conformidade” para o agente validar antes de entregar.

---

## Análise do Prompt [Número do Prompt: 3]

*   **Resumo do Prompt:**
    *   O `BASE-GEMINI.MD` é um prompt **mais curto e prescritivo** com o posicionamento “Agentic Architect”, enfatizando “zero ambiguidade”, “anti-vibe coding”, segurança por design, anti-alucinação, explicitação de trade-offs e contenção anti-zumbi. Estrutura 4 fases e, na Fase 4, **define entregáveis concretos**: `.cursorrules`, `PRD.md`, `TECH_SPECS.md`, `SECURITY_IMPLEMENTATION.md`, `ARCHITECTURE_DECISION_RECORD.md`.

*   **Pros:**
    *   **Mais alinhado ao Cursor explicitamente:** traz `.cursorrules` como artefato central, com conteúdo detalhado.
    *   **Contrato de saída forte:** lista arquivos e o que cada um deve conter, o que reduz variação e melhora reprodutibilidade por agentes.
    *   **Ênfase em tipagem forte e padrões que ajudam IA:** recomenda escolhas (TypeScript/Zod, Go, Rust etc.) para reduzir erros de implementação.
    *   **Pragmático e operacional:** menor que o `BASE-SONNET.MD`, tende a ser mais fácil de seguir em sessões reais.

*   **Contras:**
    *   **Pode ser “stack-biased”:** sugere tipagem forte e algumas stacks, mas sem critério claro de seleção; pode enviesar a arquitetura mesmo quando não é adequado.
    *   **Menos profundidade em observabilidade/operabilidade:** não explicita logging/métricas/tracing, runbooks, SLOs, etc.
    *   **“Validar versões mentalmente” é frágil:** sem exigir consulta a manifestos do repo (ex.: `package.json`) ou documentação oficial, pode falhar no objetivo anti-alucinação.
    *   **Risco de assumir muitos detalhes sem insumo:** por ser direto, pode pular perguntas essenciais (ex.: requisitos não funcionais), gerando documentação incompleta.

*   **Falhas Identificadas e Sugestões de Correção:**
    *   **Falha 1: ausência de regra de ancoragem no repositório.** Sugestão: exigir leitura de “fontes locais” antes de fixar stack/versões; e, se não houver, registrar como “INCERTO” + perguntas.
    *   **Falha 2: falta um bloco mínimo de testes/qualidade.** Sugestão: adicionar seção obrigatória em `TECH_SPECS.md` com estratégia de testes (unit/integration/e2e), e exemplos de critérios de aceite por feature.
    *   **Falha 3: falta observabilidade e operação.** Sugestão: adicionar `RUNBOOK.md`/`OBSERVABILITY.md` (logs, métricas, alertas) e “definição de pronto” para deploy.
    *   **Falha 4: “forbidden actions” e “core dirs” precisam ser operacionalizados.** Sugestão: pedir uma lista explícita de diretórios/arquivos sensíveis do projeto e regras de edição (ex.: nunca tocar em migrações sem aprovação).

---

## Recomendação (prompt aprimorado, combinando pontos fortes)

A melhor síntese para uso no Cursor IDE (com agentes implementando código) é:

1. **Manter o “contrato de saída” do Prompt 3** (artefatos concretos, `.cursorrules`, e templates por arquivo).
2. **Incorporar as políticas de segurança/anti-alucinação e operação segura do Prompt 1** (fontes de verdade, checkpoints, linguagem normativa DEVE/NÃO DEVE).
3. **Adicionar “modo leve vs completo” + observabilidade/testes do Prompt 2** (para evitar burocracia em projetos pequenos, mas cobrir produção quando necessário).

### Prompt consolidado (versão sugerida)

> Use o texto abaixo como base única para gerar documentação agêntica no Cursor.

**Papel:** Você é um arquiteto/PM técnico focado em documentação pronta para desenvolvimento agêntico no Cursor IDE.

**Regras inquebráveis:**
- Não invente APIs, bibliotecas, arquivos ou comandos. Quando incerto, marque **INCERTO** e faça perguntas.
- Toda feature deve ter: requisitos funcionais + não-funcionais + casos de erro + requisitos de segurança + critérios de aceite testáveis.
- Antes de decidir stack/versões, liste “Fontes de Verdade” locais (ex.: `openapi.yaml`, manifests, README). Se não existirem, registre e pergunte.
- Defina limites de atuação do agente (arquivos/diretórios proibidos), checkpoints e aprovações humanas para ações destrutivas.

**Fases (4):**
1) Discovery & Risk: 3–4 perguntas por vez; incluir PII/compliance; confirmar objetivos e restrições.
2) Arquitetura & Stack: propor 1–2 opções com trade-offs; fixar versões; registrar ADRs.
3) Blueprint: schemas, endpoints (OpenAPI-like), fluxos, regras de validação e autorização.
4) Entrega: gerar arquivos abaixo.

**Artefatos obrigatórios (gerar em Markdown com títulos claros):**
- `.cursorrules`: contexto do projeto, stack/versões permitidas, regras de segurança, limites de edição, ações proibidas.
- `PRD.md`: user stories + critérios Given/When/Then + unhappy paths.
- `TECH_SPECS.md`: modelos/schemas, endpoints, integrações, performance, estratégia de testes.
- `SECURITY_IMPLEMENTATION.md`: authN/authZ, OWASP Top 10 aplicado ao projeto, rate limit, logging seguro.
- `ADR.md`: decisões principais com contexto/decisão/consequências.
- `OBSERVABILITY.md` (opcional conforme perfil): logs/métricas/tracing, alarmes.
- `RUNBOOK.md` (opcional conforme perfil): como rodar, deploy, rollback, incident response.

**Perfis:**
- *MVP:* pode omitir `OBSERVABILITY.md`/`RUNBOOK.md`, mas nunca omitir segurança + testes básicos.
- *Produção/Crítico:* exige todos os artefatos e requisitos de observabilidade.

---

## Observações finais (como isso impacta o uso no Cursor)

- Prompts com **contrato de saída** (Prompt 3) reduzem retrabalho no Cursor porque o agente sabe exatamente quais arquivos criar/atualizar.
- Prompts com **políticas anti-alucinação e “fontes de verdade”** (Prompt 1) reduzem divergência entre documentação e o repositório real.
- Prompts muito longos (Prompt 2) tendem a precisar de “sumário de regras inquebráveis” para evitar que o agente ignore instruções durante sessões extensas.
