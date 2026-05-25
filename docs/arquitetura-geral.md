# Arquitetura Geral do Projeto

> Como os dados fluem desde os portais públicos até as análises e o produto final.

---

## Visão Macro — 4 Fases

```mermaid
flowchart TD
    subgraph F1["Fase 1 — Piloto (Acre)"]
        direction LR
        BDplus["Base dos Dados\nBigQuery"] --> NB["Jupyter\nNotebooks"]
        PW1["Playwright\nDocker"] --> NB
        API1["APIs REST"] --> NB
        NB --> IVM1["IVM v1\n22 municípios"]
    end

    subgraph F2["Fase 2 — Região Norte"]
        direction LR
        ETL["Pipeline Python\nETL próprio"] --> PG["PostgreSQL\nDocker"]
        PG --> DBT["dbt\ntransformações"]
        DBT --> NB2["Notebooks\n450 municípios"]
    end

    subgraph F3["Fase 3 — Brasil Completo"]
        direction LR
        AF["Airflow\norquestração"] --> MINIO["MinIO\nParquet"]
        MINIO --> SP["Apache Spark\n/ Databricks"]
        SP --> RNK["Rankings\nnacionais"]
    end

    subgraph F4["Fase 4 — Produto Público"]
        direction LR
        PG2["PostgreSQL"] --> BE["FastAPI\nBackend"]
        BE --> FE["Next.js\nFrontend"]
        BE --> IA["Agente IA\nLangChain"]
    end

    F1 --> F2 --> F3 --> F4
```

---

## Fluxo de Dados — Fase 1 em Detalhe

```mermaid
flowchart LR
    subgraph FONTES["Fontes de Dados"]
        BDp["Base dos Dados\n(BigQuery)"]
        TRANSP["Portal da\nTransparência API"]
        TSE["TSE\nDivulgaCand"]
        TREGOV["Transferegov\n(obras)"]
    end

    subgraph COLETA["Coleta"]
        PY["Python\nbd.read_sql()"]
        REQ["requests\n(API REST)"]
        PW["Playwright\n(Docker)"]
    end

    subgraph STORAGE["Armazenamento Local"]
        CSV["CSV / JSON\ndata/raw/"]
        CACHE["Cache\n(evita re-scraping)"]
    end

    subgraph ANALISE["Análise"]
        PD["pandas\ntransformações"]
        JUP["Jupyter\nNotebooks"]
        PLT["Plotly\nvisualizações"]
    end

    subgraph OUTPUT["Saída"]
        IVM["Score IVM\npor município"]
        RPT["Relatório\nde insights"]
        DS["Dataset\nCSV + Parquet"]
    end

    BDp --> PY --> CSV
    TRANSP --> REQ --> CSV
    TSE --> PW --> CACHE
    TREGOV --> PW --> CACHE

    CSV --> PD
    CACHE --> PD
    PD --> JUP
    JUP --> PLT
    JUP --> IVM
    JUP --> RPT
    JUP --> DS
```

---

## Chave de Cruzamento Universal

```mermaid
erDiagram
    MUNICIPIO {
        string id_municipio PK "Código IBGE 7 dígitos"
        string nome
        string sigla_uf
    }

    MUNICIPIO ||--o{ POPULACAO : "tem"
    MUNICIPIO ||--o{ ELEICOES : "tem"
    MUNICIPIO ||--o{ SICONFI : "tem"
    MUNICIPIO ||--o{ BOLSA_FAMILIA : "tem"
    MUNICIPIO ||--o{ RAIS_EMPREGO : "tem"
    MUNICIPIO ||--o{ OBRAS : "tem"

    POPULACAO {
        string id_municipio FK
        int ano
        int populacao
    }

    ELEICOES {
        string id_municipio FK
        int ano_eleicao
        string nome_candidato
        string sigla_partido
        string cargo
        int total_votos
        string resultado
    }

    SICONFI {
        string id_municipio FK
        int ano
        float receita_liquida
        float despesa_total
        float divida_consolidada
        float gasto_pessoal
    }

    BOLSA_FAMILIA {
        string id_municipio FK
        int ano
        int qtd_familias_beneficiadas
        float valor_total_pago
    }

    RAIS_EMPREGO {
        string id_municipio FK
        int ano
        int vinculos_ativos
        float remuneracao_media
    }

    OBRAS {
        string id_municipio FK
        string id_obra
        string status
        float valor_contratado
        float valor_pago
        date data_inicio
        date previsao_fim
    }
```

---

## Decisão de Stack por Fase

```mermaid
quadrantChart
    title Stack por Complexidade vs. Fase de Aprendizado
    x-axis Simples --> Complexo
    y-axis Fase Inicial --> Fase Avançada
    quadrant-1 Futuro
    quadrant-2 Avançado Agora
    quadrant-3 Começo
    quadrant-4 Complexo Desnecessário
    pandas: [0.15, 0.15]
    Jupyter: [0.1, 0.1]
    BD+ BigQuery: [0.2, 0.2]
    Playwright Docker: [0.45, 0.35]
    PostgreSQL: [0.35, 0.45]
    dbt: [0.45, 0.5]
    Apache Spark: [0.7, 0.7]
    Databricks: [0.75, 0.75]
    Airflow: [0.65, 0.65]
    FastAPI: [0.5, 0.8]
    Next.js: [0.55, 0.85]
    LangChain: [0.8, 0.9]
```
