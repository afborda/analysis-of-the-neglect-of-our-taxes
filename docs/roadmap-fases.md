# Roadmap de Fases — Progressão do Projeto

> Cada fase tem um objetivo de aprendizado principal, além dos entregáveis de dados.

---

## Linha do Tempo

```mermaid
gantt
    title Roadmap — Análise da Negligência nos Nossos Impostos
    dateFormat  YYYY-MM
    axisFormat  %b/%Y

    section Fase 1 · Piloto Acre
    Setup Google Cloud + BD+        :f1a, 2026-06, 1w
    Notebooks 01-03 (perfil/eleições/social)  :f1b, after f1a, 3w
    Notebooks 04-06 (finanças/salários/ficha) :f1c, after f1b, 3w
    Notebook 07 · IVM v1 + ranking  :f1d, after f1c, 2w
    Revisão e documentação          :f1e, after f1d, 1w

    section Fase 2 · Região Norte
    Pipeline ETL próprio            :f2a, after f1e, 3w
    PostgreSQL + dbt setup          :f2b, after f2a, 2w
    Playwright batch por estado     :f2c, after f2b, 3w
    Análise comparativa 7 estados   :f2d, after f2c, 3w

    section Fase 3 · Brasil Completo
    Spark + Databricks setup        :f3a, after f2d, 2w
    Airflow + MinIO orquestração    :f3b, after f3a, 3w
    Coleta nacional (5570 municípios):f3c, after f3b, 6w
    Rankings nacionais + dataset público :f3d, after f3c, 3w

    section Fase 4 · Produto Público
    FastAPI backend                 :f4a, after f3d, 3w
    Next.js frontend                :f4b, after f4a, 4w
    Agente IA                       :f4c, after f4b, 3w
    Deploy + domínio público        :f4d, after f4c, 1w
```

---

## O que Cada Fase Ensina

```mermaid
mindmap
  root((Projeto))
    Fase 1 · Acre
      Python para dados
        pandas
        plotly
        jupyter
      Coleta de dados
        BD+ BigQuery
        requests API REST
        Playwright básico
      Análise
        EDA exploratória
        Cruzamento de fontes
        Normalização de dados
        Índice composto IVM

    Fase 2 · Região Norte
      Engenharia de dados
        Pipeline ETL
        PostgreSQL
        SQLAlchemy
      Transformação
        dbt modelos
        SQL avançado
        Data quality checks
      Coleta em escala
        Playwright batch
        Rate limiting
        Cache inteligente

    Fase 3 · Brasil
      Big Data
        Apache Spark
        DataFrames distribuídos
        Parquet columnar
      Orquestração
        Airflow DAGs
        MinIO S3
        Databricks free
      Escala nacional
        5570 municípios
        Histórico 24 anos

    Fase 4 · Produto
      Backend
        FastAPI
        API REST pública
        Autenticação
      Frontend
        Next.js App Router
        Visualizações D3
        Dashboard por cidade
      Inteligência
        LangChain agents
        RAG sobre dados
        Perguntas em português
```

---

## Dependências Entre Fases

```mermaid
flowchart TD
    F1["Fase 1\nPiloto Acre\n✓ Metodologia IVM validada\n✓ Fontes de dados mapeadas\n✓ Notebooks funcionando"]

    F2["Fase 2\nRegião Norte\n✓ Pipeline reutilizável por UF\n✓ Schema PostgreSQL definido\n✓ dbt models prontos"]

    F3["Fase 3\nBrasil Completo\n✓ 5.570 municípios no banco\n✓ Histórico 2000-2024\n✓ Dataset público liberado"]

    F4["Fase 4\nProduto Público\n✓ Site no ar\n✓ API pública\n✓ Agente IA respondendo"]

    F1 -->|"Schema + metodologia\nvalidados no piloto"| F2
    F2 -->|"Pipeline modular\npronto para escalar"| F3
    F3 -->|"Banco completo\npara servir produto"| F4

    style F1 fill:#4CAF50,color:#fff
    style F2 fill:#2196F3,color:#fff
    style F3 fill:#FF9800,color:#fff
    style F4 fill:#9C27B0,color:#fff
```

---

## Critérios de Conclusão por Fase

### Fase 1 — Pronto quando:
- [ ] 7 notebooks executam sem erros (`jupyter nbconvert --execute`)
- [ ] IVM calculado para 22 municípios do Acre com score 0–100
- [ ] Pelo menos 1 insight surpresa documentado por notebook
- [ ] Dataset exportado em CSV + Parquet
- [ ] Metodologia do IVM documentada e revisada

### Fase 2 — Pronto quando:
- [ ] Pipeline coleta dados dos 7 estados do Norte automaticamente
- [ ] PostgreSQL rodando em Docker com schema normalizado
- [ ] dbt models passando em `dbt test`
- [ ] Ranking regional dos 7 estados publicado

### Fase 3 — Pronto quando:
- [ ] Todos os 5.570 municípios no banco
- [ ] Histórico completo 2000–2024
- [ ] Dataset público no Kaggle ou HuggingFace
- [ ] Pipeline roda em menos de 24h para atualização completa

### Fase 4 — Pronto quando:
- [ ] Site público acessível com domínio próprio
- [ ] API REST documentada no Swagger
- [ ] Agente responde 10 perguntas-teste corretamente
- [ ] Tempo de resposta < 2s para busca por município
