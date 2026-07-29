# Comparing Cloud vs. Local Compute

The primary difference between local and cloud compute lies in **where the data is processed** and **how the underlying resources are managed**. While DuckDB provides the engine for both, MotherDuck extends that engine into a managed cloud environment to handle larger scales and team collaboration.

**Local Compute (DuckDB)**

- Runs entirely **in-process** on your own machine.
- Data lives on your local disk and stays there during processing.
- Performance is bound by your laptop's CPU, memory, and disk.
- Ideal for fast, iterative exploration of data that fits on your machine.

**Cloud Compute (MotherDuck)**

- Runs on **managed, serverless cloud infrastructure**.
- Data is stored in the cloud and results are streamed back to you.
- Scales beyond local hardware limits and supports team collaboration.
- Ideal for large historical datasets and shared, governed workflows.

## Key Technical Differences

| Feature | Local Compute (DuckDB) | Cloud Compute (MotherDuck) |
| --- | --- | --- |
| **Query Plan** | Standard operators like table scans and projections. | Includes **hybrid operators** (e.g., `Batch download sync`) for cloud-to-local data streaming. |
| **Data Movement** | Minimal; data stays on the local disk. | Significant; results are **streamed from the cloud** to your local machine. |
| **Optimization** | Performance depends on local hardware and efficient SQL (e.g., using `EXTRACT` instead of `strftime`). | **Filtering early** in the cloud is critical to shrink the amount of data moved across the network, which improves performance. |

## The Hybrid Advantage

One of the most powerful features of this ecosystem is the ability to run **hybrid workflows**. You can write a single SQL query that **unions local data** (like a 2024 CSV on your desktop) with **historical cloud data** (2010–2023 stored in MotherDuck). In these scenarios, DuckDB intelligently splits the work: using local compute for fast hotspot detection or visualization and cloud compute for deeper historical or business insights.

---

## 1. Union Local + Cloud Complaints by Year (EXPLAIN ANALYZE)

_Combines yearly complaint counts from the local `elevator_requests` view with the MotherDuck cloud sample table, then reports the actual plan and timing so you can see the hybrid operators at work._

```sql
EXPLAIN ANALYZE
WITH local_counts AS (
  SELECT
    EXTRACT(YEAR FROM "Created Date") AS year,
    COUNT(*) AS complaints,
    'local' AS source
  FROM elevator_requests
  GROUP BY year
),
md_counts AS (
  SELECT
    EXTRACT(YEAR FROM created_date) AS year,
    COUNT(*) AS complaints,
    'sample_data' AS source
  FROM sample_data.nyc.service_requests
  WHERE complaint_type ILIKE '%elevator%' 
  GROUP BY year
)
SELECT *
FROM local_counts
UNION ALL
SELECT *
FROM md_counts
ORDER BY year, source;
```

