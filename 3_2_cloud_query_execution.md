# Cloud Query Execution

Two helpful keywords used throughout this file:

- **`EXPLAIN`** — Shows the query plan (how DuckDB *would* run the query) **without executing it**. `EXPLAIN ANALYZE` actually runs the query and reports the **real time and row counts** at each step, so you can find bottlenecks.
- **`strftime`** — "string format time." Converts a date/timestamp into formatted text. `strftime('%Y', created_date)` extracts the **year** so rows can be grouped by year (`%Y-%m` would give year-month).

---

## 1. Plan Only — Local View (EXPLAIN)

_Shows the execution plan for grouping local `elevator_requests` complaints by year, without running it._

```sql
EXPLAIN 
SELECT 
    strftime('%Y', "created date") AS year,
    COUNT(*) AS complaints
FROM elevator_requests
GROUP BY year
ORDER BY year;
```

## 2. Plan Only — Cloud Table (EXPLAIN)

_Shows the execution plan for the same yearly count against the MotherDuck cloud sample table._

```sql
EXPLAIN
SELECT 
    strftime('%Y', created_date) AS year,
    COUNT(*) AS complaints
FROM sample_data.nyc.service_requests
GROUP BY year
ORDER BY year;
```

## 3. Run + Measure — Cloud Table (EXPLAIN ANALYZE)

_Runs the cloud query and reports actual timing and rows processed at each step._

```sql
EXPLAIN ANALYZE
SELECT 
    strftime('%Y', created_date) AS year,
    COUNT(*) AS complaints
FROM sample_data.nyc.service_requests
GROUP BY year
ORDER BY year;
```

## 4. Run + Measure — Local View (EXPLAIN ANALYZE)

_Runs the local view query and reports actual timing, useful for comparing local vs. cloud performance._

```sql
EXPLAIN ANALYZE
SELECT 
    strftime('%Y', "created date") AS year,
    COUNT(*) AS complaints
FROM elevator_requests
GROUP BY year
ORDER BY year;
```

## 5. Run + Measure — Using EXTRACT Instead of strftime

_Same cloud query, but uses `EXTRACT(YEAR FROM ...)` to pull the year as a number, to compare plans/timing against `strftime`._

```sql
EXPLAIN ANALYZE
SELECT
  EXTRACT(YEAR FROM created_date) AS year,
  COUNT(*) AS complaints
FROM sample_data.nyc.service_requests
GROUP BY year
ORDER BY year;
```