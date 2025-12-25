---
adr_number: "005"
title: "Adapter Pattern para Multi-Exchange Support"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['technical', 'exchanges', 'decision', 'adapter', 'architecture', 'pattern']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# ADR-005: Adapter Pattern para Multi-Exchange Support

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Desacoplar workflows de trading da implementação específica de cada exchange, permitindo adicionar novas exchanges sem reescrever lógica de negócio.

**Constraints** (limites obrigatórios):
- Todos workflows DEVEM usar BaseExchange (nunca classes concretas MercadoBitcoinExchange ou BinanceExchange diretamente)
- Novos adapters DEVEM implementar todos métodos abstratos de BaseExchange
- Formato normalizado de dados (orderbook, balance, candles) DEVE ser consistente entre exchanges
- Preços DEVEM ser retornados como Decimal (nunca float ou string)
- Symbols DEVEM ser normalizados no adapter (ex: "BTC-BRL" no MB vira "BTC/BRL" internamente)
- ExchangeFactory DEVE ser único ponto de criação de instâncias de exchanges
- Credenciais de exchanges DEVEM ser encriptadas no User entity (nunca plain text)

**Non-Goals** (o que NÃO fazer):
- Criar adapters para exchanges que não suportam APIs públicas REST (apenas exchanges com APIs modernas)
- Suportar exchanges sem orderbook (ex: P2P exchanges)
- Implementar lógica de trading específica de exchange nos adapters (ex: "MB permite apenas X ordens por minuto")
- Expor métodos específicos de exchange no BaseExchange (ex: método get_funding_rate só existe em Binance Futures)
- Usar bibliotecas third-party CCXT ou similar (manter controle total sobre adapters)
- Permitir workflows acessarem atributos internos dos adapters (ex: adapter.client._session)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Adapter pattern para exchanges
:::

## Status
✅ **Aceito** (Implementado em CRY-20 v2.2)

## Data
2024-10-01 (CRY-20: Binance Support)

## Contexto

### Problema

Sistema inicialmente suportava apenas **Mercado Bitcoin**:
```python
# Código antigo (acoplado ao MB)
from src.exchanges.mercado_bitcoin import MercadoBitcoinExchange

exchange = MercadoBitcoinExchange(api_id, api_key)
orderbook = await exchange.get_orderbook("BTC-BRL")
```

**Necessidade**: Adicionar **Binance** (e potencialmente outras exchanges no futuro).

**Desafios**:
1. **Formatos Diferentes**:
   - MB: `{"symbol": "BTC-BRL", "price": "155000.00"}` (string)
   - Binance: `{"symbol": "BTCUSDT", "price": 31000.50}` (float)

2. **APIs Diferentes**:
   - MB: Autenticação via login/password → Bearer token
   - Binance: HMAC SHA256 signature com API key/secret

3. **Timeframes Diferentes**:
   - MB: 7 timeframes (1m, 5m, 15m, 30m, 1h, 6h, 1d)
   - Binance: 16 timeframes (1s, 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d, 3d, 1w, 1M)

4. **Código Acoplado**: Workflows assumiam formato MB
   ```python
   # Problema: hardcoded para MB
   if orderbook['data']['bids'][0]:
       best_bid = Decimal(orderbook['data']['bids'][0][0])  # MB format
   ```

## Decisão

Implementar **Adapter Pattern** (Gang of Four) com interface unificada.

### Arquitetura Implementada

```
┌────────────────────────────────────────────────────────┐
│              BaseExchange (Abstract Interface)         │
│  - get_balance()                                       │
│  - get_orderbook()                                     │
│  - create_order()                                      │
│  - cancel_order()                                      │
│  - get_candles()                                       │
│  - normalize_symbol()                                  │
└────────────────────────────────────────────────────────┘
               ↑                          ↑
               │                          │
   ┌───────────┴────────┐     ┌──────────┴─────────────┐
   │                    │     │                        │
┌──┴──────────────────────┐ ┌─┴─────────────────────────┐
│ MercadoBitcoinExchange  │ │   BinanceExchange         │
│ - Implements BaseExchange│ │ - Implements BaseExchange │
│ - MB API v4 client      │ │ - Binance API v3 client   │
│ - 7 timeframes          │ │ - 16 timeframes           │
│ - Bearer token auth     │ │ - HMAC signature auth     │
└─────────────────────────┘ └───────────────────────────┘
               ↓                          ↓
   ┌───────────┴────────────────────┬─────┴──────────┐
   │                                │                │
┌──┴──────────────┐  ┌─────────────┴────┐  ┌────────┴──────┐
│ OrderNormalizer │  │ SymbolNormalizer │  │ TimeframeMap  │
│ - MB → Unified  │  │ - BTC-BRL ↔      │  │ - 1m → 1m     │
│ - Binance →     │  │   BTCUSDT        │  │ - 1h → 1h     │
│   Unified       │  │                  │  │ - 1s (Binance)│
└─────────────────┘  └──────────────────┘  └───────────────┘
```

### BaseExchange (Interface Abstrata)

```python
# src/exchanges/base_exchange.py
from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from decimal import Decimal
from datetime import datetime

class BaseExchange(ABC):
    """Interface unificada para todas exchanges"""

    @abstractmethod
    async def get_balance(self, asset: str) -> Decimal:
        """Retorna saldo disponível de um ativo"""
        pass

    @abstractmethod
    async def get_orderbook(self, symbol: str, depth: int = 10) -> Dict:
        """Retorna orderbook normalizado"""
        pass

    @abstractmethod
    async def create_order(
        self,
        symbol: str,
        side: str,  # "BUY" | "SELL"
        order_type: str,  # "LIMIT" | "MARKET"
        quantity: Decimal,
        price: Optional[Decimal] = None
    ) -> Dict:
        """Cria ordem e retorna resultado normalizado"""
        pass

    @abstractmethod
    async def cancel_order(self, symbol: str, order_id: str) -> bool:
        """Cancela ordem"""
        pass

    @abstractmethod
    async def get_candles(
        self,
        symbol: str,
        timeframe: str,  # "1m", "5m", "1h", etc.
        limit: int = 100
    ) -> List[Dict]:
        """Retorna candles OHLCV normalizados"""
        pass

    @abstractmethod
    def normalize_symbol(self, symbol: str) -> str:
        """Converte símbolo para formato da exchange
        MB: BTC-BRL
        Binance: BTCUSDT
        """
        pass
```

### MercadoBitcoinExchange (Adapter Concreto)

```python
# src/exchanges/mercado_bitcoin.py (1086 lines)
class MercadoBitcoinExchange(BaseExchange):
    SUPPORTED_TIMEFRAMES = ["1m", "5m", "15m", "30m", "1h", "6h", "1d"]

    async def get_orderbook(self, symbol: str, depth: int = 10) -> Dict:
        # 1. Chama API do MB (formato específico)
        mb_symbol = self.normalize_symbol(symbol)  # "BTC-BRL"
        response = await self.client.get(f"/api/v4/{mb_symbol}/orderbook")

        # 2. Normaliza para formato unificado
        return {
            "symbol": symbol,
            "bids": [
                {"price": Decimal(bid[0]), "quantity": Decimal(bid[1])}
                for bid in response['data']['bids'][:depth]
            ],
            "asks": [
                {"price": Decimal(ask[0]), "quantity": Decimal(ask[1])}
                for ask in response['data']['asks'][:depth]
            ],
            "timestamp": datetime.fromisoformat(response['timestamp'])
        }

    def normalize_symbol(self, symbol: str) -> str:
        # BTC-BRL → BTC-BRL (sem mudança)
        # BTCBRL → BTC-BRL (adiciona hífen)
        if '-' not in symbol:
            return f"{symbol[:3]}-{symbol[3:]}"
        return symbol
```

### BinanceExchange (Adapter Concreto)

```python
# src/exchanges/binance.py
class BinanceExchange(BaseExchange):
    SUPPORTED_TIMEFRAMES = [
        "1s", "1m", "3m", "5m", "15m", "30m",
        "1h", "2h", "4h", "6h", "8h", "12h",
        "1d", "3d", "1w", "1M"
    ]

    async def get_orderbook(self, symbol: str, depth: int = 10) -> Dict:
        # 1. Chama API da Binance (formato específico)
        binance_symbol = self.normalize_symbol(symbol)  # "BTCUSDT"
        response = await self.client.get_order_book(
            symbol=binance_symbol,
            limit=depth
        )

        # 2. Normaliza para MESMO formato que MB
        return {
            "symbol": symbol,
            "bids": [
                {"price": Decimal(bid[0]), "quantity": Decimal(bid[1])}
                for bid in response['bids']
            ],
            "asks": [
                {"price": Decimal(ask[0]), "quantity": Decimal(ask[1])}
                for ask in response['asks']
            ],
            "timestamp": datetime.fromtimestamp(response['E'] / 1000)
        }

    def normalize_symbol(self, symbol: str) -> str:
        # BTC-BRL → BTCBRL (remove hífen)
        # BTC-USDT → BTCUSDT
        return symbol.replace('-', '')
```

### ExchangeFactory (Dependency Injection)

```python
# src/infrastructure/exchanges/exchange_factory.py
class ExchangeFactory:
    @staticmethod
    async def create(exchange_name: str, user: User) -> BaseExchange:
        """Cria exchange com credenciais do usuário"""

        if exchange_name == "mercado_bitcoin":
            credentials = decrypt_credentials(user.encrypted_credentials)
            return MercadoBitcoinExchange(
                api_id=credentials['mb_api_id'],
                api_key=credentials['mb_api_key'],
                account_id=credentials['mb_account_id']
            )

        elif exchange_name == "binance":
            credentials = decrypt_credentials(user.encrypted_credentials)
            return BinanceExchange(
                api_key=credentials['binance_api_key'],
                api_secret=credentials['binance_api_secret'],
                testnet=settings.BINANCE_TESTNET
            )

        else:
            raise UnsupportedExchangeError(f"Exchange {exchange_name} não suportada")
```

### Uso em Workflows (Exchange-Agnostic)

```python
# src/workflows/smart_bot_purchase.py
async def execute_purchase(bot: SmartBot, user: User):
    # Factory cria exchange correta automaticamente
    exchange: BaseExchange = await ExchangeFactory.create(bot.exchange, user)

    # Código genérico - funciona para MB e Binance
    orderbook = await exchange.get_orderbook(bot.symbol, depth=10)

    best_bid = orderbook['bids'][0]['price']  # Formato unificado!
    competitive_price = best_bid + Decimal("0.01")

    order = await exchange.create_order(
        symbol=bot.symbol,
        side="BUY",
        order_type="LIMIT",
        quantity=bot.calculate_quantity(),
        price=competitive_price
    )

    # order sempre tem mesmo formato (MB ou Binance)
    return order
```

## Consequências

### Positivas ✅

1. **Workflows Exchange-Agnostic**
   - ✅ Código de bot funciona para MB e Binance sem mudanças
   - ✅ Fácil adicionar Kraken, Coinbase, etc. (apenas novo adapter)

2. **Testabilidade**
   - ✅ Fácil criar `MockExchange(BaseExchange)` para testes
   ```python
   class MockExchange(BaseExchange):
       async def get_orderbook(self, symbol: str, depth: int) -> Dict:
           return {"bids": [{"price": Decimal("100"), "quantity": Decimal("1")}], ...}
   ```

3. **Normalização Consistente**
   - ✅ Orderbook sempre tem mesmo formato (bids/asks com price/quantity)
   - ✅ Candles sempre OHLCV com timestamps Unix
   - ✅ Orders sempre com exchange_order_id, status, filled_at

4. **Fácil Adicionar Exchanges**
   - ✅ Checklist: Implementar BaseExchange (6 métodos), adicionar ao Factory
   - 🎯 Estimativa: 2-3 dias para nova exchange

### Negativas ⚠️

1. **Workflows Ainda Acoplados ao MB** (Débito Técnico)
   - ⚠️ Código assume formato MB em alguns lugares:
   ```python
   # Problema: hardcoded para MB
   if bot.exchange == "mercado_bitcoin":
       do_mb_specific_thing()
   ```
   - 🔧 **TODO**: Refatorar para usar apenas BaseExchange interface

2. **Binance Ainda em Teste**
   - ⚠️ Adapter funciona, mas workflows têm bugs específicos de Binance
   - ⚠️ Normalização de símbolos incompleta (BTCUSDT vs BTC-USDT)
   - 🔧 **TODO**: Testes de integração com Binance testnet

3. **Complexidade Adicional**
   - ⚠️ Adapter + Normalizer + Factory = mais código que chamar MB direto
   - **Justificativa**: Complexidade vale a pena para suportar múltiplas exchanges

## Alternativas Consideradas

### Alternativa 1: CCXT Library (Rejeitada)
**Prós**:
- Suporta 100+ exchanges out-of-the-box
- Normalização automática

**Contras**:
- ❌ Não suporta async Python bem (apenas sync ou aiohttp custom)
- ❌ Não suporta features específicas de MB (post-only, stop-limit avançado)
- ❌ Debugging difícil (blackbox)

**Razão para Rejeitar**: Controle total > conveniência

### Alternativa 2: If/Else em Workflows (Rejeitada)
```python
if bot.exchange == "mercado_bitcoin":
    orderbook = await get_mb_orderbook()
elif bot.exchange == "binance":
    orderbook = await get_binance_orderbook()
```
**Razão para Rejeitar**: Unmaintainable com 5+ exchanges

## Implementação

**Evidências no Código**:
- [`src/exchanges/base_exchange.py`](../../../crypteras-improved/backend/src/exchanges/base_exchange.py)
- [`src/exchanges/mercado_bitcoin.py`](../../../crypteras-improved/backend/src/exchanges/mercado_bitcoin.py) (1086 lines)
- [`src/exchanges/binance.py`](../../../crypteras-improved/backend/src/exchanges/binance.py)
- [`src/infrastructure/exchanges/exchange_factory.py`](../../../crypteras-improved/backend/src/infrastructure/exchanges/exchange_factory.py)

## Lições Aprendidas

1. **Adapter Pattern Funciona**: MB e Binance coexistem sem conflitos
2. **Normalização é Crítica**: Diferenças sutis (string vs float) causam bugs
3. **Workflows Precisam Refatoração**: Ainda há `if exchange == "mercado_bitcoin"` espalhado

## Referências

- [Design Patterns - Adapter Pattern](https://refactoring.guru/design-patterns/adapter)
- [Issue CRY-20: Multi-Exchange Support](link-to-linear)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Implementação**: 2024-10-01 (CRY-20 v2.2)
**Status Atual**: ⚠️ Em uso, mas workflows precisam refatoração para serem 100% exchange-agnostic.
