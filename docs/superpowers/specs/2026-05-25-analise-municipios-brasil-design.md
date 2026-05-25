# Design Spec — Análise da Negligência nos Nossos Impostos

**Data:** 2026-05-25
**Autores:** Abner Fonseca + Miguel
**Status:** Aprovado para implementação

---

## 1. Objetivo

Construir uma plataforma de dados abertos que coleta, cruza e publica informações sobre os
5.570 municípios brasileiros e seus governantes — tornando acessível o que hoje está disperso
em dezenas de portais públicos sem cruzamento.

**Objetivo paralelo e primário:** aprender análise de dados, cruzamento de fontes, pandas,
Spark, Playwright, backend e frontend fazendo um projeto real com impacto público.

---

## 2. Audiência

| Fase | Audiência | Produto |
|------|-----------|---------|
| 1–3 | Time (Abner + Miguel) | Notebooks Jupyter com análises e rankings |
| 4 | Cidadão comum | Site público com busca por cidade/prefeito |
| 4 | Jornalistas e ONGs | API pública + dataset no Kaggle |

---

## 3. Perfil do Time

| Pessoa | Nível | Background |
|--------|-------|-----------|
| Abner Fonseca | Sênior | Dev SAP, SQL sólido, full-stack |
| Miguel | Intermediário | SQL sólido, boa base técnica |

---

## 4. Escopo de Dados

### 4.1 Municípios — 5.570 cidades, histórico 2000–2024

| Dimensão | Dados | Fonte |
|----------|-------|-------|
| Perfil | Nome, UF, código IBGE, área, região | IBGE |
| Demográfico | População por ano, % urbana/rural | IBGE |
| Qualidade de vida | IDH, IDHM Educação/Renda/Longevidade | PNUD/IPEA |
| Emprego | % carteira assinada, salário médio formal | RAIS/CAGED |
| Dependência social | % Bolsa Família, evolução por mandato | MDS |
| Finanças | Receita, despesa, dívida, gasto por função | SICONFI/FINBRA |
| Obras | Concluídas, paralisadas, valor contratado vs. pago | Transferegov |
| Política | Prefeito, partido, histórico de gestões | TSE |
| Integridade | Ficha limpa, contas aprovadas/reprovadas | TSE/TCEs |
| Empresas | Nº CNPJs ativos, MEIs por município | Receita Federal |

### 4.2 Estados — 26 UFs + DF, histórico 2000–2024

Mesmas dimensões financeiras e políticas, com `cargo = 'GOVERNADOR'` no TSE
e dados de SICONFI estadual.

---

## 5. Arquitetura por Fase

### Fase 1 — Piloto: Acre (22 municípios)

**Objetivo de aprendizado:** EDA, pandas, cruzamento de fontes, visualização

```
Fontes                    Coleta              Armazenamento    Análise
──────────────────────────────────────────────────────────────────────
BD+ BigQuery          →   bd.read_sql()   →   CSV local    →   Jupyter
Portal Transparência  →   requests        →   CSV local    →   pandas
TSE DivulgaCand       →   Playwright      →   JSON local   →   pandas
Transferegov          →   Playwright      →   JSON local   →   pandas
```

**Entregáveis:**
- 7 notebooks com análises completas do Acre
- Índice de Viabilidade Municipal (IVM) v1
- Ranking das 22 cidades com score 0–100
- Documento de metodologia do IVM

### Fase 2 — Região Norte (450 municípios)

**Objetivo de aprendizado:** ETL próprio, PostgreSQL, dbt, Playwright em batch

```
Fontes              Coleta                 Armazenamento        Análise
──────────────────────────────────────────────────────────────────────
APIs + Playwright → pipeline Python    →   PostgreSQL (Docker) → dbt
                    por estado/lote       + Parquet (backup)     + Jupyter
```

**Entregáveis:**
- Pipeline modular reutilizável por UF
- Banco PostgreSQL com schema normalizado
- Análise comparativa entre os 7 estados do Norte
- Rankings regionais por estado

### Fase 3 — Brasil Completo (5.570 municípios)

**Objetivo de aprendizado:** Spark, Databricks free, big data, orquestração

```
Fontes          Coleta              Storage         Transformação    Análise
────────────────────────────────────────────────────────────────────────────
Todas as    →   Playwright      →   MinIO (S3)  →   Spark + dbt  →   Spark
fontes          (Airflow jobs)      Parquet          transformações   Databricks
```

**Entregáveis:**
- Cobertura completa: 5.570 municípios × histórico 2000–2024
- Rankings nacionais de prefeitos, partidos, estados
- Dataset público no Kaggle e HuggingFace
- Atualização automática pós-eleições

### Fase 4 — Produto Público + IA

**Objetivo de aprendizado:** FastAPI, Next.js, agentes de dados com LLM

```
Banco          Backend          Frontend         IA
────────────────────────────────────────────────────────
PostgreSQL  →  FastAPI      →   Next.js      →   LangChain/LlamaIndex
(Fase 2/3)     API REST         Dashboard        Agente sobre os dados
               + Auth           Visualizações    Perguntas em linguagem
                                por cidade       natural
```

**Entregáveis:**
- Site público com busca por cidade/prefeito/partido
- Dashboard de cada município com todos os indicadores
- API pública com documentação (Swagger)
- Agente: responde perguntas como *"Qual partido mais endividou o Nordeste?"*

---

## 6. Índice de Viabilidade Municipal (IVM)

Score de 0 (pior gestão) a 100 (melhor gestão) por mandato.

### 6.1 Dimensões e Pesos

| Dimensão | Peso | Métricas | Fonte |
|----------|------|----------|-------|
| Saúde Financeira | 20% | Dívida/receita líquida; gasto pessoal ≤54% (LRF); contas aprovadas pelo TCE | SICONFI + TCEs |
| Qualidade de Vida | 20% | IDHM absoluto; variação do IDHM durante o mandato | PNUD/IPEA |
| Emprego e Autonomia | 15% | % emprego formal; salário médio formal vs. custo de vida | RAIS/CAGED |
| Dependência Social | 15% | % população em Bolsa Família; evolução vs. mandato anterior | MDS |
| Obras e Entregas | 15% | % obras concluídas; % valor paralisado sobre total contratado | Transferegov |
| Integridade do Gestor | 15% | Ficha limpa (binário); contas reprovadas; condenações ativas | TSE/TCEs |

### 6.2 Normalização

Todas as métricas são normalizadas para 0–1 antes de aplicar os pesos:

```python
from sklearn.preprocessing import MinMaxScaler

# Exemplo: normalizar dívida/receita para o universo nacional
scaler = MinMaxScaler()
df["divida_score"] = scaler.fit_transform(df[["divida_sobre_receita"]])
# Inverter: mais dívida = score menor
df["divida_score"] = 1 - df["divida_score"]
```

### 6.3 Cálculo Final

```python
df["ivm"] = (
    df["saude_financeira"]  * 0.20 +
    df["qualidade_vida"]    * 0.20 +
    df["emprego_autonomia"] * 0.15 +
    df["dependencia_social"]* 0.15 +
    df["obras_entregas"]    * 0.15 +
    df["integridade_gestor"]* 0.15
) * 100  # escala 0–100
```

---

## 7. Estrutura de Notebooks — Fase 1

| Notebook | Pergunta Central | Técnica Aprendida |
|----------|-----------------|-------------------|
| `01_perfil_municipios_ac.ipynb` | Como são os municípios do Acre? | BD+, limpeza, plotly |
| `02_eleicoes_historico_ac.ipynb` | Quem governa e como chegou lá? | JOINs, groupby, histórico |
| `03_bolsa_familia_emprego_ac.ipynb` | Qual a dependência social? | 3+ fontes, correlações |
| `04_financas_prefeituras_ac.ipynb` | As prefeituras gastam bem? | Análise financeira, índices |
| `05_salario_prefeito_vs_populacao.ipynb` | O prefeito ganha quanto? | API REST + merge local |
| `06_ficha_limpa_prefeitos_ac.ipynb` | Quem são os prefeitos? | Playwright, parsing HTML |
| `07_visao_geral_viabilidade_ac.ipynb` | Ranking de viabilidade | IVM, normalização, ranking |

---

## 8. Coleta de Dados — Decisão por Fonte

### 8.1 BD+ BigQuery (gratuito, sem Playwright)

```python
import basedosdados as bd

df = bd.read_sql("""
    SELECT p.id_municipio, p.nome, p.populacao,
           e.nome_candidato, e.sigla_partido, e.resultado, e.total_votos,
           el.qtd_eleitores_perfil AS eleitores_cadastrados
    FROM `basedosdados.br_ibge_populacao.municipio` p
    LEFT JOIN `basedosdados.br_tse_eleicoes.resultados_candidato_municipio` e
        ON p.id_municipio = e.id_municipio AND e.cargo = 'PREFEITO'
    LEFT JOIN `basedosdados.br_tse_eleicoes.perfil_eleitorado` el
        ON p.id_municipio = el.id_municipio AND el.ano = e.ano_eleicao
    WHERE p.sigla_uf = 'AC'
""", billing_project_id="analise-municipios-br")
```

**Chave universal de JOIN:** `id_municipio` (código IBGE 7 dígitos)

### 8.2 Playwright em Docker (dados sem API)

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.44.0-jammy
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY scrapers/ ./scrapers/
CMD ["python", "scrapers/run_all.py"]
```

**Padrão obrigatório de cache local:**
```python
cache = Path(f"data/raw/ficha_limpa_{id_municipio}.json")
if not cache.exists():
    dados = scraper.coletar(id_municipio)
    dados.to_json(cache)
```

**Dados coletados via Playwright:**

| Dado | Site | Volume |
|------|------|--------|
| Ficha limpa de candidatos | TSE DivulgaCand | ~60k por ciclo eleitoral |
| Obras por município | Transferegov | 5.570 municípios |
| Salário de servidores municipais | Portais municipais | Variável |

---

## 9. Cruzamento Mandato × Dívida × Partido

O cruzamento mais sofisticado do projeto:

```python
# 1. Calcular anos de mandato a partir da eleição
mandatos = df_eleicoes[df_eleicoes["resultado"] == "ELEITO"].copy()
mandatos["ano_inicio"] = mandatos["ano_eleicao"] + 1
mandatos["ano_fim"]    = mandatos["ano_eleicao"] + 4

# 2. Explodir por ano de mandato
mandatos_anos = mandatos.assign(
    ano=mandatos.apply(
        lambda r: range(r["ano_inicio"], r["ano_fim"] + 1), axis=1
    )
).explode("ano")

# 3. JOIN com SICONFI por município + ano
df_gestao = mandatos_anos.merge(
    df_siconfi[["id_municipio", "ano", "divida_consolidada", "receita_liquida"]],
    on=["id_municipio", "ano"]
)

# 4. Dívida acumulada por mandato
df_gestao["divida_sobre_receita"] = (
    df_gestao["divida_consolidada"] / df_gestao["receita_liquida"]
)
ranking = (df_gestao
    .groupby(["nome_candidato", "sigla_partido", "id_municipio"])
    ["divida_sobre_receita"].mean()
    .sort_values(ascending=False)
    .reset_index()
)
```

Mesmo padrão para governadores — `cargo = 'GOVERNADOR'` + SICONFI estadual.

---

## 10. Estrutura de Pastas do Projeto

```
analysis-of-the-neglect-of-our-taxes/
├── README.md
├── docs/
│   ├── fontes-de-dados/
│   │   └── fontes-pesquisadas.md
│   ├── brainstorming/
│   │   └── sessao-01-visao-geral.md
│   ├── estudos/
│   └── superpowers/
│       └── specs/
│           └── 2026-05-25-analise-municipios-brasil-design.md  ← este arquivo
├── notebooks/              ← Jupyter notebooks por fase
│   ├── fase1_acre/
│   ├── fase2_norte/
│   └── fase3_brasil/
├── scrapers/               ← Playwright collectors
│   ├── tse_ficha_limpa.py
│   ├── transferegov_obras.py
│   └── run_all.py
├── pipeline/               ← ETL (Fase 2+)
│   ├── collectors/
│   ├── transformers/
│   └── loaders/
├── data/
│   ├── raw/                ← dados brutos (no .gitignore)
│   └── processed/          ← dados limpos (no .gitignore)
└── docker/
    └── playwright/
        └── Dockerfile
```

---

## 11. O que NÃO está no escopo desta fase

- Dashboard web (Fase 4)
- API pública (Fase 4)
- Agente de IA (Fase 4)
- Spark e Databricks (Fase 3)
- Cobertura fora do Acre (Fase 2+)

---

## 12. Definição de Sucesso — Fase 1

- [ ] 7 notebooks rodando do zero com `jupyter nbconvert --execute`
- [ ] Índice IVM calculado para os 22 municípios do Acre
- [ ] Pelo menos 1 insight surpreendente documentado por notebook
- [ ] Metodologia do IVM escrita e revisada
- [ ] Dataset final exportado em CSV + Parquet
