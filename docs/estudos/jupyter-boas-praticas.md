# Guia: Jupyter Notebooks — Boas Práticas

> **O que você vai aprender:** como o Jupyter funciona, como organizar células,
> como rodar, depurar e commitar notebooks do jeito certo.

---

## 1. O que é um Jupyter Notebook?

Um notebook é um arquivo `.ipynb` que mistura **código Python** e **texto explicativo**
numa sequência de células. Você executa cada célula e vê o resultado abaixo dela.

```
┌─────────────────────────────────────────┐
│ # Célula Markdown                       │
│ ## Análise de População                 │
│ Buscando dados do IBGE via BD+...       │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐  ← Célula de Código
│ import basedosdados as bd               │
│ df = bd.read_sql("SELECT...", ...)      │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐  ← Output (gerado ao rodar)
│ Out[1]: DataFrame com 22 linhas         │
│         municipio    populacao  ano     │
│         Rio Branco   413418    2022     │
│         ...                            │
└─────────────────────────────────────────┘
```

**Vantagem para análise de dados:** você pode rodar uma query pesada uma vez, salvar
o resultado em uma variável e explorar sem pagar novamente pelo BigQuery.

---

## 2. Como rodar o Jupyter

```bash
# Na raiz do projeto
jupyter notebook
# Abre no browser: http://localhost:8888

# Ou com interface mais moderna:
jupyter lab
# Abre no browser: http://localhost:8888/lab
```

Para rodar um notebook sem abrir interface (útil para scripts):

```bash
# Executar e salvar o output no próprio arquivo
jupyter nbconvert \
  --to notebook \
  --execute \
  notebooks/fase1_acre/01_perfil_municipios_ac.ipynb

# Executar e salvar como HTML (para compartilhar)
jupyter nbconvert \
  --to html \
  --execute \
  notebooks/fase1_acre/01_perfil_municipios_ac.ipynb
```

---

## 3. Atalhos de teclado essenciais

### No modo de comando (Esc para entrar)

| Atalho | O que faz |
|--------|-----------|
| `Enter` | Entrar no modo edição da célula |
| `A` | Inserir célula **A**cima |
| `B` | Inserir célula a**B**aixo |
| `D D` | Deletar célula (pressionar D duas vezes) |
| `M` | Converter célula para **M**arkdown |
| `Y` | Converter célula para código (**Y** = code) |
| `Shift+Enter` | Rodar célula e ir para próxima |
| `Ctrl+Enter` | Rodar célula e ficar nela |
| `Z` | Desfazer deleção de célula |

### No modo edição (Enter para entrar)

| Atalho | O que faz |
|--------|-----------|
| `Shift+Enter` | Rodar célula e ir para próxima |
| `Ctrl+Enter` | Rodar célula e ficar nela |
| `Esc` | Voltar ao modo de comando |
| `Tab` | Autocompletar (funciona para colunas de DataFrame!) |

---

## 4. Estrutura padrão dos nossos notebooks

Todo notebook do projeto segue esta estrutura de células:

```python
# ── CÉLULA 1: Título e contexto (Markdown) ──────────────────────────────────
```
```markdown
# NB01 — Perfil dos Municípios do Acre

**Pergunta:** Como são os municípios do Acre? Tamanho, IDH, distribuição.

**Fontes:**
- `basedosdados.br_ibge_populacao.municipio`
- `basedosdados.br_ipea_idhm.municipio`

**Output:** `data/processed/perfil_municipios_ac.json`
```

```python
# ── CÉLULA 2: Imports ────────────────────────────────────────────────────────
import os
from dotenv import load_dotenv
import basedosdados as bd
import pandas as pd
import plotly.express as px

load_dotenv()
PROJECT_ID = os.getenv("GCP_PROJECT_ID")
print(f"Project: {PROJECT_ID}")  # confirmar que .env carregou
```

```python
# ── CÉLULA 3: Coleta de dados ────────────────────────────────────────────────
df_pop = bd.read_sql("""
    SELECT id_municipio, nome, ano, populacao
    FROM `basedosdados.br_ibge_populacao.municipio`
    WHERE sigla_uf = 'AC'
    ORDER BY ano DESC, populacao DESC
""", billing_project_id=PROJECT_ID)

# Salvar raw imediatamente — nunca perder uma query cara
df_pop.to_json("data/raw/populacao_ac.json", orient="records")
print(f"Coletados: {df_pop.shape[0]} registros")
df_pop.head()
```

```python
# ── CÉLULA 4: Inspeção ───────────────────────────────────────────────────────
print("Shape:", df_pop.shape)
print("\nTipos:")
print(df_pop.dtypes)
print("\nNulos:")
print(df_pop.isnull().sum())
df_pop.describe()
```

```python
# ── CÉLULA 5: Limpeza ───────────────────────────────────────────────────────
df = df_pop[df_pop["ano"] == df_pop["ano"].max()].copy()
df = df.dropna(subset=["id_municipio"])
print(f"Municípios únicos: {df['id_municipio'].nunique()}")
```

```python
# ── CÉLULA 6: Análise ───────────────────────────────────────────────────────
ranking = df.sort_values("populacao", ascending=False)
print("Top 5 por população:")
print(ranking[["nome", "populacao"]].head())
```

```python
# ── CÉLULA 7: Visualização ──────────────────────────────────────────────────
fig = px.bar(
    ranking.head(10),
    x="nome",
    y="populacao",
    title="Top 10 Municípios do Acre por População",
    labels={"nome": "Município", "populacao": "População"}
)
fig.show()
```

```python
# ── CÉLULA 8: Validações ────────────────────────────────────────────────────
assert df.shape[0] == 22, f"Esperava 22 municípios, encontrou {df.shape[0]}"
assert df["populacao"].gt(0).all(), "Tem município com população zero"
assert df["id_municipio"].nunique() == 22, "id_municipio com duplicatas"
print("✅ Todas as validações passaram")
```

```python
# ── CÉLULA 9: Salvar ────────────────────────────────────────────────────────
df.to_json(
    "data/processed/perfil_municipios_ac.json",
    orient="records",
    force_ascii=False  # manter acentos
)
print("✅ Salvo em data/processed/perfil_municipios_ac.json")
```

---

## 5. A armadilha do estado global

O Jupyter mantém todas as variáveis na memória enquanto o kernel estiver rodando.
Isso cria um perigo silencioso:

```python
# Célula 1 (rodada primeiro)
df = pd.read_json("data/processed/populacao.json")

# Célula 2 (rodada depois, mas depois você edita a célula 1)
# Se você rodar célula 2 sem rodar a célula 1 de novo,
# df ainda tem o valor antigo em memória!
df_filtrado = df[df["populacao"] > 50000]
```

**A regra de ouro:** antes de commitar ou compartilhar um notebook, sempre:

```
Kernel → Restart & Run All
```

Isso garante que o notebook roda do zero, em ordem, sem nenhum estado "fantasma".
Se falhar, você tem um bug real para corrigir.

---

## 6. Salvar dados raw logo após a query

```python
# PADRÃO CORRETO — salvar imediatamente após a query
df_raw = bd.read_sql("SELECT ...", billing_project_id=PROJECT_ID)
df_raw.to_json("data/raw/nome_dado.json", orient="records")  # ← imediatamente

# Depois você trabalha com df_raw ou recarrega do cache
df = pd.read_json("data/raw/nome_dado.json")  # se kernel reiniciar, não perde
```

**Por que isso importa?** Se o kernel cair, você perde tudo que estava na memória.
Mas se salvou o raw logo após a query, não precisa rodar o BigQuery de novo.
Queries grandes podem demorar minutos e custam bytes do seu 1 TB mensal gratuito.

---

## 7. Verificar antes de gastar

```python
# ANTES de rodar uma query grande, verificar com LIMIT
df_amostra = bd.read_sql("""
    SELECT *
    FROM `basedosdados.br_me_rais.microdados_vinculos`
    WHERE sigla_uf = 'AC'
    LIMIT 5
""", billing_project_id=PROJECT_ID)

print(df_amostra.dtypes)   # ver colunas disponíveis
print(df_amostra.head())   # ver estrutura dos dados
```

Só depois de entender a estrutura, rodar a query completa sem `LIMIT`.

---

## 8. Separar células longas em várias células menores

**Ruim — célula monolítica:**

```python
# Tudo numa célula = difícil de debugar
df = bd.read_sql("...", billing_project_id=PROJECT_ID)
df = df[df["ano"] == 2022]
df["perc_bf"] = df["familias_bf"] * 3.5 / df["populacao"] * 100
fig = px.scatter(df, x="perc_bf", y="idhm")
fig.show()
df.to_json("data/processed/social.json", orient="records")
```

**Melhor — células separadas por responsabilidade:**

```python
# Célula 1: Coleta
df_raw = bd.read_sql("...", billing_project_id=PROJECT_ID)
df_raw.to_json("data/raw/social.json", orient="records")
```

```python
# Célula 2: Limpeza
df = pd.read_json("data/raw/social.json")
df = df[df["ano"] == 2022].copy()
print(f"Registros após filtro: {df.shape[0]}")
```

```python
# Célula 3: Calcular métrica
df["perc_bf"] = (df["familias_bf"] * 3.5 / df["populacao"] * 100).round(1)
print(df[["nome", "perc_bf"]].sort_values("perc_bf", ascending=False).head())
```

```python
# Célula 4: Gráfico
fig = px.scatter(df, x="perc_bf", y="idhm", hover_name="nome",
                 title="Bolsa Família vs IDH — Acre")
fig.show()
```

```python
# Célula 5: Salvar
df.to_json("data/processed/social.json", orient="records", force_ascii=False)
print("✅ Salvo")
```

Se o gráfico deu errado, você só re-roda a célula 4 — sem pagar pelo BigQuery de novo.

---

## 9. Autocompletar para explorar DataFrames

O Jupyter tem autocompletar com `Tab`:

```python
df.      # pressionar Tab aqui mostra todos os métodos e atributos
df["    # pressionar Tab aqui mostra os nomes das colunas disponíveis!
```

Muito útil para descobrir colunas sem ter que olhar `df.columns` toda hora.

---

## 10. Usar `display()` para mostrar múltiplas saídas

Por padrão, só o último valor da célula é exibido. Para mostrar mais:

```python
# SEM display — só mostra df2
df1.head()    # não aparece
df2.head()    # aparece
```

```python
# COM display — mostra ambos
from IPython.display import display
display(df1.head())   # aparece
display(df2.head())   # aparece
```

---

## 11. Erros comuns e o que significam

| Erro | O que está acontecendo | Solução |
|------|------------------------|---------|
| `NameError: name 'df' is not defined` | Célula de coleta não foi rodada | Rodar as células anteriores ou `Restart & Run All` |
| `KeyError: 'nome_coluna'` | Coluna não existe no DataFrame atual | `print(df.columns)` para ver o que tem |
| `kernel died` | Memória insuficiente ou crash | Reduzir o tamanho da query com `LIMIT` ou `WHERE` mais restrito |
| `ConnectionError` ao rodar bd.read_sql | Credenciais não carregadas | Verificar `load_dotenv()` e `GCP_PROJECT_ID` |
| Gráfico não aparece | Plotly não renderizando | Atualizar: `jupyter nbextension enable --py widgetsnbextension` |
| Notebook com output gigante commitado | O output ficou salvo no `.ipynb` | Limpar: `Kernel → Restart & Clear Output` antes de commitar |

---

## 12. Commitar notebooks sem output

Outputs de notebooks incluem gráficos em base64 e tabelas inteiras — podem deixar o
commit gigante. O padrão do nosso projeto:

```bash
# Antes de commitar: limpar outputs
# No Jupyter: Kernel → Restart & Clear Output

# Verificar que não tem output no diff
git diff notebooks/fase1_acre/01_perfil_municipios_ac.ipynb | grep '"outputs"'
# Se aparecer "outputs": [...dados...], limpar antes de commitar
```

Alternativamente, adicionar no `.gitignore` e commitar só os `.ipynb` sem output:

```bash
# Automaticamente limpar antes de commitar (usando nbstripout)
pip install nbstripout
nbstripout --install  # configura git hook automaticamente
```

---

## 13. Checklist antes de commitar um notebook

```
[ ] Rodei Kernel → Restart & Run All e todas as células passaram
[ ] Todas as asserções da célula de validação passaram (✅)
[ ] O arquivo em data/processed/ foi gerado com sucesso
[ ] Outputs foram limpos (Kernel → Restart & Clear Output)
[ ] O notebook abre sem erros quando eu seguir o README
```
