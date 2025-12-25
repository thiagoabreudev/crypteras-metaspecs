---
title: "Engenharia de Contexto - IA do Jeito Certo"
author: "Team"
date: "2025-12-23"
last_updated: "2025-12-23"
version: "1.0.0"
status: "published"
tags: ['documentation', 'quality', 'security', 'audit', 'compliance']
---

# Engenharia de Contexto - IA do Jeito Certo

Este diretório contém materiais sobre **Engenharia de Contexto** e **Context Governance** para desenvolvimento orientado por IA.

---

## 📚 Conteúdo

### [Slides Técnicos - Enriquecimento do Curso](slides_tecnicos_enriquecimento_do_curso_ia_do_jeito_certo.md)

Material complementar que aprofunda os conceitos técnicos de **Desenvolvimento Estruturado com IA**.

**Tópicos abordados**:
- Context Governance
- Intent as Code
- Versionamento Semântico de Contexto
- Métricas de Qualidade de Contexto
- Context Observability
- Failure Modes as Specs
- Explainability by Design
- Jidoka aplicado à IA
- Anti-Patterns em Desenvolvimento com IA

---

## 🛠️ Implementação Prática

Os conceitos dos slides foram implementados como **comandos executáveis** que você pode usar para melhorar suas metaspecs.

**Localização**: [.claude/commands/metaspecs/](../.claude/commands/metaspecs/)

---

## 🎯 Comandos de Context Governance

**Separação de Responsabilidades**:
- **📁 Metaspecs** (`workshop-metaspecs/`): Governança de especificações
- **💻 Workshop** (`workshop/`): Desenvolvimento e observabilidade de código

### 📊 Resumo dos Comandos

| Repo | # | Comando | Status | Localização |
|------|---|---------|--------|-------------|
| **Metaspecs** | 1 | `add-versioning` | ✅ Implementado | `.claude/commands/metaspecs/governance/` |
| **Metaspecs** | 2 | `add-intent` | ✅ Implementado | `.claude/commands/metaspecs/governance/` |
| **Metaspecs** | 3 | `add-failure-modes` | ✅ Implementado | `.claude/commands/metaspecs/governance/` |
| **Metaspecs** | 4 | `add-explainability` | ✅ Implementado | `.claude/commands/metaspecs/governance/` |
| **Metaspecs** | 5 | `add-anti-patterns` | ✅ Implementado | `.claude/commands/metaspecs/observability/` |
| **Metaspecs** | 6 | `validate-hierarchy` | ✅ Implementado | `.claude/commands/metaspecs/validation/` |
| **Workshop** | 7 | `validate-context` | ⚠️ TBD | `.claude/commands/quality/` |
| **Workshop** | 8 | `check-drift` | ⚠️ TBD | `.claude/commands/quality/` |
| **Workshop** | 9 | `quality:observe` | ✅ Existe | `.claude/commands/quality/observe.md` |

**Legenda**:
- ✅ **Implementado**: Comando criado e funcional
- ⚠️ **TBD**: A ser implementado futuramente no repositório workshop

---

## 📁 Repositório: workshop-metaspecs

Comandos para **governança de especificações** (não código).

### 🔐 Governance (6 comandos)

#### 1. `/metaspecs:governance:add-versioning` - Versionamento Semântico

**Conceito**: Context Governance (Slide 1-5)

**O que faz**: Adiciona versionamento semântico às specs para evitar Context Clash.

**Por que é importante**:
- IA sem versionamento ≈ código sem git
- Context Drift é inevitável sem controle de versões
- Permite rollback semântico quando specs mudam

**Como usar**:
```bash
/metaspecs:governance:add-versioning                 # Versiona todas as specs
/metaspecs:governance:add-versioning business        # Apenas specs de negócio
/metaspecs:governance:add-versioning technical       # Apenas specs técnicas
```

**Localização**: `.claude/commands/metaspecs/governance/add-versioning.md`

**O que é adicionado**:
```yaml
---
spec_version: "1.0.0"           # SemVer (MAJOR.MINOR.PATCH)
valid_from: "2025-12-20"        # Data de início
last_updated: "2025-12-20"      # Última modificação
supersedes: null                # Versão anterior
status: "active"                # active | deprecated | draft
---

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-20
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão versionada
:::
```

**Referência**: Slides 4-5 (Versionamento Semântico de Contexto)

---

#### 2. `/metaspecs:governance:add-intent` - Intent as Code

**Conceito**: Intent as Code (Slides 6-9)

**O que faz**: Formaliza intenção humana com goals, constraints e non-goals.

**Por que é importante**:
- Sem intenção explícita, IA otimiza o que não deveria
- Escopo se expande silenciosamente (scope creep)
- Non-goals são tão importantes quanto goals

**Como usar**:
```bash
/metaspecs:governance:add-intent                     # Adiciona Intent às specs críticas
/metaspecs:governance:add-intent technical           # Apenas specs técnicas
```

**Localização**: `.claude/commands/metaspecs/governance/add-intent.md`

**O que é adicionado**:
```markdown
:::intent
**Goal**: [O que queremos alcançar]

**Constraints** (limites obrigatórios):
- Manter retrocompatibilidade
- Não alterar contratos públicos
- Respeitar ADRs vigentes

**Non-Goals** (o que NÃO fazer):
- Refatoração ampla
- Mudanças de arquitetura
- Otimização prematura
:::
```

**Intent funciona como**:
- Contrato de execução
- Filtro de decisões
- Limite de autonomia da IA

**Referência**: Slides 6-9 (Intent as Code)

---

#### 3. `/metaspecs:governance:add-failure-modes` - Failure Modes as Specs

**Conceito**: Falhas Conhecidas como Primeira Classe (Slides 15-16)

**O que faz**: Documenta falhas previsíveis e suas mitigações.

**Por que é importante**:
- Mesmos erros se repetem sem documentação
- IA "alucina" em cenários conhecidos
- Context clash não tem estratégia de recuperação

**Como usar**:
```bash
/metaspecs:governance:add-failure-modes              # Adiciona a specs técnicas
/metaspecs:governance:add-failure-modes <spec>       # Spec específica
```

**Localização**: `.claude/commands/metaspecs/governance/add-failure-modes.md`

**O que é adicionado**:
```markdown
:::failure_modes
**Falhas Conhecidas**:

1. **Context Clash: "limite"**
   - **Tipo**: context_clash
   - **Descrição**: Termo com dois significados
   - **Gatilho**: Specs antigas e novas coexistindo
   - **Impacto**: 🔴 Crítico
   - **Mitigação**: Preferir spec mais recente
   - **Detecção**: Validação de spec_version

2. **Reactive Loss em Props**
   - **Tipo**: hallucination
   - **Descrição**: IA usa props sem reatividade
   - **Gatilho**: `const x = props.y`
   - **Impacto**: 🟡 Médio
   - **Mitigação**: Sempre usar `toRef`
   - **Detecção**: Props não atualizam
:::
```

**Tipos de falhas**:
- `context_clash` - Conflito entre versões
- `hallucination` - IA gera código incorreto mas plausível
- `integration` - Falhas de integração
- `validation` - Falhas de validação
- `security` - Falhas de segurança

**Referência**: Slides 15-16 (Failure Modes as Specs)

---

#### 4. `/metaspecs:governance:add-explainability` - Explainability by Design

**Conceito**: Explainability as a Requirement (Slides 17-18)

**O que faz**: Configura requisitos de explainability - IA deve explicar decisões.

**Por que é importante**:
- IA em produção precisa explicar decisões
- Sem explicação = caixa preta (impossível auditar)
- Compliance pode exigir explicabilidade

**Como usar**:
```bash
/metaspecs:governance:add-explainability             # Adiciona a specs críticas
/metaspecs:governance:add-explainability <spec>      # Spec específica
```

**Localização**: `.claude/commands/metaspecs/governance/add-explainability.md`

**O que é adicionado**:
```markdown
:::explainability
**Requirement**: ✅ Required | ⚠️ Recommended | ⭕ Optional

**Output Format**:
- **Decision**: [O que foi decidido]
- **Source**: [Qual spec + versão consultada]
- **Rationale**: [Por que esta decisão]
- **Alternatives Considered**: [Outras opções]
- **Trade-offs**: [Prós e contras]

**Audit Trail**:
- Timestamp de decisão
- Specs consultadas (nome + versão)
- Contexto relevante usado
:::
```

**Níveis de explainability**:
- ✅ **Required**: Decisões críticas DEVEM ser explicadas (bloqueia se não explicar)
- ⚠️ **Recommended**: Decisões importantes DEVERIAM ser explicadas
- ⭕ **Optional**: Decisões rotineiras (pode pular)

**Referência**: Slides 17-18 (Explainability by Design)

---

#### 5. `/metaspecs:observability:add-anti-patterns` - Anti-Patterns Documentation

**Conceito**: Anti-Patterns que Devem Ser Evitados (Slides 14-15)

**O que faz**: Documenta padrões a evitar no desenvolvimento com IA.

**Por que é importante**:
- Prevenir erros comuns
- Educar desenvolvedores
- IA aprende o que NÃO fazer (tão importante quanto o que fazer)

**Como usar**:
```bash
/metaspecs:observability:add-anti-patterns              # Cria doc com 10 principais
```

**Localização**: `.claude/commands/metaspecs/observability/add-anti-patterns.md`

**10 Anti-Patterns Principais**:
1. ❌ **Prompt-Only Development** - Sem specs estruturadas
2. ❌ **Context Dump** - Jogar todo contexto sem estrutura
3. ❌ **Specs Não Versionadas** - Sem versionamento
4. ❌ **RAG Sem Governança** - RAG sem validar qualidade
5. ❌ **Agentes Sem Limites** - Sem constraints claros
6. ❌ **Specs Genéricas** - Copy-paste de templates
7. ❌ **Documentação ≠ Código** - Docs desatualizados
8. ❌ **Sem Failure Modes** - Falhas não documentadas
9. ❌ **Explainability Ausente** - IA não explica decisões
10. ❌ **Hierarquia Inexistente** - Sem precedência clara

**Arquivo criado**: `specs/_meta/ANTI_PATTERNS.md`

**Referência**: Slides 14-15 (Anti-Patterns)

---

#### 6. `/metaspecs:validation:validate-hierarchy` - Context Hierarchy

**Conceito**: Context Architecture (Slide 20)

**O que faz**: Valida que hierarquia de contexto está definida e sendo respeitada.

**Por que é importante**:
- Quando specs conflitam, qual seguir?
- Camadas superiores sempre vencem conflitos
- Sem hierarquia = decisões inconsistentes

**Como usar**:
```bash
/metaspecs:validation:validate-hierarchy             # Valida hierarquia completa
/metaspecs:validation:validate-hierarchy --strict    # Warnings = erros
```

**Localização**: `.claude/commands/metaspecs/validation/validate-hierarchy.md`

**Hierarquia esperada**:
```
1. Meta Specs (_meta/)         → Maior precedência
   ↓ (governa)
2. Business Specs (business/)  → Precedência alta
   ↓ (governa)
3. Technical Specs (technical/) → Precedência média
   ↓ (governa)
4. Execution Context (sessions/) → Menor precedência
```

**Regra de Ouro**: Camadas superiores sempre vencem.

**Validações**:
- ✅ Documentação (`CONTEXT_HIERARCHY.md` existe)
- ✅ Estrutura (camadas organizadas)
- ✅ Conflitos (detecta e valida resoluções)
- ✅ ADRs (ADRs > Guidelines)
- ✅ Referências (inferiores → superiores, não vice-versa)

**Arquivo criado**: `specs/_meta/CONTEXT_HIERARCHY.md`

**Referência**: Slide 20 (Context Architecture)

---

## 💻 Repositório: workshop

Comandos para **desenvolvimento e observabilidade de código** (não specs).

**Nota**: Estes comandos operam no repositório de código, não no repositório de metaspecs.

### ✅ Validation (2 comandos)

#### 7. `/validate-context` - Jidoka (Fail-Fast)

**Conceito**: Jidoka aplicado à IA (Slide 19)

**O que faz**: Valida contexto ANTES de execução (erro detectado → processo para).

**Por que é importante**:
- Prevenir erros ao invés de corrigi-los depois
- Contexto inválido → execução bloqueada
- Spec violada → correção antes de seguir

**Como usar**:
```bash
/validate-context               # Valida tudo
/validate-context business      # Apenas business
/validate-context --strict      # Warnings = erros
/validate-context --quick       # Só validações bloqueantes
```

**Validações realizadas**:
1. ✅ **Estrutural**: Arquivos obrigatórios existem?
2. ✅ **Versionamento**: Todas specs versionadas?
3. ⚠️ **Intent**: Specs críticas têm Intent?
4. ⚠️ **Failure Modes**: Falhas documentadas?
5. ✅ **Explainability**: Requirements configurados?
6. ✅ **Consistência**: Sem conflitos entre specs?
7. ✅ **Hierarquia**: Precedência clara?

**Ação se falhar**: 🛑 BLOQUEIA execução até correção.

**Referência**: Slide 19 (Jidoka aplicado à IA)

---

**Localização**: `workshop/.claude/commands/quality/` (TBD - a ser criado)

---

#### 8. `/check-drift` - Context Drift Detection

**Conceito**: Context Drift (Slide 2)

**O que faz**: Detecta quando contexto ficou desatualizado em relação ao código real.

**Por que é importante**:
- Context drift degrada qualidade do sistema ao longo do tempo
- Decisões inconsistentes entre execuções
- Conflitos entre documentação, código e IA

**Como usar**:
```bash
/check-drift                    # Verifica tudo
/check-drift stack              # Apenas stack tecnológica
/check-drift --critical-only    # Só drifts críticos
/check-drift --age-threshold=90 # Specs > 90 dias
```

**Tipos de drift detectados**:
- 📝 **Documentation Drift**: Spec documenta o que não existe
- 💻 **Implementation Drift**: Código implementa o que spec não documenta
- 🔤 **Semantic Drift**: Termo muda de significado sem atualização
- 🏗️ **Architectural Drift**: Stack real ≠ stack documentada

**Verificações**:
- Stack tecnológica (specs vs package.json)
- Features (specs vs endpoints/componentes)
- ADRs (decisões vs código real)
- Componentes (doc vs arquivos)
- Termos (definições vs uso)
- Atualização (specs antigas > 120 dias)

**Referência**: Slide 2 (Por que Context Governance é Necessário)

**Localização**: `workshop/.claude/commands/quality/` (TBD - a ser criado)

---

### 📊 Observability (1 comando)

#### 9. `/quality:observe` - Context Observability

**Conceito**: Garantia de Qualidade (Checklist do final dos slides)

**O que faz**: Observabilidade de qualidade de código e métricas de desenvolvimento.

**Por que é importante**:
- Se não observamos contexto, não controlamos qualidade da IA
- Quais specs são realmente úteis?
- Quais nunca são usadas? (candidatas a remoção)

**Como usar**:
```bash
/quality:observe                # Observa qualidade atual
/quality:metrics                # Métricas de desenvolvimento
```

**Localização**: `workshop/.claude/commands/quality/observe.md` (já existe)

**Métricas observadas**:
- Qualidade de código
- Cobertura de testes
- Performance
- Decisões técnicas

**Referência**: Slides 12-13 (Context Observability)

---

## 🔄 Workflow Recomendado

### 1️⃣ Setup Inicial - Metaspecs (Uma vez)

Execute os comandos de governance no repositório **workshop-metaspecs**:

```bash
# 1. Versionamento
/metaspecs:governance:add-versioning

# 2. Intent as Code
/metaspecs:governance:add-intent

# 3. Failure Modes
/metaspecs:governance:add-failure-modes

# 4. Explainability
/metaspecs:governance:add-explainability

# 5. Anti-Patterns
/metaspecs:observability:add-anti-patterns

# 6. Hierarquia
/metaspecs:validation:validate-hierarchy
```

**Resultado**: Sistema de governança completo no repositório de specs.

### 2️⃣ Durante Desenvolvimento - Workshop

No repositório **workshop** (código):

**IA automaticamente**:
- ✅ Consulta specs versionadas do repositório metaspecs
- ✅ Respeita Intent (constraints + non-goals)
- ✅ Evita failure modes conhecidos
- ✅ Explica decisões importantes

**Comandos disponíveis**:
```bash
/validate-context          # Valida contexto antes de começar (TBD)
/check-drift               # Detecta drift code vs specs (TBD)
/quality:observe           # Observa qualidade de código (existe)
```

### 3️⃣ Manutenção de Metaspecs

**Quando specs mudam** (no repositório **metaspecs**):
```bash
# Atualizar versão conforme VERSIONING_POLICY.md
# Incrementar MAJOR, MINOR ou PATCH
# Atualizar VERSION_HISTORY.md
```

**Validação periódica**:
```bash
/metaspecs:validation:validate-hierarchy       # Mensal
```

---

## 📊 Métricas de Qualidade de Contexto

### Context Quality Metrics (Slide 10-11)

**Se não medimos contexto, não controlamos qualidade da IA.**

**Métricas Recomendadas**:
1. **Context Relevance Score**: Specs são consultadas?
2. **Spec Coverage Ratio**: % do código coberto por specs
3. **Hallucination Rate**: Quantas vezes IA errou?
4. **Context Drift Incidents**: Quantos drifts detectados?
5. **Taxa de Retrabalho Pós-IA**: Quanto código precisa ser corrigido?

**Como medir**:
- `/track-context metrics` - Uso de specs
- `/check-drift` - Drift incidents
- `/validate-context` - Qualidade geral
- `/audit-spec` - Qualidade individual

---

## 🎯 Resultados Esperados

### ✅ Repositório Metaspecs (Governança)

Após implementar os 6 comandos:

**Para o Contexto**:
✅ **Versionado** - Sem Context Clash (40/40 specs = 100%)
✅ **Governado** - Com Intent e Constraints
✅ **Documentado** - Failure Modes e Anti-Patterns
✅ **Hierárquico** - Precedência clara (Meta > Business > Technical)
✅ **Rastreável** - VERSION_HISTORY.md completo

**Arquivos Criados** (9):
- VERSIONING_POLICY.md
- VERSION_HISTORY.md
- INTENT_AS_CODE_GUIDE.md
- FAILURE_MODES_GUIDE.md
- EXPLAINABILITY_GUIDE.md
- ANTI_PATTERNS.md
- CONTEXT_HIERARCHY.md
- DECISION_LOG.md
- README.md

---

### 💻 Repositório Workshop (Código)

**Para a IA**:
✅ **Clareza** - Consulta specs versionadas
✅ **Limites** - Respeita constraints e non-goals
✅ **Prevenção** - Evita failure modes conhecidos
✅ **Transparência** - Explica decisões significativas
✅ **Consistência** - Decisões alinhadas com specs

**Para o Time**:
✅ **Confiança** - IA gera código seguindo specs
✅ **Eficiência** - Menos retrabalho
✅ **Aprendizado** - Anti-patterns documentados
✅ **Autonomia** - IA trabalha com limites claros
✅ **Escalabilidade** - Specs crescem sustentavelmente

---

## 📚 Referências Completas

### Material Base
- **Slides Técnicos**: [slides_tecnicos_enriquecimento_do_curso_ia_do_jeito_certo.md](slides_tecnicos_enriquecimento_do_curso_ia_do_jeito_certo.md)
- **Curso**: [IA do Jeito Certo](https://iadojeitocerto.com.br)

### Implementação
- **Comandos**: [../.claude/commands/metaspecs/](../.claude/commands/metaspecs/)
- **README Técnico**: [../.claude/commands/metaspecs/README.md](../.claude/commands/metaspecs/README.md)

### Conceitos por Slide
- **Slides 1-5**: Context Governance e Versionamento
- **Slides 6-9**: Intent as Code
- **Slides 10-11**: Métricas de Qualidade
- **Slides 12-13**: Context Observability
- **Slides 14-15**: Anti-Patterns
- **Slides 15-16**: Failure Modes
- **Slides 17-18**: Explainability
- **Slide 19**: Jidoka
- **Slide 20**: Context Architecture (Hierarquia)
- **Slide 21**: IA como Sistema Sociotécnico
- **Slide 22-23**: Mensagem Final

---

## 🚀 Próximos Passos

1. ✅ **Ler os slides técnicos** para entender conceitos
2. ✅ **Executar setup inicial** (7 comandos acima)
3. ✅ **Integrar no workflow** (validação antes de desenvolvimento)
4. ✅ **Educar time** sobre anti-patterns
5. ✅ **Monitorar métricas** com tracking
6. ✅ **Iterar e melhorar** continuamente

---

## 💡 Citação Final

> **"A vantagem competitiva não está em usar IA, mas em usar IA melhor que os outros."**

Estes comandos implementam as melhores práticas de engenharia de contexto para desenvolvimento orientado por IA, baseadas no curso **IA do Jeito Certo**.

---

**Versão**: 1.0.0
**Data**: 2025-12-20
**Autor**: Workshop IA do Jeito Certo
