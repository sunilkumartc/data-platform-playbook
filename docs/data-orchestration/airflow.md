# Apache Airflow

> Workflow orchestration for data pipelines.

## Overview

Apache Airflow is an open-source platform for programmatically authoring, scheduling, and monitoring workflows. It's the de facto standard for data pipeline orchestration.

## Key Concepts

### DAGs (Directed Acyclic Graphs)

**DAG** = Workflow definition

**Example:**
```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG(
    'ingest_data',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily'
)

extract = BashOperator(
    task_id='extract',
    bash_command='python extract.py',
    dag=dag
)

transform = BashOperator(
    task_id='transform',
    bash_command='python transform.py',
    dag=dag
)

load = BashOperator(
    task_id='load',
    bash_command='python load.py',
    dag=dag
)

extract >> transform >> load
```

### Tasks

**Task** = Single unit of work

**Types:**
- Operators (Bash, Python, SQL)
- Sensors (wait for conditions)
- Hooks (connect to external systems)

### Operators

**BashOperator** - Run bash commands
**PythonOperator** - Run Python functions
**SQLOperator** - Run SQL queries

## Best Practices

1. **Idempotency** - Tasks should be rerunnable
2. **Atomicity** - Tasks should succeed or fail completely
3. **Dependencies** - Use clear task dependencies
4. **Error handling** - Handle failures gracefully
5. **Monitoring** - Set up alerts for failures

## Related Topics

- **[dbt](dbt.md)** - SQL-based transformations
- **[Data Orchestration](index.md)** - Orchestration overview

---

**Next**: [dbt →](dbt.md)

