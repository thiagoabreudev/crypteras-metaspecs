---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'trading', 'contexts']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Sistema de Contextos de Trading

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Organizar bots por contextos isolados (moeda-fiat-exchange) para clareza e controle de limites de subscription.

**Constraints** (limites obrigatórios):
- Contexto = combinação única de `coin-fiat-exchange-user_id`
- FREE: max 1 contexto, PRO: max 3 contextos, MAX: ilimitado
- Validar limites no backend antes de criar bot/contexto
- Cada contexto deve ter stats independentes (invested, profit, blocked)
- Nome do contexto deve ser gerado automaticamente (evitar duplicatas)
- Migração de bots legados para modelo de contextos deve ser suportada

**Non-Goals** (o que NÃO fazer):
- Permitir contextos compartilhados entre usuários
- Suportar múltiplas exchanges no mesmo contexto
- Criar hierarquia complexa de contextos (folders, tags)
- Permitir usuário renomear contextos livremente (pode causar confusão)
- Implementar contextos "virtuais" ou "templates"
- Adicionar permissões granulares por contexto
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Contextos de trading e isolamento de estratégias
:::

## Visão Geral

Sistema que organiza e limita a capacidade de trading dos usuários através de **contextos isolados** - cada contexto representa uma combinação única de moeda, fiat e exchange. É a **base do modelo de negócio freemium** do Crypteras.

**Status**: ✅ Em Produção
**Tickets**: CRY-40
**Diferencial Competitivo**: Monetização através de limites de contextos

---

## Propósito e Valor

### Para o Usuário
- **Organização**: Separa estratégias por mercado (BTC-BRL vs ETH-BRL)
- **Isolamento**: Cada contexto opera independentemente (sem interferência)
- **Flexibilidade**: Pode ter diferentes estratégias em cada contexto
- **Escalabilidade**: Upgrade para PRO/MAX libera mais contextos

### Para o Negócio
- **Monetização Clara**: Limite de contextos diferencia planos (1 vs 3 vs ilimitado)
- **Lock-in**: Histórico e configurações acumuladas em cada contexto (retenção)
- **Upsell Natural**: Usuário precisa de mais contextos → upgrade inevitável
- **Diferenciação**: Competitors cobram por número de bots; Crypteras cobra por contextos

### Problema que Resolve

**Sem Contextos**:
```
Usuário tem 10 bots, todos operando no mesmo par BTC-BRL
  → Difícil organizar
  → Difícil distinguir estratégias
  → Impossível monetizar (todos bots no mesmo mercado)
```

**Com Contextos**:
```
FREE: 1 contexto (BTC-BRL-MercadoBitcoin)
  → 1-3 bots neste contexto
  → Usuário quer operar ETH também? → Precisa fazer upgrade

PRO: 3 contextos (BTC-BRL, ETH-BRL, USDT-BRL)
  → Pode diversificar estratégias
  → 3-5 bots distribuídos entre contextos

MAX: Ilimitado (10+ contextos diferentes)
  → Scalping profissional em múltiplos pares
  → 30+ bots ativos
```

---

## Funcionalidades Principais

### 1. Definição de Contexto de Trading

**O que é um Contexto?**

Uma **combinação imutável** de 3 parâmetros que define um mercado isolado:

```
TradingContext = {Moeda Cripto} + {Moeda Fiat} + {Exchange}

Exemplos:
✅ BTC-BRL-MercadoBitcoin   → Bitcoin em Real no Mercado Bitcoin
✅ ETH-BRL-MercadoBitcoin   → Ethereum em Real no Mercado Bitcoin
✅ BTC-USDT-Binance         → Bitcoin em USDT na Binance
✅ ETH-USDT-Binance         → Ethereum em USDT na Binance
✅ LTC-BRL-MercadoBitcoin   → Litecoin em Real no Mercado Bitcoin
```

**Identificador Único (ID)**:
```
Formato: {CURRENCY}-{FIAT}-{EXCHANGE}

Exemplos:
- "BTC-BRL-MercadoBitcoin"
- "ETH-USDT-Binance"
```

**Symbol de Trading** (varia por exchange):
```
Mercado Bitcoin: BTC-BRL (com hífen)
Binance: BTCBRL (sem hífen)
```

---

### 2. Limites por Plano (Modelo de Monetização)

| Plano | Limite de Contextos | Preço | Churn Típico | Valor Vitalício |
|-------|-------------------|-------|--------------|-----------------|
| **FREE** | 1 | R$ 0/mês | ~40% | R$ 0 |
| **PRO** | 3 | R$ 19,90/mês | ~5% | R$ 478 (2 anos) |
| **MAX** | Ilimitado | R$ 97/mês | ~1% | R$ 3.492 (3 anos) |

**Validação de Limite**:
```
FREE User tenta criar 2º contexto:
  ❌ "Limite de contextos atingido: 1/1 para plano FREE.
      Faça upgrade para PRO: https://crypteras.tech/upgrade"

PRO User tenta criar 4º contexto:
  ❌ "Limite de contextos atingido: 3/3 para plano PRO.
      Faça upgrade para MAX: https://crypteras.tech/upgrade"

MAX User tenta criar 100º contexto:
  ✅ Permitido (sem limite)
```

---

### 3. Criação de Contexto (3 Validações Obrigatórias)

#### Validação 1: Usuário Deve Ter Exchange Configurada

**Regra**: Não pode criar contexto sem credenciais de exchange

```
Fluxo:
1. Usuário clica "Criar Contexto"
2. Sistema verifica: user.has_credentials() == True?
3. Se FALSE → Bloqueia com mensagem

Mensagem de Erro:
"Configure pelo menos 1 exchange antes de criar contextos.
 Acesse 'Credenciais' para adicionar chaves API do Mercado Bitcoin ou Binance."

Por quê?
- Contexto sem credenciais não serve para nada
- Usuário não consegue executar trades
- Previne contextos "mortos" no sistema
```

**Exemplo Real**:
```
João (novo usuário FREE):
  1. Cadastra-se na plataforma
  2. Tenta criar contexto BTC-BRL
  3. ❌ Bloqueado: "Configure exchange primeiro"
  4. Vai em "Credenciais" → Adiciona API do Mercado Bitcoin
  5. Volta e tenta criar contexto novamente
  6. ✅ Sucesso: Contexto criado
```

---

#### Validação 2: Assinatura Deve Estar Ativa

**Regra**: Apenas assinaturas com status ACTIVE podem criar contextos

```
Status Válidos:
✅ ACTIVE → Permite criar contextos

Status Inválidos:
❌ CANCELED → Assinatura cancelada
❌ UNPAID → Pagamento não realizado
❌ PAST_DUE → Pagamento atrasado (retry em andamento)
❌ INCOMPLETE → Checkout não finalizado

Mensagem de Erro:
"Assinatura não está ativa. Status atual: CANCELED.
 Renove sua assinatura para continuar criando contextos."
```

**Comportamento por Plano**:
```
FREE:
  - Sempre ACTIVE (não tem período de expiração)
  - Nunca expira ou cancela automaticamente

PRO/MAX:
  - Tem período (current_period_start → current_period_end)
  - Pode expirar se pagamento falhar
  - Pode ser cancelada pelo usuário
  - Webhook Stripe/MP mantém status atualizado
```

---

#### Validação 3: Respeitar Limite por Plano

**Regra**: Quantidade de contextos atuais < limite do plano

```python
# Lógica de validação
def can_create_context(subscription, current_count):
    if subscription.plan == FREE:
        return current_count < 1  # Máximo 1

    elif subscription.plan == PRO:
        return current_count < 3  # Máximo 3

    elif subscription.plan == MAX:
        return True  # Sempre permite (ilimitado)
```

**Cenários**:
```
Cenário A: FREE com 0 contextos
  → can_create_context(0) = True ✅
  → Permite criar 1º contexto

Cenário B: FREE com 1 contexto
  → can_create_context(1) = False ❌
  → Bloqueia criação do 2º contexto

Cenário C: PRO com 2 contextos
  → can_create_context(2) = True ✅
  → Permite criar 3º contexto

Cenário D: PRO com 3 contextos
  → can_create_context(3) = False ❌
  → Bloqueia criação do 4º contexto

Cenário E: MAX com 10 contextos
  → can_create_context(10) = True ✅
  → Permite criar 11º contexto (ilimitado)
```

---

### 4. Relação com Bots

**Arquitetura**: 1 Contexto → Múltiplos Bots

```
Contexto: BTC-BRL-MercadoBitcoin
├── SmartBot #1 (Compra contínua a cada 2min)
│   ├── trading_context_id: "BTC-BRL-MercadoBitcoin"
│   ├── operation_mode: CONTINUOUS
│   ├── purchase_interval: 120s
│   └── max_position: R$ 500
│
├── SmartBot #2 (Compra agendada 9h da manhã)
│   ├── trading_context_id: "BTC-BRL-MercadoBitcoin"
│   ├── operation_mode: SCHEDULED
│   ├── purchase_schedule: "0 9 * * *"
│   └── max_position: R$ 1.000
│
├── CandleBot #1 (Análise técnica 1h)
│   ├── trading_context_id: "BTC-BRL-MercadoBitcoin"
│   ├── timeframe: "1h"
│   ├── indicators: [RSI, MA, BB]
│   └── auto_execute: True
│
└── DCABot #1 (Acumulação diária)
    ├── trading_context_id: "BTC-BRL-MercadoBitcoin"
    ├── frequency: "24h"
    ├── purchase_amount: R$ 100
    └── cycles: 30
```

**IMPORTANTE**: Limite de bots é **POR USUÁRIO**, não por contexto

```
FREE User:
  - 1 contexto + 1 bot TOTAL
  - Exemplo: 1 SmartBot em BTC-BRL

PRO User:
  - 3 contextos + 3-5 bots TOTAL (distribuídos entre contextos)
  - Exemplo:
    * Contexto 1 (BTC-BRL): 2 SmartBots
    * Contexto 2 (ETH-BRL): 1 CandleBot
    * Contexto 3 (USDT-BRL): 1 DCABot

MAX User:
  - Ilimitado contextos + ilimitado bots
  - Exemplo: 10 contextos com 30 bots distribuídos
```

---

### 5. Isolamento e Independência

**Cada contexto opera de forma completamente isolada**:

```
Contexto 1: BTC-BRL-MercadoBitcoin
├── Posição aberta: 0.005 BTC (R$ 1.000 investido)
├── P&L: +R$ 50 (+5%)
└── Bots ativos: 2

Contexto 2: ETH-BRL-Binance
├── Posição aberta: 0.15 ETH (R$ 800 investido)
├── P&L: -R$ 40 (-5%)
└── Bots ativos: 1

→ Posições NÃO interferem uma com a outra
→ P&L calculado separadamente
→ Estratégias diferentes podem coexistir
```

**Benefício**: Usuário pode ter estratégia conservadora em BTC e agressiva em ETH simultaneamente

---

### 6. Ciclo de Vida de um Contexto

**Estados Possíveis**:

```
┌─────────────────────────────────────────────────┐
│ ACTIVE (Padrão)                                 │
│ - Contexto funcional                            │
│ - Bots podem operar                             │
│ - Ordens são executadas                         │
└──────────────┬──────────────────────────────────┘
               │
               ↓ (Usuário pausa)
┌──────────────▼──────────────────────────────────┐
│ PAUSED                                          │
│ - Contexto pausado manualmente                  │
│ - Bots NÃO operam                               │
│ - Posição congelada (sem novas ordens)          │
│ - Pode retomar para ACTIVE                      │
└──────────────┬──────────────────────────────────┘
               │
               ↓ (Usuário arquiva)
┌──────────────▼──────────────────────────────────┐
│ ARCHIVED                                        │
│ - Contexto arquivado (deletado logicamente)     │
│ - NÃO aparece em listagens                      │
│ - Bots automaticamente arquivados               │
│ - IRREVERSÍVEL (não pode restaurar)             │
└─────────────────────────────────────────────────┘
```

**Transições**:
```
Criar → ACTIVE
ACTIVE → PAUSED (pausar)
PAUSED → ACTIVE (retomar)
ACTIVE → ARCHIVED (arquivar)
PAUSED → ARCHIVED (arquivar)
```

---

### 7. Configurações de Contexto

Cada contexto armazena **centenas de parâmetros** de trading:

**Estratégia de Lucro/Perda**:
- `bot_percentage_profit`: Lucro mínimo para vender (ex: 2%)
- `bot_minimum_loss_percentage`: Perda mínima aceitável (ex: -1%)
- `bot_maximum_loss_percentage`: Perda máxima antes de stop loss (ex: -5%)

**Intervalos de Workflow**:
- `workflow_purchase_interval`: Intervalo entre compras (ex: 60s)
- `workflow_sale_interval`: Intervalo entre vendas (ex: 30s)
- `workflow_check_situation_interval`: Verificação de posição (ex: 15s)

**Circuit Breaker**:
- `circuit_breaker_max_daily_loss_pct`: Perda diária máxima (ex: 10%)
- `circuit_breaker_max_consecutive_losses`: Perdas consecutivas (ex: 5)

**Proteções**:
- `sanity_check_max_price_deviation_pct`: Desvio máximo de preço (ex: 10%)
- `max_order_lifetime_seconds`: Tempo máximo de ordem aberta (ex: 3600s)

**Estratégias Avançadas**:
- `trailing_stop_percentage`: Distância do trailing stop (ex: 2%)
- `adaptive_sell_enabled`: Venda adaptativa por volatilidade (true/false)
- `candle_strategy_enabled`: Estratégia de candles (true/false)
- `enable_native_trailing_stop`: Usar trailing stop nativo da exchange

---

## Casos de Uso Reais

### Caso 1: João - Usuário FREE (1 Contexto)

**Perfil**: Iniciante, quer testar plataforma

**Jornada**:
```
Dia 1: Cadastro e Configuração
  1. João cria conta FREE
  2. Adiciona credenciais do Mercado Bitcoin
  3. Cria contexto: BTC-BRL-MercadoBitcoin
  4. Configura 1 SmartBot em modo CONTINUOUS
  5. Bot começa a operar com R$ 500

Dia 7: Tentativa de Expandir
  1. João quer operar também em ETH-BRL
  2. Tenta criar 2º contexto
  3. ❌ Bloqueado: "Limite de 1/1 contextos (plano FREE)"
  4. Sistema mostra comparação FREE vs PRO
  5. João vê: "PRO = 3 contextos por R$ 19,90/mês"

Dia 15: Decisão de Upgrade
  1. João está satisfeito com resultados (+3% em BTC)
  2. Quer diversificar para ETH e USDT
  3. Faz upgrade para PRO via Stripe
  4. Imediatamente pode criar 2 contextos adicionais
  5. Cria: ETH-BRL-MercadoBitcoin e USDT-BRL-Binance
```

**Resultado**: Conversão FREE → PRO através de limite de contextos

---

### Caso 2: Maria - Usuária PRO (3 Contextos)

**Perfil**: Intermediária, diversifica portfólio

**Setup**:
```
Contexto 1: BTC-BRL-MercadoBitcoin
├── SmartBot #1 (Conservador): Compra R$ 100 a cada 6h
└── Stats: +R$ 150 lucro acumulado (15 dias)

Contexto 2: ETH-BRL-MercadoBitcoin
├── CandleBot #1 (TA com AI): Timeframe 1h, RSI+MA+BB
└── Stats: +R$ 80 lucro acumulado (7 dias)

Contexto 3: USDT-BRL-Binance
├── DCABot #1 (Acumulação): R$ 50/dia, 30 ciclos
└── Stats: -R$ 10 (2 dias, ainda acumulando)
```

**Dia 20: Tentativa de 4º Contexto**:
```
Maria quer operar LTC-BRL:
  1. Clica "Criar Contexto"
  2. Preenche: LTC + BRL + MercadoBitcoin
  3. ❌ Bloqueado: "Limite de 3/3 contextos (plano PRO)"
  4. Sistema sugere: "Upgrade para MAX (R$ 97/mês) = contextos ilimitados"

Maria decide:
  - Opção A: Arquivar contexto USDT (menos lucrativo) e criar LTC
  - Opção B: Manter 3 contextos e não operar LTC
  - Opção C: Upgrade para MAX (se começar a operar mais pares)
```

---

### Caso 3: Crypto Fund - Usuário MAX (10+ Contextos)

**Perfil**: Profissional, fundo de investimento, opera em escala

**Setup** (exemplo simplificado):
```
Contexto 1: BTC-BRL-MercadoBitcoin (6 bots, R$ 50k posição)
Contexto 2: BTC-USDT-Binance (4 bots, R$ 30k posição)
Contexto 3: ETH-BRL-MercadoBitcoin (3 bots, R$ 20k posição)
Contexto 4: ETH-USDT-Binance (2 bots, R$ 15k posição)
Contexto 5: LTC-BRL-MercadoBitcoin (2 bots, R$ 10k posição)
Contexto 6: SOL-USDT-Binance (3 bots, R$ 25k posição)
Contexto 7: ADA-USDT-Binance (2 bots, R$ 12k posição)
Contexto 8: DOT-USDT-Binance (1 bot, R$ 8k posição)
Contexto 9: MATIC-USDT-Binance (1 bot, R$ 5k posição)
Contexto 10: LINK-USDT-Binance (1 bot, R$ 7k posição)

Total: 10 contextos, 25 bots, R$ 182k investido
```

**Valor do Plano MAX**:
- Permite gestão profissional multi-mercado
- Diversificação máxima de risco
- Escalabilidade sem limites artificiais
- R$ 97/mês é despesa mínima comparada ao capital gerenciado

---

### Caso 4: Upgrade FREE → PRO (Impacto Imediato)

**Cenário**: João tem 1 contexto (limite FREE) e faz upgrade

```
ANTES do Upgrade:
  Plan: FREE
  Contextos: 1/1 (BTC-BRL-MercadoBitcoin)
  Bots: 1 SmartBot
  Bloqueio: Não pode criar 2º contexto

João clica "Upgrade para PRO":
  1. Redirecionado para checkout Stripe
  2. Paga R$ 19,90 no cartão
  3. Stripe envia webhook: checkout.session.completed
  4. Sistema atualiza subscription:
     - plan: FREE → PRO
     - status: ACTIVE
     - current_period_end: +30 dias

DEPOIS do Upgrade (imediato):
  Plan: PRO
  Contextos: 1/3 (pode criar mais 2)
  Bots: 1 (pode criar mais 2-4 dependendo do limite)
  Desbloqueio: ✅ Pode criar 2º e 3º contextos

João imediatamente:
  1. Cria contexto 2: ETH-BRL-MercadoBitcoin
  2. Cria contexto 3: USDT-BRL-Binance
  3. Adiciona 2 bots (1 em cada contexto novo)
  4. Diversifica portfólio com 3 pares diferentes
```

**Tempo de Ativação**: < 5 segundos após pagamento aprovado

---

## Regras de Negócio Principais

### 1. Imutabilidade de Contexto
```
Regra: Contexto NÃO pode ser modificado após criação

O que NÃO pode mudar:
❌ Currency (BTC → ETH) - precisa criar novo contexto
❌ Fiat (BRL → USD) - precisa criar novo contexto
❌ Exchange (MercadoBitcoin → Binance) - precisa criar novo contexto

O que PODE mudar:
✅ Configurações (intervals, percentages, flags)
✅ Status (ACTIVE ↔ PAUSED, ARCHIVED)
✅ Paper mode (true ↔ false)
```

### 2. Validação de Criação (3 Checks Obrigatórios)
```
1. user.has_credentials() == True
2. subscription.is_active() == True
3. subscription.can_create_context(current_count) == True

Falha em qualquer um → Bloqueio com erro descritivo
```

### 3. Limite é por Plano, Não por Período
```
Upgrade FREE → PRO:
  - Limite muda de 1 → 3 IMEDIATAMENTE
  - Não precisa esperar próximo billing cycle
  - Webhook atualiza subscription na hora

Downgrade (Cancelamento) PRO → FREE:
  - Limite muda de 3 → 1 no fim do período
  - Usuário mantém 3 contextos até current_period_end
  - Após expirar: precisa arquivar 2 contextos para criar novos
```

### 4. Contextos e Bots São Separados
```
Limite de Contextos: por plano (FREE=1, PRO=3, MAX=∞)
Limite de Bots: por plano (FREE=1, PRO=3-5, MAX=∞)

Validações são independentes:
  - Pode ter 1 contexto com 3 bots (PRO)
  - Pode ter 3 contextos com 1 bot cada (PRO)
  - Distribuição depende da estratégia do usuário
```

### 5. Arquivamento É Irreversível
```
Regra: Contexto arquivado NÃO pode ser restaurado

Fluxo:
1. Usuário clica "Arquivar Contexto"
2. Sistema confirma: "Tem certeza? Esta ação é irreversível"
3. Se confirmado:
   - status → ARCHIVED
   - Todos os bots no contexto → arquivados
   - Não aparece mais em listagens
4. Para operar novamente: precisa criar novo contexto

Por quê?
- Simplifica lógica de contagem (contextos ativos)
- Previne "lixo" de contextos inativos
- Força usuário a manter apenas contextos úteis
```

### 6. Upgrade Imediato, Downgrade ao Fim do Período
```
Upgrade (FREE → PRO):
  - Webhook processa
  - Limite atualiza na hora
  - Usuário pode criar contextos imediatamente

Cancelamento (PRO → FREE):
  - Stripe marca: cancel_at_period_end=True
  - Usuário mantém limite PRO (3 contextos) até fim
  - Após current_period_end: limite reduz para FREE (1 contexto)
  - Se tiver 3 contextos ativos: não pode criar novos até arquivar 2
```

---

## Métricas de Sucesso

### Produto
- **Contextos por Usuário Médio**: 1.2 (FREE), 2.5 (PRO), 5.8 (MAX)
- **Taxa de Conversão FREE → PRO**: > 12% (motivado por limite de contextos)
- **Taxa de Uso de Contextos**: % usuários com > 1 contexto (indicador de engajamento)

### Negócio
- **MRR por Contexto**: R$ 19,90/3 = R$ 6,63 (PRO), R$ 97/6 = R$ 16,17 (MAX médio)
- **Churn por Limite**: % cancelamentos mencionando "limite de contextos" (< 3%)
- **Upgrade Trigger**: % upgrades motivados por "atingir limite" (> 60%)

### Técnicas
- **Performance de Query**: Listagem de contextos < 100ms (p95)
- **Taxa de Erro na Criação**: < 0.5%
- **Uptime de Validação**: > 99.9%

---

## Problemas Comuns e Soluções

### ❌ "Não consigo criar 2º contexto (FREE)"
**Causa**: Limite do plano FREE é 1 contexto
**Solução**: Fazer upgrade para PRO (3 contextos) ou MAX (ilimitado)

### ❌ "Criei contexto mas bots não operam"
**Causa**: Credenciais da exchange não configuradas
**Solução**: Ir em "Credenciais" e adicionar API keys da exchange do contexto

### ❌ "Quero mudar BTC para ETH no meu contexto"
**Causa**: Contextos são imutáveis (não podem mudar currency/fiat/exchange)
**Solução**: Criar novo contexto ETH-BRL e arquivar o antigo BTC-BRL

### ❌ "Arquivei contexto por engano, posso recuperar?"
**Causa**: Arquivamento é irreversível (design proposital)
**Solução**: Criar novo contexto com mesmos parâmetros (histórico perdido)

### ❌ "Cancelei PRO mas ainda tenho 3 contextos ativos"
**Causa**: Cancelamento com grace period (mantém acesso até fim do período)
**Solução**: Após current_period_end, precisará arquivar 2 contextos para voltar a criar novos

---

## Roadmap Futuro

### Melhorias Planejadas (Q2-Q3 2025)
- [ ] **Validação de Unicidade**: Bloquear duplicatas (BTC-BRL-MB já existe)
- [ ] **Templates de Contexto**: Copiar configurações de um contexto para outro
- [ ] **Compartilhamento**: Compartilhar contexto com outros usuários (read-only)
- [ ] **Importação de Histórico**: Importar trades antigos para contexto novo
- [ ] **Sincronização Multi-Device**: Contextos sincronizados em tempo real

### Visão de Longo Prazo
- [ ] **Premium Contexts**: Contextos "premium" com features exclusivas (backtesting, AI avançado)
- [ ] **Context Marketplace**: Marketplace de configurações de contexto (comprar/vender setups)
- [ ] **Multi-User Contexts**: Contextos compartilhados para fundos/equipes (plano Enterprise)
- [ ] **Context Analytics**: Dashboard de performance comparativa entre contextos
- [ ] **Auto-Rebalancing**: Rebalanceamento automático de capital entre contextos

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
