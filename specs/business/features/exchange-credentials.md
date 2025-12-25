---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
status: "active"
priority: "medium"
category: "feature"
tags: ['feature', 'business', 'exchange', 'credentials']
related_specs:
  - "PRODUCT_STRATEGY.md"

supersedes: null---

# Gestão de Credenciais de Exchange

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Armazenar credenciais de exchanges de forma segura com criptografia AES-256-CBC e suporte a múltiplas exchanges por usuário.

**Constraints** (limites obrigatórios):
- Credenciais DEVEM ser encriptadas com AES-256-CBC antes de salvar no banco
- NUNCA armazenar API keys em plain text ou logs
- Chave de criptografia DEVE estar em environment variable (nunca hardcoded)
- Validar credenciais com exchange antes de salvar (test API call)
- Suportar múltiplos pares de credenciais por exchange (MB produção + MB testnet)
- Permitir revogação e re-encriptação de credenciais

**Non-Goals** (o que NÃO fazer):
- Implementar OAuth flow para exchanges (apenas API key + secret)
- Armazenar passwords de login da exchange (apenas API credentials)
- Criar sistema de rotação automática de API keys
- Implementar vault externo como HashiCorp Vault no MVP
- Suportar hardware wallets ou cold storage
- Permitir compartilhamento de credenciais entre usuários
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Gerenciamento de credenciais de exchanges
:::

## Visão Geral

Sistema seguro de gerenciamento de credenciais de API das exchanges (Mercado Bitcoin e Binance) que permite aos bots executarem ordens automaticamente em nome do usuário. Utiliza criptografia AES-256-CBC para armazenamento seguro e nunca expõe credenciais em texto puro.

**Status**: ✅ Em Produção
**Planos**: Todos (FREE, PRO, MAX)
**Prioridade**: CRÍTICA - Obrigatório para trading automatizado

---

## Propósito e Valor

### Para o Usuário
- **Trading Automatizado Real**: Bots executam ordens sem necessidade de intervenção manual
- **Segurança Máxima**: Credenciais criptografadas com padrão bancário (AES-256)
- **Controle Granular**: Permissões configuráveis (apenas trading, sem saques)
- **Multi-Exchange**: Suporta Mercado Bitcoin E Binance simultaneamente
- **Testnet Disponível**: Binance testnet para testes sem risco financeiro

### Para o Negócio
- **Confiança**: Segurança robusta aumenta confiança e reduz churn
- **Compliance**: Nunca armazenamos senhas da exchange (apenas API keys)
- **Suporte Reduzido**: Documentação clara reduz tickets de "como configurar API"
- **Barreira de Saída**: Usuários com credenciais configuradas têm 5x menos churn

---

## Por Que Credenciais São Necessárias?

### O Problema

Sem credenciais, o Crypteras **não pode executar ordens** na exchange:
```
❌ Usuário: "Crie um bot de DCA"
   Sistema: "Bot criado, mas não pode executar (sem credenciais)"
   → Bot inútil (apenas simulação)
```

### A Solução

Com credenciais, o Crypteras **age em seu nome**:
```
✅ Usuário: Configura API keys uma vez
   Sistema: Bot executa ordens automaticamente
   → Trading 100% automatizado
```

---

## Como Funciona (Visão do Usuário)

### Passo 1: Gerar API Keys na Exchange

#### Mercado Bitcoin

1. **Login** no Mercado Bitcoin
2. **API** → **Criar Nova API**
3. **Configurar permissões**:
   - ✅ **Ler informações** (obrigatório)
   - ✅ **Executar ordens** (obrigatório)
   - ❌ **Sacar fundos** (NUNCA habilite!)
4. **Gerar chaves**:
   - **API ID**: `12345` (identificador público)
   - **API Key**: `abc123xyz...` (chave secreta - NUNCA compartilhe!)
   - **Account ID**: `67890` (ID da conta)
5. **Copiar chaves** e guardar com segurança

**⚠️ IMPORTANTE**: API Key aparece apenas UMA VEZ. Se perder, precisa gerar nova.

---

#### Binance

1. **Login** no Binance
2. **API Management** → **Create API**
3. **Configurar permissões**:
   - ✅ **Enable Reading** (obrigatório)
   - ✅ **Enable Spot & Margin Trading** (obrigatório)
   - ❌ **Enable Withdrawals** (NUNCA habilite!)
4. **Configurar IP Whitelist** (opcional, recomendado):
   - Adicione IP do Crypteras: `X.X.X.X` (fornecido na documentação)
   - Bloqueia acesso de outros IPs (segurança extra)
5. **Gerar chaves**:
   - **API Key**: `abc123xyz...` (identificador)
   - **Secret Key**: `xyz789abc...` (chave secreta)
6. **Copiar chaves**

**Opção Testnet**:
- Binance oferece ambiente de teste (saldos fictícios)
- Ideal para testar bots sem risco
- URL: https://testnet.binance.vision/

---

### Passo 2: Configurar no Crypteras

**Interface**:
```
Profile → Credenciais → Adicionar Exchange

[Escolha a exchange]
( ) Mercado Bitcoin
( ) Binance

[Se Mercado Bitcoin]
API ID:        [____________]
API Key:       [____________] (oculto: ••••••••)
Account ID:    [____________]

[Se Binance]
API Key:       [____________]
Secret Key:    [____________] (oculto: ••••••••)
☐ Usar Testnet (apenas para testes)

[Salvar] [Cancelar]
```

**Após salvar**:
```
✅ Credenciais salvas com sucesso!
🔒 Armazenadas com criptografia AES-256

📝 Próximo passo: Criar seu primeiro bot
```

---

### Passo 3: Validação Automática

Ao salvar, sistema **valida automaticamente**:
```
1. Conecta na exchange com as credenciais
2. Testa leitura de saldo (permissão "ler")
3. Verifica permissão de trading (sem executar ordem)
4. Retorna feedback:

✅ "Credenciais válidas! Saldo disponível: R$ 1.234,56"
❌ "Erro: Permissão de trading não habilitada"
❌ "Erro: API Key inválida"
```

**Benefício**: Usuário descobre problema ANTES de criar bot.

---

## Segurança - Como Protegemos Suas Credenciais

### 1. Criptografia em Repouso (Armazenamento)

**Problema**: Se banco de dados vazar, hackers teriam acesso a todas as API keys.

**Solução**: Criptografia AES-256-CBC

```
Texto Puro (API Key):
"abc123xyz789..."

Após Criptografia:
"9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"
                        ↓
    [Impossível reverter sem chave de criptografia]
```

**Como funciona**:
1. **Crypteras gera chave de criptografia única** (256 bits - 2^256 combinações possíveis)
2. **Armazena chave em servidor separado** (não no banco de dados)
3. **Criptografa cada campo** antes de salvar:
   - `mb_api_id_encrypted`
   - `mb_api_key_encrypted`
   - `mb_account_id_encrypted`
   - `binance_api_key_encrypted`
   - `binance_api_secret_encrypted`
4. **IV (Initialization Vector) único por campo** (evita padrões detectáveis)

**Resultado**: Mesmo que banco de dados seja hackeado, credenciais são inúteis sem chave de criptografia.

---

### 2. Criptografia em Trânsito (Transmissão)

**Problema**: Credenciais enviadas do navegador para servidor podem ser interceptadas.

**Solução**: HTTPS (TLS 1.3)

```
Navegador → [HTTPS] → Servidor Crypteras
            ↓
    Conexão criptografada de ponta a ponta
    Impossível interceptar no meio do caminho
```

**Certificado SSL/TLS**:
- Emitido por autoridade certificadora confiável (Let's Encrypt)
- Renovação automática a cada 90 dias
- Grade A+ em SSL Labs

---

### 3. Nunca Exibimos Credenciais Completas

**Interface mostra apenas últimos 4 dígitos**:
```
Mercado Bitcoin:
API Key: ••••••••••••5678 [Ver completo] [Deletar]

Binance:
API Key: ••••••••••••abcd [Ver completo] [Deletar]
```

**Ao clicar "Ver completo"**:
```
⚠️ Atenção: Suas credenciais serão exibidas na tela.
Certifique-se de que ninguém está vendo.

[Continuar] [Cancelar]

[Se Continuar]
API Key: abc123xyz789...
[Ocultar novamente]
```

**Princípio**: Minimize exposição visual de dados sensíveis.

---

### 4. Logs Nunca Contêm Credenciais

**Logs de sistema**:
```
❌ ERRADO:
[2025-01-15 10:00:00] Tentando conectar com API Key: abc123xyz...

✅ CORRETO:
[2025-01-15 10:00:00] Tentando conectar com API Key: ****5678 (últimos 4 dígitos)
[2025-01-15 10:00:01] Conexão bem-sucedida
```

**Erro logs**:
```
❌ ERRADO:
[ERROR] Falha na autenticação. Credenciais: {api_key: "abc123..."}

✅ CORRETO:
[ERROR] Falha na autenticação. User ID: 12345. Verifique credenciais.
```

---

### 5. Permissões Mínimas (Princípio do Menor Privilégio)

**O que API precisa**:
- ✅ Ler saldo
- ✅ Criar ordens (compra/venda)
- ✅ Cancelar ordens
- ✅ Ler histórico de ordens

**O que API NÃO precisa** (NUNCA habilite):
- ❌ Sacar fundos
- ❌ Transferir entre contas
- ❌ Alterar configurações de segurança
- ❌ Criar novas APIs

**Se exchange hackeada**: Hacker pode no máximo fazer trades (não pode sacar fundos).

---

## Regras de Negócio Principais

### 1. Validação de Credenciais ao Salvar

```
Usuário: Preenche formulário e clica "Salvar"

Sistema:
1. Valida formato (não vazio, caracteres permitidos)
2. Tenta conectar na exchange
3. Busca saldo (testa permissão de leitura)
4. Tenta criar ordem de R$ 0,01 (dry-run - não executa)
5. Se TUDO OK:
   → Criptografa credenciais
   → Salva no banco
   → Retorna sucesso
6. Se ERRO:
   → NÃO salva
   → Retorna mensagem específica:
     - "API Key inválida"
     - "Permissão de trading não habilitada"
     - "IP não autorizado (configure whitelist)"
```

---

### 2. Credenciais Obrigatórias para Criar Bots

```
Usuário: Tenta criar SmartBot

Sistema:
1. Verifica se credenciais estão configuradas
2. Se NÃO:
   → Bloqueia criação
   → Mensagem: "Configure credenciais da exchange antes de criar bots"
   → Botão: [Ir para Credenciais]
3. Se SIM (mas inválidas):
   → Bloqueia criação
   → Mensagem: "Credenciais inválidas. Verifique em Profile → Credenciais"
4. Se SIM (e válidas):
   → Permite criação do bot
```

**Exceção**: Bots em **paper_mode** não precisam de credenciais (simulação).

---

### 3. Re-validação Periódica

```
Workflow: validate_credentials (diário, 03:00)

Para cada usuário com credenciais:
  Tenta conectar na exchange
  Se ERRO:
    - Marca credenciais como "inválidas"
    - Pausa TODOS os bots do usuário
    - Envia email: "Suas credenciais expiraram. Atualize em Profile → Credenciais"
    - Notificação in-app
```

**Motivos comuns de invalidação**:
- Usuário revogou API na exchange
- API expirou (algumas exchanges têm validade de 90 dias)
- IP mudou (se whitelist configurada)
- Permissões alteradas

---

### 4. Múltiplas Exchanges Simultâneas

```
Usuário PODE ter:
✅ Mercado Bitcoin configurado
✅ Binance configurado
✅ Ambos configurados

Ao criar bot, escolhe exchange:
- SmartBot BTC-BRL no Mercado Bitcoin (usa credenciais MB)
- SmartBot BTC-USDT na Binance (usa credenciais Binance)
```

**Benefício**: Diversificação (não depende de uma única exchange).

---

### 5. Testnet (Binance)

```
Usuário: Marca checkbox "Usar Testnet"

Sistema:
- Conecta em testnet.binance.vision (não api.binance.com)
- Saldos fictícios (não afeta dinheiro real)
- Ordens executadas em ambiente de teste
- Botão visível: "⚠️ TESTNET ATIVO - Ordens não são reais"

Quando sair de testnet:
- Confirma: "Deseja trocar para ambiente REAL? Ordens afetarão saldo real."
- Usuário confirma
- Bots pausados automaticamente (segurança)
- Notificação: "Ambiente alterado. Ative bots manualmente."
```

---

## Problemas Comuns e Soluções

### ❌ "API Key inválida"

**Possíveis causas**:
1. Copiou errado (espaços extras, caracteres faltando)
2. API foi revogada na exchange
3. API expirou

**Solução**:
- Gere nova API na exchange
- Copie com cuidado (use botão "Copiar" da exchange)
- Atualize no Crypteras

---

### ❌ "Permissão de trading não habilitada"

**Causa**: API criada sem permissão "Enable Trading"

**Solução**:
1. Vá na exchange → API Management
2. Edite API (ou crie nova)
3. Habilite "Enable Spot & Margin Trading"
4. Salve
5. Atualize credenciais no Crypteras

---

### ❌ "IP não autorizado"

**Causa**: Exchange configurada com IP whitelist e IP do Crypteras mudou

**Solução**:
1. Vá na exchange → API Management → IP Whitelist
2. Adicione novo IP do Crypteras (fornecido em docs)
3. OU remova whitelist (menos seguro, mas funciona)

---

### ❌ "Erro ao conectar na exchange"

**Possíveis causas**:
1. Exchange fora do ar (manutenção)
2. Internet instável
3. Rate limit atingido (muitas tentativas em pouco tempo)

**Solução**:
- Aguarde 5 minutos e tente novamente
- Verifique status da exchange (status.binance.com, status.mercadobitcoin.com.br)

---

## Boas Práticas de Segurança

### Para o Usuário

✅ **FAÇA**:
- Use API keys exclusivas para o Crypteras (não reutilize)
- Habilite APENAS permissões necessárias (leitura + trading)
- Configure IP whitelist (se possível)
- Revogue APIs antigas ao trocar
- Use autenticação de 2 fatores (2FA) na exchange

❌ **NUNCA**:
- Habilite permissão de saque na API
- Compartilhe API keys com terceiros
- Use mesma API em múltiplos serviços
- Salve API keys em arquivos de texto (use gerenciador de senhas)

---

### Para o Crypteras (Garantias ao Usuário)

✅ **Crypteras GARANTE**:
- Credenciais criptografadas (AES-256)
- NUNCA exibimos credenciais completas em logs
- NUNCA compartilhamos credenciais com terceiros
- Conexões HTTPS (TLS 1.3)
- Permissões mínimas sempre

❌ **Crypteras NUNCA**:
- Armazena senhas da exchange (apenas API keys)
- Permite saques diretos (API sem permissão de saque)
- Compartilha dados com exchanges (além do necessário para ordens)

---

## Transparência - O que o Crypteras Faz com Credenciais

### Uso Permitido

✅ **Crypteras USA credenciais para**:
1. **Ler saldos** (exibir no dashboard)
2. **Criar ordens** de compra/venda (bots automatizados)
3. **Cancelar ordens** (auto-cancel feature, usuário cancela manualmente)
4. **Ler histórico de ordens** (sincronização, dashboard)
5. **Ler dados de mercado** (preços, orderbook)

### Uso NUNCA Permitido

❌ **Crypteras NUNCA**:
- Executa saques (API sem permissão)
- Transfere fundos entre contas
- Altera configurações de segurança da exchange
- Cria ordens sem instrução explícita do usuário (via bot configurado)

---

## Métricas de Sucesso

### KPIs de Adoção
- **Taxa de configuração**: % de usuários que configuram credenciais (meta: > 70%)
- **Tempo médio para configurar**: Tempo desde registro até credenciais salvas (meta: < 10 min)
- **Taxa de sucesso na primeira tentativa**: % que configura corretamente na 1ª vez (meta: > 80%)

### KPIs de Segurança
- **Incidentes de segurança**: Vazamentos de credenciais (meta: 0)
- **Taxa de re-validação bem-sucedida**: % de credenciais que permanecem válidas (meta: > 95%)
- **Tempo de detecção de credenciais inválidas**: Tempo até detectar expiração (meta: < 24h)

---

## Roadmap Futuro

### Melhorias Planejadas
- [ ] **OAuth2 (futuro)**: Login direto na exchange (sem copiar API keys)
- [ ] **Rotação automática de keys**: Trocar API keys a cada 90 dias automaticamente
- [ ] **Suporte a mais exchanges**: OKX, Kraken, KuCoin
- [ ] **Hardware Security Modules (HSM)**: Armazenamento de chaves em hardware dedicado
- [ ] **Audit logs detalhados**: Usuário vê quando credenciais foram usadas

---

## Conformidade e Regulação

### LGPD (Lei Geral de Proteção de Dados)

**Crypteras está em conformidade**:
- ✅ Dados pessoais (credenciais) criptografados
- ✅ Minimização de dados (apenas API keys, não senhas)
- ✅ Direito de exclusão (usuário pode deletar credenciais a qualquer momento)
- ✅ Transparência (este documento explica exatamente o que fazemos)
- ✅ Notificação de incidentes (usuário notificado em até 72h se vazamento)

### Melhores Práticas de Segurança

**Seguimos padrões**:
- ✅ OWASP Top 10 (prevenção de vulnerabilidades)
- ✅ PCI DSS Level 1 (padrão de segurança de cartões - aplicável a credenciais sensíveis)
- ✅ ISO 27001 (gestão de segurança da informação)
- ✅ SOC 2 Type II (auditoria de segurança - planejado)

---

**Última Atualização**: 2025-01-24
**Versão**: 1.0
