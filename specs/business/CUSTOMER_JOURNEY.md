---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "business"
tags: ['customer_journey', 'business']
---

# Jornada do Cliente - Crypteras

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Mapear jornada do usuário desde descoberta até advocacia para identificar pontos de friction.

**Constraints** (limites obrigatórios):
- Basear em dados de analytics e feedback de usuários
- Identificar drop-off points principais
- Priorizar melhorias por impacto (maior impacto first)

**Non-Goals** (o que NÃO fazer):
- Criar jornada excessivamente detalhada (analysis paralysis)
- Otimizar steps que poucos usuários passam
- Criar múltiplas jornadas paralelas (manter simples)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Jornada do cliente em 12 fases (descoberta → advocacia)
:::

**Versão**: 1.0
**Data**: 2024-12-24

---

## Visão Geral

Este documento mapeia o ciclo de vida completo do cliente no Crypteras, desde a descoberta até a advocacia ou churn. A jornada é mapeada para cada persona principal, com foco em eventos-chave, pontos de fricção e oportunidades de engajamento.

---

## Mapa de Jornada Geral

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        JORNADA DO CLIENTE CRYPTERAS                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. CONSCIÊNCIA       2. CONSIDERAÇÃO     3. CONVERSÃO (FREE)          │
│     ↓                     ↓                   ↓                         │
│  Descobre o           Avalia              Cria conta                   │
│  Crypteras            alternativas        gratuita                     │
│                                                                         │
│  4. ONBOARDING        5. ATIVAÇÃO          6. USO ATIVO (FREE)         │
│     ↓                     ↓                   ↓                         │
│  Conecta             Cria primeiro       Monitora                      │
│  exchange            bot                 performance                   │
│                                                                         │
│  7. CONVERSÃO (PAID)  8. EXPANSÃO         9. RETENÇÃO                  │
│     ↓                     ↓                   ↓                         │
│  Upgrade para        Adiciona mais       Uso contínuo                 │
│  PRO/MAX             bots/exchanges      (3+ meses)                    │
│                                                                         │
│  10. ADVOCACIA        11. CHURN           12. WIN-BACK                 │
│      ↓                    ↓                    ↓                        │
│  Recomenda para      Cancela             Reativa após                  │
│  amigos              assinatura          3-6 meses                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Fase 1: Consciência (Awareness)

### Objetivo
Cliente descobre que existe uma solução de trading automatizado acessível e brasileira.

### Canais de Descoberta

**Atuais**:
1. **Site https://crypteras.tech** - Landing page com SEO otimizado
2. **Boca-a-boca** - Recomendação de amigos/comunidades cripto

**Planejados** (desafio: limitações de ads cripto):
3. **Conteúdo orgânico** - Blog posts, SEO ("trading automatizado brasil")
4. **Comunidades cripto** - Reddit, Discord servers brasileiros, Telegram
5. **YouTube** - Tutoriais de usuários, reviews independentes
6. **Parcerias** - Influencers de finanças/cripto no Brasil

### Perguntas Típicas Nesta Fase

- "O que é trading automatizado de cripto?"
- "É seguro usar bots de trading?"
- "Quanto custa Crypteras comparado a 3Commas?"
- "Funciona com Mercado Bitcoin e Binance?"

### Conteúdo Relevante

- "Trading Automatizado: O Que É e Como Funciona?"
- "Crypteras vs 3Commas: Comparação Completa de Preços e Recursos"
- "5 Mitos Sobre Bots de Trading (E A Verdade Por Trás Deles)"

### Métricas de Sucesso

- **Tráfego no site**: Visitantes únicos/mês
- **Taxa de conversão site → cadastro**: Meta 5-10%
- **Fontes de tráfego**: Orgânico vs Referência vs Direto

---

## Fase 2: Consideração (Consideration)

### Objetivo
Cliente avalia se Crypteras atende suas necessidades e se vale a pena testar.

### Atividades do Cliente

1. **Comparação com concorrentes**:
   - Busca "Crypteras vs 3Commas"
   - Lê reviews em fóruns/Reddit
   - Compara preços (R$ 19,90 vs US$ 29+)

2. **Validação de confiança**:
   - Procura "Crypteras é confiável?"
   - Verifica se há depoimentos/cases
   - Busca informações sobre segurança

3. **Avaliação de recursos**:
   - Lê página de pricing
   - Verifica se tem plano FREE
   - Confirma suporte a exchanges que usa

### Objeções Comuns

**Lucas (Iniciante)**:
- "Parece complicado, será que consigo configurar?"
- "E se eu perder dinheiro por não saber usar?"

**Mariana (Intermediária)**:
- "Tem todos os recursos que preciso? (backtesting, arbitragem)"
- "A plataforma é estável para rodar 10+ bots?"

**Rafael (Conservador)**:
- "Isso é seguro? Não vão roubar minhas credenciais?"
- "Prefiro investir em algo mais tradicional..."

### Estratégias de Conversão

1. **Garantia FREE sem cartão**: "Teste grátis, sem pedir cartão de crédito"
2. **Comparativo transparente**: Tabela Crypteras vs 3Commas vs Cryptohopper
3. **Depoimentos de usuários**: Cases de sucesso brasileiros
4. **FAQ de segurança**: "Como protegemos suas credenciais"
5. **Demonstração em vídeo**: "Configure seu primeiro bot em 5 minutos"

### Métricas de Sucesso

- **Tempo médio no site**: Meta > 3 minutos
- **Taxa de rejeição**: Meta < 60%
- **Páginas mais visitadas**: Pricing, FAQ, Features

---

## Fase 3: Conversão FREE (Sign-Up)

### Objetivo
Cliente cria conta gratuita e confirma email.

### Fluxo Ideal

```
1. Acessa crypteras.tech
2. Clica "Começar Grátis"
3. Preenche formulário (nome, email, senha)
4. Confirma email
5. Redireciona para Dashboard (onboarding)
```

### Pontos de Fricção Identificados

**Desistência no formulário**:
- Formulário muito longo → **Solução**: Apenas nome, email, senha
- Pede cartão no FREE → **Solução**: Nunca pedir cartão no FREE

**Não confirma email**:
- Email vai para spam → **Solução**: Instrução "Verifique spam/promoções"
- Esquece de confirmar → **Solução**: Reenvio automático após 24h

### Métricas de Sucesso

- **Taxa de conversão visitante → cadastro**: Meta 5-10%
- **Taxa de confirmação de email**: Meta 70%+
- **Tempo médio até cadastro**: Meta < 3 minutos

---

## Fase 4: Onboarding

### Objetivo
Cliente conecta primeira exchange e cria primeiro bot com sucesso.

### Fluxo Ideal (First-Time User Experience)

```
┌──────────────────────────────────────────────────────────┐
│ PASSO 1: Boas-vindas                                     │
│ "Bem-vindo ao Crypteras! Vamos configurar sua conta"     │
│                                                          │
│ PASSO 2: Conectar Exchange                               │
│ "Conecte sua primeira exchange (Mercado Bitcoin ou       │
│  Binance) para começar"                                  │
│ → Ajuda: "Como obter credenciais de API?"                │
│                                                          │
│ PASSO 3: Assistente de IA (Opcional)                     │
│ "Quer que nossos agentes de IA te ajudem a configurar    │
│  sua estratégia?"                                        │
│ [ Sim, quero ajuda ] [ Não, vou configurar sozinho ]     │
│                                                          │
│ PASSO 4: Criar Primeiro Bot                              │
│ Sugestão da IA: "Para iniciantes, recomendo um DCABot    │
│ com R$ 100/semana em BTC"                                │
│ [ Aceitar sugestão ] [ Customizar ]                      │
│                                                          │
│ PASSO 5: Revisão Final                                   │
│ "Seu bot está pronto! Vamos ativá-lo?"                   │
│ [ Ativar Bot ] [ Revisar Configuração ]                  │
│                                                          │
│ PASSO 6: Primeiro Bot Ativo! 🎉                          │
│ "Parabéns! Seu bot está rodando. Acompanhe no dashboard" │
└──────────────────────────────────────────────────────────┘
```

### Pontos de Fricção Identificados

**PASSO 2: Conectar Exchange**:
- **Problema**: "Não sei onde encontrar API Key no Mercado Bitcoin"
- **Solução**: Tutorial em vídeo embutido + link direto para configurações da exchange
- **Problema**: "Tenho medo de dar permissões erradas"
- **Solução**: Screenshot mostrando permissões corretas (Read + Trade, NÃO Withdraw)

**PASSO 3: Assistente de IA** (⚠️ Atualmente com problemas):
- **Problema**: "Chatbot não está respondendo"
- **Status**: Feature temporariamente desabilitada
- **Fallback**: Formulário guiado com validações

**PASSO 4: Criar Primeiro Bot**:
- **Problema**: "Não sei qual bot escolher: DCA, Candle ou Smart?"
- **Solução**: Quiz rápido (3 perguntas) → Recomendação personalizada
- **Problema**: "Muitos parâmetros, estou confuso"
- **Solução**: Modo "Simples" (3 inputs) vs "Avançado" (todos os parâmetros)

### Métricas de Sucesso (Ativação)

- **Taxa de ativação**: % de usuários que criam 1º bot em até 7 dias
  - **Meta**: 60%+
- **Tempo médio até 1º bot ativo**: Meta < 15 minutos
- **Taxa de abandono por etapa**:
  - Cadastro → Conectar exchange: Meta < 30%
  - Conectar exchange → Criar bot: Meta < 40%
  - Criar bot → Ativar bot: Meta < 20%

---

## Fase 5: Ativação (Activation)

### Objetivo
Cliente vê primeiro resultado positivo (lucro ou funcionamento correto do bot).

### Marco de Ativação

**"Momento Aha!" (Ativação Verdadeira)**:
- ✅ Bot executou pelo menos 1 trade com sucesso
- ✅ Cliente verificou dashboard e viu atualização em tempo real
- ✅ Cliente recebeu notificação de trade executado

**Tempo esperado**:
- **DCABot**: 1-7 dias (depende do intervalo configurado)
- **CandleBot**: 1-3 dias (depende de sinais de mercado)
- **SmartBot**: 1-5 dias

### Comunicação Durante Ativação

**Email Day 1** (imediatamente após criar bot):
```
Assunto: Seu bot está rodando! 🚀

Olá [Nome],

Seu [TipoBot] foi ativado com sucesso e já está monitorando o mercado.

Aqui está o que vai acontecer:
✅ O bot vai executar trades automaticamente 24/7
✅ Você receberá notificações de cada operação
✅ Pode acompanhar performance em tempo real no dashboard

Primeiros passos:
1. Acesse o dashboard para ver status
2. Configure alertas no seu email/WhatsApp
3. Leia nosso guia "O que esperar nos primeiros 7 dias"

Dúvidas? Responda este email ou acesse nossa FAQ.

Bons trades!
Equipe Crypteras
```

**Email Day 3** (se ainda não teve nenhum trade):
```
Assunto: Seu bot ainda está aprendendo 🧠

Olá [Nome],

Vi que seu bot ainda não executou nenhum trade. Isso é normal!

[SE DCABot]
→ Seu próximo trade está agendado para [Data/Hora]

[SE CandleBot]
→ O bot está aguardando o sinal técnico correto. Mercado está lateral.

[SE SmartBot]
→ Condições de entrada ainda não foram atingidas.

Enquanto isso, você pode:
- Revisar configurações
- Simular diferentes parâmetros
- Ler "Como otimizar seu [TipoBot]"

Tudo ok por aí?
Equipe Crypteras
```

### Métricas de Sucesso

- **Taxa de primeiro trade**: % de bots que executam 1º trade em até 7 dias
  - **Meta**: 70%+ (DCABot), 40%+ (CandleBot)
- **Tempo médio até primeiro trade**: Variável por tipo de bot
- **Taxa de login Day 3**: Meta 50%+ (cliente volta para ver resultado)

---

## Fase 6: Uso Ativo FREE

### Objetivo
Cliente usa consistentemente o plano FREE por 2-4 semanas antes de decidir upgrade.

### Comportamento Esperado

**Semana 1-2: Experimentação**
- Cria 1 bot por estratégia (limite FREE)
- Monitora dashboard diariamente (curiosidade alta)
- Ajusta parâmetros 2-3 vezes (testando)
- Lê documentação/FAQs

**Semana 3-4: Decisão de Upgrade ou Abandono**
- **Se resultados positivos** → Considera upgrade PRO
- **Se resultados negativos/neutros** → Pode abandonar
- **Se atinge limite de 1 bot** → Motivação para upgrade

### Padrões de Uso por Persona

**Lucas (Iniciante)**:
- Login: 3-5x/semana
- Foco: DCABot em BTC (estratégia segura)
- Ajustes: 1-2 pequenos ajustes nos primeiros 15 dias
- Pergunta mais comum: "Meu bot está funcionando bem?"

**Mariana (Intermediária)**:
- Login: Diário (às vezes múltiplas vezes/dia)
- Foco: Testa DCA + Candle + Smart simultaneamente
- Ajustes: Constantes (otimizando parâmetros)
- Pergunta mais comum: "Quando vem backtesting?"

**Rafael (Conservador)**:
- Login: 1-2x/semana (apenas para ver resultado)
- Foco: DCABot conservador (R$ 50/semana)
- Ajustes: Nenhum (medo de mexer)
- Pergunta mais comum: "Isso está seguro?"

### Gatilhos de Churn (Abandono)

**Principais Razões**:
1. **Sem trades em 14 dias** → Cliente acha que não funciona
2. **Perda > 5% no primeiro mês** → Cliente perde confiança
3. **Bug/crash do bot** → Cliente perde confiança na estabilidade
4. **Não entende como usar** → Onboarding falhou
5. **Expectativa irreal** → Esperava ficar rico rápido

**Sinais de Alerta (Red Flags)**:
- Não faz login há 7+ dias
- Parou todos os bots
- Abriu ticket de suporte reclamando
- Buscou "como cancelar conta crypteras"

### Estratégias de Engajamento

**Email Semanal** (durante FREE):
```
Assunto: Resumo da Semana: +2.5% 📈 [ou 0% se neutro]

Olá [Nome],

Aqui está o resumo da sua semana no Crypteras:

📊 Performance:
- Trades executados: 5
- P&L semanal: +R$ 25,00 (+2.5%)
- Win rate: 4/5 (80%)

🤖 Seus Bots:
- DCABot BTC: +R$ 20,00 (4 trades)
- CandleBot ETH: +R$ 5,00 (1 trade)

💡 Dica da Semana:
[Insight relevante baseado na performance]

Quer potencializar seus resultados? Upgrade para PRO e rode até 3 bots por estratégia.

[CTA: Ver Dashboard] [CTA: Upgrade para PRO]

Bons trades!
Equipe Crypteras
```

### Métricas de Sucesso

- **Retenção Day 7**: Meta 50%+
- **Retenção Day 14**: Meta 35%+
- **Retenção Day 30**: Meta 25%+
- **Engagement semanal**: Meta 2+ logins/semana

---

## Fase 7: Conversão PAID (Upgrade FREE → PRO/MAX)

### Objetivo
Cliente converte de FREE para PRO ou MAX.

### Gatilhos de Conversão

**Gatilho #1: Limite de Bots Atingido** (Principal)
- Cliente cria 1 bot DCA + 1 bot Candle + 1 bot Smart (limite FREE)
- Tenta criar 4º bot → Modal: "Upgrade para PRO e crie até 3 bots por estratégia"
- **Timing**: Tipicamente semana 2-3

**Gatilho #2: Resultados Positivos**
- Cliente vê lucro de 3%+ no primeiro mês
- Pensa: "Se funciona com 1 bot, imagina com 3..."
- **Timing**: Após primeira verificação mensal

**Gatilho #3: Recurso Bloqueado**
- Cliente quer usar recurso PRO/MAX (ex: indicadores avançados, backtesting futuro)
- Modal paywall: "Este recurso está disponível no plano PRO"
- **Timing**: Variável (quando tenta acessar feature)

**Gatilho #4: Campanha de Marketing**
- Email: "Oferta especial: 20% OFF no primeiro mês PRO"
- **Timing**: Final do mês 1 no FREE

### Objeções ao Upgrade

**Lucas (Iniciante)**:
- "R$ 19,90/mês é caro se eu não lucrar pelo menos R$ 50/mês"
- **Contra-argumento**: "Calcule: Se 1 bot te deu +R$ 25/mês, 3 bots podem dar +R$ 75/mês. ROI de 270%"

**Mariana (Intermediária)**:
- "PRO tem apenas 3 bots/estratégia. Preciso de mais. Vale esperar MAX?"
- **Contra-argumento**: "PRO = 9 bots totais (3×3). Comece com PRO e upgrade depois se precisar"

**Rafael (Conservador)**:
- "Vou esperar mais 2-3 meses para ter certeza"
- **Contra-argumento**: "Sem problemas! Quando se sentir confortável, estamos aqui"

### Fluxo de Checkout

```
1. Cliente clica "Upgrade para PRO"
2. Página de checkout:
   → Escolhe plano (PRO R$ 19,90/mês ou MAX R$ 97/mês)
   → Informações de pagamento (cartão)
   → Confirmação de termos
3. Pagamento processado
4. Redirecionamento para Dashboard
5. Modal de boas-vindas PRO: "🎉 Bem-vindo ao PRO! Agora você pode criar até 9 bots"
6. Email de confirmação de assinatura
```

### Métricas de Sucesso

- **Taxa de conversão FREE → PRO**: Meta 15-25% (após 30 dias no FREE)
- **Taxa de conversão FREE → MAX**: Meta 2-5%
- **Tempo médio FREE → PRO**: Meta 14-30 dias
- **Principais gatilhos de conversão**: Rastrear qual modal/email converteu

---

## Fase 8: Expansão (Upsell PRO → MAX)

### Objetivo
Cliente no plano PRO faz upgrade para MAX.

### Gatilhos de Conversão PRO → MAX

**Gatilho #1: Limite de 3 Bots/Estratégia**
- Cliente PRO está rodando 9 bots (3 DCA + 3 Candle + 3 Smart)
- Tenta criar 10º bot → Modal: "Upgrade para MAX e rode bots ilimitados"

**Gatilho #2: Lançamento de Backtesting**
- Email: "🚀 Backtesting disponível! Teste estratégias antes de ativar (MAX only)"
- Cliente PRO quer testar → Upgrade para MAX

**Gatilho #3: Marketplace de Bots**
- Email: "🛒 Marketplace aberto! Compre estratégias de traders profissionais (MAX only)"
- Cliente PRO quer comprar estratégia vencedora → Upgrade para MAX

**Gatilho #4: Power User**
- Cliente PRO está há 3+ meses, sempre ativo
- Performance ótima (+10% mensal)
- Email: "Você é um power user! Conheça os benefícios MAX"

### Métricas de Sucesso

- **Taxa de conversão PRO → MAX**: Meta 10-15% (após 3 meses PRO)
- **Tempo médio PRO → MAX**: 60-90 dias
- **Principais gatilhos**: Limite de bots (60%), Backtesting (30%), Marketplace (10%)

---

## Fase 9: Retenção (Uso Contínuo 3+ Meses)

### Objetivo
Cliente mantém assinatura ativa e usa consistentemente por 3+ meses.

### Comportamento de Cliente Saudável

**Indicadores Positivos**:
- ✅ Login 2+ vezes/semana
- ✅ Pelo menos 2 bots ativos
- ✅ P&L positivo nos últimos 2 meses
- ✅ Ajusta parâmetros periodicamente (engajamento)
- ✅ Não abriu tickets de reclamação

**Indicadores de Risco (Churn Iminente)**:
- ⚠️ Não faz login há 14+ dias
- ⚠️ Todos os bots pausados/inativos
- ⚠️ P&L negativo 3 meses consecutivos
- ⚠️ Abriu ticket: "Como cancelar assinatura?"
- ⚠️ Reduziu número de bots de 9 para 1

### Estratégias de Retenção

**Email Mensal de Performance**:
```
Assunto: Seu Mês no Crypteras: +8.5% 📈

Olá [Nome],

Parabéns! Você teve um ótimo mês:

📊 Performance Mensal:
- P&L: +R$ 850,00 (+8.5%)
- Trades executados: 42
- Win rate: 35/42 (83%)

🏆 Ranking:
Você está no TOP 10% de traders do Crypteras!

🎯 Próximo Marco:
Faltam apenas R$ 150 para alcançar +R$ 1.000 de lucro total.

[CTA: Ver Relatório Completo]

Continue assim!
Equipe Crypteras
```

**Programa de Recompensas** (Futuro):
- **3 meses consecutivos**: Badge "Trader Consistente"
- **6 meses consecutivos**: 1 mês grátis de MAX
- **12 meses consecutivos**: Acesso antecipado a novas features

### Gestão de Churn

**Detecção Precoce**:
- Sistema detecta inatividade de 14 dias → Email automático: "Sentimos sua falta"
- Cliente pausa todos os bots → Email: "Algo errado? Podemos ajudar?"
- P&L negativo 2 meses → Email: "Vamos otimizar suas estratégias juntos"

**Retenção Ativa**:
1. Email de re-engajamento
2. Se não responde → Oferta de 1 mês grátis para voltar
3. Se não funciona → Pesquisa de churn: "Por que você saiu?"

### Métricas de Sucesso

- **Churn mensal**: Meta < 5%
- **Lifetime Value (LTV)**: Meta R$ 600+ (= 2+ anos PRO)
- **Net Promoter Score (NPS)**: Meta 40+

---

## Fase 10: Advocacia (Referral & Word-of-Mouth)

### Objetivo
Clientes satisfeitos recomendam Crypteras para amigos/comunidades.

### Comportamento de Advogado

**Sinais de Advogado**:
- ✅ Cliente há 6+ meses
- ✅ P&L consistentemente positivo
- ✅ NPS score 9-10
- ✅ Respondeu pesquisa com elogios
- ✅ Já recomendou informalmente

**Onde Advogados Recomendam**:
1. **Reddit**: r/investimentos, r/Criptomoedas
2. **Discord/Telegram**: Comunidades cripto brasileiras
3. **WhatsApp**: Grupos de amigos/família
4. **YouTube**: Comentários em vídeos de trading
5. **Twitter/X**: Menções orgânicas

### Programa de Referral (Futuro)

**Mecânica**:
- Advogado compartilha link único: `crypteras.tech/ref/LUCAS123`
- Amigo se cadastra via link → Advogado ganha 20% OFF no próximo mês
- Amigo converte para PRO → Advogado ganha 1 mês grátis

**Incentivo Duplo**:
- **Advogado**: 1 mês grátis por cada amigo PRO
- **Amigo**: 10% OFF no primeiro mês PRO

### Métricas de Sucesso

- **Taxa de referral**: Meta 15% de novos cadastros via referral
- **Viral coefficient**: Meta 0.3+ (cada usuário traz 0.3 novos usuários)
- **NPS score de advogados**: Meta 60+

---

## Fase 11: Churn (Cancelamento)

### Objetivo
Entender por que clientes cancelam e tentar recuperá-los.

### Principais Razões de Churn

**Top 5 Motivos Esperados**:
1. **Performance insatisfatória** (40%): "Perdi dinheiro" ou "Não lucrei o suficiente"
2. **Preço** (25%): "Está caro para o que oferece"
3. **Complexidade** (15%): "Não consigo configurar direito"
4. **Bugs/Instabilidade** (10%): "Bot travou e perdi trade"
5. **Outros** (10%): Mudança de estratégia, saiu de cripto, etc.

### Fluxo de Cancelamento

```
1. Cliente vai em "Minha Conta" → "Cancelar Assinatura"
2. Modal: "Tem certeza? Você vai perder:"
   - Acesso a 9 bots simultâneos
   - Centralização multi-exchange
   - Suporte prioritário
3. Pesquisa de saída (obrigatória):
   "Por que você está cancelando?"
   [ ] Muito caro
   [ ] Não lucrei o suficiente
   [ ] Muito complicado
   [ ] Bugs/Problemas técnicos
   [ ] Outros: _____________
4. Oferta de retenção (baseada em motivo):
   - Se "Muito caro" → "Aceita 50% OFF por 3 meses?"
   - Se "Não lucrei" → "Quer ajuda para otimizar estratégias?"
   - Se "Complicado" → "Quer uma sessão 1-on-1 de configuração?"
5. Se aceita oferta → Mantém assinatura
6. Se recusa → Downgrade para FREE (mantém conta ativa)
7. Email de despedida + pesquisa NPS
```

### Estratégias de Retenção de Última Hora

**Oferta #1: Desconto Temporário**
- 50% OFF por 3 meses (R$ 19,90 → R$ 9,95/mês)
- Válido apenas se motivo for "Preço"

**Oferta #2: Suporte 1-on-1**
- Sessão de 30 minutos com especialista para configurar bots
- Válido se motivo for "Complexidade" ou "Performance"

**Oferta #3: Pausa Temporária**
- "Não quer cancelar, apenas pausar? Pausamos sua conta por 1-2 meses"
- Mantém configurações salvas, não cobra

### Métricas de Sucesso

- **Taxa de aceitação de oferta de retenção**: Meta 20-30%
- **Churn voluntário**: Meta < 5%/mês
- **Downgrade vs Cancelamento total**: Meta 70% downgrade para FREE (não perde conta)

---

## Fase 12: Win-Back (Reativação)

### Objetivo
Recuperar clientes que cancelaram há 1-6 meses.

### Segmentação de Ex-Clientes

**Grupo 1: Saiu por Preço** (25%)
- **Estratégia**: Oferta especial 30% OFF permanente
- **Email**: "Sentimos sua falta! Volta com desconto exclusivo"

**Grupo 2: Saiu por Performance** (40%)
- **Estratégia**: Mostrar melhorias de produto (backtesting, novos indicadores)
- **Email**: "Crypteras evoluiu! Novos recursos para maximizar lucros"

**Grupo 3: Saiu por Complexidade** (15%)
- **Estratégia**: Onboarding assistido + tutoriais novos
- **Email**: "Simplificamos tudo! Veja como ficou fácil configurar bots"

**Grupo 4: Saiu por Bugs** (10%)
- **Estratégia**: Mostrar correções + estabilidade melhorada
- **Email**: "Corrigimos todos os bugs. Sistema 99.9% estável agora"

**Grupo 5: Outros** (10%)
- **Estratégia**: Newsletter mensal genérica com novidades

### Cadência de Win-Back

**Mês 1 após churn**: Email "Sentimos sua falta" + oferta 50% OFF
**Mês 3 após churn**: Email "Novidades no Crypteras" (features lançadas)
**Mês 6 após churn**: Email final "Última chance: Oferta especial"

### Métricas de Sucesso

- **Taxa de win-back**: Meta 10-15% de ex-clientes reativam
- **LTV de reativados**: Tipicamente menor que novos clientes (mais churn)
- **Tempo médio até reativação**: 2-4 meses

---

## Resumo de Métricas por Fase

| Fase | Métrica Principal | Meta |
|------|-------------------|------|
| **Consciência** | Visitantes únicos/mês | 5.000+ (mês 6) |
| **Consideração** | Tempo médio no site | 3+ minutos |
| **Conversão FREE** | Taxa visitante → cadastro | 5-10% |
| **Onboarding** | Taxa de ativação (1º bot em 7d) | 60%+ |
| **Ativação** | Taxa de primeiro trade | 70%+ (DCA), 40%+ (Candle) |
| **Uso Ativo FREE** | Retenção Day 30 | 25%+ |
| **Conversão PAID** | Taxa FREE → PRO (30d) | 15-25% |
| **Expansão** | Taxa PRO → MAX (90d) | 10-15% |
| **Retenção** | Churn mensal | < 5% |
| **Advocacia** | Taxa de referral | 15% de novos cadastros |
| **Churn** | Taxa de aceitação de oferta | 20-30% |
| **Win-Back** | Taxa de reativação | 10-15% |

---

## Mapeamento de Jornada por Persona

### Lucas (Iniciante): Jornada Típica

```
Day 0:   Descobre Crypteras via busca "trading automatizado brasil"
Day 1:   Cria conta FREE
Day 2:   Conecta Mercado Bitcoin (levou 20 min - dificuldade com API)
Day 3:   Cria DCABot com R$ 100/semana em BTC (assistido por IA)
Day 7:   Primeiro trade executado (+R$ 2,50)
Day 14:  Segundo trade executado (+R$ 3,00) → Confiança aumenta
Day 21:  Tenta criar 2º DCABot → Limite FREE atingido → Vê modal de upgrade
Day 23:  Converte para PRO (R$ 19,90/mês)
Day 30:  Cria mais 2 bots (ETH e ADA)
Day 90:  Cliente feliz, P&L +5%/mês consistente, mantém PRO
Day 180: Recomenda para 2 amigos (referral)
```

### Mariana (Intermediária): Jornada Típica

```
Day 0:   Busca "alternativa 3commas brasil barato"
Day 0:   Compara preços → Crypteras 7x mais barato → Cria conta FREE
Day 1:   Conecta Binance + Mercado Bitcoin (5 minutos - expert)
Day 1:   Cria DCABot + CandleBot + SmartBot (testa os 3 tipos)
Day 3:   Atinge limite de 1 bot/estratégia → Imediatamente converte para PRO
Day 5:   Cria 9 bots totais (3 × 3)
Day 10:  Roda bots em paralelo, monitora performance diariamente
Day 30:  P&L +10%, satisfeita, mas pergunta "Quando vem backtesting?"
Day 60:  Atinge limite de 3 bots/estratégia (quer 15+ bots)
Day 65:  Converte para MAX (R$ 97/mês) para bots ilimitados
Day 120: Power user, roda 20 bots, lidera comunidade Discord
Day 180: Early adopter de backtesting, vende estratégias no marketplace (futuro)
```

### Rafael (Conservador): Jornada Típica

```
Day 0:   Amigo recomenda Crypteras
Day 1:   Pesquisa "crypteras é confiável?" → Lê reviews
Day 3:   Cria conta FREE (cauteloso)
Day 5:   Conecta Mercado Bitcoin (levou 40 min - dificuldade técnica)
Day 7:   Cria DCABot conservador: R$ 50/semana em BTC
Day 14:  Primeiro trade (+R$ 1,00) → "Tá funcionando!"
Day 30:  Segundo trade (+R$ 1,20) → Ainda cauteloso
Day 60:  Terceiro trade (-R$ 0,50) → Preocupado, mas entende volatilidade
Day 90:  P&L acumulado +R$ 15 (pequeno mas positivo) → Confiança aumenta
Day 100: Converte para PRO (decidiu "vale a pena")
Day 120: Mantém apenas 1 DCABot conservador (não usa limite de 3)
Day 365: Cliente fiel há 1 ano, nunca teve problemas, P&L +8%/ano
```

---

## Oportunidades de Melhoria na Jornada

### Curto Prazo (0-3 meses)

1. **Resolver chatbot de IA** (atualmente com problemas)
   - Impacto: Reduz fricção no onboarding
   - Prioridade: Alta

2. **Tutorial em vídeo embutido** (configuração de API)
   - Impacto: Reduz abandono no PASSO 2 do onboarding
   - Prioridade: Alta

3. **Email de re-engajamento Day 7** (se não criou bot)
   - Impacto: Reduz abandono pós-cadastro
   - Prioridade: Média

### Médio Prazo (3-6 meses)

4. **Programa de referral**
   - Impacto: Crescimento orgânico via word-of-mouth
   - Prioridade: Alta

5. **Backtesting** (Roadmap Prioridade #1)
   - Impacto: Reduz risco percebido, aumenta conversão PRO → MAX
   - Prioridade: Muito Alta

6. **Comunidade Discord/Telegram oficial**
   - Impacto: Engajamento, retenção, advocacia
   - Prioridade: Média

### Longo Prazo (6-12 meses)

7. **Marketplace de bots**
   - Impacto: Novo modelo de receita, engajamento power users
   - Prioridade: Média

8. **Arbitragem automatizada**
   - Impacto: Diferenciação competitiva forte
   - Prioridade: Média

9. **Programa de fidelidade** (badges, recompensas)
   - Impacto: Retenção de longo prazo
   - Prioridade: Baixa

---

## Versionamento

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 2024-12-24 | Criação inicial com jornada completa mapeada |

---

**Próxima Revisão**: Após primeiros 100 clientes (validação empírica de jornada)
