---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'backoffice', 'admin']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Backoffice / Admin Panel

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Fornecer painel administrativo para suporte técnico gerenciar usuários, bots e subscriptions.

**Constraints** (limites obrigatórios):
- Acesso restrito a admin users apenas (role-based access control)
- Audit log de TODAS ações de admin (quem fez o quê, quando)
- Nunca expor credenciais de exchange mesmo para admins
- Permitir pausar/reativar bots de usuários (troubleshooting)
- Visualizar stats agregados (total users, MRR, churn rate)
- Suportar busca de usuários por email, ID ou subscription status

**Non-Goals** (o que NÃO fazer):
- Criar sistema de permissões granulares (apenas admin on/off)
- Implementar impersonation de usuários (security risk)
- Permitir admins modificarem código de bots em produção
- Adicionar analytics avançados tipo BI (usar ferramenta externa)
- Criar workflow de aprovação multi-stage
- Expor backoffice publicamente (apenas VPN ou IP whitelist)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Painel administrativo e ferramentas de backoffice
:::

## Visão Geral

Sistema administrativo completo para gestão da plataforma Crypteras SaaS. Permite que equipe interna monitore usuários, assinaturas, bots e operações sem acesso direto ao banco de dados.

**Status**: ✅ Em Produção
**Acesso**: Apenas equipe Crypteras (não disponível para usuários finais)
**Prioridade**: CRÍTICA - Operação e suporte

---

## Propósito e Valor

### Para a Equipe Crypteras
- **Visibilidade Total**: Acesso a todos os usuários e suas atividades
- **Suporte Eficiente**: Resolve problemas de usuários rapidamente sem SQL manual
- **Monitoramento**: Identifica padrões de uso e problemas sistêmicos
- **Segurança**: Acesso auditado e controlado (não é acesso root ao banco)

### Para o Negócio
- **Redução de Downtime**: Equipe resolve problemas 3x mais rápido
- **Qualidade de Suporte**: Visão completa do contexto do usuário
- **Compliance**: Auditoria de acessos administrativos
- **Escalabilidade**: Onboarding de novos membros do suporte sem acesso ao servidor

---

## Funcionalidades Principais

### 1. Autenticação de Admin

**Diferença para autenticação de usuário**:
```
Usuário Normal:
- JWT no header Authorization
- Token de 1 hora
- Refresh token de 7 dias

Admin:
- JWT em cookie HttpOnly (proteção CSRF)
- Token de 8 horas
- Sem refresh token (requer re-login)
- Entidade separada (BoUser vs UserEntity)
```

**Login**:
```
POST /api/backoffice/auth/login

Body:
{
  "email": "admin@crypteras.tech",
  "password": "SecureAdminP@ssw0rd"
}

Response:
{
  "message": "Login successful",
  "user": {
    "email": "admin@crypteras.tech",
    "role": "admin"
  }
}

Cookie criado:
bo_access_token=eyJhbGci... (HttpOnly, Secure, SameSite=Strict)
```

**Regras de Negócio**:
- ✅ Email deve terminar com `@crypteras.tech` (validação de domínio)
- ✅ Senha hasheada com bcrypt (custo 12, igual usuários)
- ✅ Cookie HttpOnly impede acesso via JavaScript (anti-XSS)
- ✅ Cookie Secure apenas em HTTPS (produção)
- ✅ SameSite=Strict previne CSRF
- ✅ Token expira em 8 horas (sem renovação automática)

---

### 2. Listagem de Usuários

**Endpoint**: `GET /api/backoffice/users`

**Query Parameters**:
```
page: número da página (default: 1)
page_size: itens por página (default: 50, max: 100)
sort_by: campo de ordenação (created_at, email, name)
sort_order: asc ou desc (default: desc)
```

**Exemplo**:
```
GET /api/backoffice/users?page=1&page_size=50&sort_by=created_at&sort_order=desc

Response:
{
  "users": [
    {
      "user_id": "65f8a1...",
      "email": "joao@example.com",
      "full_name": "João Silva",
      "created_at": "2025-01-15T10:00:00Z",
      "last_login": "2025-01-24T14:30:00Z",
      "subscription": {
        "plan": "PRO",
        "status": "ACTIVE",
        "current_period_end": "2025-02-15T10:00:00Z"
      },
      "has_mb_credentials": true,
      "has_binance_credentials": false,
      "total_bots": 5,
      "active_bots": 3
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total_items": 245,
    "total_pages": 5
  }
}
```

**Dados Exibidos**:
- ✅ Informações básicas (email, nome, data de criação)
- ✅ Último login (identificar usuários inativos)
- ✅ Status da assinatura (plan, status, período)
- ✅ Credenciais configuradas (boolean, não exibe keys)
- ✅ Quantidade de bots (total e ativos)
- ❌ **NUNCA** exibe senhas, API keys ou tokens

---

### 3. Detalhes de Usuário Individual

**Endpoint**: `GET /api/backoffice/users/{user_id}`

**Exemplo**:
```
GET /api/backoffice/users/65f8a1...

Response:
{
  "user": {
    "user_id": "65f8a1...",
    "email": "joao@example.com",
    "full_name": "João Silva",
    "created_at": "2025-01-15T10:00:00Z",
    "last_login": "2025-01-24T14:30:00Z",

    "subscription": {
      "plan": "PRO",
      "status": "ACTIVE",
      "payment_provider": "mercadopago",
      "external_customer_id": "cus_abc123",
      "current_period_start": "2025-01-15T10:00:00Z",
      "current_period_end": "2025-02-15T10:00:00Z",
      "cancel_at_period_end": false
    },

    "credentials": {
      "mercado_bitcoin": {
        "configured": true,
        "last_4_digits": "****5678",
        "configured_at": "2025-01-15T11:00:00Z"
      },
      "binance": {
        "configured": false
      }
    },

    "bots": {
      "smart_bots": 3,
      "candle_bots": 2,
      "dca_bots": 0,
      "active": 3,
      "paused": 2,
      "archived": 0
    },

    "activity": {
      "total_orders": 245,
      "total_invested": "5420.00",
      "realized_pnl": "+420.00",
      "last_order_at": "2025-01-24T12:00:00Z"
    },

    "legal_consents": {
      "terms_accepted": true,
      "privacy_accepted": true,
      "marketing_accepted": false,
      "timestamp": "2025-01-15T10:00:00Z",
      "ip_address": "192.168.1.1"
    }
  }
}
```

**Uso Principal**:
- ✅ **Suporte**: Ver contexto completo do usuário antes de responder ticket
- ✅ **Troubleshooting**: Identificar rapidamente se problema é de config, assinatura ou API
- ✅ **Análise**: Entender padrões de uso e performance

---

### 4. Gestão de Usuários (Ações Administrativas)

**Funcionalidades Disponíveis** (futuro - planejado):

```
PUT /api/backoffice/users/{user_id}/block
→ Bloqueia usuário (impede login)
→ Pausa TODOS os bots automaticamente
→ Uso: Fraude, abuso, violação de termos

PUT /api/backoffice/users/{user_id}/unblock
→ Desbloqueia usuário

PUT /api/backoffice/users/{user_id}/subscription
→ Ajusta assinatura manualmente
→ Uso: Cortesia, compensação por problemas

POST /api/backoffice/users/{user_id}/reset-password
→ Envia email de reset de senha
→ Uso: Usuário perdeu acesso

GET /api/backoffice/users/{user_id}/activity-log
→ Histórico completo de ações
→ Uso: Investigação de problemas, auditoria
```

---

## Regras de Negócio Principais

### 1. Controle de Acesso (RBAC - Role-Based Access Control)

**Roles Disponíveis**:
```
SUPER_ADMIN:
  - Todos os acessos
  - Criar/deletar outros admins
  - Alterar configurações globais

ADMIN:
  - Ver usuários
  - Ver detalhes de usuários
  - Executar ações administrativas
  - NÃO pode criar outros admins

SUPPORT:
  - Ver usuários
  - Ver detalhes de usuários
  - NÃO pode executar ações administrativas
  - Apenas leitura
```

**Regra de Domínio**:
```python
def is_authorized_admin(email: str) -> bool:
    # Apenas emails @crypteras.tech
    return email.endswith("@crypteras.tech")

def get_admin_role(email: str) -> str:
    if email == "admin@crypteras.tech":
        return "SUPER_ADMIN"
    elif email.endswith("@crypteras.tech"):
        return "ADMIN"  # Default para domínio
    else:
        raise Unauthorized("Domínio não autorizado")
```

---

### 2. Auditoria de Acessos

**Toda ação administrativa é logada**:
```python
{
  "admin_email": "admin@crypteras.tech",
  "action": "VIEW_USER_DETAILS",
  "target_user_id": "65f8a1...",
  "timestamp": "2025-01-24T15:00:00Z",
  "ip_address": "10.0.0.5",
  "user_agent": "Mozilla/5.0..."
}
```

**Ações Auditadas**:
- ✅ Login/Logout de admin
- ✅ Visualização de lista de usuários
- ✅ Visualização de detalhes de usuário específico
- ✅ Ações administrativas (block, unblock, reset password)
- ✅ Alterações de configuração global

**Retenção**: Logs mantidos por 1 ano (compliance)

---

### 3. Paginação e Performance

**Limites de Proteção**:
```
page_size máximo: 100 usuários
Timeout de query: 30 segundos
Cache de listagens: 5 minutos (Redis)

Se > 10.000 usuários:
  → Adiciona índices MongoDB
  → Paginação via cursor (não offset)
```

**Otimizações**:
- Projection (retorna apenas campos necessários)
- Índices compostos: `{created_at: -1, email: 1}`
- Cache de contagens (`total_items`)

---

### 4. Proteção de Dados Sensíveis

**O que NUNCA é exibido no Backoffice**:
```
❌ Senhas (nem hash)
❌ API keys completas (apenas últimos 4 dígitos)
❌ Refresh tokens
❌ Chaves de criptografia
❌ Segredos de webhook
❌ Números de cartão de crédito (gerenciados por Stripe/MP)
```

**O que É exibido (seguro)**:
```
✅ Email, nome, data de criação
✅ Status de assinatura (plan, status, período)
✅ Credenciais configuradas (boolean + últimos 4 dígitos)
✅ Quantidade de bots e ordens
✅ P&L agregado
✅ Consentimentos legais (timestamp, IP)
```

---

### 5. Rate Limiting

**Proteção contra abuso**:
```
Login: 5 tentativas / 15 minutos (por IP)
Listagem: 60 requests / minuto (por admin)
Detalhes: 100 requests / minuto (por admin)
Ações: 10 requests / minuto (por admin)

Se exceder:
  → HTTP 429 Too Many Requests
  → Retry-After header com tempo de espera
```

---

## Casos de Uso Reais

### Caso 1: Suporte - Usuário Reclama "Bot Não Funciona"

**Fluxo**:
```
1. Ticket: "Meu bot não está comprando há 2 dias"

2. Admin acessa Backoffice → Busca por email

3. Vê detalhes:
   - Credenciais MB: ✅ Configuradas
   - Última ordem: 2 dias atrás (confirma reclamação)
   - Bot Status: PAUSED (!!!!)
   - Circuit Breaker: Ativado (!!!)

4. Admin identifica problema:
   → Bot pausou automaticamente (circuit breaker)
   → Motivo: 5 falhas consecutivas

5. Admin responde ticket:
   "Seu bot pausou automaticamente após 5 falhas consecutivas.
    Possível causa: Saldo insuficiente ou problema temporário na exchange.

    Solução:
    1. Verifique saldo no Mercado Bitcoin
    2. Vá em Bots → Reset Circuit Breaker
    3. Retome o bot

    Se problema persistir, responda este ticket."

Tempo de resolução: 2 minutos (vs 30 min sem backoffice)
```

---

### Caso 2: Operações - Identificar Padrão de Falhas

**Fluxo**:
```
1. Admin nota: 15 tickets similares em 1 dia
   → "Bot não executa ordens"

2. Admin acessa Backoffice → Lista usuários

3. Ordena por last_login (mais recentes)

4. Filtra mentalmente: Vê que TODOS os afetados:
   - Usam Binance
   - Criaram credenciais nas últimas 24h
   - Nenhuma ordem executada

5. Admin investiga:
   → Testa próprias credenciais Binance
   → Descobre: Binance mudou formato de API key

6. Admin:
   - Cria alerta interno
   - Atualiza documentação
   - Envia email proativo para usuários Binance
   - Prepara patch de código

Resultado: Problema resolvido proativamente para 200+ usuários
```

---

### Caso 3: Fraude - Usuário Suspeito

**Fluxo**:
```
1. Sistema detecta: 50 contas criadas com emails similares
   → joao1@temp.com, joao2@temp.com, ..., joao50@temp.com

2. Admin acessa Backoffice:
   - Lista usuários por created_at recente
   - Vê padrão suspeito (emails temporários, mesmo IP)

3. Admin verifica:
   - Todas em plano FREE (abusando trial)
   - Nenhuma com credenciais reais (apenas paper trading)
   - Mesmo User-Agent

4. Admin executa:
   PUT /api/backoffice/users/bulk-block
   Body: { user_ids: [...], reason: "Fraude - contas temporárias" }

5. Sistema:
   - Bloqueia 50 contas
   - Previne novos registros desse IP (24h)
   - Alerta equipe de segurança

Tempo de resposta: 5 minutos (vs horas sem backoffice)
```

---

## Interface do Backoffice (Planejado)

### Dashboard Principal
```
╔════════════════════════════════════════════════════════╗
║  CRYPTERAS BACKOFFICE                                  ║
╠════════════════════════════════════════════════════════╣
║  📊 MÉTRICAS                                           ║
║  ├─ Usuários Totais: 1.245                             ║
║  ├─ Ativos (7 dias): 842 (68%)                         ║
║  ├─ Assinaturas PRO: 245 (20%)                         ║
║  └─ Assinaturas MAX: 42 (3,4%)                         ║
╠════════════════════════════════════════════════════════╣
║  🔔 ALERTAS                                            ║
║  ├─ 15 usuários com circuit breaker ativo (últimas 24h)║
║  ├─ 8 assinaturas expirando em 3 dias                  ║
║  └─ 3 usuários com credenciais inválidas               ║
╠════════════════════════════════════════════════════════╣
║  🔍 BUSCAR USUÁRIO                                     ║
║  [Email ou User ID: _____________________] [Buscar]    ║
╚════════════════════════════════════════════════════════╝
```

### Lista de Usuários
```
╔════════════════════════════════════════════════════════════════════╗
║  USUÁRIOS (Página 1 de 25)                                         ║
╠════════════════════════════════════════════════════════════════════╣
║  Email              │ Plano │ Status   │ Bots │ Último Login      ║
╠════════════════════════════════════════════════════════════════════╣
║  joao@example.com   │ PRO   │ ✅ Ativo │ 5    │ Hoje, 14:32       ║
║  maria@example.com  │ MAX   │ ✅ Ativo │ 12   │ Hoje, 10:15       ║
║  pedro@example.com  │ FREE  │ ⚠️ Trial │ 1    │ Ontem, 18:20      ║
║  ana@example.com    │ PRO   │ ❌ Expirou│ 3    │ 5 dias atrás      ║
╠════════════════════════════════════════════════════════════════════╣
║  [Anterior] [1] [2] [3] ... [25] [Próxima]                        ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Métricas de Sucesso do Backoffice

### KPIs Operacionais
- **Tempo Médio de Resolução de Ticket**: Meta < 5 minutos (com backoffice vs 20 min sem)
- **Taxa de Auto-resolução**: % de usuários que resolvem problemas sem abrir ticket (após melhorias identificadas via backoffice)
- **Incidentes Proativos**: Quantidade de problemas resolvidos antes de virar ticket

### KPIs de Segurança
- **Tempo de Detecção de Fraude**: Meta < 1 hora
- **Contas Bloqueadas por Fraude**: Quantidade (tracking de evolução)
- **Acesso Não Autorizado**: Tentativas de login admin fora do domínio @crypteras.tech

---

## Problemas Comuns e Soluções

### ❌ "Cookie não está sendo criado"
**Causa**: Frontend em domínio diferente do backend (CORS)
**Solução**: Configurar CORS com `credentials: true` e `Access-Control-Allow-Credentials: true`

### ❌ "Token expira muito rápido"
**Causa**: Token de 8h sem refresh (design proposital)
**Solução**: Re-login é intencional (segurança). Considerar aumentar para 12h se muito inconveniente.

### ❌ "Listagem muito lenta com 10k+ usuários"
**Causa**: Offset pagination em grandes datasets
**Solução**: Migrar para cursor-based pagination

---

## Roadmap Futuro

### Melhorias Planejadas (Q1-Q2 2025)
- [ ] **Dashboard com métricas visuais**: Gráficos de crescimento, churn, MRR
- [ ] **Filtros avançados**: Por plano, status, bots ativos, P&L
- [ ] **Exportação de dados**: CSV de usuários para análise
- [ ] **Ações em massa**: Block/unblock múltiplos usuários
- [ ] **Logs de atividade por usuário**: Timeline completa de ações
- [ ] **Notificações proativas**: Alerta quando usuário com problema similar abre ticket
- [ ] **Integração com Intercom/Zendesk**: Contexto do backoffice no chat de suporte

### Visão de Longo Prazo
- [ ] **Machine Learning**: Detecção automática de padrões de fraude
- [ ] **Self-service para admins**: Criar/editar FAQs, documentação
- [ ] **A/B testing**: Testar mudanças em grupos de usuários
- [ ] **Feature flags**: Habilitar/desabilitar features por usuário/grupo

---

## Segurança e Compliance

### Proteções Implementadas
- ✅ **Autenticação separada**: BoUser vs UserEntity
- ✅ **Cookie HttpOnly**: Previne XSS
- ✅ **SameSite=Strict**: Previne CSRF
- ✅ **Rate limiting**: Previne brute force e abuso
- ✅ **Auditoria completa**: Todos acessos logados
- ✅ **Domínio whitelist**: Apenas @crypteras.tech

### LGPD Compliance
- ✅ **Minimização de dados**: Backoffice não exibe dados sensíveis desnecessários
- ✅ **Logs auditados**: Rastreabilidade de quem acessou dados de quem
- ✅ **Retenção limitada**: Logs de 1 ano (requisito legal)
- ✅ **Acesso justificado**: Admins só acessam dados quando necessário (suporte, investigação)

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
