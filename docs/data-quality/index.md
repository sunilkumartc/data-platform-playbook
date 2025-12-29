# Data Quality

> "Observability is just empathy for future engineers."

> Ensuring data is reliable, accurate, and trustworthy.

> "Pipelines fail quietly. People fail when no one explains why they exist."

## Overview

Data quality isn't a nice-to-have—it's a prerequisite for trust. Without quality, your platform becomes a liability, not an asset.

## Key Topics

### [Governance](governance.md)
Data governance, SLAs, schema enforcement, observability.

**Learn about:**
- SLAs and freshness
- Schema enforcement
- Metadata and lineage
- Ownership and accountability

### [Checks](checks.md)
Data quality checks and testing.

**Learn about:**
- Quality dimensions
- Testing frameworks
- Automated checks
- Quality scores

## Quality Dimensions

1. **Completeness** - Are all expected records present?
2. **Accuracy** - Does data reflect reality?
3. **Consistency** - Is data consistent across sources?
4. **Timeliness** - Is data fresh enough?
5. **Validity** - Does data conform to schema?
6. **Uniqueness** - Are there duplicates?

## Best Practices

### At Ingestion
- Schema validation
- Completeness checks
- Uniqueness checks

### Post-Transformation
- Business rule validation
- Referential integrity
- Statistical checks

### Continuous Monitoring
- Quality scores
- Anomaly detection
- Alerting

## Related Topics

- **[Data Engineering](../data-engineering/index.md)** - Platform fundamentals
- **[Data Architecture](../data-architecture/index.md)** - Storage patterns

---

**Next**: [Governance →](governance.md)

