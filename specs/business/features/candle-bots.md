---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'bots', 'candle']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Candle Bots (Análise Técnica Automatizada)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Fornecer análise técnica automatizada baseada em indicadores profissionais para trading objetivo sem emoção.

**Constraints** (limites obrigatórios):
- Mínimo 60% de confiança dos indicadores para gerar sinal BUY/SELL (não HOLD)
- Take profit e stop-loss obrigatórios (nunca permitir bot sem proteção)
- Indicadores avançados (VWAP, MFI, OBV, Volume Profile) exclusivos de plano MAX
- Validação de volume mínimo antes de executar ordem (evitar low liquidity)
- Sinais devem ser auditáveis (salvar quais indicadores geraram cada sinal)
- Respeitar limites de timeframe por exchange (MB: 7 timeframes, Binance: 16 timeframes)

**Non-Goals** (o que NÃO fazer):
- Criar indicadores proprietários ou "secretos" (usar apenas indicadores conhecidos)
- Implementar machine learning ou IA para previsão de preços no MVP
- Suportar backtesting histórico (apenas forward testing)
- Adicionar padrões de candlestick complexos (doji, hammer, engulfing) no MVP
- Permitir usuários criarem indicadores customizados via código
- Executar ordens sem validação de sinal (sempre requer confiança >= threshold)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Bots de trading baseados em padrões de candlestick
:::

## Visão Geral

Candle Bots automatizam decisões de trading baseadas em análise técnica profissional de gráficos de preços (velas japonesas). Utilizam 10 indicadores técnicos para identificar oportunidades de compra e venda, eliminando emoções e vieses humanos do processo de trading.

**Status**: ✅ Em Produção
**Planos**: PRO e MAX
**Prioridade**: ALTA

---

## Propósito e Valor

### Para o Usuário
- **Trading Profissional Acessível**: Análise técnica de nível institucional sem necessidade de expertise
- **Decisões Objetivas**: Elimina emoções (medo, ganância) das decisões de trading
- **Análise 24/7**: Monitora mercado continuamente, mesmo quando usuário está dormindo
- **Múltiplas Estratégias**: Suporta day trading, swing trading e position trading
- **Transparência Total**: Todo sinal mostra quais indicadores foram considerados

### Para o Negócio
- **Diferencial Competitivo**: Indicadores avançados (VWAP, MFI) não oferecidos por concorrentes brasileiros
- **Barreira de Saída**: Usuários que configuram estratégias complexas têm menos propensão a cancelar
- **Upsell Natural**: Indicadores avançados exclusivos do plano MAX incentivam upgrade
- **Educação do Mercado**: Usuários aprendem análise técnica usando a plataforma

---

## Conceitos Fundamentais

### O que são Candles (Velas Japonesas)?

Representação visual de preços em um período de tempo:

```
     │  Máxima (high)
     ├──┐
     │  │ Fechamento > Abertura (verde/alta)
     ├──┘
     │  Mínima (low)
```

**Informações em cada candle**:
- **Abertura**: Primeiro preço do período
- **Fechamento**: Último preço do período
- **Máxima**: Preço mais alto atingido
- **Mínima**: Preço mais baixo atingido
- **Volume**: Quantidade negociada

### Timeframes (Períodos de Análise)

Cada candle representa um período de tempo:
- **1m**: Cada vela = 1 minuto (para day trading agressivo)
- **5m, 15m, 30m**: Minutos (day trading)
- **1h, 4h**: Horas (swing trading)
- **1d**: Dia completo (position trading)
- **1w, 1M**: Semana/Mês (investimento de longo prazo)

**Regra Fundamental**: Quanto menor o timeframe, mais sinais mas mais "ruído" (falsos positivos).

---

## Indicadores Técnicos

### Indicadores Padrão (Planos PRO e MAX)

#### 1. RSI (Índice de Força Relativa)
**O que mede**: Se o ativo está "caro demais" (sobrecomprado) ou "barato demais" (sobrevendido).

**Como funciona**:
- Escala de 0 a 100
- **Abaixo de 30**: Ativo oversold (sobrevendido) → Provável reversão de alta → **COMPRAR**
- **Acima de 70**: Ativo overbought (sobrecomprado) → Provável correção → **VENDER**
- **Entre 30-70**: Neutro

**Exemplo Prático**:
```
Bitcoin em R$ 180.000, RSI = 25
→ Ativo muito vendido (apenas 25% de "força compradora")
→ Probabilidade alta de reversão
→ Sinal de COMPRA
```

**Casos de Uso**:
- Identificar fundos de mercado (compra em pânicos)
- Identificar topos (venda em euforia)

---

#### 2. Médias Móveis (MA)
**O que mede**: Tendência de preço ao longo do tempo, filtrando volatilidade de curto prazo.

**Como funciona**:
- **MA Curta** (ex: 9 períodos): Responde rápido a mudanças
- **MA Longa** (ex: 21 períodos): Mostra tendência geral

**Sinais**:
- **Golden Cross**: MA curta cruza MA longa para CIMA → **COMPRAR** (início de tendência de alta)
- **Death Cross**: MA curta cruza MA longa para BAIXO → **VENDER** (início de tendência de baixa)
- Preço acima de ambas MAs → Tendência de alta saudável
- Preço abaixo de ambas MAs → Tendência de baixa

**Exemplo Prático**:
```
MA9 (curta) = R$ 195.000
MA21 (longa) = R$ 198.000

Ontem: MA9 estava abaixo de MA21
Hoje: MA9 cruzou para cima (Golden Cross)
→ Reversão de tendência detectada
→ Sinal de COMPRA
```

---

#### 3. Bandas de Bollinger
**O que mede**: Extremos de preço e volatilidade.

**Como funciona**:
- 3 linhas: Banda Superior, Banda Média (MA), Banda Inferior
- Bandas se expandem em alta volatilidade, contraem em baixa volatilidade

**Sinais**:
- Preço toca **banda inferior** → Ativo "barato" relativo → **COMPRAR**
- Preço toca **banda superior** → Ativo "caro" relativo → **VENDER**
- Bandas muito estreitas → "Calma antes da tempestade" → Grande movimento iminente

**Exemplo Prático**:
```
Banda Superior: R$ 205.000
Banda Média: R$ 200.000
Banda Inferior: R$ 195.000
Preço Atual: R$ 194.500

→ Preço tocou banda inferior (extremo)
→ Sinal de COMPRA (reversão provável)
```

---

#### 4. Volume
**O que mede**: Força e convicção por trás de movimentos de preço.

**Como funciona**:
- **Volume alto + preço subindo** → Movimento forte (muitos compradores) → **Confirma COMPRA**
- **Volume alto + preço caindo** → Movimento forte (muitos vendedores) → **Confirma VENDA**
- **Volume baixo** → Movimento fraco → Ignorar sinal (pode ser falso)

**Exemplo Prático**:
```
Situação 1:
Preço sobe 5% com volume 200% acima da média
→ Movimento forte e sustentável → COMPRAR

Situação 2:
Preço sobe 5% com volume 50% abaixo da média
→ Movimento fraco (poucos compradores) → NÃO comprar (provável armadilha)
```

**Princípio**: "Volume confirma preço". Movimentos sem volume são suspeitos.

---

#### 5. Volatilidade
**O que mede**: Quanto o preço varia (quão "nervoso" está o mercado).

**Como funciona**:
- **Alta volatilidade**: Mercado agitado, movimentos grandes e rápidos
- **Baixa volatilidade**: Mercado calmo, movimentos pequenos

**Uso no Bot**:
- Ajusta automaticamente **stop loss** (maior em alta volatilidade)
- Ajusta automaticamente **take profit** (maior em alta volatilidade)
- Usado no **Adaptive Sell** (vende mais cedo em mercado calmo, aguarda mais em mercado volátil)

---

### Indicadores Avançados (Exclusivos Plano MAX + Binance)

#### 6. VWAP (Preço Médio Ponderado por Volume)
**O que mede**: Preço "justo" baseado em quanto volume foi negociado em cada nível de preço.

**Como funciona**:
- Usado por traders institucionais como referência
- **Preço acima do VWAP** → Compradores estão pagando "premium" → Tendência de alta
- **Preço abaixo do VWAP** → Compradores estão com "desconto" → Oportunidade de compra

**Exemplo Prático**:
```
VWAP do dia: R$ 198.000
Preço Atual: R$ 200.500

→ Preço 1,26% acima do VWAP
→ Compradores pagando premium
→ Tendência de alta confirmada
```

**Por que é valioso**: Instituições usam VWAP para avaliar se fizeram boas negociações. Se preço está consistentemente acima do VWAP, há pressão compradora forte.

---

#### 7. MFI (Índice de Fluxo Monetário)
**O que mede**: RSI "turbinado" que considera volume junto com preço.

**Como funciona**:
- Escala 0-100 (como RSI)
- **Abaixo de 20**: Oversold COM volume → **COMPRA** (mais confiável que RSI)
- **Acima de 80**: Overbought COM volume → **VENDA**

**Vantagem sobre RSI**:
```
Cenário 1 - RSI sozinho:
RSI = 25 (oversold)
Volume baixo
→ Sinal fraco (poucos vendedores, pode cair mais)

Cenário 2 - MFI:
MFI = 18 (oversold COM volume alto)
→ Muita gente vendendo em pânico
→ Sinal FORTE de compra (provável reversão)
```

---

#### 8. Volume Profile
**O que mede**: Em quais níveis de preço houve MAIS negociação (suporte/resistência baseado em volume).

**Como funciona**:
- Identifica **POC (Point of Control)**: Preço com maior volume negociado
- POC atua como "ímã" → Preço tende a retornar ao POC

**Exemplo Prático**:
```
POC = R$ 200.000 (70% do volume do dia foi negociado aqui)
Preço Atual = R$ 195.000

→ Preço está R$ 5.000 abaixo do POC
→ Alta probabilidade de retorno ao POC
→ Sinal de COMPRA (alvo R$ 200.000)
```

**Uso**: Identifica zonas de "conforto" do mercado (onde mais pessoas concordam que é preço justo).

---

#### 9. ATR (Average True Range)
**O que mede**: Volatilidade média em valores absolutos (R$).

**Como funciona**:
- **ATR alto** → Mercado volátil → Precisa stop loss mais largo
- **ATR baixo** → Mercado calmo → Pode usar stop loss mais apertado

**Exemplo Prático**:
```
ATR = R$ 3.000 (Bitcoin varia em média R$ 3k por dia)
Compra @ R$ 200.000

Stop Loss inteligente = R$ 200.000 - (3.000 × 1,5) = R$ 195.500
→ Stop 1,5× ATR (evita ser "parado" por oscilação normal)

Se ATR fosse R$ 1.000 (mercado calmo):
Stop Loss = R$ 200.000 - (1.000 × 1,5) = R$ 198.500
→ Stop mais apertado (mercado calmo não precisa tanto espaço)
```

---

#### 10. OBV (On Balance Volume)
**O que mede**: Pressão acumulada de compra vs venda ao longo do tempo.

**Como funciona**:
- Soma volume quando preço sobe, subtrai quando preço cai
- **Divergências** são poderosas:
  - Preço caindo MAS OBV subindo → Acumulação escondida → Reversão de alta provável
  - Preço subindo MAS OBV caindo → Distribuição escondida → Correção provável

**Exemplo Prático (Divergência Bullish)**:
```
Últimos 5 dias:
Preço: R$ 210k → R$ 205k → R$ 202k → R$ 198k → R$ 195k (caindo)
OBV: 1.000 → 1.200 → 1.500 → 1.800 → 2.100 (subindo!)

→ Preço cai mas volume comprador MAIOR que vendedor
→ "Smart money" está acumulando discretamente
→ Sinal FORTE de COMPRA (reversão iminente)
```

---

## Geração de Sinais - Como o Bot Decide

### Princípio de Convergência

O bot **NÃO decide** baseado em apenas 1 indicador. Ele busca **convergência** (múltiplos indicadores apontando mesma direção).

**Exemplo de Sinal de COMPRA com convergência HIGH**:
```
✅ RSI = 28 (oversold)
✅ Preço tocou banda inferior de Bollinger
✅ MA9 cruzou MA21 para cima (Golden Cross)
✅ Volume 180% acima da média (confirma movimento)
✅ VWAP = preço está 2% abaixo (desconto)
✅ MFI = 19 (oversold COM volume)

Convergência: 6 de 6 indicadores apontam COMPRA
→ Confidence: HIGH
→ Bot executa automaticamente (se auto_execute = true)
```

**Exemplo de Sinal CONFLITANTE (HOLD)**:
```
✅ RSI = 32 (leve oversold) → Sugere compra
❌ Preço acima de banda superior de Bollinger → Sugere venda
❌ MA9 abaixo de MA21 (Death Cross ativo) → Sugere venda
➖ Volume normal (não confirma nada)

Convergência: 1 compra vs 2 venda
→ Sinais conflitantes
→ Confidence: LOW
→ Bot emite sinal HOLD (não faz nada)
```

---

### Níveis de Confiança (Confidence Score)

| Confidence | Critério | Ação do Bot | Uso Recomendado |
|------------|----------|-------------|-----------------|
| **HIGH** | ≥70% dos indicadores concordam | Auto-executa (se habilitado) | Estratégias agressivas |
| **MEDIUM** | 60-69% concordam | Auto-executa (configurável) | Estratégias balanceadas |
| **LOW** | 50-59% concordam | Apenas notifica usuário | Estratégias conservadoras |

**Regra de Negócio**: Usuário pode configurar `min_confidence_score` para controlar quantos sinais serão executados:
- `min_confidence = HIGH`: Apenas sinais muito fortes → Menos trades, maior precisão
- `min_confidence = MEDIUM`: Balance → Quantidade moderada de trades
- `min_confidence = LOW`: Todos os sinais → Mais trades, mais falsos positivos

---

## Gestão de Risco

### Take Profit Automático
**O que é**: Vende automaticamente quando lucro atinge meta configurada.

**Configuração**:
```
take_profit_pct = 5%
```

**Como funciona**:
```
Comprou @ R$ 200.000
Preço atual = R$ 210.000
Lucro = 5%

→ Take profit ativado
→ Bot vende TODA a posição automaticamente
→ Lucro realizado: R$ 10.000 (por 1 BTC)
```

**Por que é importante**: Protege contra reversões. Melhor garantir 5% do que esperar 10% e acabar com 0%.

---

### Stop Loss Automático
**O que é**: Vende automaticamente quando prejuízo atinge limite.

**Configuração**:
```
stop_loss_pct = 3%
```

**Como funciona**:
```
Comprou @ R$ 200.000
Preço atual = R$ 194.000
Prejuízo = 3%

→ Stop loss ativado
→ Bot vende a mercado (urgência)
→ Prejuízo limitado: R$ 6.000 (por 1 BTC)
```

**Por que é importante**: "Corta perdas cedo, deixa lucros correrem". Evita perdas catastróficas.

---

### Trailing Stop (Proteção de Lucros)
**O que é**: Stop loss "inteligente" que sobe junto com o preço (mas nunca desce).

**Configuração**:
```
trailing_stop_pct = 3%
```

**Exemplo completo**:
```
1. Compra @ R$ 200.000
   Stop loss inicial = R$ 194.000 (3% abaixo)

2. Preço sobe para R$ 210.000 (+5%)
   Trailing stop ajusta: R$ 210.000 × 0.97 = R$ 203.700
   → Stop agora garante lucro mínimo de R$ 3.700

3. Preço sobe para R$ 220.000 (+10%)
   Trailing stop ajusta: R$ 220.000 × 0.97 = R$ 213.400
   → Stop agora garante lucro mínimo de R$ 13.400

4. Preço cai para R$ 215.000
   Stop mantém: R$ 213.400 (não cai, apenas sobe)

5. Preço cai para R$ 213.000
   → Trailing stop ativado (preço caiu abaixo de R$ 213.400)
   → Vende automaticamente
   → Lucro realizado: R$ 13.000 (vs R$ 0 se não tivesse trailing stop)
```

**Benefício**: Captura grandes tendências de alta sem devolver lucros em correções.

---

## Timeframes - Escolhendo a Estratégia Certa

### Day Trading (1m - 30m)
**Perfil**: Trader ativo, disponível durante dia, busca lucros pequenos frequentes

**Configuração Recomendada**:
```
Timeframe: 5m ou 15m
min_confidence_score: HIGH (para filtrar ruído)
take_profit_pct: 1-2%
stop_loss_pct: 0.5-1%
Indicadores principais: RSI, MA, Volume
```

**Expectativa**:
- 5-15 sinais por dia
- 60-70% de acerto (com confidence HIGH)
- Lucro médio por trade: 1-2%

---

### Swing Trading (1h - 4h)
**Perfil**: Trader com emprego, verifica bot 2-3x ao dia, busca lucros médios

**Configuração Recomendada**:
```
Timeframe: 1h
min_confidence_score: MEDIUM
take_profit_pct: 3-5%
stop_loss_pct: 2-3%
Indicadores: RSI, MA, Bollinger, VWAP, MFI
```

**Expectativa**:
- 1-3 sinais por dia
- 65-75% de acerto
- Lucro médio por trade: 3-5%

---

### Position Trading (1d - 1w)
**Perfil**: Investidor de longo prazo, verifica bot semanalmente

**Configuração Recomendada**:
```
Timeframe: 1d
min_confidence_score: HIGH
take_profit_pct: 10-20%
stop_loss_pct: 5-8%
Indicadores: Todos (convergência máxima)
```

**Expectativa**:
- 1-2 sinais por semana
- 70-80% de acerto
- Lucro médio por trade: 10-20%

---

## Regras de Negócio Principais

### 1. Validação de Plano
```
PRO: Acesso a indicadores padrão (RSI, MA, BB, Volume, Volatility)
MAX: Acesso a TODOS os indicadores (padrão + VWAP, MFI, Volume Profile, ATR, OBV)

Se usuário PRO tentar habilitar MFI → ERRO
```

### 2. Limite de Bots por Contexto
```
FREE: 1 Candle Bot por contexto
PRO: 3 Candle Bots por contexto
MAX: Ilimitado
```

### 3. Auto-Execução Requer Saldo
```
Se sinal = COMPRA mas saldo insuficiente:
  → Gera sinal (histórico)
  → Não executa ordem
  → Notifica usuário: "Saldo insuficiente"
```

### 4. Sincronização de Balance
```
Cada ordem executada pelo Candle Bot é rastreada separadamente
Balance tracking por estratégia: "candle" vs "dca" vs "smart"
Permite P&L isolado por estratégia
```

### 5. Minimum Profit
```
minimum_profit_pct = 0,5% (configurável)

Se lucro atual = 0,3% → NÃO vende (abaixo do mínimo)
Se lucro atual = 0,6% → Vende (se take_profit também atingido)
```

**Razão**: Evita vendas por lucros insignificantes (fees podem consumir lucro).

---

## Métricas de Sucesso

### KPIs por Bot
- **Win Rate**: % de trades lucrativos (meta: > 65%)
- **Profit Factor**: Total de lucros / Total de perdas (meta: > 1,5)
- **Average Profit per Trade**: R$ lucro médio (meta: > R$ 10)
- **Max Drawdown**: Maior queda de capital (meta: < 15%)

### KPIs de Produto
- **Signal Accuracy**: % de sinais HIGH que resultam em lucro (meta: > 75%)
- **Bots ativos com Candle Strategy**: Quantidade (crescimento mensal)
- **Upgrade Rate PRO → MAX**: % de usuários PRO que fazem upgrade para usar indicadores avançados

---

## Casos de Uso Reais

### Caso 1: Trader Iniciante (Plano PRO)
**Perfil**: João, 28 anos, trabalha durante o dia, quer lucro passivo

**Configuração**:
- Timeframe: 1h (swing trading)
- Indicadores: RSI + MA + Bollinger (simples)
- min_confidence: HIGH (conservador)
- auto_execute: true
- take_profit: 5%, stop_loss: 3%

**Resultado esperado**:
- 1-2 trades por dia
- Win rate: 70%
- Lucro mensal: 15-25% (composto)

---

### Caso 2: Trader Avançado (Plano MAX)
**Perfil**: Maria, 35 anos, trader profissional, quer maximizar lucros

**Configuração**:
- Timeframe: 15m (day trading)
- Indicadores: TODOS habilitados
- min_confidence: MEDIUM (aceita mais sinais)
- auto_execute: true
- take_profit: 2%, stop_loss: 1%
- trailing_stop: true

**Resultado esperado**:
- 5-10 trades por dia
- Win rate: 65%
- Lucro mensal: 30-50% (alta frequência)

---

## Problemas Comuns e Soluções

### ❌ Problema: "Bot gera muitos sinais HOLD"
**Causa**: Mercado lateral (sem tendência), indicadores conflitantes
**Solução**:
- Pausar bot temporariamente (esperar tendência)
- Reduzir quantidade de indicadores (menos conflitos)
- Aumentar min_confidence para HIGH

---

### ❌ Problema: "Stop loss ativado com frequência"
**Causa**: Volatilidade alta ou stop loss muito apertado
**Solução**:
- Aumentar stop_loss_pct (ex: 3% → 5%)
- Usar timeframe maior (ex: 1h → 4h)
- Habilitar trailing stop (protege lucros sem stop apertado)

---

### ❌ Problema: "Win rate baixo (<50%)"
**Causa**: Timeframe muito curto (muito ruído) ou indicadores inadequados
**Solução**:
- Aumentar timeframe (ex: 5m → 1h)
- Habilitar Volume (confirma movimentos)
- Aumentar min_confidence para HIGH
- Revisar período de análise (lookback_periods)

---

## Limitações e Transparência

### O que Candle Bots NÃO fazem
- ❌ **Não preveem eventos futuros** (notícias, regulações)
- ❌ **Não garantem lucro** (análise técnica tem taxa de acerto, não certeza)
- ❌ **Não reagem instantaneamente** (análise a cada 60s, não real-time)
- ❌ **Não funcionam bem em mercados laterais** (precisam de tendência)

### Quando NÃO usar Candle Bots
- Mercados com baixíssimo volume (sinais não confiáveis)
- Períodos de alta incerteza (eventos macroeconômicos)
- Mercados manipulados (pump & dump)

### Transparência para o Usuário
Todo sinal mostra:
- ✅ Quais indicadores foram calculados
- ✅ Valores exatos de cada indicador
- ✅ Por que sinal foi BUY/SELL/HOLD
- ✅ Confidence score e como foi calculado
- ✅ Se foi executado automaticamente ou não

**Princípio**: Usuário sempre sabe "por que o bot fez isso".

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
