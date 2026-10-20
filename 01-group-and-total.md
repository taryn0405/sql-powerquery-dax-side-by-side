# 01. Group and Total

**Question:** What is the total amount and invoice count for each program and vendor?

## SQL

```sql
SELECT
    Program,
    Vendor,
    SUM(Amount) AS TotalAmount,
    COUNT(*)    AS InvoiceCount
FROM dbo.FactInvoices
GROUP BY Program, Vendor
ORDER BY Program, Vendor;
```

## Power Query

```m
let
    Source = FactInvoices,
    Grouped = Table.Group(
        Source,
        {"Program", "Vendor"},
        {
            {"TotalAmount", each List.Sum([Amount]), type number},
            {"InvoiceCount", each Table.RowCount(_), Int64.Type}
        }
    ),
    Sorted = Table.Sort(Grouped, {{"Program", Order.Ascending}, {"Vendor", Order.Ascending}})
in
    Sorted
```

## DAX

```dax
Total Amount = SUM ( FactInvoices[Amount] )

Invoice Count = COUNTROWS ( FactInvoices )
```

Put `Program` on rows and `Vendor` on columns (or rows) in a matrix. For a stored table instead of a measure:

```dax
Program Vendor Totals =
SUMMARIZECOLUMNS (
    FactInvoices[Program],
    FactInvoices[Vendor],
    "TotalAmount", [Total Amount],
    "InvoiceCount", [Invoice Count]
)
```

## What to notice

- SQL and Power Query produce a **fixed-grain table**: one row per program and vendor, computed when the query runs.
- DAX defines a **measure** that recalculates for whatever grain the visual uses. The same `Total Amount` works by program, by month, or in a card with no grouping.
- All three should agree on the totals. If they do not, compare row counts first.
