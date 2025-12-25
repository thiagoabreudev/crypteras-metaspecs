---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'breaker', 'circuit']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Circuit Breaker (Proteção Automática de Risco)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Proteger capital do usuário pausando bots automaticamente quando detectadas condições de trading perigosas.

**Constraints** (limites obrigatórios):
- Ativar após N falhas consecutivas (default: 3, configurável 2-5)
- Pausar bot imediatamente (status = PAUSED, não apenas flag)
- Notificar usuário via email/dashboard quando circuit breaker ativa
- Requerer ação humana para reativar (não reset automático)
- Registrar timestamp de ativação e razão (qual threshold foi violado)
- Resetar automaticamente à meia-noite se usuário não agir (opcional, configurável)

**Non-Goals** (o que NÃO fazer):
- Implementar circuit breaker global do sistema (apenas por bot individual)
- Cancelar ordens pendentes automaticamente (apenas pausa novas ordens)
- Ativar por perda percentual isolada (usar apenas contagem de falhas consecutivas)
- Criar níveis múltiplos de circuit breaker (hard/soft)
- Implementar cooldown period obrigatório antes de permitir reativação
- Enviar SMS ou notificação push (apenas email e dashboard no MVP)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Sistema de circuit breaker para proteção de risco
:::

## Visão Geral

Sistema de segurança automatizado que **pausa bots automaticamente** quando detecta condições de trading perigosas. Funciona como um "disjuntor" elétrico: quando há sobrecarga (perdas excessivas), desliga o sistema para evitar danos maiores.

**Status**: ✅ Em Produção
**Tickets**: CRY-13, CRY-71
**Disponível para**: SmartBots (opcional, configurável)

---

## Propósito e Valor

### Para o Usuário
- **Proteção Automática**: Para trading quando perdas atingem limites perigosos
- **Paz de Espírito**: Dorme tranquilo sabendo que bot não terá perdas ilimitadas
- **Tempo para Analisar**: For human: forced pause → investigates → fixes strategy → resumes
- **Elimina Emoção**: Remove hesitação humana de "apertar o botão vermelho"

### Para o Negócio
- **Reduz Churn**: Usuários não perdem toda a carteira em um dia ruim
- **Aumenta Confiança**: Demonstra compromisso com gestão de risco
- **Reduz Tickets de Suporte**: Menos reclamações de "bot perdeu tudo"
- **Compliance**: Mostra controles de risco para possíveis reguladores

### Problema que Resolve

**Sem Circuit Breaker**:
```
Bot SmartBTC perde 3% em trade 1
  → Tenta recuperar, perde mais 2% em trade 2
  → Tenta novamente, perde 3.5% em trade 3
  → Continua... perde 2.8% em trade 4
  → Continua... perde 4% em trade 5
  → Continua... perde 3.2% em trade 6
  → Total: -18.5% de perda antes do usuário perceber
```

**Com Circuit Breaker**:
```
Bot SmartBTC perde 3% em trade 1 (1/5 falhas consecutivas)
  → Perde 2% em trade 2 (2/5)
  → Perde 3.5% em trade 3 (3/5)
  → Perde 2.8% em trade 4 (4/5)
  → Perde 4% em trade 5 (5/5) → ⛔ CIRCUIT BREAKER ACIONADO
  → Bot PAUSADO automaticamente
  → Total: -15.3% (vs -18.5%)
  → Usuário é alertado: "Circuit breaker ativado após 5 falhas consecutivas"
  → Usuário investiga: "Ah, spread está 5% devido a baixa liquidez"
  → Usuário corrige configuração e retoma
```

---

## Funcionalidades Principais

### 1. Condições de Ativação (Duplo Gatilho)

O Circuit Breaker aciona quando **QUALQUER** destas condições é atingida:

#### Gatilho 1: Perda Diária Máxima

**Regra**: Pausa bot quando perda acumulada do dia atinge limite configurado

**Configuração**:
- **Parâmetro**: `circuit_breaker_max_daily_loss_pct`
- **Valor Padrão**: `10%` (personalizável: 5%, 15%, 20%)
- **Medição**: Percentual sobre capital total alocado

**Como Funciona**:
```
Capital inicial: R$ 10.000
Limite configurado: 10%

Perda Máxima Permitida = R$ 10.000 × 10% = R$ 1.000

Trade 1: -R$ 300 (3%) → Acumulado: -R$ 300 → ✅ Continua
Trade 2: -R$ 250 (2.5%) → Acumulado: -R$ 550 → ✅ Continua
Trade 3: -R$ 400 (4%) → Acumulado: -R$ 950 → ✅ Continua
Trade 4: -R$ 120 (1.2%) → Acumulado: -R$ 1.070 → ⛔ PARA (10.7% > 10%)

Bot automaticamente PAUSADO
Usuário recebe alerta: "Circuit breaker ativado: perda diária de 10.7% excedeu limite de 10%"
```

**Exemplo Real**:
```
Maria configura DCABot com R$ 5.000
  - Limite diário: 8%
  - Perda máxima: R$ 400

Dia de crash do mercado:
  - 09:00 - Compra BTC a R$ 200k (ordem executada)
  - 09:30 - Preço cai para R$ 195k (perda não realizada: -2.5%)
  - 10:00 - Compra mais BTC a R$ 195k (média de preço reduzida)
  - 10:30 - Preço cai para R$ 188k (perda acumulada: -6%)
  - 11:00 - Compra mais BTC a R$ 188k
  - 11:30 - Preço cai para R$ 183k (perda acumulada: -8.5%)

⛔ Circuit Breaker acionado (8.5% > 8%)

Bot pausado antes de compra #4
Maria economiza ~R$ 100 adicionais que perderia em mais compras
```

---

#### Gatilho 2: Perdas Consecutivas

**Regra**: Pausa bot após N trades com prejuízo seguidos (sem intervalos de lucro)

**Configuração**:
- **Parâmetro**: `circuit_breaker_threshold`
- **Valor Padrão**: `5` perdas consecutivas
- **Range Recomendado**: 3 a 8 (dependendo do perfil de risco)

**Como Funciona**:
```
Threshold configurado: 5 perdas consecutivas

Trade #1: Venda com lucro +2% → Contador: 0 (reset)
Trade #2: Compra mal executada -1% → Contador: 1
Trade #3: Compra com slippage -0.8% → Contador: 2
Trade #4: Venda com lucro +1.5% → Contador: 0 (RESET - lucro quebra sequência)
Trade #5: Compra -1.2% → Contador: 1 (recomeça)
Trade #6: Compra -0.9% → Contador: 2
Trade #7: Compra -1.5% → Contador: 3
Trade #8: Compra -1.1% → Contador: 4
Trade #9: Compra -1.3% → Contador: 5 → ⛔ THRESHOLD ATINGIDO

Bot automaticamente PAUSADO
Mensagem: "Circuit breaker ativado: 5 perdas consecutivas (threshold: 5)"
```

**Importante**: Um único trade lucrativo **reseta o contador para zero**

**Exemplo Real**:
```
João configura SmartBot conservador:
  - Threshold: 3 perdas consecutivas (baixa tolerância)

Manhã volátil:
  - 08:00 - Compra BTC, spread alargado, perda -1.5% (1/3)
  - 08:15 - Venda automática (stop loss), perda -2% (2/3)
  - 08:30 - Compra novamente, slippage alto, perda -1.8% (3/3)

⛔ Circuit Breaker acionado após 3ª perda

Bot pausado
João recebe notificação no celular
João analisa: "Spread está 3% devido a evento de notícia"
João decide aguardar mercado normalizar antes de retomar
```

---

### 2. Comportamento Quando Acionado

#### O Que Acontece Automaticamente

**Sequência de Ações** (milissegundos):
```
1. Sistema detecta threshold atingido
   ↓
2. Registra timestamp de acionamento (circuit_breaker_triggered_at)
   ↓
3. Muda status do bot: ACTIVE → PAUSED
   ↓
4. Grava log de auditoria:
   {
     "action": "SMART_BOT_PAUSED_CIRCUIT_BREAKER",
     "bot_id": "...",
     "consecutive_failures": 5,
     "threshold": 5,
     "timestamp": "2025-01-24T15:30:45Z"
   }
   ↓
5. Workflows param de processar este bot:
   - Purchase workflow: SKIP (bot pausado)
   - Sales workflow: SKIP (bot pausado)
   - Check situation: Identifica "circuit breaker ativo"
```

**Visualmente no Dashboard**:
```
ANTES:
┌──────────────────────────────────────┐
│ SmartBot BTC                         │
│ Status: 🟢 ATIVO                     │
│ Últimas 24h: -8.5%                   │
│ Perdas consecutivas: 4/5             │
└──────────────────────────────────────┘

DEPOIS (circuit breaker acionado):
┌──────────────────────────────────────┐
│ SmartBot BTC                         │
│ Status: 🟡 PAUSADO                   │
│ ⚠️ Circuit Breaker Ativado           │
│ Motivo: 5 perdas consecutivas        │
│ Acionado em: 24/01 15:30             │
│ [RETOMAR] [ARQUIVAR]                 │
└──────────────────────────────────────┘
```

---

#### Notificações ao Usuário

**In-App (Dashboard)**:
```
┌─────────────────────────────────────────────────────┐
│ ⚠️ ALERTA: Circuit Breaker Ativado                  │
├─────────────────────────────────────────────────────┤
│ Bot: SmartBot BTC-BRL Binance                       │
│ Motivo: 5 perdas consecutivas (limite: 5)          │
│ Horário: 24/01/2025 15:30:45                        │
│                                                      │
│ O bot foi pausado automaticamente para proteger     │
│ seu capital. Revise a configuração antes de         │
│ retomar trading.                                     │
│                                                      │
│ [Ver Histórico] [Configurações] [Retomar]          │
└─────────────────────────────────────────────────────┘
```

**Email** (opcional, configurável):
```
Assunto: ⚠️ Circuit Breaker Ativado - SmartBot BTC-BRL

Olá João,

Seu bot "SmartBot BTC-BRL Binance" foi pausado automaticamente
às 15:30 devido a circuit breaker ativado.

Detalhes:
- Perdas consecutivas: 5 (limite: 5)
- Perda diária: -8.5% (limite: -10%)
- Última atividade: 24/01/2025 15:30:45

Próximos passos:
1. Revise o histórico de trades no dashboard
2. Analise condições de mercado
3. Ajuste configurações se necessário
4. Retome o bot quando pronto

Acesse o dashboard: https://crypteras.tech/dashboard

--
Crypteras Trading System
```

**Push Notification** (mobile, futuro):
```
Crypteras 🔔
Bot SmartBTC pausado (circuit breaker)
5 perdas consecutivas detectadas
Toque para ver detalhes
```

---

### 3. Processo de Recuperação (Reset Manual)

#### Passo a Passo para Retomar

**Etapa 1: Investigação**
```
Usuário acessa Dashboard → Bots → SmartBot BTC (PAUSADO)
  ↓
Vê estatísticas:
  - Perdas consecutivas: 5
  - Perda diária: -8.5%
  - Última atividade: 15:30
  - Histórico últimos 10 trades:
    ❌ -1.5% (15:25)
    ❌ -1.2% (15:20)
    ❌ -1.8% (15:15)
    ❌ -2.0% (15:10)
    ❌ -2.0% (15:05) ← Aqui começou o problema
    ✅ +1.2% (15:00)
    ✅ +0.8% (14:55)
  ↓
Conclusão: "Spread alargou às 15:05, causando 5 compras ruins seguidas"
```

**Etapa 2: Correção**
```
Opção A: Ajustar Threshold
  - Era: circuit_breaker_threshold = 5
  - Novo: circuit_breaker_threshold = 7
  - Razão: "Mercado volátil, permitir mais tentativas"

Opção B: Ajustar Limite Diário
  - Era: max_daily_loss = 10%
  - Novo: max_daily_loss = 15%
  - Razão: "Estratégia DCA aceita mais volatilidade"

Opção C: Ajustar Estratégia
  - Era: purchase_interval = 60s (compra a cada 1min)
  - Novo: purchase_interval = 300s (compra a cada 5min)
  - Razão: "Espaçar compras reduz impacto de volatilidade curta"

Opção D: Aguardar Normalização
  - Não altera configuração
  - Apenas aguarda mercado estabilizar
  - Retoma quando spread voltar ao normal
```

**Etapa 3: Retomada**
```
Usuário clica botão "RETOMAR"
  ↓
API: POST /api/smart-bots/{bot_id}/resume
  ↓
Sistema:
  1. Valida bot pertence ao usuário
  2. Muda status: PAUSED → ACTIVE
  3. Atualiza timestamp (updated_at)
  4. NÃO reseta contador de falhas consecutivas (permanece em 5)
  ↓
Próximo workflow (purchase ou sales):
  1. Verifica: bot.status == ACTIVE? → SIM
  2. Executa próximo trade
  3. Se trade for bem-sucedido:
     - Chama bot.record_purchase_success()
     - Reseta consecutive_failures = 0 ✅
  4. Se trade falhar:
     - consecutive_failures = 6
     - Circuit breaker aciona novamente (6 > 5)
```

**Comportamento Importante**:
- ⚠️ **Contador NÃO reseta ao clicar "Retomar"**
- ✅ **Contador só reseta quando próximo trade for bem-sucedido**
- 🎯 **Isso força usuário a corrigir problema antes de retomar**

**Exemplo de Recuperação Bem-Sucedida**:
```
Estado ao pausar:
  - status = PAUSED
  - consecutive_failures = 5
  - circuit_breaker_triggered_at = 15:30

Usuário retoma às 16:00:
  - status = ACTIVE
  - consecutive_failures = 5 (ainda!)

Próximo trade (16:05):
  - Bot tenta comprar BTC
  - Ordem executada com sucesso (spread voltou ao normal)
  - bot.record_purchase_success() é chamado
  - consecutive_failures = 0 ✅ (RESET)

Estado final:
  - status = ACTIVE
  - consecutive_failures = 0
  - Bot está "saudável" novamente
```

---

### 4. Configuração Personalizada

#### Níveis de Proteção Recomendados

**Perfil Conservador** (iniciantes, capital pequeno):
```json
{
  "enable_circuit_breaker": true,
  "circuit_breaker_threshold": 3,
  "circuit_breaker_max_daily_loss_pct": "5.0"
}
```
- Para após 3 perdas consecutivas OU 5% de perda diária
- Ideal para: Traders iniciantes, capital < R$ 5.000
- Proteção máxima

**Perfil Moderado** (intermediário, padrão):
```json
{
  "enable_circuit_breaker": true,
  "circuit_breaker_threshold": 5,
  "circuit_breaker_max_daily_loss_pct": "10.0"
}
```
- Para após 5 perdas consecutivas OU 10% de perda diária
- Ideal para: Maioria dos usuários PRO
- Balanceado risco/recompensa

**Perfil Agressivo** (avançado, alta tolerância):
```json
{
  "enable_circuit_breaker": true,
  "circuit_breaker_threshold": 8,
  "circuit_breaker_max_daily_loss_pct": "15.0"
}
```
- Para após 8 perdas consecutivas OU 15% de perda diária
- Ideal para: Traders experientes, capital > R$ 50.000, estratégias DCA
- Aceita mais volatilidade

**Desativado** (⚠️ NÃO RECOMENDADO):
```json
{
  "enable_circuit_breaker": false
}
```
- Circuit breaker desligado completamente
- Bot pode operar sem limites
- ⚠️ **Risco**: Perdas ilimitadas possíveis
- Use apenas em paper trading ou com stop loss manual rigoroso

---

#### Quando Ajustar Configurações

**Aumentar Threshold** (permitir mais tentativas):
- Market é naturalmente volátil (criptomoedas)
- Estratégia DCA precisa de mais resiliência
- Spread ocasionalmente alarga (baixa liquidez)

**Reduzir Threshold** (proteção mais estrita):
- Capital pequeno (cada trade representa % significativo)
- Trader iniciante (preferir segurança)
- Testando nova estratégia

**Aumentar Limite Diário**:
- Estratégia de longo prazo (aceita flutuações diárias)
- Capital grande (% pequenos = valores absolutos altos)
- Mercado historicamente volátil (ex: altcoins)

**Reduzir Limite Diário**:
- Capital pequeno ou emprestado
- Trader iniciante
- Mercado desconhecido

---

## Casos de Uso Reais

### Caso 1: DCA Bot em Mercado em Queda

**Persona**: Maria - Iniciante, R$ 3.000 em DCABot

**Cenário**:
```
Maria configura DCABot conservador:
  - Compra: R$ 100 a cada 6h
  - Threshold: 4 perdas consecutivas
  - Limite diário: 8%

Segunda-feira (mercado em queda):
  00:00 - Compra #1: BTC a R$ 200k (OK)
  06:00 - Compra #2: BTC a R$ 198k (perda não realizada: -1%, 1/4)
  12:00 - Compra #3: BTC a R$ 195k (perda não realizada: -2.5%, 2/4)
  18:00 - Compra #4: BTC a R$ 192k (perda não realizada: -4%, 3/4)

Terça-feira:
  00:00 - Compra #5: BTC a R$ 188k (perda não realizada: -6%, 4/4)

  ⛔ Circuit Breaker acionado (4 perdas consecutivas)

  Bot pausado automaticamente
  Maria recebe notificação: "Bot pausado após 4 perdas consecutivas"

Maria investiga:
  - Vê que BTC caiu 6% em 24h (queda acentuada)
  - Decide aguardar estabilização
  - Aguarda 2 dias

Quinta-feira (mercado estabilizado):
  - BTC voltou para R$ 195k
  - Maria clica "Retomar"
  - Próxima compra às 00:00 sexta-feira
  - Compra #6: BTC a R$ 195k → Ordem executada com sucesso ✅
  - consecutive_failures reseta para 0
  - Bot volta ao normal
```

**Resultado**:
- Circuit breaker evitou mais 2-3 compras durante queda acentuada
- Maria economizou ~R$ 200-300 em compras ruins
- Retomou trading quando mercado estabilizou

---

### Caso 2: SmartBot Durante Flash Crash

**Persona**: João - Intermediário, R$ 10.000 em SmartBot

**Cenário**:
```
João roda SmartBot agressivo:
  - Mode: CONTINUOUS (compra a cada 2min)
  - Threshold: 5 perdas consecutivas
  - Limite diário: 10%

Manhã normal:
  08:00 - Compra + venda com +1.2% ✅
  08:05 - Compra + venda com +0.8% ✅
  08:10 - Compra + venda com +1.5% ✅

  Lucro acumulado: +3.5%

08:15 - FLASH CRASH (notícia negativa sobre regulação)
  → BTC cai 8% em 10 minutos

  08:15 - Compra a R$ 200k, vende (stop loss) a R$ 196k: -2% ❌ (1/5)
  08:17 - Compra a R$ 196k, vende (stop loss) a R$ 192k: -2% ❌ (2/5)
  08:19 - Compra a R$ 192k, vende (stop loss) a R$ 188k: -2% ❌ (3/5)
  08:21 - Compra a R$ 188k, vende (stop loss) a R$ 184k: -2% ❌ (4/5)
  08:23 - Compra a R$ 184k, vende (stop loss) a R$ 182k: -1% ❌ (5/5)

  ⛔ Circuit Breaker acionado (5 perdas consecutivas)

  Perda acumulada: -9% (ainda dentro do limite de 10%)
  Lucro do dia: +3.5% (antes) - 9% (crash) = -5.5% (total do dia)

João recebe alerta às 08:23:
  "Circuit Breaker ativado: 5 perdas consecutivas"

João verifica notícias:
  "SEC anuncia investigação sobre exchanges"

João decide:
  - Mantém bot pausado até clareza regulatória
  - Aguarda 3 horas

11:30 - Notícia desmentida, mercado se recupera
  - BTC volta para R$ 196k

João retoma bot:
  - Próxima compra: 11:35
  - Compra a R$ 196k, vende (take profit) a R$ 198k: +1% ✅
  - consecutive_failures reseta para 0
  - Bot opera normalmente
```

**Resultado**:
- Circuit breaker interrompeu sequência de perdas no pior momento
- Sem circuit breaker: João teria mais 5-10 trades ruins (perda adicional de ~R$ 1.000+)
- Com circuit breaker: Perda limitada a -5.5% do dia (R$ 550)

---

### Caso 3: Limite Diário Atingido Antes de Consecutivas

**Persona**: Pedro - Avançado, R$ 20.000 em SmartBot MAX

**Cenário**:
```
Pedro configura SmartBot com limites balanceados:
  - Threshold: 8 perdas consecutivas (tolerante)
  - Limite diário: 12%

Dia de alta volatilidade:
  10:00 - Trade #1: -3% ❌ (1/8)
  10:15 - Trade #2: +2% ✅ → contador reseta para 0
  10:30 - Trade #3: -4% ❌ (1/8)
  10:45 - Trade #4: -2.5% ❌ (2/8)
  11:00 - Trade #5: +1% ✅ → contador reseta para 0
  11:15 - Trade #6: -5% ❌ (1/8)
  11:30 - Trade #7: -3.5% ❌ (2/8)

  Perda acumulada do dia:
  -3% + 2% - 4% - 2.5% + 1% - 5% - 3.5% = -15%

  ⛔ Circuit Breaker acionado (limite diário de 12% excedido)

  Motivo: DAILY_LOSS_EXCEEDED (não foi por consecutivas, foi por total diário)

Pedro recebe alerta:
  "Circuit breaker ativado: perda diária de 15% excedeu limite de 12%"

Pedro analisa:
  - Trades intercalados (lucro/prejuízo)
  - Não há problema de estratégia (apenas volatilidade alta)
  - Decide aguardar próximo dia (reset diário)

Dia seguinte:
  - Perda diária reseta para 0%
  - Pedro retoma bot
  - Bot opera normalmente
```

**Observação**: Circuit breaker pode acionar por **qualquer** das 2 condições (diário ou consecutivas). Neste caso, perda diária veio primeiro.

---

## Regras de Negócio Principais

### 1. Ativação é Opcional
```
Circuit breaker DEVE ser explicitamente habilitado:
  enable_circuit_breaker = true

Se false → Bot opera SEM proteção automática
```

### 2. Duplo Gatilho (Condições Independentes)
```
Condição A: |perda_diária_%| >= max_daily_loss_pct
Condição B: consecutive_losses >= circuit_breaker_threshold

Se A OU B → Circuit breaker aciona (lógica OR, não AND)
```

### 3. Reset de Contador
```
Contador de falhas consecutivas reseta quando:
  - Trade bem-sucedido executado (purchase ou sale)
  - Chamada a record_purchase_success() ou record_sale_success()

Contador NÃO reseta quando:
  - Usuário clica "Retomar" (manual resume)
  - Passa da meia-noite (perda diária reseta, mas consecutivas não)
  - Bot é pausado/retomado manualmente
```

### 4. Status do Bot Após Acionamento
```
Antes: status = ACTIVE
Depois: status = PAUSED

PAUSED → Todos os workflows PULAM este bot:
  - Purchase workflow: if not bot.status.is_active → SKIP
  - Sales workflow: if not bot.status.is_active → SKIP
  - Check situation: Detecta paused, não tenta executar trades
```

### 5. Timestamp de Acionamento
```
Campo: circuit_breaker_triggered_at
Valor: datetime UTC do momento exato de acionamento
Uso: Dashboard mostra "Circuit breaker ativo desde [timestamp]"
```

### 6. Workflows Responsáveis
```
SmartBotCheckSituationWorkflow (a cada 30s):
  - Verifica se circuit breaker deve acionar
  - Pausa bot automaticamente se threshold atingido
  - Loga evento de pausa

SmartBotPurchaseWorkflow (a cada 60s):
  - Chama bot.can_execute_purchase()
  - Se status != ACTIVE → PULA bot
  - Incrementa consecutive_failures em caso de falha

SmartBotSalesWorkflow (a cada 67s):
  - Chama bot.can_execute_sale()
  - Se status != ACTIVE → PULA bot
```

### 7. Prioridade de Mensagens
```
Se AMBAS condições atingidas simultaneamente:
  Mensagem mostra: "Daily loss exceeded" (prioridade sobre consecutivas)

Motivo: Perda diária é geralmente mais crítica que padrão de falhas
```

---

## Métricas de Sucesso

### Operacionais
- **Taxa de Acionamento**: < 5% dos bots por dia (indica mercado normal)
- **Tempo Médio até Recuperação**: < 2 horas (usuário responde rápido)
- **Taxa de Re-acionamento**: < 10% (usuário corrigiu problema antes de retomar)

### Proteção de Capital
- **Capital Economizado**: R$ 200-500 por acionamento (vs sem circuit breaker)
- **Redução de Perdas Máximas**: -30% em dias de crash (circuit breaker vs sem)
- **Drawdown Máximo**: Limitado a threshold configurado

### Satisfação do Usuário
- **NPS de Segurança**: > 8/10 (usuários sentem-se protegidos)
- **Taxa de Desativação**: < 2% (poucos usuários desligam circuit breaker)
- **Tickets de "Perdi Tudo"**: 0 (nenhum caso de perda total com circuit breaker ativo)

---

## Problemas Comuns e Soluções

### ❌ "Bot pausou mas não deveria"
**Causa**: Threshold muito baixo para volatilidade do mercado
**Solução**: Aumentar `circuit_breaker_threshold` (ex: 5 → 7)

### ❌ "Circuit breaker não acionou e perdi muito"
**Causa**: Circuit breaker desabilitado (`enable_circuit_breaker=false`)
**Solução**: Habilitar e configurar thresholds apropriados

### ❌ "Retomei o bot mas pausou novamente imediatamente"
**Causa**: Problema de estratégia não foi corrigido (spread ainda alto, saldo insuficiente)
**Solução**: Investigar causa raiz antes de retomar (não apenas clicar "Retomar")

### ❌ "Contador de falhas não resetou ao retomar"
**Causa**: Comportamento esperado (contador só reseta em trade bem-sucedido)
**Solução**: Aguardar próximo trade executar com sucesso

### ❌ "Bot pausou com 4 falhas, mas threshold é 5"
**Causa**: Limite diário foi atingido (não foram as consecutivas)
**Solução**: Ver mensagem de alerta (indica qual condição foi atingida)

---

## Roadmap Futuro

### Melhorias Planejadas (Q1-Q2 2025)
- [ ] **Reset Automático após X horas**: Circuit breaker expira após 6h sem intervenção
- [ ] **Configuração Dinâmica**: Ajustar threshold baseado em volatilidade do mercado
- [ ] **Notificações Proativas**: Alerta quando bot está em 3/5 falhas (antes de pausar)
- [ ] **Análise Post-Mortem**: Dashboard mostra "por que circuit breaker acionou" (spread, slippage, volume)
- [ ] **Circuit Breaker para CandleBots e DCABots**: Expandir proteção para todos os tipos
- [ ] **Modos de Proteção**: "Conservador", "Moderado", "Agressivo" (pré-configurados)

### Visão de Longo Prazo
- [ ] **Machine Learning**: Prediz quando circuit breaker PODERIA acionar (alerta antecipado)
- [ ] **Sugestões Automáticas**: "Recomendamos aumentar threshold para 7 baseado em seu histórico"
- [ ] **Circuit Breaker de Portfolio**: Pausa TODOS os bots se perda total > X%
- [ ] **Integração com Volatility Index**: Ajusta threshold dinamicamente conforme VIX cripto

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
