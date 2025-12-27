---
spec_version: "1.4.0"
valid_from: "2025-12-27"
last_updated: "2025-12-27"
supersedes: "1.3.0"
status: "active"
category: "technical"
tags: ['technical', 'api_specification']
---

# API Specification - Crypteras Trading System

:::version_info
**Versão**: 1.4.0
**Válida desde**: 2025-12-27
**Status**: Ativa
:::

:::intent
**Goal**: Documentar contratos da API REST para garantir consistência de interface e facilitar integração frontend/backend.

**Constraints** (limites obrigatórios):
- Manter contratos de API retrocompatíveis (sem breaking changes em minor versions)
- Seguir padrões REST (GET para consulta, POST para criação, PUT/PATCH para atualização, DELETE para remoção)
- Todos endpoints protegidos devem exigir JWT válido no header `Authorization: Bearer <token>`
- Responses devem seguir formato JSON com status HTTP apropriado (200/201 sucesso, 400 bad request, 401 unauthorized, 404 not found, 500 server error)
- Validação de inputs obrigatória (usar Pydantic models)
- Rate limiting aplicado (conforme subscription plan: FREE 10 req/min, PRO 100 req/min, MAX unlimited)
- Documentação OpenAPI/Swagger disponível em `/docs` (FastAPI auto-generated)
- Versionamento de API via path prefix (`/api/v1/`, `/api/v2/`) para breaking changes

**Non-Goals** (o que NÃO fazer):
- Alterar formato de responses existentes sem versionamento (ex: renomear campo `bot_id` para `id` quebraria clients)
- Adicionar autenticação OAuth/Social login no MVP (apenas email+senha+JWT)
- Implementar GraphQL ou gRPC (apenas REST no escopo atual)
- Expor endpoints internos de Celery/Redis (apenas API de negócio)
- Permitir requests não autenticados para recursos protegidos (exceto `/auth/login` e `/auth/register`)
- Criar endpoints SOAP/XML (apenas JSON)
- Suportar múltiplas versões de API simultaneamente (deprecated versions devem ser removidas após grace period)
:::

:::failure_modes
**Falhas Conhecidas de API e Autenticação**:

1. **Exchange Credentials - API Key Errada para Exchange**
   - **Tipo**: integration
   - **Descrição**: `ExchangeFactory.create()` usa credenciais do Mercado Bitcoin para conectar na Binance (ou vice-versa)
   - **Gatilho**: POST `/api/bots/smart` com `exchange: "binance"` mas user só tem `encrypted_credentials['mb_api_key']`
   - **Impacto**: 🔴 Crítico (autenticação falha, bot não funciona, erro 401/403 da exchange)
   - **Mitigação**: SEMPRE validar `bot.exchange` e usar credenciais correspondentes:
     ```python
     if bot.exchange == "mercado_bitcoin":
         if 'mb_api_key' not in user.encrypted_credentials:
             raise HTTPException(400, "Mercado Bitcoin credentials not configured")
         api_key = decrypt(user.encrypted_credentials['mb_api_key'])
     elif bot.exchange == "binance":
         if 'binance_api_key' not in user.encrypted_credentials:
             raise HTTPException(400, "Binance credentials not configured")
         api_key = decrypt(user.encrypted_credentials['binance_api_key'])
     ```
   - **Detecção**: Response 400 "Invalid exchange credentials". Logs mostram "401 Unauthorized" da exchange API

2. **JWT Expiration Não Validado**
   - **Tipo**: security
   - **Descrição**: IA esquece de usar `@UseGuards(JwtAuthGuard)` ou equivalente, permitindo tokens expirados
   - **Gatilho**: Implementar novo endpoint protegido
   - **Impacto**: 🔴 Crítico (sessões expiradas continuam válidas, brecha de segurança)
   - **Mitigação**: TODOS os endpoints protegidos DEVEM usar `Depends(get_current_user)` (FastAPI):
     ```python
     @router.get("/api/bots/smart")
     async def list_smart_bots(current_user: User = Depends(get_current_user)):
         # get_current_user valida JWT e expiração
         bots = await bot_repository.get_by_user(current_user.id)
         return bots
     ```
   - **Detecção**: Teste com JWT expirado deve retornar 401. `curl -H "Authorization: Bearer <expired_token>" /api/bots`

3. **Pagination Missing em List Endpoints**
   - **Tipo**: hallucination
   - **Descrição**: Endpoint GET que lista recursos não implementa paginação (retorna todos os registros)
   - **Gatilho**: GET `/api/orders`, `/api/bots/smart`, etc sem `?page=&limit=`
   - **Impacto**: 🟡 Médio (performance degrada com muitos dados, timeout, OOM)
   - **Mitigação**: TODOS os endpoints de lista DEVEM ter paginação:
     ```python
     @router.get("/api/orders")
     async def list_orders(
         page: int = Query(1, ge=1),
         limit: int = Query(50, ge=1, le=100),
         current_user: User = Depends(get_current_user)
     ):
         skip = (page - 1) * limit
         orders = await order_repository.find(user_id=current_user.id).skip(skip).limit(limit)
         total = await order_repository.count(user_id=current_user.id)
         return {
             "items": orders,
             "total": total,
             "page": page,
             "pages": (total + limit - 1) // limit
         }
     ```
   - **Detecção**: Query sem `.limit()` ou `.skip()` no repository. Response sem campos `page`, `total`

4. **Error Response Sem Status Code Correto**
   - **Tipo**: hallucination
   - **Descrição**: Exceções lançam erro genérico 500 ao invés de status apropriado (400, 404, 409, etc)
   - **Gatilho**: Validações de negócio ou recursos não encontrados
   - **Impacto**: 🟢 Baixo (funciona mas cliente recebe status errado, logs poluídos)
   - **Mitigação**: Usar exceptions do FastAPI com status correto:
     ```python
     # ✅ CORRETO
     if not bot:
         raise HTTPException(status_code=404, detail="Bot not found")

     if bot.user_id != current_user.id:
         raise HTTPException(status_code=403, detail="Forbidden")

     if bot.max_position_size < Decimal("100"):
         raise HTTPException(status_code=400, detail="max_position_size must be >= 100")

     # ❌ ERRADO
     if not bot:
         raise Exception("Bot not found")  # Retorna 500!
     ```
   - **Detecção**: Logs mostram status 500 para erros de validação. Cliente recebe `Internal Server Error` ao invés de mensagem específica

5. **Rate Limiting Não Implementado**
   - **Tipo**: hallucination
   - **Descrição**: Endpoints não aplicam rate limiting por subscription plan
   - **Gatilho**: Usuário FREE faz 100 req/min (deveria ser bloqueado em 10 req/min)
   - **Impacto**: 🟡 Médio (abuse de API, custo de infra aumenta)
   - **Mitigação**: Implementar middleware de rate limiting:
     ```python
     from slowapi import Limiter
     from slowapi.util import get_remote_address

     limiter = Limiter(key_func=get_remote_address)

     @app.get("/api/bots/smart")
     @limiter.limit("10/minute", override_defaults=False)  # FREE plan
     async def list_smart_bots(request: Request, current_user: User = Depends(get_current_user)):
         # Ajustar limite baseado em subscription
         if current_user.subscription.plan == "PRO":
             limiter.limit("100/minute")
         # ...
     ```
   - **Detecção**: Usuário consegue fazer mais requests do que permitido. Logs não mostram "429 Too Many Requests"

6. **CORS Headers Faltando ou Incorretos**
   - **Tipo**: integration
   - **Descrição**: Frontend não consegue fazer requests devido a CORS bloqueado
   - **Gatilho**: Frontend rodando em `http://localhost:3000` tenta acessar backend `http://localhost:8000`
   - **Impacto**: 🔴 Crítico (frontend não funciona, console mostra "CORS policy" error)
   - **Mitigação**: Configurar CORS corretamente no backend:
     ```python
     from fastapi.middleware.cors import CORSMiddleware

     app.add_middleware(
         CORSMiddleware,
         allow_origins=[
             "http://localhost:3000",  # Frontend dev
             "https://app.crypteras.com"  # Frontend prod
         ],
         allow_credentials=True,
         allow_methods=["*"],
         allow_headers=["*"],
     )
     ```
   - **Detecção**: Browser console mostra "Access-Control-Allow-Origin" error. Request funciona no curl mas falha no browser

7. **Frontend - Hardcoded API URLs ao Invés de Variáveis de Ambiente**
   - **Tipo**: hallucination
   - **Descrição**: IA hardcoda URLs de API no frontend (`http://localhost:7777`, `http://localhost:8000`) ao invés de usar `useRuntimeConfig()` (Nuxt) ou variáveis de ambiente
   - **Gatilho**: Implementar fetch/axios calls em componentes Vue, composables, ou páginas
   - **Impacto**: 🟡 Médio (frontend quebra em produção, requests vão para localhost ao invés de API de produção)
   - **Mitigação**: SEMPRE usar `useRuntimeConfig()` para URLs de API. NUNCA hardcodar URLs
   - **Detecção**: Buscar `http://localhost` ou `https://api` hardcoded em arquivos `.vue`, `.ts`, `.js`
   - **Código Correto (Frontend Nuxt 3)**:
     ```typescript
     // ✅ CORRETO - Usar runtime config
     const config = useRuntimeConfig()
     const apiUrl = config.public.dashboardApiBase  // Do nuxt.config.ts
     const response = await fetch(`${apiUrl}/api/smart-bots`)

     // nuxt.config.ts
     export default defineNuxtConfig({
       runtimeConfig: {
         public: {
           dashboardApiBase: process.env.DASHBOARD_API_BASE || 'http://localhost:8000',
           agnoApiBase: process.env.AGNO_API_BASE || 'http://localhost:7777'
         }
       }
     })

     // ❌ ERRADO - Hardcoded
     const response = await fetch('http://localhost:8000/api/smart-bots')  // Quebra em prod!
     ```
   - **Código Correto (Backend FastAPI)**:
     ```python
     # ✅ CORRETO - Usar Pydantic Settings
     from pydantic_settings import BaseSettings

     class Settings(BaseSettings):
         mongodb_uri: str
         dashboard_api_base: str

         class Config:
             env_file = ".env"

     settings = Settings()
     mongo_client = AsyncIOMotorClient(settings.mongodb_uri)

     # ❌ ERRADO - Hardcoded
     mongo_client = AsyncIOMotorClient('mongodb://localhost:27017')  # Quebra em prod!
     ```
:::

:::explainability
**Requirement**: ✅ Required (para decisões de integração e contratos de API)

**Output Format**:
IA DEVE explicar decisões de API seguindo este formato:

```markdown
## 🤖 Decisão de API

**Decisão**: [O que foi decidido - ex: "Usar paginação com `page` e `limit` ao invés de cursor-based"]

**Source**:
- `specs/technical/API_SPECIFICATION.md` v1.3.0 - Seção relevante
- `specs/technical/API_SPECIFICATION.md` v1.3.0 - Failure Mode #3 (Pagination Missing)
- Endpoints existentes (se aplicável)

**Rationale**:
1. [Razão baseada na spec]
2. [Compatibilidade com padrões REST]
3. [Consistência com endpoints existentes]
4. [Facilidade de uso para frontend]

**Alternatives Considered**:
1. ❌ [Alternativa 1] - [Por que não foi escolhida]
2. ❌ [Alternativa 2] - [Por que não foi escolhida]
3. ✅ [Escolhida] - [Por que foi escolhida]

**Trade-offs**:
- ✅ Pro: [Benefício 1]
- ✅ Pro: [Benefício 2]
- ⚠️ Con: [Desvantagem se houver]

**Impact**:
- **Frontend**: [Como afeta cliente]
- **Performance**: [Impacto de performance]
- **Security**: [Implicações de segurança se aplicável]

**Audit Trail**:
- Timestamp: [ISO 8601 format]
- Specs Consultadas: [API_SPECIFICATION.md v1.x.x]
- Endpoints Afetados: [lista de endpoints]
- Failure Modes Aplicados: [se aplicável]
```

**Quando Explicar** (gatilhos obrigatórios):
1. **Criação de novo endpoint**: Método HTTP, path, autenticação requerida
2. **Mudança de contrato existente**: Breaking change em request/response
3. **Escolha de status code HTTP**: 200 vs 201, 400 vs 422, etc
4. **Decisão de autenticação/autorização**: JWT, OAuth, rate limiting
5. **Implementação de paginação**: Offset vs cursor, limit default
6. **Validação de input**: Pydantic models, regex, ranges
7. **Formato de erro response**: Estrutura JSON, error codes
8. **Exchange credentials handling**: Qual API key usar, encriptação
9. **CORS configuration**: Origins permitidas, credentials
10. **Implementação de failure mode mitigation**: Correção de falhas conhecidas de API

**Audit Trail Obrigatório**:
- Timestamp da decisão
- Specs consultadas (nome + versão)
- Endpoints criados/modificados
- Failure modes aplicados (se relevante)
- Impacto em contratos existentes

**Exemplo Completo**:

```markdown
## 🤖 Decisão de API

**Decisão**: Implementar paginação offset-based em `GET /api/orders` com `page` e `limit` query params

**Source**:
- `specs/technical/API_SPECIFICATION.md` v1.3.0 - Failure Mode #3 (Pagination Missing)
- `specs/technical/API_SPECIFICATION.md` v1.3.0 - Constraint "Todos endpoints de lista DEVEM ter paginação"
- Endpoint existente: `GET /api/smart-bots` (já usa paginação similar)

**Rationale**:
1. **Failure Mode Mitigation**: Sem paginação, endpoint retorna TODOS os registros, causando timeout/OOM (FM#3)
2. **Constraint Obrigatório**: Spec exige paginação em endpoints de lista
3. **Consistência**: `GET /api/smart-bots` já usa offset-based pagination (page/limit)
4. **Simplicidade Frontend**: Offset-based é mais simples que cursor-based para UI com números de página

**Alternatives Considered**:
1. ❌ Sem paginação - Violaria constraint, causaria FM#3 (performance degrada com muitos dados)
2. ❌ Cursor-based (`after=`, `before=`) - Mais complexo, inconsistente com endpoints existentes
3. ❌ Scroll API (Elasticsearch-style) - Overhead desnecessário para caso de uso simples
4. ✅ Offset-based (page/limit) - Simples, consistente, adequado para caso de uso

**Trade-offs**:
- ✅ Pro: Simples para frontend implementar
- ✅ Pro: Consistente com `/api/smart-bots`, `/api/candle-bots`
- ✅ Pro: Permite pular para página específica (ex: página 5)
- ⚠️ Con: Performance degrada em offsets grandes (mitigado por `limit` máximo de 100)
- ⚠️ Con: Dados podem mudar entre requisições (aceitável para caso de uso de dashboard)

**Impact**:
- **Frontend**: Pode reusar componente de paginação existente
- **Performance**: Limite máximo de 100 previne queries pesadas
- **Backward Compatibility**: Endpoint novo, sem breaking changes

**Audit Trail**:
- Timestamp: 2025-12-25T11:00:00Z
- Specs Consultadas: API_SPECIFICATION.md v1.3.0
- Endpoints Criados: GET /api/orders (com paginação)
- Failure Modes Aplicados: FM#3 (Pagination Missing)
- Consistency Check: Validado contra GET /api/smart-bots
```

**Quando NÃO Explicar** (decisões triviais):
- CRUD endpoints padrão seguindo convenções REST existentes
- Status codes óbvios (200 para success, 404 para not found)
- Validações simples já cobertas por Pydantic
- Headers de autenticação padrão (`Authorization: Bearer`)
- Formatos JSON já estabelecidos no projeto
:::

:::breaking_changes
**v1.4.0** (2025-12-27):
- Adicionado novo failure mode #7: Frontend Hardcoded API URLs
- Documentado padrão correto para useRuntimeConfig() (Nuxt 3) e Pydantic Settings (FastAPI)
- Total: 7 failure modes documentados
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.3.0** (2025-12-25):
- Adicionada seção `:::explainability` com requisitos obrigatórios para decisões de API
- Definidos 10 gatilhos obrigatórios de explainability para contratos de API
- Incluído exemplo completo de explicação de decisão de paginação
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.2.0**:
- Adicionada seção `:::failure_modes` com 6 falhas conhecidas de API e autenticação
- Documentadas falhas reais: Exchange credentials erradas, JWT expiration, Pagination missing, Error status codes incorretos, Rate limiting, CORS
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Especificação da API REST e endpoints
:::

Este documento lista os endpoints REST disponíveis baseado em análise do código em `backend/src/api/routes/`.

**Base URLs**:
- Trading API: `http://localhost:7777`
- Dashboard API: `http://localhost:8000`

---

## 🔐 Autenticação

### JWT Authentication

**Endpoints de Auth**:
```
POST /api/auth/register
POST /api/auth/login
GET /api/auth/me
```

**Headers Obrigatórios** (exceto register/login):
```http
Authorization: Bearer <jwt_token>
```

**Exemplo de Login**:
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "senha123"
}
```

**Resposta**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "subscription": {
      "plan": "PRO",
      "status": "active"
    }
  }
}
```

---

## 🤖 SmartBots

**Base Path**: `/api/smart-bots`

### Listar SmartBots
```http
GET /api/smart-bots
Authorization: Bearer <token>
```

**Query Parameters**:
- `status` (opcional): `ACTIVE` | `PAUSED` | `ARCHIVED`

**Resposta**:
```json
{
  "data": [
    {
      "id": "bot_123",
      "name": "BTC Conservative",
      "exchange": "mercado_bitcoin",
      "symbol": "BTC-BRL",
      "operation_mode": "CONTINUOUS",
      "status": "ACTIVE",
      "config": {
        "max_position_size": 5000.00,
        "trailing_stop_percentage": 3.0,
        "soft_stop_loss": 2.0,
        "hard_stop_loss": 5.0
      },
      "stats": {
        "total_invested": 3000.00,
        "total_blocked": 500.00,
        "total_purchases": 15,
        "circuit_breaker_active": false
      },
      "created_at": "2024-12-01T10:00:00Z"
    }
  ],
  "total": 1
}
```

### Criar SmartBot
```http
POST /api/smart-bots
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "BTC Conservative",
  "exchange": "mercado_bitcoin",
  "symbol": "BTC-BRL",
  "operation_mode": "CONTINUOUS",
  "intervals": {
    "purchase": 63,
    "sales": 67,
    "check_situation": 33
  },
  "config": {
    "max_position_size": 5000.00,
    "trailing_stop_percentage": 3.0,
    "soft_stop_loss": 2.0,
    "hard_stop_loss": 5.0,
    "order_strategy": "LIMIT"
  }
}
```

### Obter SmartBot
```http
GET /api/smart-bots/{bot_id}
Authorization: Bearer <token>
```

### Atualizar SmartBot
```http
PATCH /api/smart-bots/{bot_id}
Authorization: Bearer <token>

{
  "config": {
    "max_position_size": 10000.00
  }
}
```

### Pausar SmartBot
```http
POST /api/smart-bots/{bot_id}/pause
Authorization: Bearer <token>
```

### Retomar SmartBot
```http
POST /api/smart-bots/{bot_id}/resume
Authorization: Bearer <token>
```

### Arquivar SmartBot
```http
POST /api/smart-bots/{bot_id}/archive
Authorization: Bearer <token>
```

---

## 📊 CandleBots

**Base Path**: `/api/candle-bots`

### Criar CandleBot
```http
POST /api/candle-bots
Authorization: Bearer <token>

{
  "name": "ETH Swing Trader",
  "exchange": "binance",
  "symbol": "ETHUSDT",
  "config": {
    "timeframe": "15m",
    "lookback_periods": 50,
    "indicators": {
      "rsi_enabled": true,
      "rsi_period": 14,
      "ma_enabled": true,
      "ma_period": 20,
      "bb_enabled": true,
      "bb_period": 20,
      "vwap_enabled": true
    },
    "take_profit_percentage": 5.0,
    "stop_loss_percentage": 2.0,
    "auto_execute": true
  }
}
```

**Endpoints** (similares ao SmartBot):
- `GET /api/candle-bots` - Listar
- `GET /api/candle-bots/{bot_id}` - Obter
- `PATCH /api/candle-bots/{bot_id}` - Atualizar
- `POST /api/candle-bots/{bot_id}/pause` - Pausar
- `POST /api/candle-bots/{bot_id}/resume` - Retomar

---

## 💰 DCABots

**Base Path**: `/api/dca-bots`

### Criar DCABot
```http
POST /api/dca-bots
Authorization: Bearer <token>

{
  "name": "BTC Weekly DCA",
  "exchange": "mercado_bitcoin",
  "symbol": "BTC-BRL",
  "config": {
    "purchase_amount_brl": 100.00,
    "frequency": "7d",
    "allocated_balance": 5000.00,
    "take_profit_percentage": 20.0,
    "stop_loss_percentage": 10.0,
    "max_cycles": 52,
    "auto_restart_after_exit": false
  }
}
```

**Endpoints** (similares ao SmartBot):
- `GET /api/dca-bots`
- `GET /api/dca-bots/{bot_id}`
- `PATCH /api/dca-bots/{bot_id}`
- `POST /api/dca-bots/{bot_id}/pause`
- `POST /api/dca-bots/{bot_id}/resume`

---

## 📈 Dashboard & Métricas

**Base Path**: `/api/dashboard`

### Health Check
```http
GET /api/dashboard/health
```

**Resposta**:
```json
{
  "status": "healthy",
  "timestamp": "2024-12-24T10:00:00Z",
  "services": {
    "mongodb": "up",
    "redis": "up",
    "celery_workers": 2
  }
}
```

### Performance Global
```http
GET /api/dashboard/performance
Authorization: Bearer <token>
```

**Resposta**:
```json
{
  "total_bots": 10,
  "active_bots": 7,
  "total_invested": 15000.00,
  "total_profit_loss": 1250.50,
  "profit_loss_percentage": 8.34,
  "best_performing_bot": {
    "id": "bot_123",
    "name": "BTC Conservative",
    "profit_percentage": 12.5
  }
}
```

### Ordens Recentes
```http
GET /api/dashboard/orders
Authorization: Bearer <token>
```

**Query Parameters**:
- `limit` (opcional): Número de ordens (default: 50)
- `status` (opcional): `FILLED` | `PENDING` | `CANCELED`
- `bot_id` (opcional): Filtrar por bot

**Resposta**:
```json
{
  "data": [
    {
      "id": "order_123",
      "exchange_order_id": "MB-123456",
      "bot_id": "bot_123",
      "bot_type": "smart_bot",
      "exchange": "mercado_bitcoin",
      "symbol": "BTC-BRL",
      "side": "BUY",
      "type": "LIMIT",
      "price": 155000.00,
      "quantity": 0.001,
      "status": "FILLED",
      "filled_at": "2024-12-24T10:15:00Z",
      "created_at": "2024-12-24T10:14:50Z"
    }
  ],
  "total": 1
}
```

---

## 💳 Assinaturas

**Base Path**: `/api/subscriptions`

### Obter Limites do Plano
```http
GET /api/subscriptions/limits
Authorization: Bearer <token>
```

**Resposta**:
```json
{
  "current_plan": "PRO",
  "limits": {
    "max_contexts": 3,
    "max_bots_total": 10
  },
  "usage": {
    "contexts_used": 2,
    "bots_created": 5
  },
  "can_create_more": true
}
```

### Upgrade de Plano
```http
POST /api/subscriptions/upgrade
Authorization: Bearer <token>

{
  "plan": "MAX",
  "payment_provider": "mercadopago"
}
```

**Resposta**:
```json
{
  "success": true,
  "payment_url": "https://www.mercadopago.com.br/subscriptions/checkout?preapproval_id=...",
  "subscription_id": "preapproval_123"
}
```

---

## ⚙️ Contextos (Legacy/Internal)

**Nota**: Contextos são internos. Usuários criam bots, não contextos diretamente.

**Base Path**: `/api/contexts`

### Listar Contextos
```http
GET /api/contexts
Authorization: Bearer <token>
```

### Criar Contexto
```http
POST /api/contexts
Authorization: Bearer <token>

{
  "name": "Contexto Principal",
  "exchange": "mercado_bitcoin",
  "symbol": "BTC-BRL"
}
```

**Validação**: Verifica limites de plano antes de criar.

---

## 🎮 Playground (AGNO)

**Base Path**: `/playground`

### Status do Playground
```http
GET /playground/status
```

**Resposta**:
```json
{
  "playground": "available",
  "version": "2.4",
  "teams": ["Market Intelligence", "Trading Execution"]
}
```

### Interagir com Team
```http
POST /playground/chat
Content-Type: application/json

{
  "team": "Market Intelligence",
  "message": "Analise o mercado BTC-BRL agora"
}
```

---

## 🔴 Códigos de Erro

### 400 Bad Request
```json
{
  "error": "validation_error",
  "detail": "max_position_size deve ser maior que 0"
}
```

### 401 Unauthorized
```json
{
  "error": "unauthorized",
  "detail": "Token inválido ou expirado"
}
```

### 403 Forbidden
```json
{
  "error": "insufficient_subscription",
  "detail": "Plano FREE permite apenas 1 contexto. Faça upgrade para PRO."
}
```

### 404 Not Found
```json
{
  "error": "not_found",
  "detail": "Bot com ID bot_123 não encontrado"
}
```

### 409 Conflict
```json
{
  "error": "version_conflict",
  "detail": "Bot foi modificado por outro processo. Recarregue e tente novamente."
}
```

### 500 Internal Server Error
```json
{
  "error": "internal_error",
  "detail": "Erro ao conectar com exchange"
}
```

---

## 📊 Rate Limiting

**Limites Atuais** (não implementados, mas recomendados):
- Endpoints de leitura: 100 requests/minuto
- Endpoints de escrita: 30 requests/minuto
- Playground: 10 requests/minuto

**Headers de Response**:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

---

## 🧪 Testando a API

### cURL Examples

**Login**:
```bash
curl -X POST http://localhost:7777/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "senha123"}'
```

**Criar SmartBot**:
```bash
TOKEN="eyJhbGciOiJIUzI1NiIs..."

curl -X POST http://localhost:7777/api/smart-bots \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "BTC Conservative",
    "exchange": "mercado_bitcoin",
    "symbol": "BTC-BRL",
    "operation_mode": "CONTINUOUS",
    "config": {
      "max_position_size": 5000.00,
      "soft_stop_loss": 2.0,
      "hard_stop_loss": 5.0
    }
  }'
```

**Listar Bots**:
```bash
curl -X GET http://localhost:7777/api/smart-bots \
  -H "Authorization: Bearer $TOKEN"
```

### Postman Collection

**Arquivo**: `docs/Crypteras_API.postman_collection.json` (existente no projeto)

Importar no Postman para ter todos endpoints pré-configurados.

---

## 📚 Documentação Interativa

### Swagger UI

FastAPI gera documentação automática:

```
http://localhost:7777/docs
http://localhost:8000/docs
```

**Features**:
- Testar endpoints diretamente
- Ver schemas de request/response
- Autenticação JWT integrada

### ReDoc

Alternativa ao Swagger:

```
http://localhost:7777/redoc
http://localhost:8000/redoc
```

---

## 🔄 Webhooks (Futuro)

**Não implementados atualmente**. Planejados para:
- Mercado Pago: Notificações de pagamento
- Binance: Order updates
- Notificações de bot events

---

## 📋 Changelog da API

### v2.4 (CRY-56) - 2024-11
- Adicionado `/api/subscriptions/limits`
- Adicionado `/api/subscriptions/upgrade`
- Validação de limites de plano em criação de bots

### v2.3 (CRY-39) - 2024-10
- Adicionado `strategy_context` em ordens (audit trail)
- Endpoint `/api/orders/{order_id}/context`

### v2.2 (CRY-20) - 2024-10
- Suporte a Binance (`exchange: "binance"`)
- 16 timeframes para CandleBots

---

**Última Atualização**: 2024-12-24
**Baseado em**: Código em `backend/src/api/routes/`
**Documentação Interativa**: http://localhost:7777/docs
