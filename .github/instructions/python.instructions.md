---
description: 'Python coding conventions for Data Engineering'
applyTo: '**/*.py'
---

# Python Coding Conventions (Databricks & Data Engineering)

## 1. Modern Python Syntax (Python 3.10+)
- **Type Hinting:** Use standard collection types. DO NOT import `List`, `Dict`, `Tuple` from `typing`.
  - BAD: `def process(items: List[str]) -> Dict[str, int]:`
  - GOOD: `def process(items: list[str]) -> dict[str, int]:`
- **Return Types:** Always specify the return type, even if it is `-> None`.
- **F-Strings:** Use f-strings for string interpolation instead of `.format()` or `%`.

## 2. Documentation & Logging
- **Docstrings:** Use **Google Style** docstrings. Must include `Args`, `Returns`, and `Raises`.
- **No Prints:** NEVER use `print()` for debugging in production code. Use the standard `logging` library.
  - `logger.info("Processing started")` instead of `print("Processing started")`.
- **Comments:** Explain *why* a block of code exists (business logic), not *what* it does (syntax).

## 3. Error Handling & Robustness
- **Specific Exceptions:** Never use bare `except:`. Catch specific exceptions (e.g., `ValueError`, `AnalysisException`).
- **Fail Fast:** Validate inputs at the beginning of the function.
- **Paths:** When manipulating file paths (even generic DBFS paths), prefer `os.path.join` or `pathlib` over string concatenation to avoid slash (`/`) errors.

## 4. Code Style (PEP 8 adapted for Data)
- **Line Length:** Extend limit to **100 characters** (standard in Data teams to accommodate long Spark chains/SQL queries) rather than the strict 79.
- **Imports:** Group imports:
    1. Standard library (`os`, `json`, `logging`)
    2. Third-party (`pandas`, `requests`)
    3. PySpark (`pyspark.sql`)
    4. Local application imports

## 5. Testing & Quality
- **Unit Tests:** Use `pytest` conventions.
- **Mocking:** Do not make real calls to Azure services (KeyVault, ADLS) in unit tests. Use `unittest.mock` to mock `dbutils` or external APIs.

## 6. Examples

### Proper Function Structure
```python
import logging

# Configure logger at module level
logger = logging.getLogger(__name__)

def calculate_metric(value: float, factor: float = 1.0) -> float:
    """Calculates a business metric based on input value.

    Args:
        value (float): The raw input value.
        factor (float, optional): Multiplier factor. Defaults to 1.0.

    Returns:
        float: The calculated metric rounded to 2 decimals.

    Raises:
        ValueError: If value is negative.
    """
    if value < 0:
        logger.error(f"Invalid input: {value}")
        raise ValueError("Value must be non-negative")

    result = value * factor
    return round(result, 2)
````

### Type Hinting for DataFrames

When returning a Spark DataFrame in a helper function:

```python
from pyspark.sql import DataFrame

def clean_data(df: DataFrame) -> DataFrame:
    """Applies standard cleaning transformations."""
    return df.dropna()