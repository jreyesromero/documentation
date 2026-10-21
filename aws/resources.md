# AWS Resources — Framework, Guides, and Reference Material

This file collects the foundational AWS documents and frameworks that underpin the SAA-C03 exam and good architecture.

---

## AWS Well-Architected Framework

The official framework that AWS uses to evaluate architectures. The exam is implicitly based on these 5 pillars. Study order: Security → Reliability → Performance Efficiency → Cost Optimization → Operational Excellence.

- **[Well-Architected Framework Overview](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)** — Start here; foundation for all domains
- **[Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)** — Aligns with Domain 1
- **[Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)** — Aligns with Domain 2
- **[Performance Efficiency Pillar](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/welcome.html)** — Aligns with Domain 3
- **[Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)** — Aligns with Domain 4
- **[Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)** — Cross-cutting; automation, observability

---

## AWS Decision Guides

Quick reference for "when should I choose X vs Y" comparisons. These are high-signal for exam questions.

- **[AWS Decision Guides](https://docs.aws.amazon.com/decision-guides/latest/)** — Central hub
  - Compute (EC2 vs Lambda vs Containers vs Elastic Beanstalk)
  - Database (RDS vs DynamoDB vs Aurora; when SQL vs NoSQL)
  - Storage (S3 vs EBS vs EFS vs FSx; when to use each)
  - Messaging (SQS vs SNS vs EventBridge; point-to-point vs pub/sub)
  - Caching (ElastiCache vs CloudFront; when in-memory vs edge)

---

## Shared Responsibility Model

Critical for understanding which security tasks are AWS's and which are yours. Exam scenario: "Who manages encryption key rotation?" → You (customer responsibility).

- **[Shared Responsibility Model Overview](https://aws.amazon.com/compliance/shared-responsibility-model/)**

**Quick reference:**
- **AWS manages:** Infrastructure, host OS, hypervisor, network infrastructure, physical security
- **You manage:** OS patches (EC2), application security, data encryption, IAM policies, network security (security groups, NACLs)
- **Both:** Patch management (AWS patches RDS; you patch EC2), configuration management, training

---

## Key Whitepapers

Deep dives into specific topics; read these after Week 5 if time permits.

### Disaster Recovery (MUST READ for Domain 2)
- **[Disaster Recovery of Workloads on AWS](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)** — Explains all 4 strategies with RPO/RTO; exam-critical

### Security & Compliance
- **[AWS Security Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/welcome.html)** — Layered defense, encryption, monitoring

### Cost Optimization
- **[Cost Optimization on AWS](https://docs.aws.amazon.com/whitepapers/latest/cost-optimization-right-sizing/welcome.html)** — Rightsizing, reserved capacity, purchasing options

### Reliability & Operations
- **[AWS Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)** — Multi-AZ, failover, recovery strategies

---

## Documentation Shortcuts (By Domain)

### Domain 1: Security
- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [VPC Security](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) (security groups, NACLs)
- [KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)
- [ACM Documentation](https://docs.aws.amazon.com/acm/)

### Domain 2: Resilience
- [Disaster Recovery Strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/) (4 strategies, critical)
- [SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/)
- [SNS Documentation](https://docs.aws.amazon.com/sns/)
- [Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Route 53 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)

### Domain 3: Performance
- [S3 Storage Classes & Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/)
- [EBS Volume Types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [RDS User Guide](https://docs.aws.amazon.com/rds/)
- [Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/)
- [CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/)
- [Kinesis Streams Guide](https://docs.aws.amazon.com/streams/latest/dev/)

### Domain 4: Cost
- [EC2 Purchasing Options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
- [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/)
- [Cost Explorer Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)

---

## Language Note

Most AWS user guides have a **Spanish version** (toggle language selector on the page or use `es_es` in URL). FAQs and whitepapers are typically **English-only**. The exam vocabulary is English, so reading official documentation in English is recommended for consistency.

---

## Quick Review Checklist

Before your exam, make sure you've reviewed:

- [ ] **4 DR strategies** (Backup & Restore, Pilot Light, Warm Standby, Active-Active) and their RPO/RTO
- [ ] **IAM policy evaluation** (explicit deny, organization SCPs, permission boundaries)
- [ ] **When to use:** RDS vs Aurora vs DynamoDB; SQS vs SNS vs EventBridge; ALB vs NLB
- [ ] **Multi-AZ vs read replicas** (HA vs read scaling)
- [ ] **EC2 purchasing models** (On-Demand, RI, Savings Plans, Spot) and cost trade-offs
- [ ] **Well-Architected Framework 5 pillars** and which services align to each
- [ ] **Shared Responsibility Model** (who manages encryption, patching, OS, security groups)

**Key principle:** The SAA-C03 tests **architectural judgment**, not memorisation. If you understand the trade-offs and can reason about "which service fits this requirement," you'll pass.
