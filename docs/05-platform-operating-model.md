# Platform & Operating Model

Building a data platform isn't just about technology—it's about creating an operating model that enables teams to move fast while maintaining quality, cost control, and reliability. This chapter covers how to structure your platform organization and processes.

## Central Platform vs Domain Ownership

### The Spectrum

```
Fully Centralized ←──────────────────────────────→ Fully Decentralized
(Platform Team)                                    (Domain Teams)
```

### Central Platform Model

**Structure:**
- Central platform team owns infrastructure
- Domain teams consume platform services
- Platform team builds self-serve capabilities

**Pros:**
- Consistency across organization
- Economies of scale
- Centralized expertise
- Easier governance

**Cons:**
- Can become bottleneck
- May not understand domain needs
- Slower to adapt

**Best for:**
- Large organizations (1000+ engineers)
- Need for strong governance
- Limited data engineering expertise in domains

### Domain Ownership Model

**Structure:**
- Domain teams own their data end-to-end
- Platform provides base infrastructure only
- Teams responsible for quality, cost, SLAs

**Pros:**
- Faster iteration
- Domain expertise
- Ownership and accountability

**Cons:**
- Inconsistency
- Duplication
- Harder governance

**Best for:**
- Smaller organizations
- High domain expertise
- Need for speed over consistency

### Hybrid Model (Recommended)

**Structure:**
- Platform team owns: Infrastructure, standards, tooling
- Domain teams own: Business logic, transformations, quality
- Shared ownership: Governance, cost optimization

**Responsibilities Matrix:**

| Area | Platform Team | Domain Teams | Shared |
|------|---------------|--------------|--------|
| Infrastructure | ✅ | | |
| Ingestion pipelines | ✅ (self-serve) | ✅ (business logic) | |
| Transformations | | ✅ | |
| Data quality | | ✅ | ✅ (standards) |
| Cost optimization | ✅ (tools) | ✅ (usage) | ✅ |
| Governance | ✅ (framework) | ✅ (compliance) | ✅ |

**Key principle**: Platform enables, domains execute.

## Paved Paths and Escape Hatches

### Paved Paths

**Definition**: Standardized, supported, well-documented ways to accomplish common tasks.

**Examples:**
- Standard ingestion patterns (CDC, batch, streaming)
- Pre-configured compute environments (Spark, Flink)
- Standard storage formats (Parquet, Delta)
- Approved tooling (dbt, Airflow)

**Benefits:**
- Faster onboarding
- Consistency
- Easier maintenance
- Better observability

**Implementation:**
```yaml
# Example: Standard ingestion template
ingestion_template:
  type: cdc
  source: postgres
  destination: gcs://raw/{source_name}
  format: parquet
  partition_by: [date]
  schema_registry: enabled
  monitoring: enabled
```

### Escape Hatches

**Definition**: Approved ways to deviate from paved paths when needed.

**When to use:**
- Unique requirements not met by standard paths
- Performance optimization
- Experimental patterns

**Process:**
1. Document why standard path doesn't work
2. Get approval (platform team review)
3. Implement with monitoring
4. Evaluate for promotion to paved path

**Example:**
```
Standard: Use Dataflow for streaming
Escape hatch: Use Flink for stateful processing (approved use case)
```

**Principle**: Make it easy to use paved paths, possible but reviewed to use escape hatches.

## Contract-First Ingestion

### The Problem

Without contracts, you get:
- Schema drift breaking downstream
- Unclear SLAs
- Ownership confusion
- Cost attribution issues

### The Solution: Data Contracts

**Contract definition:**
```yaml
source: user_events
owner: analytics-team@company.com
sla:
  freshness: 15 minutes
  availability: 99.9%
schema:
  version: 1.0
  fields:
    - name: user_id
      type: string
      required: true
    - name: event_type
      type: string
      enum: [click, view, purchase]
  evolution: backward_compatible
quality:
  completeness: > 99%
  uniqueness: > 99.9%
cost_attribution: analytics-team
```

### Contract Enforcement

**At ingestion:**
1. Validate schema matches contract
2. Check quality metrics
3. Reject if contract violated

**In platform:**
1. Store contracts in registry
2. Version contracts
3. Notify on violations
4. Track compliance

**Tools**: DataHub, Great Expectations, custom validators

### Benefits

- **Predictability**: Downstream knows what to expect
- **Quality**: Issues caught early
- **Ownership**: Clear accountability
- **Evolution**: Controlled schema changes

## Cost Attribution and Accountability

### The Problem

Without attribution:
- "The platform is expensive" (but who's using it?)
- No incentive to optimize
- Hard to justify investments

### Solution: Cost Attribution

**Attribution dimensions:**
- **Team**: Which team owns the data/pipeline
- **Project**: Which project/business unit
- **Source**: Which source system
- **Consumer**: Which downstream consumers

**Implementation:**

```sql
-- Example: Cost attribution query
SELECT
  team,
  source,
  SUM(storage_cost) as storage_cost,
  SUM(compute_cost) as compute_cost,
  SUM(total_cost) as total_cost
FROM cost_attribution
WHERE date >= CURRENT_DATE - 30
GROUP BY team, source
ORDER BY total_cost DESC
```

**Tools**: 
- Cloud cost management (AWS Cost Explorer, GCP Billing)
- Custom attribution tags
- DataHub cost tracking

### Showback vs Chargeback

**Showback** (recommended):
- Show costs to teams
- Create awareness
- Encourage optimization
- No actual billing

**Chargeback**:
- Actually bill teams
- Stronger incentive
- More complex (billing systems)
- Can create friction

**Recommendation**: Start with showback. Move to chargeback only if needed.

### Cost Accountability

**Monthly reviews:**
1. Top spenders by team
2. Cost trends (growth, anomalies)
3. Optimization opportunities
4. ROI of investments

**Goals:**
- Teams see their costs
- Teams understand cost drivers
- Teams optimize proactively

## Self-Serve Capabilities

### Ingestion Self-Serve

**Capabilities:**
- Web UI or CLI to register new sources
- Automatic pipeline generation
- Schema discovery and validation
- Monitoring setup

**Example flow:**
```bash
# Developer registers new source
platform ingest register \
  --source postgres://db.example.com/users \
  --destination gcs://raw/users \
  --sla 15min \
  --owner analytics-team

# Platform automatically:
# - Creates CDC pipeline
# - Sets up monitoring
# - Creates contract
# - Provisions resources
```

**Benefits:**
- Faster time to value (hours vs weeks)
- Reduced platform team load
- Consistency (standard patterns)

### Transformation Self-Serve

**Capabilities:**
- Managed compute (Spark, Flink clusters)
- Standard libraries and frameworks
- CI/CD integration
- Testing frameworks

**Example:**
```python
# Developer writes transformation
@platform.transform(
    input="raw.events",
    output="curated.user_events",
    schedule="hourly"
)
def transform_events(df):
    return df.filter(df.event_type == "purchase")
```

**Platform handles:**
- Resource provisioning
- Scheduling
- Monitoring
- Error handling

### Discovery Self-Serve

**Capabilities:**
- Data catalog (search, browse)
- Schema documentation
- Lineage visualization
- Usage statistics

**Tools**: DataHub, Collibra, custom catalogs

## Platform Team Structure

### Core Team Roles

**Platform Engineers:**
- Build and maintain infrastructure
- Develop self-serve capabilities
- Optimize platform performance

**Data Engineers (Platform):**
- Design ingestion patterns
- Build transformation frameworks
- Create best practices

**SRE / DevOps:**
- Reliability and observability
- Incident response
- Capacity planning

**Product Managers:**
- Platform roadmap
- User needs (domain teams)
- Success metrics

### Team Size Guidelines

**Small organization (< 100 engineers):**
- 2-3 platform engineers
- Part-time SRE
- No dedicated PM

**Medium organization (100-500 engineers):**
- 5-10 platform engineers
- 1-2 SRE
- 1 PM

**Large organization (500+ engineers):**
- 15-30 platform engineers
- 3-5 SRE
- 2-3 PM
- Dedicated cost optimization team

## Success Metrics

### Platform Health

**Adoption:**
- % of data sources using platform
- % of transformations on platform
- Active users per month

**Reliability:**
- Platform uptime (target: 99.9%)
- Pipeline success rate (target: > 99%)
- Mean time to recovery (MTTR)

**Performance:**
- Ingestion latency (p50, p95, p99)
- Query performance (p50, p95, p99)
- Resource utilization

### Developer Experience

**Time to value:**
- Time to first ingestion (target: < 1 day)
- Time to first transformation (target: < 2 days)

**Developer satisfaction:**
- NPS or survey scores
- Support ticket volume
- Documentation usage

**Self-serve adoption:**
- % of pipelines created via self-serve
- % of transformations using standard frameworks

### Cost Efficiency

**Cost per GB ingested:**
- Track over time
- Compare to industry benchmarks
- Optimize continuously

**Cost per query:**
- Average cost
- Cost by query type
- Optimization opportunities

**Total cost of ownership:**
- Platform infrastructure cost
- Operational overhead
- Developer time saved

## Operating Model Maturity

### Level 1: Ad-Hoc
- Manual pipeline creation
- No standards
- Limited self-serve
- High operational burden

### Level 2: Standardized
- Common patterns documented
- Some self-serve capabilities
- Basic governance
- Platform team bottleneck

### Level 3: Self-Serve Platform
- Most tasks self-serve
- Clear contracts and SLAs
- Cost attribution
- Platform enables, doesn't block

### Level 4: Product Platform
- Full self-serve
- Predictive quality
- Automated optimization
- Platform as competitive advantage

## Next Steps

- [Quality, Governance & Observability](06-quality-governance.md) - How to ensure quality and govern data
- [Cost Efficiency & Scale](07-cost-efficiency.md) - Advanced cost optimization

