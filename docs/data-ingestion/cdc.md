# Change Data Capture (CDC)

> Capturing database changes in real-time to maintain synchronized data across systems.

## Overview

Change Data Capture (CDC) is a pattern for capturing and propagating changes made to a database. Instead of periodically querying for changes, CDC reads the database transaction log to capture inserts, updates, and deletes as they happen.

## Why CDC?

### Problems with Traditional Approaches

**Full Extract:**
- ❌ Inefficient (transfers all data, even unchanged)
- ❌ High source system load
- ❌ Slow for large tables

**Timestamp-based Incremental:**
- ❌ Misses deletes
- ❌ May miss updates if timestamp not updated
- ❌ Requires source system changes

**CDC Solution:**
- ✅ Captures all changes (inserts, updates, deletes)
- ✅ Low overhead (reads transaction log)
- ✅ Real-time or near real-time
- ✅ Maintains transaction consistency

## CDC Patterns

### 1. Log-Based CDC

**How it works:**
Reads database transaction logs (WAL, binlog, redo logs) to capture changes.

**Architecture:**
```
Database → Transaction Log → CDC Tool → Message Queue → Storage
```

**Pros:**
- ✅ Low overhead (doesn't query tables)
- ✅ Captures all changes
- ✅ Maintains transaction consistency
- ✅ Real-time

**Cons:**
- ❌ Requires database configuration
- ❌ Database-specific (different for each DB)

**Tools:**
- Debezium (Kafka Connect)
- AWS DMS (Database Migration Service)
- Google Datastream
- Striim

### 2. Trigger-Based CDC

**How it works:**
Database triggers write changes to a change table.

**Architecture:**
```
Database → Trigger → Change Table → CDC Tool → Storage
```

**Pros:**
- ✅ Works with any database
- ✅ Captures all changes
- ✅ No external tools needed

**Cons:**
- ❌ Higher overhead (triggers on every change)
- ❌ Requires database changes
- ❌ May impact source system performance

### 3. Query-Based CDC

**How it works:**
Poll for changes using timestamps or version columns.

**Architecture:**
```
CDC Tool → Query Database → Process Changes → Storage
```

**Pros:**
- ✅ Simple, no database changes
- ✅ Works with any database
- ✅ Easy to implement

**Cons:**
- ❌ May miss deletes
- ❌ Polling overhead
- ❌ Higher latency
- ❌ May miss updates if timestamp not updated

**When to use:**
- Legacy systems
- Can't modify database
- Acceptable to miss some changes

## CDC Tools

### Debezium

**Best for:** Kafka-based CDC, open source

**Pros:**
- ✅ Open source
- ✅ Kafka-native
- ✅ Many database connectors
- ✅ Reliable

**Cons:**
- ❌ Requires Kafka infrastructure
- ❌ Self-managed

**Use when:**
- Already using Kafka
- Need open source solution
- Have operations team

### Google Datastream

**Best for:** GCP-native CDC, managed service

**Pros:**
- ✅ Managed service (no ops)
- ✅ GCP-integrated
- ✅ Reliable
- ✅ Easy setup

**Cons:**
- ❌ GCP-only
- ❌ Expensive
- ❌ Limited database support

**Use when:**
- GCP stack
- Want managed service
- Cost acceptable

### AWS DMS

**Best for:** AWS-native replication, migrations

**Pros:**
- ✅ Managed service
- ✅ AWS-integrated
- ✅ Good for migrations

**Cons:**
- ❌ AWS-only
- ❌ Can be expensive
- ❌ Complex configuration

**Use when:**
- AWS stack
- Need managed service
- Database migrations

## CDC + Current State Patterns

### Problem

CDC streams capture changes, but analytics often needs **current state** (latest value per key).

### Solution 1: Merge Pattern

**Merge CDC events into current state table:**

```sql
MERGE INTO current_state AS target
USING cdc_events AS source
ON target.id = source.id
WHEN MATCHED AND source.op = 'UPDATE' THEN
  UPDATE SET col1 = source.col1, updated_at = source.timestamp
WHEN MATCHED AND source.op = 'DELETE' THEN
  DELETE
WHEN NOT MATCHED AND source.op = 'INSERT' THEN
  INSERT (id, col1, updated_at) VALUES (source.id, source.col1, source.timestamp)
```

**Tools:** Spark, Flink, BigQuery MERGE

### Solution 2: Snapshot + Incremental

**Periodic snapshots + incremental updates:**

```sql
-- Daily snapshot
CREATE TABLE current_state_2024_01_15 AS
SELECT * FROM current_state_2024_01_14
UNION ALL
SELECT * FROM cdc_events
WHERE date = '2024-01-15'
```

**Pros:** Simple, easy to reprocess  
**Cons:** Storage overhead, slower queries

### Solution 3: Event Sourcing

**Store all events, compute current state on read:**

```sql
-- Current state = latest event per key
SELECT DISTINCT ON (id) *
FROM events
ORDER BY id, timestamp DESC
```

**Pros:** Full history, audit trail  
**Cons:** Expensive queries, complex logic

!!! tip "Recommendation"
    Use merge pattern for most cases. It's efficient and maintains current state.

## Implementation Best Practices

### 1. Handle Transaction Boundaries

CDC should maintain transaction consistency. Process all changes in a transaction together.

### 2. Handle Schema Changes

Database schema changes can break CDC. Use schema registry to handle evolution.

### 3. Handle Failures

CDC must be resilient to failures:
- Checkpoint progress
- Handle duplicate events (idempotency)
- Retry on failures

### 4. Monitor Lag

Track CDC lag (time between change and processing):
- Alert if lag > threshold
- Monitor queue depth
- Track processing rate

## Common Challenges

### Challenge 1: High-Volume Changes

**Problem:** Database with millions of changes per day.

**Solution:**
- Partition by table/date
- Parallel processing
- Batch processing (micro-batch)

### Challenge 2: Schema Evolution

**Problem:** Database schema changes break CDC.

**Solution:**
- Schema registry
- Backward-compatible changes
- Versioned schemas

### Challenge 3: Transaction Consistency

**Problem:** Need to maintain transaction boundaries.

**Solution:**
- Use transaction-aware CDC tools
- Process transactions atomically
- Handle partial transactions

## Cost Considerations

**CDC costs:**
- Infrastructure (Kafka, compute)
- Storage (change events)
- Processing (transformation)

**Optimization:**
- Filter unnecessary changes
- Compress change events
- Archive old changes

## Related Topics

- **[Batch vs Streaming](batch-vs-streaming.md)** - Other ingestion patterns
- **[Data Architecture](../data-architecture/index.md)** - How to store CDC data
- **[Data Quality](../data-quality/index.md)** - Ensuring CDC data quality

---

**Next**: [Push vs Pull →](push-vs-pull.md)

