---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "business"
tags: ['business', 'customer_communication']
---

# Diretrizes de Comunicação com Cliente - Crypteras

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir tom de voz e guidelines de comunicação para consistência em todos canais.

**Constraints** (limites obrigatórios):
- Tom: educacional, honesto, sem hype
- Sempre incluir disclaimers de risco em comunicação sobre trading
- Evitar jargão técnico desnecessário
- Português brasileiro (não Portugal)

**Non-Goals** (o que NÃO fazer):
- Criar marketing agressivo ou promessas de lucro garantido
- Usar memes ou humor excessivo (manter profissionalismo)
- Traduzir conteúdo internacional sem adaptar contexto brasileiro
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Diretrizes de comunicação com clientes
:::

**Versão**: 1.0
**Data**: 2024-12-24

---

## Princípios de Comunicação

### 1. Amigável, Não Corporativo

**Sim** ✅:
> "Oi Lucas! Vi que você criou seu primeiro bot. Parabéns! 🎉 Tem alguma dúvida?"

**Não** ❌:
> "Prezado senhor, conforme ticket #12345, informamos que sua solicitação..."

---

### 2. Educativo, Não Condescendente

**Sim** ✅:
> "DCA significa compras automáticas em intervalos fixos. Por exemplo: R$ 100 toda segunda-feira. Quer que eu te ajude a configurar?"

**Não** ❌:
> "Você não sabe o que é DCA? É básico. Pesquise no Google antes de usar o sistema."

---

### 3. Transparente Sobre Limitações

**Sim** ✅:
> "Ainda não temos TradingView integration, mas está no roadmap para Q4 2025. Enquanto isso, você pode usar nossos indicadores nativos (RSI, MACD, BB). Quer saber mais?"

**Não** ❌:
> "Sim, temos tudo que você precisa!" (quando não tem)

---

### 4. Responsivo, Não Reativo

**Sim** ✅:
> "Entendo sua frustração com a perda. Vamos analisar juntos o que aconteceu e ajustar a estratégia para evitar isso no futuro."

**Não** ❌:
> "Trading tem risco. Você aceitou os termos. Não podemos fazer nada."

---

## Canais de Comunicação

### Email (Canal Principal)

**SLA**: Resposta em até 24 horas (dias úteis)
**Horário**: Segunda a sexta, 9h-18h (BRT)
**Exceções**: Tickets marcados como URGENTE → 4-6 horas

**Template de Resposta Padrão**:
```
Oi [Nome],

[Resposta personalizada ao problema/dúvida]

[Solução ou próximos passos]

Se tiver mais dúvidas, é só responder este email!

Abs,
[Seu Nome]
Equipe Crypteras

P.S.: [Dica extra ou recurso relevante]
```

---

### WhatsApp (Futuro)

**Status**: Planejado para Q2 2025
**Uso**: Alertas críticos e notificações urgentes
**Não usar para**: Suporte geral (usar email)

**Tipos de Mensagens**:
- ⚠️ "Bot pausado por circuit breaker (perda 10%). Acesse dashboard"
- ✅ "Trade executado: +R$ 25 (BTC). Ver detalhes: [link]"
- 🔔 "Seu plano PRO vence em 3 dias. Renovar: [link]"

---

### Discord/Telegram (Comunidade - Futuro)

**Status**: Planejado para Q2 2025
**Moderação**: Equipe + membros verificados
**Regras**:
- Sem pump & dump
- Sem ofensas pessoais
- Sem spam de ref links

---

## Gestão de Tickets de Suporte

### Categorização

| Categoria | Prioridade | SLA | Exemplo |
|-----------|------------|-----|---------|
| **URGENTE** | P0 | 4-6h | "Bot travou e não para de comprar" |
| **Bug Crítico** | P1 | 24h | "Não consigo fazer login" |
| **Bug Menor** | P2 | 48h | "Dashboard não atualiza saldo" |
| **Dúvida** | P3 | 24-48h | "Como configuro trailing stop?" |
| **Feature Request** | P4 | 72h | "Quando vem TradingView?" |

---

### Templates de Resposta

#### Template: Bug Reportado

```
Oi [Nome],

Obrigado por reportar! Bugs são nossa prioridade máxima.

Confirmei o problema: [descrição do bug]

O que estamos fazendo:
1. Time técnico foi notificado
2. Investigação em andamento
3. Previsão de correção: [prazo]

Enquanto isso:
[Workaround temporário, se houver]

Vou te manter atualizado a cada 24h.

Desculpe o transtorno!

Abs,
[Nome]
Equipe Crypteras
```

---

#### Template: Dúvida de Configuração

```
Oi [Nome],

Ótima pergunta! [Resposta clara e direta]

Passo a passo:
1. [Passo 1 com screenshot se possível]
2. [Passo 2]
3. [Passo 3]

Vídeo tutorial: [link se disponível]

Funcionou? Qualquer dúvida, é só responder!

Abs,
[Nome]
Equipe Crypteras
```

---

#### Template: Reclamação de Perda Financeira

```
Oi [Nome],

Entendo sua frustração. Perder dinheiro é muito chato. 😔

Vamos analisar juntos o que aconteceu:

[Análise técnica do que ocorreu]

O que podemos fazer:
1. [Ajustar estratégia para evitar no futuro]
2. [Revisar configurações juntos]
3. [Se foi bug do sistema: compensação/crédito]

Importante lembrar:
Trading SEMPRE tem risco. Mas podemos minimizar com proteções melhores.

Quer marcar uma call de 15 min para revisar sua estratégia?

Abs,
[Nome]
Equipe Crypteras
```

---

#### Template: Feature Request

```
Oi [Nome],

Adoramos a sugestão: [feature solicitada]!

Status atual:
[✅ Já temos | 🔮 No roadmap Q[X] | 💡 Vamos avaliar]

[Se no roadmap]
Está planejado para Q[X] 2025. Quer que eu te avise quando lançar?

[Se vamos avaliar]
Vou encaminhar para time de produto. Se mais pessoas pedirem, priorizamos!

Obrigado por ajudar a melhorar o Crypteras!

Abs,
[Nome]
Equipe Crypteras
```

---

## Automações de Email

### Sequência de Boas-Vindas (FREE Tier)

**Email 1: Imediatamente após cadastro**
```
Assunto: Bem-vindo ao Crypteras! 🚀 Próximos passos

Oi [Nome],

Que bom ter você aqui!

Próximos passos para criar seu primeiro bot:
1️⃣ Conecte sua exchange (Binance ou Mercado Bitcoin)
   → Tutorial: [link]
2️⃣ Configure seu primeiro bot (recomendamos DCA)
   → Assistente de IA vai te guiar
3️⃣ Ative e acompanhe pelo dashboard

Precisa de ajuda? Responda este email!

Bons trades,
Equipe Crypteras
```

**Email 2: Day 3 (se não ativou bot)**
```
Assunto: Precisa de ajuda para começar?

Oi [Nome],

Vi que você ainda não criou seu primeiro bot. Tudo bem por aí?

Principais dúvidas de iniciantes:
❓ "Como obtenho credenciais de API?" → [Tutorial]
❓ "Qual bot criar primeiro?" → DCA é melhor para começar
❓ "É seguro?" → Sim! Seu dinheiro fica na exchange

Quer uma ajudinha? Responda este email ou assista: [Vídeo 5 min]

Abs,
Equipe Crypteras
```

**Email 3: Day 7 (resumo semanal)**
```
Assunto: Resumo da Semana: [Performance]

Oi [Nome],

Sua primeira semana no Crypteras:

📊 Performance:
- Trades executados: [X]
- P&L: [+/-] R$ [Y]
- Bots ativos: [Z]

💡 Dica da Semana:
[Insight personalizado baseado em uso]

🚀 Próximo nível:
[Se está no FREE] Quer rodar mais bots? Upgrade para PRO (R$ 19,90/mês)

Dúvidas? Responda este email!

Bons trades,
Equipe Crypteras
```

---

### Campanha de Conversão FREE → PRO

**Gatilho**: Usuário atingiu limite de 1 bot/estratégia

**Email**:
```
Assunto: Limite atingido 🚀 Hora de escalar?

Oi [Nome],

Vi que você criou [X] bots e atingiu o limite do FREE tier.

Isso é ótimo! Significa que você está usando bem o Crypteras.

Quer potencializar?

PRO (R$ 19,90/mês):
✅ 9 bots totais (3 por estratégia)
✅ Diversificação (BTC + ETH + ADA simultâneos)
✅ Suporte prioritário

Cálculo rápido:
Se 1 bot te deu +R$ [Y]/mês, imagine 9 bots?

[CTA: Upgrade para PRO]

Dúvidas? Responda este email!

Abs,
Equipe Crypteras
```

---

### Campanha de Retenção (Churn Prevention)

**Gatilho**: Usuário não faz login há 14 dias

**Email**:
```
Assunto: Sentimos sua falta! Tudo ok?

Oi [Nome],

Percebi que você não acessa há algumas semanas. Tudo bem por aí?

Seus bots estão [ativos/pausados]:
- [Bot 1]: [Status]
- [Bot 2]: [Status]

Algum problema? Podemos ajudar com:
❓ Dúvidas de configuração
❓ Otimização de estratégia
❓ Bugs ou problemas técnicos

Ou só quer dar um tempo? Sem problemas! Seus bots continuam salvos.

Abs,
Equipe Crypteras

P.S.: Responda "PAUSAR" se quiser pausar notificações por 30 dias.
```

---

## Escalação de Problemas

### Quando Escalar para Técnico

- Bug que impede uso do sistema
- Performance muito fora do esperado (perda > 20% em 1 dia)
- Inconsistência de dados (saldo incorreto)
- Problema de integração com exchange

---

### Quando Escalar para Gestão

- Cliente ameaça processo judicial
- Perda financeira > R$ 5.000 potencialmente causada por bug
- Reclamação de imprensa/mídia
- Churn de cliente MAX tier (alto valor)

---

### Quando Oferecer Compensação

**Critérios**:
- Bug confirmado do Crypteras (não da exchange)
- Perda financeira documentada
- Cliente é PRO/MAX tier e está há 3+ meses

**Compensações Possíveis**:
- 1-3 meses grátis de assinatura
- Crédito proporcional à perda (até R$ 500)
- Upgrade temporário para tier superior

**Não compensar**:
- Perdas devido a volatilidade normal do mercado
- Perdas devido a configuração errada do usuário
- Perdas em tier FREE (não há receita)

---

## Personalização por Persona

### Lucas (Iniciante)

**Tom**: Extra amigável, educativo
**Emojis**: Usar moderadamente (🚀 ✅ 💡)
**Explicações**: Simplificadas, com analogias
**Links**: Tutoriais, vídeos, guias passo-a-passo

**Exemplo**:
> "Oi Lucas! DCA é tipo guardar dinheiro no cofrinho toda semana, mas automaticamente em Bitcoin. Quer configurar?"

---

### Mariana (Intermediária)

**Tom**: Profissional, direto
**Emojis**: Raramente
**Explicações**: Técnicas, com dados
**Links**: Documentação avançada, roadmap, changelogs

**Exemplo**:
> "Oi Mariana. Identificamos o bug no CandleBot (RSI não estava calculando corretamente). Corrigido na v2.5.1 (deploy hoje 18h). Veja changelog: [link]"

---

### Rafael (Conservador)

**Tom**: Reassegurador, transparente
**Emojis**: Evitar
**Explicações**: Foco em segurança e proteções
**Links**: FAQs de segurança, disclaimers, termos de uso

**Exemplo**:
> "Oi Rafael. Entendo sua preocupação sobre segurança. Suas credenciais de API são criptografadas (AES-256) e armazenadas separadamente. Nem nossos desenvolvedores conseguem ver. Mais detalhes: [link segurança]"

---

## Comunicação de Crises

### Cenário 1: Downtime do Sistema

**Comunicação Imediata** (< 30 min):
```
🚨 ALERTA: Sistema temporariamente offline

Estamos cientes e time técnico está trabalhando.

Seus bots: PAUSADOS automaticamente (proteção)
Seu dinheiro: SEGURO na exchange

ETA de retorno: [tempo estimado]

Atualizações a cada 30 min via:
- Email
- Twitter @crypterasbr
- Status page: status.crypteras.tech

Desculpe o transtorno.
Equipe Crypteras
```

**Comunicação de Resolução**:
```
✅ Sistema ONLINE novamente

Downtime: [X] minutos
Causa: [explicação técnica simples]

Seus bots:
- Automaticamente reativados
- Zero trades perdidos (por design)

Ações tomadas:
- [Correção aplicada]
- [Medidas preventivas]

Compensação:
- [Se > 4h downtime] +1 semana grátis para PRO/MAX

Desculpe novamente.
Equipe Crypteras
```

---

### Cenário 2: Bug Causou Perdas

**Comunicação Imediata**:
```
⚠️ ALERTA: Bug identificado e corrigido

Bug: [descrição]
Impacto: [X] usuários afetados
Status: CORRIGIDO (v[versão])

Se você foi afetado:
1. Verifique seu dashboard
2. Responda este email com:
   - ID dos trades afetados
   - Perda estimada
3. Investigaremos e compensaremos caso confirmado

Desculpe MUITO.
Equipe Crypteras
```

---

## Métricas de Comunicação

| Métrica | Meta | Frequência de Medição |
|---------|------|----------------------|
| **Tempo médio de resposta** | < 24h | Semanal |
| **Taxa de resolução no 1º contato** | > 60% | Mensal |
| **CSAT (Customer Satisfaction)** | > 80% | Após cada ticket |
| **NPS** | > 40 | Trimestral |
| **Escalações para gestão** | < 5/mês | Mensal |

---

## Treinamento de Suporte

### Onboarding de Novo Atendente

**Semana 1: Produto**
- Usar Crypteras como usuário FREE por 3 dias
- Criar todos os tipos de bot (DCA, Candle, Smart)
- Ler documentação técnica completa

**Semana 2: Atendimento**
- Shadowing (acompanhar atendente experiente)
- Responder tickets supervisionados
- Estudar FAQs e templates

**Semana 3: Autonomia**
- Responder tickets sozinho
- Review semanal com supervisor
- Meta: 80% CSAT

---

## Versionamento

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 2024-12-24 | Diretrizes iniciais de comunicação |

---

**Próxima Revisão**: Trimestral ou após feedback de CSAT/NPS
