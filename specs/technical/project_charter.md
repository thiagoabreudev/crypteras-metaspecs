---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "technical"
tags: ['technical', 'project_charter']
---

# Carta do Projeto - Crypteras Trading System

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir escopo, objetivos e stakeholders do projeto para alinhamento inicial.

**Constraints** (limites obrigatórios):
- Escopo claro e limitado (evitar scope creep)
- Objetivos mensuráveis (SMART goals)
- Identificar stakeholders e responsabilidades
- Timeline realista baseado em capacidade do time

**Non-Goals** (o que NÃO fazer):
- Criar projeto waterfall rígido (aceitar mudanças)
- Definir tudo antecipadamente (agilidade)
- Comprometer com deadlines impossíveis
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Charter do projeto e objetivos
:::

## 1. Visão do Projeto

### Declaração de Visão
O Crypteras Trading System é uma **plataforma SaaS de trading automatizado de criptomoedas** que democratiza o acesso a estratégias de trading avançadas através de inteligência artificial e automação. O sistema permite que traders de todos os níveis criem, gerenciem e monitorem bots de trading que operam 24/7 em múltiplas exchanges, com gestão de risco integrada e análise técnica baseada em IA.

### Proposta de Valor
- **Automação Inteligente**: Bots especializados (SmartBot, CandleBot, DCABot) para diferentes estratégias
- **IA Integrada**: Framework AGNO com agentes especializados para análise de mercado e validação de risco
- **Multi-Exchange**: Suporte a Mercado Bitcoin (produção) e Binance (teste)
- **SaaS Escalável**: Modelo de assinatura (FREE/PRO/MAX) com limites configuráveis
- **Gestão de Risco**: Trailing stop, stop-loss multinível, circuit breaker automático

### Contexto Histórico
Este projeto é uma **reescrita completa** de um sistema legado desenvolvido em NestJS/TypeScript que operava com sucesso no Mercado Bitcoin. A v2.0 preservou as estratégias de trading validadas enquanto adicionou:
- Inteligência artificial para análise de mercado
- Arquitetura escalável (Clean Architecture)
- Suporte multi-exchange
- Cache inteligente (60-80% redução de API calls)
- Sistema de assinatura SaaS

---

## 2. Objetivos e Critérios de Sucesso

### Objetivos do Negócio

#### Objetivo 1: Produto SaaS Funcional
**Meta**: Sistema operacional com 3 planos de assinatura gerando receita recorrente
**Critérios de Sucesso**:
- ✅ Plano FREE (1 contexto) disponível
- ✅ Plano PRO (3 contextos, R$ 19,90/mês) com integração Mercado Pago
- ✅ Plano MAX (ilimitado, R$ 97/mês) com features avançadas
- ✅ Sistema de upgrade/downgrade funcional
- 🎯 **KPI**: 100+ usuários ativos até Q2 2025

#### Objetivo 2: Bots de Trading Estáveis
**Meta**: SmartBots, CandleBots e DCABots operando 24/7 sem intervenção manual
**Critérios de Sucesso**:
- ✅ SmartBot: Compra/venda competitiva com trailing stop funcional
- ✅ CandleBot: Análise técnica (10 indicadores) gerando sinais precisos
- ✅ DCABot: Dollar Cost Averaging com execução pontual
- 🎯 **KPI**: 95%+ uptime dos workflows
- 🎯 **KPI**: < 5% taxa de falha em ordens

#### Objetivo 3: Escalabilidade Horizontal
**Meta**: Sistema suportando crescimento de usuários sem degradação
**Critérios de Sucesso**:
- ✅ Celery workers escaláveis horizontalmente
- ✅ Redis como broker distribuído
- ✅ MongoDB com índices otimizados
- 🎯 **KPI**: Suportar 1000+ bots ativos simultaneamente
- 🎯 **KPI**: Latência < 2s para operações críticas

### Objetivos Técnicos

#### Objetivo 1: Arquitetura Limpa e Manutenível
**Meta**: Código organizado seguindo Clean Architecture e DDD
**Critérios de Sucesso**:
- ✅ Separação clara de camadas (Domain/Application/Infrastructure)
- ✅ Domain entities com lógica de negócio encapsulada
- ✅ Repositórios abstratos para testabilidade
- 🔧 **Débito Técnico**: Código não-PEP8 (a ser corrigido)
- 🔧 **Débito Técnico**: Resquícios de `TradingConfig` deprecated

#### Objetivo 2: Multi-Exchange Agnóstico
**Meta**: Adicionar novas exchanges com mínimo esforço
**Critérios de Sucesso**:
- ✅ Adapter Pattern implementado (`BaseExchange`)
- ✅ Mercado Bitcoin adapter completo (7 timeframes)
- ⚠️ Binance adapter funcional mas em teste (16 timeframes)
- 🔧 **Débito Técnico**: Workflows acoplados ao MB (deveria ser genérico)
- 🎯 **Meta**: Refatorar para exchange-agnostic até Q1 2025

#### Objetivo 3: Cobertura de Testes Adequada
**Meta**: Confiança para refatorar e adicionar features
**Critérios de Sucesso**:
- ✅ Testes unitários para domain entities
- ✅ Testes de integração para workflows críticos
- 🔧 **Gap**: Workflows insuficientemente testados (~60-70% cobertura)
- 🎯 **Meta**: 80%+ cobertura em workflows até Q2 2025

---

## 3. Escopo do Projeto

### No Escopo ✅

#### Features Core
1. **Sistema de Bots**
   - SmartBot (manual/scheduled/continuous)
   - CandleBot (análise técnica com 10 indicadores)
   - DCABot (dollar cost averaging)
   - Gestão de risco (trailing stop, stop-loss, circuit breaker)

2. **Sistema de Assinatura**
   - Planos FREE/PRO/MAX
   - Integração Mercado Pago/Stripe
   - Validação de limites por plano
   - Sistema de upgrades

3. **Multi-Exchange**
   - Mercado Bitcoin (produção)
   - Binance (teste)
   - Adapter Pattern para novas exchanges

4. **Infraestrutura**
   - Celery + Redis (task queue distribuída)
   - MongoDB (persistência)
   - Docker Swarm (deploy)
   - Grafana (monitoring de logs)

5. **API REST**
   - Autenticação JWT
   - Endpoints para bots (CRUD + controle)
   - Endpoints de dashboard (métricas, performance)
   - Endpoints de assinatura

6. **Frontend**
   - Nuxt.js 3 dashboard
   - Wizards de criação de bots
   - Métricas em tempo real
   - Sistema de onboarding (intro.js)

#### Workflows Automatizados
- `sync_orders` (42s): Sincronização de ordens das exchanges
- `smart_bot_purchase` (63s): Compra automática SmartBots
- `smart_bot_sales` (67s): Venda automática SmartBots
- `smart_bot_check_situation` (33s): Monitoramento de risco
- `candle_bot_analysis` (120s): Análise técnica CandleBots
- `candle_bot_risk_management` (60s): Take profit/stop-loss
- `dca_execution` (60s): Execuções programadas DCA
- `dca_risk_management` (30s): Monitoramento DCA

### Fora de Escopo ❌

#### Features Adiadas (Próximos Passos)
1. **Arbitragem**: Não será desenvolvido a curto prazo
2. **Backtesting Avançado**: Apenas análise manual por enquanto
3. **Dashboard Avançado**: Funcionalidades básicas suficientes
4. **Notificações em Tempo Real**: Apenas email/dashboard
5. **Mobile App**: Apenas web responsiva

#### Exchanges Não Priorizadas
- Kraken, Coinbase, Bybit, etc.: Apenas MB + Binance no roadmap

#### Integrações Não Planejadas
- TradingView signals
- Copy trading social
- API pública para terceiros

### Limites de Escopo

#### O Que o Sistema FAZ
- ✅ Executa estratégias de trading **automaticamente**
- ✅ Valida risco **antes** de executar ordens
- ✅ Monitora posições **continuamente**
- ✅ Sincroniza ordens **regularmente** (42s)
- ✅ Suporta **múltiplos usuários** com isolamento de dados

#### O Que o Sistema NÃO FAZ
- ❌ **Garantias de lucro**: Trading envolve risco
- ❌ **Execução instantânea**: Workflows tem intervalos (30s-120s)
- ❌ **Suporte a todas exchanges**: Apenas MB + Binance
- ❌ **Análise fundamental**: Apenas análise técnica
- ❌ **High-frequency trading**: Não é HFT

---

## 4. Stakeholders

### Stakeholders Principais

#### Equipe de Desenvolvimento
- **Papel**: Implementação, manutenção, evolução do sistema
- **Interesse**: Código limpo, arquitetura escalável, debugging eficiente
- **Prioridades**:
  1. Refatorar débito técnico (PEP8, TradingConfig deprecated)
  2. Corrigir problemas de sync_orders
  3. Tornar workflows exchange-agnostic

#### Usuários Finais (Traders)
- **Papel**: Operam bots de trading
- **Interesse**: Sistema estável, lucro consistente, facilidade de uso
- **Prioridades**:
  1. SmartBots funcionando corretamente (prioridade #1)
  2. CandleBots com sinais precisos
  3. DCABots executando pontualmente

#### Equipe de Produto/Negócio
- **Papel**: Define roadmap, prioriza features
- **Interesse**: Crescimento de usuários, receita recorrente
- **Prioridades**:
  1. Estabilizar plataforma existente
  2. Corrigir bots antes de novas features
  3. Melhorar onboarding de novos usuários

### Stakeholders Secundários

#### Exchanges (Mercado Bitcoin, Binance)
- **Papel**: Fornecem APIs de trading
- **Interesse**: Uso responsável de APIs, respeito a rate limits
- **Impacto**: Podem bloquear acesso se houver abuso

#### Provedores de Infraestrutura (DigitalOcean)
- **Papel**: Hospedam aplicação (Docker Swarm)
- **Interesse**: Uso dentro de limites de recursos
- **Impacto**: Custos de infraestrutura escalam com usuários

---

## 5. Restrições Técnicas

### Restrições de Arquitetura

#### 1. Clean Architecture Mandatória
**Motivação**: Complexidade do domínio + mudanças frequentes
**Impacto**:
- ✅ Domain layer **sem** dependências de frameworks
- ✅ Entities com lógica de negócio encapsulada
- ✅ Repositories abstratos (interfaces no domain, implementação na infra)
- ⚠️ Curva de aprendizado para novos devs

#### 2. Celery + Redis Obrigatório
**Motivação**: Escalabilidade horizontal (múltiplos workers)
**Impacto**:
- ✅ APScheduler **descontinuado** (CRY-72)
- ✅ Tasks distribuídas entre workers
- ✅ RedBeat para scheduling persistente
- ⚠️ Complexidade adicional vs APScheduler simples

#### 3. MongoDB como Banco Principal
**Motivação**: Facilidade para modelar estrutura + mudanças frequentes
**Impacto**:
- ✅ Schema flexível para rápida iteração
- ✅ Queries com índices otimizados
- ⚠️ Sem transações ACID complexas (uso de version locks)

### Restrições de Deployment

#### 1. Docker Swarm (Servidor Único - DigitalOcean)
**Motivação**: Simplicidade vs Kubernetes
**Impacto**:
- ✅ Deploy via `build-deploy.sh`
- ✅ Orquestração básica de containers
- ⚠️ Sem multi-node redundancy (single point of failure)
- 🎯 **Consideração futura**: Migrar para Kubernetes se necessário

#### 2. Intervalos Co-Primos para Workflows
**Motivação**: Evitar problemas de concorrência
**Impacto**:
- ✅ Workflows com intervalos 33s, 42s, 60s, 63s, 67s, 120s
- ✅ Reduz colisões simultâneas de tasks
- ⚠️ Não são "números redondos" (pode confundir)

### Restrições de Integrações

#### 1. Exchange APIs com Rate Limits
**Mercado Bitcoin**:
- ⚠️ Rate limits não documentados oficialmente
- ✅ Implementado retry com backoff exponencial
- ✅ Cache de 60s para balances, 30s para orderbook

**Binance**:
- ⚠️ 1200 requests/minute (hard limit)
- ✅ Cliente gerencia automaticamente
- ⚠️ Testnet tem limites mais restritivos

#### 2. OpenAI API (AGNO Framework)
**Restrições**:
- ⚠️ 400 requests/minute (configurável)
- ⚠️ Max 500 tokens/request para otimizar custos
- ✅ Histórico limitado a 3 turnos (TEAM_HISTORY_RUNS=3)
- ✅ Modelos baratos: gpt-4o-mini, gpt-3.5-turbo

---

## 6. Premissas e Dependências

### Premissas

1. **Exchanges Continuarão Disponíveis**
   - Mercado Bitcoin API v4 estável
   - Binance API v3 estável
   - Rate limits não serão reduzidos drasticamente

2. **OpenAI API Acessível**
   - GPT-4o-mini disponível e acessível
   - Custos por token permanecem viáveis
   - Latência < 2s para análise de mercado

3. **MongoDB Performance Adequada**
   - Queries com índices < 100ms
   - Write throughput suficiente para sync_orders (42s)
   - Sem necessidade de sharding a curto prazo

4. **Usuários Aceitam Intervalos de Workflow**
   - Traders entendem que não é HFT
   - 30s-120s de latência é aceitável
   - Paper trading disponível para testes

### Dependências Críticas

#### Dependências Externas
1. **Mercado Bitcoin API v4**: Sistema não funciona sem acesso
2. **OpenAI API**: Análise de mercado e validação de risco dependem
3. **Mercado Pago/Stripe**: Assinaturas PRO/MAX dependem
4. **DigitalOcean**: Deploy em produção depende

#### Dependências Internas
1. **Sync Orders Workflow**: Todos bots dependem de ordens sincronizadas
2. **Exchange Factory**: Todos workflows dependem de criar exchanges
3. **User Repository**: Autenticação e isolamento de dados
4. **Redis**: Celery não funciona sem broker

---

## 7. Riscos e Mitigações

### Riscos Técnicos

#### Risco 1: Problemas com sync_orders (ALTO)
**Descrição**: Ordens não sincronizadas corretamente + atualização incorreta de dados dos bots
**Impacto**:
- Bots operam com informações desatualizadas
- total_invested, total_blocked incorretos
- Decisões de compra/venda erradas
**Mitigação**:
- ✅ Implementado version lock em updates
- ✅ Retry com backoff exponencial
- 🔧 **TODO**: Aumentar cobertura de testes em workflows (prioridade)
- 🔧 **TODO**: Adicionar health check específico para sync_orders

#### Risco 2: Workflows Acoplados ao Mercado Bitcoin (MÉDIO)
**Descrição**: Código assume formato MB, dificulta adicionar exchanges
**Impacto**:
- Binance não funciona corretamente em produção
- Tempo de desenvolvimento alto para novas exchanges
**Mitigação**:
- ✅ Adapter Pattern implementado (BaseExchange)
- ✅ OrderNormalizer para uniformizar formatos
- 🔧 **TODO**: Refatorar workflows para usar apenas interface genérica
- 🔧 **TODO**: Testes de integração com mock exchanges

#### Risco 3: Débito Técnico Crescente (MÉDIO)
**Descrição**: Código não-PEP8 + resquícios de TradingConfig deprecated
**Impacto**:
- Dificuldade para novos desenvolvedores
- Bugs por inconsistências
- Refatorações arriscadas
**Mitigação**:
- 🔧 **TODO**: Configurar linter (black, flake8) em CI/CD
- 🔧 **TODO**: Refatorar gradualmente removendo TradingConfig
- 🔧 **TODO**: Documentar padrões em CLAUDE.meta.md

### Riscos de Negócio

#### Risco 1: Exchanges Alterarem APIs (ALTO)
**Descrição**: MB ou Binance depreciam endpoints ou mudam formatos
**Impacto**: Sistema para de funcionar
**Mitigação**:
- ✅ Versioning de APIs (v4, v3)
- ✅ Adapter Pattern facilita migração
- 🎯 **Plano**: Monitorar changelogs de exchanges

#### Risco 2: Custos de OpenAI Crescerem (MÉDIO)
**Descrição**: Aumento de preço por token ou remoção de gpt-4o-mini
**Impacto**: Margem de lucro reduzida
**Mitigação**:
- ✅ Cache agressivo (60-80% redução)
- ✅ Histórico limitado (3 turnos)
- 🎯 **Plano B**: Modelos open-source (Llama, Mistral)

---

## 8. Cronograma de Alto Nível

### Q4 2024 (Atual)
- ✅ Planos FREE/PRO/MAX lançados (CRY-56)
- ✅ SmartBot unificado (CRY-71)
- ✅ Celery migration completa (CRY-72)
- 🔧 **Em Progresso**: Correção de bots (prioridade)

### Q1 2025 (Próximo)
- 🎯 Refatorar workflows para exchange-agnostic
- 🎯 Remover resquícios de TradingConfig
- 🎯 Aplicar PEP8 em todo codebase
- 🎯 Aumentar cobertura de testes (workflows > 80%)
- 🎯 Estabilizar Binance em produção

### Q2 2025 (Futuro)
- 🔮 Melhorias em sync_orders
- 🔮 Dashboard avançado com métricas
- 🔮 Sistema de alertas em tempo real
- 🔮 100+ usuários ativos

---

## 9. Métricas de Sucesso

### Métricas Técnicas

| Métrica | Baseline Atual | Meta Q1 2025 | Meta Q2 2025 |
|---------|----------------|--------------|--------------|
| **Uptime Workflows** | 90% | 95% | 98% |
| **Taxa de Falha Ordens** | 8-10% | 5% | 3% |
| **Latência API (p95)** | 3s | 2s | 1.5s |
| **Cobertura de Testes** | 60-70% | 75% | 80% |
| **Incidentes sync_orders/semana** | 3-5 | 1-2 | < 1 |

### Métricas de Negócio

| Métrica | Baseline Atual | Meta Q1 2025 | Meta Q2 2025 |
|---------|----------------|--------------|--------------|
| **Usuários Ativos** | 10-20 | 50 | 100 |
| **Bots Ativos** | 30-50 | 150 | 300 |
| **Assinaturas PRO** | 5 | 15 | 30 |
| **Assinaturas MAX** | 1 | 5 | 10 |
| **MRR (Monthly Recurring Revenue)** | R$ 200 | R$ 800 | R$ 1.500 |

---

## 10. Aprovações e Atualizações

### Histórico de Versões

| Versão | Data | Autor | Mudanças |
|--------|------|-------|----------|
| 1.0 | 2024-12-24 | AI + Equipe | Versão inicial baseada em análise de código |

### Próxima Revisão
**Data Programada**: 2025-03-01 (Q1 review)
**Responsável**: Equipe de Produto + Desenvolvimento
**Foco**: Avaliar progresso em refatorações e correção de bots

---

**Documento aprovado para uso como referência técnica e alinhamento de equipe.**
