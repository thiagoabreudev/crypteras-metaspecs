---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2025-12-26"
supersedes: null
status: "active"
---

# Governança de Contexto - Metaspecs Crypteras

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão do sistema de governança de contexto
- Estabelece versionamento semântico para todas as specs
:::

---

## 🎯 O que é Governança de Contexto?

**Context Governance** é o conjunto de práticas, políticas e ferramentas que garantem que o contexto fornecido à IA (e equipes humanas) seja:

- ✅ **Correto**: Informações precisas e verificadas
- ✅ **Atual**: Reflete o estado real do sistema
- ✅ **Consistente**: Sem contradições entre diferentes specs
- ✅ **Intencional**: Mudanças são deliberadas e documentadas
- ✅ **Rastreável**: Histórico completo de evolução

---

## 📁 Arquivos deste Diretório

### [VERSIONING_POLICY.md](./VERSIONING_POLICY.md)
Política oficial de versionamento semântico para especificações.

**Quando consultar**:
- Antes de atualizar qualquer spec
- Para decidir se uma mudança é MAJOR, MINOR ou PATCH
- Para entender o processo de atualização de versão

**Conteúdo principal**:
- Regras de SemVer (MAJOR.MINOR.PATCH)
- Metadados obrigatórios
- Processo de atualização
- Exemplos práticos

### [VERSION_HISTORY.md](./VERSION_HISTORY.md)
Histórico centralizado de todas as mudanças de versão em todas as specs.

**Quando consultar**:
- Para verificar quando uma spec foi atualizada
- Para ver o histórico de mudanças de uma spec específica
- Ao fazer auditoria de contexto

**Conteúdo principal**:
- Tabelas de versionamento por spec
- Tipos de mudança (major, minor, patch, baseline)
- Descrições concisas de cada versão

### [README.md](./README.md) (este arquivo)
Visão geral do sistema de governança de contexto.

### [DECISION_LOG.md](./DECISION_LOG.md)
Log de decisões arquiteturais e de governança.

**Quando consultar**:
- Para entender decisões passadas sobre governança
- Ao propor mudanças no sistema de versionamento
- Durante auditorias de conformidade

**Conteúdo principal**:
- Registro cronológico de decisões
- Justificativas e contexto
- Impacto de cada decisão

### [GOVERNANCE_SUMMARY.md](./GOVERNANCE_SUMMARY.md)
Resumo executivo do sistema de governança.

**Quando consultar**:
- Para visão rápida do sistema
- Ao onboarding de novos membros
- Para apresentações executivas

**Conteúdo principal**:
- Objetivos do sistema de governança
- Principais políticas e processos
- Métricas de sucesso

### [ANTI_PATTERNS.md](./ANTI_PATTERNS.md)
Documentação de anti-patterns em desenvolvimento com IA.

**Quando consultar**:
- Durante code reviews
- Antes de gerar código com IA
- Ao configurar workflows de IA
- Para educar equipe sobre boas práticas

**Conteúdo principal**:
- 10 anti-patterns comuns em desenvolvimento com IA
- Exemplos de código problemático vs. correto
- Soluções e comandos de correção
- Checklist de qualidade pré-desenvolvimento

### [CONTEXT_HIERARCHY.md](./CONTEXT_HIERARCHY.md)
Hierarquia de precedência entre camadas de especificações.

**Quando consultar**:
- Quando specs conflitam
- Ao criar nova spec
- Durante validação de hierarquia
- Para entender precedência de decisões

**Conteúdo principal**:
- Definição de 4 camadas (Meta, Business, Technical, Execution)
- Regras de precedência e resolução de conflitos
- Exemplos de conflitos e resoluções
- Regras de referências entre camadas

---

## 🔄 Fluxo de Trabalho

### 1. Criando Nova Spec

Quando criar uma **nova** spec:

```markdown
---
spec_version: "1.0.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
---

# Título da Spec

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão desta spec
- [Descrição do que a spec cobre]
:::

[Conteúdo da spec...]
```

**Checklist**:
- [ ] Frontmatter completo com todos os campos obrigatórios
- [ ] Blocos `:::version_info` e `:::breaking_changes`
- [ ] Registrada em `VERSION_HISTORY.md`
- [ ] Referenciada no índice correspondente com versão

### 2. Atualizando Spec Existente

**Passo 1: Determinar tipo de mudança**

Use o fluxograma:

```
Altera significado de algo existente? → SIM → MAJOR
                                    → NÃO ↓
Remove algo obrigatório?           → SIM → MAJOR
                                    → NÃO ↓
Adiciona nova informação?          → SIM → MINOR
                                    → NÃO ↓
Apenas corrige/clarifica?          → SIM → PATCH
```

**Passo 2: Atualizar metadados**

```yaml
# Antes
spec_version: "1.2.3"
last_updated: "2025-12-20"
supersedes: "1.1.0"

# Depois (exemplo MINOR)
spec_version: "1.3.0"
last_updated: "2025-12-25"
supersedes: "1.2.3"
```

**Passo 3: Documentar mudança**

Se MAJOR, adicionar em `:::breaking_changes`:

```markdown
:::breaking_changes
**v2.0.0**:
- [Descrição da breaking change]
- [Impacto da mudança]

**v1.0.0** (baseline):
- Primeira versão versionada
:::
```

**Passo 4: Registrar no histórico**

Adicionar linha em `VERSION_HISTORY.md`:

```markdown
| Versão | Data | Tipo | Descrição |
|--------|------|------|-----------|
| 1.3.0  | 2025-12-25 | minor | Adicionada seção X |
| 1.2.3  | 2025-12-20 | patch | Corrigido typo |
| 1.0.0  | 2025-12-15 | baseline | Primeira versão |
```

**Passo 5: Atualizar índice**

Atualizar versão em `specs/business/index.md` ou `specs/technical/index.md`:

```markdown
- [Nome da Spec](SPEC.md) - v1.3.0
```

---

## 🚨 Princípio Jidoka (Stop-the-Line)

**Regra de Ouro**: Nenhuma spec pode ser usada como contexto sem versionamento adequado.

Se encontrar spec sem versionamento durante desenvolvimento:

```
🛑 PARE a execução
⚠️ ALERTE que a spec precisa ser versionada
📋 SUGIRA executar /governance:add-versioning
❌ NÃO prossiga até resolver
```

**Por quê?**

Specs não versionadas causam:
- **Context Drift**: Contexto desatualizado sem perceber
- **Regressão Semântica**: Significado de termos muda silenciosamente
- **Context Clash**: Versões incompatíveis misturadas
- **Perda de Rastreabilidade**: Não sabe quando mudou

---

## 📊 Comandos de Governança

### `/governance:add-versioning`
Adiciona versionamento semântico a specs existentes.

**Uso**:
```bash
/governance:add-versioning               # Versiona TODAS as specs
/governance:add-versioning business      # Apenas specs de negócio
/governance:add-versioning technical     # Apenas specs técnicas
```

### `/validation:audit-spec`
Valida conformidade de uma spec com a política de versionamento.

**Uso**:
```bash
/validation:audit-spec specs/business/CUSTOMER_PERSONAS.md
```

**Valida**:
- [ ] Frontmatter completo
- [ ] Blocos version_info e breaking_changes presentes
- [ ] SemVer válido
- [ ] Registrada em VERSION_HISTORY.md
- [ ] Referenciada no índice com versão

### `/validation:check-drift`
Detecta specs desatualizadas ou com context drift.

**Uso**:
```bash
/validation:check-drift                  # Verifica todas as specs
/validation:check-drift specs/business/  # Apenas business
```

**Detecta**:
- Specs não atualizadas há > 6 meses
- Specs sem blocos de versionamento
- Specs não registradas no VERSION_HISTORY.md
- Inconsistências entre specs relacionadas

---

## 📈 Métricas de Governança

**Healthcheck do Sistema de Contexto**:

| Métrica | Meta | Como Medir |
|---------|------|------------|
| **Cobertura de Versionamento** | 100% | % de specs com frontmatter completo |
| **Specs Atualizadas** | > 90% nos últimos 6 meses | Analisar `last_updated` |
| **Consistência de Índices** | 100% | Todas as specs listadas nos índices |
| **Auditabilidade** | 100% | Todas as specs em VERSION_HISTORY.md |

**Comandos para medir**:

```bash
# Cobertura de versionamento
grep -r "spec_version" specs --include="*.md" | wc -l

# Specs desatualizadas (últimos 6 meses)
/validation:check-drift

# Auditoria completa
/validation:audit-spec --all
```

---

## 🎓 Conceitos-Chave

### Context Drift
Quando o contexto se torna desatualizado em relação à realidade do sistema sem que ninguém perceba.

**Exemplo**:
```
2024-12: Spec diz "3 planos: FREE, PRO, MAX"
2025-03: Código adiciona plano ENTERPRISE
2025-06: IA ainda responde baseada em apenas 3 planos (DRIFT!)
```

**Solução**: Versionamento + auditoria periódica

### Regressão Semântica
Quando o significado de um termo muda ao longo do tempo sem breaking change explícito.

**Exemplo**:
```
v1.0: "limite" = valor máximo de operação
v2.0: "limite" = pré-aprovação de crédito (BREAKING!)
```

**Solução**: Documentar em `:::breaking_changes` e incrementar MAJOR

### Context Clash
Quando versões incompatíveis de contexto são misturadas na mesma interação.

**Exemplo**:
```
Prompt inclui:
- CUSTOMER_PERSONAS.md v1.0 (3 personas)
- PRODUCT_STRATEGY.md v2.0 (referencia 4 personas)

Resultado: IA confusa, respostas inconsistentes
```

**Solução**: Sempre usar versões compatíveis (mesma linha MAJOR)

---

## 🔍 Auditoria e Compliance

### Revisão Periódica

**Trimestral**:
- [ ] Executar `/validation:check-drift`
- [ ] Revisar specs críticas (PRODUCT_STRATEGY, CUSTOMER_PERSONAS)
- [ ] Atualizar métricas de governança

**Semestral**:
- [ ] Auditoria completa com `/validation:audit-spec --all`
- [ ] Revisar políticas de versionamento
- [ ] Consolidar specs relacionadas

**Anual**:
- [ ] Revisão completa de toda documentação
- [ ] Avaliar efetividade do sistema de governança
- [ ] Propor melhorias no processo

### Responsabilidades

| Papel | Responsabilidade |
|-------|------------------|
| **Product Owner** | Aprovar mudanças MAJOR em specs de negócio |
| **Tech Lead** | Aprovar mudanças MAJOR em specs técnicas |
| **Todos Devs** | Atualizar specs ao implementar features |
| **IA** | Alertar sobre drift detectado |

---

## 📚 Recursos Adicionais

### Documentação Relacionada

- [Política de Versionamento](./VERSIONING_POLICY.md) - Regras detalhadas
- [Histórico de Versões](./VERSION_HISTORY.md) - Registro completo
- [Índice Business](../business/index.md) - Specs de negócio
- [Índice Technical](../technical/index.md) - Specs técnicas

### Ferramentas

- Script Python: `add_versioning.py` - Adiciona versionamento automaticamente
- Script Python: `validate_frontmatter.py` - Valida frontmatter
- Comandos Claude: `/governance:*` - Comandos de governança
- Comandos Claude: `/validation:*` - Comandos de validação

### Referências Externas

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Architecture Decision Records](https://adr.github.io/)
- [Context-Driven AI Development](https://docs.anthropic.com/en/docs/build-with-claude/context-window)

---

## 💡 Boas Práticas

### ✅ DO

- **Versione todas as specs** desde o primeiro commit
- **Documente breaking changes** em `:::breaking_changes`
- **Atualize VERSION_HISTORY.md** sempre que versionar
- **Use SemVer rigorosamente** (MAJOR.MINOR.PATCH)
- **Revise specs trimestralmente** para detectar drift
- **Mantenha índices atualizados** com versões

### ❌ DON'T

- **Nunca** atualize spec sem incrementar versão
- **Nunca** faça breaking change sem MAJOR bump
- **Nunca** pule versões (1.0 → 1.2 sem 1.1)
- **Nunca** altere VERSION_HISTORY.md do passado (append-only)
- **Nunca** use specs não versionadas em prompts de IA
- **Nunca** misture versões MAJOR incompatíveis

---

## 🚀 Quick Start

**Para começar a usar o sistema de governança agora**:

1. **Leia** [VERSIONING_POLICY.md](./VERSIONING_POLICY.md) (5 min)
2. **Execute** `/governance:add-versioning` se specs não versionadas
3. **Valide** com `/validation:audit-spec --all`
4. **Configure** revisão trimestral no calendário
5. **Eduque** equipe sobre princípio Jidoka

---

## 📞 Suporte

**Dúvidas sobre governança de contexto?**

- Consulte [VERSIONING_POLICY.md](./VERSIONING_POLICY.md)
- Execute `/help governance`
- Revise exemplos em [VERSION_HISTORY.md](./VERSION_HISTORY.md)

---

## 📋 Status da Documentação

| Documento | Status | Última Atualização |
|-----------|--------|-------------------|
| README.md (index.md) | ✅ Atualizado | 2025-12-26 |
| VERSIONING_POLICY.md | ✅ Atualizado | 2025-12-25 |
| VERSION_HISTORY.md | ✅ Atualizado | 2025-12-25 |
| CONTEXT_HIERARCHY.md | ✅ Atualizado | 2025-12-25 |
| ANTI_PATTERNS.md | ✅ Atualizado | 2025-12-25 |
| DECISION_LOG.md | ✅ Atualizado | 2025-12-25 |
| GOVERNANCE_SUMMARY.md | ✅ Atualizado | 2025-12-25 |

---

**Última atualização**: 2025-12-26
**Versão do Sistema de Governança**: 1.0.0
**Responsável**: Context Governance Team
