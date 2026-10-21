# AWS Certified Solutions Architect – Associate (SAA-C03)

Welcome to the AWS Solutions Architect certification guide. This section prepares you for the **SAA-C03** exam, matching the official exam domains and focusing on hands-on architectural decision-making — the core skill tested.

## Exam Overview

| Item | Details |
|------|---------|
| **Code** | SAA-C03 |
| **Questions** | 65 total (50 scored, 15 unscored and not identified) |
| **Question Types** | Multiple choice (1 of 4) and multiple response (2+ of 5+) |
| **Scoring** | Scaled 100–1000; **pass = 720**; compensatory model |
| **Duration** | 130 minutes |
| **Recommended Experience** | ~1 year hands-on designing on AWS |

**Official Resources:**
- [Exam Guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)
- [Certification Page](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

---

## Domains and Weights

| Domain | Topic | Weight |
|--------|-------|--------|
| 1 | Design Secure Architectures | 30% |
| 2 | Design Resilient Architectures | 26% |
| 3 | Design High-Performing Architectures | 24% |
| 4 | Design Cost-Optimized Architectures | 20% |

---

## Study Plan (1 hour/day, ~10–13 weeks)

The phased approach prioritises by domain weight and active practice:

| Phase | Focus | Weight | Weeks |
|-------|-------|--------|-------|
| 1 | Domain 1 — Security (IAM, network, app & data protection) | 30% | 1–3 |
| 2 | Domain 2 — Resilience (decoupling, HA, DR strategies) | 26% | 4–5 |
| 3 | Domain 3 — Performance (compute, storage, DBs, network, data) | 24% | 6–8 |
| 4 | Domain 4 — Cost (purchasing, storage, network, tools) | 20% | 9–10 |
| 5 | Review + practice exams + close gaps | — | 11–13 |

**Exam-day strategy:** Identify keyword requirements ("most cost-effective", "highly available", "least operational overhead"); eliminate wrong options first; flag-and-move when unsure (~2 min per question); no penalty for guessing.

---

## Study Resources

### Recommended Courses
- **[Official AWS Exam-Prep (Coursera)](https://www.coursera.org/learn/aws-certified-solutions-architect-associate)** — structured, concise
- **[Packt SAA-C03 Specialization (Coursera)](https://www.coursera.org/specializations/packt-aws-certified-solutions-architect-associate-saa-c03)** — in-depth alternative
- **[AWS Skill Builder](https://skillbuilder.aws)** — official practice questions and full-length practice exam

---

## Files in This Section

- **[Domain 1: Secure Architectures](./01-security.md)** — IAM, network security, app & data protection (30%)
- **[Domain 2: Resilient Architectures](./02-resilience.md)** — Decoupling, HA, DR, and 4 DR strategies (26%)
- **[Domain 3: High-Performance Architectures](./03-performance.md)** — Compute, storage, DBs, CDN, data ingestion (24%)
- **[Domain 4: Cost-Optimized Architectures](./04-cost.md)** — Purchasing, storage tiering, network egress, cost tools (20%)
- **[Services Reference](./services-reference.md)** — Complete in-scope services grouped by category with exam priority tags
- **[Study Plan & Exam Strategy](./study-plan.md)** — Detailed 1h/day schedule, exam-day tips, and resource links
- **[Resources](./resources.md)** — Well-Architected Framework, Decision Guides, Shared Responsibility Model

---

## Quick Navigation by Topic

### Security (Domain 1 — 30%)
- IAM policies, roles, STS, cross-account access → [01-security.md](./01-security.md#iam-in-depth)
- Network security (security groups, NACLs, VPC) → [01-security.md](./01-security.md#network-security)
- Application & data protection → [01-security.md](./01-security.md#application--data-protection)

### Resilience (Domain 2 — 26%)
- Decoupling patterns (SQS, SNS, EventBridge) → [02-resilience.md](./02-resilience.md#decoupling)
- High availability and multi-AZ → [02-resilience.md](./02-resilience.md#high-availability)
- Disaster recovery & 4 strategies → [02-resilience.md](./02-resilience.md#disaster-recovery)

### Performance (Domain 3 — 24%)
- Storage selection (S3, EBS, EFS, FSx) → [03-performance.md](./03-performance.md#storage)
- Compute options (EC2, Lambda, Fargate, ECS, EKS) → [03-performance.md](./03-performance.md#compute)
- Databases (RDS, Aurora, DynamoDB, ElastiCache) → [03-performance.md](./03-performance.md#databases)
- Content delivery & global performance → [03-performance.md](./03-performance.md#content-delivery--global)

### Cost (Domain 4 — 20%)
- EC2 purchasing options (On-Demand, Reserved, Savings Plans, Spot) → [04-cost.md](./04-cost.md#compute-purchasing)
- Storage cost optimization (S3 lifecycle, tiering) → [04-cost.md](./04-cost.md#storage-cost-optimization)
- Network & operational costs → [04-cost.md](./04-cost.md#network--operational-costs)

---

## Key Exam Patterns

**"Choose the most cost-effective"** — Usually Reserved Instances/Savings Plans, not On-Demand. But if unpredictable workload, could be Spot or auto-scaling.

**"Most highly available"** → Multi-AZ (never single-AZ), or multi-Region if extreme resilience required.

**"Least operational overhead"** → Managed services (RDS > self-hosted DB, Lambda > EC2), serverless where possible.

**"Real-time analytics / low-latency queries"** → ElastiCache for caching, DynamoDB for NoSQL, Athena for SQL on S3 (if batch acceptable).

**"Disaster recovery"** → Know RPO/RTO for each of 4 strategies: Backup & Restore (long), Pilot Light (short), Warm Standby (shorter), Active-Active (near-zero).

---

## Coverage Summary

| Domain | Focus | Weight | Files |
|--------|-------|--------|-------|
| Security | IAM, network, app/data protection | 30% | [01-security.md](./01-security.md) |
| Resilience | Decoupling, HA, DR, 4 strategies | 26% | [02-resilience.md](./02-resilience.md) |
| Performance | Storage, compute, DBs, CDN, data | 24% | [03-performance.md](./03-performance.md) |
| Cost | Purchasing, storage, network, tools | 20% | [04-cost.md](./04-cost.md) |
| References | Services, frameworks, resources | — | [services-reference.md](./services-reference.md), [resources.md](./resources.md) |

---

**Note:** The exam guide appendix is explicitly non-exhaustive; services are not equally weighted. Focus your study on [High] and [Med] priority services in [services-reference.md](./services-reference.md). Recognise [Rec] services but don't memorise them.

**Key Principle:** The SAA-C03 tests architectural judgment, not memorisation. Understand *when to choose* each service — the trade-offs between security/resilience/performance/cost — and you'll answer most questions confidently.
