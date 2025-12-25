---
spec_version: "1.3.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.2.0"
status: "active"
category: "technical"
tags: ['technical', 'business_logic']
---

# Lógica de Negócio - Crypteras Trading System

:::version_info
**Versão**: 1.3.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Documentar regras de negócio complexas de trading para evitar bugs e facilitar onboarding de novos devs.

**Constraints** (limites obrigatórios):
- Manter sincronizado com código (atualizar quando regra muda)
- Incluir exemplos numéricos concretos
- Referenciar arquivos de código onde lógica está implementada
- Documentar edge cases e validações

**Non-Goals** (o que NÃO fazer):
- Duplicar toda lógica do código (apenas regras complexas)
- Criar especificação formal tipo Z notation
- Documentar código que é auto-explicativo
:::

:::failure_modes
**Falhas Conhecidas de Lógica de Negócio**:

1. **DCA - max_cycles Não Respeitado**
   - **Tipo**: validation
   - **Descrição**: DCABot continua executando compras mesmo após atingir `max_cycles` configurado
   - **Gatilho**: Bot configurado com `max_cycles=52` mas executa compra #53, #54, etc
   - **Impacto**: 🔴 Crítico (usuário gasta mais do que planejado)
   - **Mitigação**: Workflow `dca_bot_execution` DEVE verificar `has_reached_max_cycles()` ANTES de criar ordem. Se True, pausar bot
   - **Detecção**: `current_cycle > max_cycles` mas bot ainda está ACTIVE
   - **Código Correto**:
     ```python
     if bot.has_reached_max_cycles():
         bot.status = BotStatus.PAUSED
         await bot_repository.update(bot)
         return  # NÃO executar compra
     ```

2. **Preço Médio Ponderado Calculado Errado (DCA)**
   - **Tipo**: validation
   - **Descrição**: `update_average_price()` não calcula corretamente média ponderada após múltiplas compras
   - **Gatilho**: Múltiplas compras em DCA com preços diferentes
   - **Impacto**: 🟡 Médio (take profit/stop loss disparam em momento errado)
   - **Mitigação**: Validar fórmula: `new_avg = (old_qty * old_price + new_qty * new_price) / (old_qty + new_qty)`
   - **Detecção**: Calcular média manualmente e comparar com `bot.stats.average_price`
   - **Exemplo de Teste**:
     ```python
     # Compra 1: 0.001 BTC @ R$ 150.000
     bot.update_average_price(Decimal("150000"), Decimal("0.001"))
     assert bot.stats.average_price == Decimal("150000")

     # Compra 2: 0.001 BTC @ R$ 160.000
     bot.update_average_price(Decimal("160000"), Decimal("0.001"))
     assert bot.stats.average_price == Decimal("155000")  # (150+160)/2
     ```

3. **Circuit Breaker Não Ativa Após N Falhas Consecutivas**
   - **Tipo**: validation
   - **Descrição**: SmartBot não ativa circuit breaker mesmo após threshold de falhas (ex: 3 ordens recusadas)
   - **Gatilho**: Múltiplas ordens rejeitadas pela exchange (saldo insuficiente, símbolo inválido, etc)
   - **Impacto**: 🔴 Crítico (spam de ordens inválidas, desperdício de fees)
   - **Mitigação**: Workflow DEVE incrementar contador de falhas. Após N falhas, chamar `bot.activate_circuit_breaker()`
   - **Detecção**: Logs mostram múltiplas falhas mas `circuit_breaker_active=False`
   - **Código Correto**:
     ```python
     if order_failed:
         bot.stats.consecutive_failures += 1
         if bot.stats.consecutive_failures >= 3:
             bot.activate_circuit_breaker()
             await bot_repository.update(bot)
     else:
         bot.stats.consecutive_failures = 0  # Reset em sucesso
     ```

4. **CandleBot - Indicadores Desabilitados Mas Ainda Calculados**
   - **Tipo**: validation
   - **Descrição**: `evaluate_signal()` calcula RSI, MA, etc mesmo quando `rsi_enabled=False`
   - **Gatilho**: Configurar CandleBot com apenas alguns indicadores habilitados
   - **Impacto**: 🟢 Baixo (desperdício de CPU, sinais podem ser imprecisos)
   - **Mitigação**: SEMPRE verificar `if config.indicators.rsi_enabled:` antes de calcular/usar indicador
   - **Detecção**: Logs mostram cálculo de indicadores desabilitados. Performance degradada
   - **Código Correto**:
     ```python
     if config.indicators.rsi_enabled:
         rsi = calculate_rsi(candles, config.indicators.rsi_period)
         if rsi < 30:
             buy_signals += 1
     # Se disabled, não calcula nem usa RSI
     ```

5. **SmartBot - total_invested Negativo Após Venda**
   - **Tipo**: validation
   - **Descrição**: `total_invested` fica negativo após sync de ordem de venda (bug na subtração)
   - **Gatilho**: Venda de posição completa ou parcial
   - **Impacto**: 🔴 Crítico (`get_available_funds()` retorna valor absurdo, bot trava)
   - **Mitigação**: SEMPRE validar `total_invested >= 0` após atualização. Se negativo, setar para 0 e logar warning
   - **Detecção**: `total_invested < 0` no MongoDB ou dashboard mostra valor negativo
   - **Código Correto**:
     ```python
     if order.side == "SELL":
         bot.stats.total_invested -= order.amount
         if bot.stats.total_invested < Decimal("0"):
             logger.warning(f"total_invested ficou negativo: {bot.stats.total_invested}")
             bot.stats.total_invested = Decimal("0")
     ```

6. **Take Profit Dispara Antes do Threshold**
   - **Tipo**: validation
   - **Descrição**: `should_take_profit()` retorna True com lucro menor que `take_profit_percentage`
   - **Gatilho**: Comparação de Decimal com float ou erro de arredondamento
   - **Impacto**: 🟡 Médio (venda prematura, oportunidade de lucro perdida)
   - **Mitigação**: SEMPRE usar Decimal. Comparação: `profit >= take_profit_percentage` (não `>`)
   - **Detecção**: Bot vende com lucro de 4.9% mas `take_profit_percentage=5.0`
   - **Código Correto**:
     ```python
     profit = ((current_price - average_entry_price) / average_entry_price) * 100
     # profit e take_profit_percentage DEVEM ser Decimal
     return profit >= self.config.take_profit_percentage
     ```
:::

:::explainability
**Requirement**: ✅ Required (para regras de negócio complexas)

**Output Format**:
IA DEVE explicar decisões de lógica de negócio seguindo este formato:

```markdown
## 🤖 Decisão de Lógica de Negócio

**Decisão**: [O que foi decidido - ex: "Calcular fundos disponíveis como `max_position_size - total_invested - total_blocked`"]

**Source**:
- `specs/technical/BUSINESS_LOGIC.md` v1.3.0 - Seção relevante
- Domain entity: `backend/src/domain/entities/smart_bot.py:209-219`
- Failure modes aplicados (se relevante)

**Rationale**:
1. [Razão baseada na regra de negócio documentada]
2. [Impacto financeiro ou de trading]
3. [Proteção contra riscos]
4. [Consistência com domínio]

**Alternatives Considered**:
1. ❌ [Alternativa 1] - [Por que não foi escolhida]
2. ❌ [Alternativa 2] - [Por que não foi escolhida]
3. ✅ [Escolhida] - [Por que foi escolhida]

**Trade-offs**:
- ✅ Pro: [Benefício 1]
- ✅ Pro: [Benefício 2]
- ⚠️ Con: [Desvantagem se houver]

**Business Impact**:
- **Trading**: [Como afeta operações de trading]
- **Risk Management**: [Impacto em gestão de risco]
- **User Experience**: [Como usuário percebe]

**Audit Trail**:
- Timestamp: [ISO 8601 format]
- Specs Consultadas: [BUSINESS_LOGIC.md v1.x.x]
- Domain Entities Modificadas: [lista]
- Business Rules Aplicadas: [BR-XXX se numeradas]
- Failure Modes Aplicados: [FM#X se aplicável]
```

**Quando Explicar** (gatilhos obrigatórios):
1. **Implementação de regra de negócio complexa**: Stop-loss, trailing stop, circuit breaker
2. **Cálculo envolvendo múltiplas variáveis**: Fundos disponíveis, preço médio ponderado, profit/loss
3. **Validação que pode rejeitar entrada válida**: max_cycles, allocated_balance, circuit breaker
4. **Decisão que impacta métricas de negócio**: total_invested, total_blocked, average_price
5. **Lógica de gestão de risco**: Hard/soft stop-loss, take profit, circuit breaker
6. **Workflow de execução automática**: Quando executar compra/venda, sinais de candle
7. **Atualização de estado crítico**: Base price, next_execution, current_cycle
8. **Consolidação de múltiplos indicadores**: Evaluate signal (CandleBot)
9. **Implementação de failure mode mitigation**: Correção de bugs conhecidos de lógica
10. **Mudança em fórmula matemática**: Variação percentual, média ponderada

**Audit Trail Obrigatório**:
- Timestamp da decisão
- Specs consultadas (nome + versão)
- Domain entities afetadas (arquivo + linha)
- Business rules aplicadas
- Failure modes mitigados (se aplicável)
- Impacto em métricas de trading

**Exemplo Completo**:

```markdown
## 🤖 Decisão de Lógica de Negócio

**Decisão**: Implementar `get_available_funds()` como `max_position_size - total_invested - total_blocked`

**Source**:
- `specs/technical/BUSINESS_LOGIC.md` v1.3.0 - SmartBot "Calcular Fundos Disponíveis"
- `specs/technical/BUSINESS_LOGIC.md` v1.3.0 - Regra: "Quando total_invested/total_blocked aumentam/diminuem"
- Domain entity: `backend/src/domain/entities/smart_bot.py:209-219`

**Rationale**:
1. **Regra de Negócio**: Fundos disponíveis = teto (max_position_size) menos recursos já alocados
2. **total_invested**: Ordens filled que ainda não foram vendidas (capital em posição)
3. **total_blocked**: Ordens pending aguardando fill (capital reservado)
4. **Proteção**: Previne bot gastar mais que `max_position_size` configurado (limite de risco)

**Alternatives Considered**:
1. ❌ `max_position_size - total_invested` (sem total_blocked) - Permite criar ordens além do limite enquanto aguardam fill
2. ❌ Consultar saldo real da exchange - Ignora ordens pending, pode criar duplicatas
3. ❌ `max_position_size - (total_invested + total_blocked + margem_segurança)` - Conservador demais, desperdiça capital
4. ✅ `max_position_size - total_invested - total_blocked` - Preciso, considera filled + pending

**Trade-offs**:
- ✅ Pro: Previne overallocation (capital > max_position_size)
- ✅ Pro: Considera ordens pending (evita duplicatas)
- ✅ Pro: Simples de calcular (não requer chamada à exchange)
- ⚠️ Con: Depende de sync_orders estar atualizado (mitigado por sync a cada 42s)

**Business Impact**:
- **Trading**: Bot para de comprar quando atinge limite configurado
- **Risk Management**: User controla exposição máxima via `max_position_size`
- **User Experience**: Dashboard mostra fundos disponíveis corretos em tempo real

**Audit Trail**:
- Timestamp: 2025-12-25T11:30:00Z
- Specs Consultadas: BUSINESS_LOGIC.md v1.3.0
- Domain Entities Modificadas: smart_bot.py:209-219 (`get_available_funds()`)
- Business Rules Aplicadas: BR-SmartBot-002 (Fundos Disponíveis)
- Workflows Dependentes: smart_bot_purchase (usa este cálculo para decidir se compra)
```

**Quando NÃO Explicar** (decisões triviais):
- Getters/setters simples de propriedades
- Validações óbvias (email válido, valor > 0)
- Conversões de tipo padrão (str para Decimal)
- Lógica já amplamente documentada em docstrings
- Regras de negócio triviais sem trade-offs
:::

:::breaking_changes
**v1.3.0**:
- Adicionada seção `:::explainability` com requisitos obrigatórios para decisões de lógica de negócio
- Definidos 10 gatilhos obrigatórios de explainability para regras de trading
- Incluído exemplo completo de explicação de cálculo de fundos disponíveis
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.2.0**:
- Adicionada seção `:::failure_modes` com 6 falhas conhecidas de lógica de negócio
- Documentadas falhas reais: DCA max_cycles, Preço médio ponderado, Circuit breaker, Indicadores desabilitados, total_invested negativo, Take profit threshold
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Regras de negócio e lógica core
:::

Este documento detalha as regras de negócio reais implementadas no código, baseado em análise de `backend/src/domain/`.

---

## 🎯 Conceitos de Domínio

### Hierarquia de Entidades

```
User (1)
├─── Subscription (1) → Plan (FREE | PRO | MAX)
├─── SmartBot (N) → Orders (N)
├─── CandleBot (N) → Orders (N) + CandleSignals (N)
└─── DCABot (N) → Orders (N)
```

### User

**Arquivo**: `backend/src/domain/entities/user.py`

**Atributos Principais**:
```python
@dataclass
class User:
    id: str
    email: str
    password_hash: str  # bcrypt
    encrypted_credentials: Dict[str, str]  # AES-256
    subscription: Subscription
    onboarding_completed: bool
    created_at: datetime
```

**Credenciais Encriptadas**:
```python
encrypted_credentials = {
    "mb_api_id": "base64_encrypted...",
    "mb_api_key": "base64_encrypted...",
    "mb_account_id": "base64_encrypted...",
    "binance_api_key": "base64_encrypted...",
    "binance_api_secret": "base64_encrypted..."
}
```

**Lógica de Negócio**:
- Credenciais de exchange são **sempre** armazenadas encriptadas (AES-256-CBC)
- Usuário pode ter credenciais de múltiplas exchanges simultaneamente
- Onboarding obrigatório na primeira vez (intro.js no frontend)

### Subscription

**Arquivo**: `backend/src/domain/entities/subscription.py`

**Planos Disponíveis**:
```python
class SubscriptionPlan(str, Enum):
    FREE = "FREE"  # 1 contexto, workflows ativos
    PRO = "PRO"    # 3 contextos, candle strategies, R$ 19,90/mês
    MAX = "MAX"    # Ilimitado, indicadores avançados, R$ 97/mês
```

**Limites por Plano**:
```python
PLAN_LIMITS = {
    "FREE": {"max_contexts": 1, "max_bots_total": 1},
    "PRO": {"max_contexts": 3, "max_bots_total": 10},
    "MAX": {"max_contexts": -1, "max_bots_total": -1}  # -1 = ilimitado
}
```

**Payment Providers**:
- `mercadopago`: Mercado Pago (primary para Brasil)
- `stripe`: Stripe (fallback)

**Status do Payment**:
```python
class SubscriptionStatus(str, Enum):
    ACTIVE = "active"        # Pagamento em dia
    PAST_DUE = "past_due"   # Pagamento atrasado
    CANCELED = "canceled"    # Cancelado pelo user
    TRIALING = "trialing"    # Período de trial (se houver)
```

---

## 🤖 SmartBot - Trading Manual/Scheduled/Continuous

**Arquivo**: `backend/src/domain/entities/smart_bot.py` (716 lines)

### Modos de Operação

```python
class OperationMode(str, Enum):
    MANUAL = "MANUAL"        # User aciona compra/venda manualmente via UI
    SCHEDULED = "SCHEDULED"  # Cron-based (ex: "0 9 * * 1-5" = 9 AM dias úteis)
    CONTINUOUS = "CONTINUOUS"  # Interval-based (compra a cada 63s, vende a cada 67s)
```

**Configuração de Intervalos** (apenas para CONTINUOUS):
```python
@dataclass
class SmartBotIntervals:
    purchase: int = 63   # segundos (co-primo)
    sales: int = 67      # segundos (co-primo)
    check_situation: int = 33  # segundos (co-primo)
```

### Configuração do Bot

```python
@dataclass
class SmartBotConfig:
    # Gestão de Posição
    max_position_size: Decimal  # Máximo a investir (ex: R$ 5000)

    # Gestão de Risco
    trailing_stop_percentage: Decimal = Decimal("3.0")  # 3% (ajusta basePrice quando sobe)
    soft_stop_loss: Decimal = Decimal("2.0")  # 2% (venda urgente LIMIT)
    hard_stop_loss: Decimal = Decimal("5.0")  # 5% (venda a MERCADO imediata)

    # Estratégia de Ordem
    order_strategy: OrderStrategy = OrderStrategy.LIMIT  # LIMIT | MARKET
```

**Validação** (`__post_init__`):
```python
# Regras validadas automaticamente:
- name não pode ser vazio
- max_position_size > 0
- soft_stop_loss < hard_stop_loss  # Ex: 2% < 5%
- operation_mode in [MANUAL, SCHEDULED, CONTINUOUS]
- Se CONTINUOUS: intervals obrigatórios
- Se SCHEDULED: cron_expression obrigatório
```

### Stats do Bot

```python
@dataclass
class SmartBotStats:
    base_price: Decimal = Decimal("0")  # Preço base (ajustado por trailing stop)
    total_invested: Decimal = Decimal("0")  # Ordens filled (ex: R$ 3000)
    total_blocked: Decimal = Decimal("0")   # Ordens pending (ex: R$ 500)
    total_purchases: int = 0
    total_sales: int = 0
    circuit_breaker_active: bool = False
    circuit_breaker_activated_at: Optional[datetime] = None
    trailing_stop_adjusted_at: Optional[datetime] = None
```

### Regras de Negócio - SmartBot

#### 1. Verificar Se Pode Comprar

**Método**: `can_execute_purchase() -> bool`

**Regras**:
```python
# 1. Bot deve estar ACTIVE
if self.status != BotStatus.ACTIVE:
    return False

# 2. Circuit breaker não pode estar ativo
if self.circuit_breaker_active:
    return False

# 3. Deve ter fundos disponíveis
available = self.get_available_funds()
if available <= Decimal("0"):
    return False

return True
```

#### 2. Calcular Fundos Disponíveis

**Método**: `get_available_funds() -> Decimal`

**Fórmula**:
```python
disponível = max_position_size - total_invested - total_blocked
```

**Exemplo**:
```python
max_position_size = R$ 5000
total_invested = R$ 3000  # Ordens filled (compradas e aguardando venda)
total_blocked = R$ 500    # Ordens pending (aguardando fill)
→ Disponível = R$ 5000 - R$ 3000 - R$ 500 = R$ 1500
```

**Quando `total_invested` aumenta**:
- Ordem de COMPRA é filled → `total_invested += order.amount`

**Quando `total_invested` diminui**:
- Ordem de VENDA é filled → `total_invested -= order.amount`

**Quando `total_blocked` aumenta**:
- Ordem LIMIT criada (pending) → `total_blocked += order.amount`

**Quando `total_blocked` diminui**:
- Ordem filled ou cancelada → `total_blocked -= order.amount`

#### 3. Verificar Stop-Loss

**Método**: `should_trigger_stop_loss(current_price: Decimal) -> bool`

**Cálculo de Variação**:
```python
variation = ((current_price - base_price) / base_price) * 100
```

**Regra de Hard Stop-Loss** (5%):
```python
if variation <= -hard_stop_loss:  # Ex: <= -5%
    return True  # Venda a MERCADO imediata
```

**Exemplo**:
```python
base_price = R$ 100.000
current_price = R$ 94.500
variation = ((94500 - 100000) / 100000) * 100 = -5.5%
→ -5.5% <= -5% → True (trigger hard stop-loss)
```

**Comportamento do Sistema**:
- Hard stop-loss (>= 5%): `smart_bot_check_situation` workflow detecta e cria ordem MARKET de venda **imediata**
- Soft stop-loss (2% a 5%): Workflow detecta e ativa `urgent_sales_active = True`, venda urgente via LIMIT competitivo

#### 4. Trailing Stop (Ajuste Automático de Base Price)

**Método**: `adjust_trailing_stop(current_price: Decimal)`

**Regra**:
```python
variation = ((current_price - base_price) / base_price) * 100

if variation >= trailing_stop_percentage:  # Ex: >= 3%
    self.base_price = current_price  # Ajusta para cima
    self.trailing_stop_adjusted_at = datetime.utcnow()
```

**Exemplo**:
```python
# Compra inicial
base_price = R$ 100.000

# Preço sobe 3.5%
current_price = R$ 103.500
variation = 3.5%
→ 3.5% >= 3% → base_price = R$ 103.500 (novo piso)

# Agora, se cair 5% do novo piso:
# R$ 103.500 * 0.95 = R$ 98.325
# Sistema vende em R$ 98.325 (lucro de -1.7% vs compra original, mas protegeu 3.5% de ganho)
```

**Por Que Trailing Stop É Útil**:
- Sem trailing: Se comprou em R$ 100k, preço foi a R$ 110k (+10%), e caiu para R$ 95k → Vende com -5% de prejuízo
- Com trailing: Quando chegou em R$ 110k, basePrice virou R$ 110k. Ao cair para R$ 104.5k (-5% do novo piso), vende com +4.5% de lucro

#### 5. Circuit Breaker

**Método**: `activate_circuit_breaker()`

**Quando Ativa**:
- Após N falhas consecutivas de ordens (ex: 3 ordens recusadas pela exchange)
- Saldo insuficiente detectado repetidamente
- Erros de conexão persistentes

**Comportamento**:
```python
self.circuit_breaker_active = True
self.circuit_breaker_activated_at = datetime.utcnow()
self.status = BotStatus.PAUSED  # Bot pausado automaticamente
```

**Quando Desativa**:
- User desativa manualmente via UI
- Meia-noite (reset automático - se implementado)

**Objetivo**:
- Prevenir spam de ordens inválidas
- Evitar desperdício de fees
- Proteger contra bugs que causam loop infinito de ordens

---

## 📊 CandleBot - Análise Técnica

**Arquivo**: `backend/src/domain/entities/candle_bot.py` (547 lines)

### Configuração

```python
@dataclass
class CandleBotConfig:
    # Timeframe e Lookback
    timeframe: str  # "1m", "5m", "15m", "30m", "1h", "4h", "1d", etc.
    lookback_periods: int = 50  # Quantas candles analisar (30-200)

    # Indicadores (10 disponíveis)
    indicators: IndicatorsConfig

    # Gestão de Risco
    take_profit_percentage: Decimal = Decimal("5.0")  # 5%
    stop_loss_percentage: Decimal = Decimal("2.0")    # 2%

    # Execução
    auto_execute: bool = True  # True = executa ordens, False = apenas sinais
```

### Indicadores Técnicos (10 Total)

**Arquivo**: `backend/src/domain/services/technical_analysis_service.py`

#### Trend (Tendência)
```python
# Moving Average (MA)
ma_enabled: bool = False
ma_period: int = 20  # 5-200

# Exponential Moving Average (EMA)
ema_enabled: bool = False
ema_period: int = 12
```

#### Momentum
```python
# Relative Strength Index (RSI)
rsi_enabled: bool = False
rsi_period: int = 14  # 2-50

# Stochastic RSI
stoch_rsi_enabled: bool = False
```

#### Volatility (Volatilidade)
```python
# Bollinger Bands (BB)
bb_enabled: bool = False
bb_period: int = 20  # 10-50
bb_std_dev: float = 2.0  # Desvios padrão

# Average True Range (ATR)
atr_enabled: bool = False
atr_period: int = 14
```

#### Volume
```python
# Volume Weighted Average Price (VWAP)
vwap_enabled: bool = False

# Money Flow Index (MFI)
mfi_enabled: bool = False
mfi_period: int = 14

# On-Balance Volume (OBV)
obv_enabled: bool = False

# Volume Moving Average
volume_ma_enabled: bool = False
volume_ma_period: int = 20
```

### Regras de Negócio - CandleBot

#### 1. Geração de Sinais

**Método**: `evaluate_signal(indicators: Dict) -> Signal`

**Processo**:
```python
buy_signals = 0
sell_signals = 0
total_indicators = 0

# Para cada indicador habilitado:
if rsi_enabled:
    total_indicators += 1
    if indicators['rsi'] < 30:
        buy_signals += 1  # Sobrevenda
    elif indicators['rsi'] > 70:
        sell_signals += 1  # Sobrecompra

if bb_enabled:
    total_indicators += 1
    if price < lower_bb:
        buy_signals += 1  # Preço abaixo da banda inferior
    elif price > upper_bb:
        sell_signals += 1  # Preço acima da banda superior

# Decisão final
if buy_signals >= total_indicators * 0.6:  # 60% dos indicadores concordam
    return Signal(signal="BUY", confidence="HIGH")
elif sell_signals >= total_indicators * 0.6:
    return Signal(signal="SELL", confidence="HIGH")
else:
    return Signal(signal="HOLD", confidence="LOW")  # Sinais conflitantes
```

**Confidence Levels**:
```python
class ConfidenceLevel(str, Enum):
    LOW = "LOW"      # < 60% dos indicadores concordam
    MEDIUM = "MEDIUM"  # 60-80% concordam
    HIGH = "HIGH"    # > 80% concordam
```

#### 2. Take Profit

**Método**: `should_take_profit(current_price: Decimal) -> bool`

**Cálculo**:
```python
if not stats.average_entry_price:
    return False  # Sem posição aberta

profit = ((current_price - average_entry_price) / average_entry_price) * 100

return profit >= take_profit_percentage  # Ex: >= 5%
```

**Exemplo**:
```python
average_entry_price = R$ 100.000  # Preço médio de compra
current_price = R$ 105.500
profit = ((105500 - 100000) / 100000) * 100 = 5.5%
→ 5.5% >= 5% → True (vende para realizar lucro)
```

#### 3. Stop Loss

**Método**: `should_trigger_stop_loss(current_price: Decimal) -> bool`

**Cálculo**:
```python
if not stats.average_entry_price:
    return False

loss = ((current_price - average_entry_price) / average_entry_price) * 100

return loss <= -stop_loss_percentage  # Ex: <= -2%
```

**Diferença vs SmartBot**:
- **SmartBot**: Stop-loss baseado em `base_price` (ajustado por trailing stop)
- **CandleBot**: Stop-loss baseado em `average_entry_price` (preço médio de compra)

#### 4. Workflow de Análise

**Arquivo**: `backend/src/workflows/candle_bot_analysis_workflow.py`

**Frequência**: A cada 120s (co-primo)

**Processo**:
```python
1. Buscar todos CandleBots ativos
2. Para cada bot:
   3. Buscar candles da exchange (últimas N velas)
   4. Calcular indicadores habilitados:
      - RSI se rsi_enabled
      - MA se ma_enabled
      - BB se bb_enabled
      - etc.
   5. Consolidar sinais → evaluate_signal()
   6. Salvar sinal no MongoDB (CandleSignal document)
   7. Se auto_execute=True E signal=BUY:
      8. Criar ordem de compra LIMIT
   9. Se auto_execute=True E signal=SELL:
      10. Criar ordem de venda LIMIT
```

---

## 💰 DCABot - Dollar Cost Averaging

**Arquivo**: `backend/src/domain/entities/dca_bot.py` (318 lines)

### Configuração

```python
@dataclass
class DCABotConfig:
    # Parâmetros de Compra
    purchase_amount_brl: Decimal  # Fixo por execução (ex: R$ 100)
    frequency: str  # "1h" | "6h" | "24h" | "7d" | "30d"

    # Budget
    allocated_balance: Decimal  # Total alocado (ex: R$ 5000)

    # Gestão de Risco
    take_profit_percentage: Decimal = Decimal("20.0")  # 20%
    stop_loss_percentage: Decimal = Decimal("10.0")    # 10%

    # Ciclos
    max_cycles: Optional[int] = None  # Ex: 52 (1 ano de compras semanais)

    # Restart
    auto_restart_after_exit: bool = False  # Reinicia após take profit/stop-loss?
```

### Stats

```python
@dataclass
class DCABotStats:
    current_cycle: int = 0  # Ciclo atual (quantas compras)
    total_invested: Decimal = Decimal("0")  # Total gasto
    current_holdings: Decimal = Decimal("0")  # Quantidade acumulada (ex: 0.05 BTC)
    average_price: Decimal = Decimal("0")  # Preço médio ponderado
    next_execution: Optional[datetime] = None  # Próxima compra
```

### Regras de Negócio - DCABot

#### 1. Verificar Se Deve Executar

**Método**: `should_execute_now() -> bool`

**Regras**:
```python
if status != BotStatus.ACTIVE:
    return False

if not next_execution:
    return True  # Primeira execução

return datetime.utcnow() >= next_execution
```

#### 2. Calcular Próxima Execução

**Método**: `calculate_next_execution()`

**Mapeamento de Frequência**:
```python
frequency_map = {
    "1h": timedelta(hours=1),
    "6h": timedelta(hours=6),
    "24h": timedelta(days=1),
    "7d": timedelta(days=7),
    "30d": timedelta(days=30)
}

delta = frequency_map[self.config.frequency]
self.stats.next_execution = datetime.utcnow() + delta
```

**Exemplo**:
```python
# Compra semanal
frequency = "7d"
current_time = 2024-12-24 10:00:00
→ next_execution = 2024-12-31 10:00:00
```

#### 3. Atualizar Preço Médio Ponderado

**Método**: `update_average_price(purchase_price: Decimal, purchase_quantity: Decimal)`

**Fórmula**:
```python
old_total_value = current_holdings * average_price
new_total_value = old_total_value + (purchase_quantity * purchase_price)
new_holdings = current_holdings + purchase_quantity
new_average_price = new_total_value / new_holdings
```

**Exemplo**:
```python
# Estado inicial
current_holdings = 0.001 BTC
average_price = R$ 150.000
old_total_value = 0.001 * 150000 = R$ 150

# Nova compra
purchase_quantity = 0.001 BTC
purchase_price = R$ 160.000
purchase_value = 0.001 * 160000 = R$ 160

# Atualização
new_total_value = R$ 150 + R$ 160 = R$ 310
new_holdings = 0.001 + 0.001 = 0.002 BTC
new_average_price = R$ 310 / 0.002 = R$ 155.000

→ average_price atualizado para R$ 155.000
```

#### 4. Verificar Atingimento de Limites de Ciclo

**Método**: `has_reached_max_cycles() -> bool`

**Regra**:
```python
if not max_cycles:
    return False  # Sem limite

return current_cycle >= max_cycles
```

**Exemplo**:
```python
# DCA semanal por 1 ano
frequency = "7d"
max_cycles = 52  # 52 semanas
current_cycle = 52
→ True (parar DCA)
```

#### 5. Workflow de Execução

**Arquivo**: `backend/src/workflows/dca_bot_execution.py`

**Frequência**: A cada 60s (co-primo)

**Processo**:
```python
1. Buscar DCAbots ativos
2. Para cada bot:
   3. Verificar should_execute_now()
   4. Se False: skip
   5. Verificar has_reached_max_cycles()
   6. Se True: pausar bot
   7. Verificar allocated_balance suficiente
   8. Criar exchange via ExchangeFactory
   9. Criar ordem MARKET (garantir execução)
   10. Após fill:
       11. update_average_price()
       12. current_cycle += 1
       13. calculate_next_execution()
```

---

## 🔄 Workflows Automatizados

### sync_orders (Crítico)

**Arquivo**: `backend/src/workflows/sync_orders.py`

**Frequência**: A cada 42s (co-primo)

**Função**: Sincronizar ordens das exchanges com MongoDB

**Processo**:
```python
1. Buscar todos users ativos
2. Para cada user:
   3. Buscar bots do user (SmartBot, CandleBot, DCABot)
   4. Para cada bot:
      5. Criar exchange com credenciais do user
      6. Buscar ordens filled (últimas 1h)
      7. Normalizar formato (OrderNormalizer)
      8. Para cada ordem filled:
         9. Verificar se já existe no MongoDB
         10. Se não existe:
             11. Salvar ordem com strategy_context (CRY-39)
             12. Se ordem de COMPRA:
                 - bot.stats.total_invested += order.amount
                 - bot.stats.total_blocked -= order.amount
             13. Se ordem de VENDA:
                 - bot.stats.total_invested -= order.amount
             14. Salvar bot atualizado (optimistic locking)
```

**Problema Conhecido**:
- ~8% taxa de falha
- Ordens às vezes não sincronizam corretamente
- Causa: Race conditions, version conflicts, timeout de API

### Intervalos dos Workflows

| Workflow | Frequência | Queue | Concurrency |
|----------|-----------|-------|-------------|
| sync_orders | 42s | sync_orders | 10 |
| smart_bot_purchase | 63s | smart_buy | 5 |
| smart_bot_sales | 67s | smart_sell | 5 |
| smart_bot_check_situation | 33s | smart_risk | 5 |
| candle_bot_analysis | 120s | candle_analysis | 5 |
| candle_bot_risk_management | 60s | candle_risk | 5 |
| dca_bot_execution | 60s | dca_execution | 5 |
| dca_bot_risk_management | 30s | dca_risk | 5 |

**Por Que Co-Primos?**:
- Evita picos de carga (todos workflows rodando ao mesmo tempo)
- Distribui CPU/RAM uniformemente
- Reduz version conflicts no MongoDB

---

## 📚 Referências

**Entities**:
- [SmartBot](../../crypteras-improved/backend/src/domain/entities/smart_bot.py)
- [CandleBot](../../crypteras-improved/backend/src/domain/entities/candle_bot.py)
- [DCABot](../../crypteras-improved/backend/src/domain/entities/dca_bot.py)

**Services**:
- [RiskManagement](../../crypteras-improved/backend/src/domain/services/risk_management.py)
- [TechnicalAnalysisService](../../crypteras-improved/backend/src/domain/services/technical_analysis_service.py)

**Workflows**:
- [sync_orders](../../crypteras-improved/backend/src/workflows/sync_orders.py)
- [smart_bot_purchase](../../crypteras-improved/backend/src/workflows/smart_bot_purchase.py)

---

**Última Atualização**: 2024-12-24
**Baseado em**: Código real em `backend/src/domain/`
