---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
---

# Política de Versionamento de Contexto - Crypteras

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir política de versionamento semântico para specs e rastreamento de mudanças.

**Constraints** (limites obrigatórios):
- Seguir SemVer (MAJOR.MINOR.PATCH)
- MAJOR: breaking changes
- MINOR: adições retrocompatíveis (ex: adicionar Intent)
- PATCH: fixes e clarificações
- Documentar mudanças em :::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta


**Non-Goals** (o que NÃO fazer):
- Versionar cada parágrafo individualmente
- Criar versionamento complexo multi-dimensional
- Manter múltiplas versões ativas simultaneamente
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão da política de versionamento
- Estabelece regras para Context Governance
:::

## 🎯 Objetivo

Estabelecer regras claras de versionamento semântico para especificações (metaspecs) do Crypteras Trading System, garantindo **Context Governance** e prevenindo **Context Drift**.

---

## 📋 Versionamento Semântico (SemVer)

Todas as specs seguem o padrão **MAJOR.MINOR.PATCH**:

### MAJOR (X.0.0)

**Breaking Changes Semânticos** - Mudanças que alteram o significado ou contrato do contexto:

- Mudança de significado de termos existentes
- Remoção de seções obrigatórias
- Alteração incompatível de estrutura
- Mudança de regras de negócio que invalidam implementações anteriores

**Exemplo**:
```
v1.0.0: "limite" = valor máximo permitido
v2.0.0: "limite" = pré-aprovado por CPF (BREAKING CHANGE)
```

### MINOR (0.X.0)

**Adições Compatíveis** - Novas informações que não quebram o contexto existente:

- Nova seção adicionada
- Novos campos opcionais
- Novos exemplos
- Expansão de casos de uso
- Novas features documentadas

**Exemplo**:
```
v1.0.0: 3 personas definidas
v1.1.0: Adicionada 4ª persona (compatível com anterior)
```

### PATCH (0.0.X)

**Correções e Clarificações** - Ajustes que não alteram o contrato:

- Correção de typos
- Clarificações de texto ambíguo
- Melhoria de exemplos existentes
- Atualização de links
- Formatação

**Exemplo**:
```
v1.0.0: "O bot executa ordens"
v1.0.1: "O bot executa ordens de compra e venda" (clarificação)
```

---

## 🏷️ Metadados Obrigatórios

Toda spec DEVE incluir no frontmatter YAML:

```yaml
---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
---
```

### Campos Obrigatórios

| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| `spec_version` | string | Versão SemVer | `"1.2.3"` |
| `valid_from` | date | Data de início de validade | `"2025-12-25"` |
| `last_updated` | date | Data da última modificação | `"2025-12-25"` |
| `supersedes` | string\|null | Versão anterior substituída | `"1.0.0"` ou `null` |
| `status` | enum | Estado da spec | `"active"` \| `"deprecated"` \| `"draft"` |

### Status da Spec

- **active**: Versão atual e válida
- **deprecated**: Versão obsoleta mas ainda referenciável
- **draft**: Versão em construção, não usar em produção

---

## 📝 Blocos de Informação

### Version Info (obrigatório)

Após o título principal:

```markdown
:::version_info
**Versão**: 1.2.3
**Válida desde**: 2025-12-25
**Status**: Ativa
:::
```

### Breaking Changes (obrigatório)

Documenta histórico de mudanças incompatíveis:

```markdown
:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v2.0.0**:
- "limite" agora significa "pré-aprovado por CPF" (antes era "limite contratual")
- Campo `priority` mudou de texto livre para enum

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
:::
```

---

## 🔄 Processo de Atualização

### 1. Determinar Tipo de Mudança

**Perguntas para classificar**:

- ❓ Altera significado de algo existente? → **MAJOR**
- ❓ Remove algo obrigatório? → **MAJOR**
- ❓ Adiciona informação nova? → **MINOR**
- ❓ Apenas corrige/clarifica? → **PATCH**

### 2. Atualizar Metadados

```yaml
# Antes
spec_version: "1.2.3"
last_updated: "2025-12-20"
supersedes: "1.1.0"

# Depois (MINOR change)
spec_version: "1.3.0"
last_updated: "2025-12-25"
supersedes: "1.2.3"
```

### 3. Documentar Mudança

Adicionar em `:::breaking_changes` (se MAJOR) e sempre em `VERSION_HISTORY.md`:

```markdown
**v1.3.0** (2025-12-25):
- Adicionada seção "Failure Modes"
- Novos exemplos de validação de contexto
```

### 4. Atualizar Índice

Atualizar referência de versão em `specs/business/index.md` ou `specs/technical/index.md`:

```markdown
- [Personas do Cliente](CUSTOMER_PERSONAS.md) - v1.3.0
```

### 5. Registrar no VERSION_HISTORY.md

Adicionar linha na tabela correspondente.

---

## 🚨 Princípio Jidoka (Stop-the-Line)

**Se encontrar spec sem versionamento durante desenvolvimento**:

```
🛑 PARE a execução
⚠️ ALERTE que a spec precisa ser versionada
📋 SUGIRA executar `/governance:add-versioning` antes de prosseguir
```

**Regra**: Nenhuma spec pode ser usada como contexto sem versionamento adequado.

---

## 📊 Rastreabilidade

Toda mudança de versão DEVE ser:

1. ✅ Registrada em `VERSION_HISTORY.md`
2. ✅ Documentada em `:::breaking_changes` (se MAJOR)
3. ✅ Atualizada no índice correspondente
4. ✅ Commitada com mensagem descritiva:

```bash
git commit -m "specs: CUSTOMER_PERSONAS v1.3.0 - Add failure modes section"
```

---

## 🎯 Benefícios

| Benefício | Descrição |
|-----------|-----------|
| **Evita Context Clash** | Versões incompatíveis não se misturam |
| **Facilita Auditoria** | Histórico completo de evolução |
| **Permite Rollback** | Voltar para versão anterior se necessário |
| **Detecta Drift** | Identifica contexto desatualizado |
| **Validação Automática** | IA pode verificar compatibilidade |

---

## 📚 Exemplos

### Exemplo 1: Correção de Typo (PATCH)

```yaml
# v1.0.0 → v1.0.1
spec_version: "1.0.1"
last_updated: "2025-12-25"
supersedes: "1.0.0"
```

Mensagem: `"Fix typo in persona description"`

### Exemplo 2: Nova Seção (MINOR)

```yaml
# v1.0.1 → v1.1.0
spec_version: "1.1.0"
last_updated: "2025-12-25"
supersedes: "1.0.1"
```

Mensagem: `"Add failure modes section to personas spec"`

### Exemplo 3: Mudança de Significado (MAJOR)

```yaml
# v1.1.0 → v2.0.0
spec_version: "2.0.0"
last_updated: "2025-12-25"
supersedes: "1.1.0"
status: "active"
```

```markdown
:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v2.0.0**:
- "limite" agora significa "limite pré-aprovado por CPF" (antes era "limite contratual do convênio")
- Incompatível com v1.x - revisar todas as referências a "limite"
:::
```

Mensagem: `"BREAKING: Change 'limite' semantic meaning to pre-approved limit"`

---

## 🔍 Validação

Toda spec DEVE passar pelas seguintes validações:

- [ ] Possui frontmatter com todos os campos obrigatórios
- [ ] `spec_version` segue SemVer válido
- [ ] `valid_from` e `last_updated` são datas válidas (YYYY-MM-DD)
- [ ] `status` é um dos valores permitidos
- [ ] Possui bloco `:::version_info`
- [ ] Possui bloco `:::breaking_changes`
- [ ] Versão registrada em `VERSION_HISTORY.md`
- [ ] Versão atualizada no índice correspondente

Use `/validation:audit-spec` para validar automaticamente.

---

## 📖 Referências

- [Semantic Versioning 2.0.0](https://semver.org/)
- [VERSION_HISTORY.md](./VERSION_HISTORY.md)
- Comando: `/governance:add-versioning`
- Comando: `/validation:audit-spec`

---

**Última atualização**: 2025-12-25
**Versão**: 1.1.0
**Responsável**: Context Governance System
