# Guias de Estudo — Como Fazer Cada Tarefa

> Guias educativos explicando cada tecnologia do projeto, com exemplos práticos
> tirados diretamente do código que vamos escrever.

---

## Guias Disponíveis

| Guia | Para quem | Quando ler |
|------|-----------|-----------|
| [BigQuery + Base dos Dados](bigquery-e-bd-mais.md) | Abner e Miguel | Antes de qualquer notebook — explica como buscar dados do BD+ |
| [pandas para Análise](pandas-para-analise.md) | Miguel (principalmente) | Antes dos NB01–NB04 — merge, groupby, visualização |
| [Playwright — Scraping](playwright-scraping.md) | Abner (principalmente) | Antes da Task 8 e NB06 — como fazer scraping de portais gov |
| [Docker para Playwright](docker-para-playwright.md) | Abner | Antes da Task 8 — como containerizar o scraper |
| [Jupyter — Boas Práticas](jupyter-boas-praticas.md) | Abner e Miguel | Antes de criar qualquer notebook |

---

## Mapa de Leitura por Tarefa

```
Abner:
  Task 2 (BD+ Setup)    → bigquery-e-bd-mais.md
  Task 8 (Docker)       → docker-para-playwright.md + playwright-scraping.md
  NB05 (Salário)        → pandas-para-analise.md + jupyter-boas-praticas.md
  NB06 (Ficha Limpa)    → playwright-scraping.md

Miguel:
  NB01 (Perfil)         → bigquery-e-bd-mais.md + pandas-para-analise.md + jupyter-boas-praticas.md
  NB02 (Eleições)       → pandas-para-analise.md (seção merge e pivot)
  NB03 (BF + Emprego)   → pandas-para-analise.md (seção groupby e corr)
  NB04 (Finanças)       → pandas-para-analise.md (seção apply/lambda)
```

---

## Referências Externas Rápidas

- [BD+ Catálogo de datasets](https://basedosdados.org/dataset)
- [Plotly Express — galeria de gráficos](https://plotly.com/python/plotly-express/)
- [Playwright Python — documentação oficial](https://playwright.dev/python/docs/intro)
- [Fontes de dados pesquisadas](../fontes-de-dados/fontes-pesquisadas.md)
- [Divisão de tarefas — Abner](../divisao-tarefas/abner.md)
- [Divisão de tarefas — Miguel](../divisao-tarefas/miguel.md)
