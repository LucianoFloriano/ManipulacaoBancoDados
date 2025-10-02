---
title: "desafio10"
author: "Luciano Floriano"
date: "2025-10-02"
output:
  pdf_document: default
  html_document: default
---


```
## Using virtual environment "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu" ...
```

```
## + "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu/Scripts/python.exe" -m pip install --upgrade --no-user polars
```

```
## Using virtual environment "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu" ...
```

```
## + "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu/Scripts/python.exe" -m pip install --upgrade --no-user fastexcel
```

```
## Using virtual environment "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu" ...
```

```
## + "C:/Users/luflo/AppData/Local/R/cache/R/reticulate/uv/cache/archive-v0/qj_0-rP_hm7QfbZg_7cPu/Scripts/python.exe" -m pip install --upgrade --no-user pyarrow
```

Data de realização desse código:

``` python
import datetime

# Data e hora atual
agora = datetime.datetime.now()
print(f"Arquivo compilado em: {agora.strftime('%d/%m/%Y às %H:%M:%S')}")
```

```
## Arquivo compilado em: 02/10/2025 às 11:20:09
```

Importando os dados:

``` python
import polars as pl

# Lê um CSV, mas já traz só as colunas que interessam (código, cidade e estado)
aeroportos = pl.read_csv("airports.csv",
                         columns=["IATA_CODE", "CITY", "STATE"])

aeroportos.head(2) #mostra só as 2 primeiras linhas.
```

```
## shape: (2, 3)
## ┌───────────┬───────────┬───────┐
## │ IATA_CODE ┆ CITY      ┆ STATE │
## │ ---       ┆ ---       ┆ ---   │
## │ str       ┆ str       ┆ str   │
## ╞═══════════╪═══════════╪═══════╡
## │ ABE       ┆ Allentown ┆ PA    │
## │ ABI       ┆ Abilene   ┆ TX    │
## └───────────┴───────────┴───────┘
```


``` python
import polars as pl

wdi = pl.read_excel("WDIEXCEL.xlsx", sheet_name="Country", 
                    columns=["Short Name", "Region"]) #Puxa a aba "Country" e só pega as colunas “nome curto” e “região”.

wdi.head(2)
```

```
## shape: (2, 2)
## ┌─────────────┬───────────────────────────┐
## │ Short Name  ┆ Region                    │
## │ ---         ┆ ---                       │
## │ str         ┆ str                       │
## ╞═════════════╪═══════════════════════════╡
## │ Aruba       ┆ Latin America & Caribbean │
## │ Afghanistan ┆ South Asia                │
## └─────────────┴───────────────────────────┘
```
Um Exemplo Simples

``` python
df = pl.DataFrame({
    "grupo": ["A", "A", "B", "B", "C"],
    "valor1": [10, 15, 10, None, 25],
    "valor2": [5, None, 20, 30, None] #tabela com grupos e duas colunas de valores (alguns nulos).
})
df
```

```
## shape: (5, 3)
## ┌───────┬────────┬────────┐
## │ grupo ┆ valor1 ┆ valor2 │
## │ ---   ┆ ---    ┆ ---    │
## │ str   ┆ i64    ┆ i64    │
## ╞═══════╪════════╪════════╡
## │ A     ┆ 10     ┆ 5      │
## │ A     ┆ 15     ┆ null   │
## │ B     ┆ 10     ┆ 20     │
## │ B     ┆ null   ┆ 30     │
## │ C     ┆ 25     ┆ null   │
## └───────┴────────┴────────┘
```
Operando em valor1

``` python
df["valor1"]
```

```
## shape: (5,)
## Series: 'valor1' [i64]
## [
## 	10
## 	15
## 	10
## 	null
## 	25
## ]
```

``` python
print("Média do valor 1:",(df["valor1"].mean())) #Mostra a coluna valor1 e já calcula a média dela.
```

```
## Média do valor 1: 15.0
```
Operando em valor1


``` python
df.select([
  pl.col("valor1").mean().alias("media_v1"),
  pl.col("valor2").mean() #retorna um mini-DataFrame com as médias.
])
```

```
## shape: (1, 2)
## ┌──────────┬───────────┐
## │ media_v1 ┆ valor2    │
## │ ---      ┆ ---       │
## │ f64      ┆ f64       │
## ╞══════════╪═══════════╡
## │ 15.0     ┆ 18.333333 │
## └──────────┴───────────┘
```

Quais são as médias da variável valor1 e o valor mínimo da variável valor2 para cada um dos grupos definidos por grupo?

``` python
df.group_by("grupo").agg([
  pl.col("valor1").mean().alias("media_valor1"),
  pl.col("valor2").min().alias("min_valor2")
]).sort("grupo") #Calcula a média de valor1 e o mínimo de valor2 para cada grupo A, B, C. Depois organiza pelo nome do grupo.
```

```
## shape: (3, 3)
## ┌───────┬──────────────┬────────────┐
## │ grupo ┆ media_valor1 ┆ min_valor2 │
## │ ---   ┆ ---          ┆ ---        │
## │ str   ┆ f64          ┆ i64        │
## ╞═══════╪══════════════╪════════════╡
## │ A     ┆ 12.5         ┆ 5          │
## │ B     ┆ 10.0         ┆ 20         │
## │ C     ┆ 25.0         ┆ null       │
## └───────┴──────────────┴────────────┘
```

Calcule o percentual de vôos das cias. aéreas “AA” e “DL” que atrasaram pelo menos 30 minutos nas chegadas aos aeroportos “SEA”, “MIA” e “BWI”.

``` python
import polars as pl

# Correção: usar schema_overrides e ler arquivo ZIP corretamente
voos = pl.read_csv("flights.csv",
                   columns=["AIRLINE", "ARRIVAL_DELAY", "DESTINATION_AIRPORT"],
                   schema_overrides={"AIRLINE": pl.Utf8,
                                    "ARRIVAL_DELAY": pl.Int32,
                                    "DESTINATION_AIRPORT": pl.Utf8}) #Carrega só as colunas úteis (companhia, atraso e destino). Também define os tipos de dados certinhos.
voos.shape
```

```
## (5819079, 3)
```

``` python
voos.head(3) 
```

```
## shape: (3, 3)
## ┌─────────┬─────────────────────┬───────────────┐
## │ AIRLINE ┆ DESTINATION_AIRPORT ┆ ARRIVAL_DELAY │
## │ ---     ┆ ---                 ┆ ---           │
## │ str     ┆ str                 ┆ i32           │
## ╞═════════╪═════════════════════╪═══════════════╡
## │ AS      ┆ SEA                 ┆ -22           │
## │ AA      ┆ PBI                 ┆ -9            │
## │ US      ┆ CLT                 ┆ 5             │
## └─────────┴─────────────────────┴───────────────┘
```
Calcule o percentual de vôos das cias. aéreas “AA” e “DL” que atrasaram pelo menos 30 minutos nas chegadas aos aeroportos “SEA”, “MIA” e “BWI”.

``` python
resultado = (
  voos.drop_nulls(["AIRLINE", "DESTINATION_AIRPORT", "ARRIVAL_DELAY"])
  .filter(
    pl.col("AIRLINE").is_in(["AA", "DL"]) &
    pl.col("DESTINATION_AIRPORT").is_in(["SEA", "MIA", "BWI"])
    )
    .group_by(["AIRLINE", "DESTINATION_AIRPORT"])
    .agg([
      (pl.col("ARRIVAL_DELAY") > 30).mean().alias("atraso_medio")
      ])
) #Filtra só voos da AA e DL para os aeroportos pedidos, remove nulos e calcula o percentual que atrasou mais de 30 minutos.
resultado.sort("atraso_medio")
```

```
## shape: (6, 3)
## ┌─────────┬─────────────────────┬──────────────┐
## │ AIRLINE ┆ DESTINATION_AIRPORT ┆ atraso_medio │
## │ ---     ┆ ---                 ┆ ---          │
## │ str     ┆ str                 ┆ f64          │
## ╞═════════╪═════════════════════╪══════════════╡
## │ DL      ┆ BWI                 ┆ 0.069455     │
## │ DL      ┆ SEA                 ┆ 0.072967     │
## │ DL      ┆ MIA                 ┆ 0.090467     │
## │ AA      ┆ MIA                 ┆ 0.117894     │
## │ AA      ┆ SEA                 ┆ 0.124212     │
## │ AA      ┆ BWI                 ┆ 0.127523     │
## └─────────┴─────────────────────┴──────────────┘
```
## Segundo conjunto de exercícios

Dados Clientes


``` python
# Criando DataFrames
clientes = pl.DataFrame({
    "cliente_id": [1, 2, 3, 4],
    "nome": ["Ana", "Bruno", "Clara", "Daniel"]
})

print(clientes)
```

```
## shape: (4, 2)
## ┌────────────┬────────┐
## │ cliente_id ┆ nome   │
## │ ---        ┆ ---    │
## │ i64        ┆ str    │
## ╞════════════╪════════╡
## │ 1          ┆ Ana    │
## │ 2          ┆ Bruno  │
## │ 3          ┆ Clara  │
## │ 4          ┆ Daniel │
## └────────────┴────────┘
```
Dados Compras

``` python
pedidos = pl.DataFrame({
    "pedido_id": [101, 102, 103, 104, 105],
    "cliente_id": [1, 2, 3, 1, 5],
    "valor": [100.50, 250.75, 75.00, 130.00, 79.00]
})

print(pedidos)
```

```
## shape: (5, 3)
## ┌───────────┬────────────┬────────┐
## │ pedido_id ┆ cliente_id ┆ valor  │
## │ ---       ┆ ---        ┆ ---    │
## │ i64       ┆ i64        ┆ f64    │
## ╞═══════════╪════════════╪════════╡
## │ 101       ┆ 1          ┆ 100.5  │
## │ 102       ┆ 2          ┆ 250.75 │
## │ 103       ┆ 3          ┆ 75.0   │
## │ 104       ┆ 1          ┆ 130.0  │
## │ 105       ┆ 5          ┆ 79.0   │
## └───────────┴────────────┴────────┘
```
Exemplo INNER JOIN


``` python
res_ij = clientes.join(pedidos, on="cliente_id", how="inner") #Média dos valores comprados por cada cliente.
print(res_ij)
```

```
## shape: (4, 4)
## ┌────────────┬───────┬───────────┬────────┐
## │ cliente_id ┆ nome  ┆ pedido_id ┆ valor  │
## │ ---        ┆ ---   ┆ ---       ┆ ---    │
## │ i64        ┆ str   ┆ i64       ┆ f64    │
## ╞════════════╪═══════╪═══════════╪════════╡
## │ 1          ┆ Ana   ┆ 101       ┆ 100.5  │
## │ 2          ┆ Bruno ┆ 102       ┆ 250.75 │
## │ 3          ┆ Clara ┆ 103       ┆ 75.0   │
## │ 1          ┆ Ana   ┆ 104       ┆ 130.0  │
## └────────────┴───────┴───────────┴────────┘
```
Exemplo LEFT JOIN

``` python
res_lj = clientes.join(pedidos, on="cliente_id", how="left") 
print(res_lj)
```

```
## shape: (5, 4)
## ┌────────────┬────────┬───────────┬────────┐
## │ cliente_id ┆ nome   ┆ pedido_id ┆ valor  │
## │ ---        ┆ ---    ┆ ---       ┆ ---    │
## │ i64        ┆ str    ┆ i64       ┆ f64    │
## ╞════════════╪════════╪═══════════╪════════╡
## │ 1          ┆ Ana    ┆ 101       ┆ 100.5  │
## │ 1          ┆ Ana    ┆ 104       ┆ 130.0  │
## │ 2          ┆ Bruno  ┆ 102       ┆ 250.75 │
## │ 3          ┆ Clara  ┆ 103       ┆ 75.0   │
## │ 4          ┆ Daniel ┆ null      ┆ null   │
## └────────────┴────────┴───────────┴────────┘
```
Exemplo RIGHT JOIN

``` python
res_rj = clientes.join(pedidos, on="cliente_id", how="right") 
print(res_rj)
```

```
## shape: (5, 4)
## ┌───────┬───────────┬────────────┬────────┐
## │ nome  ┆ pedido_id ┆ cliente_id ┆ valor  │
## │ ---   ┆ ---       ┆ ---        ┆ ---    │
## │ str   ┆ i64       ┆ i64        ┆ f64    │
## ╞═══════╪═══════════╪════════════╪════════╡
## │ Ana   ┆ 101       ┆ 1          ┆ 100.5  │
## │ Bruno ┆ 102       ┆ 2          ┆ 250.75 │
## │ Clara ┆ 103       ┆ 3          ┆ 75.0   │
## │ Ana   ┆ 104       ┆ 1          ┆ 130.0  │
## │ null  ┆ 105       ┆ 5          ┆ 79.0   │
## └───────┴───────────┴────────────┴────────┘
```
Exemplo OUTER JOIN

``` python
res_oj = clientes.join(pedidos, on="cliente_id", how="full")
print(res_oj)
```

```
## shape: (6, 5)
## ┌────────────┬────────┬───────────┬──────────────────┬────────┐
## │ cliente_id ┆ nome   ┆ pedido_id ┆ cliente_id_right ┆ valor  │
## │ ---        ┆ ---    ┆ ---       ┆ ---              ┆ ---    │
## │ i64        ┆ str    ┆ i64       ┆ i64              ┆ f64    │
## ╞════════════╪════════╪═══════════╪══════════════════╪════════╡
## │ 1          ┆ Ana    ┆ 101       ┆ 1                ┆ 100.5  │
## │ 2          ┆ Bruno  ┆ 102       ┆ 2                ┆ 250.75 │
## │ 3          ┆ Clara  ┆ 103       ┆ 3                ┆ 75.0   │
## │ 1          ┆ Ana    ┆ 104       ┆ 1                ┆ 130.0  │
## │ null       ┆ null   ┆ 105       ┆ 5                ┆ 79.0   │
## │ 4          ┆ Daniel ┆ null      ┆ null             ┆ null   │
## └────────────┴────────┴───────────┴──────────────────┴────────┘
```
Exemplo CROSS JOIN

``` python
res_cj = clientes.join(pedidos, how="cross")
print(res_cj)
```

```
## shape: (20, 5)
## ┌────────────┬────────┬───────────┬──────────────────┬────────┐
## │ cliente_id ┆ nome   ┆ pedido_id ┆ cliente_id_right ┆ valor  │
## │ ---        ┆ ---    ┆ ---       ┆ ---              ┆ ---    │
## │ i64        ┆ str    ┆ i64       ┆ i64              ┆ f64    │
## ╞════════════╪════════╪═══════════╪══════════════════╪════════╡
## │ 1          ┆ Ana    ┆ 101       ┆ 1                ┆ 100.5  │
## │ 1          ┆ Ana    ┆ 102       ┆ 2                ┆ 250.75 │
## │ 1          ┆ Ana    ┆ 103       ┆ 3                ┆ 75.0   │
## │ 1          ┆ Ana    ┆ 104       ┆ 1                ┆ 130.0  │
## │ 1          ┆ Ana    ┆ 105       ┆ 5                ┆ 79.0   │
## │ …          ┆ …      ┆ …         ┆ …                ┆ …      │
## │ 4          ┆ Daniel ┆ 101       ┆ 1                ┆ 100.5  │
## │ 4          ┆ Daniel ┆ 102       ┆ 2                ┆ 250.75 │
## │ 4          ┆ Daniel ┆ 103       ┆ 3                ┆ 75.0   │
## │ 4          ┆ Daniel ┆ 104       ┆ 1                ┆ 130.0  │
## │ 4          ┆ Daniel ┆ 105       ┆ 5                ┆ 79.0   │
## └────────────┴────────┴───────────┴──────────────────┴────────┘
```
P1: Qual é o valor médio das compras realizadas para cada cliente identificado?
Como responder P1?

``` python
print(clientes)
```

```
## shape: (4, 2)
## ┌────────────┬────────┐
## │ cliente_id ┆ nome   │
## │ ---        ┆ ---    │
## │ i64        ┆ str    │
## ╞════════════╪════════╡
## │ 1          ┆ Ana    │
## │ 2          ┆ Bruno  │
## │ 3          ┆ Clara  │
## │ 4          ┆ Daniel │
## └────────────┴────────┘
```

``` python
print(pedidos)
```

```
## shape: (5, 3)
## ┌───────────┬────────────┬────────┐
## │ pedido_id ┆ cliente_id ┆ valor  │
## │ ---       ┆ ---        ┆ ---    │
## │ i64       ┆ i64        ┆ f64    │
## ╞═══════════╪════════════╪════════╡
## │ 101       ┆ 1          ┆ 100.5  │
## │ 102       ┆ 2          ┆ 250.75 │
## │ 103       ┆ 3          ┆ 75.0   │
## │ 104       ┆ 1          ┆ 130.0  │
## │ 105       ┆ 5          ┆ 79.0   │
## └───────────┴────────────┴────────┘
```
Resposta:

``` python
res = res_ij.group_by(["nome", "cliente_id"]).agg(pl.col("valor").mean())
print(res)
```

```
## shape: (3, 3)
## ┌───────┬────────────┬────────┐
## │ nome  ┆ cliente_id ┆ valor  │
## │ ---   ┆ ---        ┆ ---    │
## │ str   ┆ i64        ┆ f64    │
## ╞═══════╪════════════╪════════╡
## │ Ana   ┆ 1          ┆ 115.25 │
## │ Bruno ┆ 2          ┆ 250.75 │
## │ Clara ┆ 3          ┆ 75.0   │
## └───────┴────────────┴────────┘
```

P2: Informe os nomes e a quantidade de compras com valor mínimo de $100.00 realizadas por cada cliente.
Como responder P2?


``` python
print(clientes)
```

```
## shape: (4, 2)
## ┌────────────┬────────┐
## │ cliente_id ┆ nome   │
## │ ---        ┆ ---    │
## │ i64        ┆ str    │
## ╞════════════╪════════╡
## │ 1          ┆ Ana    │
## │ 2          ┆ Bruno  │
## │ 3          ┆ Clara  │
## │ 4          ┆ Daniel │
## └────────────┴────────┘
```

``` python
print(pedidos)
```

```
## shape: (5, 3)
## ┌───────────┬────────────┬────────┐
## │ pedido_id ┆ cliente_id ┆ valor  │
## │ ---       ┆ ---        ┆ ---    │
## │ i64       ┆ i64        ┆ f64    │
## ╞═══════════╪════════════╪════════╡
## │ 101       ┆ 1          ┆ 100.5  │
## │ 102       ┆ 2          ┆ 250.75 │
## │ 103       ┆ 3          ┆ 75.0   │
## │ 104       ┆ 1          ┆ 130.0  │
## │ 105       ┆ 5          ┆ 79.0   │
## └───────────┴────────────┴────────┘
```
Reposta:

``` python
res = (res_oj.with_columns(pl.col("valor") > 100)
       .group_by("nome")
       .agg(pl.col("valor").sum())) ##transforma os valores maiores que 100 e depois conta por cliente
print(res)
```

```
## shape: (5, 2)
## ┌────────┬───────┐
## │ nome   ┆ valor │
## │ ---    ┆ ---   │
## │ str    ┆ u32   │
## ╞════════╪═══════╡
## │ null   ┆ 0     │
## │ Bruno  ┆ 1     │
## │ Ana    ┆ 2     │
## │ Daniel ┆ 0     │
## │ Clara  ┆ 0     │
## └────────┴───────┘
```
JOIN com Múltiplas Colunas como Chave

``` python
vendas = pl.DataFrame({
    "id_venda": [1, 2, 3],
    "id_cl": [1, 2, 1],
    "id_prod": [101, 102, 103],
    "qtde": [2, 1, 1]
})

detalhes_pedidos = pl.DataFrame({
    "id_ped": [201, 202, 203],
    "cl_id": [1, 2, 1],
    "id_prod": [101, 102, 104],
    "valor": [50.00, 75.00, 100.00]
})

print(vendas)
```

```
## shape: (3, 4)
## ┌──────────┬───────┬─────────┬──────┐
## │ id_venda ┆ id_cl ┆ id_prod ┆ qtde │
## │ ---      ┆ ---   ┆ ---     ┆ ---  │
## │ i64      ┆ i64   ┆ i64     ┆ i64  │
## ╞══════════╪═══════╪═════════╪══════╡
## │ 1        ┆ 1     ┆ 101     ┆ 2    │
## │ 2        ┆ 2     ┆ 102     ┆ 1    │
## │ 3        ┆ 1     ┆ 103     ┆ 1    │
## └──────────┴───────┴─────────┴──────┘
```

``` python
print(detalhes_pedidos)
```

```
## shape: (3, 4)
## ┌────────┬───────┬─────────┬───────┐
## │ id_ped ┆ cl_id ┆ id_prod ┆ valor │
## │ ---    ┆ ---   ┆ ---     ┆ ---   │
## │ i64    ┆ i64   ┆ i64     ┆ f64   │
## ╞════════╪═══════╪═════════╪═══════╡
## │ 201    ┆ 1     ┆ 101     ┆ 50.0  │
## │ 202    ┆ 2     ┆ 102     ┆ 75.0  │
## │ 203    ┆ 1     ┆ 104     ┆ 100.0 │
## └────────┴───────┴─────────┴───────┘
```
Realizando um JOIN com Múltiplas Colunas

``` python
final = vendas.join(detalhes_pedidos,
                    left_on = ["id_cl", "id_prod"],
                    right_on = ["cl_id", "id_prod"],
                    how = "inner") #Faz o join usando duas chaves ao mesmo tempo (id_cl + id_prod). Assim só bate quem tiver cliente e produto iguais nos dois DataFrames.
print(final)
```

```
## shape: (2, 6)
## ┌──────────┬───────┬─────────┬──────┬────────┬───────┐
## │ id_venda ┆ id_cl ┆ id_prod ┆ qtde ┆ id_ped ┆ valor │
## │ ---      ┆ ---   ┆ ---     ┆ ---  ┆ ---    ┆ ---   │
## │ i64      ┆ i64   ┆ i64     ┆ i64  ┆ i64    ┆ f64   │
## ╞══════════╪═══════╪═════════╪══════╪════════╪═══════╡
## │ 1        ┆ 1     ┆ 101     ┆ 2    ┆ 201    ┆ 50.0  │
## │ 2        ┆ 2     ┆ 102     ┆ 1    ┆ 202    ┆ 75.0  │
## └──────────┴───────┴─────────┴──────┴────────┴───────┘
```
