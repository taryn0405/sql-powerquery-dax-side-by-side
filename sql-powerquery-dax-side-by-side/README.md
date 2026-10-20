<h1 align="center">SQL, Power Query, and DAX Side by Side</h1>

<p align="center">
  The same six business questions answered in three languages, on the same mock data,
  with notes on when each layer is the right place to do the work.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SQL-T--SQL-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white" alt="T-SQL" />
  <img src="https://img.shields.io/badge/Power%20Query-M-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" alt="Power Query M" />
  <img src="https://img.shields.io/badge/DAX-Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" alt="DAX" />
  <img src="https://img.shields.io/badge/Data-100%25%20Mock-2EA44F?style=for-the-badge" alt="Mock data only" />
</p>

---

<!-- After you test the queries, add a line here such as:
> **Tested:** SQL scripts were run on SQL Server (version), and Power Query and DAX were tested in Power BI Desktop (month year).
-->

## Why This Repo Exists

A reporting solution usually spans three layers, and the same question can be answered in any of them. Choosing the wrong layer leads to slow refreshes, numbers that disagree between reports, or logic nobody can find. This repo shows each question solved three ways so the tradeoffs are easy to see.

## Which Layer Should Do What

| Layer | Best for | Watch out for |
| --- | --- | --- |
| **SQL** | Heavy lifting near the source: joins, aggregation, window functions, shared date tables | Logic that only one report needs, and results that cannot respond to report slicers |
| **Power Query** | Ingestion and shaping before the model: typing, cleaning, merging files, reshaping | Slow refreshes on large tables, and logic that hides in many queries |
| **DAX** | Measures that respond to slicers and filters, time intelligence, ratios | Row-level logic that belongs upstream, and slow measures over huge grids |

A good rule of thumb: shape and store in SQL or Power Query, calculate in DAX.

## Comparisons

| Page | Question |
| --- | --- |
| [01. Group and Total](comparisons/01-group-and-total.md) | Total and count by program and vendor |
| [02. Running Total](comparisons/02-running-total.md) | Cumulative amount over time for each program |
| [03. Fiscal Calendar](comparisons/03-fiscal-calendar.md) | Fiscal year, month, and quarter for an October fiscal start |
| [04. Rolling Window](comparisons/04-rolling-window.md) | Rolling 3-month totals, and why row counting can mislead |
| [05. Year over Year](comparisons/05-year-over-year.md) | Same month last year, with safe division |
| [06. SLA Attainment](comparisons/06-sla-attainment.md) | Scoring higher-is-better and lower-is-better goals together |

Each page shows the SQL, Power Query, and DAX versions, followed by what to notice.

## Set Up the Data

**SQL:** run [`mock-data/01_create_and_load.sql`](mock-data/01_create_and_load.sql) in SQL Server or Azure SQL (a free SQL Server Express install works). It creates `FactInvoices`, `DimDate`, and `FactSLA`, and ends with a row-count check (120, 822, and 216).

**Power Query and DAX:** load the mock queries from the DAX Pattern Cheat Sheet repo (`FactInvoices`, `DimDate`, `FactSLA`) into Power BI Desktop and follow its setup steps for relationships and the date table. The Power Query pages also use functions from the Power Query Function Library repo (`fnRunningTotal`, `fnFiscalCalendar`).

The SQL script generates the same rows as the Power Query mock queries, so results can be compared across layers.

## Column Name Differences

| Concept | SQL | Power Query and DAX |
| --- | --- | --- |
| Invoice date | `InvoiceDate` | `Date` |
| Calendar date | `FullDate` | `Date` |
| Month (SLA data) | `MonthStart` | `Month` |
| Higher is better flag | `HigherIsBetter` (BIT) | `HigherIsBetter` (logical) |

## About the Data

All data in this repository is fabricated. No client, employer, or production data is included.

## Author

**Taryn Ledford**
[LinkedIn](https://www.linkedin.com/in/taryn-ledford)
