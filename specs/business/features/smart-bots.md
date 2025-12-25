---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'bots', 'smart']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# SmartBots (Bots de Trading Automatizado)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Implementar bots de trading automatizado com gestão avançada de risco e flexibilidade operacional (3 modos).

**Constraints** (limites obrigatórios):
- Stop-loss SEMPRE obrigatório (hard_stop_loss >= 5%, soft_stop_loss >= 2%)
- Circuit breaker DEVE ativar após 3 falhas consecutivas
- Trailing stop DEVE ser configurável (min 1%, max 10%)
- max_position_size DEVE ser respeitado rigorosamente (nunca ultrapassar)
- Adaptive sell DEVE validar condições de mercado antes de executar
- Bots DEVEM operar apenas quando status=ACTIVE (validação obrigatória)
- Intervals co-primos obrigatórios para CONTINUOUS (evitar picos de carga)

**Non-Goals** (o que NÃO fazer):
- Permitir bots sem stop-loss (proteção de capital é inegociável)
- Criar bots 100% autônomos sem approval (sempre requer validação de workflow)
- Suportar margin trading ou leverage no MVP (apenas spot trading)
- Implementar copy trading ou mirror trading
- Adicionar backtesting histórico no MVP (apenas forward testing)
- Permitir short selling (apenas long positions)
- Criar bots que operam em múltiplas exchanges simultaneamente (1 bot = 1 exchange)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Bots inteligentes com IA
:::

## Visão Geral

SmartBots são bots de trading automatizado com três modos de operação: CONTINUOUS (intervalos fixos), SCHEDULED (agendamentos cron) e MANUAL (apenas comandos manuais). Oferecem gestão avançada de risco com take profit, stop loss, trailing stop, circuit breaker e adaptive sell.

**Tickets Relacionados**: CRY-71, CRY-13, CRY-16, CRY-73
**Status**: ✅ Em Produção
**Prioridade**: ALTA

---

## Propósito e Valor

### Para o Usuário
- **Automação 24/7**: Trading contínuo sem necessidade de monitoramento manual
- **Gestão de Risco Profissional**: 5+ mecanismos de proteção de capital
- **Flexibilidade**: 3 modos de operação (continuous, scheduled, manual)
- **Transparência**: Rastreamento completo de investimentos, lucros e bloqueios

### Para o Negócio
- **Diferencial Competitivo**: Recursos avançados não encontrados em concorrentes
- **Retenção**: Bots automatizados mantêm usuários engajados
- **Escalabilidade**: Arquitetura suporta milhares de bots concorrentes

---

## Modos de Operação

### 1. CONTINUOUS (Intervalos Fixos)

Executa compras e vendas em intervalos de tempo fixos.

**Configuração**:
```json
{
  "operation_mode": "continuous",
  "purchase_interval_seconds": 120,  // Compra a cada 2 minutos
  "sale_interval_seconds": 120       // Venda a cada 2 minutos
}
```

**Uso**: Trading de alta frequência, arbitragem, bots agressivos

---

### 2. SCHEDULED (Agendamentos Cron)

Executa compras e vendas em horários específicos usando expressões cron.

**Configuração**:
```json
{
  "operation_mode": "scheduled",
  "purchase_schedule": "0 9 * * *",   // Todo dia às 9h
  "sale_schedule": "0 17 * * *"       // Todo dia às 17h
}
```

**Uso**: Dollar Cost Averaging diário, estratégias de timing de mercado

---

### 3. MANUAL

Bot não executa automaticamente - apenas via comandos manuais da API.

**Configuração**:
```json
{
  "operation_mode": "manual"
}
```

**Uso**: Controle total do usuário, backtesting, paper trading

---

## Gestão de Fundos (CRY-71 FIX #16)

### Modelo de Tracking

```python
# Fundos disponíveis
available_funds = max_position_size - total_invested - total_blocked

# Estados dos fundos
total_invested   # Valor em ordens FILLED (executadas)
total_blocked    # Valor em ordens PENDING (criadas mas não executadas)
total_proceeds   # Valor obtido em vendas
realized_profit  # Lucro acumulado (proceeds - cost_basis)
```

### Ciclo de Vida de Ordem

```
1. CRIAR ORDEM (PENDING)
   → block_funds(order_value)
   → total_blocked += order_value
   → available_funds -= order_value

2. ORDEM FILLED
   → convert_blocked_to_invested(filled_value)
   → total_blocked -= filled_value
   → total_invested += filled_value

3. VENDA FILLED
   → record_sale_success(proceeds, profit)
   → total_invested -= cost_basis
   → total_proceeds += proceeds
   → realized_profit += profit
   → available_funds += proceeds

4. ORDEM CANCELADA
   → release_blocked_funds(order_value)
   → total_blocked -= order_value
   → available_funds += order_value
```

**Exemplo Prático**:
```
Configuração: max_position_size = R$ 1000

1. Criação de ordem de compra R$ 100 (PENDING)
   - total_blocked = R$ 100
   - available_funds = R$ 900

2. Ordem executada (FILLED @ R$ 200.000/BTC = 0.0005 BTC)
   - total_blocked = R$ 0
   - total_invested = R$ 100
   - available_funds = R$ 900

3. Nova ordem R$ 200 (PENDING)
   - total_blocked = R$ 200
   - available_funds = R$ 700

4. Ordem executada (FILLED)
   - total_blocked = R$ 0
   - total_invested = R$ 300
   - available_funds = R$ 700

5. Venda de 0.0005 BTC @ R$ 210.000 = R$ 105
   - total_invested = R$ 200 (300 - 100)
   - total_proceeds = R$ 105
   - realized_profit = R$ 5
   - available_funds = R$ 805 (700 + 105)
```

---

## Mecanismos de Gestão de Risco

### 1. Take Profit

Vende automaticamente quando lucro atinge percentual configurado.

**Configuração**:
```json
{
  "take_profit_percentage": "5.0",  // Vende com 5% de lucro
  "base_price": "200000.00"         // Preço de referência
}
```

**Regra**:
```python
profit_pct = ((current_price - base_price) / base_price) * 100
if profit_pct >= take_profit_percentage:
    sell_all()
```

---

### 2. Stop Loss (Soft e Hard)

#### Soft Stop Loss
Vende com urgência (ordem LIMIT agressiva) quando prejuízo atinge threshold.

**Configuração**:
```json
{
  "soft_stop_loss_percentage": "2.0"  // Venda urgente com 2% de prejuízo
}
```

#### Hard Stop Loss
Vende IMEDIATAMENTE a mercado quando prejuízo atinge threshold crítico.

**Configuração**:
```json
{
  "hard_stop_loss_percentage": "5.0"  // Venda a mercado com 5% de prejuízo
}
```

**Regra**:
```python
loss_pct = ((base_price - current_price) / base_price) * 100

if loss_pct >= hard_stop_loss_percentage:
    sell_market()  # VENDA A MERCADO (garantia de execução)
elif loss_pct >= soft_stop_loss_percentage:
    sell_limit_aggressive()  # VENDA LIMIT urgente
```

---

### 3. Trailing Stop

Ajusta `base_price` automaticamente quando lucro >= threshold, protegendo lucros em tendências de alta.

**Configuração**:
```json
{
  "trailing_stop_percentage": "3.0"
}
```

**Regra**:
```python
profit_pct = ((current_price - base_price) / base_price) * 100

if profit_pct >= trailing_stop_percentage:
    # Ajusta base_price para cima (NUNCA desce)
    new_base_price = current_price * (1 - trailing_stop_percentage / 100)
    base_price = max(base_price, new_base_price)  # NUNCA reduz base_price
```

**Exemplo**:
```
1. Compra @ R$ 200.000 (base_price = 200k)
2. Preço sobe para R$ 210.000 (+5%)
   - Profit >= 3% → Ajusta base_price
   - Novo base_price = 210k * 0.97 = R$ 203.700

3. Preço sobe para R$ 220.000 (+10%)
   - Profit >= 3% → Ajusta base_price
   - Novo base_price = 220k * 0.97 = R$ 213.400

4. Preço cai para R$ 210.000
   - base_price = R$ 213.400 (não muda - nunca cai)
   - Loss = (213.4k - 210k) / 213.4k = 1.59%
   - Ainda acima do soft stop loss (2%)

5. Preço cai para R$ 208.000
   - Loss = 2.5% → Trigger SOFT STOP LOSS
   - Vende com urgência
   - Lucro realizado: R$ 8.000 (vs R$ 0 sem trailing stop)
```

---

### 4. Adaptive Sell (CRY-16)

Ajusta lucro mínimo para venda baseado em volatilidade do mercado.

**Configuração**:
```json
{
  "enable_adaptive_sell": true,
  "adaptive_sell_min_profit": "0.5",    // Mercado calmo: vende com 0.5%
  "adaptive_sell_max_profit": "5.0",    // Alta volatilidade: aguarda 5%
  "adaptive_sell_volatility_window": 20 // Usa últimos 20 candles
}
```

**Regra**:
```python
# 1. Calcula volatilidade (desvio padrão dos preços)
prices = get_last_n_candles(volatility_window)
volatility = std_dev(prices) / mean(prices)  # Normalizado

# 2. Ajusta lucro alvo baseado em volatilidade
profit_target = min_profit + (volatility * (max_profit - min_profit))

# 3. Vende apenas se lucro >= profit_target dinâmico
if current_profit >= profit_target:
    sell()
```

**Benefício**: Em mercados voláteis, aguarda lucros maiores. Em mercados calmos, realiza lucros pequenos rapidamente.

---

### 5. Circuit Breaker (CRY-13)

Pausa bot automaticamente após N falhas consecutivas ou perda diária excessiva.

**Configuração**:
```json
{
  "enable_circuit_breaker": true,
  "circuit_breaker_threshold": 5,              // Pausa após 5 falhas
  "circuit_breaker_max_daily_loss_pct": "10.0" // Pausa se perda >= 10%/dia
}
```

**Regras**:
```python
# Trigger por falhas consecutivas
if consecutive_failures >= circuit_breaker_threshold:
    pause_bot()
    send_alert("Circuit breaker ativado - falhas consecutivas")

# Trigger por perda diária
daily_loss_pct = (daily_loss / capital_inicial) * 100
if daily_loss_pct >= circuit_breaker_max_daily_loss_pct:
    pause_bot()
    send_alert("Circuit breaker ativado - perda diária excessiva")
```

**Reset**: Apenas manual via API (`POST /api/smart-bots/{id}/reset-circuit-breaker`)

---

### 6. Sanity Check (CRY-13)

Valida se preço atual vs último preço conhecido não ultrapassa threshold (previne flash crashes).

**Configuração**:
```json
{
  "enable_sanity_check": true,
  "sanity_check_max_price_change_pct": "10.0"  // Máximo 10% de variação
}
```

**Regra**:
```python
price_change_pct = abs(current_price - last_known_price) / last_known_price * 100

if price_change_pct > sanity_check_max_price_change_pct:
    skip_execution()
    send_alert("Sanity check falhou - variação anormal de preço")
    # Não executa ordem (pode ser dados incorretos ou flash crash)
```

---

### 7. Emergency Sell (CRY-71)

Vende TODA a posição a mercado se preço atingir threshold de emergência.

**Configuração**:
```json
{
  "enable_emergency_sell": true,
  "emergency_sell_price": "80000.00"  // Vende tudo se BTC < R$ 80k
}
```

**Regra**:
```python
if current_price <= emergency_sell_price:
    sell_market_all()  # Vende TUDO a mercado IMEDIATAMENTE
    send_alert("Emergency sell executado")
    pause_bot()
```

---

### 8. Auto-Cancelamento de Ordens Distantes (CRY-73)

Cancela automaticamente ordens pending que estão há muito tempo sem executar E distantes do preço atual.

**Configuração**:
```json
{
  "auto_cancel_config": {
    "enabled": true,
    "time_threshold_seconds": 300,     // 5 minutos
    "distance_threshold_pct": "2.0"    // 2% de distância
  }
}
```

**Regra**:
```python
for order in pending_orders:
    time_elapsed = now - order.created_at
    distance_pct = abs(order.price - current_price) / current_price * 100

    if (time_elapsed > time_threshold_seconds AND
        distance_pct > distance_threshold_pct):
        cancel_order(order.id)
        release_blocked_funds(order.value)
```

**Benefício**: Evita ordens "esquecidas" longe do mercado, liberando fundos bloqueados.

---

## Workflows Automatizados

### 1. smart_bot_purchase (63s)

Processa compras automáticas para bots CONTINUOUS.

**Lógica**:
```python
for bot in SmartBot.find(status=ACTIVE, operation_mode=CONTINUOUS):
    # Verifica intervalo
    if now - bot.last_purchase_at < bot.purchase_interval_seconds:
        continue  # Ainda não passou o intervalo

    # Valida circuit breaker
    if bot.consecutive_failures >= bot.circuit_breaker_threshold:
        bot.pause()
        continue

    # Calcula fundos disponíveis
    available = bot.max_position_size - bot.total_invested - bot.total_blocked

    # Calcula valor da ordem
    order_value = min(bot.purchase_amount, available)

    if order_value < minimum_order_size:
        continue  # Fundos insuficientes

    # Bloqueia fundos ANTES de criar ordem
    bot.block_funds(order_value)
    await bot_repo.update(bot)

    # Obtém orderbook
    orderbook = await exchange.get_orderbook(symbol)

    # Preço competitivo (ligeiramente acima do best bid)
    price = orderbook.best_bid + Decimal("0.01")

    # Quantidade
    quantity = order_value / price

    # Cria ordem LIMIT na exchange
    order = await exchange.create_order(
        symbol=symbol,
        side="buy",
        type="LIMIT",
        quantity=quantity,
        price=price
    )

    # Salva contexto de auditoria (CRY-39)
    await order_context_repo.save(OrderCreationContext(
        order_id=order.id,
        strategy_source="purchase",
        user_id=bot.user_id,
        symbol=symbol,
        side="buy",
        timestamp=now,
        quantity=quantity,
        price=price,
        best_bid=orderbook.best_bid,
        best_ask=orderbook.best_ask,
        spread_pct=(orderbook.best_ask - orderbook.best_bid) / orderbook.best_bid * 100
    ))

    # Atualiza última compra
    bot.last_purchase_at = now
    await bot_repo.update(bot)
```

---

### 2. smart_bot_sales (67s)

Processa vendas automáticas para bots CONTINUOUS.

**Lógica**:
```python
for bot in SmartBot.find(status=ACTIVE, operation_mode=CONTINUOUS):
    # Verifica intervalo
    if now - bot.last_sale_at < bot.sale_interval_seconds:
        continue

    # Obtém saldo disponível da crypto
    balance = await exchange.get_balance(crypto_asset)

    if balance.available <= 0:
        continue  # Sem saldo para vender

    # Obtém orderbook
    orderbook = await exchange.get_orderbook(symbol)

    # Preço competitivo (ligeiramente abaixo do best ask)
    price = orderbook.best_ask - Decimal("0.01")

    # Vende TODO o saldo
    order = await exchange.create_order(
        symbol=symbol,
        side="sell",
        type="LIMIT",
        quantity=balance.available,
        price=price
    )

    # Salva contexto
    await order_context_repo.save(OrderCreationContext(...))

    # Atualiza última venda
    bot.last_sale_at = now
    await bot_repo.update(bot)
```

---

### 3. smart_bot_check_situation (33s)

Monitora riscos e triggers de todos os bots ativos.

**Lógica**:
```python
for bot in SmartBot.find(status=ACTIVE):
    # Obtém preço atual
    ticker = await exchange.get_ticker(symbol)
    current_price = ticker.last_price

    # 1. Sanity Check
    if bot.enable_sanity_check:
        price_change = abs(current_price - bot.last_known_price) / bot.last_known_price * 100
        if price_change > bot.sanity_check_max_price_change_pct:
            send_alert("Sanity check falhou")
            continue

    # 2. Emergency Sell
    if bot.enable_emergency_sell and current_price <= bot.emergency_sell_price:
        await sell_market_all(bot)
        bot.pause()
        send_alert("Emergency sell executado")
        continue

    # 3. Hard Stop Loss
    loss_pct = ((bot.base_price - current_price) / bot.base_price) * 100
    if loss_pct >= bot.hard_stop_loss_percentage:
        await sell_market_all(bot)
        send_alert("Hard stop loss ativado")
        continue

    # 4. Soft Stop Loss
    if loss_pct >= bot.soft_stop_loss_percentage:
        await sell_limit_aggressive(bot)
        send_alert("Soft stop loss ativado")

    # 5. Take Profit
    profit_pct = ((current_price - bot.base_price) / bot.base_price) * 100
    if bot.take_profit_percentage and profit_pct >= bot.take_profit_percentage:
        await sell_limit(bot)
        send_alert("Take profit ativado")

    # 6. Trailing Stop
    if bot.should_adjust_trailing_stop(current_price):
        bot.adjust_trailing_stop(current_price)
        await bot_repo.update(bot)

    # 7. Circuit Breaker (Daily Loss)
    if bot.enable_circuit_breaker:
        daily_loss_pct = bot.calculate_daily_loss_pct()
        if daily_loss_pct >= bot.circuit_breaker_max_daily_loss_pct:
            bot.pause()
            send_alert("Circuit breaker ativado - perda diária")

    # Atualiza último preço conhecido
    bot.last_known_price = current_price
    await bot_repo.update(bot)
```

---

## Intervalos Coprimos (CRY-71)

Os workflows usam intervalos coprimos para minimizar race conditions:

```
smart_bot_purchase: 63s
smart_bot_sales: 67s
smart_bot_check_situation: 33s
sync_orders: 42s
```

**Benefício**: Workflows raramente processam o mesmo bot simultaneamente, evitando conflitos de versão (optimistic locking).

---

## API Endpoints

### Criar SmartBot
```
POST /api/smart-bots
```

### Listar SmartBots
```
GET /api/smart-bots?status=active&exchange=Binance&coin=BTC
```

### Obter Detalhes
```
GET /api/smart-bots/{id}
```

### Atualizar Configuração
```
PUT /api/smart-bots/{id}
```

### Pausar/Retomar
```
POST /api/smart-bots/{id}/pause
POST /api/smart-bots/{id}/resume
```

### Arquivar (Soft Delete)
```
DELETE /api/smart-bots/{id}
```

### Estatísticas
```
GET /api/smart-bots/{id}/stats
```

### Resetar Circuit Breaker
```
POST /api/smart-bots/{id}/reset-circuit-breaker
```

---

## Métricas de Sucesso

### KPIs
- **Win Rate**: % de trades lucrativos (meta > 60%)
- **Profit Factor**: Total profits / Total losses (meta > 1.5)
- **Average Profit per Trade**: Lucro médio por operação
- **Maximum Drawdown**: Maior queda de capital (meta < 20%)
- **Circuit Breaker Triggers**: Quantidade (monitora robustez)

---

## Problemas Comuns e Limitações

### ❌ Ordem não executa (fica PENDING)
**Causa**: Preço LIMIT muito distante do mercado
**Solução**: Auto-cancel config (CRY-73) cancela automaticamente

### ❌ Fundos bloqueados sem ordem ativa
**Causa**: Ordem cancelada mas `release_blocked_funds()` não chamado
**Solução**: Workflow sync_orders detecta e corrige

### ⚠️ Limitações
- Workflows com intervalos fixos (não real-time)
- Adaptive sell requer dados de candles (latência)
- Circuit breaker reset apenas manual

---

## Referências Técnicas

### Arquivos Relacionados
- `backend/src/domain/entities/smart_bot.py`
- `backend/src/application/use_cases/create_smart_bot.py`
- `backend/src/workflows/smart_bot_workflows.py`
- `backend/src/infrastructure/schedulers/celery_scheduler.py`

### Standards
- **Clean Architecture**: Domain, Application, Infrastructure layers
- **DDD**: Value Objects (SmartBotConfig), Entities (SmartBot)
- **SOLID**: Single Responsibility, Dependency Inversion
