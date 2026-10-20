# 05. Year-over-Year Comparison

**Question:** How does each month compare to the same month last year?

## SQL

```sql
WITH Monthly AS (
    SELECT
        DATEFROMPARTS(YEAR(InvoiceDate), MONTH(InvoiceDate), 1) AS MonthStart,
        SUM(Amount) AS Amount
    FROM dbo.FactInvoices
    GROUP BY DATEFROMPARTS(YEAR(InvoiceDate), MONTH(InvoiceDate), 1)
)
SELECT
    cur.MonthStart,
    cur.Amount,
    prior.Amount AS AmountPY,
    (cur.Amount - prior.Amount) / NULLIF(prior.Amount, 0) AS YoYPct
FROM Monthly AS cur
LEFT JOIN Monthly AS prior
    ON prior.MonthStart = DATEADD(YEAR, -1, cur.MonthStart)
ORDER BY cur.MonthStart;
```

`NULLIF(prior.Amount, 0)` turns a zero into a null, so the division returns null instead of an error.

## Power Query

```m
let
    WithMonth = Table.AddColumn(FactInvoices, "MonthStart", each Date.StartOfMonth([Date]), type date),
    Monthly = Table.Group(WithMonth, {"MonthStart"}, {{"Amount", each List.Sum([Amount]), type number}}),
    WithPriorKey = Table.AddColumn(Monthly, "PriorMonthStart", each Date.AddYears([MonthStart], -1), type date),
    Joined = Table.NestedJoin(WithPriorKey, {"PriorMonthStart"}, Monthly, {"MonthStart"}, "PY", JoinKind.LeftOuter),
    AddPY = Table.AddColumn(Joined, "AmountPY", each List.Sum([PY][Amount]), type nullable number),
    AddYoY = Table.AddColumn(
        AddPY,
        "YoYPct",
        each if [AmountPY] = null or [AmountPY] = 0 then null else ([Amount] - [AmountPY]) / [AmountPY],
        type nullable number
    ),
    Result = Table.RemoveColumns(AddYoY, {"PY", "PriorMonthStart"})
in
    Result
```

## DAX

```dax
Amount PY =
CALCULATE ( [Total Amount], SAMEPERIODLASTYEAR ( DimDate[Date] ) )

Amount YoY % =
DIVIDE ( [Total Amount] - [Amount PY], [Amount PY] )
```

## What to notice

- All three layers return blank (or null) for the first 12 months, since there is no prior year in the data. That is correct, not a bug.
- Each handles divide-by-zero its own way: `NULLIF` in SQL, an explicit `if` in Power Query, and `DIVIDE` in DAX. `DIVIDE` is the shortest, which is one reason ratios belong in measures.
- The SQL and Power Query versions match on a month key, so they work whether or not the data has gaps. Avoid `LAG(..., 12)` for this, since it counts rows and not months.
