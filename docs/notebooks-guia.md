# Guia dos Notebooks — Progressão de Aprendizado

> Os 7 notebooks da Fase 1 formam uma jornada de aprendizado progressiva.
> Cada um tem uma pergunta de negócio para responder e uma técnica nova para aprender.

---

## Mapa da Progressão

```mermaid
flowchart TD
    NB01["📓 Notebook 01\nPerfil dos Municípios\n──────────────────\n🎯 Como são as cidades do Acre?\n🛠 BD+, limpeza, plotly básico"]

    NB02["📓 Notebook 02\nEleições Históricas\n──────────────────\n🎯 Quem governa e como chegou?\n🛠 JOINs multi-tabela, groupby"]

    NB03["📓 Notebook 03\nBolsa Família + Emprego\n──────────────────\n🎯 Qual a dependência social?\n🛠 3+ fontes, correlações, scatter"]

    NB04["📓 Notebook 04\nFinanças das Prefeituras\n──────────────────\n🎯 As prefeituras gastam bem?\n🛠 Análise financeira, LRF, índices"]

    NB05["📓 Notebook 05\nSalário: Prefeito vs. Povo\n──────────────────\n🎯 O prefeito ganha quanto?\n🛠 API REST, merge com dados locais"]

    NB06["📓 Notebook 06\nFicha dos Prefeitos\n──────────────────\n🎯 Quem são os gestores?\n🛠 Playwright, parsing HTML"]

    NB07["📓 Notebook 07\nRanking IVM\n──────────────────\n🎯 Qual cidade tem melhor gestão?\n🛠 Normalização, índice composto"]

    NB01 --> NB02 --> NB03 --> NB04 --> NB05 --> NB06 --> NB07

    style NB01 fill:#E3F2FD,color:#000
    style NB02 fill:#E8F5E9,color:#000
    style NB03 fill:#FFF3E0,color:#000
    style NB04 fill:#FCE4EC,color:#000
    style NB05 fill:#F3E5F5,color:#000
    style NB06 fill:#E0F2F1,color:#000
    style NB07 fill:#FFF8E1,color:#000
```

---

## Notebook 01 — Perfil dos Municípios do Acre

**Pergunta:** Como são os municípios do Acre? Tamanho, IDH, população, distribuição.

**Fontes:**
```mermaid
flowchart LR
    IBGE["BD+\nbr_ibge_populacao.municipio"] --> NB01
    IDH["BD+\nbr_ipea_idhm.municipio"] --> NB01
    NB01["Notebook 01"] --> V1["Ranking por população"]
    NB01 --> V2["Evolução pop. 2010-2024"]
    NB01 --> V3["Mapa IDH (choropleth)"]
    NB01 --> V4["% urbana vs. rural"]
```

**Técnicas introduzidas:**
- `bd.read_sql()` — primeira query no BigQuery
- `df.head()`, `df.describe()`, `df.info()` — exploração inicial
- `plotly.express.choropleth()` — mapa colorido por indicador
- Merge com GeoJSON do IBGE para visualizar no mapa

---

## Notebook 02 — Eleições Históricas

**Pergunta:** Quem governa cada cidade do Acre desde 2000? Qual partido domina cada município?

**Fontes:**
```mermaid
flowchart LR
    TSE1["BD+\nbr_tse_eleicoes.resultados_candidato_municipio"] --> NB02
    TSE2["BD+\nbr_tse_eleicoes.perfil_eleitorado"] --> NB02
    POP["BD+\nbr_ibge_populacao.municipio"] --> NB02

    NB02["Notebook 02"] --> V1["Prefeitos eleitos 2000-2024\npor município"]
    NB02 --> V2["Domínio partidário por cidade\n(timeline colorida)"]
    NB02 --> V3["% comparecimento vs.\neleitores cadastrados vs. população"]
    NB02 --> V4["Municípios com maior\nabstenção histórica"]
```

**Técnicas introduzidas:**
- `df.merge()` com múltiplas tabelas pela chave `id_municipio`
- `df.groupby().agg()` — agregações por município e ano
- `pd.pivot_table()` — tabela dinâmica partidos × anos
- `plotly.express.timeline()` — visualização de mandatos no tempo

---

## Notebook 03 — Bolsa Família + Emprego

**Pergunta:** Qual a dependência social de cada cidade? Existe correlação com emprego formal?

**Fontes:**
```mermaid
flowchart LR
    BF["BD+\nbr_mc_bolsa_familia.municipio"] --> NB03
    RAIS["BD+\nbr_me_rais.microdados_vinculos"] --> NB03
    POP["BD+\nbr_ibge_populacao.municipio"] --> NB03

    NB03["Notebook 03"] --> V1["% população com\nBolsa Família por cidade"]
    NB03 --> V2["% com emprego formal\npor cidade"]
    NB03 --> V3["Scatter: BF vs.\nemprego formal (correlação)"]
    NB03 --> V4["Evolução temporal\n% BF por mandato"]
```

**Técnicas introduzidas:**
- Cruzamento de 3 fontes pelo mesmo `id_municipio`
- Cálculo de percentuais: `(beneficiados / populacao) * 100`
- `df.corr()` — matriz de correlação entre variáveis
- `plotly.express.scatter()` com tamanho de ponto = população

---

## Notebook 04 — Finanças das Prefeituras

**Pergunta:** As prefeituras do Acre cumprem a Lei de Responsabilidade Fiscal? Quem está endividado?

**Fontes:**
```mermaid
flowchart LR
    SICONFI["BD+\nbr_siconfi.municipios"] --> NB04

    NB04["Notebook 04"] --> V1["Gasto por função\n(saúde, educação, obras...)"]
    NB04 --> V2["Dívida / Receita\n(semáforo: verde/amarelo/vermelho)"]
    NB04 --> V3["% gasto com pessoal\nvs. limite LRF (54%)"]
    NB04 --> V4["Receita própria vs.\ndependência de FPM"]
```

**Técnicas introduzidas:**
- Cálculo de índices financeiros (ratios)
- `pd.cut()` — categorização em faixas (verde/amarelo/vermelho)
- `plotly.express.bar()` empilhado por função de gasto
- Comparação com benchmarks legais (LRF)

---

## Notebook 05 — Salário: Prefeito vs. Povo

**Pergunta:** O prefeito de cada cidade ganha quanto comparado ao salário médio da população?

**Fontes:**
```mermaid
flowchart LR
    TRANSP["Portal da Transparência\nAPI REST (requests)"] --> NB05
    RAIS["BD+\nbr_me_rais.microdados_vinculos\n(salário médio local)"] --> NB05

    NB05["Notebook 05"] --> V1["Salário do prefeito\npor município"]
    NB05 --> V2["Razão:\nprefeito ÷ salário médio local"]
    NB05 --> V3["Ranking: quem ganha\nmais vezes o salário médio"]
    NB05 --> V4["Comparação com\nmínimo nacional"]
```

**Técnicas introduzidas:**
- `requests.get()` com autenticação por header
- Cache manual: salvar resultado da API em JSON local
- `df.merge()` combinando API + BD+
- Formatação monetária: `df["salario"].map("R$ {:,.0f}".format)`

---

## Notebook 06 — Ficha dos Prefeitos

**Pergunta:** Os prefeitos eleitos têm ficha limpa? Houve condenações durante o mandato?

**Fontes:**
```mermaid
flowchart LR
    TSE_DV["TSE DivulgaCand\n(Playwright Docker)"] --> NB06
    TSE_ELEITO["BD+\nbr_tse_eleicoes.candidatos\n(dados eleitorais)"] --> NB06

    NB06["Notebook 06"] --> V1["Situação de elegibilidade\npor prefeito"]
    NB06 --> V2["Tipo de condenação\npor categoria"]
    NB06 --> V3["Partido com mais\nprefeitos com ficha suja"]
    NB06 --> V4["Correlação: ficha suja\nvs. qualidade da gestão"]
```

**Técnicas introduzidas:**
- Playwright: `browser.new_page()`, `page.goto()`, `page.inner_text()`
- Parsing de HTML com `page.query_selector_all()`
- Enriquecimento de dataset: merge de Playwright + BD+
- `df.value_counts()` para análise de categorias

---

## Notebook 07 — Ranking IVM

**Pergunta:** Qual o ranking final de viabilidade municipal do Acre?

**Fontes:**
```mermaid
flowchart LR
    NB01 --> NB07
    NB02 --> NB07
    NB03 --> NB07
    NB04 --> NB07
    NB05 --> NB07
    NB06 --> NB07

    NB07["Notebook 07\nRanking IVM"] --> OUT1["Score 0-100\npor gestão"]
    NB07 --> OUT2["Ranking das 22 cidades"]
    NB07 --> OUT3["Melhor e pior gestão\npor município (histórico)"]
    NB07 --> OUT4["Dataset final\nCSV + Parquet"]
```

**Técnicas introduzidas:**
- `sklearn.preprocessing.MinMaxScaler` — normalização 0–1
- Construção de índice composto com pesos
- `df.rank()` — ranking por coluna
- `plotly.express.bar()` horizontal com cores por score
- `df.to_parquet()` — exportar para formato colunar

---

## Progressão de Complexidade

```mermaid
xychart-beta
    title "Complexidade técnica por notebook"
    x-axis ["NB01\nPerfil", "NB02\nEleições", "NB03\nSocial", "NB04\nFinanças", "NB05\nSalários", "NB06\nFicha", "NB07\nIVM"]
    y-axis "Complexidade" 1 --> 10
    bar [2, 4, 5, 5, 6, 7, 8]
    line [2, 4, 5, 5, 6, 7, 8]
```

---

## Como Rodar Todos os Notebooks

```bash
# Instalar dependências
pip install basedosdados pandas jupyter plotly scikit-learn playwright
playwright install chromium

# Rodar todos em sequência (gera outputs HTML)
jupyter nbconvert --to notebook --execute notebooks/fase1_acre/*.ipynb

# Ou abrir interativamente
jupyter lab notebooks/fase1_acre/
```
