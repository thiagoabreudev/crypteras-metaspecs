---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2026-01-14"
supersedes: null
status: "active"
category: "technical"
tags: ['technical', 'index']
---

# Documentação Técnica - Crypteras Trading System

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Primeira versão versionada de index.md
:::

## Perfil de Contexto do Projeto

### Informações Básicas
- **Nome do Projeto**: Crypteras Trading System
- **Versão**: v2.9 (CRY-85: Cálculo de Quantidade com Taxas)
- **Repositório**: `/Users/thiagoabreu/workspace/crypteras-improved`
- **Stack Principal**: Python 3.x + FastAPI + Nuxt.js 3 + MongoDB + Redis + Celery
- **Ambiente de Produção**: Docker Swarm (DigitalOcean)

### Estrutura da Equipe
- **Modelo de Desenvolvimento**: Mudanças frequentes, domínio complexo
- **Prioridades**: Lógica dos bots > Workflows > Integrações
- **Débito Técnico Conhecido**:
  - Código não-PEP8
  - Resquícios de `TradingConfig` (deprecated) em bots
  - Workflows acoplados ao Mercado Bitcoin (deveria ser genérico)

### Restrições de Desenvolvimento
- **Deployment**: Docker Swarm em servidor único (DigitalOcean)
- **Monitoring**: Grafana (logs)
- **Escalabilidade**: Horizontal (múltiplos Celery workers)
- **Exchange Primária**: Mercado Bitcoin (Binance em teste)

---

## Camada 1: Contexto Central do Projeto

### 📜 Documentos Fundamentais
- [Carta do Projeto](project_charter.md) - v1.0.0 - Visão, objetivos, critérios de sucesso
- [Registros de Decisões Arquiteturais](adr/) - v1.0.0 - ADRs documentando decisões técnicas chave

---

## Camada 2: Arquivos de Contexto Otimizados para IA

### 🤖 Guias para Desenvolvimento com IA
- [Guia de Desenvolvimento com IA](CLAUDE.meta.md) - v1.0.0 - Padrões de código, testes, pegadinhas comuns
- [Guia de Navegação da Base de Código](CODEBASE_GUIDE.md) - v1.0.0 - Estrutura, arquivos chave, fluxos de dados

### 📋 Metadados de Contexto para IA
- [Índice Meta](meta/index.md) - Visão geral dos metadados de contexto
- [meta/intent.md](meta/intent.md) - Goals, Constraints e Non-Goals do desenvolvimento
- [meta/stack.md](meta/stack.md) - Stack tecnológica aprovada e ADRs chave
- [meta/failures.md](meta/failures.md) - Failure modes e anti-patterns conhecidos

---

## Camada 3: Contexto Específico do Domínio

### 💼 Lógica de Negócio
- [Documentação da Lógica de Negócio](BUSINESS_LOGIC.md) - v1.0.0 - Domínio, regras, workflows dos bots
- [Especificações da API](API_SPECIFICATION.md) - v1.0.0 - Endpoints REST, autenticação, modelos

---

## Camada 4: Contexto do Fluxo de Desenvolvimento

### 🔄 Workflows e Processos
- [Guia de Contribuição](CONTRIBUTING.md) - v1.0.0 - Fluxo de desenvolvimento, testes, deploy
- [Guia de Solução de Problemas](TROUBLESHOOTING.md) - v1.0.0 - Issues comuns, debugging, fixes

---

## Camada 5: Desafios e Melhorias

### 🎯 Oportunidades de Melhoria
- [Desafios Arquiteturais](ARCHITECTURE_CHALLENGES.md) - v1.0.0 - Débito técnico, refatorações planejadas

---

## Navegação Rápida

### Por Tipo de Tarefa

**🆕 Novo Desenvolvedor?**
1. Leia [Project Charter](project_charter.md) para entender visão
2. Siga [CODEBASE_GUIDE.md](CODEBASE_GUIDE.md) para navegação
3. Revise [CLAUDE.meta.md](CLAUDE.meta.md) para padrões de código
4. Configure ambiente com [CONTRIBUTING.md](CONTRIBUTING.md)

**🐛 Debugando Issue?**
1. Consulte [TROUBLESHOOTING.md](TROUBLESHOOTING.md) primeiro
2. Revise [BUSINESS_LOGIC.md](BUSINESS_LOGIC.md) para lógica dos bots
3. Cheque [ADRs](adr/) para contexto de decisões

**✨ Implementando Feature?**
1. Leia [ADRs relevantes](adr/) para contexto arquitetural
2. Siga padrões em [CLAUDE.meta.md](CLAUDE.meta.md)
3. Consulte [API_SPECIFICATION.md](API_SPECIFICATION.md) para endpoints
4. Teste conforme [CONTRIBUTING.md](CONTRIBUTING.md)

**🔧 Refatorando Código?**
1. Revise [ARCHITECTURE_CHALLENGES.md](ARCHITECTURE_CHALLENGES.md)
2. Entenda impacto em [BUSINESS_LOGIC.md](BUSINESS_LOGIC.md)
3. Siga princípios de [ADRs](adr/)

---

## Status da Documentação

| Documento | Status | Última Atualização |
|-----------|--------|-------------------|
| index.md | ✅ Atualizado | 2026-01-14 |
| project_charter.md | ✅ Atualizado | 2025-12-25 |
| adr/README.md | ✅ Atualizado | 2025-12-25 |
| adr/001-clean-architecture.md | ✅ Atualizado | 2025-12-25 |
| adr/002-celery-redis-migration.md | ✅ Atualizado | 2025-12-25 |
| adr/003-mongodb-over-postgresql.md | ✅ Atualizado | 2025-12-25 |
| adr/004-coprime-intervals.md | ✅ Atualizado | 2025-12-25 |
| adr/005-adapter-pattern-exchanges.md | ✅ Atualizado | 2025-12-25 |
| adr/006-agno-framework-ai.md | ✅ Atualizado | 2025-12-25 |
| adr/007-trading-config-deprecated.md | ✅ Atualizado | 2025-12-25 |
| adr/008-compensating-transactions.md | ✅ Novo | 2025-12-26 |
| CLAUDE.meta.md | ✅ Atualizado | 2025-12-25 |
| CODEBASE_GUIDE.md | ✅ Atualizado | 2025-12-25 |
| BUSINESS_LOGIC.md | ✅ Atualizado v1.4.0 | 2025-12-25 (CRY-85) |
| API_SPECIFICATION.md | ✅ Atualizado | 2025-12-25 |
| CONTRIBUTING.md | ✅ Atualizado | 2025-12-25 |
| TROUBLESHOOTING.md | ✅ Atualizado | 2025-12-25 |
| ARCHITECTURE_CHALLENGES.md | ✅ Atualizado | 2025-12-25 |
| meta/index.md | ✅ Novo | 2026-01-14 |
| meta/intent.md | ✅ Novo | 2026-01-14 |
| meta/stack.md | ✅ Novo | 2026-01-14 |
| meta/failures.md | ✅ Novo | 2026-01-14 |

---

**Última Atualização Geral**: 2026-01-14
**Versão da Documentação**: 1.1.0
**Baseado em**: Análise de código real + input da equipe
