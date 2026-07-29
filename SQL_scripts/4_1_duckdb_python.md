# Install pyenv on WSL
**Install pyenv**
```
curl -fsSL https://pyenv.run | bash
```

**Add this to your bash path by editing ~/.bashrc and add:**
```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

**Install python version and create the environment**
```
pyenv install 3.11.6
pyenv virtualenv 3.11.6 motherduck
```

**Activate the environment**
```
pyenv activate motherduck
```

**Install duckdb**
```
pip install --upgrade pip
pip install duckdb

```

**Test if the motherduck package works**
```python
python -c "import duckdb; print(duckdb.__version__)"
```

**Test it in VS Code**

You can also verify the install directly inside VS Code instead of the terminal:

1. Open the folder in VS Code and make sure the `motherduck` interpreter is selected (Command Palette → **Python: Select Interpreter** → choose the `motherduck` pyenv environment).
2. Create a file named `test_duckdb.py` with:

```python
import duckdb

print("DuckDB version:", duckdb.__version__)
```

3. Run it with the ▶ **Run Python File** button (or `F5`). You should see the version number and `[(42,)]` printed in the terminal.

**Delete pyenv motherduck environment**
```
pyenv uninstall motherduck
```