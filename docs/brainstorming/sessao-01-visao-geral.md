# Sessão 01 — Visão Geral e Design da Fase 1

> Data: 2026-05-25
> Participantes: Abner Fonseca (dev sênior SAP) + Miguel

---

## Contexto do Projeto

Projeto de dados abertos para analisar e publicar informações sobre os municípios brasileiros
de forma clara e acessível. Motivação: os dados existem mas estão dispersos, em formatos
difíceis e sem cruzamento. Objetivo paralelo: aprender análise de dados na prática.

---

## Perfil do Time

| Pessoa | Nível | Experiência |
|--------|-------|-------------|
| Abner Fonseca | Dev Sênior | SAP, SQL sólido, desenvolvimento full-stack |
| Miguel | Intermediário | SQL sólido, boa base técnica |

**O que querem aprender:**
- Análise exploratória de dados (EDA)
- Cruzamento de múltiplas fontes públicas
- Pandas, Spark, Databricks (free tier)
- IA e agentes sobre dados
- Backend + Frontend para visualizações

---

## Audiência Principal

**Cidadão comum** — alguém que quer pesquisar a cidade dele de forma simples.
O produto público é a consequência; o aprendizado é o objetivo primário.

---

## Decisão: Abordagem Fase 1

**Opção A escolhida:** Base dos Dados (BD+) via BigQuery como fonte principal.

- BD+ já limpou e normalizou dados de IBGE, TSE, SICONFI, CAGED, Bolsa Família
- Gratuito: 1 TB de queries/mês no free tier do Google BigQuery
- Playwright entra apenas onde não existe API (TCEs estaduais, obras, ficha limpa)
- Foco total em aprender análise e cruzamento — sem gastar tempo com ETL

**Por que não a Opção B (pipeline próprio)?**
A Fase 2 vai construir o pipeline próprio. Fase 1 é para aprender análise rápido.

---

## Mapa de Dados — BD+ vs. Coleta Própria

### Vem do BD+ (BigQuery gratuito)

| Dado | Tabela BD+ |
|------|-----------|
| Histórico de eleições 2004–2024 | `br_tse_eleicoes.candidatos` |
| Votos por candidato/município | `br_tse_eleicoes.resultados_candidato_municipio` |
| Comparecimento vs. eleitores cadastrados | `br_tse_eleicoes.perfil_eleitorado` |
| População por município por ano | `br_ibge_populacao.municipio` |
| Emprego formal + salário médio da população | `br_me_rais.microdados_vinculos` |
| Bolsa Família — % da cidade que recebe | `br_mc_bolsa_familia.municipio` |
| IDH municipal histórico | `br_ipea_idhm.municipio` |
| Receitas e despesas municipais | `br_siconfi.municipios` |

### Precisa de coleta própria (Playwright + APIs)

| Dado | Fonte | Método |
|------|-------|--------|
| Salário do prefeito | Portal da Transparência | API REST (token gratuito) |
| Salário de servidores municipais | SICONFI + portais municipais | Playwright |
| Obras por status (paralisada/concluída) | Transferegov | Playwright |
| Ficha limpa / condenações do prefeito | TSE DivulgaCand | Playwright |

**Chave de cruzamento universal:** `id_municipio` (código IBGE 7 dígitos)
Todos os datasets usam esse mesmo código — é o JOIN key entre todas as fontes.

---

## Como Conectar ao BD+ (Setup)

### 1. Criar projeto Google Cloud gratuito

1. Acesse https://console.cloud.google.com
2. Crie projeto: `analise-municipios-br`
3. Ative a API do BigQuery (gratuito)
4. Crie Service Account com permissão `BigQuery Job User`
5. Baixe o `.json` da chave

### 2. Instalar dependências

```bash
pip install basedosdados pandas jupyter plotly
```

### 3. Primeira query — população do Acre (estado piloto)

```python
import basedosdados as bd
import pandas as pd

df_populacao = bd.read_sql(
    query="""
        SELECT
            id_municipio,
            nome,
            ano,
            populacao
        FROM `basedosdados.br_ibge_populacao.municipio`
        WHERE sigla_uf = 'AC'
        ORDER BY ano DESC, populacao DESC
    """,
    billing_project_id="analise-municipios-br"
)
```

### 4. Primeiro cruzamento — eleições + população + eleitores

```python
df_cruzado = bd.read_sql(
    query="""
        SELECT
            p.id_municipio,
            p.nome AS municipio,
            p.populacao,
            e.ano_eleicao,
            e.nome_candidato,
            e.sigla_partido,
            e.resultado,
            e.total_votos,
            el.qtd_eleitores_perfil AS eleitores_cadastrados
        FROM `basedosdados.br_ibge_populacao.municipio` p
        LEFT JOIN `basedosdados.br_tse_eleicoes.resultados_candidato_municipio` e
            ON p.id_municipio = e.id_municipio
            AND e.cargo = 'PREFEITO'
            AND e.ano_eleicao = 2024
        LEFT JOIN `basedosdados.br_tse_eleicoes.perfil_eleitorado` el
            ON p.id_municipio = el.id_municipio
            AND el.ano = 2024
        WHERE p.sigla_uf = 'AC'
          AND p.ano = 2024
        ORDER BY p.populacao DESC
    """,
    billing_project_id="analise-municipios-br"
)
```

**Esse cruzamento já responde:**
- Quem ganhou em cada cidade do Acre em 2024
- Quantos votos teve
- Eleitores cadastrados vs. população total
- % de comparecimento por município

---

## Estado Piloto

**Acre (AC)** — 22 municípios
Motivo: menor estado em número de municípios, ideal para validar o pipeline antes de escalar.

---

## Seção 4 — Playwright em Docker (Servidor)

Playwright roda num container Docker no servidor do Abner.
Motivo: scraping de portais gov que renderizam JS (TCEs, Transferegov, TSE DivulgaCand).

### Dockerfile base para Playwright

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.44.0-jammy
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY scrapers/ ./scrapers/
CMD ["python", "scrapers/run_all.py"]
```

### Padrão de cache local obrigatório

```python
# Nunca rodar Playwright sem verificar cache primeiro
cache = Path("data/raw/obras_ac.json")
if not cache.exists():
    dados = scraper.coletar()
    dados.to_json(cache)
```

---

## Escopo Completo — Brasil Inteiro com Histórico

### Decisão de escopo ampliado

O projeto cobre **todos os 5.570 municípios + 26 estados + DF**, com histórico desde 2000.

### Análises históricas planejadas

**Municípios:**
- Quais partidos governaram cada cidade (histórico por mandato)
- Quais prefeitos mais contraíram dívidas
- Ranking de "piores gestões" por índice composto
- Evolução de cada cidade ao longo das gestões

**Estados:**
- Quais partidos governaram cada estado
- Quais governadores mais endividaram seus estados
- Comparativo entre estados por gestão

### O cruzamento mais importante: Mandato × Dívida × Partido

A lógica temporal do JOIN:

```
Prefeito eleito em 2020 → governa 2021–2024
SICONFI: dívida acumulada 2021, 2022, 2023, 2024
JOIN: id_municipio + ano_dentro_do_mandato
```

Tabelas envolvidas:
- `br_tse_eleicoes.resultados_candidato_municipio` → quem foi eleito, quando
- `br_siconfi.municipios` → dívida, receita, despesa por ano
- Calcular `ano_inicio_mandato` e `ano_fim_mandato` a partir do `ano_eleicao`

Para governadores: mesma lógica com `cargo = 'GOVERNADOR'` + SICONFI estadual.

### O que vem do BD+ (sem Playwright) — Brasil inteiro

| Dado | Tabela BD+ | Cobertura |
|------|-----------|-----------|
| Todos os prefeitos eleitos | `br_tse_eleicoes.resultados_candidato_municipio` | 1996–2024 |
| Todos os governadores eleitos | `br_tse_eleicoes.resultados_candidato_municipio` | 1994–2024 |
| Dívida municipal por ano | `br_siconfi.municipios` | 2000–2024 |
| Dívida estadual por ano | `br_siconfi.estados` | 2000–2024 |
| Filiação partidária | `br_tse_eleicoes.candidatos` | 1994–2024 |
| Bolsa Família por município | `br_mc_bolsa_familia.municipio` | 2004–2024 |
| Emprego formal por município | `br_me_rais.microdados_vinculos` | 2000–2024 |
| IDH por município | `br_ipea_idhm.municipio` | 1991–2010 |
| População por município | `br_ibge_populacao.municipio` | 1991–2024 |

### O que precisa do Playwright Docker — Brasil inteiro

| Dado | Volume | Estratégia |
|------|--------|-----------|
| Ficha limpa de ~60k candidatos históricos | Alto | Batch por estado, Docker paralelo |
| Obras por município (Transferegov) | Alto | Fila de jobs, 1 req/seg para não bloquear |
| Salário de servidores municipais | Médio | API Portal da Transparência (tem rate limit) |

---

## Próximas Seções a Documentar

- [ ] Seção 5: Roadmap de fases completo com stack por fase
- [ ] Seção 6: Métricas para "pior/melhor prefeito" — definição do índice composto
