# DuckDB UI

Since version 1.2.1, a web-based UI is bundled directly with the DuckDB CLI. You can launch the interface in two ways, and it will automatically open in your web browser at `localhost:4213`:

**Start the UI**
```sql
duckdb -ui
```

**Start the UI when in the CLI**
```sql
CALL start_ui();
```

## Key Features and Tools

The UI is designed for both data exploration and management:

- **Notebook-style editor:** Write and run SQL in cells, keeping related queries and their results together.
- **Schema explorer:** Browse attached databases, schemas, tables, and views in a side panel without writing queries.
- **Result grids:** View query output in an interactive table you can sort and scroll, with column type information shown inline.
- **Query history:** Revisit and re-run previous queries from the session.
- **Local files and databases:** Query CSV/Parquet files and open persistent `.duckdb` database files directly from the interface.

## Important Considerations

- **Local only:** The UI runs entirely on your machine at `localhost:4213`; it is not exposed to the network by default.
- **Version requirement:** The bundled UI requires DuckDB **1.2.1 or later**.
- **Session state:** In-memory databases exist only while the session is open—use a persistent `.duckdb` file if you want your tables to survive a restart.
- **One process:** The UI shares the same DuckDB process, so changes made in the UI affect the database file directly.

# Create a View

```sql
CREATE VIEW elevator_requests AS
SELECT *
FROM read_csv_auto('311_Elevator_Service_Requests_.csv', HEADER=TRUE);
```

# Run a Query

```sql
SELECT 
    strftime('%Y-%m', "Created Date") AS month,
    COUNT(*) AS complaints
FROM elevator_requests
WHERE "Borough" = 'MANHATTAN'
GROUP BY month
ORDER BY month;
```