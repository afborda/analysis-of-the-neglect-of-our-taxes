# Análise da Negligência nos Nossos Impostos
### *Transparência municipal para todos os brasileiros*

> **Projeto de dados abertos** criado por Abner Fonseca e Miguel para transformar dados públicos complexos em informação acessível sobre como as prefeituras do Brasil gastam o dinheiro dos cidadãos.

![Status](https://img.shields.io/badge/status-Fase%201%20em%20andamento-blue)
![Municípios](https://img.shields.io/badge/piloto-Acre%2022%20municípios-green)
![Licença](https://img.shields.io/badge/licença-MIT-lightgrey)

---

## Status Atual

| Fase | Escopo | Status |
|------|--------|--------|
| **Fase 1** | Piloto — Acre (22 municípios) | 🟡 Em andamento |
| Fase 2 | Região Norte (450 municípios) | ⏳ Planejada |
| Fase 3 | Brasil completo (5.570 municípios) | ⏳ Planejada |
| Fase 4 | Dashboard público + API + IA | ⏳ Planejada |

---

## Começo Rápido

### Pré-requisitos

- Python 3.12+
- Conta Google Cloud gratuita ([criar aqui](https://console.cloud.google.com)) com projeto `analise-municipios-br`
- Token gratuito da [API do Portal da Transparência](https://api.portaldatransparencia.gov.br/swagger-ui)
- Docker (para os scrapers Playwright)

### Instalação

```bash
git clone https://github.com/seu-usuario/analysis-of-the-neglect-of-our-taxes.git
cd analysis-of-the-neglect-of-our-taxes

# Instalar dependências Python
pip install -r requirements.txt
playwright install chromium

# Configurar variáveis de ambiente
cp .env.example .env
# Editar .env com: GCP_PROJECT_ID, GOOGLE_APPLICATION_CREDENTIALS, TRANSPARENCIA_API_TOKEN
```

### Rodar os notebooks

```bash
# Abrir JupyterLab
jupyter lab

# Ou executar todos automaticamente
jupyter nbconvert --to notebook --execute notebooks/fase1_acre/*.ipynb
```

### Rodar os scrapers no Docker

```bash
docker build -t playwright-scraper -f docker/playwright/Dockerfile .

# Coletar ficha limpa dos prefeitos do Acre
docker run -v $(pwd)/data:/app/data playwright-scraper scrapers/tse_ficha_limpa.py AC
```

---

## Documentação

| Documento | Descrição |
|-----------|-----------|
| [Arquitetura Geral](docs/arquitetura-geral.md) | Fluxo de dados, ERD, stack por fase |
| [Roadmap de Fases](docs/roadmap-fases.md) | Linha do tempo e critérios de conclusão |
| [IVM — Metodologia](docs/ivm-metodologia.md) | Como o Índice de Viabilidade Municipal é calculado |
| [Fluxo de Coleta](docs/fluxo-coleta.md) | BD+ vs Playwright vs API — quando usar cada um |
| [Guia dos Notebooks](docs/notebooks-guia.md) | Progressão de aprendizado dos 7 notebooks |
| [Fontes de Dados](docs/fontes-de-dados/fontes-pesquisadas.md) | 15 portais públicos mapeados |
| [Plano Fase 1](docs/superpowers/plans/2026-05-25-fase1-acre.md) | Plano de implementação passo a passo |

---

## Por que esse projeto existe?

O Brasil tem mais de **5.570 municípios**. Cada um recebe verbas federais, estaduais e cobra impostos locais — mas a grande maioria dos cidadãos nunca viu um relatório de gastos, não sabe quantas obras estão paradas, não conhece a ficha do seu prefeito, nem sabe se o IDH da sua cidade está caindo ou subindo.

Os dados **existem** — estão no Portal da Transparência, no IBGE, no TSE, no TCU — mas estão dispersos, em formatos difíceis, sem cruzamento e sem contexto.

**Nós vamos mudar isso.**

Esse projeto coleta, limpa, cruza e publica esses dados de forma clara, comparável e acessível. É também um projeto de aprendizado: estamos aprendendo análise de dados, Spark, Playwright, backend e frontend fazendo algo que importa.



---

## O que vamos entregar

Para cada município brasileiro, queremos consolidar:

### 📊 Perfil do Município
| Dado | Fonte Prevista |
|------|---------------|
| Nome, UF, código IBGE | IBGE |
| População estimada | IBGE — Censo / PNAD |
| IDH (Índice de Desenvolvimento Humano) | PNUD / Atlas Brasil |
| PIB per capita | IBGE |
| Área territorial (km²) | IBGE |

### 💰 Finanças Públicas
| Dado | Fonte Prevista |
|------|---------------|
| Receitas e despesas anuais | Portal da Transparência / SICONFI |
| Dívida consolidada | SICONFI / STN |
| Transferências recebidas (FPM, SUS, FUNDEB) | Tesouronacional |
| Gastos por função (saúde, educação, obras...) | SICONFI |
| Prefeituras com contas reprovadas pelo TCU/TCE | TCU / TCEs estaduais |

### 🏗️ Obras Públicas
| Dado | Fonte Prevista |
|------|---------------|
| Obras em execução | Portal da Transparência |
| Obras concluídas | Portal da Transparência |
| Obras paralisadas / atrasadas | Portal da Transparência |
| Data de início / previsão de término | Portal da Transparência |
| Valor contratado vs. valor pago | Portal da Transparência |

### 👥 Emprego e Mercado de Trabalho
| Dado | Fonte Prevista |
|------|---------------|
| Trabalhadores com carteira assinada (CLT) | CAGED / RAIS |
| Taxa de desemprego | PNAD |
| Número de MEIs e CNPJs ativos | Receita Federal |

### 🗳️ Política e Gestão
| Dado | Fonte Prevista |
|------|---------------|
| Nome do prefeito(a) atual | TSE |
| Partido do prefeito | TSE |
| Histórico de gestões anteriores | TSE |
| Situação "ficha limpa" do prefeito | TSE / DivulgaCand |
| Número de vereadores | TSE |
| Partido majoritário na câmara | TSE |
| Gestões que mais geraram dívidas | SICONFI cruzado com TSE |

### 📡 Qualidade dos Dados
| Indicador | O que significa |
|-----------|----------------|
| ✅ Atualizado | Dados do exercício atual disponíveis |
| ⚠️ Parcial | Alguns dados estão desatualizados |
| ❌ Desatualizado | Última atualização há mais de 2 anos |

---

## Rankings e Análises Planejadas

- 🏆 **Cidades mais viáveis** — combinando IDH + finanças saudáveis + obras concluídas
- ⚠️ **Cidades em risco** — alto endividamento + baixo IDH + obras paralisadas
- 🔍 **Prefeitos com ficha suja** por partido e região
- 📉 **Gestões que mais endividaram cidades** (comparativo histórico)
- 🗺️ **Mapa interativo por estado** — visualização geográfica dos indicadores
- 📊 **Comparativo entre cidades de mesmo porte** (faixas populacionais)

---

## Roadmap — Etapas do Projeto

Vamos trabalhar **estado por estado**, começando pelos menores para validar o processo antes de escalar.

```
Fase 1 — Fundação (Estado piloto)
├── Definir estado piloto (ex: RO ou AC — menos municípios)
├── Mapear e testar todas as fontes de dados
├── Criar pipeline de coleta e limpeza
├── Publicar dataset bruto no GitHub/Kaggle
└── Criar visualização básica dos indicadores

Fase 2 — Expansão Regional
├── Replicar para todos os estados de uma região
├── Refinar pipeline com os aprendizados
├── Adicionar dados de obras e finanças
└── Publicar relatório comparativo regional

Fase 3 — Brasil Completo
├── Cobertura dos 5.570 municípios
├── Dashboard interativo público
├── API pública para acesso aos dados
└── Atualização periódica automatizada

Fase 4 — Análises Avançadas
├── Modelos de viabilidade municipal
├── Alertas para cidades em deterioração
├── Integração com notícias e licitações
└── Parceria com veículos de jornalismo de dados
```

---

## Fontes de Dados

| Portal | URL | Dados |
|--------|-----|-------|
| Portal da Transparência | [transparencia.gov.br](https://portaldatransparencia.gov.br) | Gastos, obras, convênios |
| SICONFI | [siconfi.tesouro.gov.br](https://siconfi.tesouro.gov.br) | Finanças municipais |
| IBGE Cidades | [ibge.gov.br/cidades-e-estados](https://www.ibge.gov.br/cidades-e-estados.html) | Perfil, população, IDH |
| TSE DivulgaCand | [divulgacand.tse.jus.br](https://divulgacand.tse.jus.br) | Candidatos, fichas, partidos |
| Atlas Brasil | [atlasbrasil.org.br](http://www.atlasbrasil.org.br) | IDH detalhado |
| CAGED / RAIS | [gov.br/trabalho](https://www.gov.br/trabalho) | Emprego formal |
| TCU | [tcu.gov.br](https://portal.tcu.gov.br) | Contas aprovadas/reprovadas |
| Brasil.IO | [brasil.io](https://brasil.io) | Dados abertos já tratados |

---

## Stack Técnica (Planejada)

```
Coleta       →  Python (requests, Scrapy, Selenium)
Armazenamento→  PostgreSQL + Parquet no S3/MinIO
Transformação→  dbt + Pandas
Visualização →  Metabase / Apache Superset / Next.js
Orquestração →  Airflow ou Prefect
Publicação   →  GitHub Pages / Vercel + API FastAPI
```

> A stack vai evoluir conforme o projeto crescer. A prioridade inicial é **dados corretos antes de infraestrutura complexa**.

---

## Como Contribuir

Esse projeto é **aberto e colaborativo**. Você pode ajudar:

- 🐛 Reportando dados incorretos via Issues
- 🏙️ Adicionando dados de um município específico que você conhece
- 💻 Contribuindo com scripts de coleta ou análise
- 📢 Divulgando para jornalistas, pesquisadores e ativistas

---

## Autores

| Nome | Papel |
|------|-------|
| **Abner Fonseca** | Co-fundador, Engenharia de Dados |
| **Miguel** | Co-fundador, Análise e Visualização |

---

## Licença

Dados públicos, projeto público.
Código sob [MIT License](LICENSE) — use, adapte e redistribua livremente, com atribuição.

---

> *"Transparência não é favor. É obrigação. Nós só estamos facilitando o que já deveria ser fácil."*
