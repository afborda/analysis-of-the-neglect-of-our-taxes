# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open-data project (Abner + Miguel) analyzing Brazilian municipal public spending. Phase 1 pilot covers Acre (22 municipalities). Primarily Jupyter notebooks + Python scrapers — not a deployable app yet.

**GCP project:** `analise-municipios-br` (BigQuery billing)

## Commands

```bash
# Setup
pip install -r requirements.txt
playwright install chromium
cp .env.example .env  # fill GCP_PROJECT_ID, GOOGLE_APPLICATION_CREDENTIALS, TRANSPARENCIA_API_TOKEN

# Run notebooks interactively
jupyter lab

# Execute a single notebook headlessly
jupyter nbconvert --to notebook --execute notebooks/fase1_acre/00_setup_validacao.ipynb \
  --output notebooks/fase1_acre/00_setup_validacao.ipynb

# Execute all notebooks in order (order matters — see dependency chain below)
cd notebooks/fase1_acre
for nb in 00 01 02 03 04 05 06 07; do
  jupyter nbconvert --to notebook --execute ${nb}_*.ipynb \
    --output ${nb}_*.ipynb --ExecutePreprocessor.timeout=300
done

# Docker scraper (for Playwright-based scrapers)
docker build -t playwright-scraper -f docker/playwright/Dockerfile .
docker run -v $(pwd)/data:/app/data playwright-scraper scrapers/tse_ficha_limpa.py AC
```

## Architecture

### Data Collection Hierarchy

Three layers — always prefer the simplest one that has the data:

1. **BD+ / BigQuery** (`bd.read_sql()`) — primary source for most data, free tier (1 TB/month), no scraping. Uses tables like `basedosdados.br_ibge_populacao.municipio`, `br_tse_eleicoes.*`, `br_siconfi.municipios`, `br_me_rais.microdados_vinculos`, `br_mc_bolsa_familia.municipio`.
2. **API REST with token** (`requests`) — Portal da Transparência (`api.portaldatransparencia.gov.br`) for civil servant salaries. Requires `TRANSPARENCIA_API_TOKEN`. Always implement a local JSON cache to avoid redundant calls.
3. **Playwright Docker** — sites without APIs (TSE DivulgaCand for ficha limpa, Transferegov for public works). Use `scrapers/base.py` (`BaseScraper`) which enforces cache + rate limiting (1–3s random delay between requests).

### Universal Join Key

`id_municipio` (7-digit IBGE code) is the join key across **all** data sources. Acre codes all start with `12`.

### Notebook Pipeline (Phase 1)

Notebooks are **sequentially dependent** — each saves to `data/processed/` and the next reads from it:

```
00_setup_validacao  →  validates BD+ connection (22 municipalities in AC)
01_perfil_municipios  →  saves: perfil_municipios_ac.json  (base for all others)
02_eleicoes_historico  →  saves: eleicoes_prefeitos_ac.json
03_bolsa_familia_emprego  →  saves: social_emprego_ac.json
04_financas_prefeituras  →  saves: financas_ac.json
05_salario_prefeito_pop  →  saves: salarios_prefeitos_ac.json
06_ficha_limpa_prefeitos  →  saves: ficha_prefeitos_ac.json
07_ivm_ranking  →  reads all processed files → saves: ivm_ranking_ac_fase1.csv + .parquet
```

NB07 is the terminal node: it aggregates all 6 dimensions into the IVM score (0–100) using MinMax normalization + weighted sum.

### IVM — Índice de Viabilidade Municipal

Composite score (0–100) per municipal term (not per municipality). Six dimensions:

| Dimension | Weight | Direction |
|-----------|--------|-----------|
| Saúde Financeira (SICONFI) | 20% | lower debt/revenue ratio = better |
| Qualidade de Vida (IDH) | 20% | higher = better |
| Emprego e Autonomia (RAIS) | 15% | higher formal employment = better |
| Dependência Social (Bolsa Família) | 15% | **inverted** — more BF = lower score |
| Obras e Entregas (Transferegov) | 15% | higher completion % = better |
| Integridade do Gestor (TSE ficha) | 15% | binary (Ficha Limpa=1, Com Restrições=0) |

Normalization uses `sklearn.preprocessing.MinMaxScaler`. All `inverter=True` dimensions must be explicitly inverted after scaling.

### Scraper Pattern

All scrapers extend `scrapers/base.py:BaseScraper`:
- `carregar_cache(chave)` / `salvar_cache(chave, dados)` — JSON files in `data/raw/`
- `esperar()` — random sleep between requests (1–3s default)
- `coletar(**kwargs)` — abstract method to implement

### Data Directories

- `data/raw/` — gitignored; raw API responses, Playwright results, BD+ exports
- `data/processed/` — gitignored; cleaned datasets and HTML visualizations used as notebook outputs and IVM inputs

## Key Invariants

- **IDH data lag:** IDHM is from the 2010 Census (published 2013) — most recent available from PNUD. Do not treat it as current.
- **Small municipality suppression:** Cities with < 5,000 inhabitants may have RAIS data suppressed by IBGE statistical privacy rules. Treat as `NaN` and document.
- **IVM is per term, not per city:** The same municipality can have very different IVM scores across different administrations.
- **LRF thresholds:** Gasto com pessoal ≤ 45% = healthy, 45–54% = warning, > 54% = irregular (Lei de Responsabilidade Fiscal Art. 19/20).
