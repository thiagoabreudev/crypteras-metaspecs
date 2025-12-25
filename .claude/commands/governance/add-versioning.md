# Adicionar Versionamento às Metaspecs - Crypteras

Este comando adiciona **Context Governance** através de versionamento semântico às especificações do **Crypteras Trading System**.

> **Nota**: Este é um comando avançado de governança de contexto. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Context Governance

**Context Governance** é o conjunto de práticas que garantem que o contexto fornecido à IA seja correto, atual, consistente e intencional ao longo do tempo.

**Problema sem governança**:
- Context Drift (contexto desatualizado)
- Regressão semântica (significado de termos muda sem controle)
- Conflitos entre documentação e código
- Decisões inconsistentes entre execuções da IA

**Solução**: Versionamento semântico de contexto, assim como APIs e contratos.

---

## 📋 Objetivo

Adicionar metadados de versionamento às specs existentes para:
- Rastrear evolução do contexto
- Identificar breaking changes semânticos
- Permitir rollback de contexto
- Evitar Context Clash

---

## 🔍 Processo

### 1. Análise das Specs Existentes

Identifique todas as specs no diretório `specs/`:

```bash
specs/
├── business/
│   ├── index.md
│   ├── CUSTOMER_PERSONAS.md
│   ├── PRODUCT_STRATEGY.md
│   └── ...
└── technical/
    ├── index.md
    ├── CLAUDE.meta.md
    ├── CODEBASE_GUIDE.md
    └── adr/
        └── 001-*.md
```

### 2. Template de Versionamento

Adicione ao **início** de cada spec (após o frontmatter YAML):

```yaml
---
spec_version: "1.0.0"
valid_from: "2025-12-20"
last_updated: "2025-12-20"
supersedes: null
status: "active"  # active | deprecated | draft
---

# [Título da Spec]

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-20
**Status**: Ativa
:::

:::breaking_changes
Nenhuma breaking change nesta versão (primeira versão).
:::
```

**Campos obrigatórios**:
- `spec_version`: SemVer (MAJOR.MINOR.PATCH)
  - **MAJOR**: Mudanças que quebram compatibilidade (ex: "limite" muda de significado)
  - **MINOR**: Novas seções/conceitos adicionados
  - **PATCH**: Correções, clarificações, exemplos
- `valid_from`: Data de início de validade (YYYY-MM-DD)
- `last_updated`: Data da última modificação
- `supersedes`: Versão anterior que esta substitui (ou `null` se primeira)
- `status`: Estado atual da spec

### 3. Documentar Breaking Changes

Para specs que já existem e estão sendo versionadas pela primeira vez:

```markdown
:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Contexto anterior não tinha versionamento explícito
:::
```

Para mudanças futuras:

```markdown
:::breaking_changes
**v2.0.0**:
- "limite" agora significa "pré-aprovado por CPF" (antes era "limite contratual do convênio")
- Campo `priority` mudou de texto livre para enum: ["low", "medium", "high"]

**v1.1.0**:
- Adicionada seção "Failure Modes"
- Adicionados exemplos de validação

**v1.0.0** (baseline):
- Primeira versão versionada
:::
```

### 4. Criar Registro de Versionamento

Crie arquivo `specs/_meta/VERSION_HISTORY.md`:

```markdown
# Histórico de Versões das Metaspecs

Este arquivo rastreia mudanças de versão em todas as especificações.

## Specs de Negócio

### CUSTOMER_PERSONAS.md
| Versão | Data | Tipo | Descrição |
|--------|------|------|-----------|
| 1.0.0  | 2025-12-20 | baseline | Primeira versão versionada |

### PRODUCT_STRATEGY.md
| Versão | Data | Tipo | Descrição |
|--------|------|------|-----------|
| 1.0.0  | 2025-12-20 | baseline | Primeira versão versionada |

## Specs Técnicas

### CLAUDE.meta.md
| Versão | Data | Tipo | Descrição |
|--------|------|------|-----------|
| 1.0.0  | 2025-12-20 | baseline | Primeira versão versionada |

### ADRs
| ADR | Versão | Data | Tipo | Descrição |
|-----|--------|------|------|-----------|
| 001 | 1.0.0  | 2025-12-20 | baseline | Primeira versão |

---

## Tipos de Mudança

- **major**: Breaking change (altera significado, remove campos obrigatórios)
- **minor**: Adição compatível (novas seções, novos campos opcionais)
- **patch**: Correções e clarificações
- **baseline**: Primeira versão versionada
```

### 5. Atualizar Índices

Atualize `specs/business/index.md` e `specs/technical/index.md` para referenciar versões:

```markdown
## Camada 1: Arquitetura de Contexto do Cliente

- [Personas do Cliente](CUSTOMER_PERSONAS.md) - v1.0.0
- [Jornada do Cliente](CUSTOMER_JOURNEY.md) - v1.0.0
- [Voz do Cliente](VOICE_OF_CUSTOMER.md) - v1.0.0
```

---

## ✅ Checklist de Execução

**Para cada spec existente**:
- [ ] Adicionar frontmatter com `spec_version`, `valid_from`, `last_updated`, `supersedes`, `status`
- [ ] Adicionar `:::version_info` após título
- [ ] Adicionar `:::breaking_changes` com baseline
- [ ] Registrar no `VERSION_HISTORY.md`
- [ ] Atualizar referência no índice

**Criar novos arquivos**:
- [ ] `specs/_meta/VERSION_HISTORY.md`
- [ ] `specs/_meta/VERSIONING_POLICY.md` (política de versionamento)

---

## 📊 Exemplo Completo

```yaml
---
spec_version: "1.0.0"
valid_from: "2025-12-20"
last_updated: "2025-12-20"
supersedes: null
status: "active"
---

# Personas do Cliente

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-20
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Define 3 personas principais: Ana (Estudante), Carlos (Profissional), Maria (Empreendedora)
:::

## Persona Principal: Ana - Estudante Organizada

### Demografia
...
```

---

## 🎯 Benefícios

✅ **Evita Context Clash**: Diferentes versões de contexto não se misturam
✅ **Facilita Auditoria**: Histórico completo de evolução do contexto
✅ **Permite Rollback**: Voltar para versão anterior se necessário
✅ **Detecta Drift**: Fácil identificar quando contexto ficou desatualizado
✅ **Validação Automática**: IA pode validar compatibilidade de versões

---

## 🚨 Princípio Jidoka

**Se encontrar spec sem versionamento durante desenvolvimento**:
- 🛑 **PARE** a execução
- ⚠️ **ALERTE** que a spec precisa ser versionada
- 📋 **SUGIRA** executar `/add-versioning` antes de prosseguir

---

## 🔄 Manutenção

**Quando atualizar versão de uma spec**:

1. **MAJOR** (breaking change):
   - Mudança de significado de termos
   - Remoção de seções obrigatórias
   - Alteração incompatível de estrutura

2. **MINOR** (adição compatível):
   - Nova seção adicionada
   - Novos campos opcionais
   - Exemplos adicionados

3. **PATCH** (correção):
   - Typos corrigidos
   - Clarificações
   - Links atualizados

**Processo**:
1. Atualizar `spec_version` no frontmatter
2. Atualizar `last_updated` para data atual
3. Se MAJOR: atualizar `supersedes` para versão anterior
4. Documentar mudança em `:::breaking_changes`
5. Registrar em `VERSION_HISTORY.md`
6. Atualizar versão no índice

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, versione TODAS as specs em `specs/business/` e `specs/technical/`.

Se argumentos forem fornecidos (ex: `business` ou `technical` ou caminho específico), versione apenas as specs especificadas.
