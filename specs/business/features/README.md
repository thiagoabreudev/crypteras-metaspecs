---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "high"
category: "feature"
tags: ['feature', 'business', 'readme']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Catálogo de Funcionalidades - Crypteras

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Primeira versão versionada de README.md
:::

## Índice Geral

Esta pasta contém a documentação detalhada de todas as funcionalidades do sistema Crypteras.

---

## Funcionalidades Documentadas

### 1. [Autenticação e Autorização Multi-User](authentication-and-authorization.md)
**Tickets**: CRY-12, CRY-41
**Status**: ✅ Em Produção

Sistema completo de autenticação JWT com refresh token rotation, criptografia AES-256 de credenciais de exchange e compliance LGPD.

**Principais Recursos**:
- Registro e login com JWT
- Refresh token rotation (7 dias)
- Criptografia de credenciais (Mercado Bitcoin + Binance)
- Auditoria completa (IP, User-Agent, timestamps)
- Múltiplos níveis de acesso (público, autenticado, ownership, backoffice)

---

### 2. [Sistema de Assinaturas](subscription-plans.md)
**Tickets**: CRY-41, CRY-56
**Status**: ✅ Em Produção

Modelo freemium SaaS com 3 planos (FREE, PRO, MAX), integração Stripe e Mercado Pago.

**Planos**:
| Plano | Preço | Contextos | Recursos |
|-------|-------|-----------|----------|
| FREE | R$ 0/mês | 1 | Dashboard, SmartBots básicos |
| PRO | R$ 19,90/mês | 3 | Candle/DCA Bots, Indicadores técnicos |
| MAX | R$ 97/mês | Ilimitado | Indicadores avançados, Suporte prioritário |

**Integrações**:
- Stripe (cartão internacional)
- Mercado Pago (PIX, boleto, cartão)
- Webhooks com validação de signature
- Billing automático mensal

**Ver também**: [Payment Webhooks](payment-webhooks.md) para detalhes técnicos de sincronização

---

### 2.1 [Contextos de Trading](trading-contexts.md)
**Tickets**: CRY-40
**Status**: ✅ Em Produção

Sistema que organiza e limita capacidade de trading através de contextos isolados - base do modelo freemium.

**O que é um Contexto**:
- Combinação única: {Moeda} + {Fiat} + {Exchange}
- Exemplos: BTC-BRL-MercadoBitcoin, ETH-USDT-Binance
- Cada contexto opera independentemente

**Limites por Plano** (Monetização):
- FREE: 1 contexto (R$ 0/mês)
- PRO: 3 contextos (R$ 19,90/mês)
- MAX: Ilimitado (R$ 97/mês)

**3 Validações Obrigatórias**:
1. Usuário tem credenciais de exchange configuradas
2. Assinatura está ativa (status=ACTIVE)
3. Quantidade atual < limite do plano

**Relação com Bots**:
- 1 contexto → múltiplos bots (SmartBot, CandleBot, DCABot)
- Limite de bots é por usuário, não por contexto
- Exemplo: PRO com 3 contextos + 4 bots distribuídos

**Isolamento**:
- Cada contexto tem posições, P&L e histórico separados
- BTC-BRL não interfere com ETH-BRL
- Permite estratégias diferentes por mercado

**Valor de Negócio**:
- Conversão FREE → PRO: > 12% (motivado por limite)
- Lock-in: Histórico acumulado em cada contexto
- Diferenciação: Competitors cobram por bots; Crypteras por contextos

---

### 2.2 [Payment Webhooks](payment-webhooks.md)
**Tickets**: CRY-41, CRY-56
**Status**: ✅ Em Produção

Sistema de processamento assíncrono de eventos de pagamento que sincroniza automaticamente status de assinaturas entre provedores (Stripe/Mercado Pago) e plataforma.

**Eventos Stripe**:
- `checkout.session.completed`: Ativa assinatura PRO após pagamento
- `customer.subscription.updated`: Sincroniza status (active, past_due, canceled)
- `customer.subscription.deleted`: Downgrade para FREE ao cancelar

**Eventos Mercado Pago**:
- `payment.approved`: Ativa PRO após pagamento PIX/boleto
- `subscription.authorized`: Assinatura recorrente autorizada
- `subscription.cancelled`: Cancelamento imediato (sem grace period)

**Segurança**:
- ✅ Validação HMAC-SHA256 (Stripe)
- ✅ Validação indireta via API (Mercado Pago)
- ✅ Proteção anti-replay (timestamp validation)
- ✅ Idempotência total (webhooks duplicados não causam efeitos)

**Funcionalidades**:
- Ativação instantânea (< 5 segundos após pagamento)
- Retry automático de pagamentos falhos (Stripe)
- Grace period ao cancelar (mantém acesso até fim do período - Stripe)
- Reconciliação automática de status
- Logs completos para auditoria

**Casos de Uso**:
- Upgrade FREE → PRO via cartão/PIX
- Renovação automática mensal
- Recuperação de pagamento falho
- Cancelamento com/sem grace period

---

### 3. [SmartBots (Trading Automatizado)](smart-bots.md)
**Tickets**: CRY-71, CRY-13, CRY-16, CRY-73
**Status**: ✅ Em Produção

Bots de trading com 3 modos de operação e 8 mecanismos de gestão de risco.

**Ver também**: [Circuit Breaker](circuit-breaker.md) para proteção automática contra perdas

**Modos de Operação**:
- **CONTINUOUS**: Intervalos fixos (ex: compra a cada 2 min)
- **SCHEDULED**: Agendamentos cron (ex: todo dia às 9h)
- **MANUAL**: Apenas comandos manuais

**Gestão de Risco**:
1. Take Profit (lucro automático)
2. Stop Loss (soft + hard)
3. Trailing Stop (protege lucros)
4. Adaptive Sell (ajusta por volatilidade)
5. Circuit Breaker (pausa após falhas)
6. Sanity Check (valida variações anormais)
7. Emergency Sell (venda de emergência)
8. Auto-cancel de ordens distantes

**Gestão de Fundos**:
- `total_invested`: Valor em ordens executadas
- `total_blocked`: Valor em ordens pending
- `total_proceeds`: Receita de vendas
- `realized_profit`: Lucro acumulado

---

### 3.1 [Circuit Breaker (Proteção Automática)](circuit-breaker.md)
**Tickets**: CRY-13, CRY-71
**Status**: ✅ Em Produção

Sistema de segurança que pausa bots automaticamente quando detecta condições perigosas de trading.

**Duplo Gatilho**:
- **Perdas Consecutivas**: Para após N trades com prejuízo seguidos (padrão: 5)
- **Limite Diário**: Para quando perda acumulada atinge threshold (padrão: 10%)

**Configuração Personalizada**:
- Conservador: 3 perdas OU 5% diário
- Moderado: 5 perdas OU 10% diário (padrão)
- Agressivo: 8 perdas OU 15% diário

**Funcionamento**:
```
Trade 1: -2% (1/5 falhas consecutivas) → Continua
Trade 2: -1.5% (2/5) → Continua
Trade 3: +1% (lucro) → Contador reseta para 0/5 ✅
Trade 4: -2% (1/5) → Recomeça contagem
Trade 5: -1.8% (2/5) → Continua
Trade 6: -2.5% (3/5) → Continua
Trade 7: -1.2% (4/5) → Continua
Trade 8: -2% (5/5) → ⛔ CIRCUIT BREAKER ACIONADO
→ Bot pausado automaticamente
→ Usuário notificado
→ Retoma após investigar e corrigir
```

**Benefícios**:
- Protege contra perdas ilimitadas (spiral de prejuízos)
- Economiza R$ 200-500 por acionamento (vs sem proteção)
- Força análise estratégica antes de retomar
- Disponível para SmartBots (opcional, configurável)

**Recuperação**:
- Usuário investiga causa (spread alto, volatilidade, configuração)
- Ajusta parâmetros se necessário
- Clica "Retomar" no dashboard
- Próximo trade bem-sucedido reseta contador

---

### 4. [Candle Bots (Análise Técnica)](candle-bots.md)
**Tickets**: CRY-69, CRY-20
**Status**: ✅ Em Produção

Bots baseados em análise técnica profissional de candles com 10 indicadores técnicos.

**Indicadores Padrão** (todas exchanges):
- RSI (Relative Strength Index)
- MA (Moving Average: SMA, EMA)
- Bollinger Bands
- Volume
- Volatility

**Indicadores Avançados** (apenas Binance):
- VWAP (Volume Weighted Average Price)
- MFI (Money Flow Index)
- Volume Profile
- ATR (Average True Range)
- OBV (On Balance Volume)

**Funcionalidades**:
- 16 timeframes suportados (Binance), 7 timeframes (Mercado Bitcoin)
- Geração automática de sinais (BUY/SELL/HOLD)
- Confidence score (LOW/MEDIUM/HIGH)
- Auto-execução configurável
- Take profit e stop loss automáticos
- Trailing stop

**Workflows**:
- `candle_bots_analysis` (60s): Analisa candles e gera sinais
- `candle_bots_risk_management` (30s): Monitora take profit/stop loss

---

### 5. [DCA Bots (Dollar Cost Averaging)](dca-bots.md)
**Tickets**: CRY-63, CRY-66
**Status**: ✅ Em Produção

Bots de acumulação periódica automatizada com média de preço ponderada - investimento disciplinado sem emoção.

**Conceito**:
- Compra valor fixo em intervalos regulares
- Reduz impacto de volatilidade de curto prazo
- Calcula preço médio ponderado automaticamente
- Take profit e stop loss em cima da média

**Configuração**:
```json
{
  "purchase_amount": "100.00",     // Compra R$ 100 por vez
  "frequency": "24h",              // Frequência: 1h, 6h, 24h, 7d, 30d
  "allocated_balance": "3000.00",  // Budget total
  "take_profit_percent": "10.0",   // Vende tudo com 10% de lucro
  "stop_loss_percent": "5.0",      // Vende tudo com 5% de prejuízo
  "max_cycles": 30,                // Máximo 30 compras (ou null)
  "auto_restart": true             // Reinicia após take profit
}
```

**Exemplo de Acumulação**:
```
Ciclo 1: R$ 100 @ R$ 200k = 0.0005 BTC → avg_price = R$ 200k
Ciclo 2: R$ 100 @ R$ 210k = 0.000476 BTC → avg_price = R$ 204.918
Ciclo 3: R$ 100 @ R$ 190k = 0.000526 BTC → avg_price = R$ 199.333
```

**Histórico de Execuções** (CRY-66):
- Cada compra persistida como `DCAExecution`
- Tracking completo: cycle, amount, quantity, price, timestamp
- Endpoint `/executions` lista histórico

**Workflows**:
- `dca_execution` (60s): Executa compras periódicas
- `dca_risk_management` (30s): Monitora take profit/stop loss

---

### 6. [AGNO Chat (Agente de IA)](agno-chat-agent.md)
**Status**: ✅ Em Produção

Assistente conversacional de IA que auxilia usuários na configuração de bots, análise de mercado e troubleshooting.

**Funcionalidades**:
- Onboarding guiado (criação de primeiro bot passo a passo)
- Interpretação de sinais de Candle Bots
- Troubleshooting inteligente (diagnóstico de problemas)
- Educação contextual (explica conceitos de trading)
- Análise de performance (relatórios automáticos)
- Ações automatizadas (criar/pausar bots com permissão)

**Limitações por Plano**:
- FREE: 10 mensagens/dia
- PRO: 100 mensagens/dia
- MAX: Ilimitado + sugestões proativas

---

### 7. [Gestão de Credenciais de Exchange](exchange-credentials.md)
**Status**: ✅ Em Produção

Sistema seguro para configuração de API keys de exchanges que permite bots executarem ordens automaticamente.

**Segurança**:
- Criptografia AES-256-CBC em repouso
- HTTPS (TLS 1.3) em trânsito
- Nunca exibe credenciais completas (apenas últimos 4 dígitos)
- Validação automática ao salvar
- Re-validação periódica (diária)

**Exchanges Suportadas**:
- Mercado Bitcoin (API v4)
- Binance (API v3 + Testnet opcional)

**Permissões Recomendadas**:
- ✅ Ler saldo
- ✅ Executar ordens (trading)
- ❌ Sacar fundos (NUNCA habilite)

---

### 8. [Dashboard e Analytics](dashboard-analytics.md)
**Status**: ✅ Em Produção

Dashboard centralizado que consolida toda atividade de trading em uma única tela.

**Componentes**:
- Resumo executivo (saldo total, P&L, win rate)
- Gráfico de performance (comparação com Bitcoin)
- Saldos por exchange (Mercado Bitcoin + Binance)
- Performance por estratégia (Candle vs DCA vs Smart)
- Bots ativos (status, lucro, última ação)
- Ordens recentes (histórico de execuções)
- Alertas e notificações (circuit breaker, take profit próximo)

**Métricas Avançadas** (MAX):
- Sharpe Ratio
- Max Drawdown
- Profit Factor
- Average Hold Time

**Limitações por Plano**:
- FREE: 30 dias histórico
- PRO: 90 dias + comparação com mercado
- MAX: Ilimitado + métricas avançadas + exportação PDF

---

### 9. [Backoffice / Admin Panel](backoffice-admin.md)
**Status**: ✅ Em Produção
**Acesso**: Apenas equipe Crypteras (@crypteras.tech)

Sistema administrativo completo para gestão da plataforma SaaS - permite equipe interna monitorar usuários, assinaturas, bots e operações sem acesso direto ao banco de dados.

**Funcionalidades**:
- Autenticação separada (cookie HttpOnly, 8h expiration)
- Listagem de usuários com paginação (filtros, ordenação)
- Detalhes completos por usuário (assinatura, bots, P&L, credenciais)
- Ações administrativas (block/unblock, reset password)
- Auditoria completa de acessos (LGPD compliance)

**RBAC (Roles)**:
- SUPER_ADMIN: Todos os acessos + gestão de admins
- ADMIN: Visualização e ações administrativas
- SUPPORT: Apenas leitura (read-only)

**Segurança**:
- ✅ Domínio whitelist (@crypteras.tech)
- ✅ Proteção CSRF (SameSite=Strict)
- ✅ Rate limiting (login 5/15min, listagem 60/min)
- ✅ Logs auditados (retenção 1 ano)
- ❌ NUNCA exibe: senhas, API keys, tokens

**Casos de Uso**:
- Suporte rápido (contexto completo do usuário em 2 min)
- Detecção de padrões (problemas sistêmicos)
- Prevenção de fraude (contas temporárias, abuso)

**Roadmap**:
- Dashboard com métricas visuais
- Filtros avançados e exportação CSV
- Integração com Intercom/Zendesk

---

### 10. Sistema de Ordens e Sincronização
**Tickets**: CRY-54, CRY-39, CRY-18
**Status**: ✅ Em Produção

Sincronização automática de ordens com exchanges e rastreabilidade completa.

**Componentes**:

#### Order Creation Context (CRY-39)
Captura contexto ANTES de criar ordem:
```json
{
  "order_id": "abc123",
  "strategy_source": "purchase",  // purchase, sales, candle, dca
  "timestamp": "2025-01-15T10:00:00Z",
  "parameters": {
    "quantity": "0.001",
    "price": "200000.00"
  },
  "technical_indicators": {  // Se candle strategy
    "rsi": 45.2,
    "ma_short": 198500,
    "ma_long": 201000
  },
  "market_conditions": {
    "best_bid": "199999.00",
    "best_ask": "200001.00",
    "spread_pct": "0.001"
  }
}
```

**Benefício**: Auditoria completa - "Por que essa ordem foi criada?"

#### Order Strategy Map (CRY-18)
Mapping `order_id` → `strategy_source` para tracking de P&L por estratégia:
- "manual": Ordens criadas manualmente pelo usuário
- "smart": SmartBots (purchase/sales workflows)
- "candle": Candle Bots
- "dca": DCA Bots

#### Persistência Local (CRY-54)
- Todas as ordens salvas em MongoDB (além da exchange)
- Permite queries offline
- Cache local para performance

**Workflow de Sincronização**:

`sync_orders_job` (42s):
```python
# Para cada usuário
for user in users:
    # Busca ordens pending na exchange
    exchange_orders = await exchange.get_orders(user, status="pending")

    # Para cada ordem
    for order in exchange_orders:
        local_order = await order_repo.get_by_id(order.id)

        # Detecta mudanças de status
        if order.status == "FILLED" and local_order.status == "PENDING":
            # Ordem executada
            if order.strategy == "smart":
                bot = await bot_repo.get_by_id(order.bot_id)
                bot.convert_blocked_to_invested(order.filled_value)
                bot.record_purchase_success()
                await bot_repo.update(bot)

        elif order.status == "CANCELED":
            # Ordem cancelada
            if order.strategy == "smart":
                bot.release_blocked_funds(order.value)

        elif order.status == "PARTIALLY_FILLED":
            # Ordem parcialmente executada
            bot.convert_blocked_to_invested(order.partial_value)
            # Resto permanece em blocked

        # Atualiza local
        local_order.status = order.status
        local_order.filled_quantity = order.filled_quantity
        local_order.avg_fill_price = order.avg_fill_price
        await order_repo.update(local_order)
```

**Retry com Optimistic Locking**:
- Cada update incrementa `version`
- Se update falhar (version mismatch): retry 3x com backoff exponencial
- Previne race conditions

**Outros Workflows**:
- `sync_withdrawals_job` (600s): Sincroniza saques
- `sync_strategy_balances_job` (3600s): Sincroniza saldos por estratégia

---

### 11. Dashboard e Analytics (Visão Consolidada)
**Tickets**: CRY-10
**Status**: ✅ Em Produção

Dashboard em tempo real com métricas agregadas.

**Endpoints**:
- `GET /api/dashboard/summary`: Métricas agregadas
- `GET /api/dashboard/performance`: Séries temporais (gráficos)
- `GET /api/dashboard/orders`: Lista de ordens recentes
- `GET /api/dashboard/balances`: Saldos por moeda
- `GET /api/dashboard/health`: Health check do sistema

**Métricas Principais**:

```json
{
  "summary": {
    "total_invested": "5000.00",
    "total_in_positions": "3200.00",
    "realized_pnl": "450.00",
    "unrealized_pnl": "120.00",
    "win_rate": "65.5",
    "active_bots": 5,
    "pending_orders": 3
  },
  "performance_by_strategy": {
    "smart": {
      "total_invested": "2000.00",
      "realized_profit": "150.00",
      "win_rate": "60.0"
    },
    "candle": {
      "total_trades": 50,
      "win_rate": "70.0",
      "max_drawdown": "8.5"
    },
    "dca": {
      "average_price": "198500.00",
      "unrealized_profit": "120.00"
    }
  }
}
```

**Cálculos**:

**P&L Não Realizado**:
```python
unrealized_pnl = 0
for position in open_positions:
    current_value = position.quantity * current_price
    cost_basis = position.quantity * position.avg_price
    unrealized_pnl += (current_value - cost_basis)
```

**Win Rate**:
```python
win_rate = (winning_trades / total_trades) * 100
```

**Agregações MongoDB**:
- Pipelines de agregação para performance
- Caching de queries pesadas (TTL 60s)

---

### 12. Legal Compliance (LGPD)
**Tickets**: CRY-32
**Status**: ✅ Em Produção

Sistema de consentimentos legais obrigatórios em conformidade com LGPD.

**Consentimentos**:
1. **terms_accepted**: Termos de Uso (obrigatório)
2. **privacy_accepted**: Política de Privacidade (obrigatório)
3. **marketing_accepted**: Comunicações de marketing (opcional)

**Endpoints**:
- `POST /api/legal/consents`: Atualizar consentimentos
- `GET /api/legal/consents`: Obter consentimentos atuais
- `GET /api/legal/terms`: Texto dos termos de uso
- `GET /api/legal/privacy`: Texto da política de privacidade

**Auditoria**:
```json
{
  "legal_consents": {
    "terms_accepted": true,
    "privacy_accepted": true,
    "marketing_accepted": false,
    "timestamp": "2025-01-15T10:00:00Z",
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0..."
  }
}
```

**Regras**:
- Backend SEMPRE valida aceites (segurança)
- Registro requer terms + privacy = true
- IP extraction: Cloudflare-aware (CF-Connecting-IP, X-Forwarded-For)
- Retenção: 5 anos (requisito legal)
- Usuário pode revogar marketing_accepted a qualquer momento

**Direitos do Titular (LGPD)**:
- ✅ Acesso aos dados: `GET /api/auth/me`
- ✅ Atualização: `PUT /api/auth/me`
- ✅ Exclusão: `DELETE /api/users/me` (soft delete)
- ✅ Portabilidade: Export JSON completo

---

### 13. Suporte Multi-Exchange
**Tickets**: CRY-20, CRY-46
**Status**: ✅ Em Produção

Suporte a múltiplas exchanges com adapter pattern extensível.

**Exchanges Suportadas**:

#### Mercado Bitcoin (API v4)
- **Timeframes**: 1m, 5m, 15m, 30m, 1h, 1d, 1M (7 intervalos)
- **Indicadores**: RSI, MA, BB, Volume, Volatility (padrão)
- **Símbolos**: Com hífen (BTC-BRL, ETH-BRL)
- **Rate Limit**: 100 requests/min

#### Binance (API v3)
- **Timeframes**: 1s, 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M (16 intervalos)
- **Indicadores**: Todos (padrão + avançados: VWAP, MFI, Volume Profile, ATR, OBV)
- **Símbolos**: Sem hífen (BTCBRL, ETHUSDT)
- **Testnet**: Disponível (configurável via `binance_use_testnet`)
- **Rate Limit**: 1200 requests/min (weight-based)

**Adapter Pattern**:
```python
class IExchange(Protocol):
    async def get_ticker(self, symbol: str) -> Ticker
    async def get_orderbook(self, symbol: str) -> Orderbook
    async def get_candles(self, symbol: str, interval: str, limit: int) -> List[Candle]
    async def create_order(self, symbol: str, side: OrderSide, type: OrderType, quantity: Decimal, price: Decimal) -> Order
    async def cancel_order(self, order_id: str, symbol: str) -> bool
    async def get_balance(self, asset: str) -> Balance
    async def get_orders(self, symbol: str, status: str) -> List[Order]
```

**Normalização**:
- Todas as exchanges retornam objetos normalizados (`NormalizedOrder`, `NormalizedTicker`)
- Conversão de formatos específicos para formato padrão interno
- 100% backward compatible

**Endpoints**:
- `GET /api/exchanges/symbols`: Listar pares disponíveis
- `GET /api/exchanges/{exchange}/pairs`: Pares por exchange

**Rate Limiting**:
- Retry com backoff exponencial (3 tentativas)
- Circuit breaker: pausa requests após N falhas consecutivas

---

### 14. Workflows Automatizados

Sistema de workflows com Celery Beat e intervalos coprimos.

**Workflows Implementados**:

| Workflow | Intervalo | Função |
|----------|-----------|--------|
| `sync_orders` | 42s | Sincroniza ordens com exchanges |
| `sync_withdrawals` | 600s | Sincroniza saques |
| `sync_strategy_balances` | 3600s | Sincroniza saldos por estratégia |
| `anomaly_check` | 60s | Detecta anomalias (spreads, volume) |
| `smart_bot_purchase` | 63s | Compras SmartBots CONTINUOUS |
| `smart_bot_sales` | 67s | Vendas SmartBots CONTINUOUS |
| `smart_bot_check_situation` | 33s | Monitora riscos SmartBots |
| `candle_bots_analysis` | 60s | Análise técnica Candle Bots |
| `candle_bots_risk_management` | 30s | Take profit/stop loss Candle |
| `dca_execution` | 60s | Compras DCA Bots |
| `dca_risk_management` | 30s | Take profit/stop loss DCA |
| `candle_analysis` | 300s | Gera sinais por contexto |

**Intervalos Coprimos (CRY-71)**:
- Workflows usam intervalos coprimos (33, 42, 63, 67) para minimizar race conditions
- Evita que múltiplos workflows processem o mesmo bot simultaneamente
- Reduz conflitos de optimistic locking

**Scheduler**:
- **Produção**: Celery Beat (CRY-72)
- **Fallback/Dev**: APScheduler

**Configuração (.env)**:
```env
CELERY_BROKER_URL=redis://redis:6379/0
CELERY_RESULT_BACKEND=redis://redis:6379/1
```

**Monitoramento**:
- Logs estruturados (Structlog)
- Métricas de duração
- Taxa de sucesso/falha
- Alertas em falhas consecutivas

---

## Arquitetura Geral

### Camadas (Clean Architecture)

```
┌─────────────────────────────────────────┐
│           API LAYER (FastAPI)           │
│  - Routes (endpoints HTTP)              │
│  - Schemas (Pydantic DTOs)              │
│  - Dependencies (DI)                    │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│      APPLICATION LAYER (Use Cases)      │
│  - Use Cases (orquestração)             │
│  - Services (lógica aplicação)          │
│  - Decorators (tracking, logging)       │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│       DOMAIN LAYER (Business Logic)     │
│  - Entities (regras de negócio)         │
│  - Value Objects (imutáveis)            │
│  - Domain Services                      │
│  - Repository Interfaces (protocolos)   │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│    INFRASTRUCTURE LAYER (Adapters)      │
│  - Repositories (MongoDB)               │
│  - Exchanges (MB, Binance)              │
│  - Payment Gateways (Stripe, MP)        │
│  - Schedulers (Celery, APScheduler)     │
│  - Alerting, Cache, etc                 │
└─────────────────────────────────────────┘
```

### Stack Tecnológica

- **Backend**: Python 3.13, FastAPI, Motor (MongoDB async)
- **Frontend**: Nuxt 3 (Vue.js), TailwindCSS
- **Banco de Dados**: MongoDB (async)
- **Cache/Queue**: Redis, Celery
- **IA**: OpenAI GPT (AGNO v2 framework)
- **Pagamentos**: Stripe, Mercado Pago
- **Exchanges**: Mercado Bitcoin API v4, Binance API v3
- **Deploy**: Docker Swarm, Traefik (reverse proxy), Portainer

---

## Roadmap Futuro

### Melhorias Planejadas (Q1-Q2 2025)

#### Autenticação
- [ ] 2FA (Two-Factor Authentication) - TOTP
- [ ] OAuth2 Social Login (Google, GitHub)
- [ ] Session Management (listar/revogar sessões)

#### Assinaturas
- [ ] Trial 7 dias (PRO grátis)
- [ ] Coupons/Desconto
- [ ] Plano Anual (desconto)
- [ ] Plano Enterprise (custom pricing)

#### Bots
- [ ] Grid Trading Bot
- [ ] Arbitrage Bot (multi-exchange)
- ] Backtesting completo (dados históricos)
- [ ] Paper trading mode (simulação)

#### Analytics
- [ ] Dashboard em tempo real (WebSockets)
- [ ] Relatórios exportáveis (PDF, Excel)
- [ ] Gráficos avançados (Recharts/ApexCharts)
- [ ] Alertas personalizáveis (email, Telegram)

#### Multi-Exchange
- [ ] Suporte a OKX, Kraken, KuCoin
- [ ] Unified orderbook (agregado de múltiplas exchanges)

---

## Métricas de Sucesso Gerais

### Produto
- **Uptime**: > 99.5%
- **Latência API**: < 200ms (p95)
- **Taxa de Erro**: < 0.1%
- **Conversão FREE → PRO**: > 10%
- **Churn Rate**: < 5%/mês

### Técnicas
- **Cobertura de Testes**: > 80%
- **Code Quality**: SonarQube score > 90
- **Security**: Zero CVEs críticas
- **Performance**: Workflows processam > 1000 bots/min

---

## Contato e Suporte

- **Documentação Técnica**: `docs/`
- **Issues**: GitHub Issues
- **Email Suporte**: suporte@crypteras.tech
- **Comunidade**: Discord (link no site)

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
