# Domain 4: Design Cost-Optimized Architectures (20%)

This domain focuses on **choosing cost-effective options without sacrificing performance or resilience**. The exam tests whether you understand pricing trade-offs and can recommend the cheapest viable option.

---

## Focus Areas

**Compute Purchasing Models**
- **On-Demand**: Pay per second; highest per-unit cost; flexible; baseline for cost calculations.
- **Reserved Instances (RIs)**: 1- or 3-year commitment; 30–72% discount. Capacity reservation (if placed in AZ).
- **Savings Plans**: 1- or 3-year commitment; applies to EC2, Fargate, Lambda; more flexible than RIs across instance families.
- **Spot Instances**: Up to 90% discount; can be interrupted (2-min warning); ideal for stateless, fault-tolerant workloads.
- **Dedicated Hosts**: Physical server reservation; for license compliance (BYOL). Expensive, but required for some software.

**Storage Cost Optimization**
- S3 Intelligent-Tiering: auto-transitions between tiers based on access patterns; small per-object fee.
- S3 Lifecycle policies: transition to cheaper tiers (Infrequent Access, Glacier) after age threshold.
- S3 Glacier / Glacier Deep Archive: cheap long-term storage; slow retrieval; Glacier Deep Archive ~$1/TB/month.
- EBS snapshot optimization: incremental snapshots; delete old snapshots.
- Reduce S3 requests: use CloudFront to cache; batch operations.

**Network Cost Optimization**
- Data transfer OUT (egress) costs money; IN is free. Use CloudFront to cache (reduce egress to internet).
- NAT gateway charges per-GB processed + per-hour. Consider NAT instances (EC2) for high throughput.
- VPC endpoints (PrivateLink): avoid NAT gateway, direct route to S3/DynamoDB (free for S3/DynamoDB).
- Multi-region: egress across regions costs money; minimize cross-region traffic.

**Database Cost**
- RDS: On-Demand or Reserved Instances (1–3 year commitment).
- Aurora: Pay per GB stored + per-million requests. Scale read replicas on-demand.
- DynamoDB: Provisioned (pay for capacity) or on-demand (pay per request). On-demand better for unpredictable; provisioned cheaper for predictable.
- ElastiCache: Reserved nodes (1–3 year) cheaper than on-demand.

**Cost Tools & Monitoring**
- **Cost Explorer**: Visualise spending; identify savings opportunities; forecast.
- **Budgets**: Set alerts when spending approaches threshold.
- **Cost Allocation Tags**: Tag resources (Environment, Project, Owner); group costs by tag.
- **Trusted Advisor**: Automatic cost/performance/security recommendations (checks free for some, full suite with Business/Enterprise Support).
- **Compute Optimizer**: Rightsizing recommendations (unused resources, over-provisioned instances).

---

## Documentation Links

### Purchasing & Pricing
- [EC2 Purchasing Options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
- [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)

### Storage Optimization
- [S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html)

### Network & Costs
- [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)

### Cost Management
- [Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)

---

## Key Decision Points

### "On-Demand, Reserved Instances, or Spot?"
**On-Demand**: Unpredictable workload, or baseline compute. Highest cost per hour.
**Reserved Instances (RIs)**: Predictable, stable workload (e.g., database always running). 1-year saves ~40%, 3-year saves ~60%. Risk: over-commitment.
**Savings Plans**: Similar discount to RIs (40–60%), more flexible (works across instance families, Fargate, Lambda).
**Spot**: Highly variable workload, stateless, fault-tolerant (batch jobs, CI/CD, distributed processing). Up to 90% discount; interruption is risk.
**Hybrid strategy** (common): Baseline on RIs/Savings Plans, burst on On-Demand or Spot.

### "DynamoDB: provisioned or on-demand billing?"
**Provisioned (RCU/WCU)**: Fixed cost; good if traffic is predictable. Overpay if underutilised.
**On-Demand**: Pay per request; good for unpredictable/bursty. Scales automatically, but costlier per request at high volume.
**Decision rule:** Predictable steady state → provisioned. Unpredictable/spiky → on-demand. Can switch between modes.

### "How to reduce S3 costs for a data lake?"
**Lifecycle policy**: Move old objects to Intelligent-Tiering, then Glacier Deep Archive. Set expiration (delete after 7 years).
**Intelligent-Tiering**: Auto-transitions between tiers; charges $0.0025 per object per month for monitoring.
**CloudFront**: If data is frequently accessed, cache with CloudFront (reduces S3 GET requests; egress cheaper from edge).
**Batch operations**: Delete unused versions, old snapshots in bulk.

### "NAT gateway is expensive — alternatives?"
**NAT instances** (EC2): You manage the instance; cheaper bandwidth; single-AZ, single-instance (potential bottleneck). OK for low throughput.
**VPC endpoints** (PrivateLink): Free route to S3 and DynamoDB (no NAT gateway). Don't use NAT for S3 access if possible.
**NAT gateway**: Managed, HA, but $0.045/hour + data processing charges. Necessary if outbound to internet is required (not just to S3/DynamoDB).

### "Reserved Instances vs Savings Plans vs Spot for this scenario?"
**Predictable web app (steady 50 EC2 instances)**: Reserved Instances or Savings Plans (40–60% discount). Size RIs to baseline, burst with On-Demand.
**Batch job (run 100 instances for 2 hours nightly)**: Savings Plans (works with auto-scaling); or Spot Instances (risky, but up to 90% off).
**Database (RDS always running)**: Reserved Instances (1- or 3-year commitment). RI Marketplace if you want to sell unused capacity.

---

## Exam Scenarios

**Scenario 1: Cost-optimize an over-provisioned RDS instance**
Trusted Advisor flags instance as under-utilised (CPU 5–10%, unused reserved capacity). Recommendation: Downsize from db.r5.2xlarge (8 vCPU) to db.r5.large (2 vCPU). Or switch from provisioned to Aurora on-demand (pay per request). Savings: 50–70%.

**Scenario 2: Startup wants to launch with minimum cost**
Use Spot Instances for stateless application servers (up to 90% discount). RDS on-demand (flexible if usage scales slowly). S3 Standard for data (no tiering yet). Auto Scaling: scale down to 1 instance at night. CloudFront for static assets (cache = fewer origin requests = lower data transfer). Estimate: 70% less than On-Demand baseline.

**Scenario 3: Multi-region data lake + analytics**
Data ingested in us-east-1 → S3 Standard. Replicated to eu-west-1 → S3 Standard-IA (cheaper storage after replication lag). Athena queries us-east-1 (free, only S3 GET charges). Glacier lifecycle after 1 year. VPC endpoint to S3 (no NAT gateway charges). Cost per TB: ~$10/month (Intelligent-Tiering) vs $20 if all Standard.

**Scenario 4: Right-sizing a fleet**
20 x m5.large On-Demand (dev team). Compute Optimizer recommends t3.medium (less CPU needed). 1-year RI for 15 instances (baseline), 5 on-demand (variance). Savings: from $7K/month to $3K/month. Tags: Environment=Development, Project=DataPlatform. Cost allocation report shows full cost breakdown by tag.

---

## Common Exam Pitfalls

- **Confusing RI discount with capacity reservation:** Standard RIs ≠ capacity; Convertible RIs offer more flexibility but lower discount.
- **Forgetting NAT gateway egress charges:** NAT gateway charges per GB + per hour. Costs add up quickly with high throughput.
- **Overestimating Spot instance savings:** Spot is 90% off, but interruption risk means failover costs. Not ideal for stateful, critical workloads.
- **Ignoring data transfer costs:** Egress to internet, cross-region, cross-AZ all cost money. CloudFront caching can reduce egress significantly.
- **Assuming Reserved Instances work across all instance families:** Standard RIs are family/region-locked. Savings Plans are more flexible.

---

**Next:** [Services Reference](./services-reference.md)
