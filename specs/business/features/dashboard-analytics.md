---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'dashboard', 'analytics']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Dashboard e Analytics

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Exibir métricas de performance de bots e portfolio do usuário de forma clara e acionável.

**Constraints** (limites obrigatórios):
- Atualização em tempo real (polling a cada 30s ou WebSocket)
- Exibir lucro/prejuízo em BRL e percentual
- Gráficos de série temporal para profit over time
- Breakdown por bot (qual bot gerou mais lucro)
- Comparar performance com HOLD strategy (buy and hold)
- Exportar dados para CSV/JSON

**Non-Goals** (o que NÃO fazer):
- Implementar backtesting visual de estratégias
- Adicionar comparação com outros usuários (leaderboards)
- Criar reports automáticos por email (apenas on-demand)
- Implementar alertas customizáveis por Slack/Telegram
- Adicionar forecasting ou predições de IA
- Criar dashboards compartilháveis (social trading)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Dashboard e analytics de trading
:::

## Visão Geral

Dashboard centralizado que consolida toda a atividade de trading do usuário em uma única tela: saldos, P&L (lucro/prejuízo), performance de bots, ordens ativas e métricas de sucesso. Oferece visão holística do portfólio sem necessidade de acessar múltiplas exchanges.

**Status**: ✅ Em Produção
**Planos**: Todos (FREE com limitações, PRO/MAX completo)
**Prioridade**: ALTA - Interface principal do produto

---

## Propósito e Valor

### Para o Usuário
- **Visão Unificada**: Vê tudo em um só lugar (Mercado Bitcoin + Binance + Crypteras)
- **Tomada de Decisão Rápida**: Métricas visuais facilitam identificar oportunidades e problemas
- **Acompanhamento de Performance**: Sabe se está lucrando ou perdendo em tempo real
- **Comparação com Mercado**: Entende se está performando acima/abaixo do Bitcoin
- **Transparência Total**: Nenhum dado escondido - tudo visível e rastreável

### Para o Negócio
- **Engajamento**: Usuários que acessam dashboard diariamente têm 3x menos churn
- **Upsell**: Dashboard FREE limitado incentiva upgrade para PRO
- **Educação**: Métricas visuais ensinam conceitos de trading (win rate, drawdown)
- **Retention**: Dashboard bem desenhado cria "sticky product" (difícil de abandonar)
- **Dados de Produto**: Métricas mais vistas revelam o que usuários valorizam

---

## Componentes do Dashboard

### 1. Resumo Executivo (Hero Section)

**O que mostra** (topo da página, sempre visível):

```
╔═══════════════════════════════════════════════════════════╗
║  💰 SALDO TOTAL                     📊 P&L (30 DIAS)      ║
║  R$ 5.420,00                        +R$ 420,00 (+8,4%)   ║
║  ↑ +2,3% (24h)                      🟢 Lucrando          ║
╠═══════════════════════════════════════════════════════════╣
║  🤖 BOTS ATIVOS                     📈 WIN RATE          ║
║  5 de 7                             68,5%                ║
╚═══════════════════════════════════════════════════════════╝
```

**Informações**:
- **Saldo Total**: Soma de saldos em todas as exchanges (BRL + cripto convertida para BRL)
- **Variação 24h**: Quanto saldo mudou nas últimas 24h (% e R$)
- **P&L 30 dias**: Lucro/prejuízo realizado + não realizado dos últimos 30 dias
- **Status Visual**: 🟢 Verde (lucrando), 🔴 Vermelho (prejuízo), ⚪ Cinza (break-even)
- **Bots Ativos**: Quantidade de bots rodando vs total criados
- **Win Rate**: % de trades que resultaram em lucro

**Regra de Negócio**:
```
Saldo Total = Σ(saldo_fiat) + Σ(saldo_crypto × preço_atual)

Exemplo:
  Mercado Bitcoin:
    - R$ 1.000 (BRL)
    - 0,01 BTC × R$ 200.000 = R$ 2.000
  Binance:
    - R$ 500 (BRL)
    - 0,01 ETH × R$ 12.000 = R$ 120

  Total = 1.000 + 2.000 + 500 + 120 = R$ 3.620
```

---

### 2. Gráfico de Performance (Principal)

**O que mostra**:
- Linha temporal do saldo total (últimos 7/30/90 dias)
- Linha de referência: "Se tivesse apenas holding Bitcoin"
- Áreas: Lucro (verde), Prejuízo (vermelho)

**Exemplo visual**:
```
R$
6k ┤                                        ╭─╮
5k ┤                          ╭────╮       │ │
4k ┤              ╭────╮     │    ╰───────╯ │
3k ┤     ╭────────╯    ╰─────╯              │
2k ┤─────╯                                  │
   └────────────────────────────────────────┘
   01/01  05/01  10/01  15/01  20/01  25/01

   Legenda:
   ━━━ Seu portfólio
   ┄┄┄ Holding Bitcoin
   🟢 Áreas de lucro vs holding
```

**Insights visuais**:
```
Seu portfólio: +8,4% (R$ 3.000 → R$ 3.252)
Holding BTC:   +12,0% (R$ 3.000 → R$ 3.360)

📊 Performance: 70% do Bitcoin
💡 Interpretação: Você lucrou, mas Bitcoin sozinho teria sido melhor
                   → Considere aumentar exposição a BTC
```

**Controles interativos**:
- Período: [7 dias] [30 dias] [90 dias] [1 ano] [Tudo]
- Comparar com: [Bitcoin] [Ethereum] [Mercado] [Nenhum]

---

### 3. Saldos por Exchange

**O que mostra**: Distribuição de saldos entre exchanges e moedas

```
╔═══════════════════════════════════════════╗
║  MERCADO BITCOIN                 R$ 3.000 ║
║  ├─ BRL           R$ 1.000  (33%)         ║
║  └─ BTC (0,01)    R$ 2.000  (67%)         ║
╠═══════════════════════════════════════════╣
║  BINANCE                        R$ 2.420  ║
║  ├─ USDT          R$ 1.200  (50%)         ║
║  ├─ BTC (0,005)   R$ 1.000  (41%)         ║
║  └─ ETH (0,01)    R$ 220    (9%)          ║
╠═══════════════════════════════════════════╣
║  💵 TOTAL                       R$ 5.420  ║
╚═══════════════════════════════════════════╝
```

**Ações disponíveis**:
- **Sincronizar** (botão): Busca saldos atualizados da exchange
- **Ver detalhes**: Mostra histórico de depósitos/saques (se disponível)
- **Alerta de saldo baixo**: Notifica se saldo < R$ 100 (configurable)

**Regra de Negócio (Sincronização)**:
```
Ao clicar "Sincronizar":
1. Para cada exchange configurada:
   - Conecta via API
   - Busca saldos de TODAS as moedas
   - Converte crypto para BRL (preço atual)
   - Salva em cache (TTL 5 minutos)
2. Atualiza UI

Sincronização automática:
- A cada 5 minutos (background)
- Após cada ordem executada
- Ao abrir dashboard
```

---

### 4. Performance por Estratégia

**O que mostra**: Comparação de lucro/prejuízo entre diferentes tipos de bots

```
╔═════════════════════════════════════════════════════════╗
║  ESTRATÉGIA      │ INVESTIDO │ LUCRO   │ WIN RATE │ # ║
╠═════════════════════════════════════════════════════════╣
║  🎯 Candle Bots  │ R$ 2.000  │ +R$ 340 │   72%    │ 3 ║
║  💰 DCA Bots     │ R$ 1.500  │ +R$ 120 │   100%   │ 2 ║
║  🤖 Smart Bots   │ R$ 1.500  │ -R$ 40  │   55%    │ 2 ║
╠═════════════════════════════════════════════════════════╣
║  📊 TOTAL        │ R$ 5.000  │ +R$ 420 │   68,5%  │ 7 ║
╚═════════════════════════════════════════════════════════╝

📌 Melhor performer: Candle Bots (+17%)
⚠️ Atenção: Smart Bots em prejuízo (-2,7%)
```

**Insights automáticos**:
```
💡 "Seus Candle Bots estão performando 2× melhor que Smart Bots.
   Considere realocar capital de Smart → Candle."

💡 "DCA Bots têm 100% win rate (nenhum em prejuízo).
   Estratégia conservadora está funcionando."
```

---

### 5. Bots Ativos (Lista Resumida)

**O que mostra**: Status de cada bot com métricas principais

```
╔═══════════════════════════════════════════════════════════════╗
║  BOT                    │ STATUS  │ LUCRO    │ ÚLTIMA AÇÃO    ║
╠═══════════════════════════════════════════════════════════════╣
║  🎯 BTC Candle 1h       │ 🟢 Ativo│ +R$ 210  │ Compra (2h)    ║
║  💰 ETH DCA Diário      │ 🟢 Ativo│ +R$ 80   │ Compra (10h)   ║
║  🤖 BTC Smart 24h       │ ⏸️ Pausa│ -R$ 20   │ Venda (3d)     ║
║  🎯 SOL Candle 4h       │ 🔴 Erro │ +R$ 50   │ Erro API (1d)  ║
╚═══════════════════════════════════════════════════════════════╝

[Ver Todos os Bots →]
```

**Ações rápidas** (hover no bot):
- ▶️ **Retomar** (se pausado)
- ⏸️ **Pausar** (se ativo)
- 🔧 **Editar**
- 📊 **Ver Detalhes**
- 🗑️ **Arquivar**

**Cores de status**:
- 🟢 Verde: Ativo e funcionando
- ⏸️ Amarelo: Pausado manualmente
- 🔴 Vermelho: Erro (precisa atenção)
- ⚪ Cinza: Arquivado (inativo permanentemente)

---

### 6. Ordens Recentes

**O que mostra**: Últimas 10 ordens executadas (todas as exchanges)

```
╔══════════════════════════════════════════════════════════════════╗
║  DATA/HORA  │ BOT          │ AÇÃO   │ VALOR    │ PREÇO    │ STATUS║
╠══════════════════════════════════════════════════════════════════╣
║  Hoje 14:32 │ BTC Candle   │ COMPRA │ 0,0005   │ R$ 200k  │ ✅    ║
║  Hoje 10:15 │ ETH DCA      │ COMPRA │ 0,008    │ R$ 12,5k │ ✅    ║
║  Ontem 18:20│ BTC Smart    │ VENDA  │ 0,001    │ R$ 205k  │ ✅    ║
║  Ontem 09:00│ SOL Candle   │ COMPRA │ 50       │ R$ 3,20  │ ⏳    ║
╚══════════════════════════════════════════════════════════════════╝

[Ver Histórico Completo →]
```

**Status**:
- ✅ **Executada** (FILLED)
- ⏳ **Pendente** (PENDING)
- ❌ **Cancelada** (CANCELED)
- ⚠️ **Falhou** (FAILED)

**Filtros** (histórico completo):
- Por bot
- Por exchange
- Por tipo (compra/venda)
- Por status
- Por período (hoje, semana, mês, tudo)

---

### 7. Alertas e Notificações

**O que mostra**: Avisos importantes que precisam de atenção

```
╔═════════════════════════════════════════════════════════════╗
║  🔔 ALERTAS                                                 ║
╠═════════════════════════════════════════════════════════════╣
║  ⚠️ BOT "BTC Smart 24h" pausado (Circuit Breaker ativado)  ║
║     Ação: Resetar circuit breaker → [Ir para Bot]          ║
╠═════════════════════════════════════════════════════════════╣
║  💰 BOT "ETH DCA" próximo de take profit (faltam 2%)       ║
║     Lucro esperado: R$ 120 → [Ver Detalhes]                ║
╠═════════════════════════════════════════════════════════════╣
║  🔋 Saldo baixo no Mercado Bitcoin (R$ 50 restantes)       ║
║     Ação: Depositar mais fundos → [Como Depositar]         ║
╚═════════════════════════════════════════════════════════════╝
```

**Tipos de alertas**:
- 🔴 **Crítico**: Bot com erro, credenciais inválidas
- ⚠️ **Atenção**: Circuit breaker, saldo baixo, ordem pendente há muito tempo
- 💡 **Informativo**: Take profit próximo, novo sinal de Candle Bot
- ✅ **Sucesso**: Take profit atingido, bot criado com sucesso

---

### 8. Métricas Avançadas (Plano MAX)

**O que mostra** (exclusivo MAX):

```
╔═════════════════════════════════════════════════════════════╗
║  MÉTRICAS AVANÇADAS                                         ║
╠═════════════════════════════════════════════════════════════╣
║  📊 Sharpe Ratio: 1,85 (bom - acima de 1)                   ║
║     Retorno ajustado por risco vs holding Bitcoin           ║
╠═════════════════════════════════════════════════════════════╣
║  📉 Max Drawdown: -12,5% (R$ 625)                           ║
║     Maior queda de pico a vale (28/12 → 02/01)              ║
╠═════════════════════════════════════════════════════════════╣
║  🎯 Profit Factor: 2,3                                      ║
║     R$ 966 lucros / R$ 420 prejuízos                        ║
╠═════════════════════════════════════════════════════════════╣
║  ⏱️ Average Hold Time: 2,3 dias                             ║
║     Tempo médio entre compra e venda                        ║
╚═════════════════════════════════════════════════════════════╝
```

**Explicações contextuais** (hover):
```
[?] Sharpe Ratio:
    Mede retorno ajustado por risco.
    > 1 = Bom (retorno compensa risco)
    < 1 = Ruim (risco não compensa retorno)

    Seu 1,85 significa: Para cada 1% de risco, você ganha 1,85% de retorno.
```

---

## Regras de Negócio Principais

### 1. Cálculo de P&L (Profit & Loss)

#### P&L Realizado
```
P&L Realizado = Σ(vendas) - Σ(compras)

Exemplo:
  Compras: R$ 5.000
  Vendas: R$ 5.420
  P&L Realizado = R$ 420 (+8,4%)
```

#### P&L Não Realizado (Paper Gain/Loss)
```
P&L Não Realizado = (saldo_crypto_atual × preço_atual) - custo_médio

Exemplo:
  Possui: 0,01 BTC (comprado a média de R$ 180.000)
  Custo médio: 0,01 × R$ 180.000 = R$ 1.800
  Preço atual: R$ 200.000
  Valor atual: 0,01 × R$ 200.000 = R$ 2.000
  P&L Não Realizado: R$ 2.000 - R$ 1.800 = +R$ 200 (+11,1%)
```

#### P&L Total
```
P&L Total = P&L Realizado + P&L Não Realizado
```

---

### 2. Win Rate (Taxa de Acerto)

```
Win Rate = (trades_lucrativos / total_trades) × 100

Exemplo:
  Total de trades: 50
  Lucrativos: 34 (lucro > R$ 0)
  Perdedores: 16 (lucro < R$ 0)

  Win Rate = (34 / 50) × 100 = 68%
```

**Interpretação**:
- < 50%: Ruim (mais trades perdedores que vencedores)
- 50-60%: Mediano (break-even territory)
- 60-70%: Bom (maioria dos trades são lucrativos)
- > 70%: Excelente (alta taxa de acerto)

**Nota**: Win rate alto NÃO garante lucro. Pode ter 80% win rate mas se os 20% de perdas forem enormes, resultado final é prejuízo.

---

### 3. Profit Factor

```
Profit Factor = Total_Lucros / Total_Prejuízos

Exemplo:
  Lucros: R$ 966 (soma de todos os trades positivos)
  Prejuízos: R$ 420 (soma de todos os trades negativos)

  Profit Factor = 966 / 420 = 2,3
```

**Interpretação**:
- < 1: Prejuízo líquido (perdas > lucros)
- = 1: Break-even (lucros = perdas)
- 1-1,5: Lucrativo mas margem baixa
- 1,5-2: Bom (lucros são 1,5-2× maiores que perdas)
- > 2: Excelente (cada R$ 1 perdido gera R$ 2+ de lucro)

---

### 4. Max Drawdown (Maior Queda)

```
Max Drawdown = (Vale - Pico) / Pico × 100

Exemplo:
  Pico: R$ 5.000 (28/12)
  Vale: R$ 4.375 (02/01)

  Drawdown = (4.375 - 5.000) / 5.000 × 100 = -12,5%
```

**Interpretação**:
- < 10%: Excelente controle de risco
- 10-20%: Aceitável
- 20-30%: Atenção (risco moderado)
- > 30%: Alto risco (revisar estratégia)

**Uso**: Mede "pior cenário" que usuário experimentou. Importante para gestão emocional (se max drawdown foi -20%, usuário deve estar preparado para isso novamente).

---

### 5. Sincronização de Dados

```
Dashboard carrega dados de múltiplas fontes:

1. Banco de Dados Local (Crypteras):
   - Bots criados
   - Ordens executadas pelo Crypteras
   - P&L calculado
   - Configurações

2. Exchanges (via API):
   - Saldos atuais (tempo real)
   - Ordens pendentes
   - Ordens executadas fora do Crypteras (manual)

3. Market Data (CoinGecko/CoinMarketCap):
   - Preços atuais (BTC, ETH, etc)
   - Variação 24h
   - Market cap

Frequência de atualização:
- Dados locais: Tempo real (WebSocket - futuro)
- Saldos de exchange: A cada 5 minutos (cache)
- Preços de mercado: A cada 30 segundos
```

---

### 6. Limitações por Plano

| Funcionalidade | FREE | PRO | MAX |
|----------------|------|-----|-----|
| **Período histórico** | 30 dias | 90 dias | Ilimitado |
| **Exportação de dados** | ❌ | CSV | CSV + PDF |
| **Métricas avançadas** | ❌ | ❌ | ✅ (Sharpe, etc) |
| **Comparação com mercado** | ❌ | ✅ | ✅ |
| **Alertas proativos** | ❌ | 5/mês | Ilimitado |
| **Gráficos interativos** | Básicos | Completos | Avançados |

---

## Casos de Uso

### 1. Usuário FREE (João - Iniciante)

**Objetivo**: Acompanhar primeiro bot (DCA)

**Dashboard mostra**:
- Saldo total: R$ 500
- P&L 30 dias: +R$ 20 (+4%)
- 1 bot ativo (DCA BTC)
- Gráfico: Últimos 30 dias (limitado)
- Ordens: Últimas 10

**Experiência**:
```
João abre dashboard:
"💰 Saldo: R$ 520 (+R$ 20 em 30 dias)
🤖 DCA Bot ativo (próxima compra: amanhã 9h)
📊 Você está R$ 20 à frente!"

[Botão] Upgrade PRO → Desbloqueie gráficos avançados
```

---

### 2. Usuário PRO (Maria - Intermediária)

**Objetivo**: Otimizar múltiplos bots

**Dashboard mostra**:
- Saldo total: R$ 5.420
- P&L 90 dias: +R$ 420 (+8,4%)
- 7 bots (5 ativos, 2 pausados)
- Gráfico comparativo com Bitcoin
- Performance por estratégia

**Experiência**:
```
Maria abre dashboard:
"📊 Performance: 70% do Bitcoin (você: +8,4%, BTC: +12%)

💡 Insight: Candle Bots estão 2× melhor que Smart Bots.
   Sugestão: Realocar R$ 500 de Smart → Candle"

[Botão] Aplicar Sugestão
```

---

### 3. Usuário MAX (Carlos - Avançado)

**Objetivo**: Análise profunda e otimização contínua

**Dashboard mostra**:
- Saldo total: R$ 25.000
- P&L histórico completo (desde início)
- 15 bots ativos (múltiplas estratégias)
- Métricas avançadas (Sharpe, Sortino, Calmar)
- Correlação entre bots

**Experiência**:
```
Carlos abre dashboard:
"📊 Sharpe Ratio: 1,85 (bom)
📉 Max Drawdown: -12,5% (aceitável)
🎯 Profit Factor: 2,3 (excelente)

🔔 Alerta Proativo:
   Bot "ETH Swing" está correlacionado 0,95 com "BTC Swing"
   → Diversificação baixa (mesmo risco)
   → Sugestão: Adicionar SOL ou MATIC (correlação 0,3)"

[Botão] Ver Análise Completa
```

---

## Métricas de Sucesso do Dashboard

### KPIs de Engajamento
- **Daily Active Users (DAU)**: % de usuários que abrem dashboard diariamente (meta: > 40%)
- **Time on Dashboard**: Tempo médio por visita (meta: > 2 min)
- **Bounce Rate**: % que fecham em < 10s (meta: < 20%)

### KPIs de Valor
- **Upgrade from Dashboard**: % de FREE que fazem upgrade após ver limitações (meta: > 15%)
- **Action Rate**: % que executam ação (pausar bot, criar bot, etc) (meta: > 30%)
- **Alert Response Rate**: % que agem após alerta (meta: > 60%)

---

## Problemas Comuns e Soluções

### ❌ "Saldo no dashboard diferente da exchange"

**Causas**:
1. Cache desatualizado (5 min)
2. Ordem executada há pouco tempo (sincronização pendente)
3. Depósito/saque manual na exchange

**Solução**:
- Clique "🔄 Sincronizar Agora" (força atualização)
- Aguarde 1-2 minutos
- Se persistir: Vá em Profile → Credenciais → Testar Conexão

---

### ❌ "P&L está errado"

**Causas**:
1. Ordens manuais na exchange (fora do Crypteras)
2. Depósitos não rastreados
3. Saques não rastreados

**Solução**:
```
P&L considera APENAS ordens executadas pelo Crypteras.

Se você comprou Bitcoin manualmente na exchange:
→ Crypteras não sabe disso
→ P&L pode parecer incorreto

Recomendação:
✅ Use Crypteras para TODAS as operações
❌ Evite trades manuais paralelos
```

---

### ❌ "Gráfico não carrega"

**Causas**:
1. Período muito longo (MAX tentando carregar 2 anos de dados)
2. Internet lenta
3. Erro temporário

**Solução**:
- Reduza período (de "Tudo" para "90 dias")
- Recarregue página (F5)
- Limpe cache do navegador

---

## Limitações e Transparência

### O que Dashboard NÃO mostra

❌ **Ordens em exchanges externas**: Se você compra Bitcoin diretamente no Mercado Bitcoin (fora do Crypteras), não aparece no dashboard

❌ **P&L de outras plataformas**: Se você usa 3Commas ou outro bot simultaneamente, esses trades não são rastreados

❌ **Impostos calculados**: Dashboard mostra P&L bruto (não desconta IR)

❌ **Previsões de lucro futuro**: Métricas são históricas, não preditivas

### Transparência de Cálculos

**Dashboard SEMPRE explica**:
```
[?] Como calculamos P&L?
    P&L = (Vendas - Compras) + (Crypto atual × Preço atual - Custo médio)

    Seu caso:
    Vendas: R$ 5.420
    Compras: R$ 5.000
    Crypto atual: 0,01 BTC × R$ 200.000 = R$ 2.000
    Custo médio: R$ 1.800

    P&L = (5.420 - 5.000) + (2.000 - 1.800) = R$ 620
```

---

## Roadmap Futuro

### Melhorias Planejadas
- [ ] **Real-time updates**: WebSockets para atualização instantânea (sem refresh)
- [ ] **Mobile app**: Dashboard nativo iOS/Android
- [ ] **Widgets customizáveis**: Usuário arrasta/solta componentes
- [ ] **Metas pessoais**: "Quero R$ 10.000 até dezembro" (tracking)
- [ ] **Social features**: Comparar performance com amigos (opt-in)
- [ ] **Notificações push**: Alertas no celular (take profit, erro, etc)
- [ ] **Exportação automática**: Email semanal com resumo

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
