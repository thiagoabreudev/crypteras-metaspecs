---
adr_number: "003"
title: "Escolha de MongoDB ao Invés de PostgreSQL"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['technical', 'decision', 'over', 'mongodb', 'postgresql', 'architecture']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# ADR-003: Escolha de MongoDB ao Invés de PostgreSQL

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Manter flexibilidade de schema para iteração rápida em startup, priorizando velocidade de desenvolvimento sobre garantias relacionais.

**Constraints** (limites obrigatórios):
- MongoDB DEVE ser usado para todas collections (users, bots, orders, subscriptions)
- Validações de integridade DEVEM ser feitas na Application Layer (não confiar apenas em schema validation)
- Embedded documents DEVEM ser preferidos para relações 1-para-1 (ex: User.subscription)
- Referências (ObjectId) DEVEM ser usadas para relações 1-para-N (ex: User → Bots)
- Indexes obrigatórios para queries frequentes (user_id, exchange_order_id, status, created_at)
- Transações multi-document APENAS para operações críticas (criar bot + atualizar subscription)
- Motor (async driver) obrigatório para integração com FastAPI

**Non-Goals** (o que NÃO fazer):
- Migrar para PostgreSQL sem justificativa concreta de performance (MongoDB funciona bem para escala atual)
- Usar foreign keys ou joins SQL-style (usar embedded docs ou agregações)
- Criar schemas super normalizados (aceitar alguma denormalização para performance)
- Implementar ORMs pesados como SQLAlchemy ou Tortoise ORM
- Usar MongoDB como cache (Redis é melhor para isso)
- Executar agregações complexas que seriam mais eficientes em SQL (revisar query antes)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Escolha de MongoDB sobre PostgreSQL
:::

## Status
✅ **Aceito** (Implementado desde v2.0)

## Data
2024-09-01 (retroativo - decisão implementada no início do projeto v2.0)

## Contexto

### Problema

O sistema legado (NestJS/TypeScript) usava **MongoDB**, mas a equipe questionou se deveríamos migrar para PostgreSQL na reescrita (v2.0), dado que o sistema tem dados relacionais:

**Estrutura Relacional Evidente**:
```
User (1) ─────── (N) SmartBot ─────── (N) Order
User (1) ─────── (N) CandleBot ────── (N) CandleSignal
User (1) ─────── (N) DCABot ────────  (N) Order
User (1) ─────── (1) Subscription
```

**Argumentos para PostgreSQL**:
- ✅ Relações bem definidas (foreign keys)
- ✅ Transações ACID garantidas
- ✅ Joins eficientes (User + Bots + Orders)
- ✅ Schema rígido previne erros

**Argumentos para MongoDB**:
- ✅ Sistema legado já funcionava com Mongo
- ✅ Schema flexível para rápida iteração
- ✅ Queries simples (sem joins complexos)
- ✅ Embedding reduz lookups

### Motivadores para Decisão

1. **Mudanças Frequentes**: Equipe itera rapidamente em bot configs
   - SmartBot ganhou campos novos (CRY-71: unificação Manual + Traditional)
   - CandleBot ganhou 10 indicadores (CRY-69)
   - DCABot adicionado depois (CRY-63)

2. **Facilidade de Modelagem**: Time prefere documentos JSON-like
   - Entities do Domain mapeiam naturalmente para documentos
   - Sem necessidade de ORMs complexos (SQLAlchemy migrations)

3. **Experiência da Equipe**: Familiaridade com MongoDB
   - Sistema legado operou 2+ anos sem issues de banco
   - Zero conhecimento profundo de PostgreSQL

4. **Queries Simples**: Sistema não faz joins complexos
   - Maior query: "Buscar todos bots ativos de um user" (2 lookups)
   - Sem necessidade de relatórios SQL complexos

## Decisão

Manter **MongoDB 7.0** como banco de dados principal.

### Modelo de Dados Implementado

#### Collections

**users** (Usuários e Credenciais):
```json
{
  "_id": ObjectId("..."),
  "email": "user@example.com",
  "password_hash": "$2b$12$...",
  "encrypted_credentials": {
    "mb_api_id": "encrypted_base64",
    "mb_api_key": "encrypted_base64",
    "binance_api_key": "encrypted_base64",
    "binance_api_secret": "encrypted_base64"
  },
  "subscription": {
    "plan": "PRO",  // FREE | PRO | MAX
    "status": "active",
    "payment_provider": "mercadopago",
    "subscription_id": "preapproval_123",
    "current_period_end": ISODate("2025-01-24")
  },
  "onboarding_completed": false,
  "created_at": ISODate("2024-12-01")
}
```

**smart_bots** (SmartBots - Manual + Traditional):
```json
{
  "_id": ObjectId("..."),
  "user_id": "user_123",  // Referência (não foreign key)
  "name": "BTC Conservative",
  "exchange": "mercado_bitcoin",  // mercado_bitcoin | binance
  "symbol": "BTC-BRL",
  "operation_mode": "CONTINUOUS",  // MANUAL | SCHEDULED | CONTINUOUS
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
    "order_strategy": "LIMIT"  // LIMIT | MARKET
  },
  "stats": {
    "total_invested": 1500.00,
    "total_blocked": 200.00,
    "total_purchases": 15,
    "circuit_breaker_active": false
  },
  "status": "ACTIVE",  // ACTIVE | PAUSED | ARCHIVED
  "version": 3,  // Optimistic lock
  "created_at": ISODate("2024-11-01"),
  "updated_at": ISODate("2024-12-20")
}
```

**candle_bots** (CandleBots - Technical Analysis):
```json
{
  "_id": ObjectId("..."),
  "user_id": "user_123",
  "name": "ETH Swing Trader",
  "exchange": "binance",
  "symbol": "ETHUSDT",
  "config": {
    "timeframe": "15m",  // 1s, 1m, 5m, 15m, 1h, 4h, 1d, etc.
    "lookback_periods": 50,
    "indicators": {
      "rsi_enabled": true,
      "rsi_period": 14,
      "ma_enabled": true,
      "ma_period": 20,
      "bb_enabled": true,
      "bb_period": 20,
      "vwap_enabled": true,
      "mfi_enabled": false
    },
    "take_profit_percentage": 5.0,
    "stop_loss_percentage": 2.0,
    "auto_execute": true  // ou apenas gerar sinais
  },
  "last_signal": {
    "signal": "BUY",  // BUY | SELL | HOLD
    "confidence": "HIGH",  // LOW | MEDIUM | HIGH
    "timestamp": ISODate("2024-12-24T10:30:00Z"),
    "indicators_snapshot": {
      "rsi": 28.5,
      "price": 3500.00,
      "lower_bb": 3450.00
    }
  },
  "status": "ACTIVE",
  "version": 1,
  "created_at": ISODate("2024-11-15")
}
```

**dca_bots** (DCA Bots - Dollar Cost Averaging):
```json
{
  "_id": ObjectId("..."),
  "user_id": "user_123",
  "name": "BTC Weekly DCA",
  "exchange": "mercado_bitcoin",
  "symbol": "BTC-BRL",
  "config": {
    "purchase_amount_brl": 100.00,  // Fixo por execução
    "frequency": "7d",  // 1h | 6h | 24h | 7d | 30d
    "allocated_balance": 5000.00,  // Budget total
    "take_profit_percentage": 20.0,
    "stop_loss_percentage": 10.0,
    "auto_restart_after_exit": true
  },
  "stats": {
    "current_cycle": 12,
    "max_cycles": 52,  // 1 ano de compras semanais
    "total_invested": 1200.00,
    "average_price": 150000.00,  // Preço médio ponderado
    "current_holdings": 0.008,  // BTC acumulado
    "next_execution": ISODate("2024-12-31T09:00:00Z")
  },
  "status": "ACTIVE",
  "version": 2,
  "created_at": ISODate("2024-10-01")
}
```

**orders** (Ordens de Todas Estratégias):
```json
{
  "_id": ObjectId("..."),
  "exchange_order_id": "MB-123456",  // ID na exchange
  "user_id": "user_123",
  "bot_id": "smart_bot_456",  // Referência
  "bot_type": "smart_bot",  // smart_bot | candle_bot | dca_bot
  "exchange": "mercado_bitcoin",
  "symbol": "BTC-BRL",
  "side": "BUY",  // BUY | SELL
  "type": "LIMIT",  // LIMIT | MARKET
  "price": 155000.00,
  "quantity": 0.001,
  "status": "FILLED",  // PENDING | FILLED | CANCELED | FAILED
  "filled_at": ISODate("2024-12-24T10:15:00Z"),
  "strategy_context": {  // CRY-39: Order Audit Context
    "strategy": "smart_bot_purchase",
    "timestamp_before_exchange": ISODate("2024-12-24T10:14:55Z"),
    "parameters": {
      "competitive_price": 155000.00,
      "available_funds": 1000.00
    }
  },
  "created_at": ISODate("2024-12-24T10:14:50Z")
}
```

#### Índices Criados

```python
# users collection
await users.create_index("email", unique=True)

# smart_bots collection
await smart_bots.create_index("user_id")
await smart_bots.create_index([("user_id", 1), ("status", 1)])  # Compound

# candle_bots collection
await candle_bots.create_index("user_id")
await candle_bots.create_index([("user_id", 1), ("status", 1)])

# dca_bots collection
await dca_bots.create_index("user_id")
await dca_bots.create_index([("status", 1), ("stats.next_execution", 1)])

# orders collection
await orders.create_index("user_id")
await orders.create_index("exchange_order_id")
await orders.create_index([("user_id", 1), ("bot_id", 1)])
await orders.create_index([("status", 1), ("created_at", -1)])  # Sync orders
```

### Estratégia de Consistência

#### Sem Transações ACID Multi-Document
MongoDB suporta transações desde v4.0, mas **optamos por NÃO usar** devido a:
- Complexidade adicional
- Overhead de performance
- Casos de uso não exigem

#### Solução: Optimistic Locking
```python
# Repository implementation
async def update(self, bot: SmartBot) -> SmartBot:
    result = await self.collection.update_one(
        {
            "_id": ObjectId(bot.id),
            "version": bot.version  # Optimistic lock
        },
        {
            "$set": {
                "stats.total_invested": float(bot.stats.total_invested),
                "updated_at": datetime.utcnow()
            },
            "$inc": {"version": 1}  # Incrementa versão
        }
    )

    if result.matched_count == 0:
        raise VersionConflictError(f"Bot {bot.id} foi modificado por outro processo")

    bot.version += 1
    return bot
```

#### Distributed Locks (Redis)
Para operações críticas de sync_orders:
```python
from redis import Redis

redis_client = Redis.from_url(settings.CELERY_BROKER_URL)

async def sync_and_update_bot(bot_id: str):
    lock_key = f"lock:bot:{bot_id}"

    with redis_client.lock(lock_key, timeout=30):
        # Garantia de execução única
        orders = await exchange.get_filled_orders()
        bot = await bot_repository.get_by_id(bot_id)
        bot.stats.total_invested += sum(o.amount for o in orders)
        await bot_repository.update(bot)
```

## Consequências

### Positivas ✅

1. **Flexibilidade de Schema**
   - ✅ Adicionar campos sem migrations:
     ```python
     # Antes (CRY-71)
     smart_bots.operation_mode # Não existia

     # Depois (CRY-71)
     smart_bots.operation_mode = "CONTINUOUS"  # Adicionado sem migration
     ```
   - ✅ 10 indicadores adicionados ao CandleBot sem ALTER TABLE

2. **Facilidade de Modelagem**
   - ✅ Entities do Domain mapeiam 1:1 para documentos
   - ✅ Sem necessidade de ORM complexo (SQLAlchemy)
   - ✅ Embedding de configs reduz joins:
     ```json
     {
       "config": {  // Embedded, não tabela separada
         "max_position_size": 5000.00,
         "trailing_stop": 3.0
       }
     }
     ```

3. **Performance de Queries**
   - ✅ Queries simples < 10ms com índices
   - ✅ Sem joins complexos (single collection lookups)
   - ✅ Embedding evita lookups desnecessários

4. **Familiaridade da Equipe**
   - ✅ Time já conhece MongoDB
   - ✅ Zero curva de aprendizado
   - ✅ Sistema legado provou viabilidade

5. **Motor (Async Driver)**
   - ✅ Integração perfeita com FastAPI (asyncio)
   ```python
   from motor.motor_asyncio import AsyncIOMotorClient

   client = AsyncIOMotorClient(settings.MONGODB_URI)
   db = client.crypteras_trading
   bots = await db.smart_bots.find({"user_id": user_id}).to_list(100)
   ```

### Negativas ⚠️

1. **Sem Foreign Keys Nativas**
   - ⚠️ Referências são apenas strings (não validadas pelo banco)
   ```json
   {
     "user_id": "user_123"  // Pode ser ID inválido, banco não valida
   }
   ```
   - **Mitigação**: Validação em Application Layer (use cases)

2. **Sem Joins Nativos**
   - ⚠️ Buscar "User + Bots + Orders" requer múltiplos lookups
   ```python
   # PostgreSQL (single query)
   SELECT u.*, b.*, o.*
   FROM users u
   JOIN smart_bots b ON b.user_id = u.id
   JOIN orders o ON o.bot_id = b.id
   WHERE u.id = $1

   # MongoDB (3 queries)
   user = await db.users.find_one({"_id": user_id})
   bots = await db.smart_bots.find({"user_id": user_id}).to_list()
   orders = await db.orders.find({"user_id": user_id}).to_list()
   ```
   - **Impacto**: Aceitável, queries ainda < 50ms
   - **Alternativa**: Aggregation pipeline (complexo, não usado)

3. **Consistência Eventual (Sem ACID Multi-Doc)**
   - ⚠️ Operações multi-collection podem deixar dados inconsistentes
   - **Exemplo Problemático**:
     ```python
     # Se falhar entre insert order e update bot, bot fica desatualizado
     await orders.insert_one(order_doc)  # Sucesso
     await smart_bots.update_one(...)    # Falha → inconsistência
     ```
   - **Mitigação**: Optimistic locking + distributed locks (Redis)

4. **Versionamento de Schema Manual**
   - ⚠️ Sem Alembic/Flyway (migrations SQL)
   - ⚠️ Campos antigos podem existir em documentos antigos
   - **Exemplo**:
     ```json
     // Documento antigo (pré-CRY-71)
     {
       "strategy_type": "traditional"  // Campo deprecated
     }

     // Documento novo (pós-CRY-71)
     {
       "operation_mode": "CONTINUOUS"  // Campo novo
     }
     ```
   - **Mitigação**: Code handles missing fields gracefully

## Alternativas Consideradas

### Alternativa 1: PostgreSQL (Rejeitada)
**Prós**:
- Foreign keys garantidos
- Transações ACID nativas
- Joins eficientes
- Schema rígido previne bugs

**Contras**:
- ❌ Migrations complexas (Alembic)
- ❌ Mudanças frequentes em schema = muitas migrations
- ❌ Curva de aprendizado para equipe
- ❌ ORM overhead (SQLAlchemy async)

**Razão para Rejeitar**:
- **Mudanças frequentes** tornariam migrations um gargalo
- **Facilidade** de iterar é mais importante que rigidez de schema

### Alternativa 2: Hybrid (PostgreSQL + MongoDB) (Rejeitada)
**Prós**:
- PostgreSQL para dados relacionais (users, subscriptions)
- MongoDB para dados flexíveis (bots, orders)

**Contras**:
- ❌ Complexidade operacional (2 bancos)
- ❌ Sem garantia de consistência cross-database
- ❌ Backup/restore mais complexo

**Razão para Rejeitar**: Over-engineering, MongoDB suficiente

### Alternativa 3: DynamoDB (Rejeitada)
**Prós**:
- Serverless, escalabilidade automática
- Schema flexível como MongoDB

**Contras**:
- Vendor lock-in (AWS)
- Custos imprevisíveis
- Queries limitadas (sem secondary indexes fáceis)

**Razão para Rejeitar**: Preferência por self-hosted (Docker Swarm)

## Implementação

### Evidências no Código

**Motor Client** (`backend/main.py`):
```python
from motor.motor_asyncio import AsyncIOMotorClient

client = AsyncIOMotorClient(settings.MONGODB_URI)
db = client.crypteras_trading
```

**Repositories** (`backend/src/infrastructure/repositories/`):
- `smart_bot_repository_impl.py`: CRUD para SmartBots
- `candle_bot_repository_impl.py`: CRUD para CandleBots
- `dca_bot_repository_impl.py`: CRUD para DCABots
- `user_repository.py`: User + Subscription management

**Optimistic Locking** (evidência em `smart_bot_repository_impl.py`):
```python
result = await self.collection.update_one(
    {"_id": ObjectId(bot.id), "version": bot.version},
    {"$set": {...}, "$inc": {"version": 1}}
)
if result.matched_count == 0:
    raise VersionConflictError()
```

### Configuração Docker

```yaml
services:
  crypteras-mongo:
    image: mongo:7.0
    environment:
      MONGO_INITDB_DATABASE: crypteras_trading
      MONGO_INITDB_ROOT_USERNAME: crypteras_admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
```

## Lições Aprendidas

### O Que Funcionou Bem
1. **Schema Flexível**: Adicionamos 50+ campos em 6 meses sem migrations
2. **Performance**: Queries < 10ms com índices corretos
3. **Motor**: Async driver funciona perfeitamente com FastAPI

### Desafios Enfrentados
1. **Version Conflicts**: Precisou implementar optimistic locking manual
2. **Dados Inconsistentes**: sync_orders às vezes falha, bot fica desatualizado
   - Mitigação: Distributed locks (Redis)
3. **Documentos Antigos**: Campos deprecated existem em docs pré-CRY-71
   - Mitigação: Code trata `None` gracefully

### Quando PostgreSQL Seria Melhor
Se o sistema fosse diferente:
- ❌ Relatórios complexos (JOIN 10+ tabelas)
- ❌ Necessidade de ACID multi-tabelas
- ❌ Schema estável (poucas mudanças)
- ❌ Conformidade regulatória (auditoria rígida)

**Crypteras NÃO tem esses requisitos** → MongoDB é escolha correta

## Métricas de Sucesso

### Performance (Produção)
| Operação | Latência (p95) | Volume/Dia |
|----------|---------------|------------|
| **Get Bots by User** | 8ms | 10,000 |
| **Update Bot Stats** | 12ms | 5,000 |
| **Insert Order** | 6ms | 2,000 |
| **Sync Orders (bulk)** | 150ms | 1,000 |

### Disponibilidade
- ✅ MongoDB uptime: 99.8% (últimos 3 meses)
- ✅ Sem data loss desde v2.0 launch
- ✅ Backups diários automáticos

## Referências

- [MongoDB Manual](https://docs.mongodb.com/manual/)
- [Motor (Async Driver)](https://motor.readthedocs.io/)
- [Código: `user_repository.py`](../../../crypteras-improved/backend/src/infrastructure/repositories/user_repository.py)
- [Docker Compose: MongoDB Service](../../../crypteras-improved/docker-compose.yml)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Implementação**: v2.0 (Setembro 2024)
**Status Atual**: ✅ Em uso, funcionando bem. Schema flexível permite iteração rápida.
