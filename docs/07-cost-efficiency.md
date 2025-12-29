# Cost Efficiency & Scale

Cost optimization isn't about cutting corners—it's about spending wisely. At scale, small inefficiencies compound into massive waste. This chapter provides practical, actionable guidance on reducing costs by 20-40% without sacrificing quality or performance.

## Cost Drivers

### Understanding Your Costs

**Typical cost breakdown:**

| Category | % of Total | Description |
|----------|-----------|-------------|
| **Compute** | 40-60% | Query execution, transformations |
| **Storage** | 20-30% | Data storage across tiers |
| **Network** | 5-15% | Data transfer, egress |
| **Operations** | 5-10% | Pipeline orchestration, monitoring |

**Compute costs:**
- Query execution (BigQuery, Snowflake slots)
- Transformation jobs (Spark, Dataflow)
- Streaming processing (Flink, Kafka)

**Storage costs:**
- Hot storage (frequently accessed)
- Warm storage (occasional access)
- Cold storage (rarely accessed)

**Network costs:**
- Cross-region transfers
- Egress to internet
- Inter-service communication

## Common Cost Traps

### Trap 1: Over-Ingestion

**Problem**: Ingesting data that's never used.

**Symptoms:**
- Tables with zero queries in 90 days
- High storage cost, low usage
- "Just in case" ingestion

**Solution:**
```sql
-- Identify unused tables
SELECT
  table_name,
  storage_bytes,
  last_query_time,
  days_since_last_query
FROM table_usage_stats
WHERE days_since_last_query > 90
ORDER BY storage_bytes DESC
```

**Action**: Archive or delete unused data. Expected savings: 10-20%.

### Trap 2: Inefficient File Formats

**Problem**: Using JSON or CSV instead of Parquet.

**Cost impact:**
- JSON: 5-10x larger than Parquet
- CSV: 3-5x larger than Parquet
- Higher storage + compute costs

**Solution**: Convert to Parquet post-ingestion.

```python
# Convert JSON to Parquet
df = spark.read.json("s3://raw/events/*.json")
df.write.parquet("s3://raw/events_parquet/", compression="zstd")
```

**Expected savings**: 50-70% on storage, 30-50% on compute.

### Trap 3: Streaming When Batch Would Suffice

**Problem**: Using streaming for use cases that don't need real-time.

**Cost impact**: Streaming is 3-5x more expensive than batch.

**Decision framework:**
- Real-time requirement (< 1 min)? → Streaming
- Near real-time (1-15 min)? → Micro-batch
- Batch acceptable (15+ min)? → Batch

**Example:**
```
Use case: Daily reporting dashboard
Current: Streaming (cost: $1000/month)
Better: Daily batch (cost: $200/month)
Savings: $800/month (80%)
```

### Trap 4: No Lifecycle Policies

**Problem**: All data in expensive hot storage.

**Solution**: Automatically move to cheaper tiers.

```yaml
# Example: S3 lifecycle policy
lifecycle:
  - days: 30
    move_to: STANDARD_IA  # 50% cheaper
  - days: 90
    move_to: GLACIER      # 80% cheaper
```

**Expected savings**: 50-70% on old data.

### Trap 5: Small Files Problem

**Problem**: Many small files (e.g., 10,000 files of 1MB each).

**Impact:**
- Slow queries (many file opens)
- Higher compute cost (overhead)
- Inefficient storage

**Solution**: Compact small files.

```python
# Compact small files
df = spark.read.parquet("s3://data/partition=2024-01-15/")
df.coalesce(10).write.parquet("s3://data/partition=2024-01-15/")
```

**Target**: 100-500MB per file (for Parquet).

### Trap 6: Redundant Processing

**Problem**: Multiple pipelines processing same data.

**Symptoms:**
- Same source ingested multiple times
- Same transformation computed multiple times
- Duplicate storage

**Solution**: Centralize, reuse outputs.

```sql
-- Instead of:
SELECT * FROM raw.events WHERE date = '2024-01-15'  -- Pipeline A
SELECT * FROM raw.events WHERE date = '2024-01-15'  -- Pipeline B

-- Do:
CREATE TABLE shared.events_2024_01_15 AS
SELECT * FROM raw.events WHERE date = '2024-01-15'

-- Both pipelines use shared table
```

## Optimization Strategies

### 1. Right-Size Compute

**Problem**: Over-provisioned compute (paying for unused capacity).

**Solution**: Match compute to workload.

**Batch jobs:**
```python
# Start small, scale up if needed
spark.conf.set("spark.executor.instances", "10")
spark.conf.set("spark.executor.cores", "4")

# Monitor and adjust
# If CPU < 50%: Reduce instances
# If CPU > 80%: Increase instances
```

**Streaming:**
```python
# Use auto-scaling
flink_config = {
    "parallelism": "auto",
    "min_parallelism": 2,
    "max_parallelism": 20
}
```

**Expected savings**: 20-30%.

### 2. Query Optimization

**Partition pruning:**
```sql
-- BAD: Scans all partitions
SELECT * FROM events
WHERE user_id = '123'

-- GOOD: Only scans relevant partition
SELECT * FROM events
WHERE date = '2024-01-15' AND user_id = '123'
```

**Column selection:**
```sql
-- BAD: Selects all columns
SELECT * FROM users

-- GOOD: Only needed columns
SELECT user_id, name, email FROM users
```

**Predicate pushdown:**
```sql
-- BAD: Filter after join
SELECT * FROM orders o
JOIN users u ON o.user_id = u.user_id
WHERE o.date = '2024-01-15'

-- GOOD: Filter before join
SELECT * FROM (
  SELECT * FROM orders WHERE date = '2024-01-15'
) o
JOIN users u ON o.user_id = u.user_id
```

**Expected savings**: 30-50% on query costs.

### 3. Caching and Materialization

**Materialized views:**
```sql
-- Pre-compute common aggregations
CREATE MATERIALIZED VIEW daily_user_stats AS
SELECT
  date,
  user_id,
  COUNT(*) as event_count,
  SUM(amount) as total_amount
FROM events
GROUP BY date, user_id

-- Refresh incrementally
REFRESH MATERIALIZED VIEW daily_user_stats;
```

**Application caching:**
```python
# Cache frequent queries
@cache(ttl=3600)  # 1 hour
def get_user_stats(user_id):
    return query(f"SELECT * FROM user_stats WHERE user_id = {user_id}")
```

**Expected savings**: 40-60% on repeated queries.

### 4. Compression

**Storage compression:**
- Parquet with Snappy: 2-3x
- Parquet with Zstd: 4-6x
- Parquet with Brotli: 5-7x (slower)

**Recommendation**: Use Zstd for best balance.

```python
df.write.parquet(
    path,
    compression="zstd"  # 4-6x compression
)
```

**Expected savings**: 50-70% on storage.

### 5. Incremental Processing

**Problem**: Reprocessing all data every time.

**Solution**: Only process new/changed data.

```python
# Full reprocess (expensive)
df = spark.read.table("raw.events")
processed = transform(df)
processed.write.save("curated.events")

# Incremental (cheap)
last_processed = get_last_processed_timestamp()
df = spark.read.table("raw.events") \
    .filter(f"ingestion_timestamp > '{last_processed}'")
processed = transform(df)
processed.write.mode("append").save("curated.events")
update_last_processed_timestamp()
```

**Expected savings**: 80-95% on transformation costs.

### 6. Spot Instances / Preemptible

**Use for:**
- Batch jobs (can tolerate interruption)
- Non-critical workloads
- Cost-sensitive use cases

**Avoid for:**
- Streaming (needs continuous availability)
- Critical pipelines
- Low-latency requirements

**Expected savings**: 60-80% on compute.

## Streaming vs Micro-Batch

### When to Use Streaming

**Use streaming when:**
- Real-time requirement (< 1 minute)
- Event-driven architecture
- Low-latency use cases (fraud, recommendations)

**Cost**: 3-5x batch

### When to Use Micro-Batch

**Use micro-batch when:**
- Near real-time acceptable (1-15 minutes)
- Cost-sensitive
- Can tolerate small delays

**Implementation:**
```python
# Micro-batch: Process every 5 minutes
schedule = "*/5 * * * *"  # Every 5 minutes

# Instead of continuous streaming
# Process accumulated events in batches
```

**Cost**: 1.5-2x batch

**Expected savings**: 50-70% vs streaming.

## Zombie Pipeline Detection

### The Problem

**Zombie pipelines**: Running but producing no value.

**Symptoms:**
- Zero downstream consumers
- No queries in 90+ days
- High cost, zero usage
- Still running "just in case"

### Detection

```sql
-- Find zombie pipelines
SELECT
  pipeline_name,
  daily_cost,
  last_consumer_query,
  days_since_last_use,
  CASE
    WHEN days_since_last_use > 90 THEN 'ZOMBIE'
    ELSE 'ACTIVE'
  END as status
FROM pipeline_usage_stats
WHERE status = 'ZOMBIE'
ORDER BY daily_cost DESC
```

### Action Plan

1. **Identify**: Find zombies (query above)
2. **Verify**: Confirm no usage (check consumers)
3. **Notify**: Alert owners
4. **Archive**: Move to cold storage
5. **Delete**: Remove if truly unused

**Expected savings**: 5-15% of total cost.

## Cost Monitoring

### Key Metrics

**Cost per GB ingested:**
```sql
SELECT
  source,
  SUM(ingestion_cost) / SUM(volume_gb) as cost_per_gb
FROM ingestion_costs
GROUP BY source
ORDER BY cost_per_gb DESC
```

**Cost per query:**
```sql
SELECT
  query_type,
  AVG(cost) as avg_cost,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY cost) as p95_cost
FROM query_costs
GROUP BY query_type
```

**Cost trends:**
- Week-over-week growth
- Month-over-month growth
- Anomaly detection (spikes)

### Cost Attribution

**By team:**
```sql
SELECT
  team,
  SUM(cost) as total_cost,
  SUM(cost) / SUM(SUM(cost)) OVER () as pct_of_total
FROM cost_attribution
GROUP BY team
ORDER BY total_cost DESC
```

**By source:**
```sql
SELECT
  source,
  SUM(cost) as total_cost
FROM cost_attribution
GROUP BY source
ORDER BY total_cost DESC
```

**By consumer:**
```sql
SELECT
  consumer,
  SUM(query_cost) as total_cost
FROM query_costs
GROUP BY consumer
ORDER BY total_cost DESC
```

### Alerting

**Cost alerts:**
- Daily cost > threshold
- Cost spike > 20% (day-over-day)
- Unusual usage pattern
- Budget exceeded

## Practical Cost Reduction Plan

### Phase 1: Quick Wins (Week 1-2)

1. **Identify unused tables** → Archive (10-20% savings)
2. **Convert JSON to Parquet** → Storage optimization (50-70% savings)
3. **Enable lifecycle policies** → Tier old data (50-70% savings)
4. **Compact small files** → Query optimization (20-30% savings)

**Expected total**: 20-30% reduction

### Phase 2: Optimization (Week 3-4)

1. **Right-size compute** → Match to workload (20-30% savings)
2. **Optimize queries** → Partition pruning, column selection (30-50% savings)
3. **Incremental processing** → Only process new data (80-95% savings)
4. **Materialize views** → Pre-compute aggregations (40-60% savings)

**Expected total**: Additional 15-25% reduction

### Phase 3: Architecture (Month 2+)

1. **Evaluate streaming vs batch** → Use batch when possible (50-70% savings)
2. **Eliminate zombies** → Remove unused pipelines (5-15% savings)
3. **Centralize processing** → Eliminate redundancy (10-20% savings)
4. **Spot instances** → For batch jobs (60-80% savings)

**Expected total**: Additional 10-20% reduction

### Overall Target

**Total expected savings**: 40-60% with all optimizations.

## Cost-Benefit Analysis

### When to Optimize

**Optimize when:**
- Cost > $10K/month (worth engineering time)
- Cost growing > 20%/month (unsustainable)
- Specific pain point (e.g., query slowness)

**Don't optimize when:**
- Cost < $1K/month (engineering time > savings)
- One-time spike (investigate, but don't over-optimize)
- Premature (optimize after you have data)

### ROI Calculation

```python
# Example: Query optimization project
engineering_time = 40  # hours
hourly_rate = 150  # $/hour
engineering_cost = engineering_time * hourly_rate  # $6,000

monthly_savings = 5000  # $/month
annual_savings = monthly_savings * 12  # $60,000

roi = (annual_savings - engineering_cost) / engineering_cost  # 900%
payback_period = engineering_cost / monthly_savings  # 1.2 months
```

**Rule of thumb**: If payback < 3 months, do it.

## Next Steps

- [Tooling Landscape](08-tooling-landscape.md) - Tools for cost optimization
- [Leadership View](10-leadership-view.md) - Measuring and reporting costs

