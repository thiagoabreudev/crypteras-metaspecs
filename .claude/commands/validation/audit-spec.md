# Auditar Spec - Crypteras

Este comando realiza auditoria completa de uma especificação do **Crypteras Trading System**, verificando qualidade, completude e alinhamento.

> **Nota**: Este é um comando avançado de validação. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Objetivo

Auditar spec para garantir:
- Qualidade da documentação
- Completude de informações
- Alinhamento com padrões
- Usabilidade pela IA

---

## 📋 Dimensões de Auditoria

### 1. Estrutura e Formato

**Verificar elementos obrigatórios**:

```markdown
## ✅ Auditoria de Estrutura

**Frontmatter YAML**:
- [ ] spec_version presente e SemVer válido
- [ ] valid_from presente (formato YYYY-MM-DD)
- [ ] last_updated presente e atual
- [ ] supersedes correto (null ou versão anterior)
- [ ] status válido (active/deprecated/draft)

**Seções Obrigatórias**:
- [ ] :::version_info presente
- [ ] :::breaking_changes documentado
- [ ] Título claro (H1)
- [ ] Descrição/propósito no início

**Seções Recomendadas** (se spec é crítica):
- [ ] :::intent (goal + constraints + non-goals)
- [ ] :::failure_modes (falhas conhecidas)
- [ ] :::explainability (requirements)

**Formatação**:
- [ ] Markdown válido
- [ ] Links internos funcionam
- [ ] Código tem syntax highlighting
- [ ] Tabelas bem formatadas
```

**Exemplo de saída**:
```markdown
### Estrutura: ⚠️ 7/10 critérios atendidos

✅ Frontmatter completo
✅ Versionamento presente
✅ Título e descrição claros
✅ Markdown válido
❌ Falta :::intent (recomendado para spec crítica)
❌ Falta :::failure_modes
⚠️ Link quebrado: `[CODEBASE_GUIDE.md](../CODEBASE_GUIDE.md)` (arquivo não existe)
```

### 2. Qualidade do Conteúdo

**Verificar clareza e precisão**:

```markdown
## Qualidade do Conteúdo

**Clareza**:
- [ ] Linguagem clara e objetiva
- [ ] Sem ambiguidades
- [ ] Termos técnicos definidos
- [ ] Exemplos concretos fornecidos

**Precisão**:
- [ ] Informações verificáveis
- [ ] Exemplos de código funcionam
- [ ] Referências corretas
- [ ] Versões de libs especificadas

**Completude**:
- [ ] Cobre escopo completo do tópico
- [ ] Edge cases documentados
- [ ] Trade-offs explicados
- [ ] Limitações conhecidas documentadas
```

**Exemplo de saída**:
```markdown
### Qualidade: ✅ 9/12 critérios atendidos

✅ Linguagem clara
✅ Exemplos de código presentes
✅ Referências corretas
❌ Termo "limite" usado sem definição clara
❌ Exemplo de código não foi testado (sintaxe incorreta):
   ```ts
   // Linha 45: erro de sintaxe
   const goal = toRef(props 'goal')  // falta vírgula
   ```
⚠️ Falta documentar edge case: "O que acontece se meta não tem ações?"
```

### 3. Alinhamento com Padrões

**Verificar conformidade com convenções do projeto**:

```markdown
## Alinhamento com Padrões

**Convenções de Nomenclatura**:
- [ ] Nomes de arquivos seguem padrão (UPPERCASE.md)
- [ ] Nomes de seções consistentes
- [ ] Termos técnicos padronizados

**Intent as Code** (se aplicável):
- [ ] Goal específico e mensurável
- [ ] Constraints verificáveis
- [ ] Non-goals explícitos

**Failure Modes** (se aplicável):
- [ ] Tipo identificado (context_clash, hallucination, etc)
- [ ] Gatilho descrito
- [ ] Impacto classificado (🔴🟡🟢)
- [ ] Mitigação acionável
- [ ] Detecção verificável

**Explainability** (se aplicável):
- [ ] Requirement level definido
- [ ] Output format especificado
- [ ] Gatilhos documentados
```

**Exemplo de saída**:
```markdown
### Alinhamento: ⚠️ 6/9 critérios atendidos

✅ Nome de arquivo correto (CLAUDE.meta.md)
✅ Seções padronizadas
✅ Intent presente e bem escrito
❌ Failure mode "Reactive Loss" sem campo "Detecção"
❌ Explainability: Falta documentar "gatilhos" de quando explicar
⚠️ Non-goal vago: "Código ruim" (deveria ser específico)
```

### 4. Usabilidade pela IA

**Verificar se IA consegue usar a spec efetivamente**:

```markdown
## Usabilidade pela IA

**Estrutura Acessível**:
- [ ] Seções bem delimitadas (:::blocks ou headings)
- [ ] Hierarquia clara (H1 > H2 > H3)
- [ ] Informação não enterrada em parágrafos longos

**Informação Acionável**:
- [ ] Instruções específicas (não vagas)
- [ ] Exemplos de código copiáveis
- [ ] Checklists executáveis
- [ ] Decisões binárias (sim/não, não talvez)

**Rastreabilidade**:
- [ ] Versão clara (frontmatter)
- [ ] Referências a outras specs (com versão)
- [ ] Histórico de mudanças documentado
```

**Exemplo de saída**:
```markdown
### Usabilidade: ✅ 8/9 critérios atendidos

✅ Estrutura clara com :::blocks
✅ Hierarquia bem definida
✅ Exemplos de código copiáveis
✅ Checklists práticos
❌ Instrução vaga: "Código deve ser bom" (não acionável)
💡 Sugestão: Especificar "Type coverage > 90%" ao invés de "boa tipagem"
```

### 5. Consistência Interna

**Verificar que spec não se contradiz**:

```markdown
## Consistência Interna

**Verificações**:
- [ ] Goal e Constraints não conflitam
- [ ] Exemplos seguem constraints documentados
- [ ] Failure modes têm mitigações correspondentes
- [ ] Versão em frontmatter = versão em :::version_info
- [ ] Breaking changes listam todas mudanças MAJOR

**Exemplo de inconsistência**:
❌ Constraint diz "TypeScript strict mode obrigatório"
❌ Exemplo de código tem `any` sem `@ts-ignore`
```

**Exemplo de saída**:
```markdown
### Consistência: ⚠️ 4/5 critérios atendidos

✅ Goal e Constraints alinhados
✅ Versões consistentes (frontmatter = version_info)
✅ Breaking changes completos
❌ Contradição detectada:
   - Intent: "Não usar Options API"
   - Exemplo (linha 120): Mostra código com `export default { data() {...} }`
⚠️ Failure mode "MongoDB ObjectId" sem mitigação correspondente
```

### 6. Alinhamento com Outras Specs

**Verificar compatibilidade com specs relacionadas**:

```markdown
## Alinhamento com Outras Specs

**Cross-references**:
- [ ] Specs referenciadas existem
- [ ] Versões referenciadas são válidas
- [ ] Não há conflitos com outras specs

**Hierarquia**:
- [ ] Spec respeita precedência (Meta > Business > Technical)
- [ ] Não viola constraints de specs superiores

**Exemplo de conflito**:
🔴 Conflict Detected:
- Esta spec (CLAUDE.meta.md): "Usar MongoDB"
- Spec superior (index.md): Stack define "PostgreSQL"
→ Violação de hierarquia
```

**Exemplo de saída**:
```markdown
### Alinhamento Externo: ✅ 5/5 critérios atendidos

✅ Todas as referências válidas
✅ Versões citadas existem
✅ Respeita hierarquia (não viola specs superiores)
✅ Sem conflitos com specs do mesmo nível
✅ Cross-references bem documentados
```

---

## 📊 Formato de Saída Completo

```markdown
# 🔍 Auditoria de Spec

**Spec**: specs/technical/CLAUDE.meta.md
**Versão**: 1.2.0
**Data da Auditoria**: 2025-12-20T16:00:00Z

---

## 1. Estrutura e Formato: ⚠️ 7/10

✅ Frontmatter completo
✅ Versionamento presente
...
❌ Falta :::intent
❌ Falta :::failure_modes

**Score**: 70%

---

## 2. Qualidade do Conteúdo: ✅ 9/12

✅ Linguagem clara
✅ Exemplos presentes
...
❌ Termo "limite" ambíguo
❌ Código com erro de sintaxe (linha 45)

**Score**: 75%

---

## 3. Alinhamento com Padrões: ⚠️ 6/9

✅ Nome de arquivo correto
...
❌ Failure mode sem "Detecção"

**Score**: 67%

---

## 4. Usabilidade pela IA: ✅ 8/9

✅ Estrutura clara
...
❌ Instrução vaga

**Score**: 89%

---

## 5. Consistência Interna: ⚠️ 4/5

✅ Goal e Constraints alinhados
...
❌ Contradição em exemplo de código

**Score**: 80%

---

## 6. Alinhamento Externo: ✅ 5/5

✅ Sem conflitos
✅ Hierarquia respeitada

**Score**: 100%

---

## 📋 Resumo da Auditoria

| Dimensão | Score | Status |
|----------|-------|--------|
| Estrutura | 70% | ⚠️ |
| Qualidade | 75% | ⚠️ |
| Padrões | 67% | ⚠️ |
| Usabilidade | 89% | ✅ |
| Consistência | 80% | ⚠️ |
| Alinhamento | 100% | ✅ |

**Score Geral**: 80% ⚠️ BOM (melhorias recomendadas)

**Classificação**:
- 90-100%: ✅ Excelente
- 75-89%: ⚠️ Bom (melhorias recomendadas)
- 60-74%: 🟡 Regular (melhorias necessárias)
- < 60%: 🔴 Crítico (requer revisão completa)

---

## 🛠️ Ações Recomendadas (Prioridade)

### 🔴 Críticas (Impactam usabilidade)
1. **Corrigir erro de sintaxe** (linha 45)
2. **Resolver contradição** em exemplo de Options API

### 🟡 Importantes (Melhoram qualidade)
3. **Adicionar :::intent** (spec crítica deveria ter)
4. **Adicionar :::failure_modes**
5. **Clarificar termo "limite"** (definição explícita)

### 🟢 Sugestões (Aperfeiçoamento)
6. Documentar edge case de meta sem ações
7. Especificar "Detecção" em failure mode "Reactive Loss"
8. Tornar non-goals mais específicos

---

## 📈 Evolução Recomendada

**Próxima versão sugerida**: 1.3.0 (MINOR)

**Mudanças propostas**:
- Adicionar :::intent (nova seção)
- Adicionar :::failure_modes (nova seção)
- Corrigir contradições (patch)
- Clarificar termos ambíguos (patch)

**Versão futura**: 2.0.0 (se houver breaking change)
- Se "limite" mudar de significado
- Se remover seção obrigatória
- Se alterar structure incompativelmente

---

**Auditoria aprovada?** ⚠️ CONDICIONAL
- Spec é utilizável mas requer melhorias
- Corrigir críticos antes de próxima versão
- Considerar melhorias importantes
```

---

## ✅ Checklist de Auditoria

Executar TODAS as verificações:

- [ ] **Estrutura**: Frontmatter, seções, formatação
- [ ] **Qualidade**: Clareza, precisão, completude
- [ ] **Padrões**: Nomenclatura, Intent, Failure Modes, Explainability
- [ ] **Usabilidade**: Estrutura, informação acionável, rastreabilidade
- [ ] **Consistência**: Sem contradições internas
- [ ] **Alinhamento**: Compatibilidade com outras specs

---

## 🔄 Uso Recomendado

**Quando auditar**:

1. **Antes de publicar nova spec**
2. **Após atualização MAJOR**
3. **Periodicamente** (trimestral para specs críticas)
4. **Quando drift é detectado**

**Integração com workflow**:
```markdown
# Exemplo: Auditoria automática em PR

1. Desenvolvedor atualiza CLAUDE.meta.md
2. CI executa: /audit-spec CLAUDE.meta.md
3. Se score < 75%: PR bloqueado
4. Se score >= 75%: PR aprovado (warnings documentados)
```

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Argumento obrigatório: caminho da spec a auditar.

**Exemplo**: `/audit-spec specs/technical/CLAUDE.meta.md`

Flags opcionais:
- `--strict`: Score mínimo 90% (ao invés de 75%)
- `--quick`: Pula verificações não-críticas
- `--fix`: Sugerir correções automáticas (quando possível)
