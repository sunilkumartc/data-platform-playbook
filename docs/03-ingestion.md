# Ingestion Architecture

Ingestion is the foundation of your data platform. Get it wrong, and everything downstream suffers. This chapter provides deep, opinionated guidance on building reliable, cost-effective ingestion systems.

## Decision Framework

Before choosing an ingestion pattern, answer these questions:

1. **Freshness requirement**: Real-time (< 1 min), near real-time (1-15 min), or batch (15+ min)?
2. **Volume**: How many records/second? How many GB/day?
3. **Source type**: Database, API, files, event stream?
4. **Change detection**: Do you need to capture updates/deletes, or just new records?
5. **Cost sensitivity**: What's your budget per GB ingested?

## Batch vs Streaming vs CDC

### Batch Ingestion

**When to use:**
- Historical loads, backfills
- Large volumes (> 100 GB per run)
- No real-time requirement
- Source systems that don't support streaming

**Characteristics:**
- Scheduled execution (hourly, daily)
- Full or incremental extracts
- Higher latency (minutes to hours)
- Lower cost per GB
- Easier to debug and reprocess

**Common patterns:**

```python
# Full extract
SELECT * FROM source_table
WHERE ingestion_date = CURRENT_DATE

# Incremental extract (timestamp-based)
SELECT * FROM source_table
WHERE updated_at > :last_ingestion_time

# Incremental extract (change log)
SELECT * FROM source_table
WHERE id IN (
  SELECT id FROM change_log
  WHERE processed = FALSE
)
```

**Tools**: Airflow + Spark, Dataflow (batch), dbt, Fivetran

### Streaming Ingestion

**When to use:**
- Real-time analytics requirements
- Event-driven architectures
- Low-latency use cases (fraud detection, recommendations)
- High-volume, continuous data

**Characteristics:**
- Continuous processing
- Low latency (seconds to minutes)
- Higher cost per GB
- More complex failure handling
- Requires message queue/bus

**Architecture:**

```
Source → Message Queue (Kafka/Pub/Sub) → Stream Processor → Storage
```

**Tools**: Kafka, Pub/Sub, Kinesis, Flink, Dataflow (streaming), Kafka Connect

**Cost consideration**: Streaming is 3-5x more expensive than batch for the same volume. Only use when latency justifies cost.

### Change Data Capture (CDC)

**When to use:**
- Database replication
- Maintaining current state tables
- Audit trails
- Real-time synchronization

**Characteristics:**
- Captures inserts, updates, deletes
- Maintains transaction consistency
- Lower overhead than full extracts
- Requires source database support (WAL, binlog)

**CDC Patterns:**

1. **Log-based CDC**: Read database transaction logs
   - Tools: Debezium, Datastream, DMS
   - Pros: Low overhead, captures all changes
   - Cons: Requires database configuration

2. **Trigger-based CDC**: Database triggers write to change table
   - Pros: Works with any database
   - Cons: Higher overhead, may miss some changes

3. **Query-based CDC**: Poll for changes using timestamps/version columns
   - Pros: Simple, no database changes
   - Cons: May miss deletes, higher overhead

**Recommendation**: Use log-based CDC when available. It's the most reliable and efficient.

## Push vs Pull

### Push (Source-Initiated)

**Architecture:**
```
Source System → Webhook/API → Platform Ingestion Endpoint
```

**Pros:**
- Real-time delivery
- Source controls timing
- No polling overhead

**Cons:**
- Source must handle retries
- Platform must scale for bursts
- Requires source system changes

**When to use:**
- Real-time requirements
- Source has reliable infrastructure
- You control the source system

**Implementation considerations:**
- Idempotency keys (deduplicate retries)
- Rate limiting (prevent abuse)
- Authentication (secure endpoints)
- Backpressure (reject when overloaded)

### Pull (Platform-Initiated)

**Architecture:**
```
Platform Scheduler → Query Source → Process Results
```

**Pros:**
- Platform controls rate
- Easier backpressure
- No source system changes

**Cons:**
- Polling overhead
- May miss real-time events
- Higher latency

**When to use:**
- Batch processing
- Source can't push
- Rate limiting needed
- Legacy systems

**Optimization:**
- Incremental queries (only fetch new data)
- Parallel pulls (multiple workers)
- Adaptive polling (increase frequency when data available)

## Tool Selection Guide

### Ingestion Engines

| Tool | Type | Best For | Cost Model |
|------|------|----------|------------|
| **Airflow + Spark** | Batch | Large volumes, complex transforms | Compute + storage |
| **Dataflow** | Batch/Streaming | GCP-native, auto-scaling | Per vCPU-hour |
| **Fivetran** | SaaS | Database replication, zero maintenance | Per connector, per row |
| **Stitch** | SaaS | Simple extracts, cost-effective | Per row |
| **Debezium** | CDC | Kafka-based CDC, open source | Infrastructure only |
| **Datastream** | CDC | GCP-native CDC, managed | Per GB processed |
| **Kafka Connect** | Streaming | Kafka ecosystem, extensible | Infrastructure only |

### Decision Matrix

**High volume (> 1 TB/day), batch:**
→ Airflow + Spark or Dataflow (batch)

**Real-time, event streams:**
→ Kafka + Flink or Dataflow (streaming)

**Database replication, CDC:**
→ Debezium, Datastream, or Fivetran

**Multiple sources, zero maintenance:**
→ Fivetran or Stitch (SaaS)

**Cost-sensitive, simple extracts:**
→ Airflow + custom scripts

## Cost vs Freshness Trade-offs

**Rule of thumb**: Every 10x reduction in latency costs 3-5x more.

| Latency | Pattern | Cost per GB | Use Case |
|---------|---------|-------------|----------|
| < 1 min | Streaming | $0.10-0.50 | Real-time dashboards, fraud |
| 1-15 min | Micro-batch | $0.05-0.15 | Near real-time analytics |
| 15 min - 1 hr | Batch (frequent) | $0.02-0.05 | Hourly reports |
| 1-24 hrs | Batch (daily) | $0.01-0.02 | Daily ETL, data warehouse |
| > 24 hrs | Batch (weekly) | $0.005-0.01 | Historical analysis |

**Optimization strategy:**
1. Start with the slowest acceptable latency
2. Measure actual requirements (not perceived)
3. Optimize only when latency becomes a bottleneck
4. Use tiered approach: streaming for critical, batch for rest

## Common Patterns

### Pattern 1: Idempotent Ingestion

**Problem**: Source may send duplicate data (retries, failures).

**Solution**: Use idempotency keys.

```python
def ingest_record(record):
    idempotency_key = f"{source}_{record.id}_{record.timestamp}"
    
    if exists_in_dedupe_table(idempotency_key):
        return  # Already processed
    
    process_record(record)
    insert_dedupe_table(idempotency_key)
```

**Storage**: Use idempotency table with TTL (e.g., 7 days).

### Pattern 2: Checkpointing

**Problem**: Long-running jobs fail partway through.

**Solution**: Track progress, enable resume.

```python
checkpoint = get_last_checkpoint(job_id)
records = source.fetch_since(checkpoint.last_processed_id)

for record in records:
    process(record)
    checkpoint.update(record.id, record.timestamp)
    checkpoint.save()  # Frequent saves
```

**Storage**: Checkpoint table or file (S3, GCS).

### Pattern 3: Backpressure Handling

**Problem**: Source sends data faster than platform can process.

**Solution**: Implement backpressure.

```python
# Option 1: Rate limiting
rate_limiter = RateLimiter(max_per_second=1000)
for record in stream:
    rate_limiter.wait()
    process(record)

# Option 2: Queue with max size
queue = Queue(maxsize=10000)
if queue.full():
    reject_request()  # Return 503
else:
    queue.put(record)
```

### Pattern 4: Schema Evolution

**Problem**: Source schema changes break ingestion.

**Solution**: Schema registry + validation.

```python
schema_registry = SchemaRegistry()

def ingest(record):
    # Validate against latest schema
    schema = schema_registry.get_latest('source_name')
    validated = schema.validate(record)
    
    # Handle backward compatibility
    if not validated:
        handle_schema_mismatch(record, schema)
    
    store(validated)
```

**Tools**: Confluent Schema Registry, AWS Glue Schema Registry

## Error Handling

### Retry Strategy

**Exponential backoff**:
```python
max_retries = 5
base_delay = 1  # seconds

for attempt in range(max_retries):
    try:
        ingest(record)
        break
    except TransientError:
        if attempt < max_retries - 1:
            delay = base_delay * (2 ** attempt)
            sleep(delay)
        else:
            send_to_dlq(record)  # Dead letter queue
```

**Retryable errors**: Network timeouts, rate limits, temporary unavailability  
**Non-retryable errors**: Authentication failures, invalid schema, business logic errors

### Dead Letter Queue (DLQ)

**Purpose**: Store records that failed after all retries.

**Implementation**:
- Separate storage (S3, BigQuery table)
- Alert on DLQ size
- Manual review and reprocessing

**Monitoring**: Track DLQ size, error types, reprocessing success rate.

## Monitoring & Observability

### Key Metrics

**Volume metrics:**
- Records/second
- GB/day
- Partition count

**Latency metrics:**
- End-to-end latency (source → storage)
- Processing time per record
- Queue depth

**Quality metrics:**
- Schema validation failures
- Duplicate rate
- Missing data rate

**Reliability metrics:**
- Success rate
- Error rate by type
- DLQ size
- Retry count

### Alerting

**Critical alerts:**
- Ingestion stopped (zero records in last N minutes)
- Error rate > threshold (e.g., 5%)
- Latency > SLA (e.g., 15 minutes for batch)
- DLQ size > threshold

**Warning alerts:**
- Volume drop > 20%
- Schema drift detected
- Cost spike > 20%

## Cost Optimization

### Common Cost Traps

1. **Over-ingestion**: Ingesting data that's never used
   - **Solution**: Track usage, archive unused sources

2. **Inefficient formats**: Using JSON instead of Parquet
   - **Solution**: Convert to columnar formats post-ingestion

3. **Redundant ingestion**: Multiple pipelines for same source
   - **Solution**: Centralize, reuse outputs

4. **Streaming when batch would suffice**: 3-5x cost premium
   - **Solution**: Measure actual latency requirements

### Optimization Techniques

1. **Compression**: Use Snappy or Zstd (2-5x reduction)
2. **Partitioning**: Only process new partitions
3. **Incremental loads**: Only fetch changed data
4. **Batching**: Group small records into batches
5. **Lifecycle policies**: Move old data to cheaper storage

**Expected savings**: 20-40% with basic optimizations.

## Next Steps

- [Storage & Data Architecture](04-storage.md) - How to store ingested data
- [Cost Efficiency & Scale](07-cost-efficiency.md) - Advanced cost optimization

