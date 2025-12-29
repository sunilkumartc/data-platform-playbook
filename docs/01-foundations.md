# Foundations

## What is Data Engineering?

Data Engineering is the discipline of designing, building, and operating systems that transform raw data into reliable, accessible, and actionable information at scale. Unlike data science (which focuses on analysis and modeling) or software engineering (which focuses on application logic), data engineering sits at the intersection of infrastructure, reliability, and data product delivery.

### Modern Definition

At its core, data engineering is about:

1. **Reliability**: Ensuring data arrives on time, in the right format, with the right quality
2. **Scale**: Handling terabytes to petabytes of data across thousands of pipelines
3. **Velocity**: Supporting both batch and real-time use cases
4. **Governance**: Maintaining lineage, quality, and compliance
5. **Cost Efficiency**: Delivering value without breaking the bank

### The Shift: From ETL to Platform

Traditional data engineering focused on **ETL pipelines**—point-to-point data movement with transformation logic embedded in the pipeline. Modern data engineering is about **platforms**—self-serve infrastructure that enables teams to:

- Ingest data with minimal friction
- Transform data using their preferred tools
- Store data in cost-appropriate tiers
- Serve data to consumers (analytics, ML, operational systems)

## Core Principles

### 1. Data as a Product

Treat data assets as **first-class products**, not byproducts of applications.

**Implications:**
- Clear ownership and accountability
- Defined SLAs (freshness, availability, quality)
- Versioned schemas and contracts
- Documentation and discoverability
- Lifecycle management

**Anti-pattern**: "Just dump the data somewhere and we'll figure it out later."

### 2. Separation of Concerns

Establish clear boundaries between:

- **Ingestion**: Getting data into the platform
- **Transformation**: Shaping data for consumption
- **Storage**: Persisting data in appropriate formats/tiers
- **Serving**: Delivering data to consumers

**Why it matters**: Each layer can evolve independently, scale independently, and fail independently.

```
┌─────────────┐
│  Ingestion  │  ← Push/pull, CDC, streaming
└──────┬──────┘
       │
┌──────▼──────┐
│ Storage     │  ← Raw, curated, archive tiers
└──────┬──────┘
       │
┌──────▼──────┐
│Transform    │  ← ELT, streaming transforms
└──────┬──────┘
       │
┌──────▼──────┐
│  Serving    │  ← Analytics, ML, APIs
└─────────────┘
```

### 3. Platform Thinking

Build **self-serve capabilities** that enable teams, not bottlenecks.

**Platform provides:**
- Standardized ingestion paths
- Managed compute (Spark, Flink, etc.)
- Storage abstractions (tables, partitions, lifecycle)
- Metadata and discovery
- Observability and alerting

**Teams provide:**
- Business logic
- Transformation code
- Quality checks
- Documentation

**Anti-pattern**: Central team manually creates every pipeline.

### 4. Cost Awareness

Every architectural decision has cost implications. Make them explicit.

**Key cost drivers:**
- **Compute**: Query execution, transformation jobs
- **Storage**: Hot, warm, cold tiers
- **Network**: Cross-region transfers, egress
- **Operations**: Pipeline maintenance, incident response

**Principle**: Start with the cheapest solution that meets requirements. Optimize when you have data.

### 5. Contract-First Design

Define **data contracts** before ingestion begins.

**Contract includes:**
- Schema (with evolution rules)
- Freshness SLA
- Quality expectations
- Ownership and contact
- Cost attribution

**Benefit**: Prevents downstream breakage, enables automated validation.

## Platform Maturity Model

### Level 1: Ad-Hoc
- Manual pipeline creation
- No standard patterns
- Limited observability
- High operational burden

### Level 2: Standardized
- Common ingestion patterns
- Standardized storage formats
- Basic monitoring
- Some self-serve capabilities

### Level 3: Platform
- Self-serve ingestion
- Automated quality checks
- Rich metadata and discovery
- Cost attribution and optimization

### Level 4: Product
- Data contracts enforced
- Predictive quality monitoring
- Automated optimization
- Multi-tenant isolation

## Key Concepts

### Data Freshness

**Freshness** = Time between when data is generated and when it's available for consumption.

**Categories:**
- **Real-time**: < 1 minute (streaming)
- **Near real-time**: 1-15 minutes (micro-batch)
- **Batch**: 15 minutes - 24 hours
- **Historical**: > 24 hours (backfills, archives)

**Trade-off**: Lower latency = higher cost.

### Data Quality Dimensions

1. **Completeness**: Are all expected records present?
2. **Accuracy**: Does data reflect reality?
3. **Consistency**: Is data consistent across sources?
4. **Timeliness**: Is data fresh enough?
5. **Validity**: Does data conform to schema?
6. **Uniqueness**: Are there duplicates?

### Schema Evolution

Schemas change. Design for it.

**Strategies:**
- **Backward compatible**: New fields optional, old fields never removed
- **Versioning**: Explicit schema versions with migration paths
- **Schema registry**: Centralized schema management (e.g., Confluent Schema Registry)

**Anti-pattern**: Breaking changes without notice.

## Next Steps

- [End-to-End Lifecycle](02-lifecycle.md) - Understand the complete data journey
- [Platform & Operating Model](05-platform-operating-model.md) - Design your platform architecture

