# 📐 Medidas DAX — Painel Iplacas e Shock Visual

Este documento reúne as principais medidas DAX desenvolvidas no painel, organizadas por categoria de análise. Todas foram pensadas para serem **performáticas, reutilizáveis e contextualmente robustas**.

---

## 📑 Sumário

- #1--metas-e-acompanhamento
- #2--ticket-médio-e-acumulado
- #3--operacional-tempo-de-entrega
- #4--análise-de-clientes-novos-recorrentes-e-ativos
- #5--cac-custo-de-aquisição-de-cliente
- #6--kpi-card-dinâmico

---

## 1️⃣ Metas e Acompanhamento

### `Variação Meta Dia`
Calcula a variação percentual da meta diária em relação ao dia anterior, permitindo identificar saltos ou quedas no objetivo diário.

```dax
Variação Meta Dia =
VAR DiaAtual =
    MAX('dCalendario'[Date])

VAR Atual =
    CALCULATE(
        [Meta Dia],
        'dCalendario'[Date] = DiaAtual
    )

VAR Anterior =
    CALCULATE(
        [Meta Dia],
        'dCalendario'[Date] = DiaAtual - 1
    )

RETURN
    IF(
        NOT ISBLANK(Anterior),
        DIVIDE(Atual - Anterior, Anterior),
        BLANK()
    )
```

🧠 **Lógica:** armazena o dia atual em variável, calcula a meta do dia e do anterior via `CALCULATE` com filtro de data, e retorna a variação percentual usando `DIVIDE` (que já trata divisão por zero).

---

## 2️⃣ Ticket Médio e Acumulado

### `Ticket Médio por Cliente Mês Anterior`
Calcula o ticket médio (faturamento ÷ clientes únicos) considerando exclusivamente o mês imediatamente anterior ao último filtrado.

```dax
Ticket Médio por Cliente Mês Anterior =
VAR UltimaData =
    MAXX(
        ALLSELECTED('dCalendario'),
        'dCalendario'[Date]
    )

VAR DataMesAnterior =
    EOMONTH(UltimaData, -1)

VAR InicioMesAnterior =
    DATE(YEAR(DataMesAnterior), MONTH(DataMesAnterior), 1)

VAR FimMesAnterior =
    EOMONTH(DataMesAnterior, 0)

RETURN
DIVIDE(
    CALCULATE(
        [Acumulado],
        DATESBETWEEN(
            'dCalendario'[Date],
            InicioMesAnterior,
            FimMesAnterior
        )
    ),
    CALCULATE(
        DISTINCTCOUNT('Base Consolidada_Vendas'[Comprador/Empresa]),
        DATESBETWEEN(
            'dCalendario'[Date],
            InicioMesAnterior,
            FimMesAnterior
        )
    )
)
```

🧠 **Lógica:** usa `EOMONTH` para encontrar o último dia do mês anterior, monta o intervalo com `DATESBETWEEN` e divide faturamento por contagem distinta de clientes nesse período. Essencial para o card de comparativo mensal.

---

## 3️⃣ Operacional — Tempo de Entrega

### `Tempo Médio de Entrega`
Calcula a média de dias entre a aprovação e a entrega de cada pedido, ignorando registros incompletos.

```dax
Tempo Médio de Entrega =
AVERAGEX(
    FILTER(
        'Base Consolidada_Vendas',
        NOT(ISBLANK('Base Consolidada_Vendas'[Entrega])) &&
        NOT(ISBLANK('Base Consolidada_Vendas'[Aprovação]))
    ),
    VALUE('Base Consolidada_Vendas'[Entrega] - 'Base Consolidada_Vendas'[Aprovação])
)
```

🧠 **Lógica:** `AVERAGEX` itera linha a linha sobre as vendas com datas válidas e calcula a diferença Entrega − Aprovação. KPI fundamental para acompanhar SLA operacional.

---

## 4️⃣ Análise de Clientes — Novos, Recorrentes e Ativos

### `Status do Cliente`
Classifica cada cliente como **"Novo"** ou **"Recorrente"** com base em seu histórico de compras acumulado.

```dax
Status do Cliente =
VAR ClienteAtual = SELECTEDVALUE('Base Consolidada_Vendas'[Comprador/Empresa])
VAR DataMaxFiltro = MAX('dCalendario'[Date])

VAR HistoricoCompras =
    CALCULATE(
        [Quantidade de Vendas],
        'dCalendario'[Date] <= DataMaxFiltro,
        REMOVEFILTERS('dCalendario'),
        'Base Consolidada_Vendas'[Comprador/Empresa] = ClienteAtual
    )

RETURN
IF(
    [Quantidade de Vendas] >= 1,
    IF(HistoricoCompras = 1, "Novo", "Recorrente"),
    BLANK()
)
```

🧠 **Lógica:** `SELECTEDVALUE` pega o cliente em foco, `REMOVEFILTERS('dCalendario')` libera o histórico completo até a data filtrada, e o `IF` classifica conforme a quantidade total de compras.

---

### `Novos Clientes — Visão CAC`
Conta a quantidade de clientes **novos** (1ª compra) considerando apenas os canais relevantes para análise de CAC.

```dax
Novos Clientes_Visao_Cac =
CALCULATE(
    DISTINCTCOUNT('Base Consolidada_Vendas'[Comprador/Empresa]),

    KEEPFILTERS(
        'Base Consolidada_Vendas'[Canal] IN
        {"Shock", "Iplacas", "ML Iplacas", "ML Shock", "Shopee"}
    ),

    FILTER(
        VALUES('Base Consolidada_Vendas'[Comprador/Empresa]),
        VAR HistoricoCompras =
            CALCULATE(
                [Quantidade de Vendas],
                'dCalendario'[Date] <= MAX('dCalendario'[Date]),
                REMOVEFILTERS('dCalendario')
            )
        RETURN
            [Quantidade de Vendas] >= 1
            && HistoricoCompras = 1
    )
)
```

🧠 **Lógica:** `KEEPFILTERS` preserva o filtro de canais relevantes mesmo após `CALCULATE`. O `FILTER` percorre cada cliente e mantém apenas aqueles cujo histórico acumulado é igual a 1 (primeira compra).

---

### `Clientes Recorrentes`
Conta clientes que compraram **mais de uma vez** no período acumulado.

```dax
Clientes Recorrentes =
VAR DataMaxFiltro = MAX('Base Consolidada_Vendas'[Data da Venda])
RETURN
CALCULATE(
    DISTINCTCOUNT('Base Consolidada_Vendas'[Comprador/Empresa]),
    FILTER(
        VALUES('Base Consolidada_Vendas'[Comprador/Empresa]),
        VAR HistoricoCompras =
            CALCULATE(
                [Quantidade de Vendas],
                'Base Consolidada_Vendas'[Data da Venda] <= DataMaxFiltro,
                REMOVEFILTERS('dCalendario')
            )
        RETURN
        [Quantidade de Vendas] >= 1 && HistoricoCompras > 1
    )
)
```

🧠 **Lógica:** mesma estrutura do "Novos Clientes", mas mantém apenas quem tem histórico **maior que 1**. Indicador-chave para fidelização e LTV.

---

### `Clientes Ativos Atual`
Conta clientes únicos que realizaram compras dentro do **mês atual filtrado**.

```dax
Clientes Ativos Atual =
VAR UltimaData =
    MAXX(ALLSELECTED('Base Consolidada_Vendas'), 'Base Consolidada_Vendas'[Data da Venda])
RETURN
    CALCULATE(
        DISTINCTCOUNT('Base Consolidada_Vendas'[Comprador/Empresa]),
        DATESBETWEEN(
            'dCalendario'[Date],
            DATE(YEAR(UltimaData), MONTH(UltimaData), 1),
            EOMONTH(UltimaData, 0)
        )
    )
```

🧠 **Lógica:** identifica a última data com vendas (mesmo após filtros do usuário) e calcula o intervalo do mês corrente com `DATESBETWEEN`. Garante que o card sempre mostre o mês ativo correto, independente da seleção.

---

## 5️⃣ CAC — Custo de Aquisição de Cliente

> **Observação:** as medidas de CAC dependem das tabelas `CAC` (investimento) e `Novos Clientes_Visao_Cac` (denominador). A fórmula final é:
>
> ```dax
> CAC = DIVIDE(SUM('CAC'[Investimento]), [Novos Clientes_Visao_Cac])
> ```
>
> O **`Novos Clientes_Visao_Cac`** é a peça central, pois isola somente novos clientes dos canais investidos.

---

## 6️⃣ KPI Card Dinâmico

### `Acumulado KPI`
Medida inteligente que ajusta seu comportamento conforme exista ou não filtro de período aplicado. **Quando o usuário não filtra um período**, automaticamente exibe o último mês com dados.

```dax
Acumulado KPI =
VAR TemFiltroPeriodo =
    ISFILTERED('dCalendario'[Mês])
        || ISFILTERED('dCalendario'[AnoMes])
        || ISFILTERED('dCalendario'[Date])

VAR UltimoAnoMesComDados =
    MAXX(
        FILTER(
            ALL('dCalendario'[AnoMes]),
            NOT ISBLANK(
                CALCULATE([Acumulado])
            )
        ),
        'dCalendario'[AnoMes]
    )

RETURN
IF(
    TemFiltroPeriodo,
    [Acumulado],
    CALCULATE(
        [Acumulado],
        FILTER(
            ALL('dCalendario'),
            'dCalendario'[AnoMes] = UltimoAnoMesComDados
        )
    )
)
```

🧠 **Lógica:** `ISFILTERED` detecta se há filtro de período ativo. Se houver, retorna o acumulado direto; caso contrário, busca o último `AnoMes` que tem vendas registradas e retorna esse acumulado. Essencial para que os cards do dashboard **nunca apareçam vazios** ao carregar a página inicial.

---

## 🧠 Padrões Técnicos Aplicados

Diversas medidas seguem padrões DAX consolidados:

- **`VAR` + `RETURN`** — melhora performance e legibilidade ao evitar recálculos.
- **`DIVIDE`** — tratamento nativo de divisão por zero (retorna `BLANK()` em vez de erro).
- **`REMOVEFILTERS` + `CALCULATE`** — controle explícito do contexto de filtro para análises de histórico.
- **`KEEPFILTERS`** — preserva filtros externos quando se aplica filtro adicional dentro de `CALCULATE`.
- **`MAXX(ALLSELECTED(...))`** — pega a última data respeitando o que o usuário filtrou (mas ignorando o contexto de linha).
- **`DATESBETWEEN` + `EOMONTH`** — base para todas as análises de Time Intelligence customizadas.
- [ ] Implementar **medidas de previsão** (forecasting de meta);
- [ ] Adicionar análises de **LTV (Lifetime Value)** combinando recorrência e ticket médio;
- [ ] Desenvolver **medidas de drill-through** para análise de motivo de retrabalho por canal.
