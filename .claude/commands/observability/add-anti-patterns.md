# Adicionar Anti-Patterns às Metaspecs - Crypteras

Este comando documenta **Anti-Patterns** - padrões que devem ser evitados no desenvolvimento do **Crypteras Trading System** com IA.

> **Nota**: Este é um comando avançado de observabilidade. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Anti-Patterns

**Anti-Patterns** são soluções que parecem corretas mas causam problemas:
- Prompt-only development
- Context dump
- Specs não versionadas
- RAG sem governança
- Agentes sem limites claros

**Benefício**: Prevenir erros comuns em desenvolvimento orientado por IA.

---

## 📋 Objetivo

Documentar anti-patterns para:
- Prevenir erros comuns
- Educar desenvolvedores
- Guiar IA para evitar más práticas
- Criar cultura de qualidade

---

## 🔍 Anti-Patterns Comuns

### 1. Prompt-Only Development

**Descrição**: Confiar apenas em prompts sem especificações estruturadas.

**Por que é ruim**:
- Contexto não é versionado
- Sem governança
- Difícil auditar decisões
- Context drift inevitável

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

Desenvolvedor: "Claude, crie um componente de card para metas"

(Sem specs, sem versionamento, sem validação)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

1. Especificar em specs/business/features/goal-management.md
2. Versionar spec (v1.0.0)
3. Adicionar Intent as Code
4. IA consulta spec + valida contra ADRs
5. Decisão rastreável e auditável
```

**Arquivo**: Documentar em `specs/_meta/ANTI_PATTERNS.md`

---

### 2. Context Dump

**Descrição**: Jogar todo contexto possível para a IA sem estrutura.

**Por que é ruim**:
- IA não sabe o que é importante
- Contexto vira noise
- Sem priorização
- Tokens desperdiçados

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

"Aqui está todo o código do projeto (50k linhas), toda documentação
(100 arquivos), todo o histórico de git (1000 commits)... agora crie
um componente de botão."
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

Contexto estruturado em camadas:
1. Meta Specs (governança) - sempre
2. Specs relevantes (CLAUDE.meta.md, CODEBASE_GUIDE.md)
3. Código relacionado (componentes similares)
4. Exemplos específicos (1-2 files)

Total: ~5-10 arquivos relevantes ao invés de 150 irrelevantes
```

---

### 3. Specs Não Versionadas

**Descrição**: Criar specs sem versionamento semântico.

**Por que é ruim**:
- Context clash inevitável
- Impossível rastrear mudanças
- Sem rollback possível
- Breaking changes silenciosos

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

# CUSTOMER_PERSONAS.md

(Sem frontmatter, sem versão, sem histórico)

Ana é estudante...
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

---
spec_version: "1.0.0"
valid_from: "2025-12-20"
status: "active"
---

# CUSTOMER_PERSONAS.md

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-20
:::

Ana é estudante...
```

**Comando para corrigir**: `/add-versioning`

---

### 4. RAG Sem Governança

**Descrição**: Usar RAG (Retrieval-Augmented Generation) sem validar qualidade dos resultados.

**Por que é ruim**:
- IA recupera contexto obsoleto
- Sem validação de versão
- Documentação contradizendo código
- Decisões baseadas em info errada

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

RAG recupera: "Usamos MongoDB" (spec v1.0.0 de 6 meses atrás)
Código atual: PostgreSQL
IA gera: Código com Mongoose (erro!)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

1. RAG recupera contexto
2. Validar versão (spec_version)
3. Verificar status (active/deprecated)
4. Cross-check com código atual
5. Usar versão mais recente em caso de conflito
```

**Comando para validar**: `/validate-context`

---

### 5. Agentes Sem Limites

**Descrição**: Agentes de IA sem constraints ou non-goals claros.

**Por que é ruim**:
- Scope creep silencioso
- IA faz mais do que deveria
- Soluções "clever" demais
- Refatorações não solicitadas

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

Solicitação: "Adicione validação de email"

IA faz (sem constraint):
1. ✅ Validação de email
2. ❌ Refatora todo sistema de validação
3. ❌ Adiciona biblioteca Zod (nova dependência)
4. ❌ Cria camada de abstração genérica
5. ❌ Reescreve 15 arquivos não relacionados
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

Spec com Intent as Code:

:::intent
**Goal**: Adicionar validação de email

**Constraints**:
- Usar regex simples (sem libs novas)
- Modificar APENAS arquivo de validação
- Manter retrocompatibilidade

**Non-Goals**:
- Refatoração ampla
- Nova biblioteca
- Abstrações genéricas
- Mudanças não relacionadas
:::
```

**Comando para adicionar**: `/add-intent`

---

### 6. Specs Genéricas (Copy-Paste de Exemplos)

**Descrição**: Copiar templates sem adaptar ao projeto.

**Por que é ruim**:
- Contexto genérico não ajuda
- Exemplos irrelevantes
- IA não sabe o que é específico do projeto
- Perda de valor das specs

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

# CUSTOMER_PERSONAS.md

## Persona: John Doe
Age: 30-40
Job: Manager
...

(Persona genérica de template, não real do projeto)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

# CUSTOMER_PERSONAS.md

## Persona: Ana - Estudante Organizada
**Demografia**: 18-24 anos, universitária
**Objetivo**: Gerenciar metas de estudo e carreira
**Pain Point**: Metas vagas, sem acompanhamento
**Tech Savvy**: Alto (usa apps de produtividade)

(Persona específica do projeto MetaCerta, baseada em pesquisa real)
```

---

### 7. Documentação e Código Divergentes

**Descrição**: Documentação não reflete código real.

**Por que é ruim**:
- IA usa docs desatualizadas
- Decisões baseadas em contexto falso
- Context drift não detectado

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

Spec: "Usamos Vue 2 Options API"
Código: 100% Vue 3 Composition API

IA gera: Código com Options API (erro!)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

1. Spec: "Usamos Vue 3 Composition API (ADR-001 v1.0.0)"
2. Código: Vue 3 Composition API
3. Validação automática: /check-drift
4. CI bloqueia se drift detectado
```

**Comando para detectar**: `/check-drift`

---

### 8. Sem Failure Modes Documentados

**Descrição**: Não documentar falhas conhecidas.

**Por que é ruim**:
- Mesmos erros se repetem
- IA "alucina" em cenários conhecidos
- Sem mitigação preventiva

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

IA gera (pela 10ª vez):
const goal = props.goal  // Perde reatividade!

(Falha conhecida mas não documentada)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

:::failure_modes
1. **Reactive Loss em Props**
   - Tipo: hallucination
   - Gatilho: `const x = props.y`
   - Mitigação: Sempre usar `toRef(props, 'y')`
   - Detecção: Props não atualizam
:::
```

**Comando para adicionar**: `/add-failure-modes`

---

### 9. Explainability Ausente

**Descrição**: IA toma decisões sem explicar por quê.

**Por que é ruim**:
- Impossível auditar
- Difícil debugar
- Falta de confiança
- Sem aprendizado

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

IA: "Pronto, criei o componente."

(Sem explicar: Por que usou computed? Por que esta estrutura?
 Quais specs consultou? Quais alternativas considerou?)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

## 🤖 Decisão de Desenvolvimento

**Decisão**: Usar `computed` para cálculo de progresso

**Source**: CLAUDE.meta.md v1.2.0 - Seção "Computed vs Watch"

**Rationale**:
1. `computed` é padrão para valores derivados
2. Consistente com código existente
3. Performance superior

**Alternatives**: watch (rejeitado - não idiomático)
```

**Comando para configurar**: `/add-explainability`

---

### 10. Hierarquia de Contexto Inexistente

**Descrição**: Sem precedência clara quando specs conflitam.

**Por que é ruim**:
- IA não sabe qual spec seguir
- Conflitos não resolvidos
- Decisões inconsistentes

**Exemplo**:
```markdown
❌ ANTI-PATTERN:

Business spec: "Feature X é prioridade"
Technical spec: "Feature X não é viável"

IA: ???  (não sabe qual seguir)
```

**Solução**:
```markdown
✅ PATTERN CORRETO:

Hierarquia clara:
1. Meta Specs > 2. Business > 3. Technical > 4. Execution

Conflito: Business vence Technical
→ Technical deve propor mitigação ou workaround
→ Escalar se impossível
```

**Comando para validar**: `/validate-hierarchy`

---

## 📝 Estrutura de Documentação

**Criar arquivo**: `specs/_meta/ANTI_PATTERNS.md`

```markdown
# Anti-Patterns em Desenvolvimento com IA

Lista de padrões a evitar ao trabalhar com IA e metaspecs.

## 1. Prompt-Only Development ❌

**Descrição**: [...]
**Por que é ruim**: [...]
**Exemplo**: [...]
**Solução**: [...]

---

## 2. Context Dump ❌

**Descrição**: [...]
...

---

[Continuar para todos os 10 anti-patterns]

---

## Como Evitar Anti-Patterns

### Checklist de Qualidade

Antes de gerar código com IA, verificar:
- [ ] Specs estão versionadas? (/add-versioning)
- [ ] Intent está definido? (/add-intent)
- [ ] Failure modes documentados? (/add-failure-modes)
- [ ] Explainability configurado? (/add-explainability)
- [ ] Hierarquia clara? (/validate-hierarchy)
- [ ] Contexto validado? (/validate-context)
- [ ] Sem drift detectado? (/check-drift)

### Code Review de IA

Ao revisar código gerado por IA, verificar:
- [ ] Decisões explicadas?
- [ ] Specs consultadas (e versões)?
- [ ] Constraints respeitados?
- [ ] Failure modes mitigados?
- [ ] Sem refatorações não solicitadas?
- [ ] Código alinhado com ADRs?
```

---

## ✅ Checklist de Implementação

**Documentar anti-patterns**:
- [ ] Criar `specs/_meta/ANTI_PATTERNS.md`
- [ ] Listar os 10 anti-patterns principais
- [ ] Para cada um: descrição, problema, exemplo, solução
- [ ] Adicionar checklist de prevenção
- [ ] Adicionar aos índices

**Educar time**:
- [ ] Compartilhar anti-patterns doc
- [ ] Code review checklist
- [ ] Exemplos de boas/más práticas
- [ ] Workshop interno (se aplicável)

**Validação contínua**:
- [ ] CI verifica anti-patterns
- [ ] Code review inclui checklist
- [ ] Métricas de qualidade
- [ ] Retrospectivas identificam novos anti-patterns

---

## 🎯 Benefícios

✅ **Prevenção**: Evitar erros antes de acontecerem
✅ **Educação**: Time aprende boas práticas
✅ **Qualidade**: Código mais consistente
✅ **Eficiência**: Menos retrabalho
✅ **Confiança**: IA gera código melhor

---

## 📊 Exemplo Completo

**Cenário**: Code review identifica anti-pattern

**Problema detectado**:
```typescript
// ❌ Anti-Pattern: Prompt-Only Development
// Código gerado sem spec, sem versionamento, sem validação

const calculateProgress = (goal: any) => {  // any = mal sinal
  return goal.actions.length > 0
    ? goal.actions.filter(a => a.done).length / goal.actions.length
    : 0;
};
```

**Correção**:

1. **Criar spec** em `specs/business/features/goal-management.md`
2. **Versionar** spec (v1.0.0)
3. **Adicionar Intent**:
   ```markdown
   :::intent
   **Goal**: Cálculo de progresso simples e automático
   **Constraints**: Sempre entre 0-100%
   **Non-Goals**: Ponderação complexa
   :::
   ```
4. **Documentar failure mode**:
   ```markdown
   :::failure_modes
   - **Divisão por zero**: Se meta sem ações
   - **Mitigação**: Retornar 0 se total === 0
   :::
   ```
5. **Gerar código correto**:
   ```typescript
   // ✅ Pattern Correto
   // Gerado com base em goal-management.md v1.0.0
   /**
    * @sources:
    *   - specs/business/features/goal-management.md v1.0.0
    *   - specs/technical/BUSINESS_LOGIC.md v1.0.0
    */
   const calculateProgress = (goal: Goal): number => {
     const total = goal.actions.length;
     if (total === 0) return 0;  // Mitigação de failure mode
     const completed = goal.actions.filter(a => a.completed).length;
     return Math.round((completed / total) * 100);
   };
   ```

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, cria `ANTI_PATTERNS.md` com os 10 anti-patterns principais.

Se argumentos forem fornecidos:
- `add <anti-pattern-name>`: Adiciona anti-pattern específico
- `check`: Verifica código contra anti-patterns conhecidos
- `list`: Lista anti-patterns documentados

Flags opcionais:
- `--with-examples`: Inclui exemplos de código
- `--with-solutions`: Inclui soluções detalhadas
