---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: "1.0.0"
status: "active"
priority: "high"
category: "feature"
tags: ['feature', 'business', 'authorization', 'authentication']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Autenticação e Autorização Multi-User

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Implementar autenticação segura e multi-usuário com proteção de credenciais de exchanges e compliance LGPD.

**Constraints** (limites obrigatórios):
- Usar JWT com access token (15 min) + refresh token (7 dias)
- Passwords DEVEM usar bcrypt com min 10 rounds (NUNCA plain text ou MD5)
- Credenciais de exchanges DEVEM ser encriptadas com AES-256-CBC no banco
- LGPD compliance obrigatório (consentimentos explícitos, audit trail, direito ao esquecimento)
- Rate limiting: 5 tentativas de login por IP/15min (prevenir brute force)
- Session management com refresh token rotation (prevenir token replay)
- Endpoints protegidos DEVEM validar JWT em middleware

**Non-Goals** (o que NÃO fazer):
- OAuth/Social login (Google, GitHub) no MVP
- Two-factor authentication (2FA) no MVP
- Biometria ou passwordless authentication
- Single Sign-On (SSO) corporativo
- Magic links ou login sem senha
- Suporte a múltiplos dispositivos simultâneos (apenas 1 sessão ativa)
- Recuperação de senha automática (apenas manual por email no MVP)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code (governança de decisões da IA)
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Sistema de autenticação e autorização
:::

## Visão Geral

Sistema completo de autenticação e autorização multi-usuário baseado em JWT (JSON Web Tokens) com suporte a refresh token rotation, criptografia de credenciais de exchange e compliance com LGPD.

**Tickets Relacionados**: CRY-12, CRY-41
**Status**: ✅ Em Produção
**Prioridade**: CRÍTICA

---

## Propósito e Valor

### Para o Usuário
- **Segurança**: Autenticação robusta com tokens JWT de curta duração
- **Privacidade**: Credenciais de exchange criptografadas com AES-256-CBC
- **Conveniência**: Refresh token automático sem necessidade de re-login frequente
- **Transparência**: Auditoria completa de acessos e consentimentos (LGPD)

### Para o Negócio
- **Escalabilidade**: Suporte a múltiplos usuários concorrentes
- **Conformidade**: Atendimento total aos requisitos de LGPD
- **Rastreabilidade**: Audit trail completo (IP, User-Agent, timestamps)
- **Segurança**: Zero credenciais em texto puro no banco de dados

---

## Funcionalidades Principais

### 1. Registro de Novo Usuário

**Endpoint**: `POST /api/auth/register`

**Payload**:
```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd",
  "full_name": "João da Silva",
  "legal_consents": {
    "terms_accepted": true,
    "privacy_accepted": true,
    "marketing_accepted": false
  }
}
```

**Resposta**:
```json
{
  "access_token": "eyJhbGci...",
  "refresh_token": "eyJhbGci...",
  "token_type": "bearer",
  "user": {
    "user_id": "65f8a1...",
    "email": "user@example.com",
    "full_name": "João da Silva"
  }
}
```

**Regras de Negócio**:
1. ✅ Email deve ser único no sistema (validação backend + MongoDB unique index)
2. ✅ Senha deve ter mínimo 8 caracteres
3. ✅ `terms_accepted` e `privacy_accepted` devem ser `true` (obrigatórios)
4. ✅ `marketing_accepted` pode ser `false` (opcional)
5. ✅ Senha é hasheada com bcrypt (custo 12)
6. ✅ IP e User-Agent são capturados para auditoria
7. ✅ Subscription FREE é criada automaticamente
8. ✅ Retorna par de tokens JWT (access + refresh)

**Validações**:
- Email formato válido (regex: RFC 5322)
- Email normalizado para lowercase
- Senha não vazia (mínimo 8 caracteres)
- Aceites legais presentes e validados no backend

---

### 2. Login

**Endpoint**: `POST /api/auth/login`

**Payload**:
```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd"
}
```

**Resposta**:
```json
{
  "access_token": "eyJhbGci...",
  "refresh_token": "eyJhbGci...",
  "token_type": "bearer"
}
```

**Regras de Negócio**:
1. ✅ Valida email + senha (bcrypt verify)
2. ✅ Gera access token (validade: 1 hora)
3. ✅ Gera refresh token (validade: 7 dias)
4. ✅ Refresh token hasheado e armazenado no banco (bcrypt)
5. ✅ Atualiza `last_login` timestamp
6. ✅ Invalida refresh token anterior (segurança)

**JWT Claims**:
```json
{
  "sub": "user_id",
  "email": "user@example.com",
  "exp": 1234567890,
  "iat": 1234567890,
  "type": "access"
}
```

**Segurança**:
- Password never logged ou exposto em erros
- Tokens assinados com HS256 (HMAC SHA-256)
- Secret key 256-bit mínimo (JWT_SECRET)

---

### 3. Renovação de Token (Refresh)

**Endpoint**: `POST /api/auth/v2/refresh`

**Payload**:
```json
{
  "refresh_token": "eyJhbGci..."
}
```

**Resposta**:
```json
{
  "access_token": "eyJhbGci...",
  "refresh_token": "eyJhbGci...",
  "token_type": "bearer"
}
```

**Regras de Negócio**:
1. ✅ Valida assinatura do refresh token
2. ✅ Verifica expiração (7 dias)
3. ✅ Busca hash do refresh token no banco
4. ✅ Compara hash (bcrypt verify)
5. ✅ Gera NOVO par access + refresh
6. ✅ Invalida refresh token anterior (rotation)
7. ✅ Previne token replay attacks

**Fluxo de Rotation**:
```
Token A (válido) → Refresh → Token B (válido), Token A invalidado
Token B (válido) → Refresh → Token C (válido), Token B invalidado
Token A (invalidado) → Refresh → ERRO 401 (token reuse detected)
```

---

### 4. Logout

**Endpoint**: `POST /api/auth/v2/logout`

**Headers**: `Authorization: Bearer <access_token>`

**Regras de Negócio**:
1. ✅ Valida access token
2. ✅ Extrai user_id do token
3. ✅ Remove refresh_token_hash do banco
4. ✅ Invalida todos os tokens do usuário

**Nota**: Frontend deve remover tokens do localStorage/cookie

---

### 5. Obter Dados do Usuário Autenticado

**Endpoint**: `GET /api/auth/me`

**Headers**: `Authorization: Bearer <access_token>`

**Resposta**:
```json
{
  "user_id": "65f8a1...",
  "email": "user@example.com",
  "full_name": "João da Silva",
  "created_at": "2025-01-15T10:00:00Z",
  "last_login": "2025-01-15T10:30:00Z",
  "has_mb_credentials": true,
  "has_binance_credentials": false,
  "legal_consents": {
    "terms_accepted": true,
    "privacy_accepted": true,
    "marketing_accepted": false,
    "timestamp": "2025-01-15T10:00:00Z",
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0..."
  }
}
```

**Regras de Negócio**:
1. ✅ Retorna dados do usuário (NUNCA retorna senha ou refresh token)
2. ✅ Indica se credenciais de exchange estão configuradas
3. ✅ Expõe consentimentos legais para auditoria

---

### 6. Atualizar Credenciais de Exchange

#### 6.1 Mercado Bitcoin

**Endpoint**: `PUT /api/auth/credentials`

**Payload**:
```json
{
  "mb_api_id": "12345",
  "mb_api_key": "secret_key_here",
  "mb_account_id": "67890"
}
```

**Regras de Negócio**:
1. ✅ Criptografa cada campo com AES-256-CBC
2. ✅ Gera IV (Initialization Vector) único e aleatório para cada campo
3. ✅ Armazena dados criptografados + IVs separados
4. ✅ ENCRYPTION_KEY deve estar configurada no .env (256 bits)
5. ✅ Valida que campos não são vazios antes de criptografar

**Formato de Armazenamento**:
```python
# Banco de dados (MongoDB)
{
  "mb_api_id_encrypted": "base64_encrypted_data",
  "mb_api_key_encrypted": "base64_encrypted_data",
  "mb_account_id_encrypted": "base64_encrypted_data",
  "credentials_iv": "iv1:iv2:iv3"  # IVs separados por ":"
}
```

#### 6.2 Binance

**Endpoint**: `PUT /api/auth/binance-credentials`

**Payload**:
```json
{
  "binance_api_key": "api_key_here",
  "binance_api_secret": "api_secret_here",
  "binance_use_testnet": false
}
```

**Regras de Negócio**:
1. ✅ Criptografa api_key e api_secret separadamente (AES-256-CBC)
2. ✅ IVs únicos armazenados em `binance_credentials_iv`
3. ✅ `binance_use_testnet` flag booleana (não criptografada)
4. ✅ Testnet permite testes sem risco financeiro

---

## Segurança e Criptografia

### Password Hashing (bcrypt)
```python
# Registro
password_hash = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt(rounds=12))

# Login
is_valid = bcrypt.checkpw(password.encode('utf-8'), password_hash)
```

**Parâmetros**:
- Algorithm: bcrypt
- Cost factor: 12 (recomendado para 2025)
- Salt: Automático, único por senha

### Criptografia de Credenciais (AES-256-CBC)
```python
# Encryption
iv = os.urandom(16)  # 16 bytes random
cipher = Cipher(algorithms.AES(encryption_key), modes.CBC(iv))
encrypted = cipher.encryptor().update(plaintext) + cipher.encryptor().finalize()

# Storage
encrypted_b64 = base64.b64encode(encrypted).decode('utf-8')
iv_b64 = base64.b64encode(iv).decode('utf-8')
```

**Parâmetros**:
- Algorithm: AES-256
- Mode: CBC (Cipher Block Chaining)
- Key size: 256 bits (ENCRYPTION_KEY)
- IV size: 16 bytes (128 bits)
- IV: Único por campo (evita padrões detectáveis)

**Segurança**:
- ✅ IV único por campo (mesmo valor criptografado resulta em ciphertexts diferentes)
- ✅ Key rotation suportada (basta trocar ENCRYPTION_KEY e re-criptografar)
- ✅ Decryption apenas em memória (nunca logada)

---

## Auditoria e Compliance (LGPD)

### Captura de IP Real (Cloudflare-Aware)

```python
def extract_real_ip(request: Request) -> str:
    # Ordem de prioridade:
    # 1. CF-Connecting-IP (Cloudflare)
    # 2. X-Forwarded-For (proxy)
    # 3. request.client.host (direto)

    return ip_address
```

**Campos Auditados**:
- `legal_consents.ip_address`: IP do usuário no momento do aceite
- `legal_consents.user_agent`: User-Agent do navegador
- `legal_consents.timestamp`: Timestamp ISO do aceite
- `created_at`: Data de criação da conta
- `last_login`: Último acesso

**Retenção**:
- Dados auditados mantidos por tempo legal (5 anos - LGPD)
- Usuário pode solicitar dados via `GET /api/auth/me`
- Exclusão de conta: soft delete (flag `deleted_at`)

---

## Autorização e Permissões

### Níveis de Acesso

#### 1. Público (sem autenticação)
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/legal/terms`
- `GET /api/legal/privacy`

#### 2. Autenticado (JWT required)
- Middleware: `get_current_user(token: str = Depends(oauth2_scheme))`
- Validação: Assinatura JWT + expiração
- Extração: `Authorization: Bearer <token>`
- Todos os endpoints `/api/*` exceto públicos

#### 3. Ownership (recurso próprio)
- SmartBots: `bot.user_id == current_user.user_id`
- CandleBots: `bot.user_id == current_user.user_id`
- DCABots: `bot.user_id == current_user.user_id`
- Orders: `order.user_id == current_user.user_id`
- Subscription: `subscription.user_id == current_user.user_id`

#### 4. Backoffice (admin)
- `POST /api/backoffice/login` (JWT específico de BO)
- `GET /api/backoffice/users` (listar todos)
- `PUT /api/backoffice/users/{id}/block` (bloquear usuário)
- Middleware separado: `get_current_bo_user`

---

## Métricas de Sucesso

### KPIs
- **Tempo de resposta**: < 200ms (login, refresh)
- **Taxa de sucesso**: > 99.9%
- **Taxa de erro de autenticação**: < 0.1%
- **Tokens invalidados/dia**: (monitoramento de segurança)

### Monitoramento
- Logins bem-sucedidos (diário)
- Tentativas de login falhadas (alerta em picos)
- Refresh tokens rotacionados (diário)
- Tokens expirados/inválidos (erro 401)

---

## Problemas Comuns e Limitações

### ❌ Token Expirado
**Sintoma**: Erro 401 Unauthorized
**Causa**: Access token expirou (> 1 hora)
**Solução**: Frontend deve chamar `/api/auth/v2/refresh` automaticamente

### ❌ Refresh Token Inválido
**Sintoma**: Erro 401 ao renovar token
**Causa**: Refresh token expirou (> 7 dias) ou foi usado mais de uma vez
**Solução**: Usuário deve fazer login novamente

### ❌ Credenciais de Exchange Inválidas
**Sintoma**: Erro ao criar bots (400 Bad Request)
**Causa**: API keys incorretas ou sem permissões
**Solução**: Revalidar credenciais no painel da exchange

### ⚠️ Limitações
- Refresh token de 7 dias (configurável, mas trade-off segurança vs UX)
- Criptografia AES-256-CBC (necessita ENCRYPTION_KEY no .env)
- Logout não invalida access tokens ativos (stateless JWT - expira em 1h)

---

## Boas Práticas para Desenvolvedores

### Frontend

```javascript
// Interceptor Axios - Auto Refresh
axios.interceptors.response.use(
  response => response,
  async error => {
    if (error.response?.status === 401) {
      const refreshToken = localStorage.getItem('refresh_token');

      if (refreshToken) {
        try {
          const { data } = await axios.post('/api/auth/v2/refresh', {
            refresh_token: refreshToken
          });

          localStorage.setItem('access_token', data.access_token);
          localStorage.setItem('refresh_token', data.refresh_token);

          // Retry request original
          error.config.headers['Authorization'] = `Bearer ${data.access_token}`;
          return axios(error.config);
        } catch (refreshError) {
          // Refresh falhou - redireciona para login
          window.location.href = '/login';
        }
      }
    }

    return Promise.reject(error);
  }
);
```

### Backend

```python
# Dependency Injection
from api.dependencies.auth import get_current_user, get_current_subscription

@router.post("/smart-bots")
async def create_smart_bot(
    request: CreateSmartBotRequest,
    current_user: UserEntity = Depends(get_current_user),
    subscription: SubscriptionEntity = Depends(get_current_subscription)
):
    # current_user já está autenticado (JWT validado)
    # subscription já está carregada

    if not subscription.is_active():
        raise HTTPException(403, "Subscription inativa")

    # ... lógica do endpoint
```

---

## Roadmap Futuro

### Melhorias Planejadas
- [ ] **2FA (Two-Factor Authentication)**: TOTP via Google Authenticator
- [ ] **OAuth2 Social Login**: Google, GitHub, Microsoft
- [ ] **Session Management**: Listar e revogar sessões ativas
- [ ] **IP Whitelisting**: Restrição de IPs permitidos (opcional)
- [ ] **Audit Log Detalhado**: Dashboard de acessos e ações

### Considerações de Segurança Futuras
- [ ] **Rate Limiting por IP**: 100 req/min (registro/login)
- [ ] **CAPTCHA após falhas**: 5 tentativas erradas → CAPTCHA
- [ ] **Account Lockout**: 10 logins falhos/hora → bloqueio temporário
- [ ] **Password Policy**: Complexidade mínima (upper, lower, digit, special)
- [ ] **Password Expiration**: Renovação obrigatória a cada 90 dias (enterprise)

---

## Referências Técnicas

### Arquivos Relacionados
- `backend/src/domain/entities/user.py` - UserEntity
- `backend/src/application/use_cases/register_user.py` - Registro
- `backend/src/application/use_cases/login_user.py` - Login
- `backend/src/api/routes/auth.py` - Endpoints
- `backend/src/api/dependencies/auth.py` - Middlewares
- `backend/src/infrastructure/security/encryption.py` - Criptografia
- `backend/src/infrastructure/security/jwt.py` - JWT handling

### Standards e Compliance
- **LGPD**: Lei Geral de Proteção de Dados (Brasil)
- **OWASP**: Top 10 Security Risks (A02:2021 - Cryptographic Failures)
- **RFC 7519**: JSON Web Token (JWT)
- **RFC 5322**: Email Address Specification
- **NIST SP 800-63B**: Digital Identity Guidelines (Password Hashing)
