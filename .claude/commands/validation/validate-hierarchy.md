# Validar Hierarquia de Contexto - Crypteras

Este comando valida que a hierarquia de contexto do **Crypteras Trading System** está definida e sendo respeitada.

> **Nota**: Este é um comando avançado de validação. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Context Hierarchy

**Hierarquia de Contexto** define precedência quando specs conflitam:

```
1. Meta Specs (specs/_meta/)         → Maior precedência
2. Business Specs (specs/business/)  → Precedência alta
3. Technical Specs (specs/technical/) → Precedência média
4. Execution Context (.claude/sessions/) → Menor precedência
```

**Regra de Ouro**: Camadas superiores sempre vencem conflitos.

---

## 📋 Objetivo

Validar hierarquia para:
- Garantir precedência está clara
- Detectar conflitos entre camadas
- Verificar resoluções corretas
- Documentar regras de conflito

---

## 🔍 Processo de Validação

### 1. Verificar Documentação de Hierarquia

**Arquivo obrigatório**: `specs/_meta/CONTEXT_HIERARCHY.md`

```markdown
## Validação de Documentação

**Verificar que existe**:
- [ ] specs/_meta/CONTEXT_HIERARCHY.md

**Verificar conteúdo**:
- [ ] Define camadas claramente
- [ ] Ordem de precedência explícita
- [ ] Regras de resolução de conflito
- [ ] Exemplos de conflitos e resoluções
```

**Exemplo de arquivo esperado**:
```markdown
# Hierarquia de Contexto - MetaCerta

## Camadas de Contexto (Maior → Menor Precedência)

### Camada 1: Meta Specs
**Path**: `specs/_meta/`
**Precedência**: Máxima
**Propósito**: Governança, versionamento, hierarquia

**Arquivos**:
- `CONTEXT_HIERARCHY.md` - Este arquivo
- `VERSION_HISTORY.md` - Histórico de versões
- `VERSIONING_POLICY.md` - Política de versionamento

**Regra**: Meta specs definem COMO outras specs funcionam.

---

### Camada 2: Business Specs
**Path**: `specs/business/`
**Precedência**: Alta
**Propósito**: Contexto de negócio, produto, clientes

**Regra**: Business specs definem O QUE construir (requisitos, features, estratégia).

**Exemplos**:
- PRODUCT_STRATEGY.md
- CUSTOMER_PERSONAS.md
- features/*.md

---

### Camada 3: Technical Specs
**Path**: `specs/technical/`
**Precedência**: Média
**Propósito**: Arquitetura, código, implementação

**Regra**: Technical specs definem COMO construir (stack, padrões, ADRs).

**Exemplos**:
- CLAUDE.meta.md
- CODEBASE_GUIDE.md
- adr/*.md

---

### Camada 4: Execution Context
**Path**: `.claude/sessions/`
**Precedência**: Mínima
**Propósito**: Contexto de sessão específica

**Regra**: Execution context é volátil e específico da task.

**Exemplos**:
- .claude/sessions/WOR-123/context.md
- .claude/sessions/WOR-123/architecture.md

---

## Resolução de Conflitos

### Regra Geral
**Camada superior sempre vence.**

### Exemplos de Conflitos

#### Conflito: Business vs Technical
```yaml
# Business spec diz:
PRODUCT_STRATEGY.md: "Feature X é prioridade máxima"

# Technical spec diz:
TROUBLESHOOTING.md: "Feature X não é possível devido a limitação técnica"
```

**Resolução**:
1. **Business spec vence** (precedência maior)
2. Technical spec deve documentar mitigação ou workaround
3. Se impossível: Escalar para humano (não bloqueável por tech)

#### Conflito: Meta vs Business
```yaml
# Meta spec diz:
VERSIONING_POLICY.md: "Toda spec DEVE ter spec_version"

# Business spec:
VOICE_OF_CUSTOMER.md: Sem frontmatter de versão
```

**Resolução**:
1. **Meta spec vence** (precedência máxima)
2. Business spec DEVE ser atualizada para adicionar versão
3. Bloqueio até conformidade

#### Conflito: Technical ADR vs Technical Guide
```yaml
# ADR diz:
adr/001-vue3.md: "Usar Vue 3 Composition API"

# Guide diz:
CLAUDE.meta.md: "Aceito usar Options API em casos específicos"
```

**Resolução**:
1. **ADR vence** (decisões arquiteturais > guidelines)
2. CLAUDE.meta.md deve alinhar com ADR
3. Se exceção é válida: Criar novo ADR documentando exceção
```
```

**Saída de validação**:
```markdown
### Documentação de Hierarquia: ✅ PRESENTE

- ✅ specs/_meta/CONTEXT_HIERARCHY.md existe
- ✅ Define 4 camadas claramente
- ✅ Ordem de precedência explícita
- ✅ Regras de resolução documentadas
- ✅ Exemplos de conflitos presentes

**Status**: ✅ Hierarquia bem documentada
```

### 2. Validar Estrutura de Diretórios

**Verificar que camadas estão organizadas corretamente**:

```markdown
## Validação de Estrutura

**Estrutura esperada**:
```
specs/
├── _meta/              # Camada 1 (Meta)
│   ├── CONTEXT_HIERARCHY.md
│   ├── VERSION_HISTORY.md
│   └── VERSIONING_POLICY.md
├── business/           # Camada 2 (Business)
│   ├── index.md
│   ├── CUSTOMER_PERSONAS.md
│   ├── PRODUCT_STRATEGY.md
│   └── features/
│       └── *.md
├── technical/          # Camada 3 (Technical)
│   ├── index.md
│   ├── CLAUDE.meta.md
│   ├── CODEBASE_GUIDE.md
│   └── adr/
│       └── *.md
└── index.md
```

**Verificações**:
- [ ] specs/_meta/ existe (Camada 1)
- [ ] specs/business/ existe (Camada 2)
- [ ] specs/technical/ existe (Camada 3)
- [ ] .claude/sessions/ existe (Camada 4)
- [ ] Sem specs fora das camadas definidas
```

**Saída de validação**:
```markdown
### Estrutura de Camadas: ✅ VÁLIDA

- ✅ specs/_meta/ presente (Camada 1)
- ✅ specs/business/ presente (Camada 2)
- ✅ specs/technical/ presente (Camada 3)
- ✅ .claude/sessions/ presente (Camada 4)
- ✅ Sem specs soltas (todas em camadas válidas)

**Status**: ✅ Estrutura alinhada com hierarquia
```

### 3. Detectar Conflitos Entre Camadas

**Verificar se há specs conflitantes e se resolução é correta**:

```markdown
## Detecção de Conflitos

**Conflitos conhecidos a verificar**:

### Business vs Technical
- Business requer feature X
- Technical documenta limitação para feature X
→ Verificar se há mitigação ou escalação

### Meta vs Business
- Meta requer versionamento
- Business spec sem versão
→ Verificar se business spec está em conformidade

### Meta vs Technical
- Meta define padrão Y
- Technical spec viola padrão Y
→ Verificar se technical spec está em conformidade

### Technical ADR vs Technical Guide
- ADR decide Z
- Guide contradiz Z
→ Verificar se guide está alinhado com ADR
```

**Exemplo de conflito detectado**:
```markdown
### 🔴 Conflito Detectado

**Tipo**: Meta vs Business

**Camada Superior** (Meta):
- Spec: specs/_meta/VERSIONING_POLICY.md
- Regra: "Toda spec DEVE ter spec_version"

**Camada Inferior** (Business):
- Spec: specs/business/VOICE_OF_CUSTOMER.md
- Violação: Frontmatter sem spec_version

**Precedência**: Meta > Business

**Resolução Esperada**:
✅ Adicionar spec_version a VOICE_OF_CUSTOMER.md

**Status Atual**: ❌ Violação não resolvida

📋 Ação Necessária:
Execute: /add-versioning business
```

**Exemplo de conflito resolvido corretamente**:
```markdown
### ✅ Conflito Resolvido Corretamente

**Tipo**: Business vs Technical

**Camada Superior** (Business):
- Spec: specs/business/PRODUCT_STRATEGY.md
- Requisito: "OAuth com Google é prioridade Q1"

**Camada Inferior** (Technical):
- Spec: specs/technical/features/authentication.md
- Limitação: "OAuth não está no escopo do MVP"
- Mitigação: "Planejado para v2.0 (Q2)"

**Precedência**: Business > Technical

**Resolução**:
✅ Technical spec documenta limitação
✅ Technical spec propõe timeline alternativo (Q2)
✅ Escopo do MVP mantém email/senha (viável)
✅ Roadmap atualizado com OAuth em Q2

**Status**: ✅ Conflito resolvido adequadamente
```

### 4. Validar ADRs vs Guidelines

**Verificar que ADRs (decisões) têm precedência sobre guidelines**:

```markdown
## ADRs vs Guidelines

**Regra**: ADRs > Guidelines (decisões > orientações)

**Verificações**:

### ADR-001: Vue 3 Composition API
- ADR decide: "100% Composition API"
- CLAUDE.meta.md diz: ?

✅ CLAUDE.meta.md: "Usar Composition API (conforme ADR-001)"
→ Alinhado

---

### ADR-007: Pinia State Management
- ADR decide: "Usar Pinia"
- CODEBASE_GUIDE.md diz: ?

❌ CODEBASE_GUIDE.md: "Estado global com Vuex"
→ Desalinhado (viola ADR)

📋 Ação: Atualizar CODEBASE_GUIDE.md para alinhar com ADR-007
```

### 5. Validar Referências Entre Camadas

**Verificar que specs referenciam camadas superiores corretamente**:

```markdown
## Referências Entre Camadas

**Regra**: Camadas inferiores PODEM referenciar superiores (não vice-versa).

**Válido** ✅:
- Technical spec referencia Business spec
- Business spec referencia Meta spec
- Execution context referencia qualquer spec

**Inválido** ❌:
- Meta spec referencia Business/Technical
- Business spec referencia Technical
- Spec de camada superior depende de inferior

**Verificações**:

### specs/business/features/authentication.md
- Referencia: CUSTOMER_PERSONAS.md (mesma camada) ✅
- Referencia: PRODUCT_STRATEGY.md (mesma camada) ✅

### specs/technical/CLAUDE.meta.md
- Referencia: PRODUCT_STRATEGY.md (camada superior) ✅
- Referencia: adr/001-vue3.md (mesma camada) ✅

### specs/_meta/CONTEXT_HIERARCHY.md
- Referencia: CLAUDE.meta.md ❌ **VIOLAÇÃO**
→ Meta spec não deve depender de Technical spec

**Resultado**: ⚠️ 1 violação de hierarquia detectada
```

---

## ✅ Checklist de Validação

Executar TODAS as verificações:

- [ ] **Documentação**: CONTEXT_HIERARCHY.md existe e está completo
- [ ] **Estrutura**: Camadas organizadas corretamente
- [ ] **Conflitos**: Sem conflitos não resolvidos entre camadas
- [ ] **ADRs**: ADRs têm precedência sobre guidelines
- [ ] **Referências**: Camadas inferiores referenciam superiores (não vice-versa)

---

## 📊 Formato de Saída

```markdown
# 🔍 Validação de Hierarquia de Contexto

**Data**: 2025-12-20T16:30:00Z
**Diretório**: /Users/user/project/specs/

---

## 1. ✅ Documentação de Hierarquia
CONTEXT_HIERARCHY.md presente e completo.

## 2. ✅ Estrutura de Camadas
Todas as camadas organizadas corretamente.

## 3. 🔴 Conflitos Entre Camadas
1 conflito não resolvido detectado.

## 4. ⚠️ ADRs vs Guidelines
1 guideline viola ADR.

## 5. ⚠️ Referências Entre Camadas
1 referência inválida (Meta → Technical).

---

## 📋 Resumo

| Validação | Status | Bloqueante |
|-----------|--------|------------|
| Documentação | ✅ | Sim |
| Estrutura | ✅ | Sim |
| Conflitos | 🔴 | Sim |
| ADRs | ⚠️ | Não |
| Referências | ⚠️ | Não |

**Resultado Geral**: 🔴 FALHOU (1 bloqueante)

---

## 🛑 Conflitos Bloqueantes

### 1. Meta vs Business (Versionamento)
- **Superior**: VERSIONING_POLICY.md requer spec_version
- **Inferior**: VOICE_OF_CUSTOMER.md sem versão
- **Ação**: Execute /add-versioning business

---

## ⚠️ Violações Não-Bloqueantes

### 1. ADR vs Guideline
- **ADR**: 007-pinia.md decide "Usar Pinia"
- **Guide**: CODEBASE_GUIDE.md menciona "Vuex"
- **Ação**: Alinhar CODEBASE_GUIDE.md com ADR-007

### 2. Referência Inválida
- **Meta**: CONTEXT_HIERARCHY.md referencia CLAUDE.meta.md
- **Problema**: Meta não deve depender de Technical
- **Ação**: Remover referência ou tornar exemplo genérico

---

**Pode prosseguir?** ❌ NÃO (resolva conflito bloqueante)
```

---

## 🎯 Benefícios de Hierarquia Clara

✅ **Resolução de Conflitos**: Precedência sempre clara
✅ **Governança**: Meta specs controlam todas as outras
✅ **Consistência**: IA sabe qual spec seguir em caso de conflito
✅ **Manutenibilidade**: Mudanças em cascade (superior → inferior)
✅ **Rastreabilidade**: Fácil entender origem de decisões

---

## 🔄 Uso Recomendado

**Quando validar hierarquia**:

1. **Setup inicial** do projeto
2. **Após adicionar nova camada** ou reorganizar specs
3. **Quando conflitos são detectados** (via /check-drift)
4. **Periodicamente** (mensal)

**Integração com workflow**:
```markdown
# Exemplo: Validação automática de hierarquia

1. Desenvolvedor atualiza spec
2. CI executa: /validate-hierarchy
3. Se hierarquia violada: PR bloqueado
4. Se hierarquia válida: PR aprovado
```

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, valida hierarquia completa.

Se argumentos forem fornecidos (ex: `business`, `technical`), valida apenas camadas especificadas.

Flags opcionais:
- `--strict`: Trata warnings como erros
- `--fix`: Sugere correções automáticas para violações
