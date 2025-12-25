---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "high"
category: "feature"
tags: ['feature', 'business', 'plans', 'subscription']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Sistema de Assinaturas (Subscription Plans)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Implementar modelo freemium SaaS com 3 planos (FREE, PRO, MAX) e conversão gradual de usuários.

**Constraints** (limites obrigatórios):
- Pricing DEVE ser em BRL (mercado brasileiro)
- FREE plan DEVE permitir testar core features (1 contexto, SmartBots)
- Limites por plano DEVEM ser validados no backend (nunca confiar apenas em frontend)
- Downgrade DEVE preservar dados do usuário (apenas desativar recursos extras)
- Upgrade DEVE ser instantâneo (sem necessidade de aprovação manual)
- Integração Stripe + Mercado Pago obrigatória (cartão internacional + PIX/boleto)
- Grace period de 7 dias após falha de pagamento antes de downgrade

**Non-Goals** (o que NÃO fazer):
- Criar plano Enterprise ou B2B no MVP (apenas B2C)
- Oferecer trial pago de 14 dias (FREE já é trial permanente)
- Implementar planos anuais com desconto no MVP (apenas mensais)
- Adicionar add-ons ou features à la carte
- Suportar múltiplas moedas (USD, EUR) no MVP
- Criar programa de afiliados ou referral
- Implementar billing complexo (pro-rata, créditos, reembolsos automáticos)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Planos de assinatura (FREE, PRO, MAX)
:::

## Visão Geral

Sistema completo de assinaturas freemium SaaS com 3 planos (FREE, PRO, MAX), integração com Stripe e Mercado Pago, e gestão automática de limites de recursos baseada no plano do usuário.

**Tickets Relacionados**: CRY-41, CRY-56
**Status**: ✅ Em Produção
**Prioridade**: CRÍTICA

---

## Propósito e Valor

### Para o Usuário
- **Entrada Gratuita**: Plano FREE permite experimentar a plataforma sem compromisso
- **Flexibilidade**: Upgrade/downgrade conforme necessidade
- **Transparência**: Limites claros e preços fixos
- **Múltiplas Formas de Pagamento**: Stripe (cartão internacional) + Mercado Pago (PIX, boleto)

### Para o Negócio
- **Modelo Freemium**: Conversão gradual FREE → PRO → MAX
- **Receita Recorrente (MRR)**: Pagamentos mensais automáticos
- **Escalabilidade**: Limites por plano controlam custos operacionais
- **Retenção**: Período de trial (FREE) reduz barreiras de entrada

---

## Planos Disponíveis

| Recurso | FREE | PRO | MAX |
|---------|------|-----|-----|
| **Preço** | R$ 0/mês | R$ 19,90/mês | R$ 97/mês |
| **Contextos** | 1 | 3 | Ilimitado |
| **Exchanges** | Mercado Bitcoin | MB + Binance | MB + Binance |
| **Workflows Ativos** | ✅ | ✅ | ✅ |
| **SmartBots** | ✅ | ✅ | ✅ |
| **Candle Bots** | ❌ | ✅ | ✅ |
| **DCA Bots** | ❌ | ✅ | ✅ |
| **Indicadores Padrão** | RSI, MA, BB | RSI, MA, BB, Volume, Volatility | Todos |
| **Indicadores Avançados** | ❌ | ❌ | VWAP, MFI, Volume Profile, ATR, OBV |
| **Dashboard** | ✅ | ✅ | ✅ Avançado |
| **Suporte** | Comunidade | Email (48h) | Prioritário (24h) |

### Definição de Contexto
**Contexto** = Combinação única de `moeda-fiat-exchange-user_id`

Exemplos:
- `BTC-BRL-MercadoBitcoin-65f8a1...` (1 contexto)
- `ETH-BRL-MercadoBitcoin-65f8a1...` (outro contexto)
- `BTC-USDT-Binance-65f8a1...` (outro contexto)

**Limites**:
- FREE: Máximo 1 contexto (ex: apenas BTC-BRL no Mercado Bitcoin)
- PRO: Máximo 3 contextos (ex: BTC, ETH, SOL)
- MAX: Ilimitado (quantos pares desejar)

---

## Funcionalidades Principais

### 1. Criação Automática de Subscription (FREE)

**Trigger**: Registro de novo usuário (`POST /api/auth/register`)

**Regras de Negócio**:
1. ✅ Todo usuário novo recebe plano FREE automaticamente
2. ✅ Status inicial: `ACTIVE`
3. ✅ Sem período de billing (`current_period_start` e `current_period_end` = `None`)
4. ✅ Sem customer_id ou subscription_id (não há cobrança)

**Entidade Criada**:
```python
SubscriptionEntity(
    user_id=user.user_id,
    plan=SubscriptionPlan.FREE,
    status=SubscriptionStatus.ACTIVE,
    created_at=now,
    updated_at=now,
    payment_provider=None,
    external_customer_id=None,
    external_subscription_id=None,
    current_period_start=None,
    current_period_end=None,
    cancel_at_period_end=False
)
```

---

### 2. Upgrade FREE → PRO (Stripe)

**Endpoint**: `POST /api/subscriptions/checkout`

**Payload**:
```json
{
  "plan": "pro"
}
```

**Resposta**:
```json
{
  "checkout_url": "https://checkout.stripe.com/c/pay/cs_test_..."
}
```

**Fluxo**:
```
1. Frontend: POST /api/subscriptions/checkout { plan: "pro" }
2. Backend:
   - Valida current_plan = FREE
   - Cria Stripe Checkout Session:
     - mode: "subscription"
     - price: STRIPE_PRICE_ID_PRO (.env)
     - success_url: FRONTEND_URL/subscription/success
     - cancel_url: FRONTEND_URL/subscription/cancel
     - metadata: { user_id: "65f8a1..." }
   - Retorna checkout_url

3. Frontend: Redireciona para checkout_url
4. Usuário: Preenche dados de pagamento no Stripe
5. Stripe: POST /api/webhooks/stripe
   - Event: checkout.session.completed
   - Payload: { customer: "cus_...", subscription: "sub_..." }

6. Backend (webhook handler):
   - Busca subscription via metadata.user_id
   - Atualiza:
     - plan = PRO
     - status = ACTIVE
     - payment_provider = "stripe"
     - external_customer_id = customer
     - external_subscription_id = subscription
     - current_period_start = now
     - current_period_end = now + 30 days
   - Persiste

7. Frontend (polling): Detecta subscription.plan = PRO
8. Exibe mensagem de sucesso + Redireciona para /dashboard
```

**Regras de Negócio**:
1. ✅ Apenas usuários FREE podem fazer upgrade para PRO via Stripe
2. ✅ Webhook Stripe DEVE validar signature (STRIPE_WEBHOOK_SECRET)
3. ✅ Subscription só é ativada após pagamento confirmado
4. ✅ Billing automático a cada 30 dias (gerenciado pelo Stripe)

---

### 3. Upgrade PRO → MAX (Mercado Pago)

**Endpoint**: `POST /api/subscriptions/checkout/mercadopago`

**Payload**:
```json
{
  "plan": "max"
}
```

**Resposta**:
```json
{
  "init_point": "https://www.mercadopago.com.br/subscriptions/checkout?pref_id=..."
}
```

**Fluxo**:
```
1. Frontend: POST /api/subscriptions/checkout/mercadopago { plan: "max" }
2. Backend:
   - Valida current_plan = PRO (apenas PRO pode ir para MAX)
   - Cria Preference no Mercado Pago:
     - auto_recurring:
       - frequency: 1
       - frequency_type: "months"
     - plan_id: MERCADOPAGO_PLAN_ID_MAX
     - back_urls: success, pending, failure
     - notification_url: /api/webhooks/mercadopago
   - Retorna init_point

3. Frontend: Redireciona para init_point
4. Usuário: Paga via PIX/Cartão/Boleto no Mercado Pago
5. Mercado Pago: POST /api/webhooks/mercadopago
   - type: "payment"
   - data.id: "123456"

6. Backend (webhook handler):
   - Busca payment via Mercado Pago API
   - Se status = "approved":
     - Busca subscription via external_customer_id
     - Atualiza:
       - plan = MAX
       - current_period_start = now
       - current_period_end = now + 30 days
     - Persiste

7. Frontend: Detecta subscription.plan = MAX
8. Exibe mensagem de sucesso
```

**Regras de Negócio**:
1. ✅ Apenas usuários PRO podem fazer upgrade para MAX
2. ✅ Plano MAX auto-criado no startup (caso não exista no Mercado Pago)
3. ✅ Webhook NÃO confia no payload (busca payment via API)
4. ✅ Suporta PIX, boleto e cartão (via Mercado Pago)

---

### 4. Upgrade FREE → PRO/MAX (Mercado Pago)

**Endpoint**: `POST /api/subscriptions/checkout/mercadopago`

**Payload**:
```json
{
  "plan": "pro"  // ou "max"
}
```

**Regras de Negócio**:
1. ✅ FREE pode ir direto para PRO ou MAX
2. ✅ Plan ID diferente:
   - PRO: `MERCADOPAGO_PLAN_ID_PRO` (.env - obrigatório)
   - MAX: `MERCADOPAGO_PLAN_ID_MAX` (.env - auto-criado se não existir)
3. ✅ Mesmo fluxo de webhook e validação

---

### 5. Cancelamento de Subscription

**Endpoint**: `POST /api/subscriptions/cancel`

**Payload**:
```json
{
  "cancel_immediately": false  // default: false
}
```

**Resposta**:
```json
{
  "message": "Assinatura cancelada. Você terá acesso até 15/02/2025.",
  "access_until": "2025-02-15T00:00:00Z"
}
```

**Regras de Negócio**:

#### Modo Padrão (`cancel_immediately: false`)
1. ✅ Define `cancel_at_period_end = True`
2. ✅ Mantém acesso até `current_period_end`
3. ✅ Stripe/Mercado Pago: Cancela renovação automática
4. ✅ Ao chegar em `current_period_end`:
   - Webhook: `subscription.deleted`
   - Backend: Downgrade para FREE
   - Limites de contexto reduzidos automaticamente

#### Modo Imediato (`cancel_immediately: true`)
1. ✅ Downgrade instantâneo para FREE
2. ✅ `cancel_at_period_end = False`
3. ✅ `current_period_end = now`
4. ✅ Bots excedentes pausados (ex: PRO com 3 contextos → FREE com 1)
5. ⚠️ Sem reembolso proporcional (política comercial)

**Webhook Stripe**: `subscription.deleted`
```python
# Backend processa webhook
subscription.plan = SubscriptionPlan.FREE
subscription.status = SubscriptionStatus.ACTIVE
subscription.current_period_start = None
subscription.current_period_end = None
subscription.external_subscription_id = None
```

---

### 6. Validação de Limites de Contextos

**Trigger**: Criação de SmartBot, CandleBot ou DCABot

**Lógica**:
```python
# CreateContextUseCase.execute()

# 1. Busca subscription do usuário
subscription = await subscription_repo.get_by_user_id(user_id)

# 2. Valida status ativo
if not subscription.is_active():
    raise HTTPException(403, "Subscription inativa")

# 3. Obtém limite baseado no plano
limit = subscription.get_context_limit()
# FREE → 1
# PRO → 3
# MAX → None (ilimitado)

# 4. Conta contextos existentes
current_count = await bot_repo.count_unique_contexts(user_id)

# 5. Valida se pode criar novo contexto
if not subscription.can_create_context(current_count):
    raise HTTPException(
        403,
        f"Limite de contextos atingido. Plano {subscription.plan.value} permite {limit} contexto(s)."
    )

# 6. Prossegue com criação do bot
```

**Regras de Negócio**:
1. ✅ Validação ANTES de criar bot
2. ✅ Erro 403 Forbidden se limite excedido
3. ✅ Conta TODOS os bots (SmartBots + CandleBots + DCABots) por contexto único
4. ✅ MAX: Limite `None` (ilimitado - bypass validação)

---

### 7. Consulta de Subscription Atual

**Endpoint**: `GET /api/subscriptions/current`

**Headers**: `Authorization: Bearer <access_token>`

**Resposta**:
```json
{
  "plan": "pro",
  "status": "active",
  "created_at": "2025-01-15T10:00:00Z",
  "current_period_start": "2025-01-15T10:00:00Z",
  "current_period_end": "2025-02-15T10:00:00Z",
  "cancel_at_period_end": false,
  "context_limit": 3,
  "contexts_used": 2,
  "contexts_available": 1
}
```

**Regras de Negócio**:
1. ✅ Retorna subscription do `current_user`
2. ✅ Calcula contextos utilizados dinamicamente
3. ✅ Indica se subscription está próxima da renovação

---

## Integrações de Pagamento

### Stripe

**Configuração (.env)**:
```env
STRIPE_API_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID_PRO=price_...
```

**Webhooks Processados**:
1. `checkout.session.completed`: Ativa subscription após pagamento
2. `subscription.deleted`: Downgrade para FREE após cancelamento

**Validação de Webhook**:
```python
import stripe

# Valida signature
event = stripe.Webhook.construct_event(
    payload=request.body,
    sig_header=request.headers['Stripe-Signature'],
    secret=STRIPE_WEBHOOK_SECRET
)

# Processa apenas se signature válida
```

**Segurança**:
- ✅ Signature validation obrigatória (previne replay attacks)
- ✅ Secret key NUNCA exposta no frontend
- ✅ Webhook URL deve ser HTTPS (produção)

---

### Mercado Pago

**Configuração (.env)**:
```env
MERCADOPAGO_ACCESS_TOKEN=APP_USR-...
MERCADOPAGO_PLAN_ID_PRO=...
MERCADOPAGO_PLAN_ID_MAX=...
```

**Webhooks Processados**:
1. `payment.created`: Valida pagamento e ativa subscription
2. `subscription.updated`: Atualiza status da subscription

**Validação de Webhook**:
```python
# NÃO confia no payload direto
payment_id = request.json()['data']['id']

# Busca payment via API
payment = mp.payment().get(payment_id)

# Valida status no servidor MP
if payment['status'] == 'approved':
    # Processa ativação
```

**Auto-criação de Plano MAX**:
```python
# Startup da aplicação
if not plan_exists(MERCADOPAGO_PLAN_ID_MAX):
    plan = mp.plan().create({
        "auto_recurring": {
            "frequency": 1,
            "frequency_type": "months",
            "transaction_amount": 97.00,
            "currency_id": "BRL"
        },
        "back_url": FRONTEND_URL,
        "reason": "Crypteras MAX Plan"
    })

    # Atualiza .env com plan_id criado
```

**Segurança**:
- ✅ Webhook NÃO confia no payload (sempre consulta API)
- ✅ Access token SERVER-SIDE apenas
- ✅ Previne fraudes (validação dupla)

---

## Modelo de Dados

### SubscriptionEntity

```python
@dataclass
class SubscriptionEntity:
    user_id: ObjectId
    plan: SubscriptionPlan  # FREE, PRO, MAX
    status: SubscriptionStatus  # ACTIVE, CANCELED, PAST_DUE
    created_at: datetime
    updated_at: datetime

    # Payment Provider (genérico - suporta múltiplos)
    payment_provider: Optional[str]  # 'stripe', 'mercadopago'
    external_customer_id: Optional[str]  # cus_xxx (Stripe) ou customer_id (MP)
    external_subscription_id: Optional[str]  # sub_xxx (Stripe) ou subscription_id (MP)

    # Billing Period (apenas PRO/MAX)
    current_period_start: Optional[datetime]
    current_period_end: Optional[datetime]

    # Cancelamento
    cancel_at_period_end: bool = False

    def is_active(self) -> bool:
        return self.status == SubscriptionStatus.ACTIVE

    def is_pro(self) -> bool:
        return self.plan == SubscriptionPlan.PRO

    def is_max(self) -> bool:
        return self.plan == SubscriptionPlan.MAX

    def has_access_to_trading(self) -> bool:
        return self.is_active() and self.plan in [SubscriptionPlan.PRO, SubscriptionPlan.MAX]

    def is_expired(self) -> bool:
        if self.current_period_end is None:
            return False
        return datetime.utcnow() > self.current_period_end

    def get_context_limit(self) -> Optional[int]:
        """Retorna limite de contextos por plano"""
        if self.plan == SubscriptionPlan.FREE:
            return 1
        elif self.plan == SubscriptionPlan.PRO:
            return 3
        elif self.plan == SubscriptionPlan.MAX:
            return None  # Ilimitado

    def can_create_context(self, current_count: int) -> bool:
        """Valida se pode criar novo contexto"""
        limit = self.get_context_limit()
        if limit is None:  # MAX = ilimitado
            return True
        return current_count < limit
```

### SubscriptionPlan (Enum)

```python
class SubscriptionPlan(Enum):
    FREE = "free"
    PRO = "pro"
    MAX = "max"
```

### SubscriptionStatus (Enum)

```python
class SubscriptionStatus(Enum):
    ACTIVE = "active"
    CANCELED = "canceled"
    PAST_DUE = "past_due"  # Pagamento atrasado (futuro)
```

---

## Métricas de Sucesso

### KPIs de Produto
- **Taxa de Conversão FREE → PRO**: Meta > 10%
- **Taxa de Conversão PRO → MAX**: Meta > 5%
- **Churn Rate**: Meta < 5%/mês
- **MRR (Monthly Recurring Revenue)**: Crescimento mensal
- **ARPU (Average Revenue Per User)**: Valor médio por usuário

### Monitoramento
- Subscriptions ativas por plano (diário)
- Upgrades realizados (diário)
- Downgrades/Cancelamentos (diário)
- Revenue por gateway (Stripe vs Mercado Pago)
- Falhas de pagamento (alerta)

---

## Problemas Comuns e Limitações

### ❌ Webhook Não Chegou
**Sintoma**: Usuário pagou mas subscription não atualizou
**Causa**: Webhook falhou ou atrasou (network issues)
**Solução**:
1. Verificar logs do gateway (Stripe/MP)
2. Reenviar webhook manualmente
3. Implementar polling de fallback (futuro)

### ❌ Limite de Contextos Atingido
**Sintoma**: Erro 403 ao criar novo bot
**Causa**: FREE tentando criar 2º contexto
**Solução**: Fazer upgrade para PRO ou MAX

### ❌ Pagamento Recusado
**Sintoma**: Subscription não ativada após checkout
**Causa**: Cartão recusado, saldo insuficiente
**Solução**: Usuário deve tentar outro método de pagamento

### ⚠️ Limitações
- Sem suporte a coupons/desconto (futuro)
- Sem período de trial (7 dias) - FREE é o trial
- Sem reembolso proporcional em cancelamento imediato
- MAX disponível apenas via Mercado Pago (Stripe em desenvolvimento)

---

## Boas Práticas para Desenvolvedores

### Frontend - Polling de Subscription

```javascript
// Após redirect de pagamento bem-sucedido
const checkSubscription = async () => {
  const { data } = await axios.get('/api/subscriptions/current');

  if (data.plan === 'pro') {
    // Atualizado!
    showSuccess('Assinatura PRO ativada com sucesso!');
    router.push('/dashboard');
  } else {
    // Ainda processando webhook - tenta novamente em 2s
    setTimeout(checkSubscription, 2000);
  }
};

checkSubscription();
```

### Backend - Dependency Injection

```python
from api.dependencies.auth import get_current_subscription

@router.post("/smart-bots")
async def create_smart_bot(
    request: CreateSmartBotRequest,
    current_user: UserEntity = Depends(get_current_user),
    subscription: SubscriptionEntity = Depends(get_current_subscription)
):
    # subscription já validada e carregada

    # Valida limite de contextos
    current_count = await bot_repo.count_unique_contexts(current_user.user_id)

    if not subscription.can_create_context(current_count):
        raise HTTPException(
            403,
            f"Limite atingido. Plano {subscription.plan.value} permite {subscription.get_context_limit()} contexto(s)."
        )

    # Prossegue com criação
```

---

## Roadmap Futuro

### Melhorias Planejadas
- [ ] **Trial de 7 Dias**: PRO grátis por 7 dias (sem cartão)
- [ ] **Coupons**: Códigos de desconto (ex: LAUNCH50 = 50% off)
- [ ] **Plano Anual**: 12 meses com desconto (2 meses grátis)
- [ ] **Plano Enterprise**: Custom pricing, white-label
- [ ] **Reembolso Proporcional**: Crédito ao fazer downgrade
- [ ] **Portal de Billing**: Gerenciar subscription (Stripe Customer Portal)
- [ ] **Alertas de Renovação**: Email 7 dias antes da cobrança
- [ ] **Fallback Polling**: Caso webhook falhe, consulta API periodicamente

### Considerações de Escalabilidade
- [ ] **Caching de Limites**: Redis cache para `get_context_limit()`
- [ ] **Webhook Retry**: Retry automático com backoff exponencial
- [ ] **Idempotência**: Prevenir processamento duplicado de webhooks
- [ ] **Circuit Breaker**: Pausar integrações se gateway falhar

---

## Referências Técnicas

### Arquivos Relacionados
- `backend/src/domain/entities/subscription.py` - SubscriptionEntity
- `backend/src/application/use_cases/create_subscription.py` - Criação
- `backend/src/api/routes/subscriptions.py` - Endpoints
- `backend/src/api/routes/webhooks.py` - Stripe + Mercado Pago webhooks
- `backend/src/infrastructure/payment/stripe_gateway.py` - Integração Stripe
- `backend/src/infrastructure/payment/mercadopago_gateway.py` - Integração MP

### Standards e Compliance
- **PCI DSS**: Payment Card Industry Data Security Standard (Stripe compliance)
- **SCA**: Strong Customer Authentication (3D Secure via Stripe)
- **Stripe API**: v2023-10-16
- **Mercado Pago API**: v1

### Recursos Externos
- [Stripe Subscriptions](https://stripe.com/docs/billing/subscriptions)
- [Mercado Pago Recurring Payments](https://www.mercadopago.com.br/developers/pt/docs/subscriptions)
- [Webhook Security Best Practices](https://webhooks.fyi/)
