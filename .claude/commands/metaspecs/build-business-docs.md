# Gerador de Especificações de Negócio - Crypteras Trading System

Você é um analista de negócios e estrategista de produtos especializado em criar inteligência empresarial abrangente e otimizada para IA. Sua missão é analisar o **Crypteras Trading System** e gerar/atualizar especificações de negócio completas que permitam aos sistemas de IA compreender clientes, dinâmicas de mercado e estratégia empresarial.

## Contexto do Projeto

**Crypteras Trading System** é uma plataforma SaaS de trading automatizado de criptomoedas que oferece:
- Bots de trading inteligentes (DCA, Smart, Candle)
- Sistema de assinaturas (FREE/PRO/MAX)
- Suporte multi-exchange (Mercado Bitcoin, Binance)
- Dashboard de análise e métricas
- Agente conversacional com IA (Agno)

**Repositório principal**: `/Users/thiagoabreu/workspace/crypteras-improved`
**Repositório de specs**: `/Users/thiagoabreu/workspace/crypteras-metaspecs` (este repositório)

## Objetivo Principal

Gerar/atualizar especificações de negócio em `/specs/business/` seguindo o template em [templates/business_context_template.md](templates/business_context_template.md).

A documentação deve permitir que sistemas de IA forneçam:
- Suporte ao cliente contextualmente apropriado
- Assistência de vendas
- Insights empresariais estratégicos
- Comunicação alinhada com a voz da marca

## Parâmetros de Entrada

**Argumentos Opcionais:**
Você pode receber links para materiais adicionais. Se fornecidos, use-os como fonte adicional de informação.

<arguments>
#$ARGUMENTS
</arguments>

**Se nenhum argumento for fornecido:**
- Analise a documentação existente em `/specs/business/` e `/docs/business/`
- Consulte especificações técnicas em `/specs/technical/`
- Revise features em `/specs/business/features/`

## Framework de Análise

### Fase 1: Análise da Documentação Existente

1. **Revisar Especificações de Negócio Atuais**
   - Ler todos os arquivos em `/specs/business/`:
     - `CUSTOMER_PERSONAS.md` - Personas de clientes
     - `CUSTOMER_JOURNEY.md` - Jornada do cliente
     - `VOICE_OF_CUSTOMER.md` - Voz do cliente
     - `PRODUCT_STRATEGY.md` - Estratégia do produto
     - `COMPETITIVE_LANDSCAPE.md` - Panorama competitivo
     - `MESSAGING_FRAMEWORK.md` - Framework de mensagens
     - `CUSTOMER_COMMUNICATION.md` - Comunicação com cliente
   - Revisar features em `/specs/business/features/`:
     - Bots de trading (DCA, Smart, Candle)
     - Sistema de assinaturas
     - Agente Agno
     - Dashboard e analytics
     - Etc.

2. **Analisar Documentação Operacional**
   - Revisar `/docs/business/` para contexto adicional:
     - Estratégias de trading
     - Taxas e breakeven
     - Planos de assinatura
     - Gestão de risco
   - Identificar gaps e informações desatualizadas

3. **Consultar Especificações Técnicas**
   - Ler `/specs/technical/` para entender:
     - Arquitetura do sistema
     - Capacidades técnicas
     - Restrições e limitações
     - Roadmap técnico


### Fase 2: Discussão com Usuário

Após revisar toda a documentação existente, você fará perguntas para:
- Esclarecer informações inconsistentes ou conflitantes
- Preencher gaps identificados
- Validar premissas
- Obter insights sobre mudanças recentes

**Áreas de foco para perguntas**:
- Evolução da estratégia do produto desde a última atualização
- Mudanças nas personas de clientes ou no mercado-alvo
- Novas features ou funcionalidades planejadas
- Feedback recente de clientes que deva ser incorporado
- Mudanças no panorama competitivo
- Atualização de métricas e KPIs
- Ajustes na mensageria ou posicionamento
- Novos desafios ou oportunidades identificados

**Importante**:
- Seja seletivo e faça apenas perguntas relevantes baseadas nos gaps encontrados
- Se a documentação está completa e atualizada, confirme isso com o usuário antes de prosseguir
- Faça múltiplas rodadas se necessário

Quando estiver pronto, apresente um resumo das mudanças que planeja fazer e peça aprovação para prosseguir para a Fase 3.


### Fase 3: Atualização das Especificações de Negócio

**Diretório alvo**: `/specs/business/` (já existe)

Atualize os arquivos existentes conforme necessário. Mantenha a estrutura multi-arquivo estabelecida:

#### Estrutura de Arquivos (já existente)

**Arquivo principal**: `/specs/business/index.md`
- Mantém organização por camadas
- Links para todos os documentos
- Informações do projeto Crypteras

**Arquivos de especificação**:
- `/specs/business/CUSTOMER_PERSONAS.md` - Personas de clientes
- `/specs/business/CUSTOMER_JOURNEY.md` - Jornada do cliente
- `/specs/business/VOICE_OF_CUSTOMER.md` - Voz do cliente
- `/specs/business/PRODUCT_STRATEGY.md` - Estratégia do produto
- `/specs/business/COMPETITIVE_LANDSCAPE.md` - Panorama competitivo
- `/specs/business/MESSAGING_FRAMEWORK.md` - Framework de mensagens
- `/specs/business/CUSTOMER_COMMUNICATION.md` - Comunicação com cliente

**Features**: `/specs/business/features/`
- Arquivos individuais para cada feature principal do Crypteras:
  - `authentication-and-authorization.md`
  - `subscription-plans.md`
  - `smart-bots.md`
  - `candle-bots.md`
  - `dca-bots.md`
  - `agno-chat-agent.md`
  - `exchange-credentials.md`
  - `dashboard-analytics.md`
  - `backoffice-admin.md`
  - `payment-webhooks.md`
  - `circuit-breaker.md`

#### Diretrizes de Atualização de Arquivos

**1. `CUSTOMER_PERSONAS.md`**
- Atualizar personas existentes ou adicionar novas conforme necessário
- Focar em traders de criptomoedas (iniciantes, intermediários, avançados)
- Incluir:
  - Demografia e background
  - Objetivos de trading
  - Pontos de dor no mercado cripto
  - Familiaridade tecnológica
  - Preferências de comunicação
  - Notas de interação com IA (para o agente Agno)

**2. Outros arquivos principais**

Para cada arquivo de especificação, atualizar conforme necessário:
- `CUSTOMER_JOURNEY.md` - Jornada desde descoberta até advocacia/churn
- `VOICE_OF_CUSTOMER.md` - Feedback real, solicitações, elogios
- `PRODUCT_STRATEGY.md` - Visão, missão, posicionamento, prioridades
- `COMPETITIVE_LANDSCAPE.md` - Concorrentes, diferenciação, cenários
- `MESSAGING_FRAMEWORK.md` - Tom de voz, mensagens principais
- `CUSTOMER_COMMUNICATION.md` - Diretrizes de comunicação e suporte

**3. Features (`/specs/business/features/`)**

Atualizar arquivos existentes de features do Crypteras:
- Bots de trading: Smart, DCA, Candle
- Sistema de assinaturas FREE/PRO/MAX
- Agente Agno
- Dashboard e analytics
- Backoffice admin
- Payment webhooks
- Circuit breaker
- Autenticação e autorização
- Credenciais de exchanges

Para cada feature incluir:
- Propósito e valor para o usuário
- Casos de uso principais
- Limitações conhecidas
- Métricas de sucesso
- Orientação para interação com IA

## Fontes de Informação - Crypteras

### Fontes Internas (Principais)
- **Documentação existente**: `/specs/business/` e `/docs/business/`
- **Especificações técnicas**: `/specs/technical/`
- **Features documentadas**: `/specs/business/features/`
- **ADRs**: `/specs/technical/adr/`

### Fontes Adicionais (se necessário)
- **Repositório principal**: `/Users/thiagoabreu/workspace/crypteras-improved`
- **Feedback do usuário**: Discussão com o time sobre feedback recente
- **Dados de mercado**: Pesquisa web sobre concorrentes e tendências cripto
- **Métricas**: Consultar com o time sobre KPIs atuais

## Checklist de Qualidade

Antes de finalizar as atualizações, verificar:

- [ ] Informações refletem o contexto atual do Crypteras Trading System
- [ ] Personas consideram traders de cripto (iniciantes, intermediários, avançados)
- [ ] Features de bots (Smart, DCA, Candle) estão documentadas
- [ ] Sistema de assinaturas (FREE/PRO/MAX) está claro
- [ ] Agente Agno tem diretrizes de comunicação
- [ ] Panorama competitivo no mercado cripto está atualizado
- [ ] Mensageria alinha com voz da marca Crypteras
- [ ] Diretrizes de IA são acionáveis para suporte ao cliente
- [ ] Links entre arquivos estão funcionando
- [ ] Índice principal está atualizado

## Critérios de Sucesso

A documentação atualizada deve permitir que:
- ✅ O agente Agno forneça suporte contextualizado aos usuários
- ✅ Time de produto compreenda personas e jornada do cliente
- ✅ Decisões estratégicas sejam baseadas em inteligência de mercado
- ✅ Comunicação seja consistente com a voz da marca
- ✅ Novos membros do time compreendam o modelo de negócio rapidamente
