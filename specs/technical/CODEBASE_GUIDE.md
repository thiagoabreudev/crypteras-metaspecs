---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
category: "technical"
tags: ['codebase_guide', 'technical']
---

# Guia de Navegação da Base de Código - Crypteras Trading System

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Documentar estrutura da codebase para navegação eficiente e compreensão de onde cada responsabilidade reside.

**Constraints** (limites obrigatórios):
- Refletir estrutura real do repositório `/Users/thiagoabreu/workspace/crypteras-improved` com precisão
- Manter sincronizado com mudanças estruturais do projeto
- Referenciar ADRs relevantes para decisões arquiteturais (ADR-001 Clean Architecture, ADR-002 Celery, ADR-005 Exchanges)
- Indicar arquivos críticos com ⭐ e linha count
- Explicar responsabilidades de cada camada/pasta claramente
- Incluir exemplos de caminhos reais (ex: `backend/src/domain/entities/smart_bot.py:716`)

**Non-Goals** (o que NÃO fazer):
- Documentar implementação detalhada de cada arquivo (use BUSINESS_LOGIC.md ou CLAUDE.meta.md para isso)
- Duplicar conteúdo dos ADRs (apenas referencie-os)
- Incluir código-fonte inline (use file:line references)
- Criar diagramas complexos (manter texto simples e navegável)
- Documentar código deprecated ou experimental (apenas código principal ativo)
- Especular sobre estrutura futura (apenas documentar o que existe agora)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Guia de navegação da codebase
:::

Este documento mapeia a estrutura do código real em `/Users/thiagoabreu/workspace/crypteras-improved` para facilitar navegação e compreensão.

---

## 📁 Estrutura de Diretórios (Raiz)

```
crypteras-improved/
├── backend/                    # Backend Python (FastAPI + Celery)
├── frontend/                   # Frontend Nuxt.js 3 (Vue 3 + TypeScript)
├── observability/              # Dashboards Grafana
├── docs/                       # Documentação (PRDs, arquitetura)
├── scripts/                    # Scripts utilitários
├── docker-compose.yml          # Orquestração de 8 serviços
├── Dockerfile                  # Build do backend
├── README.md                   # Documentação principal
└── CLAUDE.md                   # Guia para IA (existente)
```

---

## 🐍 Backend (`backend/`)

### Estrutura Geral

```
backend/
├── src/                        # Código fonte principal
│   ├── domain/                 # Domain Layer (Clean Architecture)
│   ├── application/            # Application Layer (Use Cases + Workflows)
│   ├── infrastructure/         # Infrastructure Layer (MongoDB, Celery, etc.)
│   ├── exchanges/              # Adapters para exchanges (MB, Binance)
│   ├── agents/                 # Agentes AGNO (IA)
│   ├── teams/                  # Teams AGNO coordenados
│   ├── tools/                  # Tools para agentes (cached)
│   ├── api/                    # FastAPI routes
│   ├── core/                   # Configuração e modelos base
│   └── utils/                  # Utilitários
├── tests/                      # Testes (unit, integration)
├── main.py                     # Ponto de entrada FastAPI (1685 lines)
├── requirements.txt            # Dependências Python
└── scripts/                    # Scripts de migração/utilitários
```

### Domain Layer (`src/domain/`)

**Princípio**: Zero dependências de frameworks. Apenas lógica de negócio pura.

```
domain/
├── entities/
│   ├── smart_bot.py           # SmartBot entity (716 lines) ⭐
│   ├── candle_bot.py          # CandleBot entity (547 lines) ⭐
│   ├── dca_bot.py             # DCABot entity (318 lines) ⭐
│   ├── user.py                # User entity (credenciais encriptadas)
│   └── subscription.py        # Subscription entity (planos FREE/PRO/MAX)
├── services/
│   ├── risk_management.py     # Serviços de gestão de risco
│   ├── technical_analysis_service.py  # Cálculo de indicadores (RSI, MA, BB, etc.)
│   └── signal_consolidation_service.py  # Consolidação de sinais de candles
└── repositories/              # Interfaces abstratas (sem implementação)
    ├── smart_bot_repository.py
    ├── candle_bot_repository.py
    ├── dca_bot_repository.py
    └── user_repository.py
```

**Arquivos Mais Importantes**:

1. **`entities/smart_bot.py`** (716 lines):
   - `SmartBot` dataclass com toda lógica de negócio
   - Métodos chave: `can_execute_purchase()`, `get_available_funds()`, `should_trigger_stop_loss()`, `adjust_trailing_stop()`
   - Validações: `__post_init__()` valida soft_stop_loss < hard_stop_loss, max_position_size > 0, etc.

2. **`entities/candle_bot.py`** (547 lines):
   - `CandleBot` dataclass para análise técnica
   - `IndicatorsConfig` com 10 indicadores (RSI, MA, BB, VWAP, MFI, etc.)
   - Métodos: `evaluate_signal()`, `should_take_profit()`, `calculate_confidence()`

3. **`entities/dca_bot.py`** (318 lines):
   - `DCABot` dataclass para Dollar Cost Averaging
   - Métodos: `should_execute_now()`, `calculate_next_execution()`, `update_average_price()`
   - Frequências: 1h, 6h, 24h, 7d, 30d

### Application Layer (`src/application/`)

**Princípio**: Orquestração de Domain Services. Depende apenas do Domain.

```
application/
├── use_cases/
│   ├── contexts/
│   │   └── create_context.py      # Criar contexto de trading (validação de plano)
│   ├── auth/
│   │   └── register_user.py       # Registro de usuário (criptografia de credenciais)
│   ├── smart_bots/
│   │   ├── create_smart_bot.py    # Criar SmartBot (validação de limites)
│   │   └── update_smart_bot.py    # Atualizar config de SmartBot
│   └── subscriptions/
│       └── upgrade_to_pro.py      # Upgrade de plano (Mercado Pago)
└── workflows/                      # Workflows automatizados (Celery)
    ├── sync_orders.py             # Sincroniza ordens das exchanges (42s) ⭐
    ├── smart_bot_purchase.py      # Compra SmartBot (63s)
    ├── smart_bot_sales.py         # Venda SmartBot (67s)
    ├── smart_bot_check_situation.py  # Monitoramento de risco (33s)
    ├── candle_bot_analysis_workflow.py  # Análise técnica (120s)
    ├── candle_bot_risk_management_workflow.py  # Take profit/stop-loss (60s)
    ├── dca_bot_execution.py       # Execução DCA (60s)
    └── dca_bot_risk_management.py # Monitoramento DCA (30s)
```

**Workflows Críticos**:

1. **`sync_orders.py`** (1000+ lines):
   - **Frequência**: A cada 42s (co-primo)
   - **Função**: Busca ordens filled nas exchanges (MB, Binance) e atualiza MongoDB
   - **Problema Conhecido**: ~8% taxa de falha (ordens não sincronizadas corretamente)
   - **Fluxo**:
     ```python
     1. Para cada user ativo:
        2. Para cada bot do user:
           3. Buscar ordens filled na exchange (últimas 1h)
           4. Normalizar formato (OrderNormalizer)
           5. Salvar em MongoDB com strategy_context (CRY-39)
           6. Atualizar bot.stats.total_invested, total_blocked
     ```

2. **`smart_bot_purchase.py`**:
   - **Frequência**: A cada 63s (co-primo)
   - **Função**: Executa compras para SmartBots ativos em modo CONTINUOUS
   - **Fluxo**:
     ```python
     1. Buscar SmartBots ativos (status=ACTIVE, operation_mode=CONTINUOUS)
     2. Para cada bot:
        3. Distributed lock (Redis) para evitar version conflicts
        4. Verificar bot.can_execute_purchase()
        5. Criar exchange via ExchangeFactory
        6. Buscar orderbook
        7. Calcular preço competitivo (best_bid + 0.01)
        8. Criar ordem LIMIT na exchange
        9. Salvar ordem no MongoDB
        10. Atualizar bot.stats.total_blocked
     ```

### Infrastructure Layer (`src/infrastructure/`)

**Princípio**: Implementações concretas de interfaces do Domain. Único lugar com frameworks.

```
infrastructure/
├── repositories/
│   ├── smart_bot_repository_impl.py   # MongoDB para SmartBots
│   ├── candle_bot_repository_impl.py  # MongoDB para CandleBots
│   ├── dca_bot_repository_impl.py     # MongoDB para DCABots
│   ├── order_repository.py            # MongoDB para Orders
│   └── user_repository.py             # MongoDB para Users (com criptografia)
├── exchanges/
│   ├── exchange_factory.py            # Factory para criar exchanges
│   └── order_normalizer.py            # Normaliza formatos diferentes
├── celery/
│   ├── celery_app.py                  # Celery app (33 lines)
│   ├── celeryconfig.py                # Configuração (189 lines) ⭐
│   └── tasks/
│       ├── smart_bot_tasks.py         # Tasks SmartBot
│       ├── candle_bot_tasks.py        # Tasks CandleBot
│       ├── dca_bot_tasks.py           # Tasks DCABot
│       └── sync_orders_task.py        # Task sync_orders
└── encryption/
    └── credentials_encryptor.py       # AES-256 para credenciais de API
```

**Arquivos Importantes**:

1. **`celery/celeryconfig.py`** (189 lines):
   - **Broker**: `redis://:password@redis:6379/0`
   - **Backend**: `redis://:password@redis:6379/1`
   - **Queues**: 8 queues (sync_orders, smart_buy, smart_sell, smart_risk, candle_analysis, candle_risk, dca_execution, dca_risk)
   - **Beat Schedule**: Intervalos co-primos (33s, 42s, 60s, 63s, 67s, 120s)
   - **RedBeat**: Scheduler persistente no Redis

2. **`repositories/smart_bot_repository_impl.py`**:
   - **Collection**: `crypteras_trading.smart_bots`
   - **Métodos**:
     - `create(bot)`: Insere documento
     - `get_by_id(bot_id)`: Busca por _id
     - `get_active_bots(user_id)`: Busca bots ACTIVE de um user
     - `update(bot)`: Atualiza com **optimistic locking** (version)
   - **Índices**:
     ```python
     await collection.create_index("user_id")
     await collection.create_index([("user_id", 1), ("status", 1)])
     ```

### Exchanges (`src/exchanges/`)

**Princípio**: Adapter Pattern para múltiplas exchanges.

```
exchanges/
├── base_exchange.py              # Interface abstrata (ABC)
├── mercado_bitcoin.py            # Adapter MB API v4 (1086 lines) ⭐
└── binance.py                    # Adapter Binance API v3
```

**Interface BaseExchange**:
```python
class BaseExchange(ABC):
    @abstractmethod
    async def get_balance(self, asset: str) -> Decimal

    @abstractmethod
    async def get_orderbook(self, symbol: str, depth: int = 10) -> Dict

    @abstractmethod
    async def create_order(self, symbol: str, side: str, order_type: str, quantity: Decimal, price: Decimal = None) -> Dict

    @abstractmethod
    async def cancel_order(self, symbol: str, order_id: str) -> bool

    @abstractmethod
    async def get_candles(self, symbol: str, timeframe: str, limit: int = 100) -> List[Dict]

    @abstractmethod
    def normalize_symbol(self, symbol: str) -> str
```

**MercadoBitcoinExchange** (1086 lines):
- **Autenticação**: Login/password → Bearer token
- **Timeframes**: 7 opções (1m, 5m, 15m, 30m, 1h, 6h, 1d)
- **Formato**: `{"symbol": "BTC-BRL", "price": "155000.00"}` (strings)
- **Features**: Stop-limit, post-only orders

**BinanceExchange**:
- **Autenticação**: HMAC SHA256 (API key/secret)
- **Timeframes**: 16 opções (1s, 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M)
- **Formato**: `{"symbol": "BTCUSDT", "price": 31000.50}` (floats)
- **Status**: Em teste (ainda tem bugs)

### API (`src/api/`)

```
api/
├── routes/
│   ├── smart_bots.py             # CRUD SmartBots
│   ├── candle_bots.py            # CRUD CandleBots
│   ├── dca_bots.py               # CRUD DCABots
│   ├── dashboard.py              # Métricas e performance
│   ├── auth.py                   # Login/registro
│   └── subscriptions.py          # Planos e upgrades
└── middleware/
    ├── cors.py                   # CORS para frontend
    └── jwt_auth.py               # Autenticação JWT
```

### Main (`main.py`) - 1685 lines

**Ponto de Entrada**:
```python
# main.py
from fastapi import FastAPI
from src.api.routes import smart_bots, candle_bots, dca_bots, dashboard, auth, subscriptions

app = FastAPI(title="Crypteras Trading System")

# Middleware
app.add_middleware(CORSMiddleware, ...)

# Routes
app.include_router(smart_bots.router, prefix="/api/smart-bots")
app.include_router(candle_bots.router, prefix="/api/candle-bots")
app.include_router(dca_bots.router, prefix="/api/dca-bots")
app.include_router(dashboard.router, prefix="/api/dashboard")
app.include_router(auth.router, prefix="/api/auth")
app.include_router(subscriptions.router, prefix="/api/subscriptions")

# Lifespan (startup/shutdown)
@app.on_event("startup")
async def startup():
    # Conectar MongoDB
    # Registrar workflows no Celery Beat
    # Inicializar cache

@app.on_event("shutdown")
async def shutdown():
    # Fechar conexões
```

---

## 🎨 Frontend (`frontend/`)

### Estrutura Geral

```
frontend/
├── pages/                      # Páginas Nuxt (rotas automáticas)
│   ├── index.vue              # Dashboard principal
│   ├── smart-bots/
│   │   ├── index.vue          # Lista SmartBots
│   │   ├── [id].vue           # Detalhe SmartBot
│   │   └── create.vue         # Criar SmartBot
│   ├── candle-bots/
│   ├── dca-bots/
│   └── upgrade.vue            # Pricing page (planos)
├── components/                 # Componentes Vue
│   ├── smart-bot/
│   │   ├── SmartBotWizard.vue
│   │   ├── SmartBotCard.vue
│   │   └── SmartBotMetrics.vue
│   ├── candle-bot/
│   └── dca-bot/
├── composables/                # State management (Vue Composition API)
│   ├── useSmartBots.ts        # Estado SmartBots ⭐
│   ├── useCandleBots.ts
│   ├── useDCABots.ts
│   ├── useAuth.ts             # Autenticação
│   └── useDashboard.ts        # Métricas
├── middleware/                 # Route guards
│   ├── auth.ts                # Requer login
│   └── require-credentials.ts # Requer credenciais de exchange
├── nuxt.config.ts             # Configuração Nuxt
└── package.json
```

**Composables Principais**:

1. **`useSmartBots.ts`**:
   ```typescript
   export const useSmartBots = () => {
     const bots = ref<SmartBot[]>([])
     const loading = ref(false)

     const fetchBots = async () => {
       const response = await $fetch('/api/smart-bots')
       bots.value = response.data
     }

     const createBot = async (config: SmartBotConfig) => {
       return await $fetch('/api/smart-bots', {
         method: 'POST',
         body: config
       })
     }

     const pauseBot = async (botId: string) => {
       await $fetch(`/api/smart-bots/${botId}/pause`, { method: 'POST' })
     }

     return { bots, loading, fetchBots, createBot, pauseBot }
   }
   ```

2. **`nuxt.config.ts`**:
   ```typescript
   export default defineNuxtConfig({
     ssr: false,  // CSR-only (SSR desabilitado)
     modules: ['@nuxt/ui'],
     runtimeConfig: {
       public: {
         dashboardApiBase: 'http://localhost:8000',  // Dashboard API
         tradingApiBase: 'http://localhost:7777'     // Trading System API
       }
     }
   })
   ```

---

## 🐳 Docker (`docker-compose.yml`)

**8 Serviços Orquestrados**:

```yaml
services:
  crypteras-mongo:        # MongoDB 7.0 (port 27017)
  backend:                # FastAPI + AGNO (ports 7777, 8000)
  frontend:               # Nuxt.js dev server (port 3000)
  redis:                  # Redis 7.2 (port 6379)
  celery-beat:            # RedBeat scheduler
  celery-worker-sync:     # Worker sync_orders (10 concurrency)
  celery-worker-smart:    # Worker smart/candle/dca (5 concurrency)
  flower:                 # Monitoring dashboard (port 5555)
```

**Volumes**:
- `mongo-data`: Persistência MongoDB
- `redis-data`: Persistência Redis
- `app-logs`: Logs da aplicação

---

## 📊 Fluxo de Dados - Exemplo Completo

### Criar SmartBot (Do Frontend ao MongoDB)

```
1. FRONTEND (User Action)
   ├─ pages/smart-bots/create.vue
   ├─ User preenche wizard (name, symbol, max_position_size, etc.)
   └─ POST /api/smart-bots

2. API GATEWAY (FastAPI)
   ├─ src/api/routes/smart_bots.py
   ├─ @router.post("/")
   ├─ JWT Middleware valida token
   └─ Chama CreateSmartBotUseCase

3. APPLICATION LAYER (Use Case)
   ├─ src/application/use_cases/smart_bots/create_smart_bot.py
   ├─ CreateSmartBotUseCase.execute()
   ├─ Valida limites de plano (FREE=1, PRO=3, MAX=ilimitado)
   ├─ Cria SmartBot entity (Domain)
   └─ Chama repository.create()

4. DOMAIN LAYER (Validation)
   ├─ src/domain/entities/smart_bot.py
   ├─ SmartBot.__post_init__()
   ├─ Valida: name not empty, max_position_size > 0, soft < hard stop-loss
   └─ Retorna entity validado

5. INFRASTRUCTURE LAYER (Persistence)
   ├─ src/infrastructure/repositories/smart_bot_repository_impl.py
   ├─ SmartBotRepositoryImpl.create()
   ├─ Converte SmartBot → MongoDB document
   ├─ await collection.insert_one(doc)
   └─ Retorna SmartBot com _id

6. TASK QUEUE (Celery)
   ├─ Celery Beat detecta novo bot
   ├─ Após 63s, celery-worker-smart executa smart_bot_buy_task
   ├─ src/infrastructure/celery/tasks/smart_bot_tasks.py
   └─ Chama SmartBotPurchaseWorkflow.execute()

7. WORKFLOW (Purchase)
   ├─ src/workflows/smart_bot_purchase.py
   ├─ Redis distributed lock (prevent version conflicts)
   ├─ Verifica bot.can_execute_purchase()
   ├─ ExchangeFactory.create(bot.exchange, user)
   ├─ exchange.get_orderbook(bot.symbol)
   ├─ Calcula competitive_price = best_bid + 0.01
   ├─ exchange.create_order(...)
   ├─ Salva ordem no MongoDB
   └─ Atualiza bot.stats.total_blocked

8. SYNC ORDERS (42s later)
   ├─ sync_orders workflow detecta ordem filled
   ├─ Atualiza bot.stats.total_invested
   └─ bot.stats.total_blocked -= order.amount

9. FRONTEND UPDATE
   ├─ composables/useSmartBots.ts polling GET /api/smart-bots
   ├─ Recebe bot atualizado
   └─ SmartBotCard.vue mostra métricas atualizadas
```

---

## 🗂️ Convenções de Nomenclatura

### Arquivos
- **Entities**: `snake_case.py` (ex: `smart_bot.py`)
- **Classes**: `PascalCase` (ex: `SmartBot`, `CreateSmartBotUseCase`)
- **Funções**: `snake_case` (ex: `can_execute_purchase()`)
- **Constantes**: `UPPER_SNAKE_CASE` (ex: `MAX_RETRIES = 3`)

### MongoDB Collections
- `users` - Usuários e credenciais
- `smart_bots` - SmartBots
- `candle_bots` - CandleBots
- `dca_bots` - DCABots
- `orders` - Ordens de todas estratégias

### Celery Queues
- `sync_orders` - Sincronização de ordens (10 concurrency)
- `smart_buy` - Compras SmartBot
- `smart_sell` - Vendas SmartBot
- `smart_risk` - Monitoramento SmartBot
- `candle_analysis` - Análise técnica CandleBot
- `candle_risk` - Take profit/stop-loss CandleBot
- `dca_execution` - Execução DCA
- `dca_risk` - Monitoramento DCA

---

## 🔍 Como Encontrar Código

### Por Funcionalidade

**"Onde está a lógica de trailing stop?"**
→ `backend/src/domain/entities/smart_bot.py:adjust_trailing_stop()`

**"Onde ordens são sincronizadas?"**
→ `backend/src/workflows/sync_orders.py`

**"Onde Binance é chamada?"**
→ `backend/src/exchanges/binance.py`

**"Onde RSI é calculado?"**
→ `backend/src/domain/services/technical_analysis_service.py`

**"Onde planos FREE/PRO/MAX são validados?"**
→ `backend/src/application/use_cases/contexts/create_context.py`

### Por Tipo de Bot

**SmartBot**:
- Entity: `src/domain/entities/smart_bot.py`
- Workflows: `src/workflows/smart_bot_*.py`
- API: `src/api/routes/smart_bots.py`
- Frontend: `frontend/pages/smart-bots/`, `frontend/composables/useSmartBots.ts`

**CandleBot**:
- Entity: `src/domain/entities/candle_bot.py`
- Workflows: `src/workflows/candle_bot_*.py`
- Services: `src/domain/services/technical_analysis_service.py`
- API: `src/api/routes/candle_bots.py`

**DCABot**:
- Entity: `src/domain/entities/dca_bot.py`
- Workflows: `src/workflows/dca_bot_*.py`
- API: `src/api/routes/dca_bots.py`

---

## 📚 Próximos Passos

Para entender lógica de negócio detalhada:
→ [BUSINESS_LOGIC.md](BUSINESS_LOGIC.md)

Para entender API REST:
→ [API_SPECIFICATION.md](API_SPECIFICATION.md)

Para contribuir com código:
→ [CONTRIBUTING.md](CONTRIBUTING.md)

Para debugar problemas:
→ [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

---

**Última Atualização**: 2024-12-24
**Baseado em**: Análise de código real em `/Users/thiagoabreu/workspace/crypteras-improved`
