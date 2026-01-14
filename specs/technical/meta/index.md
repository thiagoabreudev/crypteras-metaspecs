---
spec_version: "1.0.0"
valid_from: "2026-01-14"
last_updated: "2026-01-14"
supersedes: null
status: "active"
category: "technical"
tags: ['meta', 'context', 'ai', 'development']
---

# Metadados de Contexto para IA - Crypteras

:::version_info
**Versão**: 1.0.0
**Válida desde**: 2026-01-14
**Status**: Ativa
:::

:::breaking_changes
**v1.0.0** (baseline):
- Primeira versão do índice de metadados de contexto
- Consolida intent, stack e failure modes para desenvolvimento com IA
:::

## Sobre Este Diretório

Este diretório contém **metadados de contexto estruturados** que orientam o desenvolvimento assistido por IA no projeto Crypteras. Os arquivos aqui são fragmentos modulares de contexto, otimizados para serem carregados sob demanda.

---

## Arquivos Disponíveis

### [intent.md](intent.md)
**Propósito**: Define os objetivos, restrições e não-objetivos do desenvolvimento.

**Quando consultar**:
- Antes de iniciar qualquer tarefa de desenvolvimento
- Ao decidir escopo de uma feature
- Para validar se uma abordagem está alinhada com os princípios do projeto

**Conteúdo principal**:
- **Goals**: Objetivo principal do desenvolvimento com IA
- **Constraints**: Limites obrigatórios (ADRs, stack, padrões)
- **Non-Goals**: O que explicitamente NÃO fazer

---

### [stack.md](stack.md)
**Propósito**: Lista a stack tecnológica aprovada e decisões arquiteturais chave.

**Quando consultar**:
- Ao escolher tecnologia para nova feature
- Para verificar se uma biblioteca/framework é permitido
- Ao onboarding de novos desenvolvedores/agentes

**Conteúdo principal**:
- Backend: Python 3.11+, FastAPI, AGNO
- Frontend: Nuxt.js 3, Vue 3, TypeScript
- Database: MongoDB Atlas, Redis
- Mensageria: Celery + Redis Broker
- Deploy: Docker Swarm + Portainer
- ADRs chave referenciados

---

### [failures.md](failures.md)
**Propósito**: Documenta failure modes e anti-patterns conhecidos em desenvolvimento com IA.

**Quando consultar**:
- Antes de gerar código com IA
- Durante code reviews
- Ao diagnosticar bugs causados por IA
- Para educar equipe sobre boas práticas

**Conteúdo principal**:
- 15 failure modes documentados
- Tipos: hallucination, context_clash, integration, security, architecture
- Exemplos de código problemático vs. correto
- Comandos de detecção para cada failure mode

---

## Como Usar Estes Metadados

### Para Agentes de IA

1. **Carregar `intent.md`** no início de cada sessão de desenvolvimento
2. **Consultar `stack.md`** ao implementar features técnicas
3. **Verificar `failures.md`** antes de gerar código crítico (finanças, integração, segurança)

### Para Desenvolvedores Humanos

1. Use como checklist pré-code review
2. Compartilhe com novos membros da equipe
3. Atualize quando descobrir novos failure modes

---

## Relação com Outras Specs

| Este Diretório | Specs Relacionadas |
|----------------|-------------------|
| `intent.md` | [ADR-001](../adr/001-clean-architecture.md), [ADR-007](../adr/007-trading-config-deprecated.md) |
| `stack.md` | [ADR-006](../adr/006-agno-framework-ai.md), [CLAUDE.meta.md](../CLAUDE.meta.md) |
| `failures.md` | [ANTI_PATTERNS.md](../../_meta/ANTI_PATTERNS.md), [TROUBLESHOOTING.md](../TROUBLESHOOTING.md) |

---

## 📋 Status da Documentação

| Documento | Status | Última Atualização |
|-----------|--------|-------------------|
| index.md | ✅ Novo | 2026-01-14 |
| intent.md | ✅ Ativo | 2026-01-14 |
| stack.md | ✅ Ativo | 2026-01-14 |
| failures.md | ✅ Ativo | 2026-01-14 |

---

**Última Atualização**: 2026-01-14
**Versão**: 1.0.0
**Responsável**: Equipe de Desenvolvimento Crypteras
