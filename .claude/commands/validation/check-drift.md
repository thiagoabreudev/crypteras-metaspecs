# Detectar Context Drift - Crypteras

Este comando detecta **Context Drift** - quando especificações do **Crypteras Trading System** ficaram desatualizadas em relação ao código/realidade.

> **Nota**: Este é um comando avançado de validação. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Context Drift

**Context Drift** ocorre quando:
- Specs documentam funcionalidade que não existe mais
- Código implementa feature não documentada
- Decisões arquiteturais não refletem código real
- Termos mudaram de significado sem atualização

**Problema**: IA toma decisões baseadas em contexto obsoleto.

**Solução**: Detectar drift automaticamente e alertar.

---

## 📋 Objetivo

Detectar drift entre specs e realidade para:
- Identificar specs desatualizadas
- Encontrar código não documentado
- Alertar sobre context drift crítico
- Sugerir atualizações necessárias

---

## 🔍 Tipos de Drift

### 1. Documentation Drift
Specs documentam algo que não existe (mais).

**Exemplo**:
```markdown
Spec diz: "Usamos Options API"
Código tem: 100% Composition API

🔴 DRIFT CRÍTICO: Spec desatualizada
```

### 2. Implementation Drift
Código implementa algo não documentado.

**Exemplo**:
```markdown
Spec não menciona: OAuth
Código tem: GoogleOAuthStrategy.ts

🟡 DRIFT MÉDIO: Feature não documentada
```

### 3. Semantic Drift
Termo muda de significado sem atualização.

**Exemplo**:
```markdown
Spec v1.0.0: "limite" = limite contratual
Código usa: "limite" = pré-aprovado por CPF

🔴 DRIFT CRÍTICO: Significado mudou sem versionamento
```

### 4. Architectural Drift
Decisões arquiteturais não refletem código.

**Exemplo**:
```markdown
ADR-002: "Usar MongoDB"
Código tem: PostgreSQL connection

🔴 DRIFT CRÍTICO: Stack diferente da documentada
```

---

## 🔍 Processo de Detecção

### 1. Validar Stack Tecnológica

**Verificar que stack em specs/technical/index.md corresponde ao código**:

```markdown
## Validação de Stack

**Spec diz** (specs/technical/index.md v1.0.0):
- Frontend: Vue 3
- Backend: NestJS
- Database: MongoDB
- Auth: JWT

**Código tem**:
- package.json dependencies:
  - vue: ^3.4.0 ✅
  - @nestjs/core: ^10.0.0 ✅
  - mongoose: ^8.0.0 ✅
  - @nestjs/jwt: ^10.0.0 ✅

**Resultado**: ✅ Stack alinhada (sem drift)
```

**Drift detectado**:
```markdown
## 🔴 Stack Drift Detectado

**Spec diz**: Database: MongoDB
**Código tem**:
- package.json: "pg": "^8.11.0"
- src/app.module.ts: TypeOrmModule.forRoot({ type: 'postgres' })

🔴 DRIFT CRÍTICO: Spec menciona MongoDB mas código usa PostgreSQL

📋 Ação Necessária:
1. Atualizar specs/technical/index.md v1.0.0 → v2.0.0 (breaking change)
2. OU migrar código para MongoDB (se spec está correta)
```

### 2. Validar Features Documentadas

**Verificar que features em specs/business/features/ existem no código**:

```markdown
## Validação de Features

**Para cada feature em specs/business/features/**:

### Feature: Authentication (features/authentication.md v1.0.0)
- [ ] Spec menciona: Login com email/senha
- [ ] Código tem: `auth/auth.controller.ts` com POST /auth/login ✅
- [ ] Spec menciona: Registro de usuário
- [ ] Código tem: POST /auth/register ✅
- [ ] Spec menciona: JWT tokens
- [ ] Código tem: JwtStrategy implementado ✅

**Resultado**: ✅ Feature implementada conforme spec

---

### Feature: Goal Management (features/goal-management.md v1.0.0)
- [ ] Spec menciona: Criar meta
- [ ] Código tem: POST /goals ✅
- [ ] Spec menciona: Editar meta
- [ ] Código tem: PATCH /goals/:id ✅
- [ ] Spec menciona: Deletar meta
- [ ] Código tem: DELETE /goals/:id ❌ **FALTA**

**Resultado**: ⚠️ Feature parcialmente implementada
```

**Drift detectado**:
```markdown
## ⚠️ Feature Drift Detectado

**Feature**: Goal Management (features/goal-management.md v1.0.0)

**Spec menciona**: DELETE /goals/:id
**Código**: Endpoint não encontrado

🟡 DRIFT MÉDIO: Feature documentada mas não implementada

📋 Ação Necessária:
1. Implementar endpoint DELETE (se feature é válida)
2. OU remover da spec (se feature foi descontinuada)
3. OU marcar como "Planejada" na spec (se é futuro)
```

### 3. Validar ADRs vs Código

**Verificar que decisões arquiteturais refletem código**:

```markdown
## Validação de ADRs

### ADR-001: Vue 3 Composition API
**Decisão**: Usar 100% Composition API com `<script setup>`

**Verificação no código**:
- Buscar `export default {` (Options API) → 0 ocorrências ✅
- Buscar `<script setup>` → 45 ocorrências ✅
- Buscar `defineComponent` (Options API) → 0 ocorrências ✅

**Resultado**: ✅ ADR seguido consistentemente

---

### ADR-003: NestJS Backend
**Decisão**: Usar NestJS para backend

**Verificação no código**:
- package.json tem @nestjs/core? ✅
- src/main.ts usa NestFactory? ✅
- Controllers usam decorators NestJS? ✅

**Resultado**: ✅ ADR seguido consistentemente

---

### ADR-007: Pinia State Management
**Decisão**: Usar Pinia para estado global

**Verificação no código**:
- package.json tem pinia? ❌ **NÃO TEM**
- src/stores/ existe? ❌ **NÃO EXISTE**
- main.ts configura Pinia? ❌ **NÃO CONFIGURA**

**Resultado**: 🔴 ADR NÃO seguido (drift crítico)
```

**Drift detectado**:
```markdown
## 🔴 ADR Drift Crítico

**ADR**: 007-pinia-state-management.md v1.0.0

**Decisão**: "Usar Pinia para state management"

**Código**: Pinia não está instalado ou configurado

🔴 DRIFT CRÍTICO: ADR documenta decisão não implementada

📋 Ação Necessária:
1. Implementar Pinia (se ADR é válido)
2. OU deprecar ADR-007 (se decisão foi revertida)
3. OU criar ADR-008 explicando mudança de decisão
```

### 4. Validar Componentes Documentados

**Verificar que componentes em CODEBASE_GUIDE.md existem**:

```markdown
## Validação de Componentes

**Spec documenta** (CODEBASE_GUIDE.md v1.0.0):
```
src/
  components/
    atoms/
      ButtonPrimary.vue
      InputText.vue
      Badge.vue
    molecules/
      GoalCard.vue
      ActionItem.vue
```

**Código tem**:
- components/atoms/ButtonPrimary.vue ✅
- components/atoms/InputText.vue ✅
- components/atoms/Badge.vue ❌ **FALTA**
- components/molecules/GoalCard.vue ✅
- components/molecules/ActionItem.vue ✅

**Resultado**: ⚠️ 1 componente documentado não existe
```

**Drift detectado**:
```markdown
## ⚠️ Component Drift

**Spec documenta**: components/atoms/Badge.vue

**Código**: Arquivo não existe

🟡 DRIFT MÉDIO: Componente documentado mas não criado

📋 Ação Necessária:
1. Criar Badge.vue (se componente é necessário)
2. OU remover da doc (se componente foi descartado)
```

### 5. Validar Termos de Negócio

**Verificar que termos em BUSINESS_LOGIC.md são usados consistentemente**:

```markdown
## Validação de Termos

**Spec define** (BUSINESS_LOGIC.md v1.0.0):
- "Meta": Objetivo pessoal do usuário
- "Ação": Pequeno passo para completar meta
- "Revisão": Reflexão periódica sobre progresso

**Código usa**:
- Buscar "Goal" (inglês de Meta) → 127 ocorrências ✅
- Buscar "Action" → 45 ocorrências ✅
- Buscar "Review" → 12 ocorrências ✅
- Buscar "Task" (não documentado) → 0 ocorrências ✅
- Buscar "Item" (não documentado) → 3 ocorrências ⚠️

**Resultado**: ⚠️ Termo "Item" usado mas não definido na spec
```

**Drift detectado**:
```markdown
## ⚠️ Terminology Drift

**Termo não documentado**: "Item"

**Uso no código**:
- components/ActionItem.vue (nome de componente)
- types/Item.ts (tipo não documentado)

🟡 DRIFT MÉDIO: Termo usado mas não definido

📋 Ação Necessária:
1. Adicionar definição de "Item" em BUSINESS_LOGIC.md
2. OU renomear para "Action" (termo documentado)
```

### 6. Validar Atualização Recente

**Verificar que specs não estão muito desatualizadas**:

```markdown
## Validação de Atualização

**Idade das specs**:

### Specs de Negócio
- CUSTOMER_PERSONAS.md: Atualizado há 30 dias ✅
- PRODUCT_STRATEGY.md: Atualizado há 15 dias ✅
- VOICE_OF_CUSTOMER.md: Atualizado há 180 dias ⚠️ **Antigo**

### Specs Técnicas
- CLAUDE.meta.md: Atualizado há 5 dias ✅
- CODEBASE_GUIDE.md: Atualizado há 90 dias ⚠️ **Antigo**
- API_SPECIFICATION.md: Atualizado há 200 dias 🔴 **Muito antigo**

**Threshold**:
- < 60 dias: ✅ Recente
- 60-120 dias: ⚠️ Atenção
- > 120 dias: 🔴 Requer revisão
```

**Drift suspeito**:
```markdown
## 🔴 Specs Muito Antigas

**Specs não atualizadas há > 120 dias**:
1. VOICE_OF_CUSTOMER.md (180 dias)
2. API_SPECIFICATION.md (200 dias)

🔴 DRIFT SUSPEITO: Specs antigas podem estar desatualizadas

📋 Ação Recomendada:
Revisar specs antigas e atualizar ou confirmar que estão corretas
```

---

## ✅ Checklist de Detecção

Executar TODAS as verificações:

- [ ] **Stack Tecnológica**: Specs vs package.json
- [ ] **Features**: Specs vs endpoints/componentes
- [ ] **ADRs**: Decisões vs código real
- [ ] **Componentes**: Documentação vs arquivos
- [ ] **Termos**: Definições vs uso no código
- [ ] **Atualização**: Idade das specs

---

## 📊 Formato de Saída

```markdown
# 🔍 Detecção de Context Drift

**Data**: 2025-12-20T15:30:00Z
**Diretório**: /Users/user/project/

---

## 1. ✅ Stack Tecnológica
Sem drift detectado.

## 2. ⚠️ Features
1 feature parcialmente implementada.

## 3. 🔴 ADRs
1 ADR não seguido (Pinia).

## 4. ⚠️ Componentes
1 componente documentado não existe (Badge).

## 5. ⚠️ Termos
1 termo usado mas não definido (Item).

## 6. 🔴 Atualização
2 specs muito antigas (> 120 dias).

---

## 📋 Resumo de Drift

| Categoria | Drift Detectado | Severidade |
|-----------|-----------------|------------|
| Stack | 0 | - |
| Features | 1 | 🟡 Médio |
| ADRs | 1 | 🔴 Crítico |
| Componentes | 1 | 🟡 Médio |
| Termos | 1 | 🟡 Médio |
| Atualização | 2 | 🔴 Crítico |

**Drift Crítico**: 2
**Drift Médio**: 3
**Total**: 5 drifts detectados

---

## 🛑 Drifts Críticos (Ação Imediata)

### 1. ADR-007 Não Seguido
- **Problema**: Pinia documentado mas não implementado
- **Impacto**: IA pode gerar código esperando Pinia
- **Ação**: Implementar ou deprecar ADR

### 2. Specs Antigas
- **Problema**: 2 specs não atualizadas há > 120 dias
- **Impacto**: Informação pode estar obsoleta
- **Ação**: Revisar e atualizar specs

---

## ⚠️ Drifts Médios (Ação Recomendada)

### 1. Feature Parcialmente Implementada
- **Solução**: Implementar DELETE /goals/:id

### 2. Componente Badge Faltando
- **Solução**: Criar Badge.vue ou remover da doc

### 3. Termo "Item" Não Definido
- **Solução**: Adicionar definição ou renomear

---

**Requer atenção imediata?** ✅ SIM (2 críticos)
```

---

## 🔄 Automação

**Integrar com CI/CD**:

```yaml
# .github/workflows/check-drift.yml
name: Check Context Drift
on: [push, pull_request]

jobs:
  drift-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check Context Drift
        run: claude /check-drift
      - name: Fail if Critical Drift
        if: steps.drift.outputs.critical > 0
        run: exit 1
```

**Executar periodicamente**:
- Diariamente: Check drift automatizado
- Semanalmente: Revisão manual de drifts médios
- Mensalmente: Atualização de specs antigas

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, verifica drift em TODOS os aspectos.

Se argumentos forem fornecidos (ex: `stack`, `features`, `adrs`), verifica apenas categorias especificadas.

Flags opcionais:
- `--critical-only`: Mostra apenas drifts críticos
- `--with-suggestions`: Inclui sugestões de correção
- `--age-threshold=90`: Define threshold de dias para specs antigas (padrão: 120)
