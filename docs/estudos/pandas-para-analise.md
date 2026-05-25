# Guia: pandas para Análise de Dados

> **O que você vai aprender:** o que é um DataFrame, como ler dados, filtrar, agrupar,
> cruzar tabelas e calcular métricas — tudo com exemplos do nosso projeto.

---

## 1. O que é o pandas?

pandas é a biblioteca Python para trabalhar com dados tabulares — pense nele como
um **Excel programável**: você tem linhas, colunas, pode filtrar, calcular, cruzar tabelas
e visualizar, tudo em código.

A unidade central é o **DataFrame** — uma tabela com linhas e colunas:

```python
import pandas as pd

# Criar um DataFrame manualmente (raramente você faz isso — geralmente vem do BD+)
df = pd.DataFrame({
    "municipio": ["Rio Branco", "Cruzeiro do Sul", "Sena Madureira"],
    "populacao": [413418, 90402, 47754],
    "idhm":      [0.727,   0.682,         0.651]
})

print(df)
#         municipio  populacao   idhm
# 0      Rio Branco     413418  0.727
# 1  Cruzeiro do Sul    90402  0.682
# 2  Sena Madureira    47754  0.651
```

---

## 2. Operações fundamentais

### Ver o que tem no DataFrame

```python
df.head(5)        # primeiras 5 linhas
df.tail(5)        # últimas 5 linhas
df.shape          # (número de linhas, número de colunas)
df.dtypes         # tipo de cada coluna (int, float, object, datetime)
df.describe()     # estatísticas: média, min, max, percentis
df.info()         # resumo geral: linhas, colunas, memória usada
df.isnull().sum() # quantos valores nulos por coluna
```

### Selecionar colunas

```python
df["municipio"]                          # uma coluna → Series
df[["municipio", "idhm"]]               # várias colunas → DataFrame
```

### Filtrar linhas

```python
# Municípios com IDH acima de 0.7
df[df["idhm"] > 0.7]

# Múltiplas condições: & (e), | (ou)
df[(df["idhm"] > 0.6) & (df["populacao"] > 50000)]

# Igual a um valor
df[df["municipio"] == "Rio Branco"]

# Está numa lista
df[df["municipio"].isin(["Rio Branco", "Cruzeiro do Sul"])]
```

### Ordenar

```python
df.sort_values("idhm", ascending=False)  # maior IDH primeiro
df.sort_values(["uf", "populacao"])       # múltiplas colunas
```

---

## 3. Criar novas colunas (cálculos)

É aqui que começa a análise de verdade:

```python
# % da população em Bolsa Família
df["perc_bolsa_familia"] = (df["familias_bf"] * 3.5 / df["populacao"] * 100).round(1)

# Razão salário prefeito vs. média local
df["razao_salario"] = (df["salario_prefeito"] / df["salario_medio_local"]).round(1)

# Semáforo da Lei de Responsabilidade Fiscal
df["status_lrf"] = df["perc_gasto_pessoal"].apply(
    lambda x: "✅ Saudável" if x <= 45 else "⚠️ Atenção" if x <= 54 else "🔴 Irregular"
)
```

**`apply()` com `lambda`** — para cada linha, aplica uma função:

```python
# lambda x: ... significa "para cada valor x na coluna, faça..."
df["categoria_populacao"] = df["populacao"].apply(
    lambda p: "Grande" if p > 100000 else "Médio" if p > 20000 else "Pequeno"
)
```

---

## 4. Agrupar dados — groupby()

`groupby()` é o equivalente ao `GROUP BY` do SQL — agrupa linhas e calcula algo por grupo:

```python
# Quantos mandatos por partido?
df.groupby("sigla_partido").size().reset_index(name="mandatos")

# Média de IDH por estado
df.groupby("sigla_uf")["idhm"].mean().reset_index()

# Múltiplas operações ao mesmo tempo (agg)
df.groupby("sigla_uf").agg(
    municipios=("id_municipio", "count"),
    populacao_total=("populacao", "sum"),
    idh_medio=("idhm", "mean")
).reset_index()
```

> **Por que `.reset_index()`?** O `groupby` retorna o grupo como índice. O `reset_index()`
> transforma de volta em coluna normal — facilita usar depois.

---

## 5. Cruzar tabelas — merge() (equivalente ao JOIN do SQL)

O `merge()` é o `JOIN` do pandas. A chave do nosso projeto é sempre `id_municipio`:

```python
# INNER JOIN (só linhas que existem nos dois lados)
df_resultado = df_eleicoes.merge(df_populacao, on="id_municipio")

# LEFT JOIN (mantém todos da esquerda, NULL onde não tem na direita)
df_resultado = df_eleicoes.merge(df_populacao, on="id_municipio", how="left")

# JOIN por duas colunas ao mesmo tempo
df_resultado = df_siconfi.merge(
    df_mandatos,
    on=["id_municipio", "ano"],  # combinar município E ano
    how="left"
)

# Renomear coluna ambígua após merge (colunas com mesmo nome ganham _x e _y)
df_resultado = df_resultado.rename(columns={"nome_x": "nome_municipio", "nome_y": "nome_candidato"})
```

### Exemplo real do Notebook 02:

```python
# 1. Eleições do Acre
df_eleicoes = bd.read_sql("SELECT ... FROM br_tse_eleicoes ... WHERE sigla_uf='AC'", ...)

# 2. Perfil do eleitorado
df_eleitorado = bd.read_sql("SELECT ... FROM br_tse_eleicoes.perfil_eleitorado ... WHERE sigla_uf='AC'", ...)

# 3. Cruzar: para cada eleição, adicionar quantos eleitores cadastrados tinha
df = df_eleicoes.merge(
    df_eleitorado[["id_municipio", "ano", "qtd_eleitores_perfil"]],
    left_on=["id_municipio", "ano_eleicao"],  # coluna na tabela da esquerda
    right_on=["id_municipio", "ano"],          # coluna correspondente na direita
    how="left"
)

# 4. Calcular % de comparecimento
df["perc_comparecimento"] = (df["qtd_comparecimento"] / df["qtd_eleitores_perfil"] * 100).round(1)
```

---

## 6. Explorar com value_counts() e corr()

```python
# Quantas ocorrências de cada valor?
df["sigla_partido"].value_counts()
# PT     12
# MDB     8
# PSDB    5

# Correlação entre colunas numéricas (-1 a +1)
# 1.0 = correlação perfeita positiva
# -1.0 = correlação perfeita negativa
# ~0 = sem correlação
df[["perc_bolsa_familia", "perc_emprego_formal", "idhm"]].corr()
```

---

## 7. Salvar e carregar dados localmente

```python
# Salvar (sempre em data/processed/ para dados limpos)
df.to_json("data/processed/eleicoes_ac.json", orient="records", force_ascii=False)
df.to_csv("data/processed/eleicoes_ac.csv", index=False)
df.to_parquet("data/processed/eleicoes_ac.parquet", index=False)

# Carregar
df = pd.read_json("data/processed/eleicoes_ac.json")
df = pd.read_csv("data/processed/eleicoes_ac.csv")
df = pd.read_parquet("data/processed/eleicoes_ac.parquet")
```

> **Por que Parquet?** É um formato colunar — muito mais rápido para leitura e ocupa 5–10x
> menos espaço que CSV. Ideal para o dataset final que o Spark vai ler na Fase 3.

---

## 8. Padrão de análise que usamos em cada notebook

Todo notebook segue esta sequência:

```python
# 1. COLETAR — buscar do BD+ ou carregar do cache
df_raw = bd.read_sql("...", billing_project_id=PROJECT_ID)
df_raw.to_json("data/raw/nome_dado.json", orient="records")

# 2. INSPECIONAR — entender o que chegou
print(df_raw.shape)
print(df_raw.dtypes)
print(df_raw.isnull().sum())
df_raw.head()

# 3. LIMPAR — tratar nulos, tipos errados, duplicatas
df = df_raw.dropna(subset=["id_municipio"])  # remover linhas sem ID
df["ano"] = df["ano"].astype(int)             # garantir tipo correto

# 4. CALCULAR — criar métricas de negócio
df["perc_bolsa_familia"] = (df["familias_bf"] * 3.5 / df["populacao"] * 100).round(1)

# 5. ANALISAR — agrupar, ordenar, correlacionar
ranking = df.groupby("nome")["perc_bolsa_familia"].mean().sort_values(ascending=False)

# 6. VISUALIZAR — gráfico com plotly
fig = px.bar(ranking.reset_index(), x="nome", y="perc_bolsa_familia", ...)
fig.show()

# 7. VALIDAR — asserções antes de salvar
assert df.shape[0] == 22, "Devem ter 22 municípios"
assert df["perc_bolsa_familia"].between(0, 100).all()

# 8. SALVAR — para o próximo notebook usar
df.to_json("data/processed/social_ac.json", orient="records")
```

---

## 9. Erros comuns e o que significam

| Erro | O que significa | Como resolver |
|------|----------------|--------------|
| `KeyError: 'nome_coluna'` | Coluna não existe | Verificar `df.columns` ou `df.dtypes` |
| `ValueError: cannot convert float NaN` | Tem nulos onde não deveria | `df.dropna()` ou `df.fillna(0)` |
| `MergeError: No common columns` | JOIN sem coluna em comum | Especificar `on=`, `left_on=`, `right_on=` |
| `SettingWithCopyWarning` | Editando uma cópia sem querer | Usar `df = df.copy()` antes de editar |
| DataFrame vazio após merge | Chaves não batem | Verificar tipo das chaves: `df.dtypes` |
