---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2025-12-26"
supersedes: null
status: "active"
category: "meta"
tags: ['index', 'specs', 'navigation']
---

# Crypteras Metaspecs - Índice Geral

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada do índice geral
- Estrutura completa de navegação do repositório
:::

## 🎯 Sobre este Repositório

Este repositório contém a **fonte canônica de verdade** para toda a documentação, especificações técnicas, lógica de negócio e decisões arquiteturais do **Crypteras Trading System**.

**Propósito**: Fornecer contexto completo, versionado e auditável para:
- Equipes de desenvolvimento (humanos e IA)
- Stakeholders de negócio
- Sistemas automatizados
- Auditoria e compliance

---

## 📁 Estrutura do Repositório

```
crypteras-metaspecs/
├── specs/                    # Especificações (fonte canônica)
│   ├── _meta/               # Governança e versionamento
│   ├── business/            # Contexto empresarial
│   │   └── features/        # Catálogo de funcionalidades
│   └── technical/           # Documentação técnica
│       └── adr/             # Architecture Decision Records
├── docs/                    # Documentação adicional
└── README.md               # Guia principal do projeto
```

---

## 🗂️ Navegação por Camada

### [Camada Meta - Governança](_meta/index.md)
Sistema de governança de contexto, versionamento semântico e políticas.

**Arquivos principais**:
- [Política de Versionamento](_meta/VERSIONING_POLICY.md) - Regras SemVer
- [Histórico de Versões](_meta/VERSION_HISTORY.md) - Changelog completo
- [Hierarquia de Contexto](_meta/CONTEXT_HIERARCHY.md) - Precedência entre specs
- [Anti-Patterns](_meta/ANTI_PATTERNS.md) - Padrões a evitar

**Quando usar**: Antes de criar/atualizar qualquer spec

---

### [Camada Business - Contexto Empresarial](business/index.md)
Estratégia de produto, personas, jornada do cliente, posicionamento competitivo.

**Arquivos principais**:
- [Personas de Cliente](business/CUSTOMER_PERSONAS.md) - 3 perfis detalhados
- [Jornada do Cliente](business/CUSTOMER_JOURNEY.md) - 12 fases do funil
- [Estratégia de Produto](business/PRODUCT_STRATEGY.md) - Visão e roadmap
- [Panorama Competitivo](business/COMPETITIVE_LANDSCAPE.md) - 3Commas, outros
- [Framework de Mensagens](business/MESSAGING_FRAMEWORK.md) - Tom e voz
- [Comunicação com Cliente](business/CUSTOMER_COMMUNICATION.md) - Diretrizes

**Subpasta**: [Catálogo de Features](business/features/index.md) - 13 funcionalidades documentadas

**Quando usar**: Decisões de produto, marketing, vendas, suporte

---

### [Camada Technical - Documentação Técnica](technical/index.md)
Arquitetura, código, APIs, guias de desenvolvimento.

**Arquivos principais**:
- [Carta do Projeto](technical/project_charter.md) - Visão e objetivos
- [Guia de Desenvolvimento com IA](technical/CLAUDE.meta.md) - Padrões e boas práticas
- [Guia de Navegação da Base de Código](technical/CODEBASE_GUIDE.md) - Estrutura
- [Lógica de Negócio](technical/BUSINESS_LOGIC.md) - Regras e workflows
- [Especificações da API](technical/API_SPECIFICATION.md) - Endpoints REST
- [Guia de Contribuição](technical/CONTRIBUTING.md) - Fluxo de desenvolvimento
- [Solução de Problemas](technical/TROUBLESHOOTING.md) - Debugging
- [Desafios Arquiteturais](technical/ARCHITECTURE_CHALLENGES.md) - Débito técnico

**Subpasta**: [ADRs (Architecture Decision Records)](technical/adr/index.md) - 7 decisões arquiteturais

**Quando usar**: Desenvolvimento, debugging, code review, arquitetura

---

## 🚀 Quick Start

### Para Novos Desenvolvedores
1. Leia [technical/project_charter.md](technical/project_charter.md) - Visão geral
2. Configure ambiente com [technical/CONTRIBUTING.md](technical/CONTRIBUTING.md)
3. Navegue código com [technical/CODEBASE_GUIDE.md](technical/CODEBASE_GUIDE.md)
4. Siga padrões em [technical/CLAUDE.meta.md](technical/CLAUDE.meta.md)

### Para Equipe de Produto/Marketing
1. Entenda clientes: [business/CUSTOMER_PERSONAS.md](business/CUSTOMER_PERSONAS.md)
2. Conheça jornada: [business/CUSTOMER_JOURNEY.md](business/CUSTOMER_JOURNEY.md)
3. Revise estratégia: [business/PRODUCT_STRATEGY.md](business/PRODUCT_STRATEGY.md)
4. Use mensagens: [business/MESSAGING_FRAMEWORK.md](business/MESSAGING_FRAMEWORK.md)

### Para IA/Agentes
1. **Sempre** verifique versionamento em `_meta/`
2. Consulte [business/](business/index.md) para contexto de negócio
3. Consulte [technical/](technical/index.md) para implementação
4. Respeite [hierarquia de contexto](_meta/CONTEXT_HIERARCHY.md)

---

## 📊 Estatísticas do Repositório

| Métrica | Valor |
|---------|-------|
| Total de specs | 46 arquivos |
| Cobertura de versionamento | 100% |
| Última atualização | 2025-12-26 |
| ADRs ativos | 6 |
| Features documentadas | 13 |
| Personas de cliente | 3 |

---

## 🔄 Sistema de Versionamento

Este repositório usa **Versionamento Semântico** (SemVer) para todas as specs:

- **MAJOR** (x.0.0): Breaking changes que alteram significado
- **MINOR** (0.x.0): Adições sem quebrar compatibilidade
- **PATCH** (0.0.x): Correções e clarificações

**Ver**: [Política de Versionamento](_meta/VERSIONING_POLICY.md)

---

## 📋 Navegação Rápida por Tipo de Tarefa

### 🆕 Novo na Equipe?
- [Carta do Projeto](technical/project_charter.md)
- [Guia de Código](technical/CODEBASE_GUIDE.md)
- [Personas de Cliente](business/CUSTOMER_PERSONAS.md)

### 🐛 Debugando Issue?
- [Troubleshooting](technical/TROUBLESHOOTING.md)
- [Lógica de Negócio](technical/BUSINESS_LOGIC.md)
- [ADRs](technical/adr/index.md)

### ✨ Implementando Feature?
- [Catálogo de Features](business/features/index.md)
- [API Specification](technical/API_SPECIFICATION.md)
- [CLAUDE.meta](technical/CLAUDE.meta.md)

### 🔧 Refatorando Código?
- [Desafios Arquiteturais](technical/ARCHITECTURE_CHALLENGES.md)
- [ADRs](technical/adr/index.md)
- [Anti-Patterns](_meta/ANTI_PATTERNS.md)

### 📈 Planejando Produto?
- [Estratégia de Produto](business/PRODUCT_STRATEGY.md)
- [Voz do Cliente](business/VOICE_OF_CUSTOMER.md)
- [Panorama Competitivo](business/COMPETITIVE_LANDSCAPE.md)

---

## 🛡️ Princípios de Governança

### Jidoka (Stop-the-Line)
**Nenhuma spec pode ser usada sem versionamento adequado.**

Se encontrar spec sem versão:
1. 🛑 PARE a execução
2. ⚠️ ALERTE sobre a necessidade de versionamento
3. 📋 SUGIRA executar `/metaspecs:governance:add-versioning`
4. ❌ NÃO prossiga até resolver

### Context Drift Prevention
- Specs desatualizadas causam **context drift**
- Revisão trimestral obrigatória
- Auditoria semestral completa

### Rastreabilidade Total
- Todo commit deve referenciar ticket ou ADR
- Mudanças em specs = incremento de versão
- Histórico append-only (nunca deletar)

---

## 📞 Suporte

**Dúvidas sobre este repositório?**
- Leia [_meta/README.md](_meta/README.md) para governança
- Consulte [technical/CONTRIBUTING.md](technical/CONTRIBUTING.md) para contribuir
- Revise [_meta/VERSION_HISTORY.md](_meta/VERSION_HISTORY.md) para mudanças

---

## 🎯 Próximos Passos

Após ler este índice:

1. **Escolha sua camada** (Business ou Technical)
2. **Navegue para o index.md** correspondente
3. **Localize a spec** que precisa
4. **Verifique versão** antes de usar
5. **Documente aprendizados** em specs futuras

---

**Última Atualização**: 2025-12-26
**Versão do Índice**: 1.0.0
**Mantenedor**: Equipe Crypteras
