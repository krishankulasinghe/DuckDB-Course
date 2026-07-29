# DuckDB Setup and Installation

You can set it up as a command-line tool, use its built-in UI, or install it as a Python library.

## 1. Installing the Command Line Interface (CLI)

The CLI allows you to run DuckDB directly from your terminal or PowerShell.

- **Download:** Visit the official installation page at [DuckDB Installation – DuckDB](https://duckdb.org/install/?platform=windows&environment=cli).
- **Windows:** Download the zip file, extract the `duckdb.exe` file, and place it in a folder of your choice (e.g., `C:\opt\duckdb`).
- **Mac/Linux:** You can use a `curl` command provided on the website to download it directly.
- **Run:** You can start DuckDB by double-clicking the executable, which opens a terminal immediately.
- **Global Access (Optional):** To run `duckdb` from any folder in your terminal, add the path to your folder (e.g., `C:\opt\duckdb`) to your system's **Path environment variable**.

## 2. Using the DuckDB UI

Since version 1.2.1, a web-based UI is bundled directly with the DuckDB CLI. There are two ways to launch it:

- **From Terminal:** Run the command `duckdb -ui`.
- **From inside DuckDB:** If you are already in the DuckDB prompt, run the SQL command `CALL start_ui();`.

This will open the UI in your web browser at `localhost:4213`, where you can write queries in a notebook-style environment.

## 3. Setting Up DuckDB in Python

If you want to use DuckDB for data engineering or automation scripts, you can install it as a package.

- **Virtual Environment (Recommended):** It is best practice to create a virtual environment first to keep your project isolated.
- **Installation:** Use pip to install the library:

```
pip install duckdb
```

- **Verification:** You can verify the installation by running a simple script to print the version:

```python
import duckdb
print(duckdb.__version__)
```

## 4. Connecting to MotherDuck (Cloud Setup)

If you want to extend your local DuckDB to the cloud for collaboration or scale, you can connect it to **MotherDuck**.

- **Create Account:** Sign up at `motherduck.com`.
- **Access Token:** Go to **Settings > Integrations** to create an access token. Copy this token and save it securely.
- **Configure Environment Variable:** Set the token as a system variable named `MOTHERDUCK_TOKEN`:

```
export MOTHERDUCK_TOKEN=your_token_here      # macOS / Linux
setx MOTHERDUCK_TOKEN "your_token_here"      # Windows PowerShell
```

- **Connect:** Once the token is set, you can connect to the cloud from your local DuckDB by running:

```
duckdb md:
```

## 5. Running DuckDB from the Console or a Script

Once the CLI is installed (and optionally added to your Path), you can run DuckDB in a few ways:

- **Interactive console:** Open a terminal (PowerShell, Command Prompt, or bash) and start an in-memory session:

```
duckdb
```

- **Open a persistent database file:** Pass a file name to create or reuse a local database:

```
duckdb my_database.duckdb
```

- **Run a single query inline:** Use the `-c` flag to execute one command and exit:

```
duckdb -c "SELECT 42 AS answer;"
```

- **Run a SQL script file:** Pipe a `.sql` file into the CLI to execute it as a batch:

```
duckdb my_database.duckdb < script.sql      # macOS / Linux / PowerShell
```

- **Windows .cmd / .bat file:** You can wrap commands in a batch file so they run with a double-click:

```bat
@echo off
duckdb my_database.duckdb -c "SELECT COUNT(*) FROM my_table;"
pause
```

## Test the Connection

```
duckdb md:
SHOW DATABASES;
```