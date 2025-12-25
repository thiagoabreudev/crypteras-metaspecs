---
adr_number: "002"
title: "Migração de APScheduler para Celery + Redis"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['technical', 'decision', 'redis', 'celery', 'migration', 'architecture']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# ADR-002: Migração de APScheduler para Celery + Redis

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Permitir escalabilidade horizontal de workflows distribuídos para suportar crescimento de usuários.

**Constraints** (limites obrigatórios):
- Celery DEVE ser usado para todos workflows periódicos (nunca APScheduler ou threading)
- Redis DEVE ser broker e result backend (nunca RabbitMQ ou banco de dados)
- Celery Beat DEVE ter apenas 1 instância (evitar duplicação de tasks)
- Celery Workers PODEM escalar horizontalmente (múltiplas instâncias)
- Tasks DEVEM ser idempotentes (execução múltipla do mesmo task não deve corromper dados)
- Distributed locks DEVEM ser usados para operações críticas (Redis lock com timeout)
- Task retry com exponential backoff obrigatório (max 3 retries)

**Non-Goals** (o que NÃO fazer):
- Usar RabbitMQ ou Amazon SQS como broker (Redis é suficiente para escala atual)
- Usar PostgreSQL como result backend (Redis é mais performático)
- Permitir múltiplas instâncias de Celery Beat (causaria tarefas duplicadas)
- Criar workflows síncronos que bloqueiam requests (tudo deve ser async via Celery)
- Implementar priority queues complexas no MVP (single queue é suficiente)
- Usar Celery Canvas (chain, chord, group) sem necessidade real (manter simples)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Migração para Celery + Redis
:::

## Status
✅ **Aceito** (Implementado em CRY-72)

## Data
2024-10-15 (CRY-72: Migração de APScheduler → Celery)

## Contexto

### Problema com APScheduler

O sistema inicialmente usava **APScheduler** para executar workflows automatizados:
```python
# Código antigo (v2.0-v2.7)
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()

# Workflows executados em single process
scheduler.add_job(sync_orders, 'interval', seconds=42)
scheduler.add_job(smart_bot_purchase, 'interval', seconds=63)
scheduler.add_job(smart_bot_sales, 'interval', seconds=67)
```

**Limitações Encontradas**:
1. **Não Escalável Horizontalmente**:
   - APScheduler roda em **single process**
   - Não permite múltiplos workers processando tasks
   - Gargalo conforme usuários crescem

2. **Sem Distribuição de Carga**:
   - Todos workflows rodam no mesmo processo
   - CPU-bound tasks bloqueiam outras tasks
   - Impossível dedicar workers para queues específicas

3. **Falta de Observabilidade**:
   - Sem dashboard para monitorar tasks
   - Difícil debugar falhas
   - Sem retry automático robusto

4. **Single Point of Failure**:
   - Se processo cair, todos workflows param
   - Sem redundância

### Motivadores para Mudança

1. **Planos de Crescimento**: Meta de 100+ usuários em Q2 2025
2. **Performance**: Workflows demorando muito (sync_orders bloqueando outros)
3. **Confiabilidade**: Necessidade de retry automático e error handling
4. **Observabilidade**: Equipe precisa monitorar tasks em produção (Grafana)

## Decisão

Migrar para **Celery + Redis** com arquitetura distribuída:

### Arquitetura Implementada

```
┌─────────────────────────────────────────────────────────┐
│                    Redis (Port 6379)                     │
│  - Broker: Recebe tasks de FastAPI/Beat                 │
│  - Backend: Armazena resultados de tasks                │
│  - Lock: Distributed locks (version conflicts)          │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│              Celery Beat (RedBeat Scheduler)             │
│  - Agenda tasks baseado em intervals co-primos          │
│  - Persiste schedule no Redis (não shelve DBM)          │
│  - Tasks: sync_orders, smart_buy, smart_sell, etc.      │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│                   Celery Workers                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Worker: sync (10 concurrency)                    │  │
│  │  Queue: sync_orders                               │  │
│  │  Tasks: Sincronização de ordens das exchanges     │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Worker: smart (5 concurrency)                    │  │
│  │  Queues: smart_buy, smart_sell, smart_risk,       │  │
│  │          candle_analysis, candle_risk,            │  │
│  │          dca_execution, dca_risk                  │  │
│  │  Tasks: SmartBot, CandleBot, DCABot workflows     │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│              Flower Dashboard (Port 5555)                │
│  - Monitoring em tempo real                             │
│  - Task success/failure rates                           │
│  - Worker status e concurrency                          │
└─────────────────────────────────────────────────────────┘
```

### Configuração Implementada

**Celery App** (`src/infrastructure/celery/celery_app.py`):
```python
from celery import Celery

celery_app = Celery('crypteras')
celery_app.config_from_object('src.infrastructure.celery.celeryconfig')
celery_app.autodiscover_tasks([
    'src.infrastructure.celery.tasks'
])
```

**Celery Config** (`src/infrastructure/celery/celeryconfig.py`):
```python
# Broker e Backend
broker_url = os.getenv('CELERY_BROKER_URL')  # redis://:password@redis:6379/0
result_backend = os.getenv('CELERY_RESULT_BACKEND')  # redis://:password@redis:6379/1

# Queues Dedicadas
task_routes = {
    'sync_orders': {'queue': 'sync_orders'},
    'smart_bot_buy': {'queue': 'smart_buy'},
    'smart_bot_sell': {'queue': 'smart_sell'},
    'smart_bot_risk': {'queue': 'smart_risk'},
    # ...
}

# RedBeat Schedule (intervalos co-primos)
beat_scheduler = 'redbeat.RedBeatScheduler'
beat_schedule = {
    'sync-orders': {
        'task': 'sync_orders',
        'schedule': 42.0,  # co-prime intervals
    },
    'smart-buy': {
        'task': 'smart_bot_buy',
        'schedule': 63.0,
    },
    'smart-sell': {
        'task': 'smart_bot_sell',
        'schedule': 67.0,
    },
    'smart-risk': {
        'task': 'smart_bot_risk',
        'schedule': 33.0,
    },
    # ...
}

# Configurações de Retry
task_acks_late = True
task_reject_on_worker_lost = True
task_default_retry_delay = 60
task_max_retries = 3
```

**Docker Compose** (8 serviços orquestrados):
```yaml
services:
  redis:
    image: redis:7.2-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}

  celery-beat:
    command: celery -A src.infrastructure.celery.celery_app beat --loglevel=info

  celery-worker-sync:
    command: celery -A src.infrastructure.celery.celery_app worker --queues=sync_orders --concurrency=10

  celery-worker-smart:
    command: celery -A src.infrastructure.celery.celery_app worker --queues=smart_buy,smart_sell,smart_risk,candle_analysis,candle_risk,dca_execution,dca_risk --concurrency=5

  flower:
    command: celery -A src.infrastructure.celery.celery_app flower --port=5555
```

### Migração de Tasks

**Antes (APScheduler)**:
```python
@scheduler.scheduled_job('interval', seconds=42)
async def sync_orders_job():
    await sync_orders_workflow.execute()
```

**Depois (Celery)**:
```python
@celery_app.task(name='sync_orders', bind=True, max_retries=3)
def sync_orders_task(self):
    try:
        # Celery é sync, workflow é async → bridge
        asyncio.run(sync_orders_workflow.execute())
    except Exception as exc:
        # Retry automático com backoff
        raise self.retry(exc=exc, countdown=60)
```

## Consequências

### Positivas ✅

1. **Escalabilidade Horizontal Alcançada**
   - ✅ Múltiplos workers processando tasks em paralelo
   - ✅ Concurrency configurável por worker (10 sync, 5 smart)
   - ✅ Fácil adicionar workers: `docker-compose scale celery-worker-smart=3`

2. **Isolamento de Workflows**
   - ✅ Queues dedicadas: `sync_orders` não bloqueia `smart_buy`
   - ✅ `sync_orders` (10 concurrency) processa mais rápido
   - ✅ `smart_*` (5 concurrency) evita sobrecarga de API

3. **Observabilidade Completa**
   - ✅ **Flower Dashboard** (http://localhost:5555):
     - Task success/failure rates
     - Worker status e throughput
     - Queue lengths em tempo real
   - ✅ **Logs Estruturados** (Grafana):
     - Erros categorizados por task
     - Performance tracking

4. **Retry Automático Robusto**
   - ✅ `task_max_retries=3` com backoff exponencial
   - ✅ Dead Letter Queue para tasks permanentemente falhas
   - ✅ `task_acks_late=True`: Worker crash não perde task

5. **Persistência de Schedule**
   - ✅ **RedBeat** persiste schedule no Redis (não DBM shelve)
   - ✅ Restart de `celery-beat` não perde schedules
   - ✅ Modificar schedules dinamicamente (via Redis)

6. **Distributed Locks**
   - ✅ Redis usado para locks (evitar version conflicts)
   - ✅ Workflow de compra/venda não concorrem no mesmo bot
   ```python
   from redis import Redis
   redis_client = Redis.from_url(settings.CELERY_BROKER_URL)
   lock_key = f"lock:smart_bot:{bot_id}"

   with redis_client.lock(lock_key, timeout=30):
       # Atualização segura do bot
       bot = await repository.get_by_id(bot_id)
       bot.total_invested += order.amount
       await repository.update(bot)
   ```

### Negativas ⚠️

1. **Complexidade Adicional**
   - ⚠️ 8 serviços Docker vs 1 processo Python
   - ⚠️ Redis como dependency crítica (single point of failure)
   - **Mitigação**: Docker Compose facilita orquestração

2. **Async → Sync Bridge**
   - ⚠️ Workflows são `async`, Celery tasks são `sync`
   - ⚠️ Necessário `asyncio.run()` em cada task:
   ```python
   @celery_app.task
   def sync_orders_task(self):
       asyncio.run(sync_orders_workflow.execute())  # bridge
   ```
   - **Impacto**: Overhead mínimo, funciona bem
   - **Alternativa Considerada**: Celery async mode (experimental, rejeitada)

3. **Learning Curve**
   - ⚠️ Equipe precisa entender Celery concepts (broker, worker, beat)
   - ⚠️ Debugging tasks é diferente de debugging sync code
   - **Mitigação**: Flower dashboard ajuda, documentação clara

4. **Custos de Infraestrutura**
   - ⚠️ Redis + múltiplos workers = mais RAM/CPU
   - **Baseline**: ~512MB RAM para Redis + 2 workers
   - **Projeção**: ~2GB RAM para 100+ usuários
   - **Justificativa**: Escalabilidade vale o custo

## Alternativas Consideradas

### Alternativa 1: Continuar com APScheduler (Rejeitada)
**Prós**:
- Simplicidade
- Zero dependências externas

**Contras**:
- ❌ **Não escalável horizontalmente** (bloqueador para crescimento)
- ❌ Sem isolamento de workflows
- ❌ Sem retry robusto

**Razão para Rejeitar**: Incompatível com meta de 100+ usuários

### Alternativa 2: AWS Lambda + EventBridge (Considerada)
**Prós**:
- Serverless (sem gerenciar workers)
- Escalabilidade automática
- Pay-per-use

**Contras**:
- Vendor lock-in (AWS)
- Cold starts (latência imprevisível)
- Custos difíceis de prever
- Equipe sem experiência em AWS

**Razão para Rejeitar**: Preferência por controle total (Docker Swarm + DigitalOcean)

### Alternativa 3: Celery com RabbitMQ ao invés de Redis (Rejeitada)
**Prós**:
- RabbitMQ é broker "puro" (mais robusto que Redis)
- Suporta features avançadas (dead letter, priority queues)

**Contras**:
- Mais um serviço para gerenciar
- Redis já usado para distributed locks
- RabbitMQ mais pesado (RAM)

**Razão para Rejeitar**: Redis suficiente + simplicidade

### Alternativa 4: Apache Airflow (Rejeitada)
**Prós**:
- UI visual para DAGs
- Retry e monitoring built-in

**Contras**:
- Overkill para workflows simples
- Complexo de configurar
- Footprint grande (5+ serviços)

**Razão para Rejeitar**: Celery suficiente para use case

## Implementação

### Evidências no Código

**Celery Config** (`backend/src/infrastructure/celery/celeryconfig.py`):
- 189 lines de configuração
- 8 queues definidas
- RedBeat schedule com intervalos co-primos

**Tasks** (`backend/src/infrastructure/celery/tasks/`):
```
tasks/
├── smart_bot_tasks.py (buy, sell, risk)
├── candle_bot_tasks.py (analysis, risk)
├── dca_bot_tasks.py (execution, risk)
└── sync_orders_task.py
```

**Docker Compose** (`docker-compose.yml`):
- `redis` service (port 6379)
- `celery-beat` (scheduler)
- `celery-worker-sync` (10 concurrency)
- `celery-worker-smart` (5 concurrency)
- `flower` (port 5555)

### Migração Realizada

**CRY-72 Checklist**:
- ✅ Instalar dependências: `celery[redis]==5.3.4`, `redis<5.0.0`, `celery-redbeat==2.3.3`
- ✅ Criar `celery_app.py` e `celeryconfig.py`
- ✅ Converter APScheduler jobs → Celery tasks
- ✅ Configurar Docker Compose (5 novos serviços)
- ✅ Testar localmente com `docker-compose up`
- ✅ Deploy em produção (Docker Swarm - DigitalOcean)
- ✅ Monitorar Flower dashboard
- ✅ Depreciar APScheduler (remover `apscheduler>=3.10.0` de requirements.txt)

## Métricas de Sucesso

### Antes (APScheduler)
- ⚠️ Uptime: ~85% (crashes frequentes)
- ⚠️ sync_orders bloqueando outros workflows
- ⚠️ Sem visibility de tasks

### Depois (Celery + Redis)
- ✅ Uptime: ~95% (workers auto-restart)
- ✅ sync_orders isolado (10 concurrency)
- ✅ Flower dashboard usado diariamente

### KPIs Atuais (Pós-Migração)
| Métrica | Valor |
|---------|-------|
| **Workers Ativos** | 2 (sync + smart) |
| **Task Success Rate** | ~92% |
| **Avg Task Duration** | sync_orders: 3s, smart_buy: 2s |
| **Redis Memory Usage** | ~150MB |
| **Failed Tasks/Day** | 5-10 (maioria retry e sucesso) |

## Lições Aprendidas

### O Que Funcionou Bem
1. **RedBeat**: Schedule persistente no Redis (melhor que DBM shelve)
2. **Flower**: Dashboard indispensável para debugging
3. **Queues Dedicadas**: sync_orders não bloqueia mais outros workflows
4. **Distributed Locks**: Resolveu version conflicts em updates

### Desafios Enfrentados
1. **Async Bridge**: Precisou `asyncio.run()` em tasks
2. **Redis Config**: Password authentication levou 1h para debugar
3. **Concurrency Tuning**: Balancear 10 sync vs 5 smart foi trial-and-error

### Problemas Remanescentes (Para Resolver)
1. **sync_orders Ainda Falhando**: Ordens não sincronizadas corretamente (~8% taxa de falha)
   - Próximo passo: Aumentar retry, melhorar logging
2. **Workflows Acoplados ao MB**: Deveria ser exchange-agnostic
   - Próximo passo: Refatorar para usar apenas BaseExchange interface

## Referências

- [Celery Documentation](https://docs.celeryq.dev/)
- [RedBeat GitHub](https://github.com/sibson/redbeat)
- [Issue CRY-72: Celery Migration](link-to-linear-issue)
- [Flower Dashboard](http://localhost:5555) (produção)
- [Código: `celeryconfig.py`](../../../crypteras-improved/backend/src/infrastructure/celery/celeryconfig.py)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Implementação**: 2024-10-15 (CRY-72)
**Status Atual**: ✅ Em produção, funcionando bem. Oportunidades de otimização em sync_orders.
