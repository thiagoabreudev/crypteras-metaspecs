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
