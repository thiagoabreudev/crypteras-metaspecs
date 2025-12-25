# Validar Contexto (Jidoka) - Crypteras

Este comando implementa **Jidoka** (princípio de fail-fast) para validar contexto do **Crypteras Trading System** antes de execução.

> **Nota**: Este é um comando avançado de validação. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Jidoka Aplicado à IA

**Jidoka** é o princípio Toyota de "parar a linha de produção" quando erro é detectado.

**Aplicado à IA**:
- Erro detectado → processo para
- Contexto inválido → execução bloqueada
- Spec violada → correção antes de seguir

**Benefício**: Prevenir erros ao invés de corrigi-los depois.

---

## 📋 Objetivo

Validar contexto ANTES de executar para:
- Detectar specs inválidas
- Identificar context clash
- Verificar versões compatíveis
- Bloquear execução se contexto corrompido

---

## 🔍 Processo de Validação

### 1. Validações Estruturais

**Verificar existência de arquivos obrigatórios**:

```bash
# Índices principais
specs/index.md                    ✅ Required
specs/business/index.md           ✅ Required
specs/technical/index.md          ✅ Required

# Specs de negócio (mínimo)
specs/business/CUSTOMER_PERSONAS.md    ✅ Required
specs/business/PRODUCT_STRATEGY.md     ✅ Required

# Specs técnicas (mínimo)
specs/technical/CLAUDE.meta.md         ✅ Required
specs/technical/CODEBASE_GUIDE.md      ✅ Required
specs/technical/adr/                   ✅ Required (diretório)
```

**Saída esperada**:
```markdown
## ✅ Validação Estrutural

**Índices**:
- ✅ specs/index.md
- ✅ specs/business/index.md
- ✅ specs/technical/index.md

**Specs de Negócio**:
- ✅ CUSTOMER_PERSONAS.md
- ✅ PRODUCT_STRATEGY.md
- ✅ PRODUCT_METRICS.md
- ...

**Specs Técnicas**:
- ✅ CLAUDE.meta.md
- ✅ CODEBASE_GUIDE.md
- ✅ adr/ (3 ADRs encontrados)
- ...

**Status**: ✅ Estrutura válida
```

### 2. Validações de Versionamento

**Verificar que todas as specs têm versionamento**:

```markdown
## Validação de Versionamento

**Verificando frontmatter obrigatório...**

Para cada spec:
- `spec_version` presente? ✅ / ❌
- `valid_from` presente? ✅ / ❌
- `last_updated` presente? ✅ / ❌
- `status` presente e válido (active/deprecated/draft)? ✅ / ❌
- Formato SemVer correto? ✅ / ❌
```

**Saída esperada**:
```markdown
## ✅ Validação de Versionamento

**Specs de Negócio**:
- ✅ CUSTOMER_PERSONAS.md v1.0.0 (active)
- ✅ PRODUCT_STRATEGY.md v1.1.0 (active)
- ❌ VOICE_OF_CUSTOMER.md - **FALTA versionamento**

**Specs Técnicas**:
- ✅ CLAUDE.meta.md v1.2.0 (active)
- ✅ CODEBASE_GUIDE.md v1.0.0 (active)
- ❌ adr/002-atomic-design.md - **FALTA versionamento**

**Status**: ⚠️ 2 specs sem versionamento (bloqueando execução)
```

**Ação se falhar**:
```
🛑 VALIDAÇÃO FALHOU

Specs sem versionamento:
1. specs/business/VOICE_OF_CUSTOMER.md
2. specs/technical/adr/002-atomic-design.md

📋 Solução:
Execute: /add-versioning business technical/adr

Não é possível prosseguir com contexto não versionado.
```

### 3. Validações de Intent

**Verificar que specs críticas têm Intent as Code**:

```markdown
## Validação de Intent

**Specs que DEVEM ter Intent**:
- specs/technical/CLAUDE.meta.md ✅ / ❌
- specs/technical/CODEBASE_GUIDE.md ✅ / ❌
- specs/technical/API_SPECIFICATION.md ✅ / ❌
- specs/business/PRODUCT_STRATEGY.md ✅ / ❌
- Cada ADR ✅ / ❌
```

**Saída esperada**:
```markdown
## ⚠️ Validação de Intent

**Specs Técnicas**:
- ✅ CLAUDE.meta.md - Intent presente (goal + constraints + non-goals)
- ✅ CODEBASE_GUIDE.md - Intent presente
- ❌ API_SPECIFICATION.md - **FALTA seção :::intent**

**ADRs**:
- ✅ adr/001-vue3-composition-api.md - Intent presente
- ❌ adr/002-atomic-design.md - **FALTA seção :::intent**

**Status**: ⚠️ Recomendado adicionar Intent (não bloqueante)
```

**Ação se falhar (warning, não bloqueia)**:
```
⚠️ INTENT AUSENTE (recomendado)

Specs críticas sem Intent:
1. specs/technical/API_SPECIFICATION.md
2. specs/technical/adr/002-atomic-design.md

💡 Recomendação:
Execute: /add-intent technical

Intent melhora qualidade das decisões da IA.
Prosseguir sem Intent? (requer confirmação)
```

### 4. Validações de Failure Modes

**Verificar que specs críticas documentam falhas conhecidas**:

```markdown
## Validação de Failure Modes

**Specs que DEVEM ter Failure Modes**:
- specs/technical/CLAUDE.meta.md ✅ / ❌
- specs/technical/BUSINESS_LOGIC.md ✅ / ❌
- specs/technical/API_SPECIFICATION.md ✅ / ❌
- Features críticas (auth, etc) ✅ / ❌
```

**Saída esperada**:
```markdown
## ⚠️ Validação de Failure Modes

**Specs Técnicas**:
- ✅ CLAUDE.meta.md - 4 failure modes documentados
- ❌ BUSINESS_LOGIC.md - **FALTA seção :::failure_modes**
- ✅ API_SPECIFICATION.md - 3 failure modes documentados

**Features Críticas**:
- ✅ features/authentication.md - 3 failure modes documentados
- ❌ features/goal-management.md - **FALTA seção :::failure_modes**

**Status**: ⚠️ Recomendado documentar failure modes (não bloqueante)
```

### 5. Validações de Explainability

**Verificar requisitos de explainability**:

```markdown
## Validação de Explainability

**Specs com Explainability Required**:
- specs/technical/CLAUDE.meta.md ✅ / ❌
- specs/technical/BUSINESS_LOGIC.md ✅ / ❌
- Cada ADR ✅ / ❌
```

**Saída esperada**:
```markdown
## ✅ Validação de Explainability

**Specs Técnicas**:
- ✅ CLAUDE.meta.md - Explainability: Required
- ✅ BUSINESS_LOGIC.md - Explainability: Required
- ✅ API_SPECIFICATION.md - Explainability: Required

**ADRs**:
- ✅ Todos os ADRs têm format de explainability

**Status**: ✅ Explainability configurado corretamente
```

### 6. Validações de Consistência

**Verificar consistência entre specs**:

```markdown
## Validação de Consistência

**Stack tecnológica**:
- specs/technical/index.md define stack?
- CLAUDE.meta.md alinhado com stack?
- ADRs alinhados com stack?

**Personas**:
- CUSTOMER_PERSONAS.md define personas?
- Features referenciam personas válidas?

**Roadmap**:
- PRODUCT_STRATEGY.md define roadmap?
- Features estão no roadmap ou justificadas?
```

**Exemplo de inconsistência detectada**:
```markdown
## ❌ Inconsistência Detectada

**Conflito de Stack**:
- specs/technical/index.md v1.0.0: "Frontend: Vue 3"
- specs/technical/adr/003-react-migration.md v1.0.0: "Decisão: Migrar para React"

🛑 Context Clash: Specs conflitantes sobre stack frontend

📋 Solução:
1. Atualizar specs/technical/index.md para React (breaking change, v2.0.0)
2. OU deprecar adr/003-react-migration.md (decisão cancelada)

Execução bloqueada até resolver conflito.
```

### 7. Validações de Hierarquia

**Verificar que hierarquia de contexto está clara**:

```markdown
## Validação de Hierarquia

**Regra de Precedência Esperada**:
1. Meta Specs (specs/index.md)
2. Business Specs (specs/business/)
3. Technical Specs (specs/technical/)
4. Execution Context (.claude/sessions/)

**Verificações**:
- specs/_meta/CONTEXT_HIERARCHY.md existe? ✅ / ❌
- Regras de resolução de conflito documentadas? ✅ / ❌
- Hierarquia clara para IA? ✅ / ❌
```

---

## ✅ Checklist de Validação Completa

Executar TODAS as validações:

- [ ] **Estrutural**: Arquivos obrigatórios existem
- [ ] **Versionamento**: Todas as specs versionadas
- [ ] **Intent**: Specs críticas têm Intent (warning)
- [ ] **Failure Modes**: Specs críticas documentam falhas (warning)
- [ ] **Explainability**: Requirements configurados
- [ ] **Consistência**: Sem conflitos entre specs
- [ ] **Hierarquia**: Precedência clara

---

## 🚨 Níveis de Severidade

### 🔴 Crítico (BLOQUEIA execução)
- Specs obrigatórias faltando
- Specs sem versionamento
- Context clash detectado
- Inconsistências críticas

**Ação**: 🛑 PARE execução, não prossiga sem correção

### 🟡 Warning (RECOMENDA correção)
- Intent faltando em specs críticas
- Failure modes não documentados
- Explainability ausente

**Ação**: ⚠️ ALERTE usuário, prossiga se aprovado

### 🟢 Info (SUGERE melhoria)
- Specs opcionais faltando
- Documentação incompleta
- Oportunidades de melhoria

**Ação**: 💡 INFORME usuário, prossiga normalmente

---

## 📊 Formato de Saída

```markdown
# 🔍 Validação de Contexto

**Data**: 2025-12-20T15:00:00Z
**Diretório**: /Users/user/project/specs/

---

## 1. ✅ Validação Estrutural
[Resultado detalhado]

## 2. ✅ Validação de Versionamento
[Resultado detalhado]

## 3. ⚠️ Validação de Intent
[Resultado detalhado com warnings]

## 4. ⚠️ Validação de Failure Modes
[Resultado detalhado com warnings]

## 5. ✅ Validação de Explainability
[Resultado detalhado]

## 6. ❌ Validação de Consistência
[Resultado detalhado com erros CRÍTICOS]

## 7. ✅ Validação de Hierarquia
[Resultado detalhado]

---

## 📋 Resumo

| Validação | Status | Bloqueante |
|-----------|--------|------------|
| Estrutural | ✅ | Sim |
| Versionamento | ✅ | Sim |
| Intent | ⚠️ | Não |
| Failure Modes | ⚠️ | Não |
| Explainability | ✅ | Não |
| Consistência | ❌ | Sim |
| Hierarquia | ✅ | Sim |

**Resultado Geral**: ❌ FALHOU (1 crítico, 2 warnings)

---

## 🛑 Ações Necessárias (BLOQUEANTES)

### Crítico 1: Context Clash em Stack
- **Problema**: specs/technical/index.md vs adr/003-react-migration.md
- **Impacto**: IA não sabe qual stack usar
- **Solução**: Resolver conflito (atualizar ou deprecar)

---

## ⚠️ Ações Recomendadas (NÃO bloqueantes)

### Warning 1: Intent faltando
- **Specs**: API_SPECIFICATION.md, adr/002
- **Solução**: /add-intent technical

### Warning 2: Failure Modes faltando
- **Specs**: BUSINESS_LOGIC.md, goal-management.md
- **Solução**: /add-failure-modes technical business/features

---

**Pode prosseguir?** ❌ NÃO (resolva críticos primeiro)
```

---

## 🔄 Uso no Workflow

**Quando executar validação**:

1. **Antes de `/start`** (início de desenvolvimento)
2. **Após atualizar specs** (verificar não quebrou nada)
3. **Antes de `/pr`** (garantir contexto válido)
4. **Periodicamente** (detectar context drift)

**Integração com comandos**:

```markdown
# Exemplo: /start com validação automática

1. Usuário executa: /start WOR-123
2. Sistema executa automaticamente: /validate-context
3. Se validação passar (✅): Prossegue com /start
4. Se validação falhar (❌): BLOQUEIA /start, mostra erros
```

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, valida TODAS as specs.

Se argumentos forem fornecidos (ex: `business` ou `technical`), valida apenas specs especificadas.

Flags opcionais:
- `--strict`: Trata warnings como erros (bloqueia execução)
- `--quick`: Pula validações não-bloqueantes (só estrutural + versionamento + consistência)
