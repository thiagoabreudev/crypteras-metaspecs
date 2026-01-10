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

11. **MongoDB Atlas vs Localhost - Debug com Conexão Errada**
   - **Tipo**: integration
   - **Descrição**: IA tenta debugar conectando em `mongodb://localhost:27017` ao invés de usar `MONGODB_URI` do ambiente (MongoDB Atlas)
   - **Gatilho**: Debug de dados, verificação de collections, queries manuais
   - **Impacto**: 🔴 Crítico (debug falha, IA não vê dados reais, decisões baseadas em estado vazio)
   - **Mitigação**:
     - SEMPRE ler variável de ambiente do container Docker: `docker service inspect crypteras_agno --format='{{json .Spec.TaskTemplate.ContainerSpec.Env}}'`
     - NUNCA assumir localhost
     - Banco está no **MongoDB Atlas** (não local)
     - Nome do banco: **`crypteras_trading`** (não `crypteras` ou `trading`)
   - **Detecção**: Erro "Connection refused localhost:27017" mas produção está no Atlas. IA reporta "collection vazia" mas dados existem
   - **Código Correto**:
     ```python
     # ✅ CORRETO - Verificar env do container primeiro
     # docker service inspect crypteras_agno --format='{{json .Spec.TaskTemplate.ContainerSpec.Env}}'
     import os
     mongo_uri = os.getenv('MONGODB_URI')  # mongodb+srv://...@crypteras.4etwcbo.mongodb.net/crypteras_trading
     client = AsyncIOMotorClient(mongo_uri)
     db = client.crypteras_trading  # Nome correto do banco

     # ❌ ERRADO
     # client = AsyncIOMotorClient('localhost', 27017)
     # db = client.crypteras
     ```

12. **MongoDB Collections - Assumir Nomes Sem Verificar**
   - **Tipo**: hallucination
   - **Descrição**: IA assume nomes de collections (`users`, `orders`) sem verificar o schema real (`platform_users`, `order_strategies`)
   - **Gatilho**: Queries manuais ou scripts de migração
   - **Impacto**: 🔴 Crítico (queries retornam vazio, scripts falham)
   - **Mitigação**: SEMPRE listar collections antes de query: `db.list_collection_names()`
   - **Detecção**: `db.users.find()` (errado) vs `db.platform_users.find()` (certo)

13. **Deploy Scripts - Execução Não Autorizada**
   - **Tipo**: security
   - **Descrição**: IA tenta rodar scripts de deploy (`deploy.sh`, `build-and-push.sh`) diretamente
   - **Gatilho**: Pedido de "deploy" ou "atualizar produção"
   - **Impacto**: 🔴 Crítico (risco de downtime ou estado inconsistente)
   - **Mitigação**: Scripts de deploy são APENAS para CI/CD ou humanos autorizados. IA deve apenas commitar código.
   - **Detecção**: Tentativa de exec `bash deploy.sh`

14. **Frontend Hardcoded URLs - Ignorar Runtime Config**
   - **Tipo**: integration
   - **Descrição**: IA hardcoda URLs de API no frontend (`fetch('http://api.crypteras.com')`)
   - **Gatilho**: Integração frontend-backend
   - **Impacto**: 🔴 Crítico (quebra em staging/dev/prod)
   - **Mitigação**: Usar `useRuntimeConfig().public.apiBase`
   - **Detecção**: Strings de URL hardcoded em `.vue` ou `.ts`

15. **AGNO vs API Dashboard - Arquitetura de Serviços**
   - **Tipo**: architecture
   - **Descrição**: IA confunde responsabilidades. AGNO (Agentes) não deve servir API REST para o Dashboard (Backend principal).
   - **Gatilho**: Criar endpoints para o frontend
   - **Impacto**: 🟡 Médio (acoplamento indevido)
   - **Mitigação**: Dashboard consome API do Backend Principal (`backend/src/api`). AGNO é um serviço worker/agent consumido pelo Backend.
   - **Detecção**: Criar rotas de API REST dentro do serviço AGNO para uso direto do frontend.
:::
