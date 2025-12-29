# Manager / Leadership View

> "The next generation doesn't need more dashboards. They need better stories about why the data matters."

This chapter is for data engineering leaders—managers, directors, and executives who need to build, scale, and measure data platform teams. It provides frameworks for making strategic decisions and evaluating success.

> "If Gen-Z doesn't care about your data problem, you've explained the wrong problem."

## What to Measure

### Platform Health Metrics

**Adoption Metrics**
- **% of data sources on platform**: Target 80%+ within 12 months
- **% of transformations on platform**: Target 90%+ within 18 months
- **Active users per month**: Track growth, target 20%+ MoM early on
- **Self-serve adoption**: % of pipelines created via self-serve (target 70%+)

**Why it matters**: Low adoption = platform isn't delivering value.

**Reliability Metrics**
- **Platform uptime**: Target 99.9% (8.76 hours downtime/year)
- **Pipeline success rate**: Target > 99% (less than 1% failure rate)
- **Mean time to recovery (MTTR)**: Target < 1 hour for critical pipelines
- **Incident frequency**: Track and trend (target: decreasing)

**Why it matters**: Unreliable platform = teams won't trust it.

**Performance Metrics**
- **Ingestion latency**: p50, p95, p99 (target: meet SLAs)
- **Query performance**: p50, p95, p99 (target: < 5s p95 for common queries)
- **Transformation time**: Track and optimize (target: < SLA)
- **Resource utilization**: CPU, memory (target: 60-80% for efficiency)

**Why it matters**: Slow platform = poor developer experience.

### Developer Experience Metrics

**Time to Value**
- **Time to first ingestion**: Target < 1 day (from request to data available)
- **Time to first transformation**: Target < 2 days
- **Time to production**: Track end-to-end (target: < 1 week for standard use cases)

**Developer Satisfaction**
- **NPS (Net Promoter Score)**: Survey quarterly, target 50+
- **Support ticket volume**: Track and trend (target: decreasing as platform matures)
- **Documentation usage**: Track views, searches (target: high usage = good docs)

**Self-Serve Adoption**
- **% of pipelines created via self-serve**: Target 70%+
- **% of transformations using standard frameworks**: Target 80%+
- **Support tickets per 100 pipelines**: Target < 5 (fewer tickets = better self-serve)

**Why it matters**: Poor DX = slow adoption, high support burden.

### Cost Metrics

**Cost Efficiency**
- **Cost per GB ingested**: Track and trend (target: decreasing)
- **Cost per query**: Track by query type (target: optimize expensive queries)
- **Storage cost per GB**: Track by tier (target: move to cheaper tiers)
- **Total cost of ownership**: Platform + operations + developer time

**Cost Attribution**
- **Cost by team**: Showback/chargeback (target: teams see and optimize their costs)
- **Cost by source**: Identify expensive sources (target: optimize or deprecate)
- **Cost by consumer**: Identify expensive queries (target: optimize)

**Budget Management**
- **Monthly spend vs budget**: Track and forecast
- **Cost growth rate**: Target < 20% YoY (after initial growth phase)
- **ROI of optimizations**: Track savings from projects

**Why it matters**: Uncontrolled costs = platform becomes unsustainable.

### Business Impact Metrics

**Data Availability**
- **% of critical data sources with SLAs**: Target 100%
- **SLA compliance rate**: Target > 99%
- **Data freshness**: % of sources meeting freshness SLAs (target > 95%)

**Data Quality**
- **Quality score**: Composite score across dimensions (target > 95%)
- **Quality incidents**: Track and trend (target: decreasing)
- **Time to detect quality issues**: Target < 1 hour

**Business Value**
- **Downstream consumers**: Number of teams/products using platform data
- **Query volume**: Track growth (indicates value)
- **Data products enabled**: New products/features enabled by platform

**Why it matters**: Platform exists to enable business, not as an end in itself.

## How to Scale Teams

### Team Structure

**Small Team (< 10 engineers)**
- **Structure**: Generalists (everyone does everything)
- **Roles**: 2-3 platform engineers, 1 part-time SRE
- **Focus**: Get platform working, establish patterns

**Medium Team (10-50 engineers)**
- **Structure**: Some specialization (platform, ingestion, transformations)
- **Roles**: 5-10 platform engineers, 1-2 SRE, 1 PM
- **Focus**: Scale platform, improve self-serve, optimize costs

**Large Team (50+ engineers)**
- **Structure**: Specialized teams (platform, ingestion, transformations, quality)
- **Roles**: 15-30 platform engineers, 3-5 SRE, 2-3 PM, cost optimization team
- **Focus**: Platform maturity, advanced capabilities, cost efficiency

### Hiring Strategy

**Early Stage (0-10 engineers)**
- **Hire**: Senior generalists (can do everything)
- **Skills**: Platform engineering, data engineering, operations
- **Experience**: 5+ years, worked at scale

**Growth Stage (10-50 engineers)**
- **Hire**: Mix of generalists and specialists
- **Skills**: Platform engineering, specific domains (ingestion, transformations)
- **Experience**: 3-7 years, relevant domain expertise

**Mature Stage (50+ engineers)**
- **Hire**: Specialists + leaders
- **Skills**: Deep expertise in specific areas, leadership
- **Experience**: 5+ years, leadership experience for senior roles

### Team Development

**Career Paths**
- **Individual Contributor (IC)**: Engineer → Senior → Staff → Principal
- **Management**: Engineer → Tech Lead → Manager → Director
- **Hybrid**: Tech Lead (IC + leadership)

**Skills Development**
- **Technical**: Platform engineering, data engineering, cloud, tooling
- **Soft**: Communication, collaboration, product thinking
- **Leadership**: For senior roles (influence, strategy, mentoring)

**Retention**
- **Clear career paths**: Growth opportunities
- **Interesting work**: Challenging problems, impact
- **Compensation**: Competitive, aligned with market
- **Culture**: Learning, collaboration, ownership

## Evaluating Architecture Maturity

### Maturity Model

**Level 1: Ad-Hoc**
- Manual pipeline creation
- No standard patterns
- Limited observability
- High operational burden
- **Indicators**: Everything is custom, high support burden

**Level 2: Standardized**
- Common patterns documented
- Some self-serve capabilities
- Basic monitoring
- Platform team bottleneck
- **Indicators**: Some standards, but still manual for many things

**Level 3: Self-Serve Platform**
- Most tasks self-serve
- Clear contracts and SLAs
- Cost attribution
- Platform enables, doesn't block
- **Indicators**: 70%+ self-serve, low support burden, teams move fast

**Level 4: Product Platform**
- Full self-serve
- Predictive quality
- Automated optimization
- Platform as competitive advantage
- **Indicators**: Minimal platform team involvement, high satisfaction, innovation

### Assessment Framework

**Evaluate across dimensions:**

| Dimension | Level 1 | Level 2 | Level 3 | Level 4 |
|-----------|---------|---------|---------|---------|
| **Ingestion** | Manual, custom | Some templates | Self-serve, standardized | Fully automated |
| **Transformation** | Ad-hoc scripts | Some frameworks | Standard frameworks, self-serve | Optimized, automated |
| **Storage** | Ad-hoc, no standards | Some standards | Tiered, lifecycle policies | Optimized, predictive |
| **Quality** | Manual checks | Some automated | Comprehensive, automated | Predictive, self-healing |
| **Governance** | Ad-hoc | Basic policies | Contracts, automated | Federated, self-service |
| **Observability** | Limited | Basic metrics | Comprehensive | Predictive |
| **Cost** | Unattributed | Some attribution | Full attribution, optimization | Automated optimization |

**Scoring**: Rate each dimension 1-4, average = maturity level.

### Roadmap to Maturity

**Level 1 → 2 (6-12 months)**
- Document common patterns
- Create standard templates
- Basic monitoring
- Establish SLAs

**Level 2 → 3 (12-18 months)**
- Build self-serve capabilities
- Implement contracts
- Cost attribution
- Improve observability

**Level 3 → 4 (18-24 months)**
- Advanced automation
- Predictive capabilities
- Self-healing
- Innovation

## Strategic Decisions

### Build vs Buy

**Build when:**
- Unique requirements (no tool fits)
- Competitive advantage (platform is differentiator)
- Have engineering resources
- Cost of buying > cost of building

**Buy when:**
- Standard requirements (tools exist)
- Want to move fast (time to market)
- Limited engineering resources
- Cost of building > cost of buying

**Hybrid approach** (recommended):
- Buy for standard capabilities (ingestion, orchestration)
- Build for unique/differentiating capabilities
- Customize bought tools as needed

### Central vs Decentralized

**Central when:**
- Need strong governance
- Limited domain expertise
- Want consistency
- Large organization (1000+ engineers)

**Decentralized when:**
- Need speed and autonomy
- Strong domain expertise
- Smaller organization (< 100 engineers)

**Hybrid** (recommended for most):
- Central platform team (infrastructure, standards)
- Domain teams (business logic, transformations)
- Shared governance (framework, not control)

### Cloud Strategy

**Single cloud when:**
- Simpler operations
- Better integration
- Cost optimization (commitments)
- Team expertise in one cloud

**Multi-cloud when:**
- Vendor risk mitigation
- Regulatory requirements
- Best-of-breed (different clouds for different needs)
- Large scale (leverage competition)

**Recommendation**: Start single cloud, consider multi-cloud as you scale.

## Budget Planning

### Cost Components

**Infrastructure (40-60%)**
- Compute (queries, transformations)
- Storage (hot, warm, cold)
- Network (transfer, egress)

**Tools & Licenses (10-20%)**
- SaaS tools (Fivetran, dbt Cloud)
- Software licenses
- Managed services

**Operations (10-15%)**
- Platform team (salaries)
- On-call, incident response
- Maintenance

**Development (10-15%)**
- New features
- Optimizations
- Experiments

### Budget Planning Process

**1. Baseline current spend**
- Track all costs (infrastructure, tools, people)
- Categorize by team, project, source

**2. Forecast growth**
- Volume growth (data, queries, users)
- Feature additions (new capabilities)
- Team growth (headcount)

**3. Identify optimizations**
- Cost reduction opportunities
- Efficiency improvements
- Tool consolidation

**4. Build budget**
- Base: Current spend + growth
- Optimizations: Apply savings
- Buffer: 10-20% for unknowns

**5. Track and adjust**
- Monthly reviews
- Quarterly forecasts
- Adjust as needed

### ROI Framework

**Calculate ROI for investments:**

```python
# Example: Self-serve ingestion project
engineering_cost = 200  # hours * $150/hour = $30,000
time_saved_per_pipeline = 8  # hours (manual → self-serve)
pipelines_per_month = 10
hourly_rate = 150  # $/hour

monthly_savings = time_saved_per_pipeline * pipelines_per_month * hourly_rate
# = 8 * 10 * 150 = $12,000/month

annual_savings = monthly_savings * 12  # $144,000
roi = (annual_savings - engineering_cost) / engineering_cost  # 380%
payback_period = engineering_cost / monthly_savings  # 2.5 months
```

**Decision rule**: If payback < 12 months and ROI > 100%, do it.

## Communication & Reporting

### Executive Reporting

**Monthly report structure:**
1. **Platform health**: Uptime, success rates, performance
2. **Adoption**: Users, pipelines, growth
3. **Cost**: Spend vs budget, trends, optimizations
4. **Business impact**: Data products enabled, consumers
5. **Risks & issues**: What's blocking, what needs attention

**Format**: 1-2 pages, visual (charts), actionable

### Team Communication

**Regular updates:**
- **Weekly**: Team standup, platform health
- **Monthly**: All-hands, metrics review
- **Quarterly**: Roadmap, strategy

**Channels:**
- Slack/Teams: Day-to-day
- Email: Important announcements
- Wiki/Docs: Documentation, runbooks

### Stakeholder Management

**Key stakeholders:**
- **Engineering leaders**: Platform capabilities, reliability
- **Product leaders**: Data products, business impact
- **Finance**: Cost, budget, ROI
- **Executives**: Strategy, business value

**Tailor message to audience:**
- Engineers: Technical details, capabilities
- Product: Business impact, features
- Finance: Cost, efficiency
- Executives: Strategy, value, risks

## Common Pitfalls

### Pitfall 1: Over-Engineering

**Symptom**: Building complex solutions for simple problems.

**Solution**: Start simple, add complexity only when needed.

### Pitfall 2: Ignoring Costs

**Symptom**: Costs growing unchecked.

**Solution**: Track costs from day one, optimize continuously.

### Pitfall 3: Poor Developer Experience

**Symptom**: Low adoption, high support burden.

**Solution**: Invest in self-serve, documentation, tooling.

### Pitfall 4: No Metrics

**Symptom**: Can't measure success.

**Solution**: Define metrics early, track religiously.

### Pitfall 5: Chasing Trends

**Symptom**: Adopting every new tool/pattern.

**Solution**: Adopt when it solves real problems, not because it's new.

## Success Criteria

### Platform Success

**Platform is successful when:**
- **Adoption**: 80%+ of data sources on platform
- **Reliability**: 99.9% uptime, < 1% failure rate
- **Developer experience**: < 1 day to first ingestion, high satisfaction
- **Cost**: Controlled growth, optimized spend
- **Business impact**: Enabling new products, features, insights

### Team Success

**Team is successful when:**
- **Platform maturity**: Level 3+ (self-serve platform)
- **Team growth**: Retaining talent, growing capabilities
- **Innovation**: Building new capabilities, staying current
- **Impact**: Clear business value, recognition

## Next Steps

- Review [Foundations](01-foundations.md) - Ensure solid foundation
- Review [Platform & Operating Model](05-platform-operating-model.md) - Design your operating model
- Review [Cost Efficiency](07-cost-efficiency.md) - Optimize costs

---

**Remember**: Building a data platform is a journey, not a destination. Start simple, measure everything, iterate based on data.

