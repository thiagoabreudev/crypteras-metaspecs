---
spec_version: "1.3.0"
valid_from: "2025-12-27"
last_updated: "2025-12-27"
supersedes: "1.2.0"
status: "active"
category: "technical"
tags: ['technical', 'troubleshooting']
---

# Guia de Solução de Problemas - Crypteras Trading System

:::version_info
**Versão**: 1.3.0
**Válida desde**: 2025-12-27
**Status**: Ativa
:::

:::intent
**Goal**: Documentar problemas comuns e soluções para acelerar debugging e reduzir tickets de suporte.

**Constraints** (limites obrigatórios):
- Estruturar por sintoma ("Bot não está comprando")
- Incluir comandos/queries para diagnosticar
- Listar soluções em ordem de probabilidade
- Adicionar novos problemas conforme surgem

**Non-Goals** (o que NÃO fazer):
- Documentar problemas que ocorrem 1x/ano
- Criar troubleshooting exaustivo de infraestrutura
- Substituir logs estruturados e monitoring
:::

:::failure_modes
**Falhas Operacionais Conhecidas**:

1. **Validação de Configuração - Regras Hardcoded vs Parametrizadas**
   - **Tipo**: validation
   - **Descrição**: Validações de configuração de bot usam valores hardcoded ao invés de parâmetros configuráveis
   - **Gatilho**: Criar/atualizar bot com configurações customizadas
   - **Impacto**: 🔴 Crítico (bots rejeitados por regras inflexíveis)
   - **Mitigação**: Todas as validações devem usar parâmetros de config (ex: `min_position_size`, `max_take_profit`) ao invés de valores hardcoded
   - **Detecção**: Bot é criado mas validação falha com mensagem genérica. Buscar `if value < 100:` em validações (hardcoded)
   - **Exemplo**: `if max_position_size < 1000:` ❌ → `if max_position_size < config.min_allowed_position:` ✅

2. **Exchange Credentials - Usando MB Credentials para Binance**
   - **Tipo**: integration
   - **Descrição**: Sistema usa credenciais do Mercado Bitcoin ao tentar conectar na Binance (ou vice-versa)
   - **Gatilho**: Usuário tem credenciais de múltiplas exchanges. Bot configurado para exchange diferente
   - **Impacto**: 🔴 Crítico (autenticação falha, ordens não executam)
   - **Mitigação**: `ExchangeFactory` DEVE validar exchange antes de criar cliente. Usar `user.encrypted_credentials['mb_api_key']` APENAS se `bot.exchange == "mercado_bitcoin"`
   - **Detecção**: Erro 401/403 da exchange. Logs mostram "Invalid API key" mas credenciais existem no banco
   - **Exemplo Correto**:
     ```python
     if bot.exchange == "mercado_bitcoin":
         api_key = decrypt(user.encrypted_credentials['mb_api_key'])
     elif bot.exchange == "binance":
         api_key = decrypt(user.encrypted_credentials['binance_api_key'])
     ```

3. **Cálculo de Fundos Disponíveis Incorreto**
   - **Tipo**: validation
   - **Descrição**: `get_available_funds()` retorna valor incorreto devido a `total_invested` ou `total_blocked` desatualizado
   - **Gatilho**: sync_orders falha ou order status não sincronizado corretamente
   - **Impacto**: 🟡 Médio (bot para de comprar quando ainda tem fundos OU tenta comprar sem fundos)
   - **Mitigação**: Forçar re-sync manual com `python backend/scripts/force_sync_orders.py --user-id USER_ID`. Adicionar validação de sanidade (fundos nunca negativos)
   - **Detecção**: Dashboard mostra fundos diferentes do esperado. `available = max_position_size - total_invested - total_blocked` resulta em valor absurdo
   - **Debug**: Comparar `total_invested` no MongoDB com soma de ordens FILLED no banco

4. **Stop-Loss Não Dispara Quando Deveria**
   - **Tipo**: validation
   - **Descrição**: `should_trigger_stop_loss()` retorna False mesmo com prejuízo >= threshold
   - **Gatilho**: Preço cai mas bot não vende
   - **Impacto**: 🔴 Crítico (usuário perde mais dinheiro do que configurado)
   - **Mitigação**: Verificar se `base_price` está atualizado. Validar cálculo de variação. Adicionar logs detalhados no workflow `smart_bot_check_situation`
   - **Detecção**: Preço atual está 5%+ abaixo de base_price mas bot continua ACTIVE
   - **Debug**:
     ```python
     base_price = bot.stats.base_price  # Ex: 100000
     current_price = exchange.get_ticker(bot.symbol)['last']  # Ex: 94000
     variation = ((current_price - base_price) / base_price) * 100  # -6%
     # Deveria disparar hard stop-loss (-5%)
     ```

5. **Trailing Stop Não Ajusta Base Price**
   - **Tipo**: validation
   - **Descrição**: `adjust_trailing_stop()` não atualiza `base_price` mesmo com lucro >= threshold
   - **Gatilho**: Preço sobe mas base_price não ajusta
   - **Impacto**: 🟡 Médio (oportunidade de proteger lucro perdida)
   - **Mitigação**: Verificar se workflow `smart_bot_check_situation` está chamando `adjust_trailing_stop()`. Validar `trailing_stop_percentage` configurado
   - **Detecção**: Preço subiu 5%+ mas `base_price` continua igual. `trailing_stop_adjusted_at` não atualiza
   - **Debug**: Adicionar log em `adjust_trailing_stop()` mostrando `current_price`, `base_price`, `variation`

6. **MongoDB Debug - IA Conecta em Localhost ao Invés de Atlas**
   - **Tipo**: integration
   - **Descrição**: IA tenta debugar dados conectando em `mongodb://localhost:27017` ao invés de ler `MONGODB_URI` do ambiente (MongoDB Atlas)
   - **Gatilho**: Debug de collections, verificação de dados, queries manuais
   - **Impacto**: 🔴 Crítico (debug falha completamente, IA não vê dados reais, decisões baseadas em estado vazio/incorreto)
   - **Mitigação**: SEMPRE verificar variável de ambiente PRIMEIRO. Ler `MONGODB_URI` de `.env` ou `backend/.env.prod`. NUNCA assumir localhost
   - **Detecção**: Erro "Connection refused localhost:27017" mas produção usa Atlas. IA reporta "collection vazia" mas dados existem
   - **Solução**:
     ```bash
     # 1. Verificar URI correto
     cat backend/.env.prod | grep MONGODB_URI
     # mongodb+srv://admin:***@crypteras.4etwcbo.mongodb.net/crypteras_trading...

     # 2. Conectar com URI correto
     python
     >>> import os
     >>> from motor.motor_asyncio import AsyncIOMotorClient
     >>> mongo_uri = os.getenv('MONGODB_URI') or 'mongodb+srv://...'
     >>> client = AsyncIOMotorClient(mongo_uri)
     >>> db = client.crypteras_trading
     >>> await db.list_collection_names()
     ```

7. **MongoDB Collections - Queries Falham por Nome Incorreto**
   - **Tipo**: hallucination
   - **Descrição**: IA assume nomes de collections (`db.smart_bots`, `db.trading_orders`) sem verificar se existem
   - **Gatilho**: Queries, agregações, debug de dados
   - **Impacto**: 🟡 Médio (queries retornam vazio, erros "collection not found", debug incorreto)
   - **Mitigação**: SEMPRE listar collections primeiro: `await db.list_collection_names()`. Validar antes de usar
   - **Detecção**: Query retorna vazio inesperadamente. Erro "collection 'xyz' does not exist"
   - **Solução**:
     ```bash
     # 1. Listar collections existentes
     python backend/scripts/list_collections.py
     # ou
     docker exec -it crypteras-mongo mongosh "mongodb+srv://..." --eval "db.getCollectionNames()"

     # 2. Usar collection correta em queries
     collections = await db.list_collection_names()
     print(f"Collections disponíveis: {collections}")
     # ['users', 'smart_bots', 'candle_bots', 'orders', ...]

     if 'smart_bots' in collections:
         bots = await db.smart_bots.find().to_list(10)
     ```
:::

:::breaking_changes
**v1.3.0** (2025-12-27):
- Adicionados 2 novos failure modes operacionais (#6-#7)
- FM#6: MongoDB Debug (IA conecta em localhost ao invés de Atlas)
- FM#7: MongoDB Collections (queries falham por nome incorreto)
- Total: 7 failure modes operacionais documentados
- Incluídas soluções práticas com comandos bash/python
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.2.0** (2025-12-25):
- Adicionada seção `:::failure_modes` com 5 falhas operacionais conhecidas
- Documentadas falhas reais: Validações hardcoded, Exchange credentials erradas, Fundos disponíveis incorretos, Stop-loss não dispara, Trailing stop não ajusta
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Guia de troubleshooting e problemas comuns
:::

Este documento lista problemas comuns em produção e suas soluções baseadas em experiência real.

---

## 🚨 Problemas Críticos

### 1. sync_orders Falhando (~8% Taxa de Falha)

**Sintomas**:
- Ordens aparecem como filled na exchange, mas não no dashboard
- `bot.stats.total_invested` desatualizado
- `bot.stats.total_blocked` incorreto
- Fundos disponíveis calculados errado

**Causas Raiz**:
1. **Ordens não sincronizadas corretamente da exchange**
   - Timeout na API da exchange (> 10s)
   - Rate limiting da exchange
   - Formato de resposta mudou (exchange atualizou API)

2. **Atualização de dados dos bots falhando**
   - Version conflicts (optimistic locking)
   - MongoDB connection timeout
   - Race condition entre workflows

**Como Debugar**:

```bash
# 1. Ver logs do sync_orders
docker logs crypteras-celery-worker-sync --tail 100 -f | grep "sync_orders"

# 2. Verificar tasks failed no Flower
# Abrir http://localhost:5555
# Ir em "Tasks" → filtrar por "sync_orders" → ver failed tasks

# 3. Verificar MongoDB diretamente
docker exec -it crypteras-mongo mongosh
use crypteras_trading
db.orders.find({"exchange_order_id": "MB-123456"})  # Verificar se ordem existe

# 4. Verificar bot stats
db.smart_bots.findOne({"_id": ObjectId("...")})
# Verificar total_invested, total_blocked
```

**Soluções**:

**Solução 1: Aumentar Timeout**
```python
# backend/src/exchanges/mercado_bitcoin.py
self.client.timeout = 30  # Aumentar de 10s para 30s
```

**Solução 2: Aumentar Retry**
```python
# backend/src/infrastructure/celery/celeryconfig.py
task_default_retry_delay = 120  # Aumentar de 60s para 120s
task_max_retries = 5  # Aumentar de 3 para 5
```

**Solução 3: Forçar Re-Sync Manual**
```bash
# Script para forçar sync de ordens específicas
python backend/scripts/force_sync_orders.py --user-id USER_ID --hours 24
```

**Solução 4: Adicionar Distributed Lock Mais Robusto**
```python
# backend/src/workflows/sync_orders.py
from redis import Redis

redis_client = Redis.from_url(settings.CELERY_BROKER_URL)

async def sync_orders_for_bot(bot_id: str):
    lock_key = f"lock:sync_orders:{bot_id}"

    # Timeout maior para evitar deadlocks
    with redis_client.lock(lock_key, timeout=60):  # Era 30s, agora 60s
        # Sync orders
        ...
```

**Prevenção**:
```python
# Adicionar health check específico para sync_orders
# backend/main.py

@app.get("/health/sync_orders")
async def sync_orders_health():
    # Verificar se sync_orders rodou nos últimos 2 minutos
    last_run = await get_last_sync_orders_run()
    if datetime.utcnow() - last_run > timedelta(minutes=2):
        return {"status": "unhealthy", "reason": "sync_orders not running"}

    return {"status": "healthy"}
```

---

### 2. Version Conflicts no MongoDB

**Sintomas**:
```
VersionConflictError: Bot 123abc foi modificado por outro processo
```

**Causa**:
- Múltiplos Celery workers tentando atualizar mesmo bot simultaneamente
- Workflow `smart_bot_purchase` e `sync_orders` rodando ao mesmo tempo

**Como Debugar**:
```bash
# Ver quantos workers estão ativos
docker ps | grep celery-worker

# Ver concurrency de cada worker
docker logs crypteras-celery-worker-sync | grep "concurrency"
docker logs crypteras-celery-worker-smart | grep "concurrency"
```

**Soluções**:

**Solução 1: Usar Distributed Lock (Redis)**
```python
# SEMPRE usar lock ao atualizar bot
from redis import Redis

redis_client = Redis.from_url(settings.CELERY_BROKER_URL)

async def update_bot_safely(bot_id: str):
    lock_key = f"lock:smart_bot:{bot_id}"

    with redis_client.lock(lock_key, timeout=30):
        bot = await bot_repository.get_by_id(bot_id)
        bot.stats.total_invested += 100
        await bot_repository.update(bot)  # Seguro
```

**Solução 2: Retry com Backoff**
```python
# backend/src/infrastructure/repositories/smart_bot_repository_impl.py

async def update(self, bot: SmartBot) -> SmartBot:
    max_retries = 3
    for attempt in range(max_retries):
        try:
            result = await self.collection.update_one(
                {"_id": ObjectId(bot.id), "version": bot.version},
                {"$set": {...}, "$inc": {"version": 1}}
            )

            if result.matched_count == 0:
                if attempt < max_retries - 1:
                    await asyncio.sleep(2 ** attempt)  # Exponential backoff
                    bot = await self.get_by_id(bot.id)  # Refetch
                    continue
                else:
                    raise VersionConflictError()

            bot.version += 1
            return bot

        except Exception as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)
```

**Solução 3: Reduzir Concurrency**
```yaml
# docker-compose.yml
celery-worker-smart:
  command: celery -A src.infrastructure.celery.celery_app worker --queues=smart_buy,smart_sell,smart_risk --concurrency=3  # Era 5, agora 3
```

---

### 3. Binance - Workflows Assumem Mercado Bitcoin

**Sintomas**:
- Bots com `exchange="binance"` falham ao executar ordens
- Erros: `KeyError: 'data'`, `AttributeError: 'NoneType' object has no attribute 'bids'`
- Normalização de símbolos incorreta (`BTC-BRL` vs `BTCUSDT`)

**Causa**:
- Código acoplado ao formato de resposta do Mercado Bitcoin
- Workflows com `if bot.exchange == "mercado_bitcoin"` hardcoded

**Como Debugar**:
```bash
# 1. Buscar código acoplado ao MB
grep -r "mercado_bitcoin" backend/src/workflows/
grep -r "['data']" backend/src/workflows/  # MB retorna {"data": {...}}

# 2. Verificar se exchange factory está funcionando
python -c "
from src.infrastructure.exchanges.exchange_factory import ExchangeFactory
exchange = ExchangeFactory.create('binance', user)
print(exchange.__class__.__name__)  # Deve ser 'BinanceExchange'
"
```

**Soluções**:

**Solução 1: Remover Hardcoded Exchange Checks**
```python
# ❌ ERRADO (workflow acoplado)
if bot.exchange == "mercado_bitcoin":
    orderbook = await mb_exchange.get_orderbook(symbol)
    best_bid = Decimal(orderbook['data']['bids'][0][0])  # Formato MB

elif bot.exchange == "binance":
    orderbook = await binance_exchange.get_order_book(symbol)
    best_bid = Decimal(orderbook['bids'][0][0])  # Formato Binance

# ✅ CORRETO (exchange-agnostic)
exchange: BaseExchange = await ExchangeFactory.create(bot.exchange, user)
orderbook = await exchange.get_orderbook(bot.symbol)  # Formato normalizado
best_bid = orderbook['bids'][0]['price']  # Funciona para MB e Binance
```

**Solução 2: Melhorar OrderNormalizer**
```python
# backend/src/infrastructure/exchanges/order_normalizer.py

class OrderNormalizer:
    @staticmethod
    def normalize(raw_order: Dict, exchange: str) -> Dict:
        if exchange == "mercado_bitcoin":
            return {
                "exchange_order_id": raw_order['data']['id'],
                "price": Decimal(raw_order['data']['price']),  # String
                "quantity": Decimal(raw_order['data']['quantity']),
                "status": raw_order['data']['status'].upper()
            }

        elif exchange == "binance":
            return {
                "exchange_order_id": str(raw_order['orderId']),
                "price": Decimal(str(raw_order['price'])),  # Float → String → Decimal
                "quantity": Decimal(str(raw_order['executedQty'])),
                "status": raw_order['status']  # Já uppercase
            }

        else:
            raise UnsupportedExchangeError(f"Exchange {exchange} não suportada")
```

**Solução 3: Testes com Mock Exchanges**
```python
# tests/integration/workflows/test_smart_bot_purchase_binance.py

@pytest.mark.asyncio
async def test_purchase_workflow_with_binance():
    # Mock Binance exchange
    mock_binance = MockBinanceExchange(orderbook={
        'bids': [{'price': Decimal("31000"), 'quantity': Decimal("1")}],
        'asks': [{'price': Decimal("31100"), 'quantity': Decimal("1")}]
    })

    bot = SmartBot(exchange="binance", symbol="BTCUSDT", ...)

    workflow = SmartBotPurchaseWorkflow(
        exchange_factory=MockExchangeFactory(mock_binance)
    )

    result = await workflow.execute(bot, user)

    assert result.success == True
    assert result.order.price == Decimal("31000.01")  # best_bid + 0.01
```

---

## ⚠️ Problemas Comuns

### 4. Celery Worker Não Inicia

**Sintomas**:
```bash
docker logs crypteras-celery-worker-sync
# Erro: ImportError: cannot import name 'celery_app'
```

**Causas**:
1. `celery_app.py` não encontrado
2. Import circular
3. Dependências faltando no container

**Soluções**:

```bash
# 1. Verificar se arquivo existe
docker exec -it crypteras-backend ls -la /app/backend/src/infrastructure/celery/celery_app.py

# 2. Verificar imports
docker exec -it crypteras-backend python -c "from src.infrastructure.celery.celery_app import celery_app; print(celery_app)"

# 3. Rebuild container
docker-compose build backend celery-worker-sync celery-worker-smart
docker-compose up -d
```

---

### 5. MongoDB Connection Timeout

**Sintomas**:
```
ServerSelectionTimeoutError: localhost:27017: [Errno 111] Connection refused
```

**Causas**:
1. MongoDB não iniciou ainda (healthcheck falhou)
2. Credenciais erradas em `MONGODB_URI`
3. Porta 27017 bloqueada por firewall

**Soluções**:

```bash
# 1. Verificar se MongoDB está rodando
docker ps | grep crypteras-mongo

# 2. Verificar healthcheck
docker inspect crypteras-mongo | grep Health -A 10

# 3. Testar conexão manualmente
docker exec -it crypteras-mongo mongosh -u crypteras_admin -p ${MONGO_ROOT_PASSWORD}

# 4. Verificar variáveis de ambiente
docker exec -it crypteras-backend env | grep MONGODB_URI
```

---

### 6. Redis Lock Timeout

**Sintomas**:
```
LockError: Unable to acquire lock on lock:smart_bot:123abc after 10 seconds
```

**Causas**:
1. Workflow travou e não liberou lock
2. Lock timeout muito baixo
3. Deadlock entre workflows

**Soluções**:

```bash
# 1. Verificar locks ativos no Redis
docker exec -it crypteras-redis redis-cli -a ${REDIS_PASSWORD}
KEYS lock:*

# 2. Forçar liberação de lock (último recurso)
DEL lock:smart_bot:123abc

# 3. Ver quem está segurando o lock
TTL lock:smart_bot:123abc  # Tempo restante
```

**Prevenção**:
```python
# Aumentar timeout
with redis_client.lock(lock_key, timeout=60):  # Era 30s, agora 60s
    ...

# Adicionar auto-expire em caso de crash
with redis_client.lock(lock_key, timeout=60, blocking_timeout=10):
    # Se não conseguir lock em 10s, lança exceção ao invés de esperar infinito
    ...
```

---

### 7. Frontend Não Conecta ao Backend

**Sintomas**:
- Dashboard mostra "Erro ao carregar bots"
- Console do navegador: `ERR_CONNECTION_REFUSED`

**Causas**:
1. Backend não iniciou (porta 7777 ou 8000)
2. CORS bloqueando
3. URL hardcoded errado no frontend

**Soluções**:

```bash
# 1. Verificar se backend está rodando
curl http://localhost:7777/playground/status
curl http://localhost:8000/api/dashboard/health

# 2. Verificar logs do backend
docker logs crypteras-backend --tail 50 -f

# 3. Verificar CORS
# backend/main.py
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Frontend
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Verificar URLs no Frontend**:
```typescript
// frontend/nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    public: {
      dashboardApiBase: 'http://localhost:8000',  # NÃO hardcodar!
      tradingApiBase: 'http://localhost:7777'
    }
  }
})
```

---

### 8. Flower Dashboard Não Carrega

**Sintomas**:
- http://localhost:5555 não abre
- `This site can't be reached`

**Causas**:
1. Flower não iniciou
2. Senha errada (basic auth)
3. Porta 5555 já em uso

**Soluções**:

```bash
# 1. Verificar se Flower está rodando
docker ps | grep flower
docker logs crypteras-flower --tail 50

# 2. Verificar porta
netstat -an | grep 5555  # Linux/Mac
# ou
docker port crypteras-flower

# 3. Acessar com credenciais
# Usuário: admin (default)
# Senha: ${FLOWER_PASSWORD} (.env)
```

---

## 🔧 Comandos Úteis de Debug

### Logs

```bash
# Ver logs de todos serviços
docker-compose logs -f

# Ver logs de serviço específico
docker logs crypteras-backend -f --tail 100
docker logs crypteras-celery-worker-sync -f
docker logs crypteras-celery-beat -f

# Grep em logs
docker logs crypteras-backend 2>&1 | grep "ERROR"
docker logs crypteras-celery-worker-smart 2>&1 | grep "sync_orders"
```

### MongoDB

```bash
# Conectar no MongoDB
docker exec -it crypteras-mongo mongosh -u crypteras_admin -p ${MONGO_ROOT_PASSWORD}

use crypteras_trading

# Ver collections
show collections

# Contar documentos
db.smart_bots.countDocuments()
db.orders.countDocuments()

# Buscar bot específico
db.smart_bots.findOne({"_id": ObjectId("...")})

# Buscar ordens pending
db.orders.find({"status": "PENDING"}).count()

# Verificar índices
db.smart_bots.getIndexes()
```

### Redis

```bash
# Conectar no Redis
docker exec -it crypteras-redis redis-cli -a ${REDIS_PASSWORD}

# Ver keys
KEYS *

# Ver locks ativos
KEYS lock:*

# Ver tasks pendentes
LLEN celery  # Queue default
LLEN sync_orders  # Queue específica

# Ver info do Redis
INFO memory
INFO stats
```

### Celery

```bash
# Ver workers ativos
celery -A src.infrastructure.celery.celery_app inspect active

# Ver tasks agendadas
celery -A src.infrastructure.celery.celery_app inspect scheduled

# Ver stats de workers
celery -A src.infrastructure.celery.celery_app inspect stats

# Forçar execução de task
celery -A src.infrastructure.celery.celery_app call src.infrastructure.celery.tasks.sync_orders_task
```

---

## 📊 Monitoramento Proativo

### Grafana Queries

```bash
# Logs de errors (últimas 24h)
{container="crypteras-backend"} |= "ERROR"

# Logs de sync_orders failures
{container="crypteras-celery-worker-sync"} |= "sync_orders" |= "FAILED"

# Version conflicts
{container="crypteras-celery-worker-smart"} |= "VersionConflictError"
```

### Health Checks

```bash
# Script para verificar saúde do sistema
#!/bin/bash

# Backend
curl -f http://localhost:7777/playground/status || echo "Backend DOWN"

# MongoDB
docker exec crypteras-mongo mongosh --eval "db.adminCommand('ping')" || echo "MongoDB DOWN"

# Redis
docker exec crypteras-redis redis-cli -a ${REDIS_PASSWORD} ping || echo "Redis DOWN"

# Celery workers
docker ps | grep celery-worker-sync | grep Up || echo "Celery sync worker DOWN"
```

---

## 📚 Referências

**ADRs Relacionados**:
- [ADR-002: Celery + Redis](adr/002-celery-redis-migration.md) - Troubleshooting Celery
- [ADR-003: MongoDB](adr/003-mongodb-over-postgresql.md) - Version conflicts
- [ADR-005: Adapter Pattern](adr/005-adapter-pattern-exchanges.md) - Problemas Binance

**Código**:
- [sync_orders.py](../../crypteras-improved/backend/src/workflows/sync_orders.py)
- [smart_bot_repository_impl.py](../../crypteras-improved/backend/src/infrastructure/repositories/smart_bot_repository_impl.py)

---

**Última Atualização**: 2024-12-24
**Baseado em**: Problemas reais em produção
