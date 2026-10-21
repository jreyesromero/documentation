# Domain 3: Design High-Performing Architectures (24%)

This domain focuses on **selecting the right services and configurations for speed and throughput**. Performance means low latency, high throughput, and efficient use of resources.

---

## Focus Areas

**Storage Selection**
- S3: object storage; classes (Standard, Intelligent-Tiering, Infrequent Access, Glacier, Deep Archive); lifecycle
- EBS: block storage for EC2; SSD (gp3, io2) vs HDD (st1, sc1); provisioned IOPS
- EFS: multi-AZ NFS; shared across EC2 instances; pay per GB used
- FSx: Windows file shares (SMB), Lustre (HPC), NetApp (enterprise NAS), OpenZFS

**Compute Options**
- EC2: instance families (general, compute-optimized, memory-optimized, GPU); placement groups
- Lambda: event-driven, pay per invocation + memory; 15-minute timeout
- Fargate: containers as a service; no EC2 to manage
- ECS: AWS-native container orchestration
- EKS: managed Kubernetes (less exam-heavy for performance)
- Elastic Beanstalk: PaaS; abstracts away infra

**Databases**
- RDS: managed relational (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server); multi-AZ, read replicas
- Aurora: high-performance MySQL/PostgreSQL-compatible; auto-scaling replicas; 5x faster than RDS MySQL
- DynamoDB: serverless NoSQL; millisecond latency; partition keys, sort keys; GSI/LSI
- ElastiCache: in-memory cache (Redis, Memcached); sub-millisecond latency; session store, caching layer
- Redshift: columnar data warehouse; 10x faster than traditional data warehouses for analytics

**Content Delivery & Global Performance**
- CloudFront: CDN for low-latency content delivery; edge locations; caching, compression
- Global Accelerator: AWS backbone routing; ideal for non-HTTP protocols; UDP traffic
- Route 53: DNS with latency/geolocation routing; health checks

**Data Ingestion & Processing**
- Kinesis Streams: real-time data ingestion; shards, partition keys; millisecond latency
- Kinesis Firehose: near-real-time data delivery to S3, Redshift, Splunk; simpler than Streams
- Glue: serverless ETL; data catalog; spark jobs
- Athena: SQL queries on S3; pay per GB scanned

---

## Documentation Links

### Storage
- [S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [EBS Volume Types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [EFS](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [FSx](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)

### Databases
- [Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)
- [DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html)
- [RDS FAQs](https://aws.amazon.com/rds/faqs/), [Aurora FAQs](https://aws.amazon.com/rds/aurora/faqs/), [DynamoDB FAQs](https://aws.amazon.com/dynamodb/faqs/)

### Content Delivery & Global
- [CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)

### Data Ingestion
- [Kinesis Streams](https://docs.aws.amazon.com/streams/latest/dev/introduction.html)
- [Glue](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html)
- [Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)

---

## Key Decision Points

### "RDS, Aurora, or DynamoDB for this workload?"
**RDS** (MySQL, PostgreSQL, etc.): Relational data, complex queries, ACID. Single-AZ or Multi-AZ. Managed but slower than Aurora for high volume.
**Aurora**: MySQL/PostgreSQL-compatible, 5x faster reads, auto-scaling read replicas. Choice for relational + performance. Higher cost.
**DynamoDB**: Serverless NoSQL, millisecond latency, unlimited scale. No schema, partition/sort keys only. Best for simple key-value or event log patterns.
**Rule:** Relational + high performance → Aurora. Serverless + NoSQL → DynamoDB. Traditional relational → RDS.

### "CloudFront or Global Accelerator?"
**CloudFront**: Caches content (static, dynamic); reduces origin load; integrates with WAF; good for web content.
**Global Accelerator**: No caching; routes traffic via AWS backbone; ideal for non-HTTP (UDP, real-time), or HTTP with custom routing (latency-based). Different use case — not a caching substitute.

### "ElastiCache: Redis or Memcached?"
**Redis**: Richer data types (lists, sets, sorted sets, streams); persistence, replication, Cluster mode for horizontal scaling.
**Memcached**: Simpler, lower latency, multi-threaded (better CPU utilization). No persistence. Good for simple session store, cache-aside pattern.
**Choice:** Redis for complex caching + persistence; Memcached for high-throughput simple caching.

### "EBS: gp3, gp2, io2, st1, sc1?"
**gp3**: General purpose; 3 IOPS per GB, 125 MB/s throughput; burstable; good default.
**gp2**: Older; burst for short periods. Avoid if baseline IOPS needed.
**io2**: High IOPS; 64K IOPS per volume (vs 16K for io1); databases, low-latency transactional.
**st1** (throughput): HDD, 250 MB/s; good for big data, sequential (Hadoop, log processing).
**sc1** (cold): HDD, 80 MB/s; archival, infrequent access.
**Rule:** Use gp3 for most; io2 for extreme IOPS; st1 for sequential; sc1 for cold storage.

### "Kinesis Streams or Firehose?"
**Streams**: Real-time, millisecond; you manage shards and consumer applications. For real-time processing (Lambda, EC2 consumers).
**Firehose**: Near-real-time (60–300 sec buffer); delivers to S3/Redshift/Splunk automatically; easier to operate.
**Choice:** Streams if you need sub-second latency or custom consumers; Firehose for data warehouse delivery.

---

## Exam Scenarios

**Scenario 1: High-performance transactional database**
Application: payment processing, 10K TPS, <100ms latency requirement. Use Aurora MySQL (5x faster than RDS), Multi-AZ for HA, read replicas for read scaling. Provision provisioned IOPS on EBS. ElastiCache for session store.

**Scenario 2: Global content delivery + low latency**
Website serving users worldwide. CloudFront distributes static assets (images, CSS, JS) from edge locations. Origin Shield for additional caching layer between edge and origin. Route 53 latency-based routing for dynamic content. Result: <50ms latency from edge, reduces origin load.

**Scenario 3: Real-time analytics pipeline**
IoT sensors → Kinesis Streams → Lambda (real-time aggregation) → DynamoDB (time-series data) or Redshift (analytics). Kinesis Firehose parallel path for S3 backup. Athena queries S3 for ad-hoc analytics.

**Scenario 4: Storage tiering for cost + performance**
Frequently accessed data: S3 Standard. After 30 days: S3 Intelligent-Tiering (auto-moves between tiers). After 90 days: S3 Glacier. After 1 year: S3 Glacier Deep Archive. Lifecycle policy automates transitions.

---

## Common Exam Pitfalls

- **Assuming CloudFront replaces Global Accelerator:** CloudFront caches; Global Accelerator routes. Different problems.
- **Confusing DynamoDB provisioned vs on-demand:** Provisioned: pay for capacity (RCU/WCU). On-demand: pay per request. On-demand simpler for bursty workloads; provisioned cheaper for consistent traffic.
- **Forgetting ElastiCache requires security group + subnet:** ElastiCache nodes must be in your VPC; ingress from app security group required.
- **RDS Multi-AZ ≠ read scaling:** Multi-AZ provides HA (failover); it doesn't balance reads. Use read replicas for read scaling.

---

**Next:** [Domain 4 — Cost](./04-cost.md)
