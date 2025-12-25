# Adicionar Intent as Code às Metaspecs - Crypteras

Este comando adiciona **Intent as Code** às especificações do **Crypteras Trading System**, formalizando a intenção humana em estruturas verificáveis que guiam decisões da IA.

> **Nota**: Este é um comando avançado de governança de contexto. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Intent as Code

**Intent as Code** é a formalização da intenção humana em estruturas verificáveis que guiam decisões da IA.

**Problema sem intenção explícita**:
- IA otimiza o que não deveria
- Escopo se expande silenciosamente (scope creep)
- Soluções corretas mas não intencionadas

**Solução**: Intent explícito e versionado com goals, constraints e non-goals.

---

## 📋 Objetivo

Adicionar seções de Intent às specs existentes para:
- Definir limites claros de autonomia da IA
- Documentar o que NÃO deve ser feito
- Criar guardrails para decisões
- Prevenir scope creep

---

## 🔍 Processo

### 1. Identificar Specs que Precisam de Intent

**Specs Técnicas** (alta prioridade):
- `CLAUDE.meta.md` - Intent de desenvolvimento
- `CODEBASE_GUIDE.md` - Intent de arquitetura
- `API_SPECIFICATION.md` - Intent de API design
- ADRs individuais - Intent de decisões específicas

**Specs de Negócio** (média prioridade):
- `PRODUCT_STRATEGY.md` - Intent estratégico
- `features/*.md` - Intent por feature

### 2. Template de Intent

Adicione seção `:::intent` logo após `:::version_info`:

```markdown
:::intent
**Goal**: [O que queremos alcançar]

**Constraints** (limites obrigatórios):
- [Constraint 1]
- [Constraint 2]
- [Constraint 3]

**Non-Goals** (o que NÃO fazer):
- [Non-goal 1]
- [Non-goal 2]
- [Non-goal 3]
:::
```

### 3. Exemplo: CLAUDE.meta.md (Guia de Desenvolvimento)

```markdown
---
spec_version: "1.1.0"
valid_from: "2025-12-20"
last_updated: "2025-12-20"
supersedes: "1.0.0"
status: "active"
---

# Guia de Desenvolvimento com IA - MetaCerta

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-20
**Status**: Ativa
:::

:::intent
**Goal**: Guiar desenvolvimento mantendo consistência, qualidade e alinhamento com arquitetura do projeto.

**Constraints** (limites obrigatórios):
- Manter retrocompatibilidade com código existente
- Não alterar contratos públicos (APIs, props de componentes exportados)
- Respeitar todos os ADRs vigentes
- Usar APENAS stack tecnológica aprovada (Vue 3, NestJS, MongoDB)
- Seguir Atomic Design para estrutura de componentes
- TypeScript strict mode obrigatório

**Non-Goals** (o que NÃO fazer):
- Refatoração ampla de código não relacionado
- Mudanças de arquitetura sem ADR aprovado
- Introdução de novas tecnologias/frameworks
- Otimização prematura (performance sem evidência de problema)
- Código "clever" em detrimento de clareza
- Abstrações genéricas sem caso de uso concreto
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code

**v1.0.0** (baseline):
- Primeira versão versionada
:::

# Visão Geral Rápida
...
```

### 4. Exemplo: PRODUCT_STRATEGY.md

```markdown
:::intent
**Goal**: Criar ferramenta educacional funcional que demonstra desenvolvimento orientado por IA.

**Constraints** (limites obrigatórios):
- Manter simplicidade (MVP apenas)
- Foco em valor pedagógico sobre escalabilidade
- Prazo máximo: 6 sprints
- Budget: Zero (infraestrutura gratuita)
- Sem funcionalidades comerciais (paywall, monetização)

**Non-Goals** (o que NÃO fazer):
- Sistema de produção escalável
- Gamificação avançada (badges, leaderboards)
- Metas compartilhadas/colaborativas
- Integrações com ferramentas externas (Google Calendar, Todoist)
- Mobile apps nativos
- Features de analytics avançadas
:::
```

### 5. Exemplo: ADR (Architecture Decision Record)

```markdown
# ADR-001: Vue 3 Composition API

:::intent
**Goal**: Definir padrão de desenvolvimento de componentes Vue consistente e moderno.

**Constraints**:
- 100% Composition API (`<script setup>`)
- TypeScript obrigatório
- Props e Emits tipados
- Sem Options API

**Non-Goals**:
- Suporte a Vue 2
- Migração automática de Options API para Composition API
- Mixins (deprecated)
:::

## Contexto
...
```

### 6. Exemplo: Feature Spec

```markdown
# Feature: Autenticação de Usuário

:::intent
**Goal**: Implementar autenticação segura e simples para MVP.

**Constraints**:
- Usar JWT (conforme ADR-005)
- Sessão via localStorage (apenas para demo)
- Email + senha (sem OAuth no MVP)
- Password hash com bcrypt (min 10 rounds)

**Non-Goals**:
- OAuth/Social login (Google, GitHub)
- Two-factor authentication (2FA)
- "Lembrar-me" com cookies persistentes
- Recuperação de senha (MVP usa email manual)
- Rate limiting avançado
:::
```

---

## ✅ Checklist de Execução

**Para specs técnicas**:
- [ ] `CLAUDE.meta.md` - Intent de desenvolvimento
- [ ] `CODEBASE_GUIDE.md` - Intent de arquitetura
- [ ] `API_SPECIFICATION.md` - Intent de API design
- [ ] Cada ADR - Intent da decisão específica

**Para specs de negócio**:
- [ ] `PRODUCT_STRATEGY.md` - Intent estratégico do produto
- [ ] Cada feature em `features/*.md` - Intent da feature

**Para cada intent adicionado**:
- [ ] Goal claro e mensurável
- [ ] Constraints específicos e validáveis
- [ ] Non-goals explícitos (prevenir scope creep)
- [ ] Incrementar versão MINOR da spec (ex: 1.0.0 → 1.1.0)
- [ ] Documentar mudança em `:::breaking_changes`

---

## 📊 Validação de Qualidade do Intent

**Intent bem escrito**:
✅ Goal específico e mensurável
✅ Constraints verificáveis (pode testar se foram seguidos)
✅ Non-goals claros (previnem mal-entendidos)
✅ Alinhado com estratégia do projeto

**Intent mal escrito**:
❌ Goal vago ("melhorar qualidade")
❌ Constraints não verificáveis ("código deve ser bom")
❌ Non-goals ausentes ou vagos
❌ Contradições com outros intents

---

## 🎯 Como IA Deve Usar Intent

**Durante desenvolvimento**:

1. **Ler intent da spec relevante**:
   ```markdown
   Lendo CLAUDE.meta.md...
   Intent: "Não alterar contratos públicos"
   ```

2. **Validar decisão contra intent**:
   ```markdown
   Decisão: Adicionar prop `color` ao ButtonPrimary
   Validação: ✅ Não altera contrato existente (nova prop opcional)
   ```

3. **Bloquear violação de constraint**:
   ```markdown
   Decisão: Substituir Vue 3 por React
   Validação: ❌ BLOQUEADO - Viola constraint "Usar APENAS stack aprovada"
   ```

4. **Prevenir non-goal**:
   ```markdown
   Solicitação: "Adicione OAuth com Google"
   Validação: ❌ BLOQUEADO - Non-goal explícito em feature/authentication.md
   Sugestão: Confirme se requisito mudou ou se é para versão futura
   ```

---

## 🚨 Princípio Jidoka (Fail-Fast)

**Se IA detectar violação de Intent**:

1. 🛑 **PARE** a execução imediatamente
2. ⚠️ **ALERTE** o humano sobre a violação
3. 📋 **MOSTRE** qual intent foi violado e como
4. 🤔 **PERGUNTE** se intent deve ser atualizado ou decisão reconsiderada

**Exemplo**:
```
🛑 VIOLAÇÃO DE INTENT DETECTADA

Spec: specs/technical/CLAUDE.meta.md
Intent Violado: Constraint "Não alterar contratos públicos"

Ação Tentada: Remover prop `variant` de ButtonPrimary.vue

Impacto:
- 15 componentes usam esta prop
- Breaking change para todos os consumidores

Sugestões:
1. Deprecar prop gradualmente (manter + adicionar warning)
2. Criar novo componente ButtonV2 (sem breaking change)
3. Atualizar intent se mudança é necessária

Deseja prosseguir? (requer aprovação explícita)
```

---

## 🔄 Manutenção de Intents

**Quando atualizar intent**:

1. **Mudança de estratégia** (ex: "agora vamos suportar OAuth"):
   - Atualizar `Non-Goals` (remover OAuth)
   - Pode adicionar aos `Goals` ou `Constraints`
   - Incrementar versão MINOR
   - Documentar mudança

2. **Novo constraint descoberto** (ex: "descobrimos que precisamos WCAG AA"):
   - Adicionar a `Constraints`
   - Incrementar versão MINOR
   - Documentar mudança

3. **Clarificação** (sem mudança de significado):
   - Melhorar redação
   - Incrementar versão PATCH
   - Documentar mudança

---

## 📚 Referência: Tipos de Constraints Comuns

**Constraints Técnicos**:
- Stack tecnológica (ex: "Apenas Vue 3, NestJS, MongoDB")
- Padrões arquiteturais (ex: "Atomic Design obrigatório")
- Qualidade (ex: "TypeScript strict mode", "Cobertura > 80%")
- Performance (ex: "FCP < 1.5s", "Bundle size < 200KB")
- Segurança (ex: "Bcrypt min 10 rounds", "JWT exp < 24h")

**Constraints de Negócio**:
- Budget (ex: "Zero custo", "Max R$100/mês infra")
- Timeline (ex: "MVP em 6 sprints", "Feature freeze 1 semana antes demo")
- Escopo (ex: "Apenas features do roadmap Q1")
- Compliance (ex: "LGPD compliant", "Sem dados sensíveis em logs")

**Constraints de Processo**:
- Retrocompatibilidade (ex: "Sem breaking changes em minor versions")
- Code review (ex: "Toda mudança requer aprovação")
- Testes (ex: "E2E para fluxos críticos obrigatório")
- Documentação (ex: "Código complexo deve ter comentários")

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, adicione Intent a TODAS as specs principais.

Se argumentos forem fornecidos (ex: `technical` ou caminho específico), adicione Intent apenas às specs especificadas.
