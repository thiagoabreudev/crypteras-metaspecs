---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "technical"
tags: ['contributing', 'technical']
---

# Guia de Contribuição - Crypteras Trading System

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir processo de contribuição para manter qualidade e consistência do código.

**Constraints** (limites obrigatórios):
- Code review obrigatório
- Testes automatizados devem passar (CI/CD)
- Seguir style guide (PEP8 para Python)
- Commits semânticos (conventional commits)
- Branch naming: feature/*, bugfix/*, hotfix/*

**Non-Goals** (o que NÃO fazer):
- Criar processo burocrático que desencoraja contribuições
- Exigir 100% de cobertura de testes (pragmatismo)
- Rejeitar PRs por style nitpicks (use linter)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Guia de contribuição para desenvolvedores
:::

Este documento descreve o workflow de desenvolvimento real usado pela equipe.

---

## 🚀 Configuração do Ambiente Local

### Pré-requisitos

- Docker e Docker Compose instalados
- Python 3.10+ (para desenvolvimento local sem Docker)
- Node.js 18+ (para frontend local)
- Git

### Setup Inicial

```bash
# 1. Clonar repositório
git clone <repo-url>
cd crypteras-improved

# 2. Copiar .env example
cp backend/.env.example backend/.env

# 3. Editar credenciais (opcional para desenvolvimento)
# backend/.env
MONGO_ROOT_PASSWORD=change_this_password
MONGO_USER_PASSWORD=change_this_password
REDIS_PASSWORD=change_this_redis_password
OPENAI_API_KEY=sk-...  # Obrigatório se testar IA

# 4. Subir todos serviços
docker-compose up -d

# 5. Verificar que tudo está rodando
docker-compose ps
```

**Serviços Disponíveis**:
- Backend (FastAPI): http://localhost:7777, http://localhost:8000
- Frontend (Nuxt.js): http://localhost:3000
- MongoDB: localhost:27017
- Redis: localhost:6379
- Flower (Celery): http://localhost:5555 (user: admin, pass: veja .env)

---

## 🔄 Workflow de Desenvolvimento

### Fluxo de Git

**Branches**:
- `main` - Produção (protegida)
- `develop` - Desenvolvimento (não existe atualmente, tudo em main)
- `feature/CRY-XX-nome` - Features
- `fix/descricao` - Bugfixes

**Convenção de Commits**:
```bash
# Features
git commit -m "feat(smart-bot): add circuit breaker reset endpoint"

# Bugfixes
git commit -m "fix(sync-orders): handle exchange timeout correctly"

# Refatoração
git commit -m "refactor(workflows): remove TradingConfig references"

# Testes
git commit -m "test(candle-bot): add integration tests for signal generation"

# Documentação
git commit -m "docs(adr): add ADR-008 for Binance integration"
```

### Workflow Típico

```bash
# 1. Criar branch
git checkout -b feature/CRY-XX-minha-feature

# 2. Desenvolver localmente
docker-compose up -d  # Se não estiver rodando
# Editar código...

# 3. Testar localmente
# Backend
docker exec -it crypteras-backend pytest backend/tests/

# Frontend
docker exec -it crypteras-frontend npm run test  # Se houver testes

# 4. Verificar lint (futuro - não implementado ainda)
# black --check backend/src/
# flake8 backend/src/

# 5. Commit e push
git add .
git commit -m "feat(bot): descrição da mudança"
git push origin feature/CRY-XX-minha-feature

# 6. Abrir Pull Request no GitHub/GitLab
# - Descrever mudanças
# - Linkar issue (CRY-XX)
# - Solicitar review

# 7. Após aprovação, merge para main
```

---

## 🧪 Testes

### Executar Testes Localmente

**Backend (Python + Pytest)**:
```bash
# Todos os testes
docker exec -it crypteras-backend pytest backend/tests/

# Testes específicos
docker exec -it crypteras-backend pytest backend/tests/unit/domain/test_smart_bot.py

# Com cobertura
docker exec -it crypteras-backend pytest backend/tests/ --cov=backend/src --cov-report=html

# Ver relatório de cobertura
open backend/htmlcov/index.html
```

**Estrutura de Testes**:
```
tests/
├── unit/               # Testes unitários (Domain entities)
│   ├── domain/
│   │   ├── test_smart_bot.py
│   │   ├── test_candle_bot.py
│   │   └── test_dca_bot.py
│   └── services/
│       └── test_technical_analysis.py
├── integration/        # Testes de integração (Workflows + Repos)
│   ├── workflows/
│   │   ├── test_smart_bot_purchase_workflow.py
│   │   └── test_sync_orders_workflow.py
│   └── repositories/
│       └── test_smart_bot_repository.py
└── fixtures/           # Fixtures compartilhados
    ├── bots.py
    └── users.py
```

**Boas Práticas de Testes**:
```python
# ✅ CORRETO: Testes de Domain (sem mocks)
def test_smart_bot_can_execute_purchase_when_active():
    bot = SmartBot(
        name="Test",
        status=BotStatus.ACTIVE,
        max_position_size=Decimal("5000"),
        stats=SmartBotStats(total_invested=Decimal("2000"))
    )

    assert bot.can_execute_purchase() == True

# ✅ CORRETO: Testes de Workflow (com mocks)
@pytest.mark.asyncio
async def test_purchase_workflow_creates_order():
    mock_exchange = MockExchange(orderbook={...})
    workflow = SmartBotPurchaseWorkflow(
        exchange_factory=MockExchangeFactory(mock_exchange)
    )

    result = await workflow.execute(bot, user)

    assert result.success == True
    assert mock_exchange.create_order_called == True
```

---

## 🏗️ Padrões de Código

### Clean Architecture - OBRIGATÓRIO

**Domain Layer** (`src/domain/`):
- ❌ NUNCA importar FastAPI, Celery, MongoDB, AGNO
- ✅ APENAS stdlib + typing
- ✅ Lógica de negócio pura

```python
# ✅ CORRETO
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class SmartBot:
    def can_execute_purchase(self) -> bool:
        return self.status == BotStatus.ACTIVE

# ❌ ERRADO
from fastapi import HTTPException  # NÃO no Domain!
from motor.motor_asyncio import AsyncIOMotorClient  # NÃO no Domain!
```

**Application Layer** (`src/application/`):
- ✅ Depende apenas do Domain
- ✅ Orquestra Domain Services
- ❌ Não conhece MongoDB, FastAPI diretamente

**Infrastructure Layer** (`src/infrastructure/`):
- ✅ Implementa interfaces do Domain
- ✅ Único lugar com frameworks

### PEP8 - SEGUIR (Futuro)

**Nota**: Código atual NÃO segue PEP8 (débito técnico). Código novo DEVE seguir:

```python
# ✅ CORRETO
def get_available_funds(self) -> Decimal:
    return self.max_position_size - self.total_invested

# ❌ ERRADO
def getAvailableFunds(self)->Decimal:  # camelCase, sem espaço
    return self.maxPositionSize-self.totalInvested
```

### Type Hints - OBRIGATÓRIO

```python
# ✅ CORRETO
from typing import Optional, List
from decimal import Decimal

async def create_order(
    symbol: str,
    side: str,
    quantity: Decimal,
    price: Optional[Decimal] = None
) -> Dict[str, Any]:
    ...

# ❌ ERRADO
async def create_order(symbol, side, quantity, price=None):  # Sem types
    ...
```

### Decimal para Dinheiro - OBRIGATÓRIO

```python
# ✅ CORRETO
from decimal import Decimal

price = Decimal("155000.50")
quantity = Decimal("0.001")
total = price * quantity

# ❌ ERRADO
price = 155000.50  # float tem erros de arredondamento!
quantity = 0.001
```

---

## 📝 Adicionando Novas Features

### Adicionar Novo Tipo de Bot

**1. Criar Entity no Domain**:
```python
# backend/src/domain/entities/grid_bot.py

from dataclasses import dataclass
from decimal import Decimal

@dataclass
class GridBot:
    name: str
    grid_size: int  # Quantidade de níveis de grid
    price_range_min: Decimal
    price_range_max: Decimal

    def calculate_grid_levels(self) -> List[Decimal]:
        """Calcula níveis de preço do grid"""
        step = (self.price_range_max - self.price_range_min) / self.grid_size
        return [self.price_range_min + (i * step) for i in range(self.grid_size + 1)]
```

**2. Criar Repository Interface (Domain)**:
```python
# backend/src/domain/repositories/grid_bot_repository.py

from abc import ABC, abstractmethod

class IGridBotRepository(ABC):
    @abstractmethod
    async def create(self, bot: GridBot) -> GridBot:
        pass

    @abstractmethod
    async def get_by_id(self, bot_id: str) -> GridBot:
        pass
```

**3. Implementar Repository (Infrastructure)**:
```python
# backend/src/infrastructure/repositories/grid_bot_repository_impl.py

class GridBotRepositoryImpl(IGridBotRepository):
    def __init__(self, mongo_client: AsyncIOMotorClient):
        self.collection = mongo_client.crypteras_trading.grid_bots

    async def create(self, bot: GridBot) -> GridBot:
        doc = {"name": bot.name, "grid_size": bot.grid_size, ...}
        result = await self.collection.insert_one(doc)
        bot.id = str(result.inserted_id)
        return bot
```

**4. Criar Workflow**:
```python
# backend/src/workflows/grid_bot_execution.py

async def execute_grid_bot(bot: GridBot, user: User):
    exchange = await ExchangeFactory.create(bot.exchange, user)
    levels = bot.calculate_grid_levels()

    for level in levels:
        # Colocar ordem em cada nível do grid
        await exchange.create_order(...)
```

**5. Criar Celery Task**:
```python
# backend/src/infrastructure/celery/tasks/grid_bot_tasks.py

@celery_app.task(name='grid_bot_execution')
def grid_bot_execution_task():
    asyncio.run(execute_grid_bot_workflow())
```

**6. Adicionar ao Beat Schedule**:
```python
# backend/src/infrastructure/celery/celeryconfig.py

beat_schedule = {
    # ... outros workflows
    'grid-bot-execution': {
        'task': 'grid_bot_execution',
        'schedule': 71.0,  # Co-primo
    },
}
```

**7. Criar API Route**:
```python
# backend/src/api/routes/grid_bots.py

router = APIRouter()

@router.post("/")
async def create_grid_bot(config: GridBotConfig, current_user: User = Depends(get_current_user)):
    use_case = CreateGridBotUseCase(grid_bot_repository)
    bot = await use_case.execute(current_user.id, config)
    return bot
```

**8. Frontend (Composable)**:
```typescript
// frontend/composables/useGridBots.ts

export const useGridBots = () => {
  const bots = ref<GridBot[]>([])

  const fetchBots = async () => {
    const response = await $fetch('/api/grid-bots')
    bots.value = response.data
  }

  const createBot = async (config: GridBotConfig) => {
    return await $fetch('/api/grid-bots', {
      method: 'POST',
      body: config
    })
  }

  return { bots, fetchBots, createBot }
}
```

---

## 🚢 Deploy

### Deploy em Produção (Docker Swarm)

**Processo Atual** (manual via script):

```bash
# 1. Conectar ao servidor
ssh deploy@droplet.digitalocean.com

# 2. Executar script de deploy
cd /root/crypteras-improved
./build-deploy.sh

# Script faz:
# - docker-compose build
# - docker-compose down
# - docker-compose up -d
# - Verifica health checks
```

**Após Deploy**:
```bash
# Verificar serviços
docker service ls  # Se usando Docker Swarm
# ou
docker-compose ps  # Se usando Docker Compose

# Ver logs
docker logs crypteras-backend -f --tail 100

# Verificar Flower
curl http://localhost:5555

# Verificar Backend
curl http://localhost:7777/playground/status
```

**Rollback** (se necessário):
```bash
# Voltar para versão anterior
git checkout <commit-anterior>
./build-deploy.sh
```

---

## 🔍 Debug em Desenvolvimento

### Backend

```bash
# Logs em tempo real
docker logs crypteras-backend -f

# Entrar no container
docker exec -it crypteras-backend bash

# Python REPL dentro do container
docker exec -it crypteras-backend python
>>> from src.domain.entities.smart_bot import SmartBot
>>> bot = SmartBot(...)
>>> bot.can_execute_purchase()

# Debugar com ipdb (adicionar no código)
import ipdb; ipdb.set_trace()
```

### MongoDB

```bash
# Entrar no MongoDB
docker exec -it crypteras-mongo mongosh -u crypteras_admin -p ${MONGO_ROOT_PASSWORD}

use crypteras_trading

# Ver collections
show collections

# Buscar bots
db.smart_bots.find().pretty()

# Contar ordens pending
db.orders.find({"status": "PENDING"}).count()
```

### Redis

```bash
# Entrar no Redis
docker exec -it crypteras-redis redis-cli -a ${REDIS_PASSWORD}

# Ver keys
KEYS *

# Ver locks
KEYS lock:*

# Ver queue Celery
LLEN sync_orders
```

### Celery

```bash
# Ver workers ativos
docker exec -it crypteras-backend celery -A src.infrastructure.celery.celery_app inspect active

# Ver tasks scheduled
docker exec -it crypteras-backend celery -A src.infrastructure.celery.celery_app inspect scheduled

# Forçar execução de task
docker exec -it crypteras-backend celery -A src.infrastructure.celery.celery_app call src.infrastructure.celery.tasks.sync_orders_task
```

---

## 📚 Recursos

**Documentação**:
- [CLAUDE.meta.md](CLAUDE.meta.md) - Guia de desenvolvimento com IA
- [CODEBASE_GUIDE.md](CODEBASE_GUIDE.md) - Navegação do código
- [BUSINESS_LOGIC.md](BUSINESS_LOGIC.md) - Regras de negócio
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problemas comuns

**ADRs Importantes**:
- [ADR-001: Clean Architecture](adr/001-clean-architecture.md)
- [ADR-002: Celery + Redis](adr/002-celery-redis-migration.md)
- [ADR-005: Adapter Pattern](adr/005-adapter-pattern-exchanges.md)

---

**Última Atualização**: 2024-12-24
**Baseado em**: Workflow real da equipe
