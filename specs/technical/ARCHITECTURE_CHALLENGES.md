---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "technical"
tags: ['technical', 'architecture_challenges']
---

# Desafios Arquiteturais e Débito Técnico - Crypteras Trading System

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Documentar desafios arquiteturais conhecidos e technical debt para transparência e priorização.

**Constraints** (limites obrigatórios):
- Categorizar por severidade (critical, high, medium, low)
- Incluir impacto e esforço estimado para resolver
- Referenciar issues/tickets relacionados
- Atualizar conforme desafios são resolvidos

**Non-Goals** (o que NÃO fazer):
- Listar todos micro-refactorings desejados
- Criar análise acadêmica de arquitetura ideal
- Gerar ansiedade sem plano de ação
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Desafios arquiteturais e soluções
:::

Este documento lista os desafios técnicos atuais e oportunidades de melhoria identificadas pela equipe.

---

## 🔴 Prioridade Alta - Impacta Produção

### 1. Código Não Segue PEP8

**Problema**:
- Código Python não segue convenções PEP8
- Nomes de variáveis inconsistentes (`camelCase` vs `snake_case`)
- Imports fora de ordem
- Linhas muito longas (> 120 caracteres)
- Espaçamento inconsistente

**Impacto**:
- ⚠️ Difícil leitura para novos desenvolvedores
- ⚠️ Code reviews demoram mais
- ⚠️ IDE warnings constantes
- ⚠️ Inconsistência dificulta manutenção

**Exemplos Reais**:
```python
# ❌ Problemas encontrados no código atual

# 1. camelCase ao invés de snake_case
def getUserBalance(userId):  # Deveria ser get_user_balance(user_id)
    ...

# 2. Imports fora de ordem
from src.domain.entities.smart_bot import SmartBot  # local
import asyncio  # stdlib (deveria vir primeiro)
from fastapi import FastAPI  # third-party

# 3. Linhas muito longas
bot = SmartBot(name="BTC Conservative", max_position_size=Decimal("5000"), soft_stop_loss=Decimal("2.0"), hard_stop_loss=Decimal("5.0"), ...)  # > 150 caracteres

# 4. Espaçamento inconsistente
result=await repository.create(bot)  # Falta espaço ao redor de =
```

**Solução Proposta**:

**Fase 1: Configurar Linters (1 sprint)**
```bash
# 1. Adicionar dependências
pip install black flake8 isort mypy

# 2. Configurar pyproject.toml
[tool.black]
line-length = 100
target-version = ['py310']

[tool.isort]
profile = "black"
line_length = 100

[tool.flake8]
max-line-length = 100
extend-ignore = E203, W503  # Compatível com black

# 3. Adicionar pre-commit hooks
pip install pre-commit
```

**.pre-commit-config.yaml**:
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.0.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
```

**Fase 2: Refatoração Gradual (3-4 sprints)**
```bash
# Aplicar black em um módulo por vez
black backend/src/domain/entities/smart_bot.py
black backend/src/domain/entities/candle_bot.py
# ...

# Verificar com flake8
flake8 backend/src/domain/
```

**Fase 3: CI/CD Enforcement (1 sprint)**
```yaml
# .github/workflows/lint.yml
name: Lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.10
      - name: Install dependencies
        run: |
          pip install black flake8 isort
      - name: Run black
        run: black --check backend/src/
      - name: Run flake8
        run: flake8 backend/src/
      - name: Run isort
        run: isort --check backend/src/
```

**Estimativa**: 5-6 sprints para refatorar todo codebase
**ROI**: Código 50% mais fácil de manter, onboarding de novos devs 30% mais rápido

---

### 2. Resquícios de TradingConfig (Deprecated)

**Problema**:
- `TradingConfig` entity foi depreciada em CRY-82 (v2.8)
- Código antigo ainda referencia `TradingConfig`
- Imports deprecated ainda existem
- Algumas funcionalidades quebradas por dependências antigas

**Impacto**:
- ⚠️ Código confuso (mistura de padrão antigo e novo)
- ⚠️ Possíveis bugs se código antigo for chamado
- ⚠️ Novos devs não sabem qual padrão seguir

**Localizações Conhecidas**:
```bash
# Encontrar todas referências
$ grep -r "TradingConfig" backend/src/
backend/src/domain/entities/trading_config.py  # Arquivo deprecated (pode deletar)
backend/src/workflows/legacy/old_purchase.py:15  # Import antigo
backend/src/api/routes/contexts.py:42  # Comentário antigo
```

**Solução Proposta**:

**Fase 1: Audit (1 sprint)**
```bash
# 1. Listar todos arquivos com TradingConfig
grep -r "TradingConfig" backend/src/ > trading_config_audit.txt

# 2. Classificar:
# - Código ativo (precisa refatorar)
# - Código comentado (pode deletar)
# - Arquivos deprecated (pode deletar)

# 3. Criar checklist de refatoração
```

**Fase 2: Refatoração (2-3 sprints)**
```python
# ❌ Código antigo (remover)
from src.domain.entities.trading_config import TradingConfig

config = await trading_config_repository.get_by_user(user_id)
await workflow.execute(config)

# ✅ Código novo (já implementado para SmartBot)
from src.domain.entities.smart_bot import SmartBot

bot = await smart_bot_repository.get_by_id(bot_id)
await workflow.execute(bot)
```

**Fase 3: Limpeza (1 sprint)**
```bash
# Deletar arquivos deprecated
rm backend/src/domain/entities/trading_config.py
rm backend/src/infrastructure/repositories/trading_config_repository.py

# Deletar testes antigos
rm tests/unit/domain/test_trading_config.py

# Atualizar ADR-007
# Marcar como "COMPLETAMENTE REMOVIDO"
```

**Estimativa**: 4-5 sprints
**ROI**: Código 40% mais limpo, zero ambiguidade

---

### 3. Workflows Acoplados ao Mercado Bitcoin

**Problema**:
- Workflows assumem formato de resposta do Mercado Bitcoin
- Código com `if bot.exchange == "mercado_bitcoin"` hardcoded
- Binance não funciona corretamente por causa desse acoplamento

**Impacto**:
- 🔴 **Binance ainda em teste** (não production-ready)
- ⚠️ Difícil adicionar novas exchanges (Kraken, Coinbase, etc.)
- ⚠️ Manutenção duplicada (código para MB + código para Binance)

**Exemplos Reais**:
```python
# ❌ Problema 1: Acesso direto a formato MB
orderbook = await exchange.get_orderbook(symbol)
best_bid = Decimal(orderbook['data']['bids'][0][0])  # MB retorna {"data": {...}}
# Binance NÃO retorna "data"! Causa KeyError

# ❌ Problema 2: Hardcoded exchange checks
if bot.exchange == "mercado_bitcoin":
    # Lógica específica MB
    ...
elif bot.exchange == "binance":
    # Lógica específica Binance (duplicada)
    ...

# ❌ Problema 3: Normalização incompleta
symbol = bot.symbol  # Pode ser "BTC-BRL" ou "BTCUSDT"
# Código assume sempre "BTC-BRL" (formato MB)
```

**Solução Proposta**:

**Fase 1: Completar Normalização (2 sprints)**
```python
# ✅ CORRETO: Usar apenas BaseExchange interface
exchange: BaseExchange = await ExchangeFactory.create(bot.exchange, user)
orderbook = await exchange.get_orderbook(bot.symbol)  # Formato JÁ normalizado
best_bid = orderbook['bids'][0]['price']  # Funciona para MB E Binance
```

**OrderNormalizer Aprimorado**:
```python
# backend/src/infrastructure/exchanges/order_normalizer.py

class OrderNormalizer:
    @staticmethod
    def normalize_orderbook(raw: Dict, exchange: str) -> Dict:
        """Normaliza orderbook de qualquer exchange para formato único"""
        if exchange == "mercado_bitcoin":
            return {
                "bids": [
                    {"price": Decimal(bid[0]), "quantity": Decimal(bid[1])}
                    for bid in raw['data']['bids']
                ],
                "asks": [
                    {"price": Decimal(ask[0]), "quantity": Decimal(ask[1])}
                    for ask in raw['data']['asks']
                ]
            }

        elif exchange == "binance":
            return {
                "bids": [
                    {"price": Decimal(str(bid[0])), "quantity": Decimal(str(bid[1]))}
                    for bid in raw['bids']  # SEM "data"!
                ],
                "asks": [
                    {"price": Decimal(str(ask[0])), "quantity": Decimal(str(ask[1]))}
                    for ask in raw['asks']
                ]
            }

        else:
            raise UnsupportedExchangeError(f"Exchange {exchange} não suportada")
```

**Fase 2: Refatorar Workflows (3-4 sprints)**
```python
# ❌ ANTES: Workflow acoplado
async def smart_bot_purchase(bot: SmartBot):
    if bot.exchange == "mercado_bitcoin":
        mb_exchange = MercadoBitcoinExchange(...)
        orderbook = await mb_exchange.get_orderbook(bot.symbol)
        best_bid = Decimal(orderbook['data']['bids'][0][0])

    elif bot.exchange == "binance":
        binance_exchange = BinanceExchange(...)
        orderbook = await binance_exchange.get_order_book(bot.symbol)
        best_bid = Decimal(orderbook['bids'][0][0])

# ✅ DEPOIS: Workflow exchange-agnostic
async def smart_bot_purchase(bot: SmartBot):
    exchange: BaseExchange = await ExchangeFactory.create(bot.exchange, user)
    orderbook = await exchange.get_orderbook(bot.symbol)  # Normalizado
    best_bid = orderbook['bids'][0]['price']  # Funciona para qualquer exchange
```

**Fase 3: Testes de Integração (2 sprints)**
```python
# tests/integration/exchanges/test_all_exchanges.py

@pytest.mark.parametrize("exchange_name", ["mercado_bitcoin", "binance"])
@pytest.mark.asyncio
async def test_purchase_workflow_works_for_all_exchanges(exchange_name):
    bot = SmartBot(exchange=exchange_name, ...)
    user = User(...)  # Com credenciais de ambas exchanges

    workflow = SmartBotPurchaseWorkflow(...)
    result = await workflow.execute(bot, user)

    assert result.success == True
    assert result.order.price > Decimal("0")
```

**Estimativa**: 7-8 sprints
**ROI**: Binance production-ready, fácil adicionar Kraken/Coinbase (2-3 dias por exchange)

---

## 🟡 Prioridade Média - Melhoria de Qualidade

### 4. Cobertura de Testes Insuficiente em Workflows

**Problema**:
- Workflows têm ~60-70% de cobertura de testes
- Testes de integração incompletos
- Falta testes de edge cases

**Impacto**:
- ⚠️ Bugs descobertos em produção (sync_orders ~8% falha)
- ⚠️ Medo de refatorar (sem confiança nos testes)
- ⚠️ Regressões não detectadas

**Gaps Identificados**:
```bash
# Workflows com baixa cobertura
backend/src/workflows/sync_orders.py  # 55% cobertura
backend/src/workflows/smart_bot_check_situation.py  # 60%
backend/src/workflows/candle_bot_analysis_workflow.py  # 50%
```

**Solução Proposta**:

**Fase 1: Aumentar Cobertura de Unit Tests (2 sprints)**
```python
# tests/unit/workflows/test_sync_orders.py

@pytest.mark.asyncio
async def test_sync_orders_updates_total_invested_correctly():
    # Arrange
    bot = SmartBot(stats=SmartBotStats(total_invested=Decimal("1000")))
    filled_order = Order(amount=Decimal("500"), side="BUY", status="FILLED")

    # Act
    await sync_orders_workflow.process_order(bot, filled_order)

    # Assert
    assert bot.stats.total_invested == Decimal("1500")  # 1000 + 500

@pytest.mark.asyncio
async def test_sync_orders_handles_version_conflict():
    # Arrange
    bot = SmartBot(version=5)
    # Simular outro processo atualizou bot (version agora é 6)

    # Act & Assert
    with pytest.raises(VersionConflictError):
        await bot_repository.update(bot)
```

**Fase 2: Testes de Integração com Mocks (2 sprints)**
```python
# tests/integration/workflows/test_smart_bot_purchase_integration.py

@pytest.mark.asyncio
async def test_purchase_workflow_end_to_end():
    # Setup
    user = await create_test_user()
    bot = await create_test_smart_bot(user_id=user.id)
    mock_exchange = MockExchange()

    # Execute workflow completo
    workflow = SmartBotPurchaseWorkflow(
        bot_repository=TestBotRepository(),
        exchange_factory=MockExchangeFactory(mock_exchange)
    )

    result = await workflow.execute(bot, user)

    # Verificações completas
    assert result.success == True
    assert mock_exchange.create_order_called == True
    assert bot.stats.total_blocked > Decimal("0")

    # Verificar estado no DB
    saved_bot = await bot_repository.get_by_id(bot.id)
    assert saved_bot.stats.total_blocked == bot.stats.total_blocked
```

**Fase 3: Testes de Edge Cases (2 sprints)**
```python
# tests/integration/workflows/test_edge_cases.py

@pytest.mark.asyncio
async def test_sync_orders_with_exchange_timeout():
    # Simular timeout de 30s na exchange
    mock_exchange = MockExchange(simulate_timeout=True)

    with pytest.raises(ExchangeTimeoutError):
        await sync_orders_workflow.execute()

@pytest.mark.asyncio
async def test_purchase_workflow_with_insufficient_balance():
    bot = SmartBot(max_position_size=Decimal("5000"))
    bot.stats.total_invested = Decimal("5000")  # Tudo investido

    result = await purchase_workflow.execute(bot, user)

    assert result.success == False
    assert result.reason == "insufficient_balance"
```

**Meta**: 80%+ cobertura em workflows críticos
**Estimativa**: 6 sprints
**ROI**: 50% menos bugs em produção, confiança para refatorar

---

### 5. Falta de CI/CD Robusto

**Problema**:
- Deploy manual via script `build-deploy.sh`
- Sem testes automáticos em PR
- Sem lint em CI/CD
- Sem validação de cobertura mínima

**Impacto**:
- ⚠️ Bugs chegam em produção
- ⚠️ PRs aprovados sem rodar testes
- ⚠️ Código não-PEP8 merged

**Solução Proposta**:

**Fase 1: GitHub Actions (1 sprint)**
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: 3.10
      - run: pip install black flake8 isort
      - run: black --check backend/src/
      - run: flake8 backend/src/
      - run: isort --check backend/src/

  test:
    runs-on: ubuntu-latest
    services:
      mongo:
        image: mongo:7.0
        ports:
          - 27017:27017
      redis:
        image: redis:7.2-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
      - run: pip install -r backend/requirements.txt
      - run: pytest backend/tests/ --cov=backend/src --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v2

  coverage-check:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Check coverage >= 70%
        run: |
          coverage=$(pytest --cov=backend/src --cov-report=term | grep TOTAL | awk '{print $4}' | sed 's/%//')
          if (( $(echo "$coverage < 70" | bc -l) )); then
            echo "Coverage $coverage% is below 70%"
            exit 1
          fi
```

**Fase 2: Deploy Automatizado (2 sprints)**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Build Docker images
        run: docker-compose build

      - name: Push to registry
        run: |
          docker tag crypteras-backend:latest registry.digitalocean.com/crypteras/backend:${{ github.sha }}
          docker push registry.digitalocean.com/crypteras/backend:${{ github.sha }}

      - name: Deploy to production (Docker Swarm)
        run: |
          ssh deploy@droplet "
            docker pull registry.digitalocean.com/crypteras/backend:${{ github.sha }}
            docker service update --image registry.digitalocean.com/crypteras/backend:${{ github.sha }} crypteras_backend
          "
```

**Estimativa**: 3 sprints
**ROI**: Zero bugs por falta de testes, deploy 10x mais rápido

---

## 🟢 Prioridade Baixa - Nice to Have

### 6. Documentação de API Desatualizada

**Problema**:
- Endpoints não documentados com OpenAPI/Swagger
- Frontend descobre endpoints lendo código backend

**Solução**:
- FastAPI gera docs automáticos em `/docs`
- Adicionar docstrings em todos endpoints

**Estimativa**: 2 sprints

---

### 7. Logs Não Estruturados Suficientemente

**Problema**:
- Logs misturados (print + logging + structlog)
- Difícil filtrar no Grafana

**Solução**:
- Padronizar 100% structlog
- Adicionar trace IDs para rastrear requests

**Estimativa**: 1 sprint

---

## 📊 Roadmap de Refatoração Sugerido

### Q1 2025 (Jan-Mar)
- ✅ **Sprint 1-2**: Configurar linters (black, flake8, isort) + CI/CD básico
- ✅ **Sprint 3-4**: Remover resquícios de TradingConfig
- ✅ **Sprint 5-6**: Aplicar PEP8 em domain/ e workflows/

### Q2 2025 (Abr-Jun)
- ✅ **Sprint 1-3**: Refatorar workflows para exchange-agnostic
- ✅ **Sprint 4-5**: Aumentar cobertura de testes (workflows > 80%)
- ✅ **Sprint 6**: Deploy automatizado (CI/CD completo)

### Q3 2025 (Jul-Set)
- ✅ **Sprint 1-2**: Binance production-ready
- ✅ **Sprint 3**: Adicionar Kraken support (teste de extensibilidade)
- ✅ **Sprint 4-6**: Melhorias de observabilidade (logs estruturados, trace IDs)

---

## 📚 Referências

**ADRs Relacionados**:
- [ADR-005: Adapter Pattern](adr/005-adapter-pattern-exchanges.md)
- [ADR-007: TradingConfig Deprecated](adr/007-trading-config-deprecated.md)

**Documentos Relacionados**:
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problemas atuais
- [CONTRIBUTING.md](CONTRIBUTING.md) - Padrões a seguir

---

**Última Atualização**: 2024-12-24
**Baseado em**: Feedback da equipe + análise de código
