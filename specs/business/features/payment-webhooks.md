---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "high"
category: "feature"
tags: ['feature', 'business', 'payment', 'webhooks']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Payment Webhooks (Stripe + Mercado Pago)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Processar webhooks de Stripe e Mercado Pago para atualizar status de subscription automaticamente.

**Constraints** (limites obrigatórios):
- Validar assinatura HMAC do webhook antes de processar
- Implementar idempotência (mesma webhook processada múltiplas vezes não corrompe dados)
- Retry com exponential backoff se processamento falhar
- Registrar TODOS webhooks recebidos (audit trail)
- Atualizar subscription status instantaneamente (upgrade/downgrade/cancel)
- Notificar usuário por email sobre mudanças de subscription

**Non-Goals** (o que NÃO fazer):
- Processar webhooks de outras plataformas de pagamento no MVP
- Implementar reconciliação manual de pagamentos
- Criar sistema de disputes ou chargebacks complexo
- Adicionar split payments ou marketplace functionality
- Implementar delayed capture ou pre-auth
- Criar fluxo de refund automático
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Sistema de webhooks de pagamento
:::

## Visão Geral

Sistema de processamento assíncrono de eventos de pagamento que sincroniza automaticamente o status de assinaturas entre os provedores de pagamento (Stripe e Mercado Pago) e a plataforma Crypteras.

**Status**: ✅ Em Produção
**Tickets**: CRY-41, CRY-56
**Integrações**: Stripe (internacional) + Mercado Pago (Brasil)

---

## Propósito e Valor

### Para o Negócio
- **Automação Completa**: Assinaturas são ativadas/canceladas automaticamente sem intervenção manual
- **Receita Recorrente**: Cobranças mensais automáticas (PRO: R$ 19,90, MAX: R$ 97)
- **Redução de Churn**: Retry automático de pagamentos falhos (Stripe)
- **Compliance**: Auditoria completa de transações e mudanças de status

### Para o Usuário
- **Ativação Instantânea**: Acesso PRO/MAX liberado segundos após pagamento
- **Transparência**: Status da assinatura sempre atualizado (ativa, expirada, cancelada)
- **Grace Period**: Mantém acesso até fim do período ao cancelar (Stripe)
- **Múltiplas Formas de Pagamento**: Cartão internacional (Stripe) ou PIX/boleto (Mercado Pago)

### Para Operações
- **Zero Manutenção**: Sincronização automática reduz tickets de suporte
- **Resiliência**: Retry automático de webhooks garante consistência
- **Rastreabilidade**: Logs completos para troubleshooting

---

## Funcionalidades Principais

### 1. Webhooks Stripe

#### Evento: `checkout.session.completed`
**Quando Acontece**: Usuário finaliza pagamento no checkout Stripe

**O Que Faz**:
```
Usuário clica "Assinar PRO" → Redirecionado para checkout Stripe
  ↓
Usuário preenche cartão e confirma pagamento
  ↓
Stripe cria assinatura recorrente (cobrança mensal automática)
  ↓
Stripe envia webhook "checkout.session.completed"
  ↓
Sistema recebe webhook, valida assinatura HMAC-SHA256
  ↓
Localiza assinatura do usuário no banco de dados
  ↓
Atualiza:
  ✅ Plano: FREE → PRO
  ✅ Status: ACTIVE
  ✅ ID da assinatura Stripe (para futuros webhooks)
  ✅ Período: data_inicio e data_fim (ex: 24/01 a 24/02)
  ↓
Usuário pode usar recursos PRO imediatamente
```

**Regras de Negócio**:
- ✅ Validação de assinatura HMAC-SHA256 obrigatória (previne fraude)
- ✅ Timestamp do webhook deve ser recente (previne replay attacks)
- ✅ Busca assinatura por `user_id` presente nos metadados da sessão
- ✅ Se usuário não encontrado → HTTP 400 (Stripe retenta automaticamente)
- ✅ Idempotente: Webhook duplicado atualiza mesmos dados (sem efeito colateral)

**Exemplo Real**:
```
Maria assina o plano PRO às 14h30
  → Stripe cobra R$ 19,90 no cartão
  → Webhook recebido às 14h30:02 (2 segundos depois)
  → Sistema ativa assinatura PRO
  → Maria recarrega página: "✅ Assinatura PRO Ativa até 24/02/2025"
  → Maria pode criar até 3 contextos de trading
```

---

#### Evento: `customer.subscription.updated`
**Quando Acontece**: Status da assinatura muda no Stripe (pagamento coletado, falha, cancelamento agendado)

**O Que Faz**:
- Sincroniza status atual da assinatura Stripe → banco de dados local
- Atualiza datas de período (início/fim)
- Atualiza flag `cancel_at_period_end` (se usuário agendou cancelamento)

**Mapeamento de Status**:

| Status Stripe | Status Interno | Significado | Acesso Trading |
|---------------|----------------|-------------|----------------|
| `active` | ACTIVE | Pagamento em dia | ✅ SIM |
| `past_due` | PAST_DUE | Falha no pagamento, retry em andamento | ❌ NÃO |
| `unpaid` | UNPAID | Falha permanente (sem retry) | ❌ NÃO |
| `canceled` | CANCELED | Assinatura cancelada | ❌ NÃO |
| `incomplete` | INCOMPLETE | Checkout iniciado mas não finalizado | ❌ NÃO |

**Cenário: Pagamento Falha (Cartão Expirado)**:
```
Dia 24/02: Stripe tenta cobrar R$ 19,90 (renovação mensal)
  ↓
Pagamento falha (cartão expirado)
  ↓
Stripe envia webhook: customer.subscription.updated (status=past_due)
  ↓
Sistema atualiza status → PAST_DUE
  ↓
Pedro tenta criar bot: "❌ Assinatura inativa. Atualize forma de pagamento."
  ↓ (Stripe retenta pagamento automaticamente em 3, 5, 7 dias)
  ↓
Pedro atualiza cartão no dia 25/02
  ↓
Stripe cobra R$ 19,90 com sucesso
  ↓
Stripe envia webhook: customer.subscription.updated (status=active)
  ↓
Sistema atualiza status → ACTIVE
  ↓
Pedro pode usar bots novamente
```

**Regras de Negócio**:
- ✅ Webhook correlacionado por `stripe_subscription_id` (ID único da assinatura)
- ✅ Se assinatura não encontrada no banco → retorna "ignored" (não falha)
- ✅ Atualiza período sempre que Stripe enviar novos valores
- ✅ Idempotente: Mesmo webhook múltiplas vezes = mesmo resultado

---

#### Evento: `customer.subscription.deleted`
**Quando Acontece**: Assinatura é completamente cancelada no Stripe

**O Que Faz**:
```
Assinatura cancelada no Stripe (fim do período de cancelamento)
  ↓
Stripe envia webhook: customer.subscription.deleted
  ↓
Sistema processa:
  ✅ Plano: PRO/MAX → FREE
  ✅ Status: ACTIVE (FREE é sempre ativo)
  ✅ Remove ID da assinatura Stripe (limpa vínculo)
  ✅ Limpa datas de período
  ✅ Limpa flag de cancelamento
  ↓
Usuário volta para plano FREE (acesso limitado)
```

**Exemplo Real**:
```
Carlos cancela assinatura PRO no dia 15/01 (período termina 24/01)
  → Stripe marca: cancel_at_period_end=True
  → Carlos mantém acesso PRO até 24/01 (grace period)

Dia 24/01 23:59:59:
  → Stripe finaliza cancelamento
  → Stripe envia webhook: customer.subscription.deleted

Dia 25/01 00:00:10:
  → Sistema processa webhook
  → Carlos downgrade para FREE
  → Carlos perde acesso a bots extras (só 1 contexto)
  → Carlos recebe email: "Sua assinatura PRO expirou. Reative quando quiser!"
```

**Regras de Negócio**:
- ✅ Downgrade imediato para FREE (preserva conta do usuário)
- ✅ Usuário pode se inscrever novamente a qualquer momento (novo checkout)
- ✅ Idempotente: FREE → FREE não causa efeitos colaterais

---

### 2. Webhooks Mercado Pago (IPN)

#### Evento: `payment.approved`
**Quando Acontece**: Pagamento aprovado no checkout Mercado Pago

**O Que Faz**:
```
Usuário clica "Assinar PRO com Mercado Pago"
  ↓
Redirecionado para checkout MP (PIX, boleto ou cartão)
  ↓
Usuário paga R$ 19,90
  ↓
Mercado Pago aprova pagamento
  ↓
MP envia webhook IPN: payment.approved
  ↓
Sistema busca detalhes do pagamento na API do MP
  ↓
Extrai user_id do campo external_reference
  ↓
Localiza assinatura do usuário
  ↓
Verifica se plano != PRO (proteção contra duplicação)
  ↓
Atualiza:
  ✅ Plano: FREE → PRO
  ✅ Status: ACTIVE
  ✅ ID do pagamento MP (armazenado para auditoria)
  ↓
Usuário ganha acesso PRO
```

**Diferença vs Stripe**:
- **Stripe**: Cria assinatura recorrente (cobra automaticamente todo mês)
- **Mercado Pago**: Pagamento único (usuário deve renovar manualmente) OU assinatura recorrente (preapproval)

**Regras de Negócio**:
- ✅ Busca pagamento na API do MP (segurança: valida que pagamento é real)
- ✅ Verifica `external_reference` (user_id) para vincular ao usuário correto
- ✅ Proteção de idempotência: Se plano == PRO, ignora webhook duplicado
- ✅ Se usuário não encontrado → HTTP 400

**Exemplo Real**:
```
Ana escolhe "Pagar com PIX"
  → Checkout MP gera QR Code
  → Ana paga R$ 19,90 via app do banco
  → MP aprova pagamento em 5 segundos
  → Webhook enviado e processado
  → Ana recarrega página: "✅ Assinatura PRO Ativa"
```

---

#### Evento: `subscription.authorized` (Assinatura Recorrente)
**Quando Acontece**: Assinatura recorrente (preapproval) é autorizada no MP

**O Que Faz**:
- Ativa/renova assinatura PRO
- Similar ao checkout.session.completed do Stripe

**Mapeamento de Status**:

| Status MP | Status Interno | Acesso Trading |
|-----------|----------------|----------------|
| `authorized` | ACTIVE | ✅ SIM |
| `paused` | ACTIVE | ✅ SIM (mantém acesso, pausa renovação) |
| `cancelled` | CANCELED | ❌ NÃO |

**Regras de Negócio**:
- ✅ Busca detalhes do preapproval na API MP
- ✅ Correlaciona por `payer_id` (ID do pagador MP)
- ✅ Atualiza status conforme mapeamento
- ✅ Se assinatura não encontrada → retorna "ignored"

---

#### Evento: `subscription.cancelled`
**Quando Acontece**: Assinatura recorrente é cancelada no MP

**O Que Faz**:
```
Usuário cancela assinatura no dashboard do Mercado Pago
  ↓
MP envia webhook: subscription.cancelled
  ↓
Sistema processa:
  ✅ Status: CANCELED
  ✅ Plano: FREE
  ↓
Downgrade imediato (SEM grace period como Stripe)
```

**Diferença vs Stripe**:
- **Stripe**: Cancelamento com grace period (at_period_end=True)
- **Mercado Pago**: Cancelamento imediato (perde acesso na hora)

---

### 3. Segurança de Webhooks

#### Stripe: Validação de Assinatura HMAC-SHA256

**Como Funciona**:
```
Stripe envia header: Stripe-Signature
  Formato: "t=1674567890,v1=abcdef123456..."

Sistema extrai:
  - t (timestamp): quando webhook foi enviado
  - v1 (signature): HMAC-SHA256 do payload

Sistema valida:
  1. Recalcula HMAC-SHA256(payload + timestamp, webhook_secret)
  2. Compara com assinatura recebida
  3. Verifica se timestamp é recente (< 5 minutos)

Se válido: Processa webhook
Se inválido: HTTP 400 "Invalid webhook signature"
```

**Proteções**:
- ✅ **Anti-Fraude**: Apenas webhooks assinados pelo Stripe são processados
- ✅ **Anti-Replay**: Timestamp impede reutilização de webhooks antigos
- ✅ **Integridade**: Payload não pode ser modificado (qualquer alteração quebra assinatura)

**Cenário de Ataque (Bloqueado)**:
```
Atacante intercepta webhook legítimo:
  POST /api/webhooks/stripe
  Body: {"type": "checkout.session.completed", ...}
  Stripe-Signature: "t=1674567890,v1=abc123..."

Atacante modifica payload (tenta ativar assinatura de outro usuário):
  Body: {"type": "checkout.session.completed", "user_id": "attacker_id"}
  (Mantém mesma Stripe-Signature)

Sistema valida:
  HMAC-SHA256(novo_payload, secret) != "abc123..."
  ❌ Assinatura inválida

Sistema rejeita: HTTP 400
  → Webhook ignorado
  → Atacante não ganha acesso PRO
```

---

#### Mercado Pago: Validação Indireta via API

**Como Funciona**:
```
MP envia webhook IPN:
  Body: {"type": "payment", "data": {"id": "pay_123"}}
  (SEM assinatura HMAC)

Sistema processa:
  1. Extrai payment_id = "pay_123"
  2. Faz request autenticado para API MP:
     GET https://api.mercadopago.com/v1/payments/pay_123
     Header: Authorization: Bearer <access_token>
  3. API retorna detalhes do pagamento
  4. Se pagamento não existe → API retorna 404
  5. Se existe → processa normalmente
```

**Proteções**:
- ✅ **Validação Indireta**: Recurso deve existir na API MP (não pode ser fabricado)
- ✅ **Autenticação OAuth**: Access token impede acesso não autorizado
- ❌ **Sem Proteção de Replay**: Webhook antigo pode ser reenviado (mitigado por idempotência)

---

### 4. Idempotência (Proteção contra Webhooks Duplicados)

**Problema**: Provedores de pagamento enviam webhooks múltiplas vezes (retry automático em caso de falha)

**Solução**: Todos os webhooks são idempotentes = processar 2x o mesmo webhook = mesmo resultado final

#### Exemplo: Webhook `checkout.session.completed` Duplicado

```
Webhook #1 (15:30:00):
  Localiza assinatura do usuário (user_id=abc123)
  Atualiza: plan=FREE → plan=PRO, status=ACTIVE, stripe_subscription_id=sub_xyz
  Estado final: PRO, ACTIVE

Webhook #2 (15:30:05) - DUPLICADO (retry do Stripe):
  Localiza assinatura do usuário (user_id=abc123)
  Atualiza: plan=PRO → plan=PRO, status=ACTIVE, stripe_subscription_id=sub_xyz
  Estado final: PRO, ACTIVE (IDÊNTICO)

Efeito: NENHUM efeito colateral
  ✅ Não duplica cobrança
  ✅ Não cria segunda assinatura
  ✅ Apenas sobrescreve mesmos valores
```

#### Proteções Específicas por Evento

**`checkout.session.completed`**:
- Busca por `user_id` (sempre encontra mesma assinatura)
- Atualiza com mesmos valores do Stripe

**`customer.subscription.updated`**:
- Busca por `stripe_subscription_id` (sempre encontra mesma assinatura)
- Atualiza status e período (valores idênticos em replay)

**`payment.approved` (MP)**:
- Verifica se `plan != PRO` ANTES de atualizar
- Se já PRO → ignora webhook (proteção explícita)

---

### 5. Tratamento de Erros

#### Erros que Retornam HTTP 400 (Webhook Rejeitado)

**Stripe**:
- ❌ Assinatura HMAC inválida → "Invalid webhook signature"
- ❌ Timestamp muito antigo (replay attack) → "Webhook too old"
- ❌ Session sem `subscription_id` → "Missing subscription_id"
- ❌ Session sem `user_id` nos metadados → "Missing user_id"
- ❌ Usuário não encontrado no banco → "User not found"

**Mercado Pago**:
- ❌ Falha ao buscar recurso na API MP → "Failed to fetch payment/preapproval"
- ❌ Pagamento sem `external_reference` → "Missing user_id"
- ❌ Usuário não encontrado no banco → "User not found"

**Consequência de HTTP 400**:
- Provedor de pagamento **retenta webhook automaticamente**
- Retry com backoff exponencial (1min, 5min, 30min, 1h, ...)
- Até 5 dias de tentativas (Stripe)

---

#### Erros que Retornam HTTP 200 (Webhook Reconhecido, mas Ignorado)

**Cenários**:
- ✅ Assinatura não encontrada no banco → `{"status": "ignored", "reason": "subscription_not_found"}`
- ✅ Tipo de evento não suportado → `{"status": "ignored", "event_type": "invoice.paid"}`

**Por Que HTTP 200?**:
- Provedor de pagamento **para de retentar**
- Evita retry infinito de eventos não suportados
- Graceful degradation (sistema continua funcionando)

**Exemplo**:
```
Stripe envia evento novo: "invoice.payment_failed"
  (Sistema ainda não tem handler para esse evento)

Sistema processa:
  Tipo de evento não mapeado
  Log: "Unhandled Stripe event: invoice.payment_failed"
  Retorna: HTTP 200 {"status": "ignored"}

Stripe: Webhook processado com sucesso (não retenta)
Sistema: Sem quebra, continua funcionando
```

---

#### Logs e Monitoramento

**Logs de Sucesso** (INFO):
- `stripe_webhook_received` - Webhook recebido e validado
- `checkout_completed_processed` - Usuário ativado com sucesso
- `subscription_updated_processed` - Status sincronizado
- `mercadopago_webhook_received` - Webhook MP processado

**Logs de Atenção** (WARNING):
- `stripe_webhook_unhandled_event` - Evento não suportado (ignorado)
- `stripe_subscription_not_found_in_db` - Assinatura não existe (pode ser race condition)
- `mercadopago_webhook_unhandled_event` - Evento MP não suportado

**Logs de Erro** (ERROR):
- `stripe_webhook_signature_invalid` - Possível ataque ou misconfiguration
- `stripe_webhook_error` - Falha crítica no processamento
- `mercadopago_webhook_error` - Falha crítica no processamento

---

## Casos de Uso Reais

### Caso 1: Usuário FREE Assina PRO via Stripe

**Persona**: João - Trader Iniciante (plano FREE, quer criar mais bots)

**Jornada**:
```
14h00: João tenta criar segundo contexto
  Sistema: "❌ Limite atingido (1/1). Faça upgrade para PRO."

14h01: João clica "Upgrade para PRO"
  Sistema redireciona para Stripe Checkout
  URL: https://checkout.stripe.com/c/pay/cs_live_abc123

14h02: João preenche dados do cartão Visa
  Valor: R$ 19,90/mês
  João confirma pagamento

14h02:10: Stripe aprova pagamento
  Stripe cria subscription_id: sub_1A2B3C
  Stripe envia webhook: checkout.session.completed

14h02:12: Sistema recebe webhook
  ✓ Valida assinatura HMAC-SHA256
  ✓ Localiza assinatura de João (user_id nos metadados)
  ✓ Atualiza: plan=PRO, status=ACTIVE
  ✓ Armazena: stripe_subscription_id=sub_1A2B3C
  ✓ Define período: 24/01/2025 a 24/02/2025

14h02:15: João recarrega página do dashboard
  Sistema: "✅ Assinatura PRO Ativa até 24/02/2025"

14h03: João cria segundo contexto (BTC-BRL na Binance)
  Sistema: "✓ Contexto criado (2/3)"

14h05: João cria terceiro contexto (ETH-BRL)
  Sistema: "✓ Contexto criado (3/3)"

24/02/2025: Stripe cobra R$ 19,90 automaticamente
  Webhook: customer.subscription.updated (renovação)
  Novo período: 24/02 a 24/03

João nunca precisa renovar manualmente!
```

---

### Caso 2: Pagamento Falha, Stripe Retenta Automaticamente

**Persona**: Maria - Trader Intermediária (PRO há 3 meses, cartão venceu)

**Jornada**:
```
24/04 00:00: Stripe tenta cobrar R$ 19,90 (renovação automática)
  Cartão de Maria expirou em 03/2025
  Pagamento falha: "Card expired"

24/04 00:01: Stripe envia webhook: customer.subscription.updated
  Status: past_due (pagamento atrasado, retry em andamento)

24/04 00:02: Sistema processa webhook
  ✓ Atualiza status → PAST_DUE
  Maria recebe email: "⚠️ Falha no pagamento. Atualize seu cartão."

24/04 08:00: Maria tenta criar bot
  Sistema: "❌ Assinatura inativa. Atualize forma de pagamento."

24/04 09:00: Maria acessa Stripe Customer Portal
  Atualiza cartão (novo cartão válido até 05/2027)

27/04 00:00: Stripe retenta pagamento (retry automático)
  Pagamento aprovado: R$ 19,90 cobrado
  Webhook: customer.subscription.updated (status=active)

27/04 00:02: Sistema processa webhook
  ✓ Atualiza status → ACTIVE
  Maria recebe email: "✅ Pagamento processado. Assinatura reativada!"

27/04 10:00: Maria volta a usar bots normalmente
  Sistema: Acesso PRO restaurado
```

**Stripe Retry Schedule**:
- Tentativa 1: Imediatamente (falha)
- Tentativa 2: +3 dias (27/04 - sucesso)
- (Se falhasse: +5 dias, +7 dias, +9 dias... até 23 dias)

---

### Caso 3: Usuário Cancela Assinatura, Mantém Acesso até Fim do Período

**Persona**: Carlos - Trader Avançado (PRO, quer pausar trading por 3 meses)

**Jornada**:
```
15/01: Carlos decide pausar assinaturas
  Período atual: 24/12/2024 a 24/01/2025
  Acesso PRO: ✅ Ativo (9 dias restantes)

15/01 10:00: Carlos clica "Cancelar Assinatura"
  Sistema: "Tem certeza? Você perderá acesso em 24/01."
  Carlos confirma: "Cancelar ao fim do período"

15/01 10:01: Sistema chama API Stripe
  Stripe.Subscription.modify(
    subscription_id: sub_xyz,
    cancel_at_period_end: True
  )

15/01 10:02: Stripe envia webhook: customer.subscription.updated
  cancel_at_period_end: True
  status: active (ainda ativo)

15/01 10:03: Sistema processa webhook
  ✓ Atualiza flag: cancel_at_period_end=True
  ✓ Status permanece ACTIVE
  Carlos vê: "⚠️ Assinatura cancelada. Acesso até 24/01/2025."

16/01 - 23/01: Carlos continua usando bots normalmente
  Sistema: Acesso PRO mantido (grace period)

24/01 23:59:59: Período termina
  Stripe: Não cobra nova cobrança (cancelamento confirmado)

25/01 00:00:00: Stripe envia webhook: customer.subscription.deleted
  Webhook: Assinatura totalmente removida

25/01 00:00:05: Sistema processa webhook
  ✓ Downgrade: plan=FREE
  ✓ Remove stripe_subscription_id
  ✓ Limpa período e flags
  Carlos recebe email: "Sua assinatura PRO expirou. Volte quando quiser!"

25/01 08:00: Carlos tenta criar segundo contexto
  Sistema: "❌ Limite atingido (1/1). Plano FREE."
```

**Vantagem do Grace Period**:
- Usuário não perde acesso imediatamente
- Pode mudar de ideia e reativar antes do fim
- Experiência mais justa (pagou pelo mês completo)

---

### Caso 4: Pagamento com PIX via Mercado Pago

**Persona**: Ana - Trader Iniciante (não tem cartão internacional)

**Jornada**:
```
10/02 14:00: Ana clica "Upgrade para PRO"
  Sistema oferece 2 opções:
  [ ] Stripe (cartão internacional)
  [✓] Mercado Pago (PIX, boleto, cartão nacional)

10/02 14:01: Ana escolhe Mercado Pago
  Sistema cria preferência de pagamento:
    Valor: R$ 19,90
    external_reference: user_id (abc123)
    Redirect: checkout.mercadopago.com.br/preferences/pref_xyz

10/02 14:02: Ana redirecionada para checkout MP
  Ana escolhe: "Pagar com PIX"
  MP gera QR Code

10/02 14:03: Ana escaneia QR Code no app do banco
  Confirma pagamento: R$ 19,90
  Transação aprovada em 3 segundos

10/02 14:03:05: MP aprova pagamento
  payment_id: pay_789
  status: approved

10/02 14:03:06: MP envia webhook IPN
  POST /api/webhooks/mercadopago
  Body: {"type": "payment", "data": {"id": "pay_789"}}

10/02 14:03:07: Sistema recebe webhook
  ✓ Busca detalhes na API MP: GET /v1/payments/pay_789
  ✓ Extrai external_reference: abc123 (user_id de Ana)
  ✓ Localiza assinatura de Ana
  ✓ Verifica: plan != PRO (proteção idempotência)
  ✓ Atualiza: plan=PRO, status=ACTIVE

10/02 14:03:10: Ana retorna para Crypteras
  Sistema: "✅ Pagamento aprovado! Assinatura PRO ativada."

10/02 14:05: Ana cria 3 contextos (BTC, ETH, SOL)
  Sistema: Todos criados com sucesso (limite PRO: 3/3)
```

**Observação**: Mercado Pago pagamento único (não recorrente neste fluxo)
- Ana precisa renovar manualmente todo mês
- OU usar assinatura recorrente MP (preapproval)

---

### Caso 5: Webhook Duplicado (Idempotência)

**Cenário**: Stripe retenta webhook devido a timeout de rede

**Jornada**:
```
12/03 10:00:00: Stripe envia webhook: checkout.session.completed
  POST /api/webhooks/stripe
  Body: {"type": "checkout.session.completed", "user_id": "user_abc"}

12/03 10:00:02: Sistema processa webhook
  ✓ Valida assinatura HMAC
  ✓ Localiza usuário user_abc
  ✓ Atualiza: plan=FREE → plan=PRO
  ✓ Prepara resposta: HTTP 200

12/03 10:00:03: (TIMEOUT DE REDE - resposta não chega no Stripe)

12/03 10:00:10: Stripe não recebeu confirmação (HTTP 200)
  Stripe assume: "Webhook falhou"
  Stripe agenda retry #1

12/03 10:01:00: Stripe retenta webhook (DUPLICADO)
  POST /api/webhooks/stripe
  Body: (IDÊNTICO ao anterior)

12/03 10:01:02: Sistema processa webhook novamente
  ✓ Valida assinatura HMAC (mesma assinatura, válida)
  ✓ Localiza usuário user_abc
  ✓ Atualiza: plan=PRO → plan=PRO (SEM MUDANÇA)
  ✓ Retorna HTTP 200

12/03 10:01:03: Stripe recebe confirmação
  Stripe: "Webhook processado com sucesso"
  Stripe para de retentar

Resultado Final:
  ✅ Usuário tem plano PRO (correto)
  ✅ Sem duplicação de assinatura
  ✅ Sem cobrança dupla
  ✅ Sistema resiliente a falhas de rede
```

---

## Regras de Negócio Principais

### 1. Hierarquia de Planos
```
FREE (R$ 0) → PRO (R$ 19,90) → MAX (R$ 97)
  ↑ Upgrade permitido
  ↓ Downgrade via cancelamento (não direto)
```

### 2. Acesso ao Trading
```
Regra: user.can_trade() == (plan in [PRO, MAX]) AND (status == ACTIVE)

Exemplos:
  plan=PRO, status=ACTIVE → ✅ Pode criar bots
  plan=PRO, status=PAST_DUE → ❌ Não pode criar bots
  plan=FREE, status=ACTIVE → ❌ Limite: 1 contexto
```

### 3. Limites por Plano
```
FREE:
  - Contextos: 1
  - SmartBots: 1
  - CandleBots: 1
  - DCABots: 1

PRO:
  - Contextos: 3
  - SmartBots: 3
  - CandleBots: 5
  - DCABots: 5

MAX:
  - Contextos: Ilimitado
  - Todos os bots: Ilimitado
```

### 4. Período de Assinatura
```
PRO/MAX: SEMPRE tem current_period_start e current_period_end
FREE: Período = None (sem cobrança)

Validação: Se plan == PRO e period_end == None → Erro (estado inválido)
```

### 5. Cancelamento
```
Stripe: cancel_at_period_end=True
  → Mantém acesso até fim do período
  → Não cobra próxima renovação
  → Webhook subscription.deleted enviado ao fim

Mercado Pago: Cancelamento imediato
  → Perde acesso na hora
  → Downgrade para FREE instantâneo
```

### 6. Idempotência Obrigatória
```
Todos os webhooks DEVEM ser idempotentes:
  - Processar 2x o mesmo webhook = mesmo resultado
  - Sem efeitos colaterais (duplicação, cobranças extras)
  - Protegido por:
    - Busca por IDs únicos (user_id, stripe_subscription_id)
    - Verificações de estado (if plan != PRO)
    - Atualizações com mesmos valores
```

### 7. Validação de Segurança
```
Stripe: OBRIGATÓRIO validar HMAC-SHA256
  - Previne webhooks falsificados
  - Previne replay attacks (timestamp)

Mercado Pago: Validação indireta via API
  - Busca recurso na API MP (verifica existência)
  - Sem validação de assinatura (limitação do MP)
```

### 8. Eventual Consistency
```
Tempo médio: < 5 segundos
Pior caso: < 5 minutos (se retry)

Usuário pode ver:
  "Processando pagamento..." (webhook ainda não chegou)
  Solução: Polling ou WebSocket para atualização em tempo real
```

### 9. Retry Automático
```
Stripe: Até 5 dias de retries
  - 1min, 5min, 30min, 1h, 3h, 6h, 12h, 24h...
  - Sistema DEVE retornar HTTP 200 para parar retry
  - HTTP 4xx/5xx → Stripe continua retentando

Mercado Pago: Similar (retry automático em caso de erro)
```

### 10. Graceful Degradation
```
Se assinatura não encontrada no banco:
  → Retorna HTTP 200 {"status": "ignored"}
  → NÃO falha (pode ser race condition)
  → Webhook será retentado e pode encontrar na próxima vez

Se evento não suportado:
  → Retorna HTTP 200 {"status": "ignored"}
  → Log de warning (não erro)
  → Sistema continua funcionando
```

---

## Métricas de Sucesso

### Operacionais
- **Taxa de Sucesso de Webhooks**: > 99.9%
- **Tempo Médio de Processamento**: < 200ms
- **Taxa de Retry**: < 5% (maioria processa na primeira tentativa)
- **Downtime de Sincronização**: 0 segundos (webhooks sempre processados)

### Negócio
- **Conversão FREE → PRO**: > 10%
- **Taxa de Falha de Pagamento**: < 2% (cartões válidos)
- **Taxa de Recuperação (Retry)**: > 60% (pagamentos inicialmente falhos que são recuperados)
- **Churn Rate**: < 5%/mês

### Segurança
- **Webhooks Rejeitados (Assinatura Inválida)**: 0 (nenhum ataque bem-sucedido)
- **Tempo de Detecção de Fraude**: < 1 minuto (validação instantânea)

---

## Problemas Comuns e Soluções

### ❌ "Paguei mas assinatura não ativou"
**Causa**: Webhook ainda não processado (eventual consistency)
**Solução**: Aguardar até 5 minutos. Se persistir, verificar logs de webhook.

### ❌ "Webhook signature invalid"
**Causa**: Webhook secret incorreto em .env ou payload modificado
**Solução**: Verificar STRIPE_WEBHOOK_SECRET no Stripe Dashboard

### ❌ "Subscription not found in database"
**Causa**: Race condition (webhook chegou antes da criação da assinatura)
**Solução**: Stripe retenta automaticamente. Assinatura será ativada no retry.

### ❌ "Duplicate subscription activated"
**Causa**: Webhook duplicado sem proteção de idempotência
**Solução**: Verificar if plan != PRO antes de ativar (já implementado)

---

## Roadmap Futuro

### Melhorias Planejadas (Q1-Q2 2025)
- [ ] **Reconciliação Batch**: Job diário/horário sincroniza com Stripe/MP (detecta divergências)
- [ ] **Dashboard de Webhooks**: Monitoramento em tempo real (sucesso, falhas, retry)
- [ ] **Dead Letter Queue**: Persistir webhooks que falharam validação
- [ ] **Webhook Replay Manual**: Admin pode reprocessar webhooks via backoffice
- [ ] **Alertas Proativos**: Slack/email quando webhook falha > 3x
- [ ] **Assinatura Anual**: Desconto 20% (10 meses pelo preço de 12)
- [ ] **Trial 7 dias**: PRO grátis por 7 dias (webhook trial_will_end)

### Visão de Longo Prazo
- [ ] **Multi-Currency**: Suporte USD, EUR (Stripe internacional)
- [ ] **Invoice Generation**: PDF de notas fiscais automáticas
- [ ] **Refund Handling**: Webhook refund.created (estorno automático)
- [ ] **Dunning Management**: Emails automáticos para recuperar pagamentos falhos
- [ ] **Churn Prediction**: ML para detectar usuários em risco (oferecer desconto)

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
