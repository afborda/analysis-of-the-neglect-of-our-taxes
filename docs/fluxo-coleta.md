# Fluxo de Coleta de Dados

> Para cada dado que precisamos, há uma decisão: usar BD+ (rápido e gratuito)
> ou coletar por conta própria com Playwright/API (mais trabalhoso, mais controle).

---

## Árvore de Decisão: Como Coletar Cada Dado

```mermaid
flowchart TD
    START([Preciso de um dado]) --> Q1{Está disponível\nno BD+?}

    Q1 -->|Sim| Q2{Está atualizado\n< 1 ano?}
    Q2 -->|Sim| BD["✅ Usar BD+\nbd.read_sql()"]
    Q2 -->|Não| Q3{Tem API REST\npública?}

    Q1 -->|Não| Q3

    Q3 -->|Sim| Q4{Precisa de\ntoken/cadastro?}
    Q4 -->|Não| API_FREE["✅ requests\nsem autenticação"]
    Q4 -->|Sim| API_TOKEN["✅ requests\ncom token gratuito"]

    Q3 -->|Não| Q5{Site renderiza\nJavaScript?}
    Q5 -->|Não| BEAUTIFULSOUP["✅ requests +\nBeautifulSoup"]
    Q5 -->|Sim| PLAYWRIGHT["✅ Playwright\nDocker no servidor"]

    style BD fill:#4CAF50,color:#fff
    style API_FREE fill:#4CAF50,color:#fff
    style API_TOKEN fill:#2196F3,color:#fff
    style BEAUTIFULSOUP fill:#FF9800,color:#fff
    style PLAYWRIGHT fill:#f44336,color:#fff
```

---

## Mapa Completo: Dado → Método

```mermaid
flowchart LR
    subgraph BD_PLUS["✅ BD+ BigQuery — Gratuito, sem configuração extra"]
        direction TB
        P1["População por município/ano\nbr_ibge_populacao.municipio"]
        P2["IDH Municipal histórico\nbr_ipea_idhm.municipio"]
        P3["Eleições 1996–2024\nbr_tse_eleicoes.*"]
        P4["Emprego formal + salário\nbr_me_rais.microdados_vinculos"]
        P5["Saldo de empregos mensal\nbr_me_caged.microdados"]
        P6["Bolsa Família por município\nbr_mc_bolsa_familia.municipio"]
        P7["Finanças municipais\nbr_siconfi.municipios"]
        P8["CNPJs ativos\nbr_rf_cnpj.estabelecimentos"]
    end

    subgraph API_REST["🔵 API REST — Token gratuito necessário"]
        direction TB
        A1["Salário de servidores federais\nPortal da Transparência API"]
        A2["Convênios e transferências\nPortal da Transparência API"]
        A3["Obras federais\nPortal da Transparência API"]
    end

    subgraph PLAYWRIGHT_BOX["🔴 Playwright Docker — Para sites sem API"]
        direction TB
        PW1["Ficha limpa de candidatos\nTSE DivulgaCand"]
        PW2["Obras paralisadas + status\nTransferegov"]
        PW3["Contas aprovadas/reprovadas\nTCEs estaduais (26 portais)"]
        PW4["Salário servidores municipais\nPortais municipais de transparência"]
    end
```

---

## Como Funciona o BD+ na Prática

```mermaid
sequenceDiagram
    participant Notebook as Jupyter Notebook
    participant SDK as BD+ Python SDK
    participant GCP as Google Cloud\n(sua conta gratuita)
    participant BQ as BigQuery\n(projeto basedosdados)

    Notebook->>SDK: bd.read_sql(query, billing_project_id)
    SDK->>GCP: Autentica com service account .json
    GCP->>BQ: Executa query no projeto basedosdados
    BQ-->>GCP: Resultado (cobrado na sua cota gratuita)
    GCP-->>SDK: DataFrame com os dados
    SDK-->>Notebook: pandas DataFrame pronto para usar

    Note over GCP,BQ: 1 TB/mês gratuito<br/>Uma query típica usa ~50-200MB<br/>Centenas de queries por mês sem custo
```

---

## Como Funciona o Playwright Docker

```mermaid
sequenceDiagram
    participant Script as Python Script
    participant Docker as Container Docker\n(servidor Abner)
    participant Cache as Cache Local\ndata/raw/
    participant Site as Site Governamental\n(TSE, Transferegov, TCE)

    Script->>Cache: Verifica cache local
    alt Cache existe
        Cache-->>Script: Retorna dados salvos
    else Cache não existe
        Script->>Docker: Inicia Playwright (headless Chromium)
        Docker->>Site: Abre URL no browser real
        Site-->>Docker: Renderiza JavaScript
        Docker->>Site: Preenche formulários / navega
        Site-->>Docker: Dados carregados na página
        Docker-->>Script: Extrai dados do DOM
        Script->>Cache: Salva JSON no disco
        Script-->>Script: Usa dados coletados
    end
```

---

## Configuração do Docker para Playwright

```mermaid
flowchart TD
    subgraph SERVIDOR["Servidor do Abner"]
        subgraph DOCKER["Container Docker"]
            PW_IMG["Imagem base:\nmcr.microsoft.com/playwright/python"]
            CHROME["Chromium\npré-instalado"]
            SCRIPT["scrapers/\ntse_ficha_limpa.py\ntransferegov_obras.py\ntce_contas.py"]
        end

        subgraph VOLUMES["Volumes montados"]
            RAW["data/raw/\n(cache JSON)"]
            LOGS["logs/\n(erros e progresso)"]
        end

        DOCKER --> VOLUMES
    end

    EXTERNO["Portais governamentais\n(internet)"]
    DOCKER <-->|"HTTP/HTTPS\nheadless browser"| EXTERNO
```

**Comando para rodar:**
```bash
docker build -t playwright-scraper ./docker/playwright/

# Roda o scraper de ficha limpa para o Acre
docker run -v $(pwd)/data:/app/data playwright-scraper \
  python scrapers/tse_ficha_limpa.py --uf AC

# Roda todos os scrapers em paralelo por estado
docker run -v $(pwd)/data:/app/data playwright-scraper \
  python scrapers/run_all.py --uf AC --workers 3
```

---

## Rate Limiting e Boa Conduta

```mermaid
flowchart LR
    REQ["Requisição\nao portal"] --> WAIT["Aguardar\n1–3 segundos\nentre requests"]
    WAIT --> CHECK{Portal\nbloqueou?}
    CHECK -->|Não| NEXT["Próxima\npágina/cidade"]
    CHECK -->|Sim| BACKOFF["Exponential\nbackoff:\n5s → 10s → 30s"]
    BACKOFF --> RETRY["Tentar\nnovamente"]
    RETRY --> CHECK

    note["⚠️ Portais governamentais são\nrecursos públicos compartilhados.\nSempre usar rate limiting.\nNunca scraping agressivo."]
```

**Padrão de implementação:**
```python
import time
import random

def requisicao_com_espera(page, url: str, espera_min=1.0, espera_max=3.0):
    page.goto(url)
    page.wait_for_load_state("networkidle")
    # Espera aleatória para não parecer bot
    time.sleep(random.uniform(espera_min, espera_max))
```
