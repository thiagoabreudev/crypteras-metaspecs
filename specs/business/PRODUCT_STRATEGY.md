---
spec_version: "1.2.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.1.0"
status: "active"
category: "business"
tags: ['business', 'product_strategy']
---

# Estratégia do Produto - Crypteras

:::version_info
**Versão**: 1.2.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir direção estratégica do produto Crypteras para guiar decisões de features, pricing e roadmap.

**Constraints** (limites obrigatórios):
- Manter foco em mercado brasileiro (preço em BRL, suporte em português)
- Preço deve ser 7x menor que concorrentes internacionais (3Commas ~$99/mês → Crypteras MAX ~$29/mês)
- Target primário: traders iniciantes a intermediários (não experts)
- Simplicidade sobre features avançadas (80/20 - features que 80% dos usuários usam)
- Estratégias DEVEM ter proteção de capital (stop-loss, circuit breaker obrigatórios)
- Comunicação DEVE ser honesta (sem promessas de "ficar rico rápido")
- Roadmap alinhado com capacidade do time (evitar over-commitment)

**Non-Goals** (o que NÃO fazer):
- Competir com exchanges (não vamos criar exchange própria)
- Target traders profissionais/institucionais (nosso foco é varejo)
- Adicionar copy trading ou social trading no MVP (escopo futuro)
- Suportar trading de ações, forex ou commodities (apenas crypto)
- Criar app mobile nativo no MVP (apenas web responsivo)
- Internacionalização antes de validar mercado brasileiro (Brasil first)
- Adicionar gamificação complexa (badges, leaderboards) no MVP
- Prometidas de lucro garantido ou retorno fixo (ilegal e antiético)
:::

:::explainability
**Requirement**: ⚠️ Recommended (para decisões de produto e features)

**Output Format**:
IA DEVE explicar decisões de produto seguindo este formato:

```markdown
## 🤖 Decisão de Produto

**Decisão**: [O que foi decidido - ex: "Priorizar dashboard de performance ao invés de copy trading"]

**Source**:
- `specs/business/PRODUCT_STRATEGY.md` v1.2.0 - Seção relevante
- `specs/business/CUSTOMER_PERSONAS.md` - Persona aplicável
- Roadmap ou constraints de produto

**Rationale**:
1. [Alinhamento com visão e missão]
2. [Impacto em personas principais]
3. [Viabilidade técnica e negócio]
4. [Constraints de produto respeitadas]

**Alternatives Considered**:
1. ❌ [Alternativa 1] - [Por que não priorizar]
2. ❌ [Alternativa 2] - [Por que não priorizar]
3. ✅ [Escolhida] - [Por que priorizar]

**Trade-offs**:
- ✅ Pro: [Benefício para usuário]
- ✅ Pro: [Alinhamento estratégico]
- ⚠️ Con: [Desvantagem se houver]

**User Impact**:
- **Primary Persona**: [Como afeta persona principal]
- **Secondary Persona**: [Como afeta persona secundária]
- **Value Proposition**: [Valor agregado]
- **Adoption**: [Facilidade de adoção]

**Business Impact**:
- **Revenue**: [Impacto em receita]
- **Churn**: [Impacto em retenção]
- **Acquisition**: [Impacto em aquisição]
- **Roadmap**: [Prioridade no roadmap]

**Audit Trail**:
- Timestamp: [ISO 8601]
- Specs Consultadas: [PRODUCT_STRATEGY.md v1.x.x, CUSTOMER_PERSONAS.md]
- Personas Consideradas: [lista]
- Constraints Aplicados: [lista]
```

**Quando Explicar** (gatilhos obrigatórios):
1. **Escolha de feature para MVP**: Incluir vs pospor
2. **Decisão de UX que impacta persona principal**: Layout, workflow, copywriting
3. **Escolha de pricing ou planos**: FREE vs PRO vs MAX
4. **Priorização de roadmap**: Feature A antes de Feature B
5. **Decisão que viola Non-Goal**: Quando exceção é justificável
6. **Trade-off produto vs tecnologia**: Simplicidade vs poder, acessibilidade vs features
7. **Mudança de posicionamento**: Target, mensagem, diferenciação
8. **Feature deprecation**: Remover funcionalidade existente
9. **Onboarding ou comunicação**: Como apresentar produto ao usuário
10. **Métrica de sucesso**: Qual KPI priorizar

**Audit Trail Obrigatório**:
- Timestamp da decisão
- Specs consultadas (PRODUCT_STRATEGY, CUSTOMER_PERSONAS, etc)
- Personas afetadas
- Constraints de produto aplicados
- Impacto em visão/missão/roadmap

**Exemplo Completo**:

```markdown
## 🤖 Decisão de Produto

**Decisão**: Priorizar dashboard de analytics no MVP ao invés de copy trading social

**Source**:
- `specs/business/PRODUCT_STRATEGY.md` v1.2.0 - Non-Goal "copy trading no MVP"
- `specs/business/PRODUCT_STRATEGY.md` v1.2.0 - Constraint "Simplicidade sobre features avançadas"
- `specs/business/CUSTOMER_PERSONAS.md` v1.0.0 - Persona "João" (iniciante) precisa entender performance

**Rationale**:
1. **Non-Goal Definido**: Copy trading explicitamente fora do escopo MVP (escopo futuro)
2. **Persona Principal**: João (iniciante) precisa VER resultados antes de confiar no bot
3. **Trust Building**: Analytics transparente → confiança → retenção
4. **Viabilidade**: Dashboard é mais simples tecnicamente que copy trading (social graph, compliance)

**Alternatives Considered**:
1. ❌ Copy Trading - Fora do escopo MVP (Non-Goal), complexidade alta, risco regulatório
2. ❌ Gamificação (badges, leaderboards) - Também fora do MVP (Non-Goal), distrai de core value
3. ❌ Nenhum analytics - Usuário não sabe se bot funciona, alta churn esperada
4. ✅ Dashboard de Performance - Simples, essencial para trust, alinhado com MVP

**Trade-offs**:
- ✅ Pro: Trust building essencial para conversão FREE → PRO
- ✅ Pro: Ajuda João (persona) entender se estratégia funciona
- ✅ Pro: Simples de implementar (queries MongoDB, gráficos Chart.js)
- ⚠️ Con: Copy trading ficaria para v2.0 (aceitável - Non-Goal confirmado)

**User Impact**:
- **Primary Persona (João - Iniciante)**: Vê lucro/prejuízo claro, decide se continua usando
- **Secondary Persona (Ana - Estudante)**: Analytics = aprendizado sobre trading
- **Value Proposition**: "Veja exatamente quanto seus bots estão ganhando ou perdendo"
- **Adoption**: Alta - usuário quer ver resultados imediatamente

**Business Impact**:
- **Revenue**: Dashboard → confiança → upgrade FREE para PRO (~30% conversion esperada)
- **Churn**: Reduz churn de usuários iniciantes (transparência)
- **Acquisition**: Case studies com métricas reais → marketing mais eficaz
- **Roadmap**: MVP Sprint 3 (após bots funcionais)

**Audit Trail**:
- Timestamp: 2025-12-25T15:00:00Z
- Specs Consultadas: PRODUCT_STRATEGY.md v1.2.0, CUSTOMER_PERSONAS.md v1.0.0
- Personas Consideradas: João (primary), Ana (secondary)
- Constraints Aplicados: "Simplicidade sobre features avançadas", Non-Goal "copy trading no MVP"
- Roadmap Impact: MVP Sprint 3
```

**Quando NÃO Explicar** (decisões triviais):
- UX triviais sem impacto em personas (cor de botão, padding)
- Copywriting minor (ajustes de texto sem mudança de mensagem)
- Features óbvias já no roadmap aprovado
- Decisões de implementação já cobertas por specs técnicas
- Bugs fixes que restauram comportamento esperado
:::

:::breaking_changes
**v1.2.0**:
- Adicionada seção `:::explainability` com requisitos recomendados para decisões de produto
- Definidos 10 gatilhos para explicar decisões de features e UX
- Incluído exemplo completo de priorização de feature
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Estratégia de produto, visão e roadmap
:::

**Versão**: 1.0
**Data**: 2024-12-24

---

## Visão e Missão

### Visão (Onde Queremos Chegar)

> "Ser a plataforma líder de trading automatizado de criptomoedas na América Latina até 2027, reconhecida por democratizar acesso a estratégias profissionais através de IA acessível."

**Marcos de Visão**:
- **2025**: 500+ usuários pagantes no Brasil
- **2026**: 5.000+ usuários, expansão para Argentina e México
- **2027**: 20.000+ usuários, líder LATAM em trading automatizado

---

### Missão (Por Que Existimos)

> "Tornar trading automatizado de criptomoedas acessível para todos os brasileiros, democratizando estratégias profissionais através de tecnologia de IA, preços justos e suporte em português."

**Princípios Fundamentais**:
1. **Acessibilidade**: Preço brasileiro, não internacional
2. **Simplicidade**: Fácil para iniciantes, poderoso para experts
3. **Transparência**: Sem promessas de "ficar rico rápido"
4. **Segurança**: Proteção de capital como prioridade #1
5. **Inovação**: IA como diferenciador competitivo

---

## Posicionamento de Mercado

### Declaração de Posicionamento

```
Para traders brasileiros de criptomoedas (iniciantes a intermediários)
que querem automatizar operações sem perder controle,

Crypteras é uma plataforma SaaS de trading automatizado
que combina agentes de IA conversacionais com preços acessíveis,

Diferente de 3Commas e Cryptohopper (caros e complexos),
nosso produto oferece assistência de IA em português e preço 7x menor,

Porque acreditamos que trading automatizado deve ser acessível
para todos os brasileiros, não apenas para grandes traders.
```

---

### Matriz de Posicionamento Competitivo

```
         Alta Complexidade
               │
               │   3Commas
               │   (Expert traders)
               │
         ┌─────┼─────┐
         │     │     │
   Caro  │     │     │  Barato
         │     │     │
         └─────┼─────┘
               │ Crypteras
               │ (Iniciantes + IA)
               │
               │ Cryptohopper
         Baixa Complexidade
```

**Nosso Quadrante**: Baixa complexidade (fácil de usar) + Barato (preço brasileiro)

---

## Estratégia de Produto

### 1. Estratégia de Crescimento (2025-2027)

**Fase 1: Product-Market Fit (Q1-Q2 2025)**
- **Objetivo**: Validar se produto resolve dor do cliente
- **Meta**: 25 usuários pagantes/mês
- **Foco**: Onboarding perfeito, estabilidade, suporte excelente
- **Aprendizados-chave**:
  - Qual persona converte mais? (Lucas, Mariana, Rafael)
  - Qual tipo de bot é mais usado? (DCA, Candle, Smart)
  - Principal ponto de fricção no onboarding?
  - Taxa de churn real vs esperada?

**Fase 2: Crescimento Inicial (Q3-Q4 2025)**
- **Objetivo**: Escalar aquisição mantendo qualidade
- **Meta**: 100+ usuários pagantes/mês
- **Foco**: Lançar backtesting (diferenciador), conteúdo educacional, referral
- **Lançamentos**:
  - ✅ Backtesting (Prioridade #1)
  - ✅ Mais exchanges (OKX, Bybit)
  - ✅ Programa de referral

**Fase 3: Escala (2026)**
- **Objetivo**: Consolidar liderança no Brasil, iniciar LATAM
- **Meta**: 500+ usuários/mês
- **Foco**: Marketplace de bots, arbitragem, expansão internacional
- **Lançamentos**:
  - ✅ Marketplace de bots (receita adicional)
  - ✅ Arbitragem automatizada
  - ✅ Suporte em espanhol (Argentina, México)

**Fase 4: Liderança LATAM (2027)**
- **Objetivo**: Referência em trading automatizado na América Latina
- **Meta**: 1.000+ usuários/mês
- **Foco**: Enterprise features, API para desenvolvedores, parcerias

---

### 2. Estratégia de Diferenciação

**Como Nos Diferenciamos?**

| Dimensão | Concorrentes | Crypteras |
|----------|--------------|-----------|
| **Preço** | $29-99/mês (internacional) | R$ 19,90-97/mês (brasileiro) |
| **IA** | Não têm IA conversacional | Agentes de IA para configuração |
| **Complexidade** | Interface complexa, muitas features | Simples para iniciante, poderoso para expert |
| **Suporte** | Inglês apenas | Português nativo |
| **Foco Geográfico** | Global (genérico) | Brasil (específico) |
| **Onboarding** | Manual, documentação técnica | Assistido por IA |

**Diferenciação Sustentável**:
1. **IA Conversacional** → Difícil de copiar (requer investimento em ML/NLP)
2. **Preço Brasileiro** → Vantagem estrutural (operação no Brasil)
3. **Comunidade Local** → Network effect (quanto mais brasileiros, mais valor)

---

### 3. Estratégia de Monetização

**Modelo Freemium**:

```
FREE (R$ 0/mês)
↓ (15-25% convertem em 30 dias)
PRO (R$ 19,90/mês)
↓ (10-15% convertem em 90 dias)
MAX (R$ 97/mês)
```

**Drivers de Conversão**:
- FREE → PRO: Limite de 1 bot/estratégia atingido
- PRO → MAX: Limite de 3 bots/estratégia atingido + Backtesting

**Receitas Adicionais (Futuro)**:
1. **Marketplace de Bots** (comissão de 20-30%)
   - Traders vendem estratégias por R$ 50-200
   - Crypteras leva 25% por transação
   - Potencial: +R$ 10.000/mês com 500 vendas/mês

2. **API Enterprise** (R$ 500/mês)
   - Para fundos de investimento e traders profissionais
   - Acesso programático, rate limits maiores

3. **Consultoria de Estratégias** (R$ 200/hora)
   - Sessão 1-on-1 para otimizar estratégias
   - Apenas para clientes MAX

---

### 4. Estratégia de Produto (Build, Buy, Partner)

**Build (Desenvolver Internamente)**:
- ✅ DCA, Candle, SmartBots (core product)
- ✅ Agentes de IA (diferenciador)
- ✅ Dashboard centralizado
- ✅ Backtesting (roadmap #1)
- ✅ Arbitragem (roadmap #3)

**Buy (Comprar/Licenciar)**:
- ❌ Não aplicável (estágio inicial)

**Partner (Parcerias)**:
- 🤝 **Exchanges**: Binance, Mercado Bitcoin (API integration)
- 🤝 **Stripe/Mercado Pago**: Pagamentos
- 🤝 **Influencers cripto**: Marketing de afiliados (futuro)
- 🤝 **TradingView**: Integração de gráficos (futuro)

---

## Prioridades Estratégicas

### Top 3 Prioridades (2025)

**Prioridade #1: Backtesting** 🥇
- **Por quê**: Principal objeção de conversão ("Quero testar antes de arriscar")
- **Impacto esperado**: +30% conversão FREE → PRO
- **Timeline**: Q1-Q2 2025
- **Recurso**: 1 dev full-time por 2 meses

**Prioridade #2: Estabilidade e Confiabilidade** 🥈
- **Por quê**: Bugs destroem confiança → churn alto
- **Impacto esperado**: Reduzir churn de 10% para 5%
- **Timeline**: Contínuo
- **Ações**:
  - Resolver chatbot de IA (atualmente quebrado)
  - Monitoramento 24/7 de bots
  - Testes automatizados (cobertura 80%+)

**Prioridade #3: Conteúdo Educacional** 🥉
- **Por quê**: SEO orgânico (ads cripto são restritos)
- **Impacto esperado**: +50% tráfego orgânico
- **Timeline**: Q1-Q4 2025
- **Ações**:
  - Blog: 2 posts/semana
  - YouTube: 1 vídeo/semana
  - Tutoriais embutidos no produto

---

## Roadmap de Produto

### Q1 2025 (Jan-Mar)

**Foco**: Product-Market Fit + Estabilidade

- ✅ **Resolver chatbot de IA** (bloqueador de conversão)
- ✅ **Melhorar onboarding** (tutorial interativo)
- ✅ **Monitoramento de bots** (alertas se bot travar)
- 🔮 **Backtesting MVP** (versão simplificada)
- 🔮 **Email automation** (sequência de boas-vindas)

**KPIs**:
- 25 usuários pagantes/mês
- Churn < 10%
- NPS > 30

---

### Q2 2025 (Abr-Jun)

**Foco**: Crescimento + Diferenciação

- ✅ **Backtesting completo** (versão final)
- ✅ **Adicionar OKX exchange**
- ✅ **Programa de referral** (incentivo R$ 10 OFF)
- 🔮 **Comunidade Discord** (engajamento)
- 🔮 **Blog SEO** (10 posts de alta qualidade)

**KPIs**:
- 50 usuários pagantes/mês
- 15% conversão FREE → PRO
- 20% de novos usuários via referral

---

### Q3 2025 (Jul-Set)

**Foco**: Escala + Receitas Adicionais

- ✅ **Marketplace de bots MVP** (compra/venda)
- ✅ **Adicionar Bybit exchange**
- ✅ **App mobile** (PWA responsivo)
- 🔮 **Arbitragem automatizada MVP**
- 🔮 **YouTube channel** (1 vídeo/semana)

**KPIs**:
- 100 usuários pagantes/mês
- 10% conversão PRO → MAX
- R$ 2.000/mês receita marketplace

---

### Q4 2025 (Out-Dez)

**Foco**: Consolidação + Expansão Internacional

- ✅ **Arbitragem completa** (multi-exchange)
- ✅ **Suporte em espanhol** (preparação LATAM)
- ✅ **Integração TradingView** (charts)
- 🔮 **API Enterprise** (para fundos)
- 🔮 **Parcerias com influencers** (marketing)

**KPIs**:
- 150 usuários pagantes/mês
- MRR R$ 10.000+
- Churn < 5%

---

### 2026+ (Longo Prazo)

**Visão Futura**:
- 🔮 **Copy Trading** (copiar traders profissionais)
- 🔮 **Social Trading** (rede social de traders)
- 🔮 **IA Preditiva** (machine learning para previsões)
- 🔮 **Expansion LATAM** (Argentina, México, Chile)
- 🔮 **White-label** (para exchanges oferecerem Crypteras)

---

## Princípios de Produto

### 1. Simplicidade Progressiva

**Princípio**:
> "Simples para iniciantes, poderoso para experts"

**Aplicação**:
- **Modo Simples**: 3 inputs (tipo bot, moeda, budget)
- **Modo Avançado**: 20+ parâmetros customizáveis
- Usuário escolhe nível de complexidade

**Anti-padrão**:
❌ Mostrar todos os 20 parâmetros de uma vez (sobrecarga cognitiva)

---

### 2. IA como Assistente, Não Substituto

**Princípio**:
> "IA sugere, humano decide"

**Aplicação**:
- IA recomenda estratégia, mas usuário aprova
- IA valida sinais, mas não executa sem confirmação (exceto se configurado)
- IA explica "por quê", não apenas "o quê"

**Anti-padrão**:
❌ "IA faz tudo sozinho" → Usuário perde controle e confiança

---

### 3. Transparência Sobre Riscos

**Princípio**:
> "Nunca prometemos retorno garantido"

**Aplicação**:
- Disclaimers claros: "Trading envolve risco de perda"
- Mostrar P&L negativo honestamente (não esconder perdas)
- Educar sobre gestão de risco

**Anti-padrão**:
❌ Marketing agressivo "Ganhe R$ 10.000/mês garantido!"

---

### 4. Feedback em Tempo Real

**Princípio**:
> "Usuário sempre sabe o que está acontecendo"

**Aplicação**:
- Dashboard atualiza a cada 30s
- Notificações de trades executados
- Status de bot visível (rodando, pausado, erro)

**Anti-padrão**:
❌ "Bot rodando" mas usuário não sabe se está funcionando

---

### 5. Proteção de Capital

**Princípio**:
> "Não perder dinheiro é mais importante que ganhar"

**Aplicação**:
- Circuit breaker obrigatório (não pode desabilitar)
- Stop-loss padrão (2% soft, 5% hard)
- Validações de sanity check antes de executar

**Anti-padrão**:
❌ Permitir configurações perigosas sem avisos

---

## Métricas de Sucesso do Produto

### Métricas de Adoção

| Métrica | Definição | Meta 2025 |
|---------|-----------|-----------|
| **Cadastros FREE** | Novos usuários/mês | 200/mês (Q4) |
| **Taxa de ativação** | % que cria 1º bot em 7d | 60%+ |
| **Usuários ativos** | Login 2+ vezes/mês | 70% da base |

### Métricas de Conversão

| Métrica | Definição | Meta 2025 |
|---------|-----------|-----------|
| **FREE → PRO (30d)** | % conversão após 30d no FREE | 15-25% |
| **PRO → MAX (90d)** | % conversão após 90d no PRO | 10-15% |
| **MRR** | Monthly Recurring Revenue | R$ 10.000+ (Q4) |

### Métricas de Retenção

| Métrica | Definição | Meta 2025 |
|---------|-----------|-----------|
| **Churn mensal** | % cancelamentos/mês | < 5% |
| **LTV** | Lifetime Value médio | R$ 600+ (2 anos PRO) |
| **NPS** | Net Promoter Score | 40+ |

### Métricas de Qualidade

| Métrica | Definição | Meta 2025 |
|---------|-----------|-----------|
| **Uptime** | Disponibilidade do sistema | 99.5%+ |
| **Bugs críticos** | Bugs que travam bots | < 1/mês |
| **Tempo resposta suporte** | SLA de email | < 24h |

---

## Trade-offs e Decisões Estratégicas

### Trade-off 1: Simplicidade vs Funcionalidades

**Decisão**: Priorizar simplicidade para FREE/PRO, funcionalidades para MAX

**Rationale**:
- 90% dos usuários são iniciantes/intermediários (Lucas, Mariana)
- Power users (que precisam de tudo) são 10% mas pagam MAX
- Melhor ter 1.000 usuários PRO felizes que 100 usuários MAX frustrados

**Aplicação**:
- FREE/PRO: Interface simples, wizards guiados
- MAX: Modo avançado, API, customizações extremas

---

### Trade-off 2: Muitas Exchanges vs Qualidade

**Decisão**: Adicionar exchanges gradualmente (2 → 5 → 10 → 20)

**Rationale**:
- Melhor ter 2 exchanges funcionando perfeitamente que 20 bugadas
- Binance + Mercado Bitcoin = 80% do mercado brasileiro
- Cada exchange adicional requer 2-4 semanas de integração + testes

**Roadmap de Exchanges**:
- Q1 2025: Binance, Mercado Bitcoin (atual)
- Q2 2025: +OKX
- Q3 2025: +Bybit
- Q4 2025: +KuCoin, +Kraken
- 2026: +15 exchanges (paridade com 3Commas)

---

### Trade-off 3: Desenvolver In-house vs Integrar Terceiros

**Decisão**: Desenvolver core in-house, integrar ferramentas auxiliares

**Core (In-house)**:
- Bots de trading (DCA, Candle, Smart)
- Agentes de IA
- Backtesting
- Arbitragem

**Terceiros (Integração)**:
- Pagamentos (Stripe/Mercado Pago)
- Gráficos (TradingView)
- Email (SendGrid)
- Monitoramento (Sentry)

**Rationale**: Core é diferenciação competitiva, terceiros são commodities.

---

## Riscos Estratégicos

### Risco 1: Regulação Brasileira 🔴

**Descrição**: CVM ou Banco Central proíbem bots de trading automatizado

**Probabilidade**: Baixa (5%)
**Impacto**: Crítico (fim do negócio)

**Mitigação**:
- Monitorar regulação constantemente
- Consultar advogados especializados em cripto
- Ter plano de pivô (ex: educação financeira)

---

### Risco 2: Concorrentes Grandes Entrando no Brasil 🟡

**Descrição**: 3Commas lança versão em português com preço brasileiro

**Probabilidade**: Média (30%)
**Impacto**: Alto (perda de diferenciação)

**Mitigação**:
- Acelerar desenvolvimento de IA (diferenciador único)
- Construir comunidade forte (network effect)
- Lançar marketplace antes deles

---

### Risco 3: Crash do Mercado Cripto 🟡

**Descrição**: Bear market severo → usuários saem de cripto

**Probabilidade**: Média (40% em 2-3 anos)
**Impacto**: Médio (churn aumenta, cadastros caem)

**Mitigação**:
- Educar que DCA funciona melhor em bear market
- Diversificar para outros ativos (forex, ações - futuro)
- Manter custos baixos para sobreviver bear market

---

### Risco 4: Bugs Críticos Causando Perdas 🔴

**Descrição**: Bug no bot causa perda de R$ 10.000+ de um cliente

**Probabilidade**: Baixa (10%)
**Impacto**: Crítico (processo judicial, reputação)

**Mitigação**:
- Testes automatizados rigorosos (80%+ cobertura)
- Paper trading obrigatório por 7 dias antes de real trading (futuro)
- Seguro de responsabilidade civil (quando crescer)

---

## Versionamento

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 2024-12-24 | Estratégia inicial de produto |

---

**Próxima Revisão**: Trimestral (ou após mudanças significativas de mercado)
