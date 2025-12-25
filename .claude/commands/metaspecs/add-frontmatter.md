# Adicionar Frontmatter aos Arquivos Markdown

Este comando adiciona frontmatter YAML aos arquivos markdown do projeto, incluindo metadados de versionamento e informações contextuais.

## 🎯 Objetivo

Garantir que todos os arquivos markdown importantes tenham frontmatter consistente com:
- Informações de versionamento
- Metadados de documentação
- Tags e categorias
- Datas de criação e atualização

---

## 📋 Tipos de Frontmatter

### 1. Frontmatter para Specs (specs/business/ e specs/technical/)

```yaml
---
spec_version: "1.0.0"
valid_from: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
supersedes: null
status: "active"
category: "[business|technical|meta]"
tags: ["tag1", "tag2", "tag3"]
---
```

### 2. Frontmatter para Features (specs/business/features/)

```yaml
---
spec_version: "1.0.0"
valid_from: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
status: "active"
priority: "[high|medium|low]"
category: "feature"
tags: ["feature-name", "domain"]
related_specs:
  - "PRODUCT_STRATEGY.md"
  - "CUSTOMER_PERSONAS.md"
---
```

### 3. Frontmatter para ADRs (specs/technical/adr/)

```yaml
---
adr_number: "001"
title: "[Título da Decisão]"
date: "YYYY-MM-DD"
status: "[proposed|accepted|deprecated|superseded]"
deciders: ["nome1", "nome2"]
consulted: ["nome3", "nome4"]
informed: ["nome5"]
supersedes: null
superseded_by: null
tags: ["architecture", "decision"]
---
```

### 4. Frontmatter para Documentação Geral (docs/)

```yaml
---
title: "[Título do Documento]"
author: "[Nome do Autor]"
date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "1.0.0"
status: "[draft|review|published]"
tags: ["documentation"]
---
```

---

## 🔍 Processo de Adição

### Passo 1: Identificar Arquivos sem Frontmatter

```bash
# Encontrar arquivos .md sem frontmatter
find specs/ docs/ -name "*.md" -type f
```

Para cada arquivo, verificar se inicia com `---` (indica frontmatter existente).

### Passo 2: Determinar Tipo de Frontmatter

Baseado no caminho do arquivo:

- `specs/business/*.md` → Frontmatter de Spec (business)
- `specs/business/features/*.md` → Frontmatter de Feature
- `specs/technical/*.md` → Frontmatter de Spec (technical)
- `specs/technical/adr/*.md` → Frontmatter de ADR
- `specs/_meta/*.md` → Frontmatter de Spec (meta)
- `docs/*.md` → Frontmatter de Documentação Geral

### Passo 3: Gerar Frontmatter Apropriado

**Para cada arquivo identificado**:

1. Ler conteúdo do arquivo
2. Extrair título (primeira linha `# Título`)
3. Gerar frontmatter baseado no tipo
4. Inserir frontmatter no início do arquivo
5. Preservar conteúdo original

### Passo 4: Validar Frontmatter

Verificar que:
- YAML está sintaticamente correto
- Campos obrigatórios estão presentes
- Datas estão no formato ISO (YYYY-MM-DD)
- Status é válido
- Versões seguem SemVer

---

## 📝 Exemplos

### Exemplo 1: Spec de Negócio

**Arquivo**: `specs/business/CUSTOMER_PERSONAS.md`

**Antes**:
```markdown
# Personas do Cliente

## Persona Principal: Carlos - Operador de Digitalização
...
```

**Depois**:
```markdown
---
spec_version: "1.0.0"
valid_from: "2025-12-23"
last_updated: "2025-12-23"
supersedes: null
status: "active"
category: "business"
tags: ["personas", "customer-research", "ux"]
---

# Personas do Cliente

## Persona Principal: Carlos - Operador de Digitalização
...
```

### Exemplo 2: ADR

**Arquivo**: `specs/technical/adr/001-nuxt-composition-api.md`

**Antes**:
```markdown
# ADR-001: Nuxt 4.2.2 Composition API

## Status
Accepted

## Context
...
```

**Depois**:
```markdown
---
adr_number: "001"
title: "Nuxt 4.2.2 Composition API"
date: "2025-12-23"
status: "accepted"
deciders: ["tech-lead", "frontend-team"]
consulted: []
informed: ["all-developers"]
supersedes: null
superseded_by: null
tags: ["architecture", "frontend", "vue", "nuxt"]
---

# ADR-001: Nuxt 4.2.2 Composition API

## Status
Accepted

## Context
...
```

### Exemplo 3: Feature Spec

**Arquivo**: `specs/business/features/digitalization.md`

**Antes**:
```markdown
# Feature: Sistema de Digitalização

## Objetivo
...
```

**Depois**:
```markdown
---
spec_version: "1.0.0"
valid_from: "2025-12-23"
last_updated: "2025-12-23"
status: "active"
priority: "high"
category: "feature"
tags: ["digitalization", "core-feature", "ai"]
related_specs:
  - "PRODUCT_STRATEGY.md"
  - "CUSTOMER_PERSONAS.md"
  - "../technical/CLAUDE.meta.md"
---

# Feature: Sistema de Digitalização

## Objetivo
...
```

---

## ✅ Checklist de Execução

**Para cada diretório**:

- [ ] `specs/_meta/` - Frontmatter de Meta Specs
- [ ] `specs/business/` - Frontmatter de Business Specs
- [ ] `specs/business/features/` - Frontmatter de Features
- [ ] `specs/technical/` - Frontmatter de Technical Specs
- [ ] `specs/technical/adr/` - Frontmatter de ADRs
- [ ] `docs/` - Frontmatter de Documentação Geral

**Para cada arquivo processado**:

- [ ] Frontmatter YAML válido
- [ ] Campos obrigatórios preenchidos
- [ ] Datas no formato correto (YYYY-MM-DD)
- [ ] Status válido
- [ ] Tags relevantes adicionadas
- [ ] Conteúdo original preservado
- [ ] Arquivo salvo com encoding UTF-8

---

## 🔧 Implementação

### Algoritmo Simplificado

```python
def add_frontmatter(file_path: str):
    """Adiciona frontmatter a um arquivo markdown."""

    # 1. Ler arquivo
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # 2. Verificar se já tem frontmatter
    if content.startswith('---'):
        print(f"✅ {file_path} já possui frontmatter")
        return

    # 3. Determinar tipo baseado no caminho
    frontmatter_type = determine_type(file_path)

    # 4. Extrair metadados do conteúdo
    title = extract_title(content)

    # 5. Gerar frontmatter
    frontmatter = generate_frontmatter(
        file_path=file_path,
        frontmatter_type=frontmatter_type,
        title=title
    )

    # 6. Inserir frontmatter
    new_content = f"{frontmatter}\n\n{content}"

    # 7. Salvar arquivo
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(new_content)

    print(f"✅ Frontmatter adicionado a {file_path}")
```

### Campos Obrigatórios por Tipo

**Spec**:
- `spec_version` (obrigatório)
- `valid_from` (obrigatório)
- `last_updated` (obrigatório)
- `status` (obrigatório)

**ADR**:
- `adr_number` (obrigatório)
- `title` (obrigatório)
- `date` (obrigatório)
- `status` (obrigatório)

**Feature**:
- `spec_version` (obrigatório)
- `status` (obrigatório)
- `priority` (obrigatório)

**Documentação**:
- `title` (obrigatório)
- `date` (obrigatório)

---

## 📊 Validação de Qualidade

Após adicionar frontmatter, validar:

```markdown
### Validação de Frontmatter

**Verificações automáticas**:
- [ ] YAML parsing sem erros
- [ ] Campos obrigatórios presentes
- [ ] Tipos de dados corretos (string, array, null)
- [ ] Datas em formato ISO (YYYY-MM-DD)
- [ ] Versões em SemVer (X.Y.Z)
- [ ] Status em lista válida
- [ ] Tags são array de strings

**Verificações manuais**:
- [ ] Tags são relevantes ao conteúdo
- [ ] Related specs existem e caminhos corretos
- [ ] Deciders/Consulted/Informed são pessoas reais (ADRs)
- [ ] Priority alinhado com roadmap (Features)
```

---

## 🎯 Benefícios

✅ **Versionamento**: Rastreamento de mudanças em specs
✅ **Metadados**: Informações contextuais acessíveis
✅ **Descoberta**: Tags facilitam busca e categorização
✅ **Governança**: Status e datas para auditoria
✅ **Relações**: Links entre specs relacionadas
✅ **Automação**: Frontmatter permite processamento automatizado

---

## 🚨 Cuidados

⚠️ **Backup**: Sempre fazer backup antes de modificar arquivos em lote
⚠️ **Encoding**: Usar UTF-8 para preservar caracteres especiais
⚠️ **Preservação**: Manter formatação e conteúdo original
⚠️ **Validação**: Verificar YAML parsing após inserção
⚠️ **Git**: Revisar diffs antes de commit

---

## 📚 Referências

- YAML Spec: https://yaml.org/spec/1.2/spec.html
- SemVer: https://semver.org/
- ADR Template: https://github.com/joelparkerhenderson/architecture-decision-record
- Frontmatter Examples: https://jekyllrb.com/docs/front-matter/

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

**Uso**:
- Sem argumentos: Processa todos os arquivos .md em `specs/` e `docs/`
- Com caminho: Processa apenas arquivo ou diretório especificado
- Com `--dry-run`: Mostra preview sem modificar arquivos
- Com `--validate`: Apenas valida frontmatter existente
- Com `--type=<tipo>`: Força tipo de frontmatter (spec|adr|feature|doc)

**Exemplos**:
```bash
# Adicionar frontmatter a todos os arquivos
/add-frontmatter

# Adicionar apenas a specs de negócio
/add-frontmatter specs/business/

# Preview sem modificar
/add-frontmatter --dry-run

# Validar frontmatter existente
/add-frontmatter --validate

# Forçar tipo ADR para um arquivo
/add-frontmatter specs/technical/adr/001-example.md --type=adr
```
