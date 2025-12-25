---
spec_version: "1.0.0"
valid_from: "2025-12-25"
status: "active"
applies_to: ["all"]
---

# Anti-Patterns em Desenvolvimento com IA - Crypteras

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Active
:::

Este documento lista **padrões que devem ser evitados** ao trabalhar com IA e metaspecs no desenvolvimento do Crypteras Trading System.

> **Importante**: Anti-patterns são soluções que parecem corretas mas causam problemas. Consulte este documento durante code reviews e ao configurar workflows de IA.

---

## 📋 Índice de Anti-Patterns

1. [Prompt-Only Development](#1-prompt-only-development-)
2. [Context Dump](#2-context-dump-)
3. [Specs Não Versionadas](#3-specs-não-versionadas-)
4. [RAG Sem Governança](#4-rag-sem-governança-)
5. [Agentes Sem Limites](#5-agentes-sem-limites-)
6. [Specs Genéricas](#6-specs-genéricas-copy-paste-)
7. [Documentação e Código Divergentes](#7-documentação-e-código-divergentes-)
8. [Sem Failure Modes Documentados](#8-sem-failure-modes-documentados-)
9. [Explainability Ausente](#9-explainability-ausente-)
10. [Hierarquia de Contexto Inexistente](#10-hierarquia-de-contexto-inexistente-)

---

## 1. Prompt-Only Development ❌

### Descrição
Confiar apenas em prompts ad-hoc sem especificações estruturadas e versionadas.

### Por que é ruim
- ✘ Contexto não é versionado
- ✘ Sem governança ou rastreabilidade
- ✘ Difícil auditar decisões
- ✘ Context drift inevitável
- ✘ Conhecimento perdido entre iterações

### Exemplo

```markdown
❌ ANTI-PATTERN:

Desenvolvedor → IA:
"Claude, crie um componente para exibir estratégias de trading"

(Sem specs, sem versionamento, sem validação, sem governança)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

1. Especificar em specs/business/features/trading-contexts.md
2. Versionar spec (v1.0.0) com frontmatter
3. Adicionar Intent as Code com goals/constraints
4. IA consulta spec + valida contra ADRs
5. Decisão rastreável e auditável

Resultado: Código consistente, decisões documentadas, contexto versionado
```

### Comando de Correção
```bash
# Criar spec estruturada ao invés de prompt solto
/build-business-docs
/add-intent
/add-versioning
```

---

## 2. Context Dump ❌

### Descrição
Jogar todo contexto possível para a IA sem estrutura ou priorização.

### Por que é ruim
- ✘ IA não sabe o que é importante vs. ruído
- ✘ Contexto vira noise ao invés de signal
- ✘ Sem priorização clara
- ✘ Tokens desperdiçados
- ✘ Performance degradada

### Exemplo

```markdown
❌ ANTI-PATTERN:

"Aqui está todo o código do projeto (50k linhas), toda documentação
(100 arquivos), todo o histórico de git (1000 commits), todos os ADRs,
todas as specs business e technical...

Agora crie um endpoint de autenticação simples."

(Contexto irrelevante 99%, sinal perdido em ruído)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

Contexto estruturado em camadas de relevância:

1. **Meta Specs** (sempre): GOVERNANCE_RULES.md, META_SPEC_SPEC.md
2. **Specs Relevantes**:
   - specs/business/features/authentication-and-authorization.md
   - specs/technical/CLAUDE.meta.md
3. **Código Relacionado**:
   - backend/api/auth/routes.py (exemplo similar)
4. **ADRs Aplicáveis**:
   - adr/006-agno-framework-ai.md

Total: ~5-7 arquivos altamente relevantes ao invés de 150 irrelevantes
```

### Comando de Correção
```bash
# Validar que apenas contexto relevante está sendo usado
/validate-context
/track-context
```

---

## 3. Specs Não Versionadas ❌

### Descrição
Criar especificações sem versionamento semântico e frontmatter.

### Por que é ruim
- ✘ Context clash inevitável entre versões
- ✘ Impossível rastrear mudanças
- ✘ Sem rollback possível
- ✘ Breaking changes silenciosos
- ✘ IA não sabe qual versão usar

### Exemplo

```markdown
❌ ANTI-PATTERN:

# CUSTOMER_PERSONAS.md

Ana é estudante universitária que precisa organizar suas metas...

(Sem frontmatter, sem versão, sem data, sem status, sem histórico)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

---
spec_version: "1.0.0"
valid_from: "2025-12-25"
status: "active"
applies_to: ["frontend", "product"]
---

# CUSTOMER_PERSONAS.md

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Active
:::

## Ana - Trader Iniciante

**Demografia**: 25-35 anos, profissional tech
**Objetivo**: Automatizar trading de crypto
...
```

### Comando de Correção
```bash
# Adicionar versionamento a specs existentes
/add-versioning

# Validar que todas specs têm frontmatter
python validate_frontmatter.py
```

---

## 4. RAG Sem Governança ❌

### Descrição
Usar RAG (Retrieval-Augmented Generation) sem validar qualidade e atualidade dos resultados.

### Por que é ruim
- ✘ IA recupera contexto obsoleto
- ✘ Sem validação de versão
- ✘ Documentação contradizendo código
- ✘ Decisões baseadas em informação errada
- ✘ Context drift não detectado

### Exemplo

```markdown
❌ ANTI-PATTERN:

RAG recupera: "Usamos MongoDB para dados transacionais"
(spec v1.0.0 de 6 meses atrás, status: deprecated)

Código atual: PostgreSQL em produção

IA gera: Código com Mongoose/MongoDB (ERRO!)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

RAG Pipeline com Validação:

1. RAG recupera contexto
2. **Validar versão**: spec_version
3. **Verificar status**: active | deprecated | obsolete
4. **Cross-check com código**: grep codebase
5. **Precedência**: Código real > Spec mais recente > Spec antiga
6. **Alertar conflitos**: Context drift detectado

Resultado: IA usa MongoDB spec v3.0.0 (active) + valida contra código
```

### Comando de Correção
```bash
# Validar contexto antes de usar
/validate-context
/check-drift

# Auditar spec para garantir qualidade
/audit-spec
```

---

## 5. Agentes Sem Limites ❌

### Descrição
Agentes de IA sem constraints, non-goals ou limites claros.

### Por que é ruim
- ✘ Scope creep silencioso
- ✘ IA faz mais do que deveria
- ✘ Soluções "clever" demais (over-engineering)
- ✘ Refatorações não solicitadas
- ✘ Dependências não aprovadas

### Exemplo

```markdown
❌ ANTI-PATTERN:

Solicitação: "Adicione validação de credenciais de exchange"

IA faz (sem constraints):
1. ✅ Validação de credenciais (OK)
2. ❌ Refatora todo sistema de validação (não pedido)
3. ❌ Adiciona biblioteca Zod (nova dependência)
4. ❌ Cria camada de abstração genérica (over-engineering)
5. ❌ Reescreve 15 arquivos não relacionados (scope creep)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

Spec com Intent as Code:

:::intent
**Goal**: Validar credenciais de exchange (API key/secret)

**Constraints**:
- Usar validação simples (sem libs novas)
- Modificar APENAS backend/api/exchanges/validator.py
- Manter retrocompatibilidade com código existente
- Seguir padrão de outras validações no projeto

**Non-Goals**:
- ❌ Refatoração ampla do sistema
- ❌ Nova biblioteca de validação
- ❌ Abstrações genéricas
- ❌ Mudanças em arquivos não relacionados
:::

Resultado: IA faz apenas o solicitado, nada mais
```

### Comando de Correção
```bash
# Adicionar intent com constraints claros
/add-intent
```

---

## 6. Specs Genéricas (Copy-Paste) ❌

### Descrição
Copiar templates ou exemplos genéricos sem adaptar ao projeto específico.

### Por que é ruim
- ✘ Contexto genérico não ajuda IA
- ✘ Exemplos irrelevantes para o projeto
- ✘ IA não sabe o que é específico do Crypteras
- ✘ Perda de valor das specs
- ✘ Poluição de contexto

### Exemplo

```markdown
❌ ANTI-PATTERN:

# CUSTOMER_PERSONAS.md

## Persona: John Doe
**Age**: 30-40
**Job**: Manager
**Goals**: Be productive
**Pain Points**: Not organized

(Persona genérica de template, não real do Crypteras)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

# CUSTOMER_PERSONAS.md

## Persona: Carlos - Trader Experiente

**Demografia**: 28-45 anos, profissional de tecnologia ou finanças
**Experiência**: 2-5 anos em trading de crypto
**Objetivo**: Automatizar estratégias de trading complexas
**Pain Point**: Bots existentes são inflexíveis ou não suportam múltiplas exchanges
**Tech Savvy**: Alto (confortável com APIs e configurações técnicas)
**Exchanges**: Mercado Bitcoin, Binance
**Estratégias**: DCA, Grid Trading, arbitragem

(Persona específica do Crypteras, baseada em usuários reais)
```

### Comando de Correção
```bash
# Rebuild specs com contexto real do projeto
/build-business-docs
```

---

## 7. Documentação e Código Divergentes ❌

### Descrição
Documentação não reflete o código real em produção (context drift).

### Por que é ruim
- ✘ IA usa documentação desatualizada
- ✘ Decisões baseadas em contexto falso
- ✘ Context drift não detectado
- ✘ Código gerado quebra em produção
- ✘ Perda de confiança nas specs

### Exemplo

```markdown
❌ ANTI-PATTERN:

Spec (specs/technical/frontend/vue.md):
"Usamos Vue 2 Options API em todos componentes"

Código real (frontend/):
100% Vue 3 Composition API

IA gera: Código com Options API (ERRO! Incompatível!)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

1. **Spec atualizada**:
   "Usamos Vue 3 Composition API (ADR-001 v1.0.0)"

2. **Código consistente**:
   100% Vue 3 Composition API

3. **Validação automática**:
   /check-drift detecta divergências

4. **CI/CD bloqueio**:
   Pipeline falha se drift detectado

5. **Update protocol**:
   Código muda → Spec atualiza (MAJOR version bump)
```

### Comando de Correção
```bash
# Detectar drift entre docs e código
/check-drift

# Auditar spec para garantir alinhamento
/audit-spec
```

---

## 8. Sem Failure Modes Documentados ❌

### Descrição
Não documentar falhas conhecidas, bugs recorrentes ou edge cases.

### Por que é ruim
- ✘ Mesmos erros se repetem
- ✘ IA "alucina" em cenários conhecidos
- ✘ Sem mitigação preventiva
- ✘ Perda de conhecimento tribal
- ✘ Debugging recorrente

### Exemplo

```markdown
❌ ANTI-PATTERN:

IA gera (pela 10ª vez):

// Bug conhecido mas não documentado
const strategy = props.strategy  // Perde reatividade!

computed(() => {
  return strategy.status  // Não atualiza! (Vue reactivity loss)
})
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

:::failure_modes

### 1. Reactive Loss em Props (Vue)
**Tipo**: hallucination
**Frequência**: Alta (comum em código gerado por IA)
**Gatilho**: Destructuring ou const direto de props
**Sintoma**: Props não atualizam na interface

**Exemplo Errado**:
```typescript
const strategy = props.strategy  // ❌ Perde reatividade
```

**Solução**:
```typescript
const strategy = toRef(props, 'strategy')  // ✅ Mantém reatividade
```

**Detecção**: Props não reagem a mudanças do parent
**Mitigação**: Sempre usar toRef/toRefs para props

---

### 2. Race Condition em Ordens de Trading
**Tipo**: edge_case
**Gatilho**: Múltiplas ordens simultâneas na mesma exchange
**Sintoma**: Ordens duplicadas ou conflitantes
**Solução**: Lock baseado em symbol+exchange antes de executar ordem
:::
```

### Comando de Correção
```bash
# Adicionar failure modes a specs
/add-failure-modes
```

---

## 9. Explainability Ausente ❌

### Descrição
IA toma decisões sem explicar por quê, quais specs consultou ou alternativas considerou.

### Por que é ruim
- ✘ Impossível auditar decisões
- ✘ Difícil debugar código gerado
- ✘ Falta de confiança na IA
- ✘ Sem aprendizado organizacional
- ✘ Black box inaceitável

### Exemplo

```markdown
❌ ANTI-PATTERN:

Desenvolvedor: "Adicione cálculo de risco para estratégias"

IA: "Pronto, criei o método calculateRisk() ✓"

(Sem explicar: Por que esta fórmula? Qual spec consultou?
 Quais alternativas considerou? Por que esta abordagem?)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

## 🤖 Decisão de Desenvolvimento

**Decisão**: Implementar cálculo de risco baseado em Kelly Criterion

**Source Specs**:
- specs/business/strategies/risk-management.md v2.0.0
- specs/technical/BUSINESS_LOGIC.md v1.5.0

**Rationale**:
1. Kelly Criterion é padrão da indústria para sizing
2. Alinha com requisito de "gestão matemática de risco" (business spec)
3. Consistente com código existente em risk_manager.py
4. Performance testada e validada

**Alternatives Considered**:
- Fixed % per trade (rejeitado - muito conservador)
- Martingale (rejeitado - risco exponencial)
- Risk parity (considerado para v2)

**Implementation**:
- File: backend/domain/risk/kelly_calculator.py
- Tests: tests/risk/test_kelly_criterion.py
- Validation: Backtest com dados históricos
```

### Comando de Correção
```bash
# Configurar IA para sempre explicar decisões
/add-explainability
```

---

## 10. Hierarquia de Contexto Inexistente ❌

### Descrição
Sem precedência clara quando specs conflitam ou contradizem código.

### Por que é ruim
- ✘ IA não sabe qual spec seguir
- ✘ Conflitos não resolvidos
- ✘ Decisões inconsistentes
- ✘ Paralisia de decisão
- ✘ Código incoerente

### Exemplo

```markdown
❌ ANTI-PATTERN:

Conflito detectado:

Business spec (v1.0.0): "Feature de Grid Trading é prioridade máxima"
Technical spec (v1.0.0): "Grid Trading não é viável com arquitetura atual"
ADR-005 (v1.0.0): "Adapter pattern deve suportar qualquer estratégia"

IA: ??? (não sabe qual seguir, gera código quebrado ou para)
```

### Solução Correta

```markdown
✅ PATTERN CORRETO:

Hierarquia Definida (META_SPEC_SPEC.md):

1. **Meta Specs** (maior precedência)
   - GOVERNANCE_RULES.md
   - META_SPEC_SPEC.md

2. **Business Specs**
   - specs/business/*

3. **Technical Specs**
   - specs/technical/*
   - ADRs

4. **Execution Context**
   - Código atual
   - Environment

**Resolução de Conflito**:

Conflito: Business quer Grid Trading vs. Technical diz não viável

Precedência: Business > Technical
→ Technical deve propor mitigação ou workaround
→ Se impossível, escalar para Product Owner
→ Documentar decisão em ADR

Resultado: IA tenta implementar, documenta limitações técnicas, propõe roadmap
```

### Comando de Correção
```bash
# Validar hierarquia e resolver conflitos
/validate-hierarchy
```

---

## 🛡️ Como Evitar Anti-Patterns

### Checklist de Qualidade Pré-Desenvolvimento

Antes de gerar código com IA, verificar:

- [ ] **Specs versionadas?** → `/add-versioning`
- [ ] **Intent definido?** → `/add-intent`
- [ ] **Failure modes documentados?** → `/add-failure-modes`
- [ ] **Explainability configurado?** → `/add-explainability`
- [ ] **Hierarquia clara?** → `/validate-hierarchy`
- [ ] **Contexto validado?** → `/validate-context`
- [ ] **Sem drift detectado?** → `/check-drift`

### Code Review Checklist

Ao revisar código gerado por IA, verificar:

**Rastreabilidade**:
- [ ] Decisões explicadas com rationale?
- [ ] Specs consultadas (com versões)?
- [ ] Alternatives consideradas e documentadas?

**Constraints**:
- [ ] Constraints respeitados?
- [ ] Non-goals não violados?
- [ ] Sem scope creep?

**Qualidade**:
- [ ] Failure modes mitigados?
- [ ] Sem refatorações não solicitadas?
- [ ] Código alinhado com ADRs?
- [ ] Testes incluídos?

**Contexto**:
- [ ] Specs estão atualizadas?
- [ ] Sem context drift?
- [ ] Versões corretas usadas?

---

## 📊 Exemplo Completo: Detectar e Corrigir Anti-Pattern

### Cenário
Code review detecta múltiplos anti-patterns em PR.

### Problema Detectado

```typescript
// ❌ MÚLTIPLOS ANTI-PATTERNS DETECTADOS:

// Anti-Pattern #1: Prompt-Only Development
// (Código gerado sem spec, sem versionamento)

// Anti-Pattern #5: Agentes Sem Limites
// (Refatoração não solicitada, over-engineering)

// Anti-Pattern #9: Explainability Ausente
// (Sem explicar decisões ou fontes)

class TradingStrategyManager {
  // any type = red flag
  private strategies: any[] = [];

  // Over-engineering: cache não solicitado
  private cache: Map<string, any> = new Map();

  // Método genérico demais (scope creep)
  async executeStrategy(strategyId: string, options?: any) {
    // Validação ausente (failure mode não mitigado)
    const strategy = this.strategies.find(s => s.id === strategyId);
    return strategy.execute(options);
  }

  // Feature não solicitada
  private clearCache() {
    this.cache.clear();
  }
}
```

### Correção Passo a Passo

#### 1️⃣ Criar Spec Estruturada

```bash
/build-business-docs
```

Criar `specs/business/features/trading-contexts.md`:

```markdown
---
spec_version: "1.0.0"
valid_from: "2025-12-25"
status: "active"
---

# Trading Contexts and Strategies

:::intent
**Goal**: Gerenciar execução de estratégias de trading

**Constraints**:
- Suportar múltiplas exchanges
- Tipo-safe (TypeScript strict)
- Sem cache (YAGNI - You Aren't Gonna Need It)

**Non-Goals**:
- ❌ Cache genérico
- ❌ Abstrações complexas
- ❌ Features não solicitadas
:::

:::failure_modes
### Estratégia Não Encontrada
**Mitigação**: Validar strategyId antes de executar
**Error**: Throw StrategyNotFoundError
:::
```

#### 2️⃣ Adicionar Intent e Failure Modes

```bash
/add-intent
/add-failure-modes
```

#### 3️⃣ Gerar Código Correto com Explainability

```typescript
// ✅ PATTERN CORRETO

/**
 * Trading Strategy Manager
 *
 * @sources
 *   - specs/business/features/trading-contexts.md v1.0.0
 *   - specs/technical/BUSINESS_LOGIC.md v1.5.0
 *   - adr/005-adapter-pattern-exchanges.md v1.0.0
 *
 * @decisions
 *   - No cache: YAGNI principle (business spec non-goal)
 *   - Type-safe: TypeScript strict mode (technical spec)
 *   - Validation: Mitigate StrategyNotFoundError (failure mode)
 */

import { TradingStrategy, StrategyNotFoundError } from './types';

class TradingStrategyManager {
  private strategies: Map<string, TradingStrategy>;

  constructor(strategies: TradingStrategy[]) {
    this.strategies = new Map(
      strategies.map(s => [s.id, s])
    );
  }

  /**
   * Execute trading strategy
   *
   * @throws {StrategyNotFoundError} If strategy not found
   */
  async executeStrategy(strategyId: string): Promise<void> {
    // Mitigation: Validate before execute (failure mode)
    const strategy = this.strategies.get(strategyId);

    if (!strategy) {
      throw new StrategyNotFoundError(
        `Strategy ${strategyId} not found`
      );
    }

    await strategy.execute();
  }
}
```

#### 4️⃣ Validar

```bash
# Verificar que não há drift
/check-drift

# Validar hierarquia
/validate-hierarchy

# Auditar spec
/audit-spec
```

### Resultado

✅ **Anti-patterns corrigidos**:
- ✅ Spec estruturada e versionada
- ✅ Intent com constraints claros
- ✅ Failure modes mitigados
- ✅ Explainability completo
- ✅ Sem over-engineering
- ✅ Type-safe
- ✅ Rastreável e auditável

---

## 🎯 Benefícios de Evitar Anti-Patterns

### Para o Time
- ✅ **Prevenção**: Evitar erros antes de acontecerem
- ✅ **Educação**: Aprender boas práticas continuamente
- ✅ **Eficiência**: Menos retrabalho e debugging

### Para o Código
- ✅ **Qualidade**: Código mais consistente e maintainável
- ✅ **Rastreabilidade**: Decisões auditáveis
- ✅ **Confiança**: IA gera código melhor

### Para o Produto
- ✅ **Velocidade**: Desenvolvimento mais rápido
- ✅ **Estabilidade**: Menos bugs em produção
- ✅ **Escalabilidade**: Arquitetura mais sólida

---

## 📚 Referências

- [META_SPEC_SPEC.md](./META_SPEC_SPEC.md) - Governança de specs
- [GOVERNANCE_RULES.md](./GOVERNANCE_RULES.md) - Regras de governança
- [CLAUDE.meta.md](../technical/CLAUDE.meta.md) - Boas práticas de código

---

## 🔄 Manutenção

Este documento deve ser atualizado quando:

1. **Novo anti-pattern identificado**: Code review encontra padrão ruim recorrente
2. **Evolução de specs**: Novas versões do META_SPEC_SPEC.md
3. **Feedback do time**: Retrospectivas identificam problemas

**Responsável**: Tech Lead + IA Engineer
**Frequência**: Revisão trimestral ou quando necessário
**Versão**: Seguir semver (MAJOR para novos anti-patterns críticos)

---

:::metadata
**Criado**: 2025-12-25
**Autor**: Crypteras Team
**Revisores**: Tech Lead, IA Engineer
**Próxima Revisão**: 2026-03-25
:::
