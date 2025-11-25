---
description: 'Master Instructions: Project Architecture, Local Dev (Connect), and Orchestration'
applyTo: '**/*.py, **/*.yml, **/*.toml'
---

# Master Project Instructions: API Test (Local & DABs)

## 1. Domain-Specific Standards (Orchestration)
Before generating code, verify the file type and apply the specific rules from these reference files:
- **Python Logic:** Follow rules in `.github/python.instructions.md` (Type hinting, logging, error handling).
- **SQL & Delta:** Follow rules in `.github/databricks-sql.instructions.md` (ANSI, CTEs, Optimization).
- **Commits:** Follow rules in `.github/conventional-commits.instructions.md` (Conventional Commits in Spanish).
- **PRs:** Follow rules in `.github/copilot-pull-request-description-instructions.md`.

## 2. Project Architecture & Context
- **Execution Mode:** **Local-First**. Code runs locally on your machine using `databricks-connect`, NOT in Notebooks.
- **Deployment:** Managed via Databricks Asset Bundles (`databricks.yml`).
- **Dependency Management:** Uses `uv` and `pyproject.toml`.
- **Structure:**
  ```text
  src/
  └── api_test/           # Main Package
      ├── main.py         # Entry Point (referenced in pyproject.toml)
      └── modules/        # Domain logic
````

## 3\. Local Development Rules (Databricks Connect)

### Spark Session Management

Since we run locally, NEVER use the global `spark` variable. Always use this pattern to support both local and remote execution:

```python
from databricks.connect import DatabricksSession
from pyspark.sql import SparkSession

def get_spark() -> SparkSession:
    """Gets or creates a Spark session compatible with Databricks Connect."""
    try:
        return DatabricksSession.builder.getOrCreate()
    except Exception:
        return SparkSession.builder.getOrCreate()
```

### Environment Variables

  - DO NOT rely on `dbutils.widgets` for configuration.
  - Use `python-dotenv` to load local `.env` files.
  - Use **Pydantic V2** (`pydantic_settings`) to validate configurations.

### File System & Paths

  - **Local FS:** Use standard `pathlib` or `os` for local artifacts.
  - **Distributed FS:** Use `abfss://` paths strictly for Spark operations.
  - **No Magic Commands:** Never use `%run`, `%sql`, or `%pip`.

## 4\. Specific Library Constraints

### Pydantic V2

We use Pydantic `>2.0`. Enforce V2 syntax:

  - **Use:** `model_validate()` instead of `parse_obj()`.
  - **Use:** `model_dump()` instead of `dict()`.
  - **Config:** Use `model_config = ConfigDict(...)` instead of `class Config:`.

### Databricks Asset Bundles (DABs)

  - **Validation:** When modifying `databricks.yml`, ensure the `bundle.name` matches the directory or project scope.
  - **Build:** The project builds as a wheel (`uv build --wheel`). Ensure `pyproject.toml` dependencies sync with imports.

## 5\. Testing Strategy

  - **Framework:** `pytest`.
  - **Integration:** When testing Spark transformations, use the `get_spark()` session.
  - **Mocking:** You MUST mock `dbutils` (secrets/fs) when running unit tests locally to avoid connection errors.
