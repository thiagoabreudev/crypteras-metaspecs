---
adr_number: "000"
title: "Architecture Decision Records (ADRs)"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['readme', 'technical', 'decision', 'architecture']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# Architecture Decision Records (ADRs)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::explainability
**Requirement**: ✅ Required (para TODOS os ADRs)

**Propósito de ADRs**:
ADRs (Architecture Decision Records) **JÁ SÃO** explicações de decisões arquiteturais. Cada ADR documenta:
- **Context**: Por que a decisão foi necessária
- **Decision**: O que foi decidido
- **Consequences**: Impactos positivos e negativos
- **Alternatives Considered**: Outras opções avaliadas

**Como IA Deve Usar ADRs**:

1. **SEMPRE Consultar ADRs Relevantes** antes de:
   - Criar nova entity, service, ou componente arquitetural
   - Escolher tecnologia, framework, ou biblioteca
   - Refatorar código que afeta múltiplos módulos
   - Implementar integração com serviço externo

2. **Referenciar ADR em Explicações**:
   ```markdown
   **Source**:
   - ADR-001: Clean Architecture v1.2.0 - Constraint "Domain NUNCA importa frameworks"
   - ADR-005: Adapter Pattern v1.0.0 - BaseExchange interface
   ```

3. **Validar Compliance com ADR**:
   - Verificar se decisão atual viola constraints de ADR ativo
   - Se violar: Justificar exceção OU criar novo ADR superseding
   - Se alinhado: Documentar aplicação do padrão

**ADRs Críticos por Área**:

| Área | ADRs Obrigatórios | Quando Consultar |
|------|------------------|------------------|
| **Arquitetura** | ADR-001 (Clean Architecture) | Criar entities, repositories, use cases |
| **Exchanges** | ADR-005 (Adapter Pattern) | Integrar com MB, Binance, ou nova exchange |
| **Workflows** | ADR-002 (Celery), ADR-004 (Co-Prime) | Criar/modificar tasks assíncronas |
| **Banco de Dados** | ADR-003 (MongoDB) | Schema design, queries, indexação |
| **IA/Agentes** | ADR-006 (AGNO) | Implementar agentes, chat, workflows com LLM |
| **Deprecation** | ADR-007 (TradingConfig) | Evitar usar código deprecated |

**Template para Criar Novo ADR**:
```markdown
# ADR-XXX: [Título da Decisão]

## Status
🟡 Proposto | ✅ Aceito | ⚠️ Depreciado | 🔴 Substituído

## Contexto
[Qual problema estamos resolvendo? Por quê agora?]

## Decisão
[O que decidimos fazer? Como funcionará?]

## Consequências

### Positivas ✅
1. [Benefício 1]
2. [Benefício 2]

### Negativas ⚠️
1. [Trade-off 1]
2. [Mitigação]

## Alternativas Consideradas

### Alternativa 1: [Nome]
**Prós**: ...
**Contras**: ...
**Razão para Rejeitar**: ...

## Implementação
[Evidências no código, arquivos afetados]

## Referências
- [Links para docs, issues, PRs]
```

**Quando Criar Novo ADR** (IA deve sugerir):
1. **Breaking Architectural Change**: Mudar de MongoDB para PostgreSQL
2. **Novo Pattern Significativo**: Introduzir Event Sourcing, CQRS
3. **Escolha de Tecnologia Crítica**: Framework de frontend, message broker
4. **Deprecation de Padrão Existente**: Como ADR-007 (TradingConfig)
5. **Trade-off com Impacto Duradouro**: Performance vs Manutenibilidade

**Quando NÃO Criar ADR**:
- Decisões táticas/reversíveis facilmente
- Escolhas triviais sem trade-offs
- Implementações específicas já cobertas por ADR existente
- Refatorações internas sem impacto arquitetural
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção `:::explainability` com guia completo de uso de ADRs pela IA
- Definida tabela de ADRs críticos por área
- Incluído template para criar novos ADRs
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Primeira versão versionada de README.md
:::

Este diretório contém registros de decisões arquiteturais importantes tomadas no projeto Crypteras Trading System.

## O que é um ADR?

Um Architecture Decision Record (ADR) documenta uma decisão arquitetural significativa junto com seu contexto e consequências. Cada ADR descreve:

- **Contexto**: Qual problema estávamos tentando resolver?
- **Decisão**: O que decidimos fazer?
- **Consequências**: Quais são os trade-offs e impactos dessa decisão?
- **Status**: Aceito, Proposto, Depreciado, Substituído

## Índice de ADRs

### ADRs Ativos

| ID | Título | Versão | Status | Data |
|----|--------|--------|--------|------|
| [001](001-clean-architecture.md) | Adoção de Clean Architecture | v1.0.0 | ✅ Aceito | 2024-12-24 |
| [002](002-celery-redis-migration.md) | Migração de APScheduler para Celery + Redis | v1.0.0 | ✅ Aceito | 2024-12-24 |
| [003](003-mongodb-over-postgresql.md) | Escolha de MongoDB ao invés de PostgreSQL | v1.0.0 | ✅ Aceito | 2024-12-24 |
| [004](004-coprime-intervals.md) | Uso de Intervalos Co-Primos para Workflows | v1.0.0 | ✅ Aceito | 2024-12-24 |
| [005](005-adapter-pattern-exchanges.md) | Adapter Pattern para Multi-Exchange Support | v1.0.0 | ✅ Aceito | 2024-12-24 |
| [006](006-agno-framework-ai.md) | Framework AGNO para Agentes de IA | v1.0.0 | ✅ Aceito | 2024-12-24 |

### ADRs Depreciados

| ID | Título | Versão | Status | Substituído Por |
|----|--------|--------|--------|-----------------|
| [007](007-trading-config-deprecated.md) | TradingConfig Entity (Depreciado) | v1.0.0 | ⚠️ Depreciado | Bots independentes (CRY-82) |

## Como Usar Este Diretório

### Para Novos Desenvolvedores
Leia os ADRs ativos na ordem para entender as decisões arquiteturais fundamentais do projeto.

### Ao Propor Mudança Arquitetural
Crie um novo ADR seguindo o template abaixo:

```markdown
# ADR-XXX: [Título da Decisão]

## Status
Proposto | Aceito | Depreciado | Substituído por ADR-YYY

## Contexto
[Descreva o problema ou situação que motivou a decisão]

## Decisão
[Descreva a decisão tomada]

## Consequências

### Positivas
- [Benefício 1]
- [Benefício 2]

### Negativas
- [Trade-off 1]
- [Trade-off 2]

## Alternativas Consideradas
- [Opção A]: [Razão para rejeitar]
- [Opção B]: [Razão para rejeitar]

## Referências
- [Link para issue, PR, ou documentação relacionada]
```

## Convenções

1. **Numeração**: ADRs são numerados sequencialmente (001, 002, 003...)
2. **Imutabilidade**: ADRs aceitos não devem ser editados. Se uma decisão for revertida, crie um novo ADR marcando o antigo como "Depreciado"
3. **Clareza**: Escreva para que qualquer desenvolvedor entenda o raciocínio anos depois
4. **Rastreabilidade**: Referencie issues, PRs, ou discussões relevantes

---

**Última atualização**: 2024-12-24
