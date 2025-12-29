# Data Architecture

> "If a data problem can't be explained in one screen, the system is already broken."

> Designing storage systems that are fast, cost-effective, and scalable.

## Overview

Storage is where data lives. Get the architecture right, and queries are fast, costs are low, and operations are smooth. Get it wrong, and you'll pay in performance, cost, and complexity.

## Key Topics

### [Storage](storage.md)
Data lake vs warehouse, partitioning, formats, lifecycle policies.

**Learn about:**
- Data lake vs data warehouse
- Storage tiers (hot, warm, cold)
- Partitioning strategies
- File formats (Parquet, Avro, Delta)
- Compression and optimization

### [Lakehouse](lakehouse.md)
Modern approach combining lake storage with warehouse capabilities.

**Learn about:**
- Lakehouse architecture
- Delta Lake, Iceberg, Hudi
- Benefits and trade-offs
- Implementation patterns

### [Ingestion Architecture](ingestion-architecture.md)
How ingestion fits into overall architecture.

**Learn about:**
- Ingestion patterns
- Storage layer design
- Data flow architecture
- Scalability patterns

## Architecture Patterns

### Data Lake

**Characteristics:**
- Schema-on-read (flexible schemas)
- File-based storage (S3, GCS, ADLS)
- Supports structured, semi-structured, unstructured data
- Lower storage cost
- Requires compute engine (Spark, Presto) for queries

**Best for:**
- Raw data storage
- Diverse data types (logs, images, documents)
- Cost-sensitive, large volumes
- ELT patterns (load first, transform later)

### Data Warehouse

**Characteristics:**
- Schema-on-write (enforced schemas)
- Table-based storage
- Optimized for SQL queries
- Higher storage cost
- Integrated compute

**Best for:**
- Curated, analysis-ready data
- SQL-heavy workloads
- Business intelligence, reporting
- Fast query performance

### Lakehouse

**Characteristics:**
- Combines lake storage with warehouse capabilities
- Cost-effective storage (lake)
- Fast queries (warehouse)
- Single source of truth

**Best for:**
- Modern data platforms
- Need both flexibility and performance
- Cost-conscious organizations

## Storage Tiers

| Tier | Use Case | Access Pattern | Cost |
|------|----------|----------------|------|
| **Hot** | Active queries, dashboards | Frequent, low latency | High |
| **Warm** | Ad-hoc analysis, reporting | Occasional, moderate latency | Medium |
| **Cold** | Compliance, historical | Rare, high latency acceptable | Low |

!!! tip "Lifecycle Policies"
    Automatically move data between tiers based on age/access patterns. Expected savings: 50-70% on storage costs.

## Related Topics

- **[Data Ingestion](../data-ingestion/index.md)** - Getting data into storage
- **[Data Processing](../data-processing/index.md)** - Processing stored data
- **[Data Quality](../data-quality/index.md)** - Ensuring data reliability

---

**Next**: [Storage →](storage.md)

