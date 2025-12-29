# Data Quality Checks

> Implementing automated quality checks and testing.

## Overview

Data quality checks validate that data meets expectations. They should be automated, run continuously, and alert when issues are detected.

## Quality Dimensions

### 1. Completeness

**Check:** Are all expected records present?

**Example:**
```python
# Check record count
expected_count = 10000
actual_count = df.count()

if actual_count < expected_count * 0.99:
    raise QualityCheckFailed("Record count below threshold")
```

### 2. Accuracy

**Check:** Does data reflect reality?

**Example:**
```python
# Check business rules
if df.filter(df.order_amount < 0).count() > 0:
    raise QualityCheckFailed("Negative order amounts found")
```

### 3. Consistency

**Check:** Is data consistent across sources?

**Example:**
```python
# Compare across sources
source1_sum = df1.select(sum("amount")).first()[0]
source2_sum = df2.select(sum("amount")).first()[0]

if abs(source1_sum - source2_sum) > threshold:
    raise QualityCheckFailed("Inconsistent totals")
```

### 4. Timeliness (Freshness)

**Check:** Is data fresh enough?

**Example:**
```python
# Check data age
max_age = df.select(max("ingestion_timestamp")).first()[0]
current_time = datetime.now()

if (current_time - max_age).total_seconds() > 3600:
    raise QualityCheckFailed("Data is stale")
```

### 5. Validity

**Check:** Does data conform to schema?

**Example:**
```python
# Schema validation
schema = StructType([
    StructField("user_id", StringType(), False),
    StructField("email", StringType(), False)
])

try:
    df = spark.createDataFrame(data, schema)
except Exception as e:
    raise QualityCheckFailed(f"Schema validation failed: {e}")
```

### 6. Uniqueness

**Check:** Are there duplicates?

**Example:**
```python
# Check for duplicates
duplicate_count = df.groupBy("id").count().filter("count > 1").count()

if duplicate_count > 0:
    raise QualityCheckFailed(f"Found {duplicate_count} duplicate IDs")
```

## Testing Frameworks

### Great Expectations

**Best for:** Comprehensive quality testing

**Example:**
```python
import great_expectations as ge

df = ge.read_csv("data.csv")

# Expectation: No null values
df.expect_column_values_to_not_be_null("user_id")

# Expectation: Values in range
df.expect_column_values_to_be_between("age", 0, 120)

# Expectation: Unique values
df.expect_column_values_to_be_unique("user_id")

# Validate
results = df.validate()
```

### dbt Tests

**Best for:** SQL-based quality checks

**Example:**
```sql
-- models/schema.yml
models:
  - name: users
    columns:
      - name: user_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - unique
          - not_null
          - accepted_values:
              values: ['@company.com']
```

### Custom Validators

**Best for:** Business-specific rules

**Example:**
```python
def validate_order_data(df):
    checks = [
        check_completeness(df),
        check_accuracy(df),
        check_uniqueness(df)
    ]
    
    failures = [c for c in checks if not c.passed]
    if failures:
        raise QualityCheckFailed(failures)
```

## Automated Checks

### In CI/CD

**Run tests before deployment:**
```yaml
# .github/workflows/quality.yml
- name: Run quality tests
  run: |
    dbt test
    pytest tests/quality/
```

### In Pipelines

**Check as pipeline stage:**
```python
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

## Quality Scores

### Composite Score

**Calculate overall quality:**
```python
quality_score = (
    completeness_score * 0.3 +
    accuracy_score * 0.3 +
    freshness_score * 0.2 +
    consistency_score * 0.2
)
```

### Tracking

**Monitor over time:**
- Quality trends
- Degradation detection
- Improvement tracking

## Best Practices

1. **Automate** - Don't rely on manual checks
2. **Fail fast** - Catch issues early
3. **Alert** - Notify when quality drops
4. **Document** - Document all checks
5. **Review** - Regularly review and update checks

## Related Topics

- **[Governance](governance.md)** - Quality governance framework
- **[Data Engineering](../data-engineering/index.md)** - Platform fundamentals

---

**Next**: [Data Engineering →](../data-engineering/index.md)

