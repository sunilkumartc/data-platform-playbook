# dbt (Data Build Tool)

> SQL-based transformations for analytics engineering.

## Overview

dbt (data build tool) enables analytics engineers to transform data in their warehouses using SQL. It's the standard tool for analytics engineering.

## Key Concepts

### Models

**Model** = SQL transformation

**Example:**
```sql
-- models/staging/stg_orders.sql
{{ config(materialized='view') }}

select
    order_id,
    user_id,
    amount,
    created_at
from {{ source('raw', 'orders') }}
where status = 'completed'
```

### Tests

**Test** = Data quality check

**Example:**
```yaml
# models/schema.yml
models:
  - name: stg_orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
```

### Macros

**Macro** = Reusable SQL

**Example:**
```sql
-- macros/date_spine.sql
{% macro date_spine(start_date, end_date) %}
    select date_day
    from {{ ref('date_spine') }}
    where date_day between '{{ start_date }}' and '{{ end_date }}'
{% endmacro %}
```

## Best Practices

1. **Staging models** - Clean raw data first
2. **Intermediate models** - Build incrementally
3. **Marts** - Final business logic
4. **Tests** - Test everything
5. **Documentation** - Document all models

## Related Topics

- **[Airflow](airflow.md)** - Orchestrating dbt
- **[Data Orchestration](index.md)** - Orchestration overview

---

**Next**: [Data Processing →](../data-processing/index.md)

