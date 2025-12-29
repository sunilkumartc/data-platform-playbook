# Push vs Pull

> Source-initiated vs platform-initiated ingestion patterns.

## Overview

Ingestion can be initiated by either the source system (push) or the platform (pull). Each approach has different trade-offs in terms of latency, complexity, and cost.

## Push (Source-Initiated)

**Architecture:**
```
Source System → Webhook/API → Platform Ingestion Endpoint
```

### Characteristics

- **Real-time delivery** - Source controls timing
- **Event-driven** - Data arrives as events happen
- **Lower latency** - No polling delay

### Pros

- ✅ Real-time delivery
- ✅ Source controls timing
- ✅ No polling overhead
- ✅ Event-driven architecture

### Cons

- ❌ Source must handle retries
- ❌ Platform must scale for bursts
- ❌ Requires source system changes
- ❌ More complex error handling

### When to Use

- Real-time requirements
- Source has reliable infrastructure
- You control the source system
- Event-driven architecture

### Implementation

**Webhook endpoint:**
```python
@app.post("/ingest/{source}")
async def ingest_webhook(source: str, data: dict):
    # Validate request
    if not validate_request(source, data):
        return {"error": "Invalid request"}, 400
    
    # Process data
    process_data(source, data)
    
    return {"status": "success"}
```

**Key considerations:**
- **Idempotency keys** - Deduplicate retries
- **Rate limiting** - Prevent abuse
- **Authentication** - Secure endpoints
- **Backpressure** - Reject when overloaded

## Pull (Platform-Initiated)

**Architecture:**
```
Platform Scheduler → Query Source → Process Results
```

### Characteristics

- **Scheduled execution** - Platform controls timing
- **Polling-based** - Query source periodically
- **Higher latency** - Depends on polling frequency

### Pros

- ✅ Platform controls rate
- ✅ Easier backpressure
- ✅ No source system changes
- ✅ Simpler error handling

### Cons

- ❌ Polling overhead
- ❌ May miss real-time events
- ❌ Higher latency
- ❌ May miss data if source unavailable

### When to Use

- Batch processing
- Source can't push
- Rate limiting needed
- Legacy systems

### Implementation

**Scheduled query:**
```python
@schedule.every(hours=1)
def pull_data():
    # Query source
    data = query_source("SELECT * FROM table WHERE updated_at > ?", last_pull_time)
    
    # Process data
    process_data(data)
    
    # Update last pull time
    update_last_pull_time()
```

**Optimization:**
- **Incremental queries** - Only fetch new data
- **Parallel pulls** - Multiple workers
- **Adaptive polling** - Increase frequency when data available

## Comparison

| Aspect | Push | Pull |
|-------|------|------|
| **Latency** | Low (real-time) | Higher (polling delay) |
| **Complexity** | Higher (source changes) | Lower (no source changes) |
| **Cost** | Higher (always-on) | Lower (scheduled) |
| **Reliability** | Depends on source | Platform-controlled |
| **Scalability** | Burst handling needed | Easier to scale |

## Hybrid Approach

**Use both patterns:**
- **Push** for real-time, critical data
- **Pull** for batch, non-critical data

**Example:**
```python
# Real-time events (push)
@app.post("/events")
async def ingest_events(data: dict):
    process_realtime_events(data)

# Batch data (pull)
@schedule.daily()
def pull_batch_data():
    process_batch_data()
```

## Decision Framework

**Use Push when:**
- ✅ Real-time requirement
- ✅ Source can push
- ✅ Event-driven architecture
- ✅ You control source system

**Use Pull when:**
- ✅ Batch acceptable
- ✅ Source can't push
- ✅ Rate limiting needed
- ✅ Legacy systems

## Best Practices

### Push Best Practices

1. **Idempotency** - Handle duplicate events
2. **Rate limiting** - Prevent abuse
3. **Authentication** - Secure endpoints
4. **Backpressure** - Reject when overloaded
5. **Retry logic** - Source should retry on failure

### Pull Best Practices

1. **Incremental queries** - Only fetch new data
2. **Checkpointing** - Track last processed record
3. **Error handling** - Retry on failures
4. **Adaptive polling** - Adjust frequency based on data availability
5. **Parallel processing** - Multiple workers for large sources

## Related Topics

- **[Batch vs Streaming](batch-vs-streaming.md)** - Ingestion patterns
- **[CDC](cdc.md)** - Change data capture
- **[Data Architecture](../data-architecture/index.md)** - Storage patterns

---

**Next**: [Data Architecture →](../data-architecture/index.md)

