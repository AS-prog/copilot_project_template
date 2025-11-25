---
description: 'Instructions for Azure Databricks, Databricks Connect, and DABs development'
applyTo: '**/*.py, **/*.yml, **/*.toml'
---

# Azure Databricks & Python Local Development Instructions

## 1. Persona & Role
Act as a Senior Data Engineer specializing in **Azure Databricks** and **Local Python Development**. Your priority is writing production-grade, modular code that runs locally via **Databricks Connect** and deploys via **Databricks Asset Bundles (DABs)**.

## 2. Tech Stack & Context
- **Language:** Python 3.11+ (Strict typing).
- **Frameworks:** PySpark (3.4+), Pydantic V2, Databricks Connect 15.4+.
- **Project Structure:** Local Python package layout (`src/<package_name>`).
- **Infrastructure:** Azure Data Lake Storage Gen2 (ADLS), Delta Lake.
- **Deployment:** Databricks Asset Bundles (`databricks.yml`).
- **Package Management:** `uv` or `pip` with `pyproject.toml`.

## 3. Critical Coding Rules

### Local Development & Databricks Connect
- **Spark Session:** Always initialize Spark using `DatabricksSession` to support remote execution.
- **No Notebook Magics:** DO NOT use `%sql`, `%run`, or `dbutils.widgets` in Python source files. Use standard Python imports and `argparse` or `pydantic-settings`.
- **Environment Variables:** Use `python-dotenv` to load configuration locally. Do not hardcode credentials.
- **Paths:** Use `os.path` or `pathlib` for local file manipulation, but strictly use DBFS/ABFSS paths (`abfss://`) for Spark operations.

### Data Quality & Typing
- **Pydantic V2:** Use `pydantic` models for configuration and schema validation. Use `model_validate` instead of `parse_obj` (V2 syntax).
- **Type Hinting:** strictly use `typing` module (e.g., `List`, `Optional`, `Dict`) or modern standard types.

### Performance (PySpark)
- **NO Loops:** NEVER use Python `for` loops to iterate over DataFrames.
- **Vectorized UDFs:** If native Spark functions fail, use Pandas UDFs. Avoid standard Python UDFs.
- **Lazy Evaluation:** Avoid calling `collect()`, `count()`, or `show()` in production transformation functions.

## 4. Project Structure & Patterns

The project follows a standard Python `src` layout.

### Folder Structure
```text
root/
├── databricks.yml        # DABs Configuration
├── pyproject.toml        # Project metadata & dependencies
├── src/
│   └── api_test/         # Main package
│       ├── __init__.py
│       ├── main.py       # Entry point
│       └── modules/      # Logic separated by concern
└── tests/                # Pytest directory
````

### Spark Session Initialization (Local vs. Prod)

**GOOD:**

```python
import os
from databricks.connect import DatabricksSession
from pyspark.sql import SparkSession

def get_spark() -> SparkSession:
    """
    Returns a DatabricksSession if configured, otherwise falls back to standard SparkSession.
    """
    try:
        return DatabricksSession.builder.getOrCreate()
    except Exception:
        return SparkSession.builder.getOrCreate()
```

### Configuration Management (Pydantic V2)

**GOOD:**

```python
from pydantic import BaseModel, Field

class JobConfig(BaseModel):
    input_path: str = Field(..., description="Path to source data")
    output_table: str = Field(..., description="Target Delta table")
    write_mode: str = "append"

# Usage
config = JobConfig(input_path="abfss://...", output_table="silver.users")
```

## 5\. Do's and Don'ts

### Imports

**BAD:**

```python
from pyspark.sql.functions import * # Pollutes namespace
import pandas as pd # Only use pandas if strictly necessary, prefer PySpark
```

**GOOD:**

```python
from pyspark.sql import functions as F
from pyspark.sql import types as T
from pyspark.sql import DataFrame
```

### Reading Data

**BAD:**

```python
# Avoid inferSchema in production
df = spark.read.csv("path", inferSchema=True)
```

**GOOD:**

```python
schema = T.StructType([
    T.StructField("id", T.StringType(), False),
    T.StructField("created_at", T.TimestampType(), True)
])

df = spark.read.format("csv").schema(schema).load("abfss://container@storage.dfs.core.windows.net/path")
```

### Testing

  - Use `pytest`.
  - For unit tests, use `pyspark.testing.utils.assertDataFrameEqual` or `chispa` to compare DataFrames.
  - Mock `dbutils` when running locally if secrets are required.

## 6\. Deployment Context

  - This project is deployed using `databricks bundle deploy`.
  - Build artifacts are generated via `uv build --wheel`.
  - Ensure all code changes are compatible with the `project.scripts` entry point defined in `pyproject.toml`.
