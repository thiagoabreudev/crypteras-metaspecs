---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "business"
tags: ['business', 'voice_of_customer']
---

# Voz do Cliente - Crypteras

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Coletar e documentar feedback de usuários para guiar roadmap de produto.

**Constraints** (limites obrigatórios):
- Usar múltiplos canais: NPS, support tickets, entrevistas
- Categorizar feedback por tema (UX, pricing, features)
- Priorizar por frequência e impacto
- Fechar loop com usuários (comunicar quando feedback é implementado)

**Non-Goals** (o que NÃO fazer):
- Implementar tudo que usuários pedem (manter visão de produto)
- Coletar feedback passivamente sem ação
- Criar sistema complexo de votação de features
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Feedback, objeções e requisitos dos clientes
:::

**Versão**: 1.0
**Data**: 2024-12-24

---

## Visão Geral

Este documento captura padrões de feedback esperado, linguagem do cliente, objeções comuns e comparações competitivas. Como o Crypteras está em pré-lançamento, este documento é baseado em:
- Análise de mercado e concorrentes
- Feedback de usuários de plataformas similares (3Commas, Cryptohopper)
- Personas definidas
- Pesquisa de comunidades cripto brasileiras

---

## Temas de Elogios (Feedback Positivo Esperado)

### Tema 1: Preço Acessível 💰

**Citações Esperadas**:
> "Finalmente um bot de trading que não custa uma fortuna! R$ 19,90 é menos que uma pizza."

> "Testei 3Commas por 1 mês (US$ 49) e aqui pago R$ 20. Mesmas funcionalidades, preço brasileiro."

> "Plano FREE permitiu testar sem risco. Quando vi que funcionava, R$ 19,90 foi fácil de justificar."

**Contexto**:
- Comparação com 3Commas ($29-99/mês = R$ 145-495/mês)
- Principal diferenciador para mercado brasileiro
- Baixa barreira de entrada para iniciantes

**Ação para IA**:
- Reforçar valor: "Sim! Nossa missão é democratizar trading automatizado no Brasil"
- Evitar: "É barato porque é inferior" → Explicar: "Preço brasileiro + eficiência operacional"

---

### Tema 2: Agentes de IA Úteis 🤖

**Citações Esperadas** (quando chatbot funcionar):
> "Conversei com o assistente de IA e ele configurou meu bot em 5 minutos. Não precisei ler manual."

> "Perguntei 'qual estratégia é melhor para iniciante?' e a IA sugeriu DCA conservador. Funcionou!"

> "Adoro que a IA valida sinais antes de executar. Me sinto mais seguro."

**Contexto**:
- Diferencial único vs concorrentes (3Commas não tem IA conversacional)
- Reduz fricção no onboarding
- Aumenta confiança em decisões

**Ação para IA**:
- Capitalizar: "Nossos agentes de IA são treinados especificamente para trading brasileiro"
- Transparência: "A IA sugere, mas você sempre tem controle final"

---

### Tema 3: Centralização Multi-Exchange 📊

**Citações Esperadas**:
> "Ver Binance + Mercado Bitcoin no mesmo dashboard economiza meu tempo."

> "Antes eu tinha 3 abas abertas. Agora só preciso do Crypteras."

> "Consolidação de P&L entre exchanges é perfeita. Sei exatamente quanto lucrei."

**Contexto**:
- Gerenciar múltiplas exchanges é dor comum
- Competidor Bitsgap cobra caro por isso
- Crypteras oferece nativamente desde FREE tier

**Ação para IA**:
- Educar: "Pode conectar quantas exchanges quiser no mesmo plano"
- Upsell: "No PRO, você pode rodar estratégias diferentes em cada exchange"

---

### Tema 4: Simplicidade para Iniciantes 🎓

**Citações Esperadas**:
> "Nunca usei bot antes. Criei meu primeiro DCABot em 10 minutos."

> "Tutoriais em português são ótimos. 3Commas só tem em inglês."

> "Modo 'Simples' esconde parâmetros avançados. Não me sinto sobrecarregado."

**Contexto**:
- Lucas (persona iniciante) representa 60% da base esperada
- Concorrentes são muito técnicos
- Simplicidade = baixa fricção = alta conversão

**Ação para IA**:
- Validar: "Ótimo começo! DCABot é perfeito para aprender"
- Educar gradualmente: "Quando se sentir confortável, pode explorar CandleBots"

---

## Solicitações Frequentes (Feature Requests)

### Top 10 Solicitações Esperadas

| # | Solicitação | Frequência | Status | Resposta da IA |
|---|-------------|------------|--------|----------------|
| 1 | **Backtesting** | Muito Alta | 🔮 Roadmap Prioridade #1 | "Está em desenvolvimento! Prioridade máxima." |
| 2 | **Mais exchanges** (OKX, Bybit, KuCoin) | Alta | 🔮 Roadmap | "Planejado! Queremos ter 14+ exchanges como 3Commas." |
| 3 | **App mobile nativo** | Alta | 🔮 Futuro | "Por enquanto, o site é responsivo. App nativo no roadmap." |
| 4 | **Notificações WhatsApp** | Média | 🔮 Futuro | "Atualmente temos email. WhatsApp está planejado." |
| 5 | **Marketplace de bots** | Média | 🔮 Roadmap | "Sim! Planejado para MAX tier." |
| 6 | **Copy trading** (copiar traders) | Média | 🔮 Futuro | "Ótima ideia! Estamos avaliando." |
| 7 | **Arbitragem automatizada** | Alta | 🔮 Roadmap Prioridade #3 | "Está no roadmap! Após backtesting." |
| 8 | **Integração TradingView** | Média | 🔮 Futuro | "Planejado para PRO/MAX." |
| 9 | **API para desenvolvedores** | Baixa | 🔮 Futuro | "Interessante para power users. Vamos considerar." |
| 10 | **Mais indicadores técnicos** | Média | ✅ Já tem 10+ | "Já temos RSI, MACD, BB, VWAP, MFI, etc. Qual falta?" |

---

## Objeções Comuns e Tratamento

### Objeção 1: "Bots de trading são seguros?"

**Variações**:
- "Vocês não vão roubar minhas credenciais de API?"
- "E se hackearem o Crypteras e roubarem meu dinheiro?"
- "Já ouvi falar de exchange que sumiu com tudo..."

**Perfil**: Rafael (conservador), Lucas (iniciante preocupado)

**Tratamento**:
```
Resposta da IA:

"Entendo sua preocupação! Segurança é nossa prioridade #1:

✅ NÃO guardamos suas credenciais em texto puro (criptografia AES-256)
✅ API keys TEM PERMISSÃO apenas para LER saldos e CRIAR ORDENS
✅ API keys NÃO TÊM PERMISSÃO para SACAR (withdraw disabled)
✅ Seu dinheiro NUNCA sai da exchange (fica na Binance/MB)

Em resumo: Mesmo se hackearem o Crypteras, NÃO podem sacar seu dinheiro.

Quer mais detalhes sobre nossa arquitetura de segurança?"
```

**Escalonamento**:
- Se ainda preocupado → Link para FAQ de segurança detalhado
- Se menciona "regulação" → Explicar que exchanges é que são reguladas, não bots

---

### Objeção 2: "E se o bot errar e eu perder tudo?"

**Variações**:
- "Bot pode travar e vender tudo no prejuízo?"
- "E se der bug e ele comprar R$ 100.000 de Bitcoin?"
- "Tenho medo de configurar errado..."

**Perfil**: Rafael (conservador), Lucas (iniciante)

**Tratamento**:
```
Resposta da IA:

"Ótima pergunta! Temos várias proteções:

🛡️ CIRCUIT BREAKER: Se perder 10% em 1 dia, bot para automaticamente
🛡️ STOP-LOSS DUPLO: Soft (2%) ativa venda urgente, Hard (5%) vende tudo
🛡️ LIMITE DE BUDGET: Você define max R$ X por operação
🛡️ SANITY CHECK: Validamos preços antes de executar (rejeita se >= 5% fora do mercado)

Recomendo começar com:
- Plano FREE (sem risco financeiro além do que você alocar)
- DCABot conservador (R$ 50-100/semana)
- Budget baixo (R$ 500 total)

Depois de 2-4 semanas, aumenta se estiver confiante. Quer ajuda para configurar?"
```

**Escalonamento**:
- Se quer testar sem risco → Sugerir Paper Trading (se disponível)
- Se quer garantias → Explicar que trading sempre tem risco, mas ferramentas minimizam

---

### Objeção 3: "R$ 19,90/mês não compensa se eu não lucrar"

**Variações**:
- "Prefiro fazer manual e economizar R$ 20"
- "3 bots no FREE são suficientes para mim"
- "Vou esperar mais tempo antes de pagar"

**Perfil**: Lucas (iniciante), Rafael (conservador)

**Tratamento**:
```
Resposta da IA:

"Faz sentido! Vamos calcular juntos:

SE você lucra 1% ao mês com R$ 2.000 investidos:
→ Lucro: R$ 20/mês
→ Custo Crypteras PRO: R$ 19,90/mês
→ Lucro líquido: ~R$ 0 (empate)

MAS:
✅ PRO permite 3 bots/estratégia = 9 bots totais
✅ Diversificação reduz risco (BTC, ETH, ADA)
✅ Tempo economizado: ~5h/mês (vale quanto para você?)

SE você lucra 2% ao mês:
→ Lucro: R$ 40/mês
→ Custo: R$ 19,90/mês
→ Lucro líquido: R$ 20/mês (ROI de 100%!)

Recomendo: Fique no FREE até ter 3+ trades lucrativos. Aí o PRO se paga sozinho.

Faz sentido?"
```

**Escalonamento**:
- Se insistir em FREE → "Sem problemas! FREE é ilimitado no tempo"
- Se menciona ROI → Mostrar casos de usuários similares (quando disponível)

---

### Objeção 4: "Crypteras tem TUDO que 3Commas oferece?"

**Variações**:
- "3Commas tem 20 exchanges. Vocês têm 2..."
- "Falta TradingView integration"
- "Não tem SmartTrade (ordens complexas)"

**Perfil**: Mariana (intermediária/avançada)

**Tratamento**:
```
Resposta da IA:

"Honestamente? Ainda não temos TUDO que 3Commas oferece. Somos mais novos.

✅ O QUE TEMOS:
- DCA, Candle, SmartBots (3 tipos vs 2 do 3Commas)
- Agentes de IA conversacionais (3Commas NÃO tem)
- Preço 7x mais barato (R$ 19,90 vs US$ 49)
- Suporte em português (3Commas só inglês)

🔮 O QUE ESTÁ VINDO:
- Backtesting (Prioridade #1 - próximos 3 meses)
- Mais exchanges (OKX, Bybit, KuCoin)
- Arbitragem automatizada
- Marketplace de bots

❌ O QUE FALTA AINDA:
- TradingView integration (roadmap futuro)
- 20+ exchanges (temos 2, adicionando gradualmente)

Nossa vantagem: IA + preço brasileiro. Vale testar no FREE e decidir?
```

**Escalonamento**:
- Se power user → Oferecer acesso antecipado a features beta
- Se deal-breaker → Agradecer feedback e pedir para revisitar em 6 meses

---

### Objeção 5: "Prefiro Tesouro Direto / Renda Fixa"

**Variações**:
- "Cripto é muito volátil"
- "Não quero arriscar meu dinheiro"
- "Renda fixa é mais seguro"

**Perfil**: Rafael (conservador extremo)

**Tratamento**:
```
Resposta da IA:

"Respeito 100%! Tesouro Direto é ótimo para preservação de capital.

Cripto é DIFERENTE:
✅ Renda Fixa: Baixo risco, 10-12% ao ano (previsível)
✅ Cripto: Médio/Alto risco, -50% a +200% ao ano (volátil)

Cripto NÃO substitui renda fixa. É COMPLEMENTAR:

Sugestão de carteira conservadora:
- 70% Renda Fixa (Tesouro, CDB)
- 20% Ações (B3)
- 10% Cripto (hedge contra inflação)

SE alocar 10% em cripto:
→ Use Crypteras para DCABot conservador
→ Compra automática mensal (R$ 100-200)
→ Reduz volatilidade (compra na alta E na baixa)

Não é para ficar rico rápido. É diversificação inteligente.

Faz sentido para você?"
```

**Escalonamento**:
- Se totalmente avesso a risco → Agradecer e não insistir
- Se aberto a pequena alocação → Sugerir DCA com valor mínimo

---

## Comparações Competitivas

### Crypteras vs 3Commas

**Quando clientes comparam**:

| Critério | 3Commas | Crypteras | Vencedor |
|----------|---------|-----------|----------|
| **Preço** | $29-99/mês (R$ 145-495) | R$ 19,90-97/mês | ✅ **Crypteras** (7-8x mais barato) |
| **Exchanges** | 20+ exchanges | 2 (Binance, MB) | ❌ 3Commas |
| **IA Conversacional** | ❌ Não tem | ✅ Agentes de IA | ✅ **Crypteras** |
| **Backtesting** | ✅ Tem | 🔮 Roadmap | ❌ 3Commas |
| **TradingView** | ✅ Integrado | 🔮 Roadmap | ❌ 3Commas |
| **Suporte em Português** | ❌ Apenas inglês | ✅ Português nativo | ✅ **Crypteras** |
| **Marketplace de Bots** | ✅ Tem | 🔮 Roadmap | ❌ 3Commas |
| **Complexidade** | Alta (muitos recursos) | Média (foca no essencial) | ✅ **Crypteras** (para iniciantes) |
| **Estabilidade** | Muito alta (5+ anos) | Desconhecida (novo) | ❌ 3Commas |

**Mensagem da IA**:
> "Se você precisa de 20 exchanges e TradingView HOJE, escolha 3Commas. Se quer IA conversacional + preço brasileiro + funcionalidades essenciais, Crypteras é ideal. Teste FREE e compare!"

---

### Crypteras vs Cryptohopper

| Critério | Cryptohopper | Crypteras | Vencedor |
|----------|--------------|-----------|----------|
| **Preço** | $19-99/mês | R$ 19,90-97/mês | ≈ **Empate** (similar) |
| **Exchanges** | 10+ | 2 | ❌ Cryptohopper |
| **IA Conversacional** | ❌ Não tem | ✅ Tem | ✅ **Crypteras** |
| **Marketplace** | ✅ Tem (500+ bots) | 🔮 Roadmap | ❌ Cryptohopper |
| **Paper Trading** | ✅ Tem | 🔮 Planejado | ❌ Cryptohopper |
| **Suporte Português** | ❌ Inglês | ✅ Português | ✅ **Crypteras** |

---

### Crypteras vs Bitsgap

| Critério | Bitsgap | Crypteras | Vencedor |
|----------|---------|-----------|----------|
| **Foco Principal** | Arbitragem | Trading automatizado | - |
| **Preço** | $29-149/mês | R$ 19,90-97/mês | ✅ **Crypteras** |
| **Arbitragem** | ✅ Nativa | 🔮 Roadmap | ❌ Bitsgap |
| **Grid Trading** | ✅ Tem | ❌ Não tem | ❌ Bitsgap |
| **IA** | ❌ Não tem | ✅ Tem | ✅ **Crypteras** |

---

## Padrões de Linguagem do Cliente

### Terminologia que Clientes Usam

**Iniciantes (Lucas)**:
- "Robo" (não "bot")
- "Comprar automático" (não "DCA")
- "Parar perda" (não "stop-loss")
- "Gráfico de vela" (não "candlestick chart")

**Intermediários (Mariana)**:
- "Bot de DCA"
- "Stop-loss"
- "Candle analysis"
- "Spread bid-ask"

**Conservadores (Rafael)**:
- "Automatizar investimento"
- "Segurança"
- "Risco controlado"
- "Diversificação"

### Preferências de Comunicação

**Tom de Voz Esperado do Cliente**:
- **Lucas**: Casual, curioso, faz muitas perguntas
- **Mariana**: Direto, técnico, compara features
- **Rafael**: Formal, cauteloso, pergunta sobre riscos

**Canais Preferidos**:
- **Email**: Todos os perfis
- **WhatsApp**: Lucas e Rafael (urgências)
- **Discord/Telegram**: Mariana (comunidade)

---

## Indicadores de Satisfação vs Insatisfação

### Sinais de Cliente Satisfeito 😊

1. **NPS 9-10**: "Recomendaria definitivamente!"
2. **Login frequente**: 3+ vezes/semana
3. **Múltiplos bots ativos**: Usa limite máximo do plano
4. **Feedback positivo**: Responde pesquisas com elogios
5. **Referral orgânico**: Menciona Crypteras em comunidades
6. **Upgrade**: FREE → PRO ou PRO → MAX
7. **Engajamento**: Ajusta parâmetros, testa estratégias

### Sinais de Cliente Insatisfeito 😞

1. **NPS 0-6**: "Não recomendaria"
2. **Inatividade**: Não faz login há 14+ dias
3. **Bots pausados**: Todos os bots inativos
4. **Tickets de suporte**: Reclamações recorrentes
5. **Busca cancelamento**: "Como cancelo minha conta?"
6. **Feedback negativo**: Críticas em pesquisas
7. **P&L negativo**: Perdas consistentes 3+ meses

---

## Temas de Reclamação (Esperados)

### Reclamação 1: "Bot travou / não executou trade"

**Frequência Esperada**: Alta (inevitável em software)

**Resposta da IA**:
```
"Desculpe pelo problema! Vamos resolver:

1. Verifique se o bot está ativo no dashboard
2. Confirme se há saldo disponível na exchange
3. Veja logs para identificar erro

Se o problema persistir:
→ Acesse Suporte → Abra ticket com ID do bot
→ Resposta em até 24h

Enquanto isso, quer que eu pause o bot para evitar mais problemas?"
```

---

### Reclamação 2: "Perdi dinheiro / bot vendeu no prejuízo"

**Frequência Esperada**: Média (volatilidade cripto)

**Resposta da IA**:
```
"Entendo sua frustração. Vamos analisar:

Trading de cripto SEMPRE tem risco. Algumas vendas com prejuízo são NORMAIS.

Verifique:
1. Foi stop-loss? (Proteção automática)
2. Foi circuit breaker? (Perda diária > 10%)
3. Foi venda manual sua?

Se foi stop-loss:
→ É uma PROTEÇÃO. Evitou perda maior.
→ Quer ajustar o threshold? (ex: 2% → 3%)

Se foi erro do bot:
→ Abra ticket para análise técnica
→ Vamos investigar e compensar se for bug

Importante: Nunca invista mais do que pode perder. Quer revisar sua estratégia?"
```

---

### Reclamação 3: "Suporte demora muito para responder"

**Frequência Esperada**: Média (especialmente se crescer rápido)

**Resposta da IA**:
```
"Desculpe pela demora! Nossa meta é 24h.

Enquanto aguarda:
1. Consulte nossa FAQ: [link]
2. Veja tutoriais em vídeo: [link]
3. Entre na comunidade Discord: [link]

Para urgências (bot travado, perda de dinheiro):
→ Marque ticket como URGENTE
→ Prioridade máxima (resposta em 4-6h)

Seu ticket atual: #12345 (status: em análise)
Previsão de resposta: Hoje às 18h
```

---

## Versionamento

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 2024-12-24 | Criação inicial baseada em pesquisa de mercado |

---

**Próxima Revisão**: Após primeiros 50 clientes (validar feedback real vs esperado)
