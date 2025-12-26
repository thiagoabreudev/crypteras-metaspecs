---
spec_version: "1.0.0"
valid_from: "2025-12-26"
last_updated: "2025-12-26"
supersedes: null
status: "active"
category: "meta"
tags: ['index', 'root', 'navigation']
---

# Crypteras Metaspecs - Documentação Canônica

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-26
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão do índice raiz
- Ponto de entrada principal do repositório de metaspecs
:::

## 📖 Bem-vindo ao Repositório de Metaspecs

Este é o **repositório de metaspecs do Crypteras Trading System** - a fonte canônica de verdade para toda documentação de negócio, técnica e decisões arquiteturais do projeto.

**O que são Metaspecs?**
> Especificações versionadas, governadas e auditáveis que servem como contexto estruturado para desenvolvimento orientado por IA, garantindo consistência, rastreabilidade e qualidade ao longo do ciclo de vida do produto.

---

## 🎯 Propósito deste Repositório

### Para Desenvolvedores (Humanos e IA)
- ✅ **Contexto Completo**: Toda informação necessária para desenvolvimento
- ✅ **Versionado**: Histórico completo de mudanças com SemVer
- ✅ **Governado**: Hierarquia clara de precedência entre specs
- ✅ **Auditável**: Rastreabilidade total de decisões e mudanças

### Para Stakeholders de Negócio
- ✅ **Estratégia de Produto**: Visão, roadmap, posicionamento
- ✅ **Personas e Jornada**: Quem são nossos usuários e como os servimos
- ✅ **Funcionalidades**: Catálogo completo do que foi construído

### Para Compliance e Auditoria
- ✅ **Decision Log**: Todas decisões arquiteturais documentadas (ADRs)
- ✅ **Version History**: Changelog completo de todas as mudanças
- ✅ **Failure Modes**: Falhas conhecidas e mitigações

---

## 🗂️ Estrutura do Repositório

```
crypteras-metaspecs/
├── index.md                 # 👈 VOCÊ ESTÁ AQUI (ponto de entrada)
├── WORKFLOW.md              # Workflow de engenharia de contexto
├── HIERARCHY_VALIDATION_REPORT.md  # Validação da hierarquia
│
├── specs/                   # 📁 ESPECIFICAÇÕES (fonte canônica)
│   ├── index.md            # Índice geral de especificações
│   │
│   ├── _meta/              # 🔐 Governança e versionamento
│   │   ├── index.md
│   │   ├── VERSIONING_POLICY.md
│   │   ├── VERSION_HISTORY.md
│   │   ├── CONTEXT_HIERARCHY.md
│   │   └── ANTI_PATTERNS.md
│   │
│   ├── business/           # 💼 Contexto Empresarial
│   │   ├── index.md
│   │   ├── CUSTOMER_PERSONAS.md
│   │   ├── CUSTOMER_JOURNEY.md
│   │   ├── PRODUCT_STRATEGY.md
│   │   ├── COMPETITIVE_LANDSCAPE.md
│   │   ├── MESSAGING_FRAMEWORK.md
│   │   ├── CUSTOMER_COMMUNICATION.md
│   │   ├── VOICE_OF_CUSTOMER.md
│   │   └── features/       # 🎯 Catálogo de funcionalidades (13 features)
│   │       ├── index.md
│   │       ├── agno-chat-agent.md
│   │       ├── authentication-and-authorization.md
│   │       ├── backoffice-admin.md
│   │       ├── candle-bots.md
│   │       ├── circuit-breaker.md
│   │       ├── dashboard-analytics.md
│   │       ├── dca-bots.md
│   │       ├── exchange-credentials.md
│   │       ├── payment-webhooks.md
│   │       ├── smart-bots.md
│   │       ├── subscription-plans.md
│   │       └── trading-contexts.md
│   │
│   └── technical/          # 🛠️ Documentação Técnica
│       ├── index.md
│       ├── project_charter.md
│       ├── CLAUDE.meta.md  # ⭐ Guia de desenvolvimento com IA
│       ├── CODEBASE_GUIDE.md
│       ├── BUSINESS_LOGIC.md
│       ├── API_SPECIFICATION.md
│       ├── CONTRIBUTING.md
│       ├── TROUBLESHOOTING.md
│       ├── ARCHITECTURE_CHALLENGES.md
│       └── adr/            # 📋 Architecture Decision Records (7 ADRs)
│           ├── index.md
│           ├── 001-clean-architecture.md
│           ├── 002-celery-redis-migration.md
│           ├── 003-mongodb-over-postgresql.md
│           ├── 004-coprime-intervals.md
│           ├── 005-adapter-pattern-exchanges.md
│           ├── 006-agno-framework-ai.md
│           └── 007-trading-config-deprecated.md
│
└── .claude/                # 🤖 Comandos de governança para IA
    └── commands/
        └── metaspecs/
            ├── governance/
            ├── observability/
            └── validation/
```

---

## 🚀 Quick Start

### 🆕 Novo no Repositório?

**1. Entenda a estrutura**
- Leia este arquivo (`index.md` raiz) - ✅ Você está aqui
- Navegue para [specs/index.md](specs/index.md) - Índice geral de especificações

**2. Escolha sua camada**
- **Negócio?** → [specs/business/index.md](specs/business/index.md)
- **Técnico?** → [specs/technical/index.md](specs/technical/index.md)
- **Governança?** → [specs/_meta/index.md](specs/_meta/index.md)

**3. Leia o guia relevante**
- **Desenvolvedor?** → [specs/technical/CLAUDE.meta.md](specs/technical/CLAUDE.meta.md) ⭐
- **Produto/Marketing?** → [specs/business/PRODUCT_STRATEGY.md](specs/business/PRODUCT_STRATEGY.md)
- **IA/Agente?** → [specs/_meta/CONTEXT_HIERARCHY.md](specs/_meta/CONTEXT_HIERARCHY.md)

---

## 📋 Navegação Rápida por Persona

### 👨‍💻 Desenvolvedor Full-Stack
**Seu fluxo de trabalho**:
1. [specs/technical/CLAUDE.meta.md](specs/technical/CLAUDE.meta.md) - Padrões de código
2. [specs/technical/CODEBASE_GUIDE.md](specs/technical/CODEBASE_GUIDE.md) - Estrutura do código
3. [specs/technical/BUSINESS_LOGIC.md](specs/technical/BUSINESS_LOGIC.md) - Regras de negócio
4. [specs/technical/API_SPECIFICATION.md](specs/technical/API_SPECIFICATION.md) - Endpoints
5. [specs/technical/adr/](specs/technical/adr/index.md) - Decisões arquiteturais

### 🎨 Product Manager
**Seu fluxo de trabalho**:
1. [specs/business/PRODUCT_STRATEGY.md](specs/business/PRODUCT_STRATEGY.md) - Estratégia e roadmap
2. [specs/business/CUSTOMER_PERSONAS.md](specs/business/CUSTOMER_PERSONAS.md) - Quem são nossos usuários
3. [specs/business/CUSTOMER_JOURNEY.md](specs/business/CUSTOMER_JOURNEY.md) - Jornada do cliente
4. [specs/business/features/](specs/business/features/index.md) - Catálogo de funcionalidades
5. [specs/business/COMPETITIVE_LANDSCAPE.md](specs/business/COMPETITIVE_LANDSCAPE.md) - Concorrência

### 📣 Marketing/Vendas
**Seu fluxo de trabalho**:
1. [specs/business/MESSAGING_FRAMEWORK.md](specs/business/MESSAGING_FRAMEWORK.md) - Tom e voz da marca
2. [specs/business/VOICE_OF_CUSTOMER.md](specs/business/VOICE_OF_CUSTOMER.md) - Objeções comuns
3. [specs/business/CUSTOMER_PERSONAS.md](specs/business/CUSTOMER_PERSONAS.md) - Segmentação
4. [specs/business/COMPETITIVE_LANDSCAPE.md](specs/business/COMPETITIVE_LANDSCAPE.md) - Posicionamento

### 🤖 IA/Agente Autônomo
**Seu fluxo de trabalho**:
1. [specs/_meta/CONTEXT_HIERARCHY.md](specs/_meta/CONTEXT_HIERARCHY.md) - Precedência entre specs
2. [specs/_meta/VERSIONING_POLICY.md](specs/_meta/VERSIONING_POLICY.md) - Como versionar
3. [specs/business/](specs/business/index.md) - Contexto de negócio
4. [specs/technical/](specs/technical/index.md) - Contexto técnico
5. [specs/_meta/ANTI_PATTERNS.md](specs/_meta/ANTI_PATTERNS.md) - O que NÃO fazer

### 🔍 Auditor/Compliance
**Seu fluxo de trabalho**:
1. [specs/_meta/VERSION_HISTORY.md](specs/_meta/VERSION_HISTORY.md) - Changelog completo
2. [specs/technical/adr/](specs/technical/adr/index.md) - Decisões arquiteturais
3. [specs/_meta/CONTEXT_HIERARCHY.md](specs/_meta/CONTEXT_HIERARCHY.md) - Governança
4. [specs/technical/project_charter.md](specs/technical/project_charter.md) - Objetivos e escopo

---

## 🔐 Princípios de Governança

### 1. Jidoka (Stop-the-Line)
> "Qualquer pessoa tem o direito e a responsabilidade de parar toda a linha quando detecta um problema"

**Aplicado às metaspecs**:
- ❌ **Spec sem versionamento?** → PARE, adicione versão
- ❌ **Conflito entre specs?** → PARE, resolva conforme hierarquia
- ❌ **Spec desatualizada (>120 dias)?** → PARE, atualize ou deprecie

### 2. METASPECS-FIRST
> "Metaspecs management vem ANTES do código"

**Fluxo correto**:
1. ✅ Ler metaspecs relacionadas
2. ✅ Entender regras de negócio
3. ✅ Verificar ADRs
4. ✅ DEPOIS implementar código

### 3. Context Drift Prevention
> "Specs desatualizadas causam context drift"

**Práticas obrigatórias**:
- ✅ Revisão trimestral de todas as specs
- ✅ Auditoria semestral completa
- ✅ Specs antigas >120 dias = ⚠️ WARNING

### 4. Rastreabilidade Total
> "Sem rastreabilidade = sem auditoria"

**Regras**:
- ✅ Todo commit referencia ticket ou ADR
- ✅ Mudanças em specs = incremento de versão
- ✅ Histórico append-only (nunca deletar)

---

## 🎯 Hierarquia de Contexto (Precedência)

Quando specs conflitam, a camada **superior** sempre vence:

```
1. Meta Specs (_meta/)          → 🥇 MÁXIMA precedência
   ↓ (governa)
2. Business Specs (business/)   → 🥈 ALTA precedência
   ↓ (governa)
3. Technical Specs (technical/) → 🥉 MÉDIA precedência
   ↓ (governa)
4. Execution Context (código)   → MÍNIMA precedência
```

**Regra de Ouro**: Camadas superiores sempre vencem.

**Exemplo**:
- `_meta/VERSIONING_POLICY.md` > `technical/CONTRIBUTING.md`
- `business/PRODUCT_STRATEGY.md` > `technical/API_SPECIFICATION.md`
- ADRs (`technical/adr/`) > Guidelines gerais

**Documentação completa**: [specs/_meta/CONTEXT_HIERARCHY.md](specs/_meta/CONTEXT_HIERARCHY.md)

---

## 📊 Estatísticas do Repositório

| Métrica | Valor |
|---------|-------|
| **Total de specs** | 46 arquivos |
| **Cobertura de versionamento** | 100% |
| **Última atualização** | 2025-12-26 |
| **ADRs ativos** | 7 decisões arquiteturais |
| **Features documentadas** | 13 funcionalidades |
| **Personas de cliente** | 3 perfis detalhados |
| **Camadas de contexto** | 4 (Meta, Business, Technical, Execution) |

---

## 🔄 Versionamento Semântico

Este repositório usa **SemVer** (Semantic Versioning) para todas as specs:

- **MAJOR** (x.0.0): Breaking changes que alteram significado
- **MINOR** (0.x.0): Adições sem quebrar compatibilidade
- **PATCH** (0.0.x): Correções e clarificações

**Exemplo**:
```yaml
---
spec_version: "1.2.3"  # MAJOR.MINOR.PATCH
valid_from: "2025-12-26"
last_updated: "2025-12-26"
supersedes: "1.2.2"    # Versão anterior
status: "active"       # active | deprecated | draft
---
```

**Política completa**: [specs/_meta/VERSIONING_POLICY.md](specs/_meta/VERSIONING_POLICY.md)

---

## 🛠️ Comandos de Governança (IA)

Este repositório inclui comandos executáveis para governança de contexto via Claude:

### Governance
- `/metaspecs:governance:add-versioning` - Adiciona versionamento SemVer
- `/metaspecs:governance:add-intent` - Formaliza Intent as Code
- `/metaspecs:governance:add-failure-modes` - Documenta falhas conhecidas
- `/metaspecs:governance:add-explainability` - Configura explainability

### Observability
- `/metaspecs:observability:add-anti-patterns` - Documenta anti-patterns

### Validation
- `/metaspecs:validation:validate-hierarchy` - Valida hierarquia de contexto

**Documentação completa**: [WORKFLOW.md](WORKFLOW.md)

---

## 📚 Documentos Principais

### 🔐 Governança (Meta)
- [Política de Versionamento](specs/_meta/VERSIONING_POLICY.md) - Como versionar specs
- [Histórico de Versões](specs/_meta/VERSION_HISTORY.md) - Changelog completo
- [Hierarquia de Contexto](specs/_meta/CONTEXT_HIERARCHY.md) - Precedência entre specs
- [Anti-Patterns](specs/_meta/ANTI_PATTERNS.md) - O que evitar

### 💼 Negócio
- [Estratégia de Produto](specs/business/PRODUCT_STRATEGY.md) - Visão e roadmap
- [Personas de Cliente](specs/business/CUSTOMER_PERSONAS.md) - 3 perfis detalhados
- [Jornada do Cliente](specs/business/CUSTOMER_JOURNEY.md) - 12 fases do funil
- [Catálogo de Features](specs/business/features/index.md) - 13 funcionalidades

### 🛠️ Técnico
- [CLAUDE.meta.md](specs/technical/CLAUDE.meta.md) ⭐ - Guia de desenvolvimento com IA
- [Guia de Código](specs/technical/CODEBASE_GUIDE.md) - Estrutura do codebase
- [Lógica de Negócio](specs/technical/BUSINESS_LOGIC.md) - Regras e workflows
- [API Specification](specs/technical/API_SPECIFICATION.md) - Endpoints REST
- [ADRs](specs/technical/adr/index.md) - 7 decisões arquiteturais

---

## 🎓 Workflow de Desenvolvimento

### Para Desenvolvedores (METASPECS-FIRST)

**SEMPRE antes de implementar**:
1. ✅ Ler metaspecs relacionadas (`specs/business/`, `specs/technical/`)
2. ✅ Entender business logic (`specs/technical/BUSINESS_LOGIC.md`)
3. ✅ Verificar ADRs (`specs/technical/adr/`)
4. ✅ DEPOIS implementar seguindo `specs/technical/CLAUDE.meta.md`

**Durante implementação**:
- ✅ Consultar specs constantemente (não confiar em memória)
- ✅ Se detectar conflito → aplicar Jidoka (PARE, documente, resolva)
- ✅ Se specs desatualizadas → atualizar ANTES de prosseguir

**Após implementação**:
- ✅ Atualizar specs se necessário (incrementar versão)
- ✅ Documentar decisões em ADRs se relevante
- ✅ Atualizar `VERSION_HISTORY.md`

---

## 📞 Suporte

**Dúvidas sobre este repositório?**
- Leia [specs/_meta/README.md](specs/_meta/README.md) para governança
- Consulte [WORKFLOW.md](WORKFLOW.md) para engenharia de contexto
- Revise [specs/_meta/VERSION_HISTORY.md](specs/_meta/VERSION_HISTORY.md) para mudanças

**Contribuindo?**
- Siga [specs/technical/CONTRIBUTING.md](specs/technical/CONTRIBUTING.md)
- Respeite [specs/_meta/VERSIONING_POLICY.md](specs/_meta/VERSIONING_POLICY.md)
- Evite [specs/_meta/ANTI_PATTERNS.md](specs/_meta/ANTI_PATTERNS.md)

---

## 🎯 Próximos Passos

Após ler este índice:

1. **Escolha sua persona** (Desenvolvedor, PM, Marketing, IA, Auditor)
2. **Navegue para specs/index.md** → [specs/index.md](specs/index.md)
3. **Localize a camada** (Business ou Technical)
4. **Verifique versão** antes de usar qualquer spec
5. **Aplique princípios** (Jidoka, METASPECS-FIRST, Context Drift Prevention)

---

## 💡 Citação Final

> **"A vantagem competitiva não está em usar IA, mas em usar IA melhor que os outros."**
>
> — Workshop "IA do Jeito Certo"

Este repositório implementa as melhores práticas de **Engenharia de Contexto** para desenvolvimento orientado por IA, garantindo consistência, qualidade e rastreabilidade ao longo do ciclo de vida do Crypteras Trading System.

---

**Última Atualização**: 2025-12-26
**Versão do Índice**: 1.0.0
**Mantenedor**: Equipe Crypteras
**Repositório de Código**: `/Users/thiagoabreu/workspace/crypteras-improved`
