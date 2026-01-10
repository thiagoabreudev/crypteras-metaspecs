:::stack
**Stack Tecnológica Aprovada**:

*   **Backend**: Python 3.11+
*   **Framework Web**: FastAPI
*   **Framework IA**: AGNO (Agentes, Tools, Memory)
*   **Frontend**: Nuxt.js 3 (Vue 3 + Composition API + TypeScript)
*   **Banco de Dados**: MongoDB Atlas (Driver: Motor/AsyncIO)
*   **Cache/Locking**: Redis
*   **Mensageria/Tasks**: Celery + Redis Broker
*   **Deploy**: Docker Swarm + Portainer

**Decisões Arquiteturais Chave (ADRs)**:
*   **ADR-001**: Clean Architecture (Domain isolado)
*   **ADR-005**: Adapter Pattern para Exchanges (BaseExchange)
*   **ADR-006**: AGNO Framework para Agentes IA
*   **ADR-007**: Depreciação de TradingConfig (Config por Bot)
:::
