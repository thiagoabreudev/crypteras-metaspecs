---
adr_number: "007"
title: "TradingConfig Entity (Depreciado)"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['config', 'technical', 'trading', 'decision', 'architecture', 'deprecated']

spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
---

# ADR-007: TradingConfig Entity (Depreciado)

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Documentar depreciação de TradingConfig e migração para Bots independentes para evitar uso acidental de código legacy.

**Constraints** (limites obrigatórios):
- NUNCA usar TradingConfig em código novo (sempre usar SmartBot, CandleBot ou DCABot)
- NUNCA importar de `src/domain/entities/trading_config.py`
- Código legacy que ainda usa TradingConfig deve ter warning explícito no log
- Migration path documentado para usuarios que ainda tem TradingConfig no banco
- Todas referências a TradingConfig em documentação devem estar marcadas como DEPRECATED

**Non-Goals** (o que NÃO fazer):
- Remover código TradingConfig do repositório (manter para migração gradual)
- Forçar migração automática de TradingConfigs existentes (usuário decide quando migrar)
- Criar compatibilidade entre TradingConfig e novos Bots (são incompatíveis por design)
- Permitir criação de novas TradingConfigs (endpoint POST /trading-config deve retornar 410 Gone)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Deprecação de TradingConfig
:::

## Status
⚠️ **DEPRECIADO** (Substituído por Bots Independentes em CRY-82 v2.8)

## Data
- **Criação**: 2024-09-01 (v2.0)
- **Depreciação**: 2024-11-13 (v2.8 CRY-82)

## Contexto

### Decisão Original (v2.0-v2.7)

Sistema inicialmente tinha **TradingConfig Entity** centralizada:

```python
# DEPRECATED - não usar mais
@dataclass
class TradingConfigEntity:
    user_id: str
    exchange: str  # "mercado_bitcoin" | "binance"
    symbol: str  # "BTC-BRL"

    # Parâmetros compartilhados
    max_position_size: Decimal
    trailing_stop_percentage: Decimal
    soft_stop_loss: Decimal
    hard_stop_loss: Decimal

    # Stats compartilhados
    base_price: Decimal
    total_invested: Decimal
    total_blocked: Decimal
```

**Problema**: **Todas estratégias** (traditional, candle, DCA) compartilhavam mesma config!

```python
# v2.0-v2.7 (RUIM)
config = await config_repository.get_by_user(user_id)

# Traditional bot usa config.max_position_size
await traditional_workflow.execute(config)

# Candle bot USA MESMA config.max_position_size (conflito!)
await candle_workflow.execute(config)

# DCA bot TAMBÉM usa mesma config (caos!)
await dca_workflow.execute(config)
```

**Consequências**:
- ⚠️ Impossível ter Traditional + Candle rodando simultaneamente
- ⚠️ `total_invested` misturava compras de estratégias diferentes
- ⚠️ Mudar `max_position_size` afetava TODAS estratégias

## Decisão: Depreciar TradingConfig

**CRY-82 (v2.8)**: Remover TradingConfig, cada bot é independente.

### Nova Arquitetura (v2.8+)

```python
# SmartBot é independente
smart_bot = SmartBot(
    name="BTC Conservative",
    max_position_size=5000.00,  # Config própria
    stats=SmartBotStats(
        total_invested=1200.00,  # Stats próprios
        total_blocked=300.00
    )
)

# CandleBot é independente
candle_bot = CandleBot(
    name="ETH Swing",
    max_position_size=3000.00,  # Config diferente!
    stats=CandleBotStats(...)
)

# DCABot é independente
dca_bot = DCABot(
    name="BTC Weekly DCA",
    allocated_balance=10000.00,  # Conceito diferente de max_position_size
    stats=DCABotStats(...)
)
```

**Benefícios**:
- ✅ SmartBot, CandleBot, DCABot rodam **simultaneamente** sem conflitos
- ✅ Cada bot tem `total_invested`, `total_blocked` próprios
- ✅ Mudar config de um bot não afeta outros

## Migração (v2.7 → v2.8)

### Script de Migração

```python
# scripts/migrate_cry82.py
async def migrate_trading_config_to_bots():
    configs = await trading_config_repository.get_all()

    for config in configs:
        # 1. Criar SmartBot para cada config antiga
        smart_bot = SmartBot(
            user_id=config.user_id,
            name=f"{config.symbol} Bot",
            exchange=config.exchange,
            symbol=config.symbol,
            max_position_size=config.max_position_size,
            trailing_stop_percentage=config.trailing_stop_percentage,
            soft_stop_loss=config.soft_stop_loss,
            hard_stop_loss=config.hard_stop_loss,
            stats=SmartBotStats(
                base_price=config.base_price,
                total_invested=config.total_invested,
                total_blocked=config.total_blocked
            ),
            operation_mode="CONTINUOUS",  # Assume continuous
            intervals={
                "purchase": 63,
                "sales": 67,
                "check_situation": 33
            }
        )

        await smart_bot_repository.create(smart_bot)

        # 2. Arquivar TradingConfig antiga (não deletar, backup)
        await trading_config_repository.archive(config.id)
```

### Código Afetado

**Antes (v2.0-v2.7)**:
```python
# Workflows recebiam TradingConfig
async def smart_bot_purchase(config: TradingConfigEntity):
    available = config.max_position_size - config.total_invested - config.total_blocked
    # ...
```

**Depois (v2.8+)**:
```python
# Workflows recebem SmartBot
async def smart_bot_purchase(bot: SmartBot):
    available = bot.get_available_funds()  # Método próprio
    # ...
```

## Consequências da Depreciação

### Positivas ✅

1. **Bots Independentes**
   - ✅ User pode ter 3 SmartBots + 2 CandleBots + 1 DCABot simultaneamente
   - ✅ Cada bot com configs diferentes

2. **Stats Isolados**
   - ✅ `smart_bot.stats.total_invested` não mistura com `candle_bot.stats.total_invested`
   - ✅ Fácil rastrear performance por bot

3. **Código Mais Limpo**
   - ✅ Workflows não precisam diferenciar estratégias
   - ✅ Menos if/else

### Negativas ⚠️

1. **Resquícios de Código**
   - ⚠️ **Problema Atual**: Código antigo ainda referencia TradingConfig
   ```python
   # PROBLEMA: código deprecated ainda existe
   if config.max_position_size:  # config não existe mais!
       ...
   ```
   - 🔧 **TODO**: Refatorar todo código que menciona `TradingConfig`

2. **Migração Manual Necessária**
   - ⚠️ Usuários de v2.7 precisaram rodar script de migração
   - ⚠️ Alguns configs antigos perdidos se migração falhou

## Trabalho Remanescente (Débito Técnico)

### TODOs para Remover Completamente TradingConfig

1. ✅ **Criar Entities Independentes**: SmartBot, CandleBot, DCABot (feito CRY-82)
2. ✅ **Migração de Dados**: Script `migrate_cry82.py` (feito)
3. 🔧 **Remover Imports**: Buscar `from .trading_config import TradingConfig` e deletar
4. 🔧 **Refatorar Workflows**: Substituir `config: TradingConfigEntity` por `bot: SmartBot`
5. 🔧 **Deletar Arquivo**: `src/domain/entities/trading_config.py` (manter por enquanto para referência)
6. 🔧 **Deletar Repository**: `TradingConfigRepository` e implementações
7. 🔧 **Atualizar Testes**: Remover testes de TradingConfig

### Comando para Encontrar Resquícios

```bash
# Encontrar todas referências a TradingConfig
grep -r "TradingConfig" backend/src/

# Encontrar imports deprecated
grep -r "from.*trading_config import" backend/src/
```

## Lições Aprendidas

1. **Design Inicial Importa**: TradingConfig parecia boa ideia (centralizar config), mas limitou crescimento
2. **Migração É Custosa**: Quebrar mudança afetou todos workflows
3. **Depreciar Gradualmente**: Manter TradingConfig por 1-2 versões facilitou transição
4. **Documentar Bem**: Este ADR ajuda novos devs entenderem por que código antigo existe

## Referências

- [Issue CRY-82: ExchangeCredentials Refactor](link)
- [Migration Script: `scripts/migrate_cry82.py`](../../../crypteras-improved/scripts/migrate_cry82.py)
- ~~[Código Deprecated: `trading_config.py`](../../../crypteras-improved/backend/src/domain/entities/trading_config.py)~~ (manter por enquanto)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Depreciação**: 2024-11-13 (v2.8 CRY-82)
**Status Atual**: ⚠️ Depreciado, mas código antigo ainda existe. Refatoração em progresso.
