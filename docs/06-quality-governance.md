# Quality, Governance & Observability

Data quality and governance aren't nice-to-haves—they're prerequisites for trust. Without them, your platform becomes a liability, not an asset. This chapter covers how to build quality into your platform and govern data effectively.

## Data Quality Framework

### Quality Dimensions

**1. Completeness**
- Are all expected records present?
- Metrics: Record count, null rate, missing partitions

**2. Accuracy**
- Does data reflect reality?
- Metrics: Validation failures, business rule violations

**3. Consistency**
- Is data consistent across sources?
- Metrics: Cross-source comparisons, duplicate rates

**4. Timeliness (Freshness)**
- Is data fresh enough?
- Metrics: Data age, SLA compliance

**5. Validity**
- Does data conform to schema?
- Metrics: Schema validation failures, type mismatches

**6. Uniqueness**
- Are there duplicates?
- Metrics: Duplicate count, primary key violations

### Quality Checks

**At ingestion:**
```python
# Schema validation
schema = get_contract_schema(source)
if not schema.validate(record):
    reject_with_error(record, "Schema violation")

# Completeness check
if record_count < expected_min:
    alert("Low record count")

# Uniqueness check
if duplicate_count > threshold:
    alert("High duplicate rate")
```

**Post-transformation:**
```python
# Business rule validation
if order_amount < 0:
    flag_anomaly("Negative order amount")

# Referential integrity
if user_id not in users_table:
    flag_anomaly("Orphaned record")

# Statistical checks
if current_avg > historical_avg * 2:
    flag_anomaly("Unusual spike")
```

**Tools**: Great Expectations, dbt tests, custom validators

## SLAs and Freshness

### Defining SLAs

**SLA components:**
- **Freshness**: Maximum acceptable data age
- **Availability**: Uptime target (e.g., 99.9%)
- **Quality**: Minimum quality thresholds
- **Latency**: End-to-end processing time

**Example SLA:**
```yaml
source: user_events
sla:
  freshness: 15 minutes  # Data must be < 15 min old
  availability: 99.9%    # Available 99.9% of time
  quality:
    completeness: > 99%
    accuracy: > 99.5%
  latency:
    p50: < 5 minutes
    p95: < 15 minutes
    p99: < 30 minutes
```

### Freshness Monitoring

**Track data age:**
```sql
-- Example: Check freshness
SELECT
  source,
  MAX(ingestion_timestamp) as last_ingestion,
  CURRENT_TIMESTAMP - MAX(ingestion_timestamp) as age,
  CASE
    WHEN age > INTERVAL '15 minutes' THEN 'VIOLATED'
    ELSE 'OK'
  END as status
FROM raw.events
GROUP BY source
```

**Alerting:**
- Alert when freshness > SLA
- Alert on trends (getting slower)
- Alert on complete stops

**Dashboards:**
- Freshness by source (real-time)
- SLA compliance over time
- Violation trends

## Schema Enforcement

### Schema Evolution

**Backward compatibility rules:**
- ✅ Add optional fields
- ✅ Make required fields optional
- ❌ Remove fields (without deprecation period)
- ❌ Change field types (without migration)

**Versioning strategy:**
```yaml
schema:
  version: 2.0
  changes_from_1.0:
    - Added: new_field (optional)
    - Deprecated: old_field (remove in 3.0)
    - Changed: field_type (with migration path)
```

**Migration process:**
1. Deploy new schema version
2. Support both versions (dual-write)
3. Migrate consumers to new version
4. Deprecate old version
5. Remove old version

### Schema Registry

**Purpose**: Centralized schema management

**Features:**
- Schema storage and versioning
- Compatibility checking
- Client libraries (auto-validation)

**Tools**: Confluent Schema Registry, AWS Glue Schema Registry, custom

**Usage:**
```python
# Register schema
schema_registry.register(
    subject="user_events",
    schema=user_events_schema,
    compatibility="BACKWARD"
)

# Validate on ingestion
schema = schema_registry.get_latest("user_events")
if not schema.validate(record):
    reject("Schema violation")
```

## Metadata and Lineage

### Metadata Types

**Technical metadata:**
- Schema, data types, partitions
- Storage location, format
- Ingestion timestamps, versions

**Business metadata:**
- Description, purpose
- Owner, contact
- Business glossary terms
- Data classification (PII, sensitive)

**Operational metadata:**
- Freshness, quality metrics
- Usage statistics
- Cost attribution
- Dependencies

### Data Catalog

**Purpose**: Discoverable, searchable metadata

**Features:**
- Search by name, description, tags
- Browse by domain, owner
- View schema, samples, statistics
- See lineage, usage

**Tools**: DataHub, Collibra, AWS Glue Catalog, custom

**Example entry:**
```yaml
name: user_events
description: User interaction events (clicks, views, purchases)
owner: analytics-team@company.com
domain: customer
tags: [events, user-behavior, pii]
schema:
  - name: user_id
    type: string
    description: Unique user identifier
  - name: event_type
    type: string
    enum: [click, view, purchase]
lineage:
  sources: [web_app, mobile_app]
  consumers: [analytics_dashboards, ml_models]
```

### Lineage Tracking

**Purpose**: Understand data flow and dependencies

**Types:**
- **Upstream**: Where data comes from
- **Downstream**: Who uses this data
- **Transformation**: How data is transformed

**Implementation:**
```python
# Track lineage automatically
@track_lineage(
    inputs=["raw.events"],
    outputs=["curated.user_events"],
    transformation="filter_and_aggregate"
)
def transform_events():
    ...
```

**Use cases:**
- Impact analysis (what breaks if source changes?)
- Root cause analysis (where did bad data come from?)
- Compliance (data flow documentation)

**Visualization:**
```
raw.events → transform → curated.user_events → dashboard
                ↓
            ml_features → model
```

## Ownership and Accountability

### Data Ownership Model

**Owner responsibilities:**
- Define and maintain contracts
- Ensure quality and freshness
- Respond to issues
- Approve schema changes
- Optimize costs

**Assignment:**
- By domain (e.g., analytics team owns analytics data)
- By source system (e.g., payments team owns payment data)
- Explicit assignment in catalog

**Escalation:**
- Owner unresponsive → manager
- Critical issue → on-call rotation

### Access Control

**Principle of least privilege:**
- Users get minimum access needed
- Role-based access control (RBAC)
- Row-level security for sensitive data

**Implementation:**
```sql
-- Example: Row-level security
CREATE POLICY user_data_policy ON user_events
FOR SELECT
USING (user_id = current_user_id() OR is_admin());

-- Column masking
CREATE POLICY mask_pii ON users
FOR SELECT
USING (
  CASE
    WHEN has_pii_access() THEN email
    ELSE '***'
  END
);
```

**Tools**: BigQuery row-level security, Snowflake dynamic masking, custom

## Observability

### Metrics

**Pipeline metrics:**
- Volume (records/second, GB/day)
- Latency (end-to-end, per stage)
- Success/failure rates
- Error types and counts

**Data metrics:**
- Freshness (data age)
- Quality scores (by dimension)
- Schema drift
- Duplicate rates

**Infrastructure metrics:**
- Resource utilization (CPU, memory, storage)
- Queue depths
- Cache hit rates
- Network throughput

**Business metrics:**
- Cost per GB, per query
- User satisfaction
- Time to value

### Logging

**Structured logging:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "pipeline": "user_events_ingestion",
  "stage": "ingestion",
  "records_processed": 10000,
  "duration_ms": 5000,
  "status": "success"
}
```

**Log levels:**
- **DEBUG**: Detailed execution info
- **INFO**: Normal operations
- **WARN**: Potential issues
- **ERROR**: Failures that don't stop pipeline
- **CRITICAL**: Failures that stop pipeline

**Retention**: 30-90 days for operational logs

### Alerting

**Alert categories:**

**Critical (page on-call):**
- Pipeline stopped (zero records)
- SLA violation (freshness, availability)
- Data quality breach (completeness < threshold)

**Warning (notify team):**
- Degradation (latency increasing, quality dropping)
- Cost anomaly (spike > 20%)
- Schema drift detected

**Info (dashboard only):**
- Normal operations
- Scheduled maintenance
- Capacity planning

**Alert design:**
- **Actionable**: Clear what to do
- **Specific**: Not "something is wrong"
- **Rare**: Only alert on real issues
- **Grouped**: Related alerts together

**Example:**
```
❌ BAD: "Pipeline error"
✅ GOOD: "user_events ingestion failed: Schema validation error on field 'user_id' (expected string, got int). Last successful: 10:15 AM. Owner: analytics-team."
```

### Dashboards

**Operational dashboard:**
- Pipeline health (all pipelines)
- Freshness by source
- Quality scores
- Error rates
- Cost trends

**Team dashboard:**
- My pipelines (owned by team)
- My data (sources and consumers)
- My costs
- My SLAs

**Executive dashboard:**
- Platform health (high-level)
- Total cost and trends
- Adoption metrics
- SLA compliance

**Tools**: Grafana, Datadog, custom dashboards

## Root Cause Analysis (RCA)

### Process

**1. Detect issue:**
- Alert fires or user reports

**2. Gather context:**
- When did it start?
- What changed recently?
- What's the scope?

**3. Trace lineage:**
- Where did bad data come from?
- What transformations touched it?
- Who consumes it?

**4. Identify root cause:**
- Source system change?
- Schema drift?
- Transformation bug?
- Infrastructure issue?

**5. Fix and prevent:**
- Immediate fix
- Long-term prevention
- Update monitoring

### RCA Template

```markdown
## Incident: [Title]

**Time**: [When]
**Impact**: [What broke, who affected]
**Duration**: [How long]

**Timeline**:
- 10:00 AM: Alert fired
- 10:05 AM: Investigation started
- 10:15 AM: Root cause identified
- 10:30 AM: Fix deployed
- 10:35 AM: Verified resolution

**Root Cause**:
[What actually caused it]

**Fix**:
[What we did]

**Prevention**:
[How we'll prevent it]
```

## Quality Automation

### Automated Quality Checks

**In CI/CD:**
```yaml
# Example: dbt tests in CI
- name: Run data quality tests
  run: dbt test
  on:
    schedule: daily
    on_push: true
```

**In pipelines:**
```python
# Quality checks as pipeline stage
def quality_check_stage(df):
    checks = [
        completeness_check(df),
        uniqueness_check(df),
        business_rule_check(df)
    ]
    
    if any(check.failed for check in checks):
        send_to_quarantine(df)
        alert_owner()
    
    return df
```

### Quality Scores

**Composite score:**
```python
quality_score = (
    completeness_score * 0.3 +
    accuracy_score * 0.3 +
    freshness_score * 0.2 +
    consistency_score * 0.2
)
```

**Track over time:**
- Quality trends
- Degradation detection
- Improvement tracking

## Compliance and Privacy

### Data Classification

**Categories:**
- **Public**: No restrictions
- **Internal**: Company use only
- **Confidential**: Restricted access
- **PII**: Personally identifiable information
- **Sensitive**: Financial, health data

**Tagging:**
```yaml
data_classification: PII
privacy_level: high
retention_days: 365
access_restrictions: [encryption, masking, audit_logging]
```

### GDPR / Privacy Compliance

**Requirements:**
- Right to access
- Right to deletion
- Data minimization
- Consent tracking

**Implementation:**
- Tag PII data
- Automated deletion (retention policies)
- Access logging
- Consent management

### Audit Logging

**Log all access:**
- Who accessed what data
- When
- Why (query, purpose)
- Result (rows returned)

**Retention**: 7 years for compliance

**Tools**: Cloud audit logs, custom logging

## Next Steps

- [Cost Efficiency & Scale](07-cost-efficiency.md) - Optimize costs while maintaining quality
- [Tooling Landscape](08-tooling-landscape.md) - Tools for quality and governance

