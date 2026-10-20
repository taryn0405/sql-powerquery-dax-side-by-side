# 04. Rolling 3-Month Total

**Question:** For each month, what is the total over that month and the two before it?

## SQL

```sql
WITH Months AS (
    SELECT DISTINCT MonthStart
    FROM dbo.DimDate
),
Monthly AS (
    SELECT
        DATEFROMPARTS(YEAR(InvoiceDate), MONTH(InvoiceDate), 1) AS MonthStart,
        SUM(Amount) AS Amount
    FROM dbo.FactInvoices
    GROUP BY DATEFROMPARTS(YEAR(InvoiceDate), MONTH(InvoiceDate), 1)
)
SELECT
    mo.MonthStart,
    COALESCE(mt.Amount, 0) AS Amount,
    SUM(COALESCE(mt.Amount, 0)) OVER (
        ORDER BY mo.MonthStart
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS Rolling3Months
FROM Months AS mo
LEFT JOIN Monthly AS mt
    ON mt.MonthStart = mo.MonthStart
ORDER BY mo.MonthStart;
```

Joining to the date table first guarantees one row per month, so "2 rows back" means "2 months back" even when a month has no invoices. Months after the last invoice (Oct to Dec 2026 here) show zeros, so filter them out if you do not want them.

## Power Query

```m
let
    WithMonth = Table.AddColumn(FactInvoices, "MonthStart", each Date.StartOfMonth([Date]), type date),
    Monthly = Table.Group(WithMonth, {"MonthStart"}, {{"Amount", each List.Sum([Amount]), type number}}),
    Sorted = Table.Buffer(Table.Sort(Monthly, {{"MonthStart", Order.Ascending}})),
    Amounts = List.Buffer(Sorted[Amount]),
    Rolling = List.Transform(
        List.Positions(Amounts),
        each
            let
                start = List.Max({0, _ - 2})
            in
                List.Sum(List.Range(Amounts, start, _ - start + 1))
    ),
    Result = Table.FromColumns(
        Table.ToColumns(Sorted) & {Rolling},
        Table.ColumnNames(Sorted) & {"Rolling3Months"}
    )
in
    Result
```

This version counts **positions**, not calendar months. It matches SQL only if every month has at least one invoice. With gaps, join to a month list first (as the SQL does), or use `fnRollingWindow` to filter rows to a window instead of producing a moving series.

## DAX

```dax
Rolling 3 Months =
CALCULATE (
    [Total Amount],
    DATESINPERIOD ( DimDate[Date], MAX ( DimDate[Date] ), -3, MONTH )
)
```

Use `DimDate[Month Label]` on the axis. `DATESINPERIOD` works on real calendar dates, so gaps are handled correctly.

## What to notice

- SQL and DAX define the window on **calendar months**, either through a joined date table or through date functions. The Power Query version defines it on **row positions**, which is simpler but silently wrong when months are missing.
- In the first three months of data, all three layers sum fewer than three months. That is expected and matches.
