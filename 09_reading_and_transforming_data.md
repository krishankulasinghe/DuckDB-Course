# Reading and Transforming Data

This walkthrough opens local DuckDB database files, inspects their tables, and runs aggregation queries to summarize elevator complaints.

## 1. Open the Elevator Database

_Launch the DuckDB CLI against the persistent `elevator_data.duckdb` file so its tables are available._

```
duckdb elevator_data.duckdb
```

## 2. Inspect the Available Tables

_List the tables in the database and view the column structure of `elevator_requests`._

```sql
Show tables

describe elevator_requests;
```

## 3. Count Complaints by Type (Raw Data)

_Group the raw `elevator_requests` rows by complaint type and rank them from most to least frequent._

```sql
SELECT 
      "Complaint Type",
      COUNT(*) AS complaint_count
  FROM elevator_requests
  GROUP BY "Complaint Type"
  ORDER BY complaint_count DESC;
```

## 4. Open the Transformed (ELT) Database

_Launch the DuckDB CLI against `elt.duckdb`, which holds the cleaned/transformed tables._

```
duckdb elt.duckdb
```

## 5. Count Complaints by Type (Cleaned Data)

_Group the cleaned `clean_requests` table by complaint type to compare against the raw counts above._

```sql
SELECT 
      complaint_type,
      COUNT(*) AS complaint_count
  FROM clean_requests
  GROUP BY complaint_type
  ORDER BY complaint_count DESC;
```

# Troubleshooting: The _ctypes Error

If Python fails to import DuckDB with a `_ctypes` error, your pyenv Python was likely built without the required system libraries. The steps below reinstall those libraries and rebuild the environment.

## Install Required Linux Packages

_Install the build dependencies needed to compile Python with C extension support._

```
sudo apt update
sudo apt install -y build-essential libffi-dev libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
    liblzma-dev
```

## Clean Up the Existing pyenv Setup

_Remove the broken environment and Python version so they can be rebuilt cleanly._

```
pyenv uninstall motherduck
pyenv uninstall 3.11.6
pyenv install 3.11.6
```

## Re-create the Environment

_Recreate the `motherduck` virtual environment and reinstall DuckDB and pandas._

```
pyenv virtualenv 3.11.6 motherduck
pyenv activate motherduck
python -m pip install --upgrade pip
pip install duckdb pandas
```