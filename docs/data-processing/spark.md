# Apache Spark

> Distributed data processing at scale.

## Overview

Apache Spark is a unified analytics engine for large-scale data processing. It's the standard for batch and streaming data processing.

## Key Concepts

### RDDs (Resilient Distributed Datasets)

**RDD** = Immutable distributed collection

**Example:**
```python
rdd = sc.parallelize([1, 2, 3, 4, 5])
rdd.map(lambda x: x * 2).collect()
# [2, 4, 6, 8, 10]
```

### DataFrames

**DataFrame** = Structured data with schema

**Example:**
```python
df = spark.read.parquet("s3://data/events/")
df.filter(df.date == "2024-01-15") \
  .groupBy("user_id") \
  .agg(sum("amount").alias("total")) \
  .show()
```

### Datasets

**Dataset** = Typed DataFrame (Scala/Java)

## Best Practices

1. **Partitioning** - Partition data appropriately
2. **Caching** - Cache frequently used data
3. **Broadcast joins** - Broadcast small tables
4. **Avoid shuffles** - Minimize data movement
5. **Resource tuning** - Tune executor memory/cores

## Related Topics

- **[Data Processing](index.md)** - Processing overview
- **[Data Architecture](../data-architecture/index.md)** - Storage patterns

---

**Next**: [BigQuery →](bigquery.md)

