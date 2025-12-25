---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'bots']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# DCA Bots (Dollar Cost Averaging)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Implementar Dollar Cost Averaging automatizado para acumulação disciplinada de longo prazo com redução de volatilidade.

**Constraints** (limites obrigatórios):
- Frequências permitidas: 1h, 6h, 24h, 7d, 30d (intervalos padronizados)
- Compra de valor fixo em BRL (não quantidade fixa de cripto)
- Cálculo correto de preço médio ponderado após cada compra
- Take profit e stop-loss calculados sobre média (não sobre última compra)
- Executar compras apenas se saldo suficiente (nunca falhar silenciosamente)
- Próxima execução deve ser calculada e exibida ao usuário

**Non-Goals** (o que NÃO fazer):
- Implementar DCA com ajuste dinâmico de valor (valor fixo apenas)
- Adicionar lógica de "pular compra se preço alto demais" (DCA puro não faz market timing)
- Vender automaticamente antes do take profit (DCA é acumulação)
- Permitir frequências customizadas além das padronizadas
- Implementar DCA reverso (vender periodicamente)
- Adicionar rebalanceamento automático de portfolio
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Bots de Dollar Cost Averaging (DCA)
:::

## Visão Geral

DCA Bots automatizam a estratégia de Dollar Cost Averaging - compras periódicas de valor fixo que reduzem o impacto da volatilidade através da média de preço ponderada. Ideal para investidores de longo prazo que acreditam no ativo mas querem evitar o risco de comprar tudo no pico.

**Status**: ✅ Em Produção
**Planos**: PRO e MAX
**Prioridade**: ALTA

---

## Propósito e Valor

### Para o Usuário
- **Disciplina Automatizada**: Elimina decisão emocional de "quando comprar"
- **Redução de Risco**: Média de preço suaviza impacto de volatilidade
- **Investimento Passivo**: "Configure e esqueça" - bot acumula automaticamente
- **Transparência Total**: Histórico completo de todas as compras e preço médio
- **Proteção de Lucros**: Take profit e stop loss automáticos sobre a média

### Para o Negócio
- **Retenção de Longo Prazo**: DCA incentiva usuários a manter conta ativa por meses
- **Comportamento Saudável**: Educa usuários sobre investimento disciplinado (vs speculation)
- **Menor Suporte**: Estratégia simples gera menos dúvidas que trading ativo
- **Upsell Natural**: Usuários que acumulam querem monitoramento avançado (MAX)

---

## O que é Dollar Cost Averaging?

### Conceito Fundamental

Em vez de investir R$ 3.000 de uma só vez, você investe R$ 100 por semana durante 30 semanas.

**Problema que resolve**: "Comprei Bitcoin a R$ 220k e caiu para R$ 180k no dia seguinte"

**Com DCA**:
```
Semana 1: R$ 100 @ R$ 220.000 = 0,000454 BTC
Semana 2: R$ 100 @ R$ 210.000 = 0,000476 BTC
Semana 3: R$ 100 @ R$ 200.000 = 0,000500 BTC
Semana 4: R$ 100 @ R$ 190.000 = 0,000526 BTC
Semana 5: R$ 100 @ R$ 180.000 = 0,000555 BTC

Total investido: R$ 500
Total acumulado: 0,002511 BTC
Preço médio: R$ 199.124 (vs R$ 220.000 se tivesse comprado tudo na semana 1)
```

**Benefício**: Você "lucrou" com a queda, pois comprou mais barato nas semanas 3-5.

---

### Quando DCA Funciona Melhor

✅ **Tendência de alta de longo prazo com volatilidade**
- Bitcoin historicamente: sobe 200% ao ano mas com quedas de 30-50% no meio
- DCA acumula nas quedas, multiplica nas subidas

✅ **Mercados laterais (range)**
- Compra barato, vende caro, reinicia
- Acumula posição gradualmente

❌ **Tendência de baixa prolongada (bear market)**
- Continua comprando enquanto ativo desvaloriza
- Requer convicção de que ativo vai recuperar
- Solução: stop loss para limitar perdas

---

## Como Funciona o DCA Bot

### Parâmetros Principais

```json
{
  "purchase_amount": "100.00",      // Compra R$ 100 por vez
  "frequency": "24h",               // Frequências: 1h, 6h, 24h, 7d, 30d
  "allocated_balance": "3000.00",   // Budget total: R$ 3.000
  "take_profit_percent": "10.0",    // Vende tudo com 10% de lucro
  "stop_loss_percent": "5.0",       // Vende tudo com 5% de prejuízo
  "max_cycles": 30,                 // Máximo 30 compras (ou null = ilimitado)
  "auto_restart": true,             // Reinicia após take profit
  "order_strategy": "MARKET"        // MARKET (garantia) ou LIMIT (melhor preço)
}
```

---

### Ciclo de Execução

```
1. Bot criado em 01/01/2025 às 00:00
   next_execution = 02/01/2025 às 00:00 (frequency = 24h)
   current_cycle = 0

2. 02/01/2025 00:00 - PRIMEIRA COMPRA
   Preço: R$ 200.000
   Compra: R$ 100 / R$ 200.000 = 0,0005 BTC

   Atualiza:
   - total_invested = R$ 100
   - total_quantity = 0,0005 BTC
   - average_price = R$ 200.000
   - current_cycle = 1
   - next_execution = 03/01/2025 00:00

3. 03/01/2025 00:00 - SEGUNDA COMPRA
   Preço: R$ 210.000
   Compra: R$ 100 / R$ 210.000 = 0,000476 BTC

   Cálculo de nova média:
   - total_quantity = 0,0005 + 0,000476 = 0,000976 BTC
   - total_invested = R$ 200
   - average_price = R$ 200 / 0,000976 = R$ 204.918

   Atualiza:
   - current_cycle = 2
   - next_execution = 04/01/2025 00:00

4. 04/01/2025 00:00 - TERCEIRA COMPRA
   Preço: R$ 190.000 (caiu!)
   Compra: R$ 100 / R$ 190.000 = 0,000526 BTC

   Cálculo de nova média:
   - total_quantity = 0,000976 + 0,000526 = 0,001502 BTC
   - total_invested = R$ 300
   - average_price = R$ 300 / 0,001502 = R$ 199.733

   → Média CAIU de R$ 204.918 para R$ 199.733
   → DCA "aproveitou" a queda

5. Continua até:
   - max_cycles alcançado (30 compras)
   - allocated_balance esgotado (R$ 3.000 investidos)
   - take_profit atingido (vende e opcionalmente reinicia)
   - stop_loss atingido (vende e para)
```

---

## Frequências Disponíveis

| Frequência | Uso Recomendado | Ciclos/mês | Exemplo |
|------------|-----------------|------------|---------|
| **1h** | Day trading DCA (curto prazo) | ~720 | Acumula rapidamente em mercado volátil |
| **6h** | Swing trading DCA | ~120 | 4 compras por dia |
| **24h** | DCA diário | ~30 | 1 compra por dia (mais comum) |
| **7d** | DCA semanal | ~4 | Investimento mensal distribuído |
| **30d** | DCA mensal | ~12/ano | Investimento anual distribuído |

---

## Take Profit e Stop Loss

### Take Profit (Realização de Lucro)

**Configuração**: `take_profit_percent = 10%`

**Como funciona**:
```
Situação:
- average_price = R$ 199.733 (média das compras)
- current_price = R$ 220.000
- total_quantity = 0,001502 BTC

Cálculo de lucro:
profit_pct = ((220.000 - 199.733) / 199.733) × 100 = 10,14%

→ Lucro >= 10% (take_profit atingido)
→ Bot vende TODA a posição (0,001502 BTC) a mercado
→ Proceeds = 0,001502 × R$ 220.000 = R$ 330,44
→ Realized profit = R$ 330,44 - R$ 300 (invested) = R$ 30,44

Se auto_restart = true:
  → Reseta: total_quantity = 0, average_price = 0, total_invested = 0
  → current_cycle = 0
  → Reinicia acumulação com mesmo budget

Se auto_restart = false:
  → Bot para (status = STOPPED)
  → Usuário deve criar novo bot manualmente
```

**Por que 10%?**
- Menor que 10%: Take profit muito frequente, perde grandes tendências
- Maior que 20%: Demora muito, risco de reversão antes de atingir
- 10-15%: Sweet spot para cripto (volátil o suficiente)

---

### Stop Loss (Proteção de Capital)

**Configuração**: `stop_loss_percent = 5%`

**Como funciona**:
```
Situação:
- average_price = R$ 200.000
- current_price = R$ 190.000
- total_quantity = 0,001 BTC

Cálculo de perda:
loss_pct = ((200.000 - 190.000) / 200.000) × 100 = 5%

→ Prejuízo >= 5% (stop_loss atingido)
→ Bot vende TODA a posição a mercado (urgente)
→ Proceeds = 0,001 × R$ 190.000 = R$ 190
→ Realized loss = R$ 190 - R$ 200 (invested) = -R$ 10

Se auto_restart = true:
  → Reseta e reinicia (mas perdeu R$ 10)

Se auto_restart = false:
  → Bot para definitivamente
```

**Por que stop loss é controverso no DCA?**
- ✅ **Argumento a favor**: Protege contra bear markets prolongados
- ❌ **Argumento contra**: DCA é estratégia de longo prazo, stop loss pode vender no fundo

**Recomendação**:
- **Mercados incertos**: Use stop loss moderado (10-15%)
- **Convicção forte no ativo**: Desabilite stop loss (null)

---

## Gestão de Budget

### Allocated Balance (Budget Total)

**Regra de Negócio**:
```
allocated_balance = purchase_amount × max_cycles

Exemplo:
purchase_amount = R$ 100
max_cycles = 30
allocated_balance = R$ 3.000

→ Bot consegue fazer 30 compras antes de esgotar budget
```

---

### Validações de Budget

```
1. ANTES de criar bot:
   Se allocated_balance < purchase_amount → ERRO
   "Budget insuficiente para ao menos 1 compra"

2. ANTES de cada compra:
   remaining_balance = allocated_balance - total_invested

   Se remaining_balance < purchase_amount:
     → Pausa bot automaticamente
     → Notifica usuário: "Budget esgotado"
     → status = PAUSED
```

---

### Max Cycles vs Allocated Balance

**Cenário 1: Max cycles atingido ANTES de esgotar budget**
```
allocated_balance = R$ 5.000
purchase_amount = R$ 100
max_cycles = 30

Após 30 compras:
- total_invested = R$ 3.000
- remaining_balance = R$ 2.000 (sobrou)
→ Bot para (max_cycles atingido)
→ Usuário pode sacar R$ 2.000 restantes
```

**Cenário 2: Budget esgotado ANTES de max cycles**
```
allocated_balance = R$ 2.500
purchase_amount = R$ 100
max_cycles = 30

Após 25 compras:
- total_invested = R$ 2.500
- remaining_balance = R$ 0
→ Bot pausa (budget esgotado)
→ current_cycle = 25 (não atingiu 30)
```

**Cenário 3: Max cycles = null (ilimitado)**
```
allocated_balance = R$ 10.000
purchase_amount = R$ 100
max_cycles = null

→ Bot compra até esgotar R$ 10.000
→ Máximo de 100 compras teóricas
→ Para apenas quando budget acabar ou take profit/stop loss
```

---

## Order Strategy

### MARKET (Padrão - Recomendado para DCA)

**Como funciona**:
- Cria ordem "a mercado" (executa imediatamente ao melhor preço disponível)
- **Garantia de execução**: 99,9% de chance de executar
- **Desvantagem**: Pode pagar ligeiramente mais caro (slippage)

**Exemplo**:
```
Orderbook:
Melhor ask (venda): R$ 200.000
Segunda melhor: R$ 200.050

Ordem MARKET de R$ 100:
→ Executa a R$ 200.000 (melhor ask)
→ Compra: 0,0005 BTC
→ Garantido
```

**Quando usar**: DCA (prioridade é executar, não preço exato)

---

### LIMIT (Opcional - Para usuários avançados)

**Como funciona**:
- Cria ordem "limitada" ao preço especificado
- **Vantagem**: Pode conseguir preço melhor
- **Desvantagem**: Ordem pode NÃO executar se mercado não atingir o preço

**Exemplo**:
```
Preço atual: R$ 200.000
Cria ordem LIMIT de R$ 100 @ R$ 199.000 (1% abaixo)

Cenário A (mercado cai para R$ 199.000):
→ Ordem executa
→ Comprou 1% mais barato que MARKET

Cenário B (mercado sobe para R$ 205.000):
→ Ordem NÃO executa (ficou "para trás")
→ Perdeu o ciclo de DCA
→ Média de preço impactada
```

**Quando usar**: Mercados muito voláteis onde 1-2% de diferença importam

**Recomendação para DCA**: Use MARKET. Prioridade é executar, não otimizar centavos.

---

## Histórico de Execuções

### DCAExecution (Rastreabilidade Completa)

Cada compra gera registro permanente:

```json
{
  "bot_id": "abc123",
  "cycle": 5,
  "amount": "100.00",
  "quantity": "0.0005",
  "price": "200000.00",
  "timestamp": "2025-01-05T00:00:00Z",
  "status": "FILLED",
  "order_id": "order_xyz"
}
```

**Endpoint**: `GET /api/dca-bots/{id}/executions`

**Benefícios**:
- Auditoria completa (usuário vê todas as compras)
- Cálculo retroativo de média (mesmo se bot for deletado)
- Declaração de imposto de renda (histórico de compras)

**Exemplo de resposta**:
```json
{
  "executions": [
    {
      "cycle": 1,
      "date": "2025-01-01",
      "amount": "R$ 100",
      "price": "R$ 200.000",
      "quantity": "0,0005 BTC"
    },
    {
      "cycle": 2,
      "date": "2025-01-02",
      "amount": "R$ 100",
      "price": "R$ 210.000",
      "quantity": "0,000476 BTC"
    }
  ],
  "summary": {
    "total_cycles": 2,
    "total_invested": "R$ 200",
    "total_quantity": "0,000976 BTC",
    "average_price": "R$ 204.918",
    "current_value": "R$ 220" // Se preço atual = R$ 225.000
  }
}
```

---

## Regras de Negócio Principais

### 1. Validação de Criação

```
ANTES de criar DCA Bot, validar:

✅ allocated_balance >= purchase_amount
   → Mínimo 1 compra possível

✅ allocated_balance <= saldo disponível do usuário
   → Não pode alocar mais que possui

✅ purchase_amount >= mínimo da exchange
   → Mercado Bitcoin: mínimo R$ 10
   → Binance: mínimo R$ 10 (equivalente)

✅ frequency in [1h, 6h, 24h, 7d, 30d]
   → Apenas frequências suportadas

✅ Se take_profit ou stop_loss:
   → take_profit > 0 AND stop_loss > 0
   → take_profit > stop_loss (não faz sentido contrário)

❌ Se qualquer validação falhar → ERRO 400
```

---

### 2. Execução de Compra Periódica

```
Workflow: dca_execution (60s)

Para cada DCA Bot:
  Se status != ACTIVE:
    → Skip (bot pausado ou parado)

  Se now < next_execution:
    → Skip (ainda não chegou a hora)

  Se max_cycles != null AND current_cycle >= max_cycles:
    → Pausa bot
    → Notifica: "Máximo de ciclos atingido"
    → Skip

  remaining = allocated_balance - total_invested
  Se remaining < purchase_amount:
    → Pausa bot
    → Notifica: "Budget esgotado"
    → Skip

  # EXECUTA COMPRA
  current_price = get_ticker(exchange, symbol)
  quantity = purchase_amount / current_price

  Se order_strategy == MARKET:
    order = create_market_order(side="buy", quantity=quantity)
  Senão:
    limit_price = current_price × 0.99  # 1% abaixo
    order = create_limit_order(side="buy", quantity=quantity, price=limit_price)

  # Aguarda fill (MARKET é instantâneo, LIMIT pode demorar)
  filled_price = order.avg_fill_price
  filled_quantity = order.filled_quantity

  # Atualiza bot (recalcula média)
  bot.execute_cycle(quantity=filled_quantity, price=filled_price)

  # Salva execução
  save(DCAExecution(...))

  # Agenda próxima
  bot.calculate_next_execution()  # Soma frequency
```

---

### 3. Monitoramento de Take Profit e Stop Loss

```
Workflow: dca_risk_management (30s)

Para cada DCA Bot com posição aberta:
  Se total_quantity <= 0:
    → Skip (sem posição)

  current_price = get_ticker(exchange, symbol)

  # Calcula lucro/prejuízo atual
  profit_pct = ((current_price - average_price) / average_price) × 100

  # Take Profit
  Se take_profit_percent != null AND profit_pct >= take_profit_percent:
    → Vende TODA a posição (total_quantity)
    → order = create_market_order(side="sell", quantity=total_quantity)
    → proceeds = total_quantity × current_price
    → realized_profit = proceeds - total_invested

    Se auto_restart == true:
      → Reseta: total_quantity = 0, average_price = 0, total_invested = 0, current_cycle = 0
      → status = ACTIVE (continua rodando)
      → Notifica: "Take profit atingido. Lucro: R$ X. Reiniciando acumulação"
    Senão:
      → status = STOPPED
      → Notifica: "Take profit atingido. Bot parado. Lucro: R$ X"

  # Stop Loss
  loss_pct = ((average_price - current_price) / average_price) × 100

  Se stop_loss_percent != null AND loss_pct >= stop_loss_percent:
    → Vende TODA a posição a mercado (urgente)
    → order = create_market_order(side="sell", quantity=total_quantity)
    → proceeds = total_quantity × current_price
    → realized_loss = proceeds - total_invested

    Se auto_restart == true:
      → Reseta e reinicia
      → Notifica: "Stop loss ativado. Prejuízo: R$ X. Reiniciando"
    Senão:
      → status = STOPPED
      → Notifica: "Stop loss ativado. Prejuízo: R$ X. Bot parado"
```

---

### 4. Cálculo de Average Price (Média Ponderada)

```
Fórmula:
average_price = total_invested / total_quantity

Exemplo passo a passo:

Ciclo 1:
  Compra: R$ 100 @ R$ 200.000 = 0,0005 BTC
  total_invested = R$ 100
  total_quantity = 0,0005 BTC
  average_price = 100 / 0,0005 = R$ 200.000 ✓

Ciclo 2:
  Compra: R$ 100 @ R$ 210.000 = 0,000476 BTC
  total_invested = R$ 200 (100 + 100)
  total_quantity = 0,000976 BTC (0,0005 + 0,000476)
  average_price = 200 / 0,000976 = R$ 204.918 ✓

Ciclo 3:
  Compra: R$ 100 @ R$ 190.000 = 0,000526 BTC
  total_invested = R$ 300
  total_quantity = 0,001502 BTC
  average_price = 300 / 0,001502 = R$ 199.733 ✓
```

**Por que ponderada?**
- Compra de R$ 100 @ R$ 190k compra MAIS BTC (0,000526)
- Compra de R$ 100 @ R$ 210k compra MENOS BTC (0,000476)
- Média pondera automaticamente (mais peso para preços menores)

---

## Métricas de Sucesso

### KPIs por Bot
- **Average Price**: Preço médio de compra (quanto menor, melhor)
- **Unrealized P&L**: Lucro não realizado (current_price - average_price)
- **Success Rate**: % de bots que atingem take profit vs stop loss
- **Average Hold Time**: Tempo médio até take profit

### KPIs de Produto
- **DCA Bots Ativos**: Quantidade total
- **Total AUM (Assets Under Management)**: Soma de allocated_balance de todos os bots
- **Monthly Executed Cycles**: Total de compras executadas no mês
- **Average Cycle Value**: R$ médio por compra

---

## Casos de Uso Reais

### Caso 1: Investidor Conservador (Longo Prazo)
**Perfil**: Pedro, 45 anos, investe R$ 500/mês em Bitcoin para aposentadoria

**Configuração**:
```
purchase_amount: R$ 100
frequency: 7d (semanal)
allocated_balance: R$ 6.000 (12 meses)
take_profit: 50% (vende apenas em bull run forte)
stop_loss: null (sem stop - convicção de longo prazo)
max_cycles: null (ilimitado até esgotar budget)
auto_restart: false
```

**Expectativa**:
- 4 compras por mês
- Acumula por 12-15 meses
- Aproveita todas as quedas
- Vende apenas em grande valorização (>50%)

---

### Caso 2: Trader Oportunista (Médio Prazo)
**Perfil**: Ana, 30 anos, quer acumular mas realizar lucros regularmente

**Configuração**:
```
purchase_amount: R$ 200
frequency: 24h (diário)
allocated_balance: R$ 3.000
take_profit: 15% (realiza lucro moderado)
stop_loss: 8% (protege capital)
max_cycles: 15
auto_restart: true (reinveste lucros)
```

**Expectativa**:
- 15 dias de acumulação
- Take profit ativa 2-3x por mês (volatilidade cripto)
- Reinveste lucros automaticamente
- Crescimento composto

---

### Caso 3: Iniciante Cauteloso
**Perfil**: Lucas, 22 anos, primeira vez investindo em cripto

**Configuração**:
```
purchase_amount: R$ 50
frequency: 24h
allocated_balance: R$ 500
take_profit: 10% (conservador)
stop_loss: 5% (proteção)
max_cycles: 10
auto_restart: false (quer avaliar resultado antes de continuar)
```

**Expectativa**:
- 10 dias de teste
- Aprende sobre volatilidade sem grande risco
- Para após 10 compras para avaliar
- Decide se quer continuar baseado em resultado

---

## Problemas Comuns e Soluções

### ❌ Problema: "Budget esgotou antes do esperado"
**Causa**: allocated_balance menor que (purchase_amount × max_cycles)
**Solução**:
- Aumentar allocated_balance, OU
- Reduzir purchase_amount, OU
- Reduzir max_cycles

---

### ❌ Problema: "Média de preço subindo (queria que caísse)"
**Causa**: Mercado em forte alta - cada compra é mais cara que a anterior
**Solução**: DCA não é ideal para bull run vertical. Considerar pausar e retomar em correção.

---

### ❌ Problema: "Take profit nunca ativa"
**Causa**: take_profit_percent muito alto (ex: 50% em mercado lateral)
**Solução**: Reduzir para 10-15% (mais realista para cripto)

---

### ❌ Problema: "Ordem LIMIT não executa"
**Causa**: Preço limite muito agressivo (muito abaixo do mercado)
**Solução**: Mudar para MARKET ou reduzir agressividade (ex: 0,5% abaixo vs 2%)

---

## Limitações e Transparência

### O que DCA NÃO é
- ❌ **Não é garantia de lucro**: Em bear market prolongado, média cai mas continua em prejuízo
- ❌ **Não é timing de mercado**: DCA é anti-timing (compra independente de preço)
- ❌ **Não é melhor que lump sum em bull run**: Se mercado só sobe, comprar tudo no início seria melhor

### Quando DCA é Inferior a Lump Sum
```
Cenário: Bitcoin a R$ 180k em 01/01

Lump Sum: Compra R$ 3.000 @ R$ 180k = 0,01666 BTC
DCA 30 dias: Compra R$ 100/dia durante 30 dias enquanto preço sobe para R$ 220k
  → Acumula menos BTC (média > R$ 180k)

Se preço em 30/01 = R$ 250k:
  Lump Sum: Lucro = (250k - 180k) × 0,01666 = R$ 1.166
  DCA: Lucro menor (média foi R$ 200k, acumulou menos BTC)
```

**Quando usar DCA mesmo assim?**
- Você NÃO sabe se mercado vai subir ou cair
- Prefere menor risco (distribuir compras) vs maior retorno potencial
- Não tem R$ 3.000 de uma vez (DCA permite investir gradualmente)

---

### Transparência para o Usuário

O que o usuário sempre vê:
- ✅ Histórico completo de todas as compras (data, preço, quantidade)
- ✅ Preço médio atualizado em tempo real
- ✅ Lucro/prejuízo não realizado (vs média)
- ✅ Próxima execução programada
- ✅ Budget restante
- ✅ Ciclos executados vs máximo

**Princípio**: Zero surpresas. Usuário sabe exatamente o que bot está fazendo.

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
