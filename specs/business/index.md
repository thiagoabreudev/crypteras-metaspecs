---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "business"
tags: ['business', 'index']
---

# Contexto Empresarial - Crypteras Trading System

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Primeira versão versionada de index.md
:::

**Versão**: 1.0.0
**Data**: 2025-12-25
**Status**: Ativo

---

## Perfil de Contexto Empresarial

### Fundação da Empresa

**Nome**: Crypteras
**Website**: https://crypteras.tech
**Indústria**: FinTech - Trading Automatizado de Criptomoedas
**Mercado-Alvo**: Brasil (com planos de expansão internacional futura)
**Estágio**: Pre-lançamento (preparando para primeiros 25 usuários pagantes)

### Informações do Produto

**Nome do Produto**: Crypteras Trading System
**Categoria**: SaaS - Plataforma de Trading Automatizado com IA
**Proposta de Valor**:
> "Tornar trading automatizado de criptomoedas acessível para todos os brasileiros através de bots inteligentes com suporte de IA, preços competitivos e centralização multi-exchange."

**Diferenciação Principal**:
1. **Agentes de IA conversacionais** que auxiliam na configuração e otimização de estratégias
2. **Preço 7-8x mais acessível** que concorrentes internacionais (3Commas)
3. **Plataforma brasileira** com suporte em português e foco em exchanges locais (Mercado Bitcoin)
4. **Centralização multi-exchange** - gerencie todas suas exchanges em um só lugar

### Modelo de Negócio

**Tipo**: SaaS - Subscription-based (Freemium)
**Modelo de Receita**: Assinaturas mensais recorrentes
**Planos de Pricing**:

| Plano | Preço | Bots por Estratégia | Recursos Principais |
|-------|-------|---------------------|---------------------|
| **FREE** | R$ 0/mês | 1 bot/estratégia | Acesso básico, dashboard, 1 bot DCA/Candle/Smart |
| **PRO** | R$ 19,90/mês | 3 bots/estratégia | Bots ilimitados por tipo, IA agents, multi-exchange |
| **MAX** | R$ 97/mês | Ilimitado | Tudo do PRO + prioridade, backtesting (futuro), marketplace (futuro) |

**Comparação com Concorrentes**:
- **3Commas**: $29-99/mês (≈ R$ 145-495/mês)
- **Crypteras**: **7-8x mais barato** que 3Commas

### Escala e Métricas

**Métricas Atuais** (Pre-lançamento):
- Usuários Ativos: 0 (em fase de lançamento)
- MRR (Monthly Recurring Revenue): R$ 0
- Meta Inicial: 25 usuários pagantes/mês

**Tamanho do Mercado Brasileiro**:
- **Volume transacionado**: ~US$ 318 bilhões (jul/24 - jun/25)
- **Crescimento institucional**: Volume quase triplicou em 2025
- **Liderança regional**: Brasil lidera América Latina em volume cripto
- **Base global de usuários cripto**: 580+ milhões (2025)

---

## Camada 1: Arquitetura de Contexto do Cliente

### [Personas do Cliente](CUSTOMER_PERSONAS.md) - v1.0.0
Perfis detalhados dos usuários-alvo, incluindo traders iniciantes, intermediários e avançados.

### [Jornada do Cliente](CUSTOMER_JOURNEY.md) - v1.0.0
Mapeamento completo do ciclo de vida: descoberta → cadastro → configuração → uso → upgrade → advocacia.

### [Voz do Cliente](VOICE_OF_CUSTOMER.md) - v1.0.0
Feedback esperado, objeções comuns, comparações competitivas e padrões de comunicação.

---

## Camada 2: Arquitetura de Contexto do Produto

### [Estratégia do Produto](PRODUCT_STRATEGY.md) - v1.0.0
Visão, missão, posicionamento de mercado, prioridades estratégicas e roadmap.

### [Catálogo de Recursos](features/) - v1.0.0
Documentação detalhada de todas as funcionalidades implementadas e suas regras de negócio.

**Ver [Catálogo Completo de Features](features/README.md)** para documentação técnica detalhada.

#### **Funcionalidades Core Implementadas**

**Autenticação e Gestão de Usuários**:
- [Autenticação e Autorização Multi-User](features/authentication-and-authorization.md) - v1.0.0 - JWT, refresh tokens, criptografia de credenciais
- [Sistema de Assinaturas](features/subscription-plans.md) - v1.0.0 - Planos FREE/PRO/MAX, Stripe + Mercado Pago
- [Contextos de Trading](features/trading-contexts.md) - v1.0.0 - Organização por mercado, base do modelo freemium (1 vs 3 vs ilimitado)
- [Payment Webhooks](features/payment-webhooks.md) - v1.0.0 - Sincronização automática de pagamentos via webhooks
- Compliance LGPD - Consentimentos legais e auditoria

**Bots de Trading Automatizado**:
- [SmartBots](features/smart-bots.md) - v1.0.0 - 3 modos (CONTINUOUS, SCHEDULED, MANUAL), 8 mecanismos de risco
- [CandleBots](features/candle-bots.md) - v1.0.0 - Análise técnica com 10 indicadores, sinais automáticos
- [DCABots](features/dca-bots.md) - v1.0.0 - Dollar Cost Averaging, média de preço ponderada

**Gestão de Risco e Proteção**:
- [Circuit Breaker](features/circuit-breaker.md) - v1.0.0 - Proteção automática: pausa após perdas consecutivas ou limite diário
- Take Profit e Stop Loss (soft + hard)
- Trailing Stop (proteção de lucros)
- Adaptive Sell (ajuste por volatilidade)
- Sanity Check (validação de preços)
- Emergency Sell (venda de emergência)
- Auto-cancel de ordens distantes

**Experiência do Usuário**:
- [AGNO Chat](features/agno-chat-agent.md) - v1.0.0 - Agente de IA conversacional para onboarding, troubleshooting e educação
- [Dashboard e Analytics](features/dashboard-analytics.md) - v1.0.0 - Visão unificada de performance, P&L, métricas avançadas
- [Gestão de Credenciais](features/exchange-credentials.md) - v1.0.0 - Configuração segura de API keys com criptografia AES-256

**Infraestrutura e Operações**:
- [Backoffice / Admin Panel](features/backoffice-admin.md) - v1.0.0 - Painel administrativo para equipe interna, suporte e compliance
- Sistema de Ordens e Sincronização - Audit trail, strategy mapping, persistência local
- Suporte Multi-Exchange - Mercado Bitcoin + Binance
- Workflows Automatizados - 12 workflows Celery com intervalos coprimos

#### **Funcionalidades Futuras (Roadmap)**
- Backtesting completo - Teste de estratégias com dados históricos (Prioridade #1)
- Marketplace de Bots - Compra/venda de estratégias
- Arbitragem Automatizada - Detecção e execução multi-exchange
- Grid Trading Bot - Negociação em grade de preços
- 2FA e OAuth2 Social Login
- Trial de 7 dias e coupons de desconto


---

## Camada 3: Contexto de Mercado e Competitivo

### [Panorama Competitivo](COMPETITIVE_LANDSCAPE.md) - v1.0.0
Análise de concorrentes (3Commas, Cryptohopper, Bitsgap), forças/fraquezas e posicionamento.

---

## Camada 4: Contexto Empresarial Operacional

### [Framework de Mensagens](MESSAGING_FRAMEWORK.md) - v1.0.0
Voz da marca, mensagens centrais, proposições de valor por público.

### [Diretrizes de Comunicação com Cliente](CUSTOMER_COMMUNICATION.md) - v1.0.0
Princípios de comunicação, tom amigável/casual, SLA de suporte (24h), estratégias de engajamento.

---

## Como Usar Esta Documentação

### Para Equipes de Produto
- Consulte [PRODUCT_STRATEGY.md](PRODUCT_STRATEGY.md) para entender prioridades
- Use [features/](features/) para especificações detalhadas de recursos
- Consulte [CUSTOMER_PERSONAS.md](CUSTOMER_PERSONAS.md) para entender perfil do usuário

### Para Equipes de Marketing/Vendas
- Leia [MESSAGING_FRAMEWORK.md](MESSAGING_FRAMEWORK.md) para mensagens aprovadas
- Consulte [COMPETITIVE_LANDSCAPE.md](COMPETITIVE_LANDSCAPE.md) para objeções competitivas
- Use [CUSTOMER_PERSONAS.md](CUSTOMER_PERSONAS.md) para segmentação

### Para Equipes de Suporte
- Siga [CUSTOMER_COMMUNICATION.md](CUSTOMER_COMMUNICATION.md) para diretrizes de tom
- Consulte [VOICE_OF_CUSTOMER.md](VOICE_OF_CUSTOMER.md) para objeções comuns
- Use [CUSTOMER_JOURNEY.md](CUSTOMER_JOURNEY.md) para entender contexto do usuário

### Para Desenvolvimento de IA
- Esta documentação alimenta agentes de IA para:
  - Suporte ao cliente contextualmente apropriado
  - Recomendações de produto personalizadas
  - Mensagens de marketing dinâmicas
  - Detecção de churn e oportunidades de upsell

---

## Versionamento e Status da Documentação

| Documento | Status | Última Atualização |
|-----------|--------|-------------------|
| index.md | ✅ Atualizado | 2025-12-25 |
| README.md | ✅ Atualizado | 2025-12-25 |
| CUSTOMER_PERSONAS.md | ✅ Atualizado | 2025-12-25 |
| CUSTOMER_JOURNEY.md | ✅ Atualizado | 2025-12-25 |
| VOICE_OF_CUSTOMER.md | ✅ Atualizado | 2025-12-25 |
| PRODUCT_STRATEGY.md | ✅ Atualizado | 2025-12-25 |
| COMPETITIVE_LANDSCAPE.md | ✅ Atualizado | 2025-12-25 |
| MESSAGING_FRAMEWORK.md | ✅ Atualizado | 2025-12-25 |
| CUSTOMER_COMMUNICATION.md | ✅ Atualizado | 2025-12-25 |
| features/README.md | ✅ Atualizado | 2025-12-25 |
| features/agno-chat-agent.md | ✅ Atualizado | 2025-12-25 |
| features/authentication-and-authorization.md | ✅ Atualizado | 2025-12-25 |
| features/backoffice-admin.md | ✅ Atualizado | 2025-12-25 |
| features/candle-bots.md | ✅ Atualizado | 2025-12-25 |
| features/circuit-breaker.md | ✅ Atualizado | 2025-12-25 |
| features/dashboard-analytics.md | ✅ Atualizado | 2025-12-25 |
| features/dca-bots.md | ✅ Atualizado | 2025-12-25 |
| features/exchange-credentials.md | ✅ Atualizado | 2025-12-25 |
| features/payment-webhooks.md | ✅ Atualizado | 2025-12-25 |
| features/smart-bots.md | ✅ Atualizado | 2025-12-25 |
| features/subscription-plans.md | ✅ Atualizado | 2025-12-25 |
| features/trading-contexts.md | ✅ Atualizado | 2025-12-25 |

### Histórico de Versões

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0.0 | 2025-12-25 | Reconstrução dos índices de navegação - versão canônica |

---

## Contato e Feedback

Para atualizações nesta documentação, entre em contato com a equipe de Produto.

**Última atualização**: 2025-12-25
**Versão da Documentação**: 1.0.0
**Próxima revisão**: Trimestral ou quando houver mudanças significativas no produto/mercado
