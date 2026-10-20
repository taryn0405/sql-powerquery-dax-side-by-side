# 03. Fiscal Calendar

**Question:** For each date, what are the fiscal year, month, and quarter with an October fiscal year start?

Convention: the fiscal year is named for the calendar year in which it ends (Oct 2025 to Sep 2026 is FY2026).

## SQL

The full build is in [`mock-data/01_create_and_load.sql`](../mock-data/01_create_and_load.sql). The core logic:

```sql
-- Fiscal year
CASE
    WHEN @FiscalStartMonth > 1 AND MONTH(d) >= @FiscalStartMonth THEN YEAR(d) + 1
    ELSE YEAR(d)
END

-- Fiscal month (1 to 12)
((MONTH(d) - @FiscalStartMonth + 12) % 12) + 1

-- Fiscal quarter
((FiscalMonth - 1) / 3) + 1
```

Spot check:

```sql
SELECT FullDate, FiscalYear, FiscalMonth, FiscalQuarter, FiscalPeriodLabel
FROM dbo.DimDate
WHERE FullDate IN ('2025-09-30', '2025-10-01', '2026-01-01')
ORDER BY FullDate;
```

## Power Query

Using `fnFiscalCalendar` from the Power Query Function Library:

```m
= fnFiscalCalendar(#date(2024, 10, 1), #date(2026, 12, 31), 10)
```

## DAX

```dax
DimDate (DAX) =
VAR FiscalStartMonth = 10
VAR Base =
    ADDCOLUMNS (
        CALENDAR ( DATE ( 2024, 10, 1 ), DATE ( 2026, 12, 31 ) ),
        "Fiscal Year",
            IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) ),
        "Fiscal Month",
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
    )
RETURN
    ADDCOLUMNS ( Base, "Fiscal Quarter", ROUNDUP ( [Fiscal Month] / 3, 0 ) )
```

## Expected results (all three layers)

| Date | Fiscal Year | Fiscal Month | Fiscal Quarter |
| --- | --- | --- | --- |
| 2025-09-30 | 2025 | 12 | 4 |
| 2025-10-01 | 2026 | 1 | 1 |
| 2026-01-01 | 2026 | 4 | 2 |

## What to notice

- **Modulo with negative numbers:** for dates before the fiscal start month, `MONTH - FiscalStartMonth` is negative. In T-SQL, `%` returns a negative remainder for a negative dividend, and Power Query's `Number.Mod` can too. DAX's `MOD` takes the sign of the divisor, so it would wrap correctly on its own. Adding 12 first is harmless in DAX and keeps the logic identical everywhere.
- A date table built once in SQL is shared by every report. One built in Power Query or DAX lives inside a single model. Build it where the most people will reuse it.
