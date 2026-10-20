# 02. Running Total by Group

**Question:** What is the cumulative amount over time for each program?

## SQL

```sql
SELECT
    Program,
    InvoiceDate,
    InvoiceID,
    Amount,
    SUM(Amount) OVER (
        PARTITION BY Program
        ORDER BY InvoiceDate, InvoiceID
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS RunningTotal
FROM dbo.FactInvoices
ORDER BY Program, InvoiceDate, InvoiceID;
```

Use `ROWS` rather than the default `RANGE`, and add a tiebreaker (`InvoiceID`) so rows on the same date get a stable order.

## Power Query

Using `fnRunningTotal` from the Power Query Function Library:

```m
let
    Source = FactInvoices,
    Result = fnRunningTotal(Source, "Program", "Date", "Amount", "RunningTotal")
in
    Result
```

The function sorts by group and date, buffers the table, and builds each total in a single pass with `List.Generate`. Invoices on the same date have no guaranteed order between them, so the in-between totals on a tied date can differ from SQL, while the last total of each program matches.

## DAX

```dax
Running Total =
CALCULATE (
    [Total Amount],
    FILTER (
        ALLSELECTED ( DimDate[Date] ),
        DimDate[Date] <= MAX ( DimDate[Date] )
    )
)
```

Put `DimDate[Date]` (or month) on the axis and `Program` as the legend or a slicer.

## What to notice

- SQL and Power Query compute the total **per invoice row**. DAX computes it **per date shown in the visual**, so it will not match row-for-row unless the visual is at invoice grain.
- SQL window functions and `List.Generate` are single-pass and fast. A DAX running total recalculates for every cell, which is fine for a chart but slow for very large grids.
- Choose by where the result is needed. Stored in a table or exported: SQL or Power Query. Responding to slicers: DAX.
