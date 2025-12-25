---
adr_number: "001"
title: "Adoção de Clean Architecture"
date: "2025-12-25"
status: "accepted"
deciders: ["tech-lead"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ['clean', 'technical', 'decision', 'architecture']

spec_version: "1.2.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.1.0"
---

# ADR-001: Adoção de Clean Architecture

:::version_info
**Versão**: 1.2.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Estabelecer separação estrita entre lógica de negócio (Domain) e detalhes de implementação (Infrastructure) para facilitar testes, manutenção e evolução do sistema.

**Constraints** (limites obrigatórios):
- Domain Layer NUNCA pode importar FastAPI, Celery, MongoDB, AGNO ou qualquer framework externo
- Domain Layer deve usar APENAS stdlib Python (decimal, datetime, enum, dataclasses, typing)
- Toda lógica de negócio (cálculos de stop-loss, trailing stop, circuit breaker) deve residir em Domain Entities
- Application Layer depende APENAS de Domain (nunca de Infrastructure diretamente)
- Infrastructure Layer implementa interfaces definidas no Domain (Dependency Inversion Principle)
- Workflows devem orquestrar Domain Services e Use Cases (sem lógica de negócio inline)
- Todas mudanças estruturais de camadas requerem ADR novo (ex: adicionar nova camada "Presentation")

**Non-Goals** (o que NÃO fazer):
- Criar camadas adicionais além das 3 definidas (Domain, Application, Infrastructure)
- Permitir exceções de "dependência invertida" (ex: Domain importando Infrastructure em casos especiais)
- Misturar código de diferentes camadas no mesmo arquivo
- Usar hexagonal architecture ou onion architecture (escolhemos Clean Architecture especificamente)
- Refatorar código legado que ainda funciona apenas para "seguir Clean Architecture" (fazer gradualmente)
- Criar abstrações prematuras no Domain (apenas quando há necessidade real de inversão)
:::

:::explainability
**Requirement**: ✅ Required (decisão arquitetural crítica)

**Como IA Deve Usar Este ADR**:

ADRs (Architecture Decision Records) **JÁ SÃO** explicações de decisões arquiteturais. Este ADR documenta:
- **Context**: Por que a decisão foi necessária
- **Decision**: O que foi decidido
- **Consequences**: Impactos positivos e negativos
- **Alternatives Considered**: Outras opções avaliadas

**Quando IA Deve Referenciar Este ADR**:
1. **Criar nova entidade no Domain**: Deve seguir regras de zero dependencies
2. **Implementar Use Case**: Deve depender apenas de Domain interfaces
3. **Adicionar integração externa**: Implementar como Infrastructure Layer
4. **Refatorar código acoplado**: Migrar para camadas apropriadas
5. **Resolver Failure Mode #8** (CLAUDE.meta.md): Clean Architecture violations

**Output Format ao Aplicar Este ADR**:

```markdown
## 🤖 Decisão Arquitetural

**Decisão**: [Implementação específica - ex: "Criar SmartBotRepository interface no Domain"]

**Source**:
- ADR-001: Adoção de Clean Architecture v1.2.0
- Constraint: "Domain Layer NUNCA pode importar FastAPI, Celery, MongoDB"
- Consequência Positiva: "Testabilidade Excelente"

**Rationale**:
1. **ADR-001 Define**: Domain deve ter interfaces, Infrastructure implementa
2. **Testabilidade**: Interface permite mock em testes
3. **Dependency Inversion**: Domain não depende de detalhes (MongoDB)

**Implementation**:
- **Domain**: `src/domain/repositories/smart_bot_repository.py` (interface abstrata)
- **Infrastructure**: `src/infrastructure/repositories/smart_bot_repository_impl.py` (MongoDB)

**ADR Compliance Check**:
- ✅ Domain usa apenas stdlib Python
- ✅ Infrastructure implementa interface do Domain
- ✅ Application depende apenas de interfaces
- ❌ Breaking Rules?: Não

**Audit Trail**:
- ADR Consultado: ADR-001 v1.2.0
- Constraints Aplicados: Zero dependencies no Domain
- Camadas Afetadas: Domain + Infrastructure
```

**Gatilhos Obrigatórios para Referenciar ADR-001**:
1. Criar nova entity/value object/domain service
2. Adicionar import em qualquer arquivo Domain
3. Implementar repository, exchange adapter, external API
4. Refatorar código que viola camadas
5. Criar novo workflow ou use case
6. Adicionar nova dependência ao projeto
7. Revisar PR que modifica estrutura de camadas

**ADR Compliance Checklist** (IA deve validar):
```python
# Domain Layer - DEVE estar livre de frameworks
✅ Apenas imports: decimal, datetime, enum, dataclasses, typing, abc
❌ NUNCA: fastapi, celery, motor, pymongo, agno, openai

# Application Layer - DEVE depender apenas de Domain
✅ Imports de src.domain.*
❌ NUNCA: Imports diretos de src.infrastructure.*

# Infrastructure Layer - ÚNICO lugar com frameworks
✅ Implementa interfaces do Domain
✅ Usa FastAPI, MongoDB, Celery, etc
```

**Quando NÃO Precisa Explicar** (já coberto pelo ADR):
- Decisão já documentada neste ADR (não duplicar)
- Seguir padrão arquitetural já estabelecido
- Implementação trivial de interface Domain
:::

:::breaking_changes
**v1.2.0**:
- Adicionada seção `:::explainability` com instruções de como IA deve usar este ADR
- Definidos gatilhos obrigatórios para referenciar ADR-001
- Incluído ADR Compliance Checklist para validação automática
- Incrementada versão MINOR conforme MetaCerta (adição de conteúdo não-breaking)

**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- ADR: Adoção de Clean Architecture
:::

## Status
✅ **Aceito** (Implementado desde v2.0)

## Data
2024-09-01 (retroativo - decisão implementada no início do projeto v2.0)

## Contexto

### Problema
O sistema legado (NestJS/TypeScript) tinha problemas de acoplamento:
- Lógica de negócio misturada com código de framework
- Difícil testar componentes isoladamente
- Mudanças em bibliotecas externas impactavam o core
- Código de trading espalhado entre controllers, services, repositories

### Motivadores para Mudança
1. **Complexidade do Domínio**: Trading envolve regras complexas (trailing stop, stop-loss multinível, circuit breaker)
2. **Mudanças Frequentes**: Equipe precisa iterar rapidamente em estratégias de trading
3. **Testabilidade**: Código de trading precisa ser testado sem depender de APIs externas
4. **Manutenibilidade**: Novos desenvolvedores precisam entender o sistema rapidamente

## Decisão

Adotar **Clean Architecture** (Uncle Bob) com separação estrita em 3 camadas:

### 1. Domain Layer (Camada Interna - Zero Dependências)
**Responsabilidades**:
- Entities: `SmartBot`, `CandleBot`, `DCABot`, `User`, `Subscription`
- Value Objects: `TradingContext`, `OperationMode`, `IndicatorsConfig`
- Domain Services: `RiskManagement`, `TechnicalAnalysisService`, `AdaptivePricing`
- Repository Interfaces (abstratas)

**Regras**:
- ❌ **ZERO** imports de frameworks (FastAPI, Celery, MongoDB, etc.)
- ✅ Apenas stdlib Python e typing
- ✅ Lógica de negócio pura (funções determinísticas)

**Exemplo Real** (`src/domain/entities/smart_bot.py`):
```python
@dataclass
class SmartBot:
    name: str
    max_position_size: Decimal
    soft_stop_loss: Decimal
    hard_stop_loss: Decimal

    def can_execute_purchase(self) -> bool:
        """Lógica de negócio pura - sem dependências externas"""
        if self.status != BotStatus.ACTIVE:
            return False
        if self.circuit_breaker_active:
            return False
        available = self.get_available_funds()
        return available > Decimal("0")

    def should_trigger_stop_loss(self, current_price: Decimal) -> bool:
        """Cálculo de risco - domínio puro"""
        variation = ((current_price - self.base_price) / self.base_price) * 100
        return variation <= -self.hard_stop_loss
```

### 2. Application Layer (Use Cases + Workflows)
**Responsabilidades**:
- Use Cases: `CreateSmartBotUseCase`, `RegisterUserUseCase`
- Workflows: `SmartBotPurchaseWorkflow`, `SyncOrdersWorkflow`
- Orquestração de Domain Services

**Regras**:
- ✅ Depende **apenas** do Domain Layer
- ✅ Define interfaces que Infrastructure implementa
- ❌ Não conhece detalhes de MongoDB, FastAPI, Celery

**Exemplo Real** (`src/application/use_cases/smart_bots/create_smart_bot.py`):
```python
class CreateSmartBotUseCase:
    def __init__(
        self,
        bot_repository: ISmartBotRepository,  # Interface do Domain
        user_repository: IUserRepository      # Interface do Domain
    ):
        self.bot_repository = bot_repository
        self.user_repository = user_repository

    async def execute(self, user_id: str, config: SmartBotConfig) -> SmartBot:
        # Validação de negócio
        user = await self.user_repository.get_by_id(user_id)
        if not user.can_create_bot():
            raise InsufficientSubscriptionError()

        # Criação de entidade (Domain)
        bot = SmartBot(
            name=config.name,
            max_position_size=config.max_position_size,
            # ...
        )

        # Persistência (via interface)
        return await self.bot_repository.create(bot)
```

### 3. Infrastructure Layer (Framework + External Services)
**Responsabilidades**:
- Repository Implementations: `SmartBotRepositoryImpl` (MongoDB)
- Exchange Adapters: `MercadoBitcoinExchange`, `BinanceExchange`
- Framework Code: FastAPI routes, Celery tasks
- External APIs: OpenAI, Mercado Pago, Stripe

**Regras**:
- ✅ Implementa interfaces definidas no Domain
- ✅ Depende de Domain + Application
- ✅ Único lugar onde frameworks aparecem

**Exemplo Real** (`src/infrastructure/repositories/smart_bot_repository_impl.py`):
```python
class SmartBotRepositoryImpl(ISmartBotRepository):
    def __init__(self, mongo_client: AsyncIOMotorClient):
        self.collection = mongo_client.crypteras_trading.smart_bots

    async def create(self, bot: SmartBot) -> SmartBot:
        """Converte Entity → MongoDB Document"""
        doc = {
            "name": bot.name,
            "max_position_size": float(bot.max_position_size),
            "created_at": datetime.utcnow(),
            # ...
        }
        result = await self.collection.insert_one(doc)
        bot.id = str(result.inserted_id)
        return bot
```

### Dependency Injection
FastAPI é configurado para injetar dependências automaticamente:

```python
# main.py
def create_app() -> FastAPI:
    app = FastAPI()

    # Infrastructure
    mongo_client = AsyncIOMotorClient(settings.MONGODB_URI)

    # Repositories (implementations)
    bot_repository = SmartBotRepositoryImpl(mongo_client)
    user_repository = MongoUserRepository(mongo_client)

    # Use Cases (injected)
    create_bot_use_case = CreateSmartBotUseCase(
        bot_repository=bot_repository,
        user_repository=user_repository
    )

    # Routes
    app.include_router(smart_bots_router)

    return app
```

## Consequências

### Positivas ✅

1. **Testabilidade Excelente**
   - Domain layer testável sem mocks (lógica pura)
   - Use cases testáveis com mocks simples (interfaces)
   - Exemplo: `tests/unit/domain/test_smart_bot.py` testa lógica sem banco

2. **Independência de Framework**
   - Trocar FastAPI por Flask/Django: apenas Infrastructure muda
   - Trocar MongoDB por PostgreSQL: apenas Repository implementations
   - Domain e Application permanecem intocados

3. **Clareza de Responsabilidades**
   - Novos desenvolvedores sabem onde colocar código:
     - Regra de negócio? → Domain
     - Orquestração? → Application
     - Detalhes técnicos? → Infrastructure

4. **Facilita Refatorações**
   - Remover `TradingConfig` deprecated: mudanças isoladas
   - Tornar workflows exchange-agnostic: refatorar Infrastructure

5. **Suporte a Mudanças Frequentes**
   - Adicionar novo tipo de bot: criar Entity no Domain
   - Mudar estratégia de risco: alterar Domain Service
   - Zero impacto em Infrastructure

### Negativas ⚠️

1. **Curva de Aprendizado**
   - Novos devs precisam entender camadas e fluxo de dependências
   - Tentação de "pular" camadas e acoplar código
   - **Mitigação**: Documentação clara (este ADR) + code reviews

2. **Mais Código (Boilerplate)**
   - Interfaces abstratas no Domain + implementações na Infrastructure
   - Exemplo: `ISmartBotRepository` (interface) + `SmartBotRepositoryImpl`
   - **Mitigação**: Ganho de manutenibilidade compensa

3. **Complexidade Inicial**
   - Projeto simples ficaria over-engineered
   - Mas Crypteras **não é simples**: 3 tipos de bots, multi-exchange, IA
   - **Justificativa**: Complexidade do domínio justifica arquitetura

4. **Risco de Débito Técnico**
   - Se regras forem violadas, perde-se benefícios
   - **Problema Atual**: Código não-PEP8, resquícios de TradingConfig
   - **Mitigação**: Linter em CI/CD, refatorações graduais

## Alternativas Consideradas

### Alternativa 1: MVC Tradicional (Rejeitada)
**Prós**:
- Simplicidade
- Familiaridade para maioria dos devs

**Contras**:
- Lógica de negócio espalhada entre Models/Controllers
- Difícil testar sem banco de dados
- Acoplamento alto com framework

**Razão para Rejeitar**: Complexidade do domínio de trading exige separação clara

### Alternativa 2: Hexagonal Architecture (Considerada)
**Prós**:
- Similar a Clean Architecture
- Foca em "ports and adapters"

**Contras**:
- Menos conhecida que Clean Architecture
- Documentação menos acessível

**Razão para Rejeitar**: Clean Architecture mais conhecida pela equipe, resultados similares

### Alternativa 3: Monolito Simples com Services (Rejeitada)
**Prós**:
- Rápido para prototipar
- Menos código

**Contras**:
- Sistema legado era assim e ficou unmaintainable
- Mudanças frequentes causariam caos

**Razão para Rejeitar**: Já experimentamos e não funcionou no legado

## Implementação

### Evidências no Código

**Domain Layer** (`backend/src/domain/`):
```
domain/
├── entities/
│   ├── smart_bot.py (716 lines - lógica complexa)
│   ├── candle_bot.py (547 lines)
│   ├── dca_bot.py (318 lines)
│   ├── user.py
│   └── subscription.py
├── services/
│   ├── risk_management.py
│   ├── technical_analysis_service.py
│   └── signal_consolidation_service.py
└── repositories/ (interfaces abstratas)
    ├── smart_bot_repository.py
    └── user_repository.py
```

**Application Layer** (`backend/src/application/`):
```
application/
├── use_cases/
│   ├── contexts/create_context.py
│   ├── auth/register_user.py
│   └── smart_bots/create_smart_bot.py
└── workflows/ (orquestração de domain services)
```

**Infrastructure Layer** (`backend/src/infrastructure/`):
```
infrastructure/
├── repositories/ (MongoDB implementations)
├── exchanges/ (Mercado Bitcoin, Binance)
└── celery/ (Task queue)
```

### Métricas de Sucesso
- ✅ Domain entities **sem** imports de frameworks
- ✅ Use cases testáveis com mocks simples
- ✅ Migração de APScheduler → Celery afetou **apenas** Infrastructure
- ⚠️ Ainda há violações: resquícios de TradingConfig

## Lições Aprendidas

### O Que Funcionou Bem
1. **Entities com Lógica Rica**: `SmartBot.can_execute_purchase()` eliminou if/else espalhados
2. **Repository Pattern**: Trocar MongoDB por outro banco seria trivial
3. **Testabilidade**: Testes de domain rodam em < 1s sem banco

### Desafios Enfrentados
1. **Resquícios de TradingConfig**: Bots ainda têm código antigo acoplado
2. **Código não-PEP8**: Violações de convenções prejudicam leitura
3. **Workflows Acoplados ao MB**: Deveriam usar apenas BaseExchange interface

### Recomendações Futuras
1. **Enforce com Linter**: Configurar arquivador pre-commit hooks + CI/CD
2. **Refatorar Gradualmente**: Remover TradingConfig em sprints dedicados
3. **Code Reviews**: Bloquear PRs que violem camadas

## Referências

- [Clean Architecture - Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Código Real: `backend/src/domain/entities/smart_bot.py`](../../../crypteras-improved/backend/src/domain/entities/smart_bot.py)
- [Issue CRY-82: ExchangeCredentials Refactor](link-to-linear-or-github-issue)

---

**ADR Aprovado Por**: Equipe de Desenvolvimento
**Data de Implementação**: v2.0 (Setembro 2024)
**Status Atual**: Em uso, com oportunidades de melhoria (PEP8, TradingConfig cleanup)
