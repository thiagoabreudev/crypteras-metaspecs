---
adr_number: "006"
title: "Framework AGNO para Agentes de IA"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['technical', 'decision', 'framework', 'agno', 'architecture']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# ADR-006: Framework AGNO para Agentes de IA

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Adicionar inteligência de mercado aos bots através de agentes AGNO especializados, mantendo decisões humanas como override final.

**Constraints** (limites obrigatórios):
- Agentes AGNO são consultivos (fornecem recomendações, não executam ordens)
- Decisão final de executar ordem sempre requer aprovação de workflow (não pode ser 100% autônomo)
- Tools devem ter caching obrigatório (evitar requests desnecessários a exchanges)
- Prompts dos agentes devem ser versionados e auditáveis
- Custos de tokens devem ser monitorados (alertar se exceder $X/dia)
- Agentes não podem modificar estado de bots diretamente (apenas via use cases)
- Teams devem ter hierarquia clara (Market Intelligence → Trading Execution)

**Non-Goals** (o que NÃO fazer):
- Permitir agentes executarem ordens sem validação humana/workflow
- Usar AGNO para tarefas que não requerem IA (ex: cálculo de preço médio ponderado)
- Criar agentes genéricos que fazem tudo (manter especialização)
- Compartilhar mesmo agente entre múltiplos usuários simultaneamente (isolamento obrigatório)
- Armazenar API keys de LLM no código (usar environment variables)
- Treinar ou fine-tunar modelos (usar apenas modelos pré-treinados via API)
- Criar dependência crítica de IA (sistema deve funcionar se AGNO estiver offline)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Adoção do framework Agno para IA
:::

## Status
✅ **Aceito** (Implementado desde v2.0)

## Data
2024-09-01 (retroativo - decisão no início do projeto v2.0)

## Contexto

### Problema

Sistema legado (NestJS) tinha regras fixas baseadas em percentuais:
```typescript
// Sistema antigo - sem IA
if (variation >= 2.0) {
  // Trailing profit
  bot.basePrice = currentPrice;
}

if (variation <= -5.0) {
  // Hard stop-loss
  sellAtMarket();
}
```

**Limitações**:
- ❌ Sem análise de contexto de mercado
- ❌ Decisões puramente mecânicas
- ❌ Não considera volatilidade, volume, spread
- ❌ Regras fixas para todos cenários

**Necessidade**: Adicionar **inteligência** para:
1. Analisar mercado antes de operar
2. Validar se é bom momento para comprar/vender
3. Ajustar parâmetros dinamicamente (adaptive sell)
4. Detectar anomalias (pump & dump, flash crash)

### Por Que AGNO?

**AGNO** é um framework Python para criar agentes de IA especializados que colaboram em "teams".

**Alternativas Consideradas**:
- LangChain: Muito genérico, overhead alto
- AutoGPT: Focado em autonomous agents (não nosso use case)
- Custom: Muito trabalho implementar do zero

**Razões para AGNO**:
- ✅ Especializado em trading/finance
- ✅ Suporte nativo para tools (get_balance, get_orderbook, etc.)
- ✅ Teams coordenados (Market Intelligence + Trading Execution)
- ✅ Cache de contexto (reduz tokens)
- ✅ Fácil integração com OpenAI

## Decisão

Adotar **AGNO Framework** com arquitetura de Agentes + Teams + Tools.

### Arquitetura Implementada

```
┌─────────────────────────────────────────────────────────────┐
│                     AGNO TEAMS                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │   Market Intelligence Team                           │   │
│  │   - Análise de mercado                               │   │
│  │   - Detecção de tendências                           │   │
│  │   - Relatórios de oportunidades                      │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │   Trading Execution Team                             │   │
│  │   - Validação de risco                               │   │
│  │   - Execução otimizada                               │   │
│  │   - Monitoramento de posições                        │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    AGNO AGENTS                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  MarketAnalysisAgent (gpt-4o-mini)                   │   │
│  │  - Analisa volatilidade, tendência, liquidez        │   │
│  │  - Tools: get_ticker, get_candles, get_orderbook    │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  RiskManagementAgent (gpt-4o-mini)                   │   │
│  │  - Valida trades antes de executar                  │   │
│  │  - Calcula exposição de risco                        │   │
│  │  - Tools: get_balance, get_position                  │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  ExecutionAgent (gpt-4o-mini)                        │   │
│  │  - Otimiza execução de ordens                        │   │
│  │  - Escolhe melhor momento dentro de janela          │   │
│  │  - Tools: create_order, cancel_order                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                     AGNO TOOLS (Cached)                     │
│  - get_balance(asset) [TTL: 60s]                            │
│  - get_orderbook(symbol, depth) [TTL: 30s]                  │
│  - get_ticker(symbol) [TTL: 30s]                            │
│  - get_candles(symbol, timeframe, limit) [TTL: 300s]        │
│  - create_order(symbol, side, type, qty, price) [no cache]  │
│  - cancel_order(symbol, order_id) [no cache]                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│              Exchange APIs (MB, Binance)                     │
└─────────────────────────────────────────────────────────────┘
```

### Agentes Especializados

#### 1. MarketAnalysisAgent
**Responsabilidade**: Analisar condições de mercado

**Tools Disponíveis**:
- `get_ticker`: Preço atual, variação 24h, volume
- `get_orderbook`: Spread, liquidez, support/resistance
- `get_candles`: Análise de tendência (últimas 50 velas)

**Prompt Otimizado** (CRY-6):
```
Você é um analista de mercado de criptomoedas especializado.

MISSÃO: Analisar o mercado {symbol} e fornecer recomendação BUY/SELL/HOLD.

CONTEXTO:
- Usuário opera SmartBot com trailing stop de 3%
- Objetivo: identificar boas oportunidades de compra/venda

ANÁLISE OBRIGATÓRIA:
1. Volatilidade: Baixa (<2%), Média (2-5%), Alta (>5%)
2. Tendência: Bullish, Bearish, Sideways
3. Liquidez: Suficiente? (spread < 0.5%)
4. Volume: Normal, Acima da média, Abaixo da média

FORMATO DE RESPOSTA (OBRIGATÓRIO):
{
  "recommendation": "BUY" | "SELL" | "HOLD",
  "confidence": "LOW" | "MEDIUM" | "HIGH",
  "volatility": "LOW" | "MEDIUM" | "HIGH",
  "trend": "BULLISH" | "BEARISH" | "SIDEWAYS",
  "reason": "1-2 frases explicando"
}
```

#### 2. RiskManagementAgent
**Responsabilidade**: Validar trades antes de executar

**Tools Disponíveis**:
- `get_balance`: Saldo disponível (BRL e criptos)
- `get_position`: Posições abertas (bots ativos)

**Validações**:
- Saldo suficiente para compra?
- Exposição total < limite?
- Ordem não viola stop-loss?

#### 3. ExecutionAgent
**Responsabilidade**: Executar ordens otimizadamente

**Tools Disponíveis**:
- `create_order`: Colocar ordem na exchange
- `cancel_order`: Cancelar ordem existente
- `get_orderbook`: Verificar preço competitivo

**Estratégia**:
- Se spread < 0.3%: ordem LIMIT (maker fee 0.3%)
- Se spread > 0.5%: ordem MARKET (garantir execução)

### Teams Coordenados

#### Market Intelligence Team
```python
# src/teams/market_intelligence_team.py
from agno import Team

market_intelligence_team = Team(
    name="Market Intelligence",
    agents=[
        MarketAnalysisAgent,
        RiskManagementAgent
    ],
    instructions="""
    Você é um time de inteligência de mercado.
    Analise o mercado e forneça recomendações de compra/venda.

    FLUXO:
    1. MarketAnalysisAgent analisa mercado
    2. RiskManagementAgent valida risco
    3. Time retorna recomendação consolidada
    """,
    model="gpt-4o-mini",  # Modelo otimizado
    max_loops=1,  # Sem loops infinitos
    context_window=3  # CRY-6: apenas 3 turnos de histórico
)
```

#### Trading Execution Team
```python
# src/teams/trading_execution_team.py
trading_execution_team = Team(
    name="Trading Execution",
    agents=[
        ExecutionAgent,
        RiskManagementAgent
    ],
    instructions="""
    Você é um time de execução de trades.

    FLUXO:
    1. RiskManagementAgent valida se pode executar
    2. ExecutionAgent coloca ordem otimizada
    3. Time confirma execução
    """,
    model="gpt-4o-mini",
    max_loops=1,
    context_window=3
)
```

### Cache de Tools (CRY-6 Otimização)

```python
# src/tools/cached_tools.py
from functools import lru_cache
from datetime import datetime, timedelta

class CachedExchangeTools:
    def __init__(self, exchange: BaseExchange):
        self._cache = {}

    async def get_ticker(self, symbol: str):
        cache_key = f"ticker:{symbol}"
        if self._is_cached(cache_key, ttl=30):  # 30s TTL
            return self._cache[cache_key]['data']

        ticker = await self.exchange.get_ticker(symbol)
        self._cache[cache_key] = {
            'data': ticker,
            'timestamp': datetime.utcnow()
        }
        return ticker

    async def get_balance(self, asset: str):
        cache_key = f"balance:{asset}"
        if self._is_cached(cache_key, ttl=60):  # 60s TTL
            return self._cache[cache_key]['data']

        balance = await self.exchange.get_balance(asset)
        self._cache[cache_key] = {
            'data': balance,
            'timestamp': datetime.utcnow()
        }
        return balance

    # create_order, cancel_order: SEM cache (sempre executar)
```

## Consequências

### Positivas ✅

1. **Análise Inteligente de Mercado**
   - ✅ IA detecta volatilidade, tendência, liquidez
   - ✅ Recomendações contextuais (não apenas % fixos)

2. **Validação de Risco Automática**
   - ✅ IA valida saldo, exposição, stop-loss antes de operar
   - ✅ Reduz erros humanos

3. **Adaptive Sell (CRY-16)**
   - ✅ IA ajusta target de venda baseado em volatilidade
   - ✅ Mercado volátil: take profit maior
   - ✅ Mercado estável: take profit conservador

4. **Cache Reduz Custos (CRY-6)**
   - ✅ 60-80% redução em chamadas à OpenAI API
   - ✅ Custos: ~R$ 50/mês vs R$ 250/mês sem cache

5. **Testabilidade**
   - ✅ Fácil mockar tools para testes
   - ✅ Agentes testáveis isoladamente

### Negativas ⚠️

1. **Latência Adicional**
   - ⚠️ Chamada OpenAI: +500ms-2s por análise
   - ⚠️ Total: 3s-5s para decisão (vs 50ms sem IA)
   - **Mitigação**: Aceitável para trading não-HFT

2. **Custos de API**
   - ⚠️ OpenAI API: ~R$ 50/mês (com cache)
   - ⚠️ Sem cache: ~R$ 250/mês
   - **Justificativa**: Valor agregado compensa

3. **Dependência de OpenAI**
   - ⚠️ Se OpenAI cair, sistema perde inteligência
   - **Mitigação**: Fallback para regras fixas (legado)

4. **Não Determinístico**
   - ⚠️ IA pode dar respostas diferentes para mesma situação
   - ⚠️ Difícil debugar decisões
   - **Mitigação**: Logs detalhados de análise

## Alternativas Consideradas

### Alternativa 1: Regras Fixas (Status Quo Legado) (Rejeitada)
**Razão para Rejeitar**: Sem contexto de mercado, decisões sub-ótimas

### Alternativa 2: Custom ML Models (Rejeitada)
**Prós**:
- Controle total
- Sem dependência OpenAI

**Contras**:
- Tempo de desenvolvimento: meses
- Necessidade de dados de treino
- Expertise em ML necessária

**Razão para Rejeitar**: Time-to-market vs LLMs prontos

### Alternativa 3: LangChain (Rejeitada)
**Razão para Rejeitar**: AGNO mais especializado para finance

## Implementação

**Evidências no Código**:
- [`src/agents/market_analysis_agent.py`](../../../crypteras-improved/backend/src/agents/market_analysis_agent.py)
- [`src/teams/market_intelligence_team.py`](../../../crypteras-improved/backend/src/teams/market_intelligence_team.py)
- [`src/tools/`](../../../crypteras-improved/backend/src/tools/)

## Métricas de Sucesso

### Custos OpenAI (Pós-Cache CRY-6)
- ✅ Antes: R$ 250/mês
- ✅ Depois: R$ 50/mês (80% redução)

### Qualidade de Decisões
- ⚠️ Difícil medir objetivamente
- ✅ Usuários reportam menos "compras no topo"
- ✅ Adaptive sell funciona bem em mercados voláteis

## Lições Aprendidas

1. **Cache é Crítico**: Sem cache, custos inviáveis
2. **Prompts Precisam Ser Específicos**: Vagos = respostas inconsistentes
3. **Context Window Small**: 3 turnos suficiente (CRY-6)

## Referências

- [AGNO Framework Docs](https://docs.agno.dev)
- [OpenAI API Pricing](https://openai.com/pricing)
- [Issue CRY-6: Token Limit Optimization](link)
- [Issue CRY-16: Adaptive Sell](link)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Implementação**: v2.0 (Setembro 2024)
**Status Atual**: ✅ Em uso, funcionando bem. Custos controlados com cache.
