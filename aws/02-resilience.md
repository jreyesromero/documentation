# Domain 2: Design Resilient Architectures (26%)

This domain focuses on **decoupling, scaling, and disaster recovery**. The exam tests whether you can build systems that remain available despite failures — the core SRE skill.

---

## Focus Areas

**Decoupling & Events (SQS, SNS, EventBridge, Step Functions, API Gateway)**
- SQS: decouple producers/consumers; standard vs FIFO; visibility timeout, dead-letter queues
- SNS: pub/sub for fan-out messaging; topic subscriptions (HTTP, email, Lambda, SQS)
- EventBridge: event bus for cross-service events; rules, targets
- Step Functions: stateful workflow orchestration
- API Gateway: REST/HTTP API gateway, throttling, caching

**Scaling & Load Balancing**
- Auto Scaling: horizontal scaling based on metrics; termination policies; lifecycle hooks
- ALB (Application Load Balancer): Layer 7, path/hostname routing; sticky sessions
- NLB (Network Load Balancer): Layer 4, ultra-high throughput; connection draining
- ELB (Classic): legacy; don't memorise

**Compute & Containers**
- Lambda: event-driven, scale to zero, 15-minute timeout limit
- Fargate: containers without managing EC2
- ECS: AWS-native orchestration; task vs service
- EKS: managed Kubernetes (out of scope for core coverage but appears in cost/performance Qs)
- EC2: when you need persistent state or custom OS

**High Availability & Multi-AZ**
- Multi-AZ (synchronous replication, RTO ~1 min): RDS, Aurora, ALB
- Read replicas (asynchronous, lag tolerance): RDS, Aurora, DynamoDB global tables
- Route 53: health checks, failover policies
- ElastiCache: Multi-AZ for Redis

**Disaster Recovery**
- **4 DR strategies** — know these perfectly:
  1. **Backup & Restore** — RPO/RTO: hours. Restore from S3/Glacier backup.
  2. **Pilot Light** — RPO/RTO: 15–30 min. Minimal env always running; scale on failover.
  3. **Warm Standby** — RPO/RTO: minutes. Full env scaled-down; scale up on failover.
  4. **Active-Active** — RPO/RTO: near-zero. Full env in each region; traffic to both.

---

## Documentation Links

### Decoupling & Messaging
- [Disaster Recovery Strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html) — whitepaper with all 4 strategies
- [SQS FAQ](https://aws.amazon.com/sqs/faqs/)
- [SNS Guide](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)

### Scaling & Load Balancing
- [Elastic Load Balancing Types](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-types.html)
- [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
- [Route 53 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)

---

## Key Decision Points

### "SQS or SNS? Or both?"
**SQS** for point-to-point: one producer, one consumer (or many consumers pulling from queue).
**SNS** for pub/sub: one producer, many subscribers. Multiple subscriptions to same topic.
**Both together**: Pub to SNS topic, subscribe SQS queue to topic → decoupled fan-out to many queues.

### "Synchronous (Multi-AZ) or asynchronous (read replica)?"
**Multi-AZ** (synchronous): RTO ~1 min (automatic failover), RPO near-zero. Costs ~2x.
**Read replica** (asynchronous): Can have lag; used for read scaling, not HA by itself (but can promote on failure). RPO = lag.
**Choose:** Multi-AZ for critical databases; read replicas for scaling reads or standby in another region.

### "ALB or NLB?"
**ALB**: Path/hostname routing, good for microservices. Layer 7 (understands HTTP).
**NLB**: Ultra-high throughput, low latency, non-HTTP protocols. Layer 4 (connection-based).
**Choose:** ALB for typical apps; NLB for extreme performance (gaming, IoT, non-HTTP protocols).

### "Lambda or containers? Containers or EC2?"
**Lambda**: Stateless, event-driven, <15 min execution; scales to zero. Lowest ops overhead.
**Fargate/ECS**: Stateless containers, simpler than EKS; still serverless (no EC2 to manage). Good for long-running services.
**ECS vs EKS**: ECS is AWS-native, simpler; EKS is Kubernetes (portable, industry standard). Both use Fargate for serverless containers.
**EC2**: Persistent state, custom OS, long-running background processes, bursting compute needs.

### "Which DR strategy: pilot light, warm standby, or active-active?"
**Backup & Restore**: Acceptable for non-critical, low RTO tolerance (<1 hour). Very cheap.
**Pilot Light**: Good for RTO ~15–30 min. Minimal standby running (DNS, minimal DB replicas). Quick scale-up.
**Warm Standby**: RTO ~5 min. Full env scaled down. Best for "reasonable cost, low RTO" requirement.
**Active-Active**: RTO ~0, RPO ~0. Expensive (duplicate infra). Required for "zero downtime" mission-critical systems.

---

## Exam Scenarios

**Scenario 1: E-commerce order processing — decoupled system**
Customer submits order → API Gateway → Lambda → SQS (for order queue). Separate worker Lambda consumes from SQS, processes, updates database and SNS topic. SNS topic has subscriptions for email notification and inventory update. If inventory service is down, order still queued; worker retries.

**Scenario 2: Multi-region failover with Route 53**
Primary region: ALB + ASG. Secondary region: ALB + ASG (warm standby). Route 53 health check on primary ALB; if primary fails, DNS switches to secondary (failover policy). RTO: time for Route 53 propagation (~30 sec) + health check interval.

**Scenario 3: Auto Scaling with custom metrics**
Application tracks business-level metric (e.g., messages in queue). CloudWatch metric published to `OrderQueueDepth`. Auto Scaling policy: scale when `OrderQueueDepth > 100` or CPU > 70%. Step scaling: add 2 instances per 50 messages in queue.

**Scenario 4: Multi-AZ RDS with read replicas**
Primary in AZ-a, Multi-AZ standby in AZ-b (automatic failover, sync replication). Additional read replica in AZ-c for read scaling (async replication). Standby is never used for reads (not promoted until failover). Read replica can be promoted to standalone if needed.

---

## Common Exam Pitfalls

- **Confusing Multi-AZ with read replicas:** Multi-AZ is for HA (automatic failover, sync); read replicas are for read scaling and cross-region standby (async, can have lag).
- **Forgetting Route 53 health checks:** DNS failover doesn't happen until health check fails; the check interval (10 sec default) adds to RTO.
- **Misunderstanding SQS visibility timeout:** Messages are invisible during processing; if the worker crashes before deleting the message, it reappears after the timeout. Longer timeout = fewer duplicates, but slower failure detection.
- **Active-Active ≠ every region:** Active-Active means all regions serve traffic simultaneously; Active-Standby means one region is passive until failover.

---

**Next:** [Domain 3 — Performance](./03-performance.md)
