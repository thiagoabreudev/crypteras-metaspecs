---
spec_version: "1.5.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.4.0"
status: "active"
category: "technical"
tags: ['technical', 'claude.meta']
---

# CLAUDE.meta.md - Guia de Desenvolvimento com IA

:::version_info
**Versão**: 1.5.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.5.0**:
- Adicionada referência explícita ao ADR-006 na seção AGNO Framework
- Melhor rastreabilidade de decisões arquiteturais (ADR > Guide)

**v1.4.0**:
- Versão anterior
:::

:::intent
**Goal**: Guiar desenvolvimento com IA mantendo consistência arquitetural, qualidade de código e alinhamento com regras de negócio do sistema de trading.

**Constraints** (limites obrigatórios):
- Respeitar TODOS os ADRs vigentes (especialmente ADR-001 Clean Architecture, ADR-005 Adapter Pattern, ADR-007 TradingConfig Deprecated)
- Manter retrocompatibilidade com código existente (sem breaking changes em minor versions)
- Não alterar contratos públicos (APIs, interfaces, domain entities) sem aprovação explícita
- Usar APENAS stack tecnológica aprovada (Python 3.x + FastAPI + Nuxt.js 3 + MongoDB + Redis + Celery)
- Seguir Clean Architecture rigorosamente (Domain NUNCA depende de Infrastructure)
- PEP8 obrigatório para novo código (snake_case, imports ordenados, line max 100 chars)
- Type hints obrigatórios (todas funções/métodos devem ter tipos explícitos)
- Usar Decimal para valores monetários (NUNCA float)
- Async/await para todas operações I/O
- Testes obrigatórios para workflows (cobertura mínima 80%)

**Non-Goals** (o que NÃO fazer):
- Refatoração ampla de código não relacionado ao escopo da tarefa
- Mudanças de arquitetura sem ADR aprovado (ex: migrar de MongoDB para PostgreSQL)
- Introdução de novas tecnologias/frameworks sem discussão (ex: Django, SQLAlchemy, TypeORM)
- Otimização prematura (performance sem evidência de problema real)
- Código "clever" em detrimento de clareza e manutenibilidade
- Abstrações genéricas sem caso de uso concreto (YAGNI - You Ain't Gonna Need It)
- Remoção de código deprecated sem plano de migração documentado
- Alteração de intervalos co-primos do Celery Beat para números redondos
- Misturar lógica de exchange específica nos workflows (deve usar BaseExchange)
- Usar TradingConfig (deprecated desde v2.8)
- Ignorar circuit breaker, stop-loss ou trailing stop (são salvaguardas críticas)
:::

:::failure_modes
**Falhas Conhecidas de Desenvolvimento com IA**:

1. **PEP8 Violations - Imports no Meio do Código**
   - **Tipo**: hallucination
   - **Descrição**: IA coloca imports no meio de funções/classes ao invés do topo do arquivo
   - **Gatilho**: Adicionar nova dependência durante refatoração
   - **Impacto**: 🟡 Médio (code review identifica, mas polui PRs)
   - **Mitigação**: SEMPRE imports no topo (stdlib → third-party → local). Usar linter automático (flake8, black)
   - **Detecção**: `grep -r "^    import" backend/src/` (indent indica import dentro de função)

2. **Float ao Invés de Decimal para Valores Monetários**
   - **Tipo**: hallucination
   - **Descrição**: IA usa `float` para preços, quantidades ou valores em BRL/BTC
   - **Gatilho**: Qualquer cálculo envolvendo dinheiro ou criptomoedas
   - **Impacto**: 🔴 Crítico (erros de arredondamento causam bugs financeiros)
   - **Mitigação**: SEMPRE `from decimal import Decimal`. NUNCA `price = 155000.50` (use `Decimal("155000.50")`)
   - **Detecção**: Code review + testes unitários com valores decimais críticos

3. **TradingConfig Deprecated - Usar SmartBot Individual**
   - **Tipo**: context_clash
   - **Descrição**: IA tenta usar `TradingConfig` (removido em v2.8) ao invés de configurações individuais por bot
   - **Gatilho**: Implementar features de configuração de trading
   - **Impacto**: 🔴 Crítico (código não compila, imports falham)
   - **Mitigação**: Cada bot (SmartBot, CandleBot, DCABot) tem sua própria config. Ver ADR-007
   - **Detecção**: `grep -r "TradingConfig" backend/src/` (deve retornar 0 resultados em código novo)

4. **Frontend - Reatividade Perdida em Componentes Vue**
   - **Tipo**: hallucination
   - **Descrição**: IA atualiza dados em apenas um componente/composable, esquecendo dependências reativas
   - **Gatilho**: Atualização de estado compartilhado (ex: bots, orders, balances)
   - **Impacto**: 🟡 Médio (UI desatualizada, usuário não vê mudanças)
   - **Mitigação**: Usar `ref`/`reactive` corretamente. Atualizar TODOS os composables que dependem do estado
   - **Detecção**: Teste manual - mudar estado e verificar se TODOS os componentes atualizam

5. **Conexão a Banco/Servidor Errado**
   - **Tipo**: integration
   - **Descrição**: IA hardcoda URLs (`localhost:27017`, `localhost:8000`) ao invés de usar variáveis de ambiente
   - **Gatilho**: Implementar nova integração ou endpoint
   - **Impacto**: 🔴 Crítico (código funciona local mas quebra em produção)
   - **Mitigação**: SEMPRE usar `useRuntimeConfig()` (frontend) ou Pydantic Settings (backend). Ver NEVER_HARDCODE_URLS.md
   - **Detecção**: `grep -r "localhost" frontend/` e `grep -r "localhost" backend/src/`

6. **Deploy Não Autorizado**
   - **Tipo**: security
   - **Descrição**: IA executa `git push`, `docker push` ou deploy em produção sem autorização explícita
   - **Gatilho**: Finalizar implementação de feature
   - **Impacto**: 🔴 Crítico (deploy acidental em produção)
   - **Mitigação**: IA NUNCA deve fazer deploy. Apenas criar PR. Deploy é manual ou via CI/CD aprovado
   - **Detecção**: Revisar comandos executados pela IA antes de aprovar

7. **Regras Hardcoded para Mercado Bitcoin (Não Funcionam na Binance)**
   - **Tipo**: integration
   - **Descrição**: IA assume formato de resposta do Mercado Bitcoin (`response['data']['bids']`) sem usar BaseExchange
   - **Gatilho**: Implementar workflows de compra/venda ou análise de orderbook
   - **Impacto**: 🔴 Crítico (bots com Binance falham completamente)
   - **Mitigação**: SEMPRE usar `BaseExchange` interface. Workflows devem ser exchange-agnostic. Ver ADR-005
   - **Detecção**: Teste com mock de Binance. Buscar `['data']` em workflows (formato específico do MB)

8. **Clean Architecture Violation - Domain Importando Infrastructure**
   - **Tipo**: hallucination
   - **Descrição**: IA importa FastAPI, MongoDB, Celery dentro de `src/domain/`
   - **Gatilho**: Implementar nova entity ou domain service
   - **Impacto**: 🔴 Crítico (viola arquitetura, cria acoplamento)
   - **Mitigação**: Domain APENAS usa stdlib Python + typing. Ver ADR-001
   - **Detecção**: `grep -r "from fastapi" backend/src/domain/` (deve retornar 0 resultados)

9. **AGNO Framework - Implementação Sem Consultar Documentação (ADR-006)**
   - **Tipo**: hallucination
   - **Decisão Arquitetural**: Framework AGNO para agentes de IA (conforme ADR-006 v1.1.0)
   - **Descrição**: IA implementa agentes AGNO usando padrões incorretos ou APIs inexistentes sem consultar docs oficiais
   - **Gatilho**: Implementar ou modificar agentes AI (chat, assistentes, workflows com LLM)
   - **Impacto**: 🔴 Crítico (agentes não funcionam, erros em runtime, padrões incorretos)
   - **Mitigação**: SEMPRE consultar https://docs.agno.com/introduction ANTES de implementar. Seguir exemplos oficiais. Validar imports e métodos disponíveis. Consultar ADR-006 para constraints e decisões
   - **Detecção**: Imports de `agno` falham. Métodos inexistentes. Código não segue padrões da documentação oficial
   - **Exemplo Correto**: Consultar docs para `Agent`, `Tool`, `Workflow`, `Memory` patterns antes de implementar
   - **Referência**: specs/technical/adr/006-agno-framework-ai.md v1.1.0

10. **Nuxt 3 / Vue 3 - Implementação Sem Consultar Documentação**
   - **Tipo**: hallucination
   - **Descrição**: IA usa padrões Vue 2 / Nuxt 2 deprecados ou inventa APIs que não existem em Nuxt 3 / Vue 3
   - **Gatilho**: Implementar componentes, composables, layouts, middleware no frontend
   - **Impacto**: 🔴 Crítico (componentes não renderizam, composables falham, build quebra)
   - **Mitigação**: SEMPRE consultar https://nuxt.com/docs/getting-started/introduction e https://vuejs.org/guide/introduction.html. Usar Composition API (não Options API). Verificar `auto-imports` do Nuxt 3
   - **Detecção**: Build falha. Console mostra warnings de APIs deprecadas. Composables não funcionam (`useAsyncData`, `useFetch`, `useRuntimeConfig`)
   - **Exemplos de Erros Comuns**:
     - ❌ `export default { data() {} }` (Options API - deprecado)
     - ✅ `<script setup>` com `ref()`, `computed()`, `watch()` (Composition API)
     - ❌ `asyncData()` (Nuxt 2)
     - ✅ `useAsyncData()`, `useFetch()` (Nuxt 3)
:::

:::explainability
**Requirement**: ✅ Required (para decisões arquiteturais e de desenvolvimento)

**Output Format**:
IA DEVE explicar decisões seguindo este formato:

```markdown
## 🤖 Decisão de Desenvolvimento

**Decisão**: [O que foi decidido - ex: "Usar `computed` ao invés de `watch` para cálculo de progresso"]

**Source**:
- `specs/technical/CLAUDE.meta.md` v1.4.0 - Seção relevante
- Código existente: `caminho/arquivo.py:linha-linha` (se aplicável)
- ADRs relacionados (se aplicável)

**Rationale**:
1. [Razão principal baseada na spec]
2. [Consistência com código existente]
3. [Performance ou qualidade]
4. [Alinhamento com arquitetura]

**Alternatives Considered**:
1. ❌ [Alternativa 1] - [Por que não foi escolhida]
2. ❌ [Alternativa 2] - [Por que não foi escolhida]
3. ✅ [Escolhida] - [Por que foi escolhida]

**Trade-offs**:
- ✅ Pro: [Benefício 1]
- ✅ Pro: [Benefício 2]
- ⚠️ Con: [Desvantagem se houver]

**Audit Trail**:
- Timestamp: [ISO 8601 format]
- Specs Consultadas: [nome.md v1.x.x, outro.md v2.x.x]
- Código Analisado: [arquivos com line numbers]
```

**Quando Explicar** (gatilhos obrigatórios):
1. **Escolha de padrão arquitetural**: Clean Architecture, camadas, injeção de dependências
2. **Decisão entre múltiplas alternativas válidas**: `Decimal` vs `float`, `async/await` vs sync, LIMIT vs MARKET
3. **Trade-off significativo identificado**: Performance vs manutenibilidade, complexidade vs flexibilidade
4. **Desvio de padrão existente**: Requer justificativa forte e explícita
5. **Introdução de nova dependência**: Biblioteca, framework, package
6. **Mudança de assinatura de API pública**: Métodos, classes, interfaces do Domain
7. **Implementação de failure mode mitigation**: Quando corrigindo falhas conhecidas listadas em `:::failure_modes`
8. **Escolha de estratégia de teste**: Unit vs integration, mocks vs real implementations
9. **Decisão sobre AGNO ou Nuxt/Vue patterns**: SEMPRE consultar docs oficiais primeiro
10. **Violação potencial de Non-Goals**: Quando necessário justificar exceção

**Audit Trail Obrigatório**:
- Timestamp da decisão (ISO 8601)
- Specs consultadas (nome + versão exata)
- Arquivos de código analisados (com line numbers se relevante)
- Contexto usado (ADRs, failure modes, constraints específicos)

**Exemplo Completo**:

```markdown
## 🤖 Decisão de Desenvolvimento

**Decisão**: Usar `Decimal` para cálculo de `get_available_funds()` ao invés de `float`

**Source**:
- `specs/technical/CLAUDE.meta.md` v1.4.0 - Failure Mode #2 (Float vs Decimal)
- `specs/technical/CLAUDE.meta.md` v1.4.0 - Constraints "Usar Decimal para valores monetários"
- Código existente: `backend/src/domain/entities/smart_bot.py:209-219`

**Rationale**:
1. **Failure Mode Mitigation**: Float causa erros de arredondamento em valores monetários (Failure Mode #2)
2. **Constraint Obrigatório**: Spec exige "Usar Decimal para valores monetários (NUNCA float)"
3. **Código Existente**: SmartBot já usa Decimal em `max_position_size`, `total_invested`, `total_blocked`
4. **Precisão Crítica**: Fundos disponíveis afetam decisões de compra - imprecisão causaria bugs financeiros

**Alternatives Considered**:
1. ❌ `float` - Violaria constraint obrigatório, causaria bugs de arredondamento (ex: 155000.50 → 155.00049999999998)
2. ❌ Arredondar para 2 casas decimais - Ainda suscetível a erros acumulativos em múltiplas operações
3. ✅ `Decimal` - Precisão arbitrária, padrão para valores monetários, consistente com codebase

**Trade-offs**:
- ✅ Pro: Precisão financeira correta
- ✅ Pro: Consistente com entities existentes
- ✅ Pro: Evita Failure Mode #2
- ⚠️ Con: Performance ~10% mais lenta que float (aceitável para valores monetários)

**Audit Trail**:
- Timestamp: 2025-12-25T10:30:00Z
- Specs Consultadas: CLAUDE.meta.md v1.4.0
- Código Analisado: smart_bot.py:209-219, trading_config.py (deprecated)
- Failure Modes Aplicados: FM#2 (Float vs Decimal)
```

**Quando NÃO Explicar** (decisões triviais):
- Nomes de variáveis descritivas óbvias
- Formatação de código (PEP8 já define)
- Ordem de imports (stdlib → third-party → local é padrão)
- Uso de type hints (obrigatório por constraint)
- Decisões já amplamente documentadas em código
:::

:::breaking_changes
**v1.4.0**:
- Adicionada seção `:::explainability` com requisitos obrigatórios para decisões de desenvolvimento
- Definidos 10 gatilhos obrigatórios de explainability
- Incluído exemplo completo de explicação de decisão
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.3.0**:
- Adicionados 2 novos failure modes: AGNO framework (docs.agno.com) e Nuxt 3 / Vue 3 (nuxt.com/docs)
- Total: 10 falhas conhecidas de desenvolvimento com IA
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.2.0**:
- Adicionada seção `:::failure_modes` com 8 falhas conhecidas de desenvolvimento com IA
- Documentadas falhas reais: PEP8, Float vs Decimal, TradingConfig deprecated, Reatividade Vue, Hardcoded URLs, Deploy não autorizado, MB vs Binance, Clean Architecture
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Metadados e instruções para Claude AI
:::

Este arquivo fornece orientação ao Claude Code (claude.ai/code) e outros sistemas de IA ao trabalhar com código neste repositório.

## Visão Geral do Projeto

Sistema de trading automatizado de criptomoedas usando **Clean Architecture**, **AGNO** para IA, e **Celery** para workflows distribuídos. Suporta 3 tipos de bots (SmartBot, CandleBot, DCABot) operando em Mercado Bitcoin e Binance.

**Repositório Real**: `/Users/thiagoabreu/workspace/crypteras-improved`
**Stack**: Python 3.x + FastAPI + Nuxt.js 3 + MongoDB + Redis + Celery
**Deploy**: Docker Swarm (DigitalOcean)

---

## 🎯 Prioridade #1: Lógica dos Bots

### SmartBot (Prioridade Máxima)

**Arquivo Principal**: `backend/src/domain/entities/smart_bot.py` (716 lines)

**Conceito**: Bot flexível para trading manual/scheduled/continuous com gestão de risco avançada.

#### Modos de Operação
```python
class OperationMode(str, Enum):
    MANUAL = "MANUAL"        # User aciona manualmente
    SCHEDULED = "SCHEDULED"  # Cron-based (ex: "0 9 * * 1-5" = 9 AM dias úteis)
    CONTINUOUS = "CONTINUOUS"  # Interval-based (ex: compra a cada 63s)
```

#### Lógica de Negócio Chave

**1. Verificar Se Pode Comprar** (`can_execute_purchase()`):
```python
def can_execute_purchase(self) -> bool:
    """
    Regras:
    - Bot deve estar ACTIVE
    - Circuit breaker não pode estar ativo
    - Deve ter fundos disponíveis (max_position_size - invested - blocked)
    """
    if self.status != BotStatus.ACTIVE:
        return False

    if self.circuit_breaker_active:
        return False

    available = self.get_available_funds()
    return available > Decimal("0")
```

**2. Calcular Fundos Disponíveis** (`get_available_funds()`):
```python
def get_available_funds(self) -> Decimal:
    """
    Fórmula: max_position_size - total_invested - total_blocked

    Exemplo:
    - max_position_size = R$ 5000
    - total_invested = R$ 3000 (ordens filled)
    - total_blocked = R$ 500 (ordens pending)
    → Disponível = R$ 1500
    """
    return (
        self.max_position_size
        - self.stats.total_invested
        - self.stats.total_blocked
    )
```

**3. Verificar Stop-Loss** (`should_trigger_stop_loss()`):
```python
def should_trigger_stop_loss(self, current_price: Decimal) -> bool:
    """
    Regras:
    - Hard stop-loss: Perda >= 5% → Venda a MERCADO imediata
    - Soft stop-loss: 2% <= Perda < 5% → Venda urgente (LIMIT competitivo)

    Cálculo:
    variation = ((current_price - base_price) / base_price) * 100
    """
    if not self.stats.base_price:
        return False

    variation = (
        (current_price - self.stats.base_price) / self.stats.base_price
    ) * 100

    return variation <= -self.hard_stop_loss  # Ex: -5%
```

**4. Trailing Stop** (`adjust_trailing_stop()`):
```python
def adjust_trailing_stop(self, current_price: Decimal):
    """
    Se lucro >= 3% (trailing_stop_percentage), ajusta base_price para cima.

    Exemplo:
    - base_price = R$ 100.000
    - current_price = R$ 103.500 (+3.5%)
    → base_price = R$ 103.500 (novo piso)

    Próxima vez que cair 3%, vende em R$ 100.395 (lucro garantido)
    """
    if not self.stats.base_price:
        return

    variation = (
        (current_price - self.stats.base_price) / self.stats.base_price
    ) * 100

    if variation >= self.trailing_stop_percentage:
        self.stats.base_price = current_price
        self.stats.trailing_stop_adjusted_at = datetime.utcnow()
```

**5. Circuit Breaker** (`activate_circuit_breaker()`):
```python
def activate_circuit_breaker(self):
    """
    Ativa após N falhas consecutivas (ex: 3 ordens recusadas).
    Pausa bot até reset manual ou até meia-noite.

    Previne: Spam de ordens inválidas, waste de fees
    """
    self.circuit_breaker_active = True
    self.circuit_breaker_activated_at = datetime.utcnow()
    self.status = BotStatus.PAUSED
```

### CandleBot (Prioridade Alta)

**Arquivo Principal**: `backend/src/domain/entities/candle_bot.py` (547 lines)

**Conceito**: Bot baseado em análise técnica de candles (RSI, MA, BB, VWAP, MFI, etc.).

#### Indicadores Suportados (10 total)
```python
@dataclass
class IndicatorsConfig:
    # Trend
    ma_enabled: bool = False
    ma_period: int = 20  # 5-200
    ema_enabled: bool = False

    # Momentum
    rsi_enabled: bool = False
    rsi_period: int = 14  # 2-50
    stoch_rsi_enabled: bool = False

    # Volatility
    bb_enabled: bool = False  # Bollinger Bands
    bb_period: int = 20  # 10-50
    atr_enabled: bool = False

    # Volume
    vwap_enabled: bool = False
    mfi_enabled: bool = False  # Money Flow Index
    obv_enabled: bool = False  # On-Balance Volume
    volume_ma_enabled: bool = False
```

#### Lógica de Geração de Sinais

**1. Consolidar Indicadores** (`evaluate_signal()`):
```python
def evaluate_signal(self, indicators: Dict) -> Signal:
    """
    Analisa múltiplos indicadores e retorna sinal consolidado.

    Regras:
    - BUY: RSI < 30 + Preço < Lower BB + Volume acima da média
    - SELL: RSI > 70 + Preço > Upper BB + Lucro >= take_profit%
    - HOLD: Sinais conflitantes ou confiança baixa
    """
    buy_signals = 0
    sell_signals = 0
    total_indicators = 0

    # RSI
    if self.config.indicators.rsi_enabled:
        total_indicators += 1
        if indicators['rsi'] < 30:
            buy_signals += 1
        elif indicators['rsi'] > 70:
            sell_signals += 1

    # Bollinger Bands
    if self.config.indicators.bb_enabled:
        total_indicators += 1
        if indicators['price'] < indicators['lower_bb']:
            buy_signals += 1
        elif indicators['price'] > indicators['upper_bb']:
            sell_signals += 1

    # Decisão final
    if buy_signals >= total_indicators * 0.6:
        return Signal(signal="BUY", confidence=self._calculate_confidence(buy_signals, total_indicators))
    elif sell_signals >= total_indicators * 0.6:
        return Signal(signal="SELL", confidence=self._calculate_confidence(sell_signals, total_indicators))
    else:
        return Signal(signal="HOLD", confidence="LOW")
```

**2. Validação de Take Profit / Stop Loss**:
```python
def should_take_profit(self, current_price: Decimal) -> bool:
    """
    Verifica se deve vender por lucro.

    Exemplo:
    - average_entry_price = R$ 100.000 (preço médio de compra)
    - current_price = R$ 105.000
    - take_profit_percentage = 5%
    → Vende (lucro = 5%)
    """
    if not self.stats.average_entry_price:
        return False

    profit = ((current_price - self.stats.average_entry_price) / self.stats.average_entry_price) * 100
    return profit >= self.config.take_profit_percentage
```

### DCABot (Prioridade Média)

**Arquivo Principal**: `backend/src/domain/entities/dca_bot.py` (318 lines)

**Conceito**: Dollar Cost Averaging - compra fixa em intervalos regulares.

#### Lógica de Execução

**1. Verificar Se Deve Executar** (`should_execute_now()`):
```python
def should_execute_now(self) -> bool:
    """
    Verifica se chegou hora de comprar baseado em next_execution.

    Exemplo:
    - frequency = "7d" (semanal)
    - next_execution = 2024-12-31 09:00:00
    - now = 2024-12-31 09:05:00
    → True (executar)
    """
    if self.status != BotStatus.ACTIVE:
        return False

    if not self.stats.next_execution:
        return True  # Primeira execução

    return datetime.utcnow() >= self.stats.next_execution
```

**2. Calcular Próxima Execução**:
```python
def calculate_next_execution(self):
    """
    Calcula timestamp da próxima compra baseado em frequency.

    Frequências:
    - "1h" → +1 hora
    - "6h" → +6 horas
    - "24h" → +1 dia
    - "7d" → +7 dias
    - "30d" → +30 dias
    """
    frequency_map = {
        "1h": timedelta(hours=1),
        "6h": timedelta(hours=6),
        "24h": timedelta(days=1),
        "7d": timedelta(days=7),
        "30d": timedelta(days=30)
    }

    delta = frequency_map[self.config.frequency]
    self.stats.next_execution = datetime.utcnow() + delta
```

**3. Atualizar Preço Médio Ponderado**:
```python
def update_average_price(self, purchase_price: Decimal, purchase_quantity: Decimal):
    """
    Atualiza preço médio após compra.

    Fórmula:
    new_avg = (old_qty * old_price + new_qty * new_price) / (old_qty + new_qty)

    Exemplo:
    - Compra 1: 0.001 BTC @ R$ 150.000 = R$ 150
    - Compra 2: 0.001 BTC @ R$ 160.000 = R$ 160
    → Média = (0.001*150000 + 0.001*160000) / 0.002 = R$ 155.000
    """
    old_total_value = self.stats.current_holdings * self.stats.average_price
    new_total_value = old_total_value + (purchase_quantity * purchase_price)
    self.stats.current_holdings += purchase_quantity
    self.stats.average_price = new_total_value / self.stats.current_holdings
```

---

## 🛠️ Padrões de Código

### 1. Clean Architecture - SEMPRE Respeitar Camadas

**Domain Layer** (`src/domain/`):
- ❌ **NUNCA** importar FastAPI, Celery, MongoDB, AGNO
- ✅ **APENAS** stdlib Python + typing
- ✅ Lógica de negócio pura

```python
# ✅ CORRETO (Domain)
from dataclasses import dataclass
from decimal import Decimal
from datetime import datetime

@dataclass
class SmartBot:
    def can_execute_purchase(self) -> bool:
        # Lógica pura, sem dependências externas
        return self.status == BotStatus.ACTIVE

# ❌ ERRADO (Domain)
from fastapi import HTTPException  # NÃO!
from motor.motor_asyncio import AsyncIOMotorClient  # NÃO!
```

**Application Layer** (`src/application/`):
- ✅ Depende **apenas** do Domain
- ✅ Orquestra Domain Services
- ❌ NÃO conhece MongoDB, FastAPI diretamente

```python
# ✅ CORRETO (Application)
class CreateSmartBotUseCase:
    def __init__(self, bot_repository: ISmartBotRepository):  # Interface do Domain
        self.repository = bot_repository

    async def execute(self, user_id: str, config: dict) -> SmartBot:
        bot = SmartBot(...)  # Domain entity
        return await self.repository.create(bot)  # Interface
```

**Infrastructure Layer** (`src/infrastructure/`):
- ✅ Implementa interfaces do Domain
- ✅ Único lugar com MongoDB, FastAPI, Celery

```python
# ✅ CORRETO (Infrastructure)
from motor.motor_asyncio import AsyncIOMotorClient

class SmartBotRepositoryImpl(ISmartBotRepository):
    def __init__(self, mongo_client: AsyncIOMotorClient):
        self.collection = mongo_client.crypteras_trading.smart_bots

    async def create(self, bot: SmartBot) -> SmartBot:
        doc = {"name": bot.name, ...}
        await self.collection.insert_one(doc)
        return bot
```

### 2. PEP8 - SEMPRE Seguir

**Problema Atual**: Código não segue PEP8 (débito técnico).

**O Que Fazer**:
- ✅ **Nomes de variáveis**: `snake_case` (não `camelCase`)
- ✅ **Nomes de classes**: `PascalCase`
- ✅ **Constantes**: `UPPER_SNAKE_CASE`
- ✅ **Indentação**: 4 espaços (não tabs)
- ✅ **Linha máxima**: 100 caracteres (não 120+)
- ✅ **Imports**: Ordem correta (stdlib → third-party → local)

```python
# ✅ CORRETO (PEP8)
from datetime import datetime  # stdlib
from decimal import Decimal

from fastapi import FastAPI  # third-party
from motor.motor_asyncio import AsyncIOMotorClient

from src.domain.entities.smart_bot import SmartBot  # local

class SmartBotRepository:
    def __init__(self, mongo_client: AsyncIOMotorClient):
        self.mongo_client = mongo_client

    async def get_active_bots(self) -> list[SmartBot]:
        bots = await self.collection.find({"status": "ACTIVE"}).to_list(100)
        return [self._to_entity(bot) for bot in bots]

# ❌ ERRADO (Não-PEP8)
class smartBotRepository:  # Nome errado
    def __init__(self,mongoClient):  # Espaço faltando
        self.MongoClient=mongoClient  # Nome errado, espaço faltando

    async def getActiveBots(self)->list[SmartBot]:  # camelCase, espaço faltando
        Bots=await self.collection.find({"status":"ACTIVE"}).to_list(100)  # Nome errado
        return [self._toEntity(Bot) for Bot in Bots]  # Nome errado
```

### 3. Type Hints - SEMPRE Usar

```python
# ✅ CORRETO
from decimal import Decimal
from typing import Optional, List, Dict

async def calculate_competitive_price(
    orderbook: Dict[str, List[Dict]],
    side: str,
    offset: Decimal = Decimal("0.01")
) -> Decimal:
    if side == "BUY":
        best_bid = Decimal(orderbook['bids'][0]['price'])
        return best_bid + offset
    else:
        best_ask = Decimal(orderbook['asks'][0]['price'])
        return best_ask - offset

# ❌ ERRADO (Sem types)
async def calculate_competitive_price(orderbook, side, offset=0.01):
    if side == "BUY":
        return orderbook['bids'][0]['price'] + offset
    # ...
```

### 4. Decimal para Dinheiro - SEMPRE

**⚠️ CRÍTICO**: NUNCA usar `float` para valores monetários!

```python
from decimal import Decimal

# ✅ CORRETO
price = Decimal("155000.50")
quantity = Decimal("0.001")
total = price * quantity  # Decimal("155.0005")

# ❌ ERRADO
price = 155000.50  # float tem erros de arredondamento!
quantity = 0.001
total = price * quantity  # 155.00049999999998 (impreciso!)
```

### 5. Async/Await - SEMPRE em I/O

```python
# ✅ CORRETO (async para I/O)
async def get_balance(self, asset: str) -> Decimal:
    response = await self.client.get(f"/balance/{asset}")  # I/O bound
    return Decimal(response['balance'])

# ❌ ERRADO (sync para I/O)
def get_balance(self, asset: str) -> Decimal:
    response = self.client.get(f"/balance/{asset}")  # Bloqueia!
    return Decimal(response['balance'])
```

---

## 🧪 Abordagens de Teste

### Prioridade: Testar Workflows (Gap Atual)

**Problema**: Workflows insuficientemente testados (~60% cobertura).

#### 1. Testes de Domain Entities (Padrão)

```python
# tests/unit/domain/test_smart_bot.py
from decimal import Decimal
from src.domain.entities.smart_bot import SmartBot, BotStatus

def test_can_execute_purchase_when_active():
    bot = SmartBot(
        name="Test Bot",
        status=BotStatus.ACTIVE,
        max_position_size=Decimal("5000"),
        stats=SmartBotStats(total_invested=Decimal("2000"), total_blocked=Decimal("500"))
    )

    assert bot.can_execute_purchase() == True  # R$ 2500 disponível

def test_cannot_execute_purchase_when_circuit_breaker_active():
    bot = SmartBot(
        name="Test Bot",
        status=BotStatus.ACTIVE,
        circuit_breaker_active=True,  # Bloqueado
        max_position_size=Decimal("5000")
    )

    assert bot.can_execute_purchase() == False
```

#### 2. Testes de Workflows (PRIORIZAR)

```python
# tests/integration/workflows/test_smart_bot_purchase_workflow.py
import pytest
from src.workflows.smart_bot_purchase import SmartBotPurchaseWorkflow
from tests.mocks.mock_exchange import MockExchange

@pytest.mark.asyncio
async def test_purchase_workflow_creates_order_successfully():
    # Arrange
    bot = SmartBot(name="Test", status=BotStatus.ACTIVE, max_position_size=Decimal("5000"))
    user = User(id="user_123")
    mock_exchange = MockExchange(orderbook={
        'bids': [{'price': Decimal("155000"), 'quantity': Decimal("1")}],
        'asks': [{'price': Decimal("155100"), 'quantity': Decimal("1")}]
    })

    workflow = SmartBotPurchaseWorkflow(
        bot_repository=MockBotRepository(),
        exchange_factory=MockExchangeFactory(mock_exchange)
    )

    # Act
    result = await workflow.execute(bot, user)

    # Assert
    assert result.success == True
    assert result.order.price == Decimal("155000.01")  # best_bid + 0.01
    assert result.order.side == "BUY"
```

#### 3. Mocks Úteis

```python
# tests/mocks/mock_exchange.py
class MockExchange(BaseExchange):
    def __init__(self, orderbook: Dict = None, balance: Decimal = None):
        self.orderbook = orderbook or {
            'bids': [{'price': Decimal("100"), 'quantity': Decimal("1")}],
            'asks': [{'price': Decimal("101"), 'quantity': Decimal("1")}]
        }
        self.balance = balance or Decimal("10000")

    async def get_orderbook(self, symbol: str, depth: int = 10) -> Dict:
        return self.orderbook

    async def get_balance(self, asset: str) -> Decimal:
        return self.balance

    async def create_order(self, symbol: str, side: str, order_type: str, quantity: Decimal, price: Decimal = None) -> Dict:
        return {
            "exchange_order_id": "MOCK-123",
            "status": "FILLED",
            "price": price,
            "quantity": quantity
        }
```

---

## ⚠️ Pegadinhas Comuns

### 1. TradingConfig Deprecated - NÃO Usar

```python
# ❌ ERRADO (código antigo)
from src.domain.entities.trading_config import TradingConfig  # DEPRECATED!

config = await trading_config_repository.get_by_user(user_id)

# ✅ CORRETO (v2.8+)
from src.domain.entities.smart_bot import SmartBot

bot = await smart_bot_repository.get_by_id(bot_id)
```

**Por quê?**: TradingConfig foi depreciado em CRY-82 (v2.8). Cada bot agora é independente.

**Ver**: [ADR-007: TradingConfig Deprecated](adr/007-trading-config-deprecated.md)

### 2. Workflows Acoplados ao Mercado Bitcoin

```python
# ❌ ERRADO (acoplado ao MB)
if bot.exchange == "mercado_bitcoin":
    orderbook = await mb_exchange.get_orderbook(symbol)
    best_bid = Decimal(orderbook['data']['bids'][0][0])  # Formato MB

# ✅ CORRETO (exchange-agnostic)
exchange: BaseExchange = await ExchangeFactory.create(bot.exchange, user)
orderbook = await exchange.get_orderbook(bot.symbol)  # Formato normalizado
best_bid = orderbook['bids'][0]['price']  # Funciona para MB e Binance
```

**Por quê?**: Workflows devem funcionar para qualquer exchange (MB, Binance, futuras).

**Ver**: [ADR-005: Adapter Pattern](adr/005-adapter-pattern-exchanges.md)

### 3. sync_orders Pode Falhar - Sempre Validar

```python
# ❌ ERRADO (assume sync sempre funciona)
bot.stats.total_invested += order.amount
await bot_repository.update(bot)

# ✅ CORRETO (valida se ordem foi sincronizada)
synced_order = await order_repository.get_by_exchange_id(order.exchange_order_id)
if synced_order and synced_order.status == "FILLED":
    bot.stats.total_invested += synced_order.amount
    await bot_repository.update(bot)
else:
    logger.warning(f"Ordem {order.exchange_order_id} ainda não sincronizada")
```

**Por quê?**: `sync_orders` workflow tem ~8% taxa de falha. Ordens podem não estar atualizadas.

**Ver**: [TROUBLESHOOTING.md](TROUBLESHOOTING.md#sync_orders-failures)

### 4. Version Conflicts - Usar Optimistic Locking

```python
# ❌ ERRADO (pode causar version conflict)
bot = await bot_repository.get_by_id(bot_id)
await asyncio.sleep(5)  # Outro processo pode alterar bot!
bot.stats.total_invested += 100
await bot_repository.update(bot)  # Conflito!

# ✅ CORRETO (distributed lock)
from redis import Redis

redis_client = Redis.from_url(settings.CELERY_BROKER_URL)
lock_key = f"lock:smart_bot:{bot_id}"

with redis_client.lock(lock_key, timeout=30):
    bot = await bot_repository.get_by_id(bot_id)
    bot.stats.total_invested += 100
    await bot_repository.update(bot)  # Seguro
```

**Por quê?**: Múltiplos Celery workers podem atualizar mesmo bot simultaneamente.

**Ver**: [ADR-002: Celery + Redis](adr/002-celery-redis-migration.md)

### 5. Intervalos Co-Primos - Não Mudar para Números Redondos

```python
# ❌ ERRADO (causa picos de carga)
beat_schedule = {
    'sync-orders': {'schedule': 60.0},
    'smart-buy': {'schedule': 60.0},  # Todos a cada 60s!
    'smart-sell': {'schedule': 60.0},
}

# ✅ CORRETO (co-primos)
beat_schedule = {
    'sync-orders': {'schedule': 42.0},  # 2 × 3 × 7
    'smart-buy': {'schedule': 63.0},    # 3 × 3 × 7
    'smart-sell': {'schedule': 67.0},   # Primo
}
```

**Por quê?**: Números redondos causam picos de CPU/RAM. Co-primos distribuem carga uniformemente.

**Ver**: [ADR-004: Co-Prime Intervals](adr/004-coprime-intervals.md)

---

## 📚 Referências Rápidas

### Arquivos Mais Importantes (Top 10)

1. **`backend/src/domain/entities/smart_bot.py`** (716 lines) - Lógica SmartBot
2. **`backend/src/domain/entities/candle_bot.py`** (547 lines) - Lógica CandleBot
3. **`backend/src/domain/entities/dca_bot.py`** (318 lines) - Lógica DCABot
4. **`backend/src/workflows/sync_orders.py`** - Sincronização de ordens (crítico)
5. **`backend/src/workflows/smart_bot_purchase.py`** - Compra SmartBot
6. **`backend/src/workflows/smart_bot_sales.py`** - Venda SmartBot
7. **`backend/src/exchanges/base_exchange.py`** - Interface exchange
8. **`backend/src/infrastructure/celery/celeryconfig.py`** - Configuração Celery
9. **`backend/main.py`** (1685 lines) - FastAPI app + rotas
10. **`frontend/composables/useSmartBots.ts`** - State management frontend

### ADRs Obrigatórios para Entender Sistema

- **[ADR-001: Clean Architecture](adr/001-clean-architecture.md)** - Estrutura de camadas
- **[ADR-002: Celery + Redis](adr/002-celery-redis-migration.md)** - Workflows distribuídos
- **[ADR-005: Adapter Pattern](adr/005-adapter-pattern-exchanges.md)** - Multi-exchange
- **[ADR-007: TradingConfig Deprecated](adr/007-trading-config-deprecated.md)** - Código antigo

### Próximos Documentos

- **[CODEBASE_GUIDE.md](CODEBASE_GUIDE.md)** - Navegação detalhada do código
- **[BUSINESS_LOGIC.md](BUSINESS_LOGIC.md)** - Regras de negócio completas
- **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** - Problemas comuns e soluções

---

**Última Atualização**: 2024-12-24
**Versão**: 1.0 (baseado em análise de código real)
