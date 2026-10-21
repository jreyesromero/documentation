# AWS SAA-C03 Study Plan & Exam Strategy

## 1-Hour-Per-Day Study Schedule (10–13 Weeks)

Designed for active learning: prioritise by domain weight, practice questions from the start, and actively review every answer.

### Phase 1: Domain 1 — Security (30%, Weeks 1–3)

**Weeks 1–2: IAM Foundation**
- Day 1–2: IAM concepts — identity-based vs resource-based policies, roles vs users, groups, trust policies
- Day 3–4: Policy evaluation logic — how AWS decides allow/deny with multiple policies
- Day 5–6: STS, assume role, cross-account access — how to grant access across AWS accounts
- Day 7: Permission boundaries, policy permissions, least privilege principle
- Day 8–9: IAM Identity Center, federation, SAML — modern multi-account SSO
- Day 10–11: Organizations, SCPs — centralized governance across accounts
- Day 12–14: Practice questions on IAM (30+ questions); review every explanation

**Week 3: Network & Data Security**
- Day 1–2: VPC, subnets, security groups (stateful) vs NACLs (stateless)
- Day 3–4: VPC endpoints (PrivateLink), NAT gateway, S3 access without internet
- Day 5–6: KMS (encryption at rest), ACM (TLS certificates), key policies
- Day 7–10: Secrets Manager vs Parameter Store, Cognito vs IAM Identity Center
- Day 11–14: WAF, Shield, GuardDuty, Macie; practice questions (30+ questions)

**Review:** Domain 1 should take ~20 hours. Expect 70%+ on practice tests before moving forward.

---

### Phase 2: Domain 2 — Resilience (26%, Weeks 4–5)

**Week 4: Decoupling & Messaging**
- Day 1–2: SQS — standard vs FIFO, visibility timeout, dead-letter queues, polling
- Day 3–4: SNS — pub/sub, subscriptions, fan-out patterns
- Day 5–6: EventBridge — event bus, rules, targets; when to use vs SNS/SQS
- Day 7–10: Step Functions, API Gateway — workflow orchestration and API management
- Day 11–14: Practice questions on messaging (30+ questions)

**Week 5: Scaling, HA, and DR**
- Day 1–2: Auto Scaling — policies, lifecycle, metrics, termination; ASGs and EC2
- Day 3–4: Load balancing — ALB (Layer 7) vs NLB (Layer 4) vs GWLB; sticky sessions
- Day 5–6: Route 53 — routing policies (latency, geolocation, failover, weighted)
- Day 7–8: Multi-AZ vs read replicas; RDS, Aurora, DynamoDB replicas; when to use each
- Day 9–10: **The 4 DR strategies** — memorise these perfectly with RPO/RTO:
  - Backup & Restore (hours)
  - Pilot Light (~15 min)
  - Warm Standby (~5 min)
  - Active-Active (near-zero)
- Day 11–14: Practice questions on resilience (30+ questions); 4 DR scenarios

**Review:** Domain 2 should take ~16 hours. Expect 70%+ on practice tests before moving forward.

---

### Phase 3: Domain 3 — Performance (24%, Weeks 6–8)

**Week 6: Storage & Compute**
- Day 1–2: S3 storage classes (Standard, IA, Intelligent-Tiering, Glacier)
- Day 3–4: EBS (gp3, io2, st1, sc1) — which type for which workload
- Day 5–6: EFS, FSx — when NFS or file share instead of block storage
- Day 7–8: Compute options — EC2, Lambda, Fargate, ECS, EKS, Elastic Beanstalk
- Day 9–10: When to choose which compute: stateless → Lambda/Fargate; persistent state → EC2
- Day 11–14: Practice questions on storage & compute (30+ questions)

**Week 7: Databases & Caching**
- Day 1–2: RDS vs Aurora — performance, cost, multi-AZ, read replicas
- Day 3–4: DynamoDB — partition keys, sort keys, provisioned vs on-demand, GSI
- Day 5–6: ElastiCache (Redis vs Memcached) — when to use for caching/sessions
- Day 7–8: DynamoDB Global Tables, Aurora read replicas, cross-region replication
- Day 9–10: Multi-AZ failover, read scaling with replicas; RPO/RTO differences
- Day 11–14: Practice questions on databases (30+ questions)

**Week 8: Global Performance & Data**
- Day 1–2: CloudFront — caching, edge locations, origin, cache behaviours
- Day 3–4: Global Accelerator — when it's better than CloudFront (non-HTTP, UDP)
- Day 5–6: Kinesis Streams vs Firehose — real-time vs near-real-time ingestion
- Day 7–8: Glue, Athena — ETL and SQL on S3
- Day 9–10: Data pipeline patterns; when to use each tool
- Day 11–14: Practice questions on global & data services (30+ questions)

**Review:** Domain 3 should take ~24 hours. Expect 70%+ on practice tests before moving forward.

---

### Phase 4: Domain 4 — Cost (20%, Weeks 9–10)

**Week 9: Compute & Storage Cost**
- Day 1–2: EC2 purchasing (On-Demand, Reserved, Savings Plans, Spot)
- Day 3–4: When to use each: unpredictable (On-Demand), predictable (RI/Savings Plans), batch (Spot)
- Day 5–6: Savings Plans flexibility vs Reserved Instances trade-offs
- Day 7–8: S3 lifecycle, Intelligent-Tiering, Glacier — moving data to cheaper tiers
- Day 9–10: DynamoDB provisioned vs on-demand; when to use each
- Day 11–14: Practice questions on compute & storage cost (30+ questions)

**Week 10: Network, Database, and Tools**
- Day 1–2: NAT gateway egress charges vs NAT instances vs VPC endpoints
- Day 3–4: CloudFront egress reduction, cross-region data transfer costs
- Day 5–6: RDS Reserved Instances, Aurora Serverless; database cost optimisation
- Day 7–8: Cost Explorer, Budgets, Cost Allocation Tags, Trusted Advisor, Compute Optimizer
- Day 9–10: Rightsizing recommendations; cost-optimised architecture patterns
- Day 11–14: Practice questions on cost optimisation (30+ questions); cost scenarios

**Review:** Domain 4 should take ~12 hours. Expect 70%+ on practice tests before proceeding.

---

### Phase 5: Review & Practice Exams (Weeks 11–13)

**Week 11: Targeted Gap Review**
- Review domains where you scored <70% on practice tests
- Re-read the most confusing sections (e.g., policy evaluation logic, 4 DR strategies)
- Focus on "why" questions you got wrong; don't just memorise correct answers
- Take 2–3 timed practice exams (50 questions each)

**Week 12: Full-Length Practice Exams**
- Take AWS Skill Builder official practice exam (65 questions, 130 min)
- Aim for 75%+ (720/1000 scaled); borderline passing is 72%
- Review every wrong answer; note if it's a knowledge gap or misreading
- Focus on questions with multiple "plausible" options; learn why others are wrong

**Week 13: Final Review & Exam**
- Review high-weight domains (1 & 2) if time is short
- Memorise the 4 DR strategies, AWS service matrix, key pricing differences
- Take one final timed practice exam if scoring is uneven
- Exam day: read keywords ("most cost-effective", "highly available"), eliminate wrong options first, flag-and-move if unsure

---

## Exam-Day Strategy

### Before the Exam
- Get 8 hours of sleep the night before; no all-nighters
- Eat a light breakfast; avoid heavy foods that make you drowsy
- Test your exam centre computer + keyboard (if remote, test internet, audio, camera)
- Bring ID and arrive 15 minutes early

### During the Exam (65 Questions, 130 Minutes = ~2 min per question)

**Read for keywords:**
- "Most cost-effective" → Usually RI/Savings Plans, not On-Demand
- "Highly available" → Multi-AZ, no single points of failure
- "Least operational overhead" → Managed services, serverless
- "Real-time" → Kinesis Streams, not Firehose
- "DR" → Identify RPO/RTO requirement and match to strategy

**Elimination technique:**
1. Read the question and scenario
2. Eliminate 1–2 obviously wrong options (mismatched to requirement)
3. For remaining options, re-read the scenario for clues
4. Choose the *most aligned* option, not the perfect option

**Flag-and-move:**
- Flag if you're unsure (takes seconds)
- Come back to flagged questions if time remains
- **No penalty for guessing** — never leave questions blank

**Time management:**
- Don't spend >2–3 min per question
- If you're stuck after 90 sec, make your best guess and move on
- If time remains (rare), review flagged questions

**Common pitfalls to avoid:**
- Don't overthink security/compliance requirements — re-read for keywords
- Don't assume highest-cost option is wrong (sometimes it's right for HA)
- Don't forget the 4 DR strategies; they appear on nearly every SAA exam

---

## Recommended Courses & Practice Resources

### Official AWS Resources (Highest Fidelity)
- **[AWS Skill Builder](https://skillbuilder.aws)** — official practice questions + full-length practice exam; start here after Week 5
- **[Official Exam-Prep Course (Coursera)](https://www.coursera.org/learn/aws-certified-solutions-architect-associate)** — structured, ~30 hours; covers all domains

### Alternative (In-Depth)
- **[Packt SAA-C03 Specialization (Coursera)](https://www.coursera.org/specializations/packt-aws-certified-solutions-architect-associate-saa-c03)** — longer, more detailed; good if you have time

### Study Resources by Type
- **Whitepapers:** [Disaster Recovery](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html), [Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- **FAQs:** [S3](https://aws.amazon.com/s3/faqs/), [EC2](https://aws.amazon.com/ec2/faqs/), [RDS](https://aws.amazon.com/rds/faqs/), [DynamoDB](https://aws.amazon.com/dynamodb/faqs/) — skim before Week 7
- **Decision Guides:** [AWS Decision Guides](https://docs.aws.amazon.com/decision-guides/latest/) — when-to-choose-X comparisons

### Practice Test Strategy
- **Week 5 onwards:** Take 30–50 question practice tests after each domain
- **Week 11–13:** Take full-length 65-question practice exams from AWS Skill Builder
- **Review ruthlessly:** Read the explanation for every question, right or wrong
- **Track weak topics:** Keep a spreadsheet of missed questions by topic; review before exam

---

## Key Formulas & Numbers to Memorise

| Item | Value | Context |
|------|-------|---------|
| SAA-C03 pass score | 720 / 1000 | Scaled score; ~72% raw |
| Exam questions | 65 (50 scored, 15 unscored) | 130 minutes |
| Domain 1 weight | 30% | Security |
| Domain 2 weight | 26% | Resilience |
| Domain 3 weight | 24% | Performance |
| Domain 4 weight | 20% | Cost |
| **DR: Backup & Restore** | **Hours (RPO/RTO)** | Cheapest |
| **DR: Pilot Light** | **15–30 min** | Minimal env |
| **DR: Warm Standby** | **5 min** | Scaled-down |
| **DR: Active-Active** | **~0 min** | Most expensive |
| Aurora throughput | **5x faster than RDS** | Standard MySQL/PostgreSQL |
| DynamoDB latency | **Milliseconds** | NoSQL key-value |
| ElastiCache latency | **Sub-millisecond** | In-memory cache |
| Lambda timeout | **15 minutes max** | Serverless function |
| Spot discount | **Up to 90%** | On-Demand cost |
| RI discount (1-year) | **~40%** | On-Demand cost |
| RI discount (3-year) | **~60%** | On-Demand cost |
| Savings Plans discount | **40–60%** | On-Demand cost |

---

## Final Checklist Before Exam

- [ ] You've scored 75%+ on at least 2 AWS Skill Builder practice exams
- [ ] You can name the 4 DR strategies and their RPO/RTO
- [ ] You know when to use ALB vs NLB, SQS vs SNS, RDS vs Aurora vs DynamoDB
- [ ] You understand cross-account IAM (roles, trust policies, assume role)
- [ ] You know the difference between Multi-AZ (HA) and read replicas (scaling + standby)
- [ ] You can explain when to choose On-Demand, Reserved, Savings Plans, or Spot
- [ ] You've reviewed at least 100 practice questions
- [ ] You know the Well-Architected Framework 5 pillars (Security, Reliability, Performance, Cost, Ops)
- [ ] You're comfortable with elimination technique; you won't second-guess correct answers

**Good luck! The SAA-C03 tests architecture judgment, not trivia. If you understand the trade-offs, you'll pass.**
