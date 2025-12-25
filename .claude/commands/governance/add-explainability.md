# Adicionar Explainability Requirements às Metaspecs - Crypteras

Este comando adiciona requisitos de **Explainability** (explicabilidade) às especificações do **Crypteras Trading System**, garantindo que IA em produção possa explicar suas decisões.

> **Nota**: Este é um comando avançado de governança de contexto. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Explainability by Design

**Explainability** é a capacidade da IA de explicar suas decisões de forma compreensível para humanos.

**Problema sem explainability**:
- Decisões da IA são "caixas-pretas"
- Impossível auditar escolhas
- Difícil debugar quando IA erra
- Falta de confiança no sistema

**Solução**: Explainability como requisito obrigatório em specs críticas.

---

## 📋 Objetivo

Adicionar requisitos de explainability para:
- IA deve explicar decisões importantes
- Decisões são auditáveis
- Rastreamento de fontes (qual spec foi usada)
- Transparência em produção

---

## 🔍 Processo

### 1. Identificar Specs que Precisam de Explainability

**Alta prioridade** (decisões críticas):
- `CLAUDE.meta.md` - Decisões de desenvolvimento
- `API_SPECIFICATION.md` - Decisões de integração
- `BUSINESS_LOGIC.md` - Decisões de negócio
- ADRs - Decisões arquiteturais

**Média prioridade**:
- Features com lógica complexa
- Specs com múltiplas alternativas válidas

### 2. Template de Explainability

Adicione seção `:::explainability` após `:::failure_modes`:

```markdown
:::explainability
**Requirement**: ✅ Required | ⚠️ Recommended | ⭕ Optional

**Output Format**:
- **Decision**: [O que foi decidido]
- **Source**: [Qual spec/seção foi consultada]
- **Rationale**: [Por que esta decisão]
- **Alternatives Considered**: [Outras opções avaliadas]
- **Trade-offs**: [Prós e contras]

**Audit Trail**:
- Timestamp de decisão
- Specs consultadas (nome + versão)
- Contexto relevante usado
:::
```

### 3. Exemplo: CLAUDE.meta.md

```markdown
:::explainability
**Requirement**: ✅ Required (para decisões arquiteturais)

**Output Format**:
IA DEVE explicar decisões seguindo este formato:

```markdown
## 🤖 Decisão de Desenvolvimento

**Decisão**: Usar `computed` ao invés de `watch` para cálculo de progresso

**Source**:
- `specs/technical/CLAUDE.meta.md` v1.1.0 - Seção "Computed vs Watch"
- Código existente: `composables/useGoalProgress.ts:15-20`

**Rationale**:
1. `computed` é padrão correto para valores derivados (conforme spec)
2. Código existente usa este padrão consistentemente
3. Performance superior (cache automático)
4. Menos propenso a bugs de sincronização

**Alternatives Considered**:
1. ❌ `watch` - Não idiomático, não recomendado pela spec
2. ❌ `ref` manual - Requer sincronização manual, propenso a bugs
3. ✅ `computed` - Escolhido (idiomático, performático, seguro)

**Trade-offs**:
- ✅ Pro: Cache automático
- ✅ Pro: Código limpo
- ✅ Pro: Consistente com codebase
- ⚠️ Con: Nenhum significativo

**Audit Trail**:
- Timestamp: 2025-12-20T14:30:00Z
- Specs Consultadas: CLAUDE.meta.md v1.1.0, CODEBASE_GUIDE.md v1.0.0
- Código Analisado: composables/useGoalProgress.ts, components/molecules/GoalCard.vue
```
```

**Quando Explicar** (gatilhos obrigatórios):
1. Escolha de padrão arquitetural
2. Decisão entre múltiplas alternativas válidas
3. Trade-off significativo identificado
4. Desvio de padrão existente (requer justificativa forte)
5. Introdução de nova dependência
6. Mudança de assinatura de API pública

**Audit Trail Obrigatório**:
- Timestamp da decisão
- Specs consultadas (nome + versão)
- Arquivos de código analisados (com line numbers)
- Contexto usado (ex: ADR-003, CUSTOMER_PERSONAS v1.0.0)
:::
```

### 4. Exemplo: BUSINESS_LOGIC.md

```markdown
:::explainability
**Requirement**: ✅ Required (para regras de negócio)

**Output Format**:

```markdown
## 🤖 Decisão de Lógica de Negócio

**Decisão**: Status "overdue" aplicado quando `deadline < now && progress < 100`

**Source**:
- `specs/business/features/goal-management.md` v1.0.0 - Seção "Status Calculation"
- `specs/technical/BUSINESS_LOGIC.md` v1.0.0 - Seção "Goal Status Rules"

**Rationale**:
1. Spec de negócio define claramente critério para "atrasada"
2. Meta só está atrasada se NÃO foi concluída (progress < 100)
3. Metas concluídas após prazo não são "overdue", são "completed"

**Alternatives Considered**:
1. ❌ `deadline < now` (sem check de progress) - Marca metas concluídas como atrasadas
2. ❌ `deadline < now && status !== 'completed'` - Depende de status manual, propenso a bugs
3. ✅ `deadline < now && progress < 100` - Escolhido (automático, baseado em progresso)

**Trade-offs**:
- ✅ Pro: Automático (não requer atualização manual)
- ✅ Pro: Consistente com specs de negócio
- ⚠️ Con: Meta pode ficar "overdue" brevemente antes de ser marcada "completed"

**Business Impact**:
- Usuário vê claramente metas atrasadas
- Métrica de "taxa de sucesso no prazo" fica correta
- Dashboard de alertas funciona adequadamente

**Audit Trail**:
- Timestamp: 2025-12-20T14:35:00Z
- Specs Consultadas: goal-management.md v1.0.0, BUSINESS_LOGIC.md v1.0.0
- Business Rules Aplicadas: BR-003 (Goal Status Calculation)
```
```

**Quando Explicar**:
1. Implementação de regra de negócio complexa
2. Cálculo envolvendo múltiplas variáveis
3. Validação que pode rejeitar entrada válida
4. Decisão que impacta métricas de negócio
:::
```

### 5. Exemplo: ADR (Architecture Decision Record)

```markdown
# ADR-003: NestJS Backend Framework

:::explainability
**Requirement**: ✅ Required (decisão arquitetural crítica)

**Output Format**:
ADRs JÁ são explicações de decisões. Esta spec documenta que:

1. **Toda decisão arquitetural DEVE ter ADR**
2. **ADR DEVE seguir formato padrão**:
   - Context (contexto que levou à decisão)
   - Decision (o que foi decidido)
   - Rationale (por que decidimos isso)
   - Consequences (impactos positivos e negativos)
   - Alternatives Considered (outras opções avaliadas)

3. **ADR DEVE ser referenciado** quando IA implementar código relacionado

**Audit Trail Obrigatório** (ao criar ADR):
- Data da decisão
- Stakeholders envolvidos
- Specs consultadas
- Referências externas (docs, artigos, benchmarks)
- Versão inicial: 1.0.0

**Quando Criar Novo ADR**:
1. Escolha de framework/biblioteca principal
2. Padrão arquitetural (ex: Atomic Design, DDD, CQRS)
3. Estratégia de dados (ex: MongoDB vs PostgreSQL)
4. Estratégia de autenticação/autorização
5. Decisão que afeta múltiplos módulos
6. Trade-off significativo com consequências duradouras
:::
```

### 6. Exemplo: Feature Spec (Goal Management)

```markdown
:::explainability
**Requirement**: ⚠️ Recommended (para decisões de produto)

**Output Format**:

```markdown
## 🤖 Decisão de Feature

**Decisão**: Progress calculado como `(actions completed / total actions) * 100`

**Source**:
- `specs/business/features/goal-management.md` v1.0.0 - Seção "Progress Calculation"
- `specs/business/CUSTOMER_PERSONAS.md` v1.0.0 - Ana (estudante) quer clareza visual

**Rationale**:
1. Simples e intuitivo (persona Ana valoriza simplicidade)
2. Automático (não requer input manual)
3. Mensurável (0-100%)
4. Visual (fácil representar em barra de progresso)

**Alternatives Considered**:
1. ❌ Progress manual (usuário informa %) - Propenso a erro, não automático
2. ❌ Ponderado por prioridade de ação - Muito complexo para MVP
3. ✅ Simples count de ações - Escolhido (simples, automático, claro)

**User Impact**:
- Ana (estudante) vê progresso claro e visual
- Motivação aumenta ao ver barra preenchendo
- Métrica de "metas em progresso" funciona corretamente

**Audit Trail**:
- Timestamp: 2025-12-20T14:40:00Z
- Specs Consultadas: goal-management.md v1.0.0, CUSTOMER_PERSONAS.md v1.0.0
- Personas Consideradas: Ana (primary), Carlos (secondary)
```
```

**Quando Explicar**:
1. Escolha de UX que impacta persona principal
2. Decisão de cálculo que afeta métricas mostradas
3. Feature que pode ser implementada de múltiplas formas
:::
```

---

## ✅ Checklist de Execução

**Para specs técnicas críticas**:
- [ ] `CLAUDE.meta.md` - Explainability ✅ Required
- [ ] `API_SPECIFICATION.md` - Explainability ✅ Required
- [ ] `BUSINESS_LOGIC.md` - Explainability ✅ Required
- [ ] Cada ADR - Explainability ✅ Required

**Para specs de produto**:
- [ ] `PRODUCT_STRATEGY.md` - Explainability ⚠️ Recommended
- [ ] Features críticas - Explainability ⚠️ Recommended

**Para cada explainability requirement adicionado**:
- [ ] Requirement level definido (Required/Recommended/Optional)
- [ ] Output format especificado
- [ ] Gatilhos de "quando explicar" documentados
- [ ] Audit trail obrigatório definido
- [ ] Incrementar versão MINOR da spec
- [ ] Documentar mudança em `:::breaking_changes`

---

## 📊 Níveis de Explainability

### ✅ Required (Obrigatório)
Decisões críticas que DEVEM ser explicadas:
- Decisões arquiteturais
- Regras de negócio complexas
- Escolhas de segurança
- Trade-offs com consequências duradouras

**IA DEVE bloquear se não conseguir explicar.**

### ⚠️ Recommended (Recomendado)
Decisões importantes que DEVERIAM ser explicadas:
- Escolhas de UX/produto
- Implementações com múltiplas alternativas
- Otimizações de performance
- Refatorações significativas

**IA DEVE tentar explicar, mas pode prosseguir se não conseguir.**

### ⭕ Optional (Opcional)
Decisões rotineiras:
- Nomes de variáveis
- Formatação de código
- Ordem de imports
- Decisões triviais

**IA pode pular explicação.**

---

## 🚨 Como IA Deve Gerar Explicações

**Passo 1: Detectar gatilho**
```
Gatilho detectado: Escolha de padrão arquitetural
Spec: CLAUDE.meta.md
Requirement: ✅ Required
```

**Passo 2: Coletar contexto**
```
Consultando specs relevantes...
- CLAUDE.meta.md v1.1.0
- CODEBASE_GUIDE.md v1.0.0
- ADR-001 v1.0.0

Analisando código existente...
- composables/useGoalProgress.ts
- components/molecules/GoalCard.vue
```

**Passo 3: Avaliar alternativas**
```
Alternativas identificadas:
1. Computed
2. Watch
3. Ref manual

Avaliando contra specs...
```

**Passo 4: Gerar explicação**
```markdown
## 🤖 Decisão de Desenvolvimento

**Decisão**: ...
**Source**: ...
**Rationale**: ...
...
```

**Passo 5: Apresentar ao humano**
```
Apresentando explicação para aprovação...

✅ Explicação gerada conforme CLAUDE.meta.md v1.1.0
📋 Audit trail completo
🔍 Pronto para revisão
```

---

## 🔄 Auditoria de Decisões

Crie arquivo `specs/_meta/DECISION_LOG.md` para rastrear decisões:

```markdown
# Log de Decisões da IA

Este arquivo rastreia decisões importantes tomadas pela IA durante desenvolvimento.

## 2025-12-20

### Decisão: Usar Computed para Progress Calculation
- **Timestamp**: 2025-12-20T14:30:00Z
- **Spec**: CLAUDE.meta.md v1.1.0
- **Context**: Implementação de useGoalProgress.ts
- **Decision**: Computed (ao invés de watch)
- **Rationale**: Padrão idiomático, performance, consistência
- **Approved By**: Human (via /work command)
- **Audit Trail**: [Link para explicação completa]

### Decisão: Status "overdue" Logic
- **Timestamp**: 2025-12-20T14:35:00Z
- **Spec**: BUSINESS_LOGIC.md v1.0.0
- **Context**: Goal status calculation
- **Decision**: `deadline < now && progress < 100`
- **Rationale**: Automático, baseado em progresso, consistente com specs
- **Approved By**: Human (via /work command)
- **Audit Trail**: [Link para explicação completa]
```

---

## 🎯 Benefícios de Explainability

✅ **Transparência**: Decisões não são caixas-pretas
✅ **Auditabilidade**: Possível rastrear por que decisão foi tomada
✅ **Debugging**: Fácil identificar decisões incorretas
✅ **Aprendizado**: Time aprende com decisões da IA
✅ **Confiança**: Humanos confiam mais em IA explicável
✅ **Compliance**: Requisito em alguns domínios (healthcare, finance)

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, adicione Explainability Requirements a TODAS as specs críticas.

Se argumentos forem fornecidos (ex: `technical` ou spec específica), adicione apenas às specs especificadas.
