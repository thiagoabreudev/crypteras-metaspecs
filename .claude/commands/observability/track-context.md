# Track Context Usage - Crypteras

Este comando implementa **Context Observability** - rastreamento de qual contexto do **Crypteras Trading System** foi usado pela IA.

> **Nota**: Este é um comando avançado de observabilidade. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Context Observability

**Context Observability** é como logs/métricas/traces, mas para decisões da IA:
- Qual spec foi consultada
- Qual versão foi usada
- Quais fontes influenciaram a decisão
- Timestamp de uso

**Benefício**: Auditoria completa de decisões da IA.

---

## 📋 Objetivo

Rastrear uso de contexto para:
- Auditar decisões da IA
- Identificar specs mais usadas
- Detectar specs nunca usadas (candidatas a remoção)
- Medir impacto de atualizações de specs

---

## 🔍 O Que Rastrear

### 1. Decision Log

**Arquivo**: `specs/_meta/DECISION_LOG.md`

**Estrutura**:
```markdown
# Log de Decisões da IA

Rastreamento de decisões importantes tomadas pela IA durante desenvolvimento.

## 2025-12-20

### 14:30:00 - Decisão: Padrão de Componente

**Context**:
- Comando: /work ./.claude/sessions/WOR-123/
- Task: Implementar GoalCard component
- File: components/molecules/GoalCard.vue

**Decision**: Usar `computed` para cálculo de progresso

**Sources Used**:
- specs/technical/CLAUDE.meta.md v1.2.0 (Seção: Computed vs Watch)
- specs/technical/CODEBASE_GUIDE.md v1.0.0 (Estrutura de componentes)
- Código existente: composables/useGoalProgress.ts

**Rationale**:
1. CLAUDE.meta.md recomenda `computed` para valores derivados
2. Padrão existente no código usa `computed` consistentemente
3. Performance superior (cache automático)

**Alternatives Considered**:
1. ❌ watch - Não idiomático
2. ❌ ref manual - Propenso a bugs
3. ✅ computed - Escolhido

**Result**: ✅ Implementado com sucesso

**Approved By**: Human (via /work command)

---

### 14:35:00 - Decisão: Lógica de Negócio

**Context**:
- Task: Cálculo de status "overdue"
- File: composables/useGoalProgress.ts

**Decision**: `deadline < now && progress < 100`

**Sources Used**:
- specs/business/features/goal-management.md v1.0.0
- specs/technical/BUSINESS_LOGIC.md v1.0.0

**Rationale**:
1. Spec de negócio define claramente critério
2. Meta só está atrasada se NÃO concluída
3. Metas concluídas após prazo são "completed", não "overdue"

**Business Impact**:
- Métrica de "taxa de sucesso no prazo" fica correta
- Usuário vê claramente metas atrasadas
- Dashboard de alertas funciona adequadamente

**Result**: ✅ Implementado

**Approved By**: Human (via /work command)
```

### 2. Spec Usage Metrics

**Arquivo**: `specs/_meta/SPEC_USAGE_METRICS.md`

**Estrutura**:
```markdown
# Métricas de Uso de Specs

Rastreamento de quais specs são consultadas e com qual frequência.

## Período: 2025-12-15 a 2025-12-20

### Specs Mais Usadas

| Spec | Versão | Consultas | Última Uso |
|------|--------|-----------|------------|
| CLAUDE.meta.md | v1.2.0 | 45 | 2025-12-20 16:30 |
| CODEBASE_GUIDE.md | v1.0.0 | 32 | 2025-12-20 16:15 |
| BUSINESS_LOGIC.md | v1.0.0 | 28 | 2025-12-20 15:45 |
| goal-management.md | v1.0.0 | 15 | 2025-12-20 14:35 |
| CUSTOMER_PERSONAS.md | v1.0.0 | 12 | 2025-12-19 10:20 |

### Specs Nunca Usadas (Candidatas a Revisão)

| Spec | Versão | Última Atualização | Dias sem Uso |
|------|--------|-------------------|--------------|
| VOICE_OF_CUSTOMER.md | v1.0.0 | 2025-06-15 | 180 |
| SALES_PROCESS.md | v1.0.0 | 2025-07-01 | 160 |

💡 **Insight**: Specs não usadas há > 90 dias podem estar obsoletas ou irrelevantes.

### Specs por Categoria

**Business**: 67 consultas (35%)
**Technical**: 120 consultas (63%)
**Meta**: 5 consultas (2%)

### Tendências

📈 **Em alta**:
- CLAUDE.meta.md (+20% vs semana anterior)
- BUSINESS_LOGIC.md (+15%)

📉 **Em baixa**:
- CUSTOMER_PERSONAS.md (-10%)
- PRODUCT_STRATEGY.md (-5%)

### Correlações

**Features mais implementadas**:
1. Goal Management (35 decisões)
2. Authentication (28 decisões)
3. Dashboard (18 decisões)

**Specs usadas em conjunto** (top 3):
1. CLAUDE.meta.md + CODEBASE_GUIDE.md (30x)
2. BUSINESS_LOGIC.md + goal-management.md (25x)
3. CLAUDE.meta.md + BUSINESS_LOGIC.md (20x)
```

### 3. Version Usage Tracking

**Rastrear quais versões de specs estão sendo usadas**:

```markdown
## Uso de Versões

### CLAUDE.meta.md

| Versão | Uso | Última Consulta |
|--------|-----|-----------------|
| v1.2.0 | 45 | 2025-12-20 16:30 ✅ (atual) |
| v1.1.0 | 3  | 2025-12-18 10:15 ⚠️ (obsoleta) |
| v1.0.0 | 0  | - (deprecated) |

⚠️ **Alert**: 3 decisões usando versão obsoleta (v1.1.0)

📋 **Ação**: Verificar se código gerado com v1.1.0 precisa atualização
```

---

## 🔍 Como Implementar Tracking

### Opção 1: Manual (IA adiciona ao log)

**Durante decisão, IA deve**:

1. **Identificar specs consultadas**
2. **Capturar versões usadas**
3. **Adicionar entrada ao DECISION_LOG.md**
4. **Atualizar SPEC_USAGE_METRICS.md**

**Exemplo de código para IA**:

```markdown
## 🤖 Tracking de Decisão

Após tomar decisão usando specs, adicione ao log:

```markdown
### [TIMESTAMP] - Decisão: [TÍTULO]

**Context**: [Task, arquivo, comando]
**Decision**: [O que foi decidido]
**Sources Used**:
- [spec] v[version] (seção/linha)
- [spec] v[version]
**Rationale**: [Por quê]
**Result**: [Resultado]
```
```

### Opção 2: Automática (via metadata)

**IA pode adicionar metadata aos arquivos gerados**:

```typescript
// components/molecules/GoalCard.vue

/**
 * GoalCard Component
 *
 * @generated-by: Claude AI
 * @generated-at: 2025-12-20T14:30:00Z
 * @sources:
 *   - specs/technical/CLAUDE.meta.md v1.2.0
 *   - specs/technical/CODEBASE_GUIDE.md v1.0.0
 *   - composables/useGoalProgress.ts (existing code)
 * @decisions:
 *   - Use `computed` for progress calculation (CLAUDE.meta.md)
 *   - Follow Atomic Design structure (CODEBASE_GUIDE.md)
 */
<script setup lang="ts">
...
</script>
```

**Benefício**: Rastreamento automático de fontes no próprio código.

### Opção 3: Structured Logging

**IA pode gerar logs estruturados**:

```json
{
  "timestamp": "2025-12-20T14:30:00Z",
  "event": "decision_made",
  "context": {
    "command": "/work",
    "task_id": "WOR-123",
    "file": "components/molecules/GoalCard.vue"
  },
  "decision": {
    "title": "Use computed for progress",
    "choice": "computed",
    "alternatives": ["watch", "ref"]
  },
  "sources": [
    {
      "spec": "specs/technical/CLAUDE.meta.md",
      "version": "1.2.0",
      "section": "Computed vs Watch"
    },
    {
      "spec": "specs/technical/CODEBASE_GUIDE.md",
      "version": "1.0.0",
      "section": "Component Structure"
    }
  ],
  "rationale": "CLAUDE.meta.md recommends computed for derived values",
  "result": "success",
  "approved_by": "human"
}
```

**Benefício**: Fácil análise com ferramentas (jq, SQL, etc).

---

## 📊 Análise de Métricas

### Detectar Specs Subutilizadas

```markdown
## 📉 Specs com Baixo Uso

**Critério**: < 5 consultas em 30 dias

| Spec | Consultas (30d) | Status |
|------|-----------------|--------|
| VOICE_OF_CUSTOMER.md | 0 | 🔴 Nunca usado |
| SALES_PROCESS.md | 1 | 🟡 Pouco usado |
| INDUSTRY_TRENDS.md | 3 | 🟡 Pouco usado |

💡 **Recomendação**:
- Avaliar se specs ainda são relevantes
- Considerar deprecar ou mesclar com outras specs
- Atualizar para tornar mais útil
```

### Detectar Specs Críticas

```markdown
## 🔥 Specs Mais Críticas

**Critério**: > 20 consultas em 7 dias

| Spec | Consultas (7d) | Impacto |
|------|----------------|---------|
| CLAUDE.meta.md | 45 | 🔴 Crítico |
| CODEBASE_GUIDE.md | 32 | 🔴 Crítico |
| BUSINESS_LOGIC.md | 28 | 🟡 Alto |

⚠️ **Atenção**: Atualizações nestas specs têm alto impacto
✅ **Benefício**: Foco em qualidade das specs mais usadas
```

### Detectar Context Drift via Uso

```markdown
## 🔍 Drift Detectado via Padrão de Uso

**Anomalia**: CUSTOMER_PERSONAS.md não consultado em 45 dias

**Contexto**: Feature de personalização sendo implementada

🔴 **Alert**: Feature deveria consultar personas mas não está

📋 **Ação**: Verificar se feature está alinhada com personas
```

---

## ✅ Checklist de Tracking

**Setup inicial**:
- [ ] Criar `specs/_meta/DECISION_LOG.md`
- [ ] Criar `specs/_meta/SPEC_USAGE_METRICS.md`
- [ ] Definir formato de log (manual/metadata/structured)

**Durante desenvolvimento**:
- [ ] IA registra decisões no DECISION_LOG.md
- [ ] IA atualiza contadores de uso
- [ ] IA adiciona metadata a código gerado (opcional)

**Análise periódica** (semanal):
- [ ] Revisar specs mais/menos usadas
- [ ] Identificar anomalias (drift, subutilização)
- [ ] Atualizar métricas agregadas

---

## 🎯 Benefícios

✅ **Auditabilidade**: Histórico completo de decisões
✅ **Insights**: Entender impacto real das specs
✅ **Drift Detection**: Identificar specs obsoletas
✅ **Priorização**: Focar qualidade onde importa
✅ **Aprendizado**: Analisar padrões de uso

---

## 📚 Exemplo de Uso Completo

**Cenário**: Desenvolvedor implementa feature de autenticação

**Tracking automático**:

1. **IA consulta specs**:
   - CLAUDE.meta.md v1.2.0
   - features/authentication.md v1.0.0
   - adr/005-jwt-auth.md v1.0.0

2. **IA registra decisão**:
   ```markdown
   ### 14:45:00 - Decisão: Estratégia de Autenticação
   **Decision**: JWT com bcrypt (10 rounds)
   **Sources**: authentication.md v1.0.0, adr/005 v1.0.0
   **Rationale**: Conforme ADR e spec de feature
   ```

3. **IA atualiza métricas**:
   ```markdown
   | authentication.md | v1.0.0 | 16 (+1) | 2025-12-20 14:45 |
   | adr/005-jwt-auth.md | v1.0.0 | 9 (+1) | 2025-12-20 14:45 |
   ```

4. **IA adiciona metadata ao código**:
   ```typescript
   /**
    * @sources:
    *   - specs/business/features/authentication.md v1.0.0
    *   - specs/technical/adr/005-jwt-auth.md v1.0.0
    */
   ```

**Resultado**: Decisão 100% rastreável e auditável.

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, inicializa tracking (cria arquivos de log).

Se argumentos forem fornecidos:
- `init`: Cria estrutura de tracking
- `log <decision>`: Adiciona decisão manual ao log
- `metrics`: Gera relatório de métricas de uso
- `analyze`: Análise de padrões e anomalias

Flags opcionais:
- `--format=json`: Output em JSON para análise programática
- `--period=30d`: Período de análise (padrão: 7 dias)
