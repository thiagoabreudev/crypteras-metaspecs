---
spec_version: "1.2.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "medium"
category: "feature"
tags: ['agent', 'agno', 'feature', 'technical', 'automation']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: "1.1.0"
---

# Framework Agno - Agentes de IA Internos

:::version_info
**Versão**: 1.2.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.2.0**:
- Removida referência direta a ADR técnico (alinhamento com hierarquia de contexto)
- Adicionada nota sobre base técnica no Intent

**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
:::

:::intent
**Goal**: Fornecer assistente conversacional de IA para ajudar usuários com configuração de bots e dúvidas sobre trading.

**Technical Foundation**: Sistema utiliza framework de agentes de IA para implementar assistente conversacional com contexto do usuário. Detalhes de implementação técnica estão documentados em specs/technical/adr/.

**Constraints** (limites obrigatórios):
- Agente DEVE ter contexto do usuário (bots ativos, saldo, ordens)
- Respostas DEVEM ser em português brasileiro
- NUNCA dar conselhos financeiros ou garantias de lucro
- Incluir disclaimer em cada resposta sobre riscos de trading
- Limitar histórico de conversa a últimas 10 mensagens (controle de custos)
- Rate limiting: FREE 10 msgs/dia, PRO 100 msgs/dia, MAX ilimitado

**Non-Goals** (o que NÃO fazer):
- Permitir agente executar ordens automaticamente sem confirmação
- Implementar chat em tempo real (WebSockets) no MVP
- Adicionar suporte a áudio/voz
- Criar personalidades customizáveis do agente
- Implementar fine-tuning ou treinamento custom do modelo
- Suportar múltiplos idiomas além de português
:::

## Visão Geral

**Agno** é um framework open-source multi-agente (https://docs.agno.com) usado internamente pelo Crypteras para construir agentes de IA que automatizam análises e tomadas de decisão nos workflows de trading.

**O QUE É**: Framework técnico para criar agentes, teams e workflows autônomos
**O QUE NÃO É**: Chatbot conversacional para usuários finais

**Status**: ✅ Em Produção (uso interno nos workflows)
**Exposição**: Interna apenas - Não há interface de usuário
**Prioridade**: ALTA - Core da automação inteligente

---

## Propósito e Valor

### Para o Sistema Crypteras

O Agno framework permite que o Crypteras:

1. **Automatize Decisões Complexas**
   - Validação de sinais de trading antes da execução
   - Análise de condições de mercado em tempo real
   - Otimização de parâmetros baseada em volatilidade
   - Gestão de risco automatizada

2. **Coordene Múltiplos Agentes Especializados**
   - Teams de agentes trabalhando em conjunto
   - Cada agente é especialista em uma área (análise, risco, execução)
   - Coordenação via Team Leaders

3. **Reduza Custos com LLMs**
   - Otimização agressiva de tokens (50-500 tokens/request)
   - Cache inteligente de respostas
   - Modelos mais baratos (gpt-4o-mini, gpt-3.5-turbo)
   - Sem histórico de conversas (economia de contexto)

### Para o Negócio

- **Decisões Mais Inteligentes**: Validação por IA antes de executar ordens
- **Redução de Risco**: Análise automática de condições adversas
- **Escalabilidade**: Mesmos agentes servem todos os usuários
- **Diferencial Técnico**: Uso avançado de multi-agent systems

---

## Arquitetura Real Implementada

### Agentes Especializados

O Crypteras possui **6 agentes especializados** construídos com Agno:

#### 1. **SignalValidatorAgent** (CRY-18)
- **Função**: Valida sinais de Candle Bots antes da execução
- **Modelo**: `gpt-3.5-turbo` (análises técnicas)
- **Tokens**: Máx 500 tokens/request
- **Inputs**: Sinais de MA, RSI, Bollinger Bands, contexto de mercado
- **Outputs**: APROVAR/REJEITAR com justificativa
- **Uso**: Executado automaticamente nos workflows de Candle Bots

**Exemplo de validação**:
```python
signal_data = {
    'signal': 'BUY',
    'confidence': 75,
    'individual_signals': {
        'ma_crossover': 'BUY',
        'rsi': 'NEUTRAL',
        'bollinger': 'BUY'
    },
    'indicators': {
        'rsi_last': 35,  # oversold
        'current_price': 195000
    },
    'market_context': {
        'spread_pct': 0.5,
        'liquidity': 'high'
    }
}

result = validator.validate_signal(signal_data)
# Output: {'approved': True, 'reasoning': 'RSI oversold + BB lower...'}
```

#### 2. **MarketAnalysisAgent**
- **Função**: Analisa condições de mercado para otimização de parâmetros
- **Modelo**: `gpt-4o-mini` (análise de mercado)
- **Tokens**: Máx 150 tokens/request
- **Tools**: `get_ticker_price_raw`, `get_orderbook_raw`
- **Outputs**: JSON com volatilidade, tendência, liquidez, recomendações
- **Uso**: Workflow de otimização de configuração (AI Config Optimizer)

**Exemplo de análise**:
```python
# Agent analisa mercado e retorna JSON
{
    "volatility": "medium",
    "volatility_percent": 3.5,
    "trend": "bullish",
    "liquidity": "high",
    "spread_percent": 0.8,
    "recommendation": "moderate",
    "reasoning": "Volatilidade média com tendência de alta"
}
```

#### 3. **ConfigOptimizerAgent**
- **Função**: Otimiza parâmetros de trading baseado em análise de mercado
- **Modelo**: `gpt-4o-mini`
- **Inputs**: Análise de mercado (do MarketAnalysisAgent)
- **Outputs**: JSON com parâmetros otimizados (trailing_stop, stop_loss, etc)
- **Regras**: Adapta parâmetros baseado em volatilidade e tendência

**Exemplo de otimização**:
```python
# Volatilidade ALTA → stops mais largos
{
    "trailing_stop_percentage": 5.0,  # 5% (mercado volátil)
    "bot_minimum_loss_percentage": 3.0,
    "bot_maximum_loss_percentage": 6.0,
    "circuit_breaker_max_daily_loss_pct": 15.0,
    "justification": "Volatilidade alta (5.2%) requer stops largos..."
}
```

#### 4. **RiskManagementAgent**
- **Função**: Valida operações antes da execução (gestão de risco)
- **Modelo**: `gpt-4o-mini`
- **Tokens**: Máx 100 tokens/request (validação direta)
- **Tools**: `get_cached_balances`, `check_balance_for_order`
- **Outputs**: APROVADO/REJEITADO com justificativa
- **Uso**: Teams de execução de trading

#### 5. **MonitoringAgent**
- **Função**: Monitora performance, riscos e oportunidades
- **Modelo**: `gpt-3.5-turbo` (análises mais baratas)
- **Tools**: `get_cached_balances`, `analyze_orderbook`
- **Outputs**: Relatórios de status, alertas, oportunidades
- **Uso**: Teams de inteligência de mercado e monitoramento de risco

#### 6. **ExecutionAgent** ❌ DEPRECATED (CRY-82)
- **Status**: Descontinuado
- **Motivo**: Substituído por workflows diretos com ExchangeCredentials
- **Código**: Mantido por compatibilidade mas não usado

---

### Teams (Coordenação de Agentes)

O Crypteras usa **Agno Teams** para coordenar múltiplos agentes:

#### 1. **Market Intelligence Team**
- **Membros**: MarketAnalysisAgent, MonitoringAgent, SignalValidatorAgent
- **Leader Model**: `gpt-4o-mini` (200 tokens)
- **Função**: Análise de mercado e validação de sinais
- **Delegação**: Seletiva (apenas 1 agente mais relevante por tarefa)
- **Histórico**: Ativado (para confirmações)

#### 2. **Trading Execution Team**
- **Membros**: RiskManagementAgent, ExecutionAgent (deprecated)
- **Leader Model**: `gpt-4o-mini` (250 tokens)
- **Função**: Validação de risco antes de execução
- **Delegação**: Sequencial (Risk valida → Execution executa)

#### 3. **Risk Monitoring Team**
- **Membros**: RiskManagementAgent, MonitoringAgent
- **Leader Model**: `gpt-3.5-turbo` (250 tokens)
- **Função**: Monitoramento contínuo de exposição e alertas
- **Delegação**: Seletiva

---

## Ferramentas (Tools) Disponíveis

Os agentes têm acesso a ferramentas customizadas via decorator `@tool`:

### Exchange Tools
- `get_ticker_price_raw` - Preço atual de um símbolo
- `get_orderbook_raw` - Orderbook completo

### Balance Tools
- `get_cached_balances` - Saldos em cache (TTL 60s)
- `get_formatted_balances` - Saldos formatados
- `check_balance_for_order` - Valida saldo para ordem

### Orderbook Tools
- `get_cached_orderbook` - Orderbook em cache (TTL 30s)
- `analyze_orderbook` - Análise de spread e liquidez

### Analysis Tools
- Ferramentas de cálculo e validação de sinais

---

## Casos de Uso Reais

### 1. Validação de Sinal de Candle Bot (CRY-18)

**Workflow**:
1. Candle Bot gera sinal de COMPRA (baseado em MA, RSI, Bollinger)
2. `SignalValidatorAgent` recebe sinal com todos os indicadores
3. Agente analisa consistência entre indicadores
4. Retorna APROVADO/REJEITADO com justificativa
5. Workflow só executa compra se aprovado

**Benefício**: Reduz falsos positivos e melhora qualidade das entradas

---

### 2. Otimização de Parâmetros Baseada em Mercado

**Workflow**:
1. `MarketAnalysisAgent` analisa condições atuais (volatilidade, tendência, liquidez)
2. Retorna JSON com análise de mercado
3. `ConfigOptimizerAgent` recebe análise e otimiza parâmetros
4. Sugere trailing_stop, stop_loss, circuit breaker adaptados
5. Sistema pode aplicar sugestões ou alertar usuário

**Benefício**: Parâmetros se adaptam automaticamente às condições de mercado

---

### 3. Coordenação via Teams

**Workflow**:
1. Usuário (ou sistema) envia pergunta ao Market Intelligence Team
2. Team Leader decide qual agente é mais relevante
3. Agente especializado processa a tarefa
4. Leader consolida resposta
5. Retorno ao sistema

**Benefício**: Especialização + coordenação = decisões mais precisas

---

## Configuração e Otimizações

### Economia de Tokens (Crítico para Custos)

```python
# Configuração típica de um agente
agent = Agent(
    model=OpenAIChat(
        id="gpt-4o-mini",  # Modelo barato
        max_tokens=150     # Limite estrito
    ),
    instructions=[
        "Resposta ULTRA-CONCISA - máximo 2 frases",
        "Insights diretos com 1-2 dados técnicos"
    ],
    add_history_to_context=False,  # AGNO v2: Sem histórico
    num_history_runs=0,             # Zero runs anteriores
    markdown=True,
    show_tool_calls=True            # Debug
)
```

### Limites Operacionais

**Configuração atual (via .env)**:
- `PRIMARY_MODEL=gpt-4o-mini` - Coordenação e análises
- `ANALYSIS_MODEL=gpt-3.5-turbo` - Análises técnicas baratas
- `MAX_TOKENS_PER_REQUEST=500` - Limite global
- `MAX_REQUESTS_PER_MINUTE=400` - Rate limiting

**Cache para reduzir chamadas**:
- Saldos: TTL 60s
- Orderbook: TTL 30s
- Análises: TTL 300s (5 min)

---

## Métricas de Sucesso

### KPIs Técnicos
- **Taxa de Aprovação de Sinais**: % de sinais aprovados pelo SignalValidator (meta: 60-70%)
- **Latência de Validação**: Tempo médio para validar um sinal (meta: < 3s)
- **Custo por Validação**: Custo em tokens/dólares por decisão (meta: < $0.01)
- **Taxa de Erro**: % de decisões incorretas pelo agente (meta: < 5%)

### KPIs de Negócio
- **Redução de Falsos Positivos**: % de sinais ruins rejeitados (meta: 30-40%)
- **Melhoria de Win Rate**: Comparação bots com/sem validação IA (meta: +10-15%)
- **Economia de Custos**: Redução de ordens perdedoras evitadas (meta: mensurável)

---

## Limitações e Transparência

### O que os Agentes NÃO fazem

❌ **Não interagem com usuários finais** - Sem interface de chat
❌ **Não executam ordens diretamente** - Apenas analisam e recomendam
❌ **Não preveem o futuro** - Análise técnica ≠ previsão garantida
❌ **Não substituem análise humana** - São assistentes, não decisores únicos

### O que os Agentes FAZEM

✅ **Validam sinais antes da execução** - Reduzem risco
✅ **Analisam condições de mercado** - Contexto em tempo real
✅ **Otimizam parâmetros** - Adaptação automática
✅ **Coordenam decisões complexas** - Via Teams
✅ **Reduzem custos operacionais** - Automação inteligente

---

## Segurança e Privacidade

### Dados Acessados pelos Agentes

✅ **Agentes TÊM acesso a**:
- Orderbooks públicos (dados de mercado)
- Preços atuais de criptomoedas
- Saldos do usuário (via tools autorizadas)
- Sinais técnicos (MA, RSI, Bollinger)
- Contexto de mercado (spread, liquidez)

❌ **Agentes NUNCA acessam**:
- Credenciais de exchange (criptografadas)
- Senhas de usuários
- Dados de pagamento (Stripe/Mercado Pago)
- Dados pessoais sensíveis

### Isolation entre Usuários

- Cada execução de agente é **stateless** (sem memória entre runs)
- Agentes não compartilham contexto entre usuários
- Cache é isolado por usuário/contexto
- Logs são rastreáveis mas privados

---

## Roadmap Futuro

### Melhorias Planejadas (Q1-Q2 2025)

- [ ] **Backtesting assistido por IA**: Agentes testam estratégias em dados históricos
- [ ] **Auto-tuning de parâmetros**: Ajuste automático baseado em performance
- [ ] **Análise de sentimento**: Incorporar notícias/Twitter via agentes
- [ ] **Agentes proativos**: Alertas automáticos de oportunidades/riscos
- [ ] **Multi-modelo support**: Testar Claude, Gemini além de OpenAI

### Visão de Longo Prazo

- [ ] **Agentes totalmente autônomos**: Criar e ajustar bots sem intervenção
- [ ] **Aprendizado contínuo**: Melhorar sugestões baseado em resultados reais
- [ ] **Interface de chat para usuários**: Expor Teams via API REST (futuro)

---

## Diferença: Agno Framework vs Chatbot

| Aspecto | Agno Framework (Atual) | Chatbot para Usuários (Futuro?) |
|---------|------------------------|----------------------------------|
| **Público** | Interno (workflows) | Usuários finais |
| **Interface** | Código Python | UI de chat (frontend) |
| **Uso** | Automação de decisões | Assistência conversacional |
| **Exposição** | Zero (interno) | API REST + WebSocket |
| **Histórico** | Stateless (sem memória) | Conversas persistentes |
| **Custo** | Otimizado (50-500 tokens) | Maior (conversas longas) |
| **Implementação** | ✅ Produção | ❌ Não implementado |

---

## Referências Técnicas

### Documentação Oficial Agno
- **Framework**: https://docs.agno.com
- **Agents**: https://docs.agno.com/concepts/agents/introduction
- **Teams**: https://docs.agno.com/concepts/teams/introduction
- **Tools**: https://docs.agno.com/concepts/tools/introduction

### Código Crypteras
- **Agentes**: `/backend/src/agents/`
- **Teams**: `/backend/src/teams/factory.py`
- **Tools**: `/backend/src/tools/`
- **Referência Agno**: `/backend/docs/agno-framework-reference.md`

---

**Última Atualização**: 2025-12-25
**Versão**: 1.1.0
**Revisão**: Corrigida para refletir implementação real (não alucinações)
