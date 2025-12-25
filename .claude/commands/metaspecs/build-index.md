# Reconstruir Índices de Navegação - Crypteras Metaspecs

Este comando reconstrói os arquivos de índice de navegação do repositório **crypteras-metaspecs**, que é a fonte canônica de verdade para toda a documentação, especificações técnicas, lógica de negócio e decisões arquiteturais do **Crypteras Trading System**.

## Estrutura do Projeto

O projeto crypteras-metaspecs contém:

- **`/specs/`** - Especificações técnicas e de negócio (fonte canônica)
  - `/specs/technical/` - Documentação técnica, ADRs, guias de código
  - `/specs/business/` - Personas, jornada do cliente, estratégia de produto

- **`/docs/`** - Documentação adicional e guias operacionais
  - `/docs/technical/` - Arquitetura, configurações, integrações
  - `/docs/business/` - Estratégias de trading, taxas, planos de assinatura

## Uso

### /build-index (sem argumentos)

Reconstrói o arquivo **index.md** raiz do projeto em `/specs/technical/index.md` e `/specs/business/index.md`.

**O que fazer**:
1. Analisar a estrutura de diretórios em `/specs/technical/` e `/specs/business/`
2. Identificar todos os arquivos markdown principais
3. Atualizar os índices para refletir a estrutura atual
4. Manter a organização por camadas (Camada 1, 2, 3, etc.)
5. Incluir descrições breves de cada arquivo/seção
6. Verificar que todos os links estão corretos

**Não modificar**:
- Conteúdo dos arquivos individuais
- Estrutura de diretórios
- Templates

### /build-index <caminho-específico>

Reconstrói o índice de um diretório específico.

**Exemplos**:
```bash
/build-index specs/technical
/build-index specs/business
/build-index docs/technical
/build-index docs/business
```

**O que fazer**:
1. Analisar apenas o diretório especificado
2. Atualizar o arquivo index.md desse diretório
3. Verificar links relativos
4. Manter consistência com a estrutura geral

Argumentos fornecidos: #$ARGUMENTS
