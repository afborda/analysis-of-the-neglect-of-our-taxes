# Fontes de Dados Públicos — Pesquisa Completa

> Última atualização: 2026-05-25
> Pesquisado por: Claude + time do projeto

---

## Índice

1. [Portais com API REST](#1-portais-com-api-rest)
2. [Portais para Download em Massa](#2-portais-para-download-em-massa)
3. [Agregadores já Tratados](#3-agregadores-já-tratados)
4. [Dados Geoespaciais](#4-dados-geoespaciais)
5. [Limitações Conhecidas](#5-limitações-conhecidas)
6. [Estratégia de Coleta Recomendada](#6-estratégia-de-coleta-recomendada)

---

## 1. Portais com API REST

### Portal da Transparência (CGU)
- **URL:** https://api.portaldatransparencia.gov.br
- **Documentação:** https://api.portaldatransparencia.gov.br/swagger-ui.html
- **Acesso:** Requer cadastro de token gratuito
- **Rate limit:** Verificar na documentação (geralmente 500 req/dia no plano gratuito)
- **Dados disponíveis:**
  - Gastos diretos do governo federal por município
  - Convênios e transferências a municípios
  - Obras públicas federais por município
  - Contratos e licitações
  - Servidores públicos federais (nome, salário, cargo)
  - Auxílio emergencial por município
  - Emendas parlamentares por município
- **Granularidade municipal:** ✅ Sim — filtro por código IBGE do município
- **Formato:** JSON
- **Cobertura temporal:** 2004 em diante

### IBGE Serviço de Dados
- **URL:** https://servicodados.ibge.gov.br/api
- **Documentação:** https://servicodados.ibge.gov.br/api/docs
- **Acesso:** Público, sem token
- **APIs relevantes:**
  | API | Versão | O que tem |
  |-----|--------|-----------|
  | Agregados | v3 | Séries históricas: PIB, população, emprego, saúde, educação |
  | Localidades | v1 | Hierarquia: Brasil > Região > Estado > Município |
  | Malhas Geográficas | v2-v4 | Shapefiles/GeoJSON por município |
  | Pesquisas | v1 | PNAD, Censo Agropecuário, dados das cidades |
- **Granularidade municipal:** ✅ Sim — `/municipios/{id}/` em várias APIs
- **Formato:** JSON, GeoJSON
- **SIDRA:** https://sidra.ibge.gov.br — acesso programático via API também

### SICONFI (Tesouro Nacional)
- **URL:** https://siconfi.tesouro.gov.br
- **API:** https://apidatalake.tesouro.gov.br/ords/siconfi (REST via ORDS)
- **Acesso:** Consultas básicas públicas; relatórios completos requerem CPF/Cert. Digital
- **Dados disponíveis:**
  - **FINBRA** — Finanças Brasileiras: receitas e despesas municipais anuais
  - **RGF** — Relatório de Gestão Fiscal (dívida, pessoal)
  - **RREO** — Execução orçamentária bimestral
  - **MSC** — Matriz de Saldos Contábeis (balanço patrimonial)
- **Granularidade municipal:** ✅ Sim — por código do ente federativo (IBGE)
- **Formato:** JSON, CSV via download
- **Cobertura temporal:** 2002 em diante (FINBRA desde 2000)
- **Nota:** FINBRA é a fonte mais completa para finanças municipais históricas

### TSE Dados Abertos
- **URL:** https://dadosabertos.tse.jus.br
- **API:** CKAN API disponível
- **Acesso:** Público, sem token (download direto por URL)
- **Total datasets:** 167 datasets
- **Dados disponíveis:**
  | Categoria | Qtd | O que tem |
  |-----------|-----|-----------|
  | Resultados | 61 | Votação por município, zona, seção — desde 2004 |
  | Candidatos | 35 | Nome, partido, CPF, bens declarados, candidaturas |
  | Eleitores | 19 | Perfil do eleitorado por município |
  | Prestação de contas | 24 | Financiamento de campanha |
  | Pesquisas eleitorais | 8 | Dados de pesquisas registradas no TSE |
- **Granularidade municipal:** ✅ Sim — resultados por município disponíveis
- **Formato:** CSV, TXT (zip), CSV
- **Cobertura temporal:** 1994–2024 (eleições municipais em anos pares)
- **Ficha limpa:** Os dados de candidatos incluem situação de elegibilidade — cruzável com condenações via CNPJ dos partidos

---

## 2. Portais para Download em Massa

### Repositório de Dados Eleitorais do TSE
- **URL:** https://dadosabertos.tse.jus.br/dataset/
- **Como usar:** Download direto por ZIP, arquivo por eleição/estado
- **Arquivos relevantes:**
  - `consulta_cand_YYYY.zip` — todos os candidatos do ano
  - `bem_candidato_YYYY.zip` — bens declarados
  - `consulta_vagas_YYYY.zip` — vagas disputadas por município
  - `detalhe_votacao_munzona_YYYY.zip` — resultado por município

### CAGED / RAIS (Emprego Formal)
- **URL:** https://www.gov.br/trabalho/pt-br/assuntos/empregabilidade/caged
- **RAIS:** https://www.gov.br/trabalho/pt-br/assuntos/empregabilidade/rais
- **Dados disponíveis:**
  - Admissões e demissões mensais por município (CAGED)
  - Emprego formal total por município/setor (RAIS anual)
  - Remuneração média por município
  - Setor econômico dos empregos
- **Formato:** CSV e Excel por UF/ano (arquivos grandes)
- **Cobertura:** Mensal (CAGED) e anual (RAIS) desde 1985
- **Nota:** RAIS é a fonte definitiva de emprego formal por município

### Receita Federal — CNPJ e Estabelecimentos
- **URL:** https://dados.gov.br/dados/conjuntos-dados/cadastro-nacional-da-pessoa-juridica---cnpj
- **Dados disponíveis:**
  - Todos os CNPJs ativos e inativos do Brasil
  - Município de cada estabelecimento
  - CNAE (setor econômico)
  - Data de abertura/encerramento
- **Formato:** CSV comprimido (arquivos de vários GB)
- **Atualização:** Mensal
- **Nota:** Para contar empresas ativas por município, filtrar `situacao_cadastral = ATIVA`

### TCU — Tribunais de Contas
- **URL:** https://portal.tcu.gov.br/dados-abertos/
- **Dados disponíveis:**
  - Contas de governo julgadas
  - Acórdãos com condenações
  - Irregularidades em convênios
  - CADIN — cadastro de devedores
- **Nota:** TCEs estaduais têm dados municipais mais detalhados — cada estado tem o seu próprio portal

### Obras.gov.br / Transferegov
- **URL:** https://www.transferegov.br/
- **Dados disponíveis:**
  - Obras financiadas com recursos federais por município
  - Status: em execução, paralisada, concluída
  - Valor contratado vs. pago
  - Executor (prefeito/entidade responsável)
- **Nota:** Substitui antigo SICONV; dados acessíveis via Portal da Transparência também

---

## 3. Agregadores já Tratados

Estas fontes já fizeram o trabalho sujo de coletar, limpar e normalizar dados brutos.
**Recomendação: começar por aqui para o piloto.**

### Brasil.IO
- **URL:** https://brasil.io/datasets/
- **O que tem (relevante para nós):**
  | Dataset | O que tem |
  |---------|-----------|
  | `eleicoes-brasil` | Dados TSE de 1996–2024 limpos: candidatos, votações, filiados |
  | `gastos-diretos-governo-federal` | Portal da Transparência limpo |
  | `socios-brasil` | Estrutura societária de todas as empresas |
  | `salarios-magistrados` | Transparência de remunerações |
- **Acesso:** API REST gratuita + download CSV
- **Vantagem:** Dados normalizados, sem necessidade de parsing complexo

### Base dos Dados (BD+)
- **URL:** https://basedosdados.org
- **O que tem:**
  | Dataset | Fonte Original | O que tem |
  |---------|---------------|-----------|
  | `br_ibge_populacao` | IBGE | Pop. por município por ano |
  | `br_tse_eleicoes` | TSE | Candidatos + resultados normalizados |
  | `br_me_caged` | MTE/CAGED | Saldo de emprego mensal por município |
  | `br_me_rais` | MTE/RAIS | Emprego formal anual por município |
  | `br_rf_cnpj` | Receita Federal | CNPJs com município |
  | `br_inep_ideb` | INEP | IDEB por escola e município |
  | `br_siconfi_municipios` | SICONFI | Finanças municipais |
  | `br_inep_censo_escolar` | INEP | Dados educação por município |
- **Acesso:** BigQuery público (via Google Cloud), Python SDK, R SDK
- **Vantagem:** Todos os datasets no mesmo schema, join por `id_municipio` (código IBGE 7 dígitos)
- **Custo:** Consultas < 1TB/mês no BigQuery são gratuitas
- **Nota:** Esta é provavelmente a **melhor fonte para o piloto** — join fácil entre datasets

### IPEADATA
- **URL:** http://www.ipeadata.gov.br
- **O que tem:**
  - Séries históricas de indicadores econômicos por município
  - IDH histórico (via PNUD)
  - PIB per capita municipal histórico
  - Índice de Gini por município
  - Dados do Censo 1991, 2000, 2010, 2022
- **Acesso:** API REST: `http://www.ipeadata.gov.br/api/odata4/`
- **Formato:** OData/JSON

### Atlas Brasil (PNUD)
- **URL:** https://atlasbrasil.org.br
- **O que tem:**
  - IDHM (IDH Municipal) por município — 1991, 2000, 2010
  - Desagregação: IDHM Educação, IDHM Longevidade, IDHM Renda
  - Ranking municipal de IDH
  - Índice de Vulnerabilidade Social (IVS)
- **Formato:** Download CSV, sem API formal

---

## 4. Dados Geoespaciais

### IBGE Malhas Geográficas
- **URL:** https://servicodados.ibge.gov.br/api/v2/malhas
- **Formato:** GeoJSON, TopoJSON, SVG
- **Endpoint exemplo:** `/api/v2/malhas/3550308?resolucao=5&formato=application/vnd.geo+json`
  - `3550308` = código IBGE de São Paulo
- **Tipos:** Município, Microrregião, Mesorregião, Estado, Região, Brasil

### IBGE Geoportal
- **URL:** https://geoftp.ibge.gov.br/
- **Shapefiles completos** do Brasil por município (para uso no QGIS, Python GeoPandas)

---

## 5. Limitações Conhecidas

| Fonte | Limitação | Impacto |
|-------|-----------|---------|
| PNAD Contínua (IBGE) | Cobertura municipal apenas para capitais | Sem dados de desemprego para municípios pequenos |
| TCEs estaduais | Cada estado tem portal diferente, sem padronização | Difícil automatizar — 26 portais distintos |
| Portal da Transparência | Apenas gastos federais | Gastos municipais próprios precisam do SICONFI |
| SICONFI histórico | Qualidade dos dados varia antes de 2010 | Análise histórica exige validação cuidadosa |
| IDH Municipal | Último dado disponível: Censo 2010 (publicado 2013) | Precisará usar IDHM 2022 quando sair |
| Ficha Limpa TSE | Dados de inelegibilidade não têm API direta | Requer cruzamento manual de candidatos + condenações |
| CAGED | Revisões retroativas frequentes | Download deve capturar versões atualizadas |

---

## 6. Estratégia de Coleta Recomendada

### Fase 1 — Estado Piloto (Acre, 22 municípios)

**Fonte primária:** Base dos Dados (BD+) via BigQuery
- Um único `JOIN` por `id_municipio` integra IBGE + CAGED + TSE + SICONFI
- Custo zero para volumes do piloto
- Já limpo e documentado

**Complementar:** Brasil.IO para dados eleitorais históricos

**Stack mínima para o piloto:**
```
Python 3.12
├── google-cloud-bigquery  (Base dos Dados)
├── requests               (APIs REST)
├── pandas                 (transformação)
└── jupyter                (exploração)
```

### Fase 2 — Expansão Regional

Substituir BD+ por pipelines próprios:
- SICONFI API → finanças municipais
- TSE CSV → dados eleitorais com parser próprio
- IBGE API → perfil municipal atualizado

### Fase 3 — Brasil Completo

Orquestração com Airflow/Prefect, armazenamento em S3/MinIO com formato Parquet.

---

## Links Rápidos

| Portal | URL |
|--------|-----|
| Portal da Transparência API | https://api.portaldatransparencia.gov.br |
| IBGE Serviço de Dados | https://servicodados.ibge.gov.br/api |
| SICONFI API | https://apidatalake.tesouro.gov.br/ords/siconfi |
| TSE Dados Abertos | https://dadosabertos.tse.jus.br |
| Brasil.IO | https://brasil.io |
| Base dos Dados (BD+) | https://basedosdados.org |
| IPEADATA API | http://www.ipeadata.gov.br/api/odata4/ |
| CAGED | https://www.gov.br/trabalho/pt-br/assuntos/empregabilidade/caged |
| RAIS | https://www.gov.br/trabalho/pt-br/assuntos/empregabilidade/rais |
| Receita Federal CNPJ | https://dados.gov.br/dados/conjuntos-dados/cadastro-nacional-da-pessoa-juridica---cnpj |
| TCU Dados Abertos | https://portal.tcu.gov.br/dados-abertos/ |
| Atlas Brasil (PNUD/IDH) | https://atlasbrasil.org.br |
| IBGE SIDRA | https://sidra.ibge.gov.br |
| IBGE Malhas Geo | https://servicodados.ibge.gov.br/api/v2/malhas |
