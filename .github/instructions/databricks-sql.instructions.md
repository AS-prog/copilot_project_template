---
description: 'Guidelines for writing efficient, ANSI-compliant Databricks SQL and Delta Lake queries'
applyTo: '**/*.sql, **/*.ddl, **/*.dml'
---

# Databricks SQL (ANSI) Guidelines

Instructions for generating high-performance, maintainable, and ANSI-compliant SQL code within the Databricks Lakehouse environment.

## General Instructions

- **Target Dialect**: Use Databricks SQL (Spark SQL). Ensure ANSI mode compliance (`spark.sql.ansi.enabled`).
- **Table Format**: Default to Delta Lake format for all DDL operations.
- **Readability**: Prioritize Common Table Expressions (CTEs) over nested subqueries for complex logic.
- **Idempotency**: Write scripts that can be re-run without error (e.g., use `CREATE OR REPLACE`, `IF NOT EXISTS`).

## Naming and Style Conventions

### Formatting
- Use **UPPERCASE** for SQL keywords (SELECT, FROM, WHERE).
- Use **snake_case** for table names, column names, and aliases.
- Indent code using 2 or 4 spaces (be consistent).
- Left-align root keywords; indent clauses/logic.

### Aliasing
- Always alias tables in `JOIN` operations.
- Use explicit `AS` for column aliases.
- Use meaningful, short aliases (e.g., `customer_orders` -> `co`, not `t1`).

## Performance Best Practices

### Optimization Techniques
- **Early Filtering**: Apply `WHERE` clauses as early as possible (predicate pushdown).
- **Avoid `SELECT *`**: Explicitly select only necessary columns to reduce I/O (column pruning).
- **Partition Pruning**: Always include partition columns in filter conditions when querying partitioned tables.
- **Z-Ordering**: Suggest `OPTIMIZE ... ZORDER BY (col)` for columns frequently used in `WHERE` clauses.

### Expensive Operations
- **Distinct Counts**: Use `approx_count_distinct(col)` instead of `count(distinct col)` for massive datasets where slight margin of error is acceptable.
- **Ordering**: Avoid global `ORDER BY` in intermediate CTEs or views; only sort in the final output if necessary.

## Common Patterns

### 1. CTEs vs Subqueries
Use CTEs to break down complex logic into readable steps.

#### Good Example - CTE
```sql
WITH high_value_orders AS (
    SELECT customer_id, sum(amount) as total_spend
    FROM orders
    WHERE order_date >= date_sub(current_date(), 30)
    GROUP BY customer_id
)
SELECT c.email, h.total_spend
FROM customers c
JOIN high_value_orders h ON c.id = h.customer_id
WHERE h.total_spend > 1000;
````

#### Bad Example - Nested Subquery

```sql
SELECT c.email, h.total_spend
FROM customers c
JOIN (
    SELECT customer_id, sum(amount) as total_spend
    FROM orders
    WHERE order_date >= date_sub(current_date(), 30)
    GROUP BY customer_id
) h ON c.id = h.customer_id
WHERE h.total_spend > 1000;
```

### 2\. Window Functions & Filtering

Use the `QUALIFY` clause (Databricks specific) to filter window functions without a subquery wrapper.

#### Good Example - QUALIFY

```sql
SELECT
    product_id,
    sales_amount,
    sales_date
FROM sales
QUALIFY row_number() OVER (PARTITION BY product_id ORDER BY sales_date DESC) = 1;
```

#### Bad Example - Subquery Wrapper

```sql
SELECT * FROM (
    SELECT
        product_id,
        sales_amount,
        sales_date,
        row_number() OVER (PARTITION BY product_id ORDER BY sales_date DESC) as rn
    FROM sales
) sub
WHERE rn = 1;
```

### 3\. Safe Casting

Use `try_cast` or `try_to_date` to handle dirty data gracefully without failing the job.

#### Good Example

```sql
-- Returns NULL if conversion fails instead of throwing an error
SELECT
    try_cast(input_string AS INTEGER) as clean_id,
    try_to_date(date_string, 'yyyy-MM-dd') as clean_date
FROM raw_data;
```

#### Bad Example

```sql
-- May cause job failure on malformed data
SELECT
    cast(input_string AS INTEGER) as clean_id,
    to_date(date_string, 'yyyy-MM-dd') as clean_date
FROM raw_data;
```

## Common Issues and Solutions

| Issue | Solution | Example |
| :--- | :--- | :--- |
| Skewed Joins | Broadcast small tables | `JOIN /*+ BROADCAST(t_small) */ t_small` |
| Exploding Joins | Check cardinality before joining | Validate uniqueness of join keys |
| NULL comparison | Use safe null checks | Use `<=>` (null-safe equality) or `coalesce` |
| Hardcoded Dates | Use dynamic date functions | `date_sub(current_date(), 1)` |

## DDL & Table Management

  - **Delta Features**: Enable schema evolution checks or merge schema options only when necessary.
  - **Comments**: Always include comments for tables and complex columns.

```sql
CREATE TABLE IF NOT EXISTS gold_sales (
    sale_id BIGINT COMMENT 'Primary Key',
    amount DECIMAL(10, 2),
    processed_at TIMESTAMP
)
USING DELTA
PARTITIONED BY (processed_at)
COMMENT 'Aggregated sales data for reporting';
```

## Validation

  - **Syntax Check**: Verify code compiles in Databricks SQL Editor.
  - **Explain Plan**: Run `EXPLAIN` to verify predicate pushdown and join strategies.
  - **Formatting**: Ensure SQL is formatted according to project linter rules.