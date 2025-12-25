# Adicionar Failure Modes às Metaspecs - Crypteras

Este comando documenta **Failure Modes** (modos de falha) conhecidos e suas mitigações nas especificações do **Crypteras Trading System**.

> **Nota**: Este é um comando avançado de governança de contexto. Para uso básico de documentação, use `/build-tech-docs` ou `/build-business-docs`.

## 🎯 Conceito: Failure Modes as Specs

**Failure Modes** são falhas previsíveis que devem ser documentadas e mitigadas explicitamente.

**Problema sem documentação de falhas**:
- Mesmos erros se repetem
- IA "alucina" em cenários conhecidos
- Context clash não documentado
- Mitigações não são aplicadas consistentemente

**Solução**: Documentar falhas conhecidas como especificações de primeira classe.

---

## 📋 Objetivo

Adicionar documentação de Failure Modes para:
- Prevenir erros conhecidos
- Mitigar alucinações da IA
- Documentar context clash scenarios
- Criar estratégias de recuperação

---

## 🔍 Processo

### 1. Identificar Specs que Precisam de Failure Modes

**Alta prioridade**:
- `CLAUDE.meta.md` - Falhas de desenvolvimento
- `API_SPECIFICATION.md` - Falhas de integração
- `BUSINESS_LOGIC.md` - Falhas de regras de negócio
- `TROUBLESHOOTING.md` - Falhas operacionais

**Média prioridade**:
- Features críticas (`features/authentication.md`, etc)
- ADRs com trade-offs conhecidos

### 2. Template de Failure Modes

Adicione seção `:::failure_modes` após `:::intent`:

```markdown
:::failure_modes
**Falhas Conhecidas**:

1. **[Nome da Falha]**
   - **Tipo**: [context_clash | hallucination | integration | validation | security]
   - **Descrição**: [O que acontece]
   - **Gatilho**: [Quando ocorre]
   - **Impacto**: 🔴 Crítico | 🟡 Médio | 🟢 Baixo
   - **Mitigação**: [Como prevenir/resolver]
   - **Detecção**: [Como identificar]

2. **[Próxima Falha]**
   ...
:::
```

### 3. Exemplo: CLAUDE.meta.md

```markdown
:::failure_modes
**Falhas Conhecidas de Desenvolvimento**:

1. **Reactive Loss em Props**
   - **Tipo**: hallucination
   - **Descrição**: IA usa `const goal = props.goal` perdendo reatividade
   - **Gatilho**: Manipulação direta de props sem `toRef` ou `toRefs`
   - **Impacto**: 🟡 Médio (bugs sutis de reatividade)
   - **Mitigação**: Sempre usar `toRef(props, 'propName')` ou `toRefs(props)`
   - **Detecção**: Props não atualizam quando pai muda valores

2. **MongoDB ObjectId vs String**
   - **Tipo**: integration
   - **Descrição**: IA compara ObjectId com String sem conversão
   - **Gatilho**: Comparações como `goal._id === userId`
   - **Impacto**: 🔴 Crítico (bugs de autorização)
   - **Mitigação**: Sempre usar `.toString()` ao comparar ObjectIds
   - **Detecção**: Queries retornam vazias ou autorização falha

3. **Computed vs Watch Confusion**
   - **Tipo**: hallucination
   - **Descrição**: IA usa `watch` para valores derivados (deveria ser `computed`)
   - **Gatilho**: Qualquer cálculo baseado em valores reativos
   - **Impacto**: 🟢 Baixo (código funciona mas não idiomático)
   - **Mitigação**: `computed` para valores derivados, `watch` para side effects
   - **Detecção**: Code review identifica padrão incorreto

4. **Context Clash: "limite"**
   - **Tipo**: context_clash
   - **Descrição**: Termo "limite" tem dois significados no domínio
     - v1: "limite contratual do convênio"
     - v2: "pré-aprovado por CPF"
   - **Gatilho**: Specs antigas e novas coexistindo
   - **Impacto**: 🔴 Crítico (decisões de negócio erradas)
   - **Mitigação**: Usar `spec_version` e preferir versão mais recente
   - **Detecção**: Validação contra `spec_version` obrigatória
:::
```

### 4. Exemplo: API_SPECIFICATION.md

```markdown
:::failure_modes
**Falhas Conhecidas de API**:

1. **JWT Expiration Not Checked**
   - **Tipo**: security
   - **Descrição**: IA esquece de validar expiração de JWT
   - **Gatilho**: Implementação de autenticação sem JwtAuthGuard
   - **Impacto**: 🔴 Crítico (sessões expiradas ainda válidas)
   - **Mitigação**: SEMPRE usar `@UseGuards(JwtAuthGuard)` em rotas protegidas
   - **Detecção**: Teste com token expirado deve falhar

2. **Pagination Missing**
   - **Tipo**: hallucination
   - **Descrição**: IA retorna todos os registros sem paginação
   - **Gatilho**: Endpoints `GET` que retornam listas
   - **Impacto**: 🟡 Médio (performance degrada com dados)
   - **Mitigação**: Endpoints de lista DEVEM ter `?page=&limit=`
   - **Detecção**: Query sem `.limit()` ou `.skip()`

3. **Error Without Status Code**
   - **Tipo**: hallucination
   - **Descrição**: IA lança erro genérico sem status HTTP correto
   - **Gatilho**: Validações de negócio
   - **Impacto**: 🟢 Baixo (cliente recebe 500 ao invés de 400)
   - **Mitigação**: Usar `NotFoundException`, `BadRequestException`, etc
   - **Detecção**: Logs mostram status 500 para erros de validação
:::
```

### 5. Exemplo: BUSINESS_LOGIC.md

```markdown
:::failure_modes
**Falhas Conhecidas de Lógica de Negócio**:

1. **Progress Calculation Rounding**
   - **Tipo**: hallucination
   - **Descrição**: IA calcula progresso sem arredondamento (0.6666666...)
   - **Gatilho**: Cálculo de porcentagem de ações completadas
   - **Impacto**: 🟢 Baixo (UI mostra muitas casas decimais)
   - **Mitigação**: `Math.round((completed / total) * 100)`
   - **Detecção**: UI mostra "66.66666666%"

2. **Status Overdue Not Updated**
   - **Tipo**: validation
   - **Descrição**: IA não atualiza status para "overdue" após deadline
   - **Gatilho**: Visualização de metas com deadline passado
   - **Impacto**: 🟡 Médio (usuário não vê alertas)
   - **Mitigação**: Computed property que compara `deadline < now`
   - **Detecção**: Meta com prazo passado mostra "in-progress"

3. **Empty Actions Array Division by Zero**
   - **Tipo**: validation
   - **Descrição**: IA divide por zero quando meta não tem ações
   - **Gatilho**: `const progress = completed / total` com `total === 0`
   - **Impacto**: 🔴 Crítico (NaN ou erro)
   - **Mitigação**: `if (total === 0) return 0` antes do cálculo
   - **Detecção**: Progress mostra "NaN%"
:::
```

### 6. Exemplo: Feature Spec (Authentication)

```markdown
:::failure_modes
**Falhas Conhecidas de Autenticação**:

1. **Password in Logs**
   - **Tipo**: security
   - **Descrição**: IA loga objeto com senha em plain text
   - **Gatilho**: Debug logging de `LoginDto` ou `RegisterDto`
   - **Impacto**: 🔴 Crítico (vazamento de senhas)
   - **Mitigação**: NUNCA logar DTOs com senha. Logar apenas `userId` e `email`
   - **Detecção**: Buscar `console.log` ou `logger.debug` com DTOs de auth

2. **Bcrypt Rounds Too Low**
   - **Tipo**: security
   - **Descrição**: IA usa bcrypt com rounds < 10
   - **Gatilho**: Hash de senha durante registro
   - **Impacto**: 🔴 Crítico (senhas fáceis de crackear)
   - **Mitigação**: `bcrypt.hash(password, 10)` ou mais rounds
   - **Detecção**: Código contém `bcrypt.hash(password, rounds < 10)`

3. **Token Without Expiration**
   - **Tipo**: security
   - **Descrição**: IA gera JWT sem `expiresIn`
   - **Gatilho**: Geração de token JWT
   - **Impacto**: 🔴 Crítico (tokens nunca expiram)
   - **Mitigação**: `jwt.sign(payload, secret, { expiresIn: '24h' })`
   - **Detecção**: Token decodificado não tem campo `exp`
:::
```

---

## ✅ Checklist de Execução

**Para cada spec técnica principal**:
- [ ] `CLAUDE.meta.md` - Falhas de desenvolvimento
- [ ] `API_SPECIFICATION.md` - Falhas de integração
- [ ] `BUSINESS_LOGIC.md` - Falhas de lógica de negócio
- [ ] `TROUBLESHOOTING.md` - Falhas operacionais

**Para features críticas**:
- [ ] `features/authentication.md` - Falhas de segurança
- [ ] `features/goal-management.md` - Falhas de validação
- [ ] Outras features com alta criticidade

**Para cada failure mode adicionado**:
- [ ] Nome claro e descritivo
- [ ] Tipo identificado (context_clash, hallucination, etc)
- [ ] Gatilho específico
- [ ] Impacto classificado (🔴🟡🟢)
- [ ] Mitigação acionável
- [ ] Detecção verificável
- [ ] Incrementar versão MINOR da spec
- [ ] Documentar mudança em `:::breaking_changes`

---

## 📊 Tipos de Failure Modes

### 1. Context Clash
Conflito entre versões de contexto ou significados.

**Exemplo**: Termo "limite" com múltiplos significados.

**Mitigação**: Versionamento semântico + preferir versão mais recente.

### 2. Hallucination
IA gera código incorreto mas plausível.

**Exemplo**: IA usa `watch` ao invés de `computed`.

**Mitigação**: Documentar padrões corretos + exemplos.

### 3. Integration
Falhas em integrações externas ou entre componentes.

**Exemplo**: ObjectId vs String comparison.

**Mitigação**: Documentar conversões obrigatórias.

### 4. Validation
Falhas em validações de dados ou regras de negócio.

**Exemplo**: Divisão por zero em cálculo de progresso.

**Mitigação**: Guard clauses obrigatórias.

### 5. Security
Falhas de segurança conhecidas.

**Exemplo**: Senha em logs, JWT sem expiração.

**Mitigação**: Security checklist obrigatório + code review.

---

## 🚨 Como IA Deve Usar Failure Modes

**Antes de gerar código**:

1. **Ler failure modes da spec relevante**
2. **Validar se código proposto não dispara falhas conhecidas**
3. **Aplicar mitigações preventivamente**

**Exemplo**:

```
IA lendo CLAUDE.meta.md...
Failure Mode detectado: "Reactive Loss em Props"

Código proposto:
  const goal = props.goal  ❌

Mitigação aplicada:
  const goal = toRef(props, 'goal')  ✅
```

**Durante execução**:

1. **Monitorar sinais de detecção**
2. **Alertar se falha conhecida for detectada**
3. **Sugerir mitigação documentada**

---

## 🔄 Manutenção de Failure Modes

**Quando adicionar nova falha**:

1. **Falha ocorreu em produção/desenvolvimento**
2. **Documentar imediatamente no failure mode**
3. **Adicionar mitigação e detecção**
4. **Incrementar versão MINOR da spec**

**Quando remover falha**:

1. **Mitigação foi implementada em nível de código/arquitetura**
2. **Falha não pode mais ocorrer**
3. **Mover para seção "Falhas Resolvidas" (histórico)**
4. **Incrementar versão MINOR da spec**

---

## 📚 Template de Failure Mode Resolvido

```markdown
:::resolved_failures
**Falhas Resolvidas** (não ocorrem mais):

1. **[Nome da Falha]** - Resolvido em v1.2.0
   - **Resolução**: [Como foi resolvido]
   - **Data**: 2025-12-20
   - **Histórico**: [Link para PR ou commit]
:::
```

---

**Argumentos**:
<arguments>
#$ARGUMENTS
</arguments>

Se nenhum argumento for fornecido, adicione Failure Modes a TODAS as specs técnicas principais.

Se argumentos forem fornecidos (ex: `CLAUDE.meta.md` ou `features/authentication.md`), adicione apenas às specs especificadas.

**IMPORTANTE**: Peça ao usuário exemplos de falhas conhecidas antes de documentar. Não invente failure modes - documente apenas falhas reais ou muito prováveis baseadas no contexto do projeto.
