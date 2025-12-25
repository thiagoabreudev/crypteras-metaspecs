---
spec_version: "1.1.0"
valid_from: "2025-12-25"
last_updated: "2025-12-25"
supersedes: null
status: "active"
category: "business"
tags: ['business', 'customer_personas']
---

# Personas do Cliente - Crypteras

:::version_info
**Versão**: 1.1.0
**Válida desde**: 2025-12-25
**Status**: Ativa
:::

:::intent
**Goal**: Definir personas de usuários target para guiar UX, messaging e priorização de features.

**Constraints** (limites obrigatórios):
- Basear em dados reais de usuários (não apenas assumptions)
- Atualizar conforme aprendemos mais sobre usuários
- Incluir jobs-to-be-done de cada persona
- Priorizar personas por volume/receita potencial

**Non-Goals** (o que NÃO fazer):
- Criar mais de 3-4 personas (evitar fragmentação)
- Targetar personas muito nichadas (ex: traders profissionais)
- Otimizar para personas que não pagam (freeloader users)
:::

:::breaking_changes
**v1.1.0**:
- Adicionada seção Intent as Code
- Incrementada versão MINOR conforme MetaCerta

**v1.0.0** (baseline):
- Primeira versão versionada desta spec
- Define 3 personas principais: Lucas (Iniciante), Mariana (Intermediária), Rafael (Conservador)
- Estabelece framework de detecção automática de persona
:::

---

## Visão Geral

Este documento define as personas principais de clientes do Crypteras, baseado em análise de mercado, posicionamento competitivo e estratégia de produto.

---

## Persona 1: Lucas - O Trader Iniciante Curioso

### Informações Demográficas

- **Idade**: 25-32 anos
- **Localização**: Capitais brasileiras (SP, RJ, BH, Porto Alegre)
- **Ocupação**: Profissional de TI, Engenheiro, Designer, Marketing Digital
- **Renda**: R$ 5.000 - R$ 12.000/mês
- **Educação**: Superior completo ou em andamento
- **Experiência em Cripto**: 6-18 meses (iniciante/intermediário)

### Contexto Tecnológico

- **Familiaridade com Tech**: Alta - trabalha com tecnologia diariamente
- **Exchanges que usa**: Mercado Bitcoin (principal), começando a explorar Binance
- **Investimento atual em cripto**: R$ 2.000 - R$ 8.000
- **Tempo disponível**: 30-60 min/dia para monitorar mercado
- **Dispositivos**: Smartphone (Android/iOS) + Notebook

### Objetivos e Motivações

**Objetivo Principal**:
> "Automatizar meu trading para não perder oportunidades quando estou trabalhando"

**Motivações Secundárias**:
1. Gerar renda passiva complementar
2. Aprender estratégias profissionais de trading
3. Reduzir decisões emocionais (FOMO, panic selling)
4. Diversificar investimentos além de ações/fundos

**Métricas de Sucesso Pessoal**:
- Lucro mensal consistente de 3-5%
- Reduzir tempo de monitoramento para < 15 min/dia
- Não perder mais de 2% em um único trade

### Pontos de Dor

**Principais Frustrações**:
1. **Perda de oportunidades**: "Preço do BTC caiu 3% às 2h da manhã e eu estava dormindo"
2. **Decisões emocionais**: "Vendi no pânico e perdi 5% que poderia ter recuperado"
3. **Falta de conhecimento técnico**: "Não sei interpretar RSI, MACD e outros indicadores"
4. **Ferramentas complexas**: "3Commas é muito complicado e caro (US$ 49/mês)"
5. **Medo de configurar errado**: "E se eu errar nos parâmetros e perder tudo?"

**Objeções Típicas**:
- "Bots são seguros? Não vão roubar meu dinheiro?"
- "Como sei se o bot está funcionando corretamente?"
- "Preciso aprender programação para configurar?"
- "E se o bot travar e eu perder uma oportunidade?"

### Comportamento de Compra

**Jornada de Decisão**:
1. Descoberta: Vê anúncio/post sobre trading automatizado
2. Pesquisa: Compara 3Commas, Cryptohopper, Crypteras
3. Trial: Testa plano FREE por 2-4 semanas
4. Decisão: Converte para PRO se tiver 3+ trades lucrativos

**Gatilhos de Conversão FREE → PRO**:
- Atingir limite de 1 bot por estratégia no FREE
- Ver resultados positivos nos primeiros 15 dias
- Querer testar estratégias de candles com IA
- Recomendação de outro usuário/comunidade

**Objeções ao Upgrade**:
- "R$ 19,90/mês é caro se eu não tiver lucro"
- "Prefiro fazer manual ainda, estou aprendendo"
- "Não preciso de 3 bots ainda"

### Preferências de Comunicação

**Tom de Voz Preferido**: Amigável, educativo, sem jargões excessivos
**Canais Preferidos**:
- Email (verificação diária)
- WhatsApp (notificações urgentes)
- Tutoriais em vídeo (YouTube)

**Tipos de Conteúdo que Consome**:
- "Como criar seu primeiro DCABot em 5 minutos"
- "3 erros que iniciantes cometem com bots de trading"
- "Comparativo: Crypteras vs 3Commas - Qual vale mais a pena?"

**Perguntas Frequentes**:
1. "Qual estratégia é melhor para iniciantes: DCA ou Candle?"
2. "Quanto dinheiro preciso para começar?"
3. "Como configurar stop-loss para não perder muito?"
4. "O bot funciona 24/7 mesmo se eu desligar o computador?"

### Notas para Interação com IA

**Prioridades ao Interagir**:
- Simplicidade nas explicações (evitar termos técnicos)
- Validação emocional ("É normal ter dúvidas, vou te ajudar")
- Exemplos práticos com números reais
- Reassegurar sobre segurança e controle

**Gatilhos de Escalonamento**:
- Dúvidas sobre segurança das credenciais → Escalar para documentação de segurança
- Erros técnicos recorrentes → Escalar para suporte técnico
- Interesse em arbitragem/backtesting → Educá-lo sobre roadmap futuro

**Oportunidades de Upsell**:
- Quando atinge limite de 1 bot no FREE → Sugerir PRO
- Após 3 trades lucrativos → "Parabéns! Quer potencializar com mais bots?"
- Quando pergunta sobre indicadores avançados → Mostrar benefícios de CandleBots

---

## Persona 2: Mariana - A Trader Intermediária Ambiciosa

### Informações Demográficas

- **Idade**: 28-38 anos
- **Localização**: Grandes centros urbanos (SP, RJ, DF)
- **Ocupação**: Analista Financeira, Trader Autônomo, Empreendedora
- **Renda**: R$ 10.000 - R$ 25.000/mês
- **Educação**: Superior completo (Economia, Administração, TI)
- **Experiência em Cripto**: 2-4 anos (intermediária/avançada)

### Contexto Tecnológico

- **Familiaridade com Tech**: Muito alta - usa APIs, ferramentas avançadas
- **Exchanges que usa**: Binance (principal), Mercado Bitcoin, OKX
- **Investimento atual em cripto**: R$ 30.000 - R$ 150.000
- **Tempo disponível**: 2-4h/dia para trading ativo
- **Dispositivos**: Múltiplos monitores, smartphone com alertas

### Objetivos e Motivações

**Objetivo Principal**:
> "Escalar minha operação gerenciando múltiplas estratégias simultaneamente em diferentes exchanges"

**Motivações Secundárias**:
1. Maximizar lucros com arbitragem multi-exchange
2. Reduzir risco com diversificação de estratégias
3. Automatizar tarefas repetitivas (rebalanceamento)
4. Testar estratégias antes de alocar capital real (backtesting)

**Métricas de Sucesso Pessoal**:
- ROI mensal de 8-15%
- Sharpe Ratio > 1.5
- Drawdown máximo < 10%
- 10+ estratégias rodando simultaneamente

### Pontos de Dor

**Principais Frustrações**:
1. **Custo de ferramentas**: "3Commas Pro custa US$ 99/mês - muito caro para o Brasil"
2. **Falta de backtesting**: "Não posso testar estratégias sem arriscar capital"
3. **Gestão manual multi-exchange**: "Fico alternando entre 3 abas de exchanges"
4. **Falta de arbitragem automatizada**: "Vejo oportunidades mas executo tarde demais"
5. **Comunidade fragmentada**: "Não tenho onde compartilhar ou copiar estratégias vencedoras"

**Objeções Típicas**:
- "Crypteras tem todos os recursos que 3Commas oferece?"
- "Posso rodar 20+ bots simultaneamente sem travar?"
- "E se a plataforma cair no meio de um trade?"
- "Tem API para eu integrar minhas próprias análises?"

### Comportamento de Compra

**Jornada de Decisão**:
1. Descoberta: Busca ativamente por alternativas ao 3Commas
2. Comparação: Testa FREE por 1 semana (valida funcionalidades críticas)
3. Upgrade direto para PRO: Se Crypteras atender 70% dos recursos
4. Considera MAX: Quando roadmap incluir backtesting + marketplace

**Gatilhos de Conversão FREE → PRO**:
- Confirmar que múltiplos bots funcionam bem
- Ver centralização de carteira multi-exchange
- Validar estabilidade da plataforma (zero crashes em 7 dias)

**Gatilhos de Conversão PRO → MAX**:
- Lançamento de backtesting (Prioridade #1 no roadmap)
- Disponibilidade de marketplace de estratégias
- Necessidade de rodar 10+ bots simultaneamente

**Objeções ao Upgrade**:
- "R$ 97/mês MAX ainda é mais barato que 3Commas, mas o que mais desbloqueia?"
- "Prefiro esperar backtesting antes de pagar MAX"

### Preferências de Comunicação

**Tom de Voz Preferido**: Profissional, direto, data-driven
**Canais Preferidos**:
- Email (resumos semanais de performance)
- Discord/Telegram (comunidade de traders)
- API webhooks (alertas customizados)

**Tipos de Conteúdo que Consome**:
- "Análise técnica: Como otimizar CandleBots com RSI+MACD"
- "Case study: 15% ROI mensal com estratégia mista DCA+Candle"
- "Roadmap: Quando vem backtesting e arbitragem?"

**Perguntas Frequentes**:
1. "Posso conectar múltiplas contas na mesma exchange?" (ex: 2 contas Binance)
2. "Como exportar histórico de trades para análise externa?"
3. "Tem rate limiting? Quantas requisições/min suporta?"
4. "Quando vem integração com TradingView?"

### Notas para Interação com IA

**Prioridades ao Interagir**:
- Respostas diretas com dados (evitar "vendas" excessivas)
- Transparência sobre limitações atuais vs roadmap
- Acesso antecipado a features beta (early adopter profile)
- Reconhecimento de expertise ("Vejo que você já domina isso...")

**Gatilhos de Escalonamento**:
- Solicitações de features não disponíveis → Coletar feedback para roadmap
- Performance abaixo do esperado → Análise técnica detalhada
- Interesse em API/integrações → Documentação técnica avançada

**Oportunidades de Upsell**:
- Quando roda 3 bots consistentemente no PRO → "Precisa de mais? MAX oferece ilimitado"
- Ao perguntar sobre backtesting → "Está no roadmap para MAX tier, quer reservar acesso early bird?"
- Quando demonstra expertise → "Quer vender suas estratégias no marketplace futuro?"

---

## Persona 3: Rafael - O Investidor Conservador

### Informações Demográficas

- **Idade**: 35-50 anos
- **Localização**: Diversas regiões do Brasil
- **Ocupação**: Médico, Advogado, Empresário, Executivo
- **Renda**: R$ 20.000 - R$ 60.000/mês
- **Educação**: Pós-graduação
- **Experiência em Cripto**: 1-2 anos (iniciante cauteloso)

### Contexto Tecnológico

- **Familiaridade com Tech**: Média - usa apps bancários, mas evita complexidade
- **Exchanges que usa**: Mercado Bitcoin (confiança em plataforma brasileira)
- **Investimento atual em cripto**: R$ 50.000 - R$ 300.000
- **Tempo disponível**: 10-20 min/semana para revisar performance
- **Dispositivos**: iPhone, iPad

### Objetivos e Motivações

**Objetivo Principal**:
> "Diversificar patrimônio com cripto de forma segura e sem precisar virar especialista"

**Motivações Secundárias**:
1. Hedge contra inflação (alternativa à renda fixa)
2. Preservação de capital (foco em não perder, não em ganhos agressivos)
3. Automatização total (delegar decisões a bots confiáveis)
4. Recomendações de assessor financeiro/amigos de confiança

**Métricas de Sucesso Pessoal**:
- Não perder mais de 5% do capital em nenhum mês
- Retorno anual de 8-12% (acima da poupança)
- Zero horas de monitoramento ativo

### Pontos de Dor

**Principais Frustrações**:
1. **Medo de perder dinheiro**: "Cripto é muito volátil, já perdi 15% em um dia"
2. **Falta de tempo**: "Não tenho tempo para ficar olhando gráficos"
3. **Desconfiança em plataformas**: "E se a plataforma quebrar e sumir com meu dinheiro?"
4. **Complexidade técnica**: "Não entendo nada de RSI, MACD, Fibonacci..."
5. **FOMO e notícias**: "Vejo Bitcoin subindo e não sei se devo comprar agora"

**Objeções Típicas**:
- "Como sei que vocês não vão roubar minhas credenciais de API?"
- "E se o bot errar e vender tudo no prejuízo?"
- "Prefiro deixar meu dinheiro no Tesouro Direto, é mais seguro"
- "R$ 19,90/mês não parece muito, mas e se eu perder R$ 5.000 por erro do bot?"

### Comportamento de Compra

**Jornada de Decisão**:
1. Descoberta: Recomendação de amigo/assessor financeiro
2. Validação: Pesquisa reviews, busca "Crypteras confiável?" no Google
3. Trial: Testa FREE com valor pequeno (R$ 500-1.000) por 1 mês
4. Decisão: Converte para PRO se tiver lucro de 2%+ no primeiro mês SEM perdas > 3%

**Gatilhos de Conversão FREE → PRO**:
- Segurança comprovada (sem crashes, sem bugs)
- Lucro pequeno mas consistente (2-3% ao mês)
- Suporte responsivo e atencioso (respondeu em < 24h)
- Depoimentos de outros usuários conservadores

**Objeções ao Upgrade**:
- "Vou esperar mais 2-3 meses no FREE para ter certeza"
- "R$ 19,90/mês × 12 = R$ 238,80/ano - será que vale a pena?"
- "Prefiro fazer DCA manual uma vez por semana"

### Preferências de Comunicação

**Tom de Voz Preferido**: Educado, reassegurador, transparente, sem pressão de vendas
**Canais Preferidos**:
- Email (revisões mensais de performance)
- WhatsApp (alertas críticos de risco)

**Tipos de Conteúdo que Consome**:
- "5 dicas de segurança para proteger suas criptomoedas"
- "Como funciona o DCABot: estratégia de Warren Buffett aplicada a cripto"
- "Depoimento: Como ganhei 10% ao ano com trading automatizado sem estresse"

**Perguntas Frequentes**:
1. "Qual estratégia é mais segura: DCA ou Candle?"
2. "Posso definir um limite máximo de perda por mês?"
3. "O que acontece se vocês fecharem a empresa? Perco meu dinheiro?"
4. "Como faço para sacar tudo e parar de usar?"

### Notas para Interação com IA

**Prioridades ao Interagir**:
- Reassegurar constantemente sobre segurança
- Explicar riscos de forma clara e honesta
- Evitar promessas de retorno ("Não garantimos lucro, mas...")
- Validar decisão de investir com cautela

**Gatilhos de Escalonamento**:
- Dúvidas sobre segurança/privacidade → FAQ de segurança + contato humano
- Perdas acima de 5% → Análise de estratégia + ajuste de parâmetros
- Solicitação de cancelamento → Entender motivo + oferecer pausa temporária

**Oportunidades de Upsell**:
- Após 6 meses de uso consistente → "Muitos usuários como você expandem para PRO"
- Quando demonstra confiança → "Quer conhecer estratégias um pouco mais agressivas?"
- Nunca pressionar - deixar ele tomar decisão no tempo dele

---

## Matriz de Priorização de Personas

| Persona | Prioridade | % da Base Esperada | Valor Lifetime (LTV) | Esforço de Aquisição |
|---------|------------|-------------------|----------------------|----------------------|
| **Lucas (Iniciante)** | Alta | 60% | Médio (R$ 600-1.200/ano) | Baixo (conteúdo educacional) |
| **Mariana (Intermediária)** | Muito Alta | 30% | Alto (R$ 1.200-2.400/ano) | Médio (features avançadas) |
| **Rafael (Conservador)** | Média | 10% | Alto (R$ 1.200+/ano) | Alto (construção de confiança) |

---

## Insights para IA

### Detecção Automática de Persona

**Sinais de "Lucas" (Iniciante)**:
- Pergunta básicas ("O que é DCA?", "Como funciona stop-loss?")
- Primeiro bot criado
- Investimento < R$ 10.000
- Hesitação em configurações avançadas

**Sinais de "Mariana" (Intermediária)**:
- Pergunta sobre APIs, webhooks, integrações
- Cria múltiplos bots rapidamente
- Menciona outras ferramentas (3Commas, TradingView)
- Pede features não disponíveis (backtesting, arbitragem)

**Sinais de "Rafael" (Conservador)**:
- Perguntas sobre segurança, regulação, riscos
- DCABot como primeira escolha
- Demora para criar segundo bot
- Menciona Tesouro Direto, renda fixa, diversificação

### Personalização de Respostas

**Para Lucas**:
- Tone: Amigável, educativo
- Formato: Passo-a-passo, com exemplos
- CTA: "Quer que eu te ajude a configurar seu primeiro bot?"

**Para Mariana**:
- Tone: Profissional, direto
- Formato: Bullet points, dados técnicos
- CTA: "Vejo que você domina isso. Quer saber quando lançamos arbitragem?"

**Para Rafael**:
- Tone: Reassegurador, transparente
- Formato: Riscos + benefícios balanceados
- CTA: "Posso sugerir parâmetros conservadores para começar?"

---

## Versionamento

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 2024-12-24 | Criação inicial com 3 personas principais |

---

**Próxima Revisão**: Após primeiros 50 usuários (validação empírica de personas)
