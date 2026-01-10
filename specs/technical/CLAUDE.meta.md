---
spec_version: "1.8.0"
valid_from: "2026-01-10"
last_updated: "2026-01-10"
supersedes: "1.7.0"
status: "active"
category: "technical"
tags: ['technical', 'claude.meta', 'infrastructure', 'debug', 'modular']
---

# CLAUDE.meta.md - Guia de Desenvolvimento com IA

:::version_info
**Versão**: 1.8.0
**Válida desde**: 2026-01-10
**Status**: Ativa
:::

:::breaking_changes
**v1.8.0** (2026-01-10):
- **Modularização**: O arquivo foi dividido em módulos para otimização de contexto (200k tokens).
- Módulos disponíveis em `specs/technical/meta/`:
  - `intent.md`: Objetivos e Constraints
  - `stack.md`: Stack tecnológica
  - `failures.md`: Failure Modes conhecidos
- Comandos devem carregar apenas os módulos necessários.
:::

:::modules
Este arquivo é um índice. Para carregar o contexto completo, inclua:
- `specs/technical/meta/intent.md`
- `specs/technical/meta/stack.md`
- `specs/technical/meta/failures.md`
:::

:::debug_checklist
**Checklist de Debug em Produção**:
1. **Verificar Variáveis de Ambiente**:
   `docker service inspect crypteras_agno --format='{{json .Spec.TaskTemplate.ContainerSpec.Env}}'`
2. **Verificar Logs**:
   `docker service logs crypteras_agno --tail 100`
3. **Verificar Conectividade MongoDB**:
   Não assuma localhost. Use a URI do Atlas definida no env.
:::
