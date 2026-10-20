# 06. SLA Attainment

**Question:** What percent of monthly results met their goal, when some metrics are "higher is better" and others are "lower is better"?

## SQL

```sql
WITH Scored AS (
    SELECT
        Program,
        Metric,
        CASE
            WHEN HigherIsBetter = 1 AND Result >= Goal THEN 1
            WHEN HigherIsBetter = 0 AND Result <= Goal THEN 1
            ELSE 0
        END AS Met
    FROM dbo.FactSLA
)
SELECT
    Program,
    Metric,
    COUNT(*) AS ReportedCount,
    SUM(Met) AS MetCount,
    CAST(SUM(Met) AS DECIMAL(9,4)) / COUNT(*) AS AttainmentPct
FROM Scored
GROUP BY Program, Metric
ORDER BY Program, Metric;
```

## Power Query

```m
let
    Scored = Table.AddColumn(
        FactSLA,
        "Met",
        each
            if [HigherIsBetter]
            then (if [Result] >= [Goal] then 1 else 0)
            else (if [Result] <= [Goal] then 1 else 0),
        Int64.Type
    ),
    Grouped = Table.Group(
        Scored,
        {"Program", "Metric"},
        {
            {"ReportedCount", each Table.RowCount(_), Int64.Type},
            {"MetCount", each List.Sum([Met]), Int64.Type}
        }
    ),
    Result = Table.AddColumn(Grouped, "AttainmentPct", each [MetCount] / [ReportedCount], type number)
in
    Result
```

## DAX

```dax
SLA Met Count =
SUMX (
    FactSLA,
    VAR Hit =
        IF (
            FactSLA[HigherIsBetter],
            FactSLA[Result] >= FactSLA[Goal],
            FactSLA[Result] <= FactSLA[Goal]
        )
    RETURN
        IF ( Hit, 1, 0 )
)

SLA Reported Count = COUNTROWS ( FactSLA )

SLA Attainment % = DIVIDE ( [SLA Met Count], [SLA Reported Count] )
```

Put `Program` and `Metric` on a matrix. The same measure also gives a total across metrics.

## What to notice

- The rule is the same in all three layers: compare against the goal in the direction the row says. Storing `Goal` and `HigherIsBetter` **on each row** is what makes that possible, so design the data this way up front.
- SQL and Power Query stop at a fixed grain (program and metric). DAX answers at any grain, including by month or for the whole portfolio, without writing a new query.
- Attainment here is the percent of **reported results** that met goal. It is not the average result. Be explicit about which one a dashboard shows.
