# AWS Services Reference — SAA-C03 In-Scope Services

**Priority Legend:**
- **[High]**: Study in depth; likely to appear in multiple exam questions
- **[Med]**: Know what it does and when to use it; will appear in scenario questions
- **[Rec]**: Recognise the name, purpose, and basic use case; may appear in one or two questions

**Note:** The exam guide appendix is explicitly non-exhaustive. Services are not equally weighted — focus your study on [High] and [Med] priorities. [Rec] services are often there to test whether you can quickly eliminate incorrect options.

---

## Compute

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon EC2 | Virtual machines; families, types, sizes, placement groups | **[High]** | https://aws.amazon.com/ec2/ |
| EC2 Auto Scaling | Automatic horizontal scaling based on metrics; termination policies; lifecycle hooks | **[High]** | https://aws.amazon.com/ec2/autoscaling/ |
| AWS Lambda | Event-driven serverless compute; <15 min timeout; scale to zero | **[High]** | https://aws.amazon.com/lambda/ |
| AWS Fargate | Containers without managing servers; serverless container runtime for ECS/EKS | **[High]** | https://aws.amazon.com/fargate/ |
| AWS Auto Scaling | Unified scaling across EC2, Fargate, RDS, DynamoDB, ECS | **[Med]** | https://aws.amazon.com/autoscaling/ |
| AWS Batch | Batch processing at scale; job queues, compute environments | **[Med]** | https://aws.amazon.com/batch/ |
| AWS Elastic Beanstalk | Platform-as-a-Service app deployment; abstracts infrastructure | **[Med]** | https://aws.amazon.com/elasticbeanstalk/ |
| AWS Outposts | AWS infrastructure on-premises for hybrid architectures | **[Med]** | https://aws.amazon.com/outposts/ |
| VMware Cloud on AWS / Wavelength / Serverless App Repository | Niche; recognise the names | **[Rec]** | https://aws.amazon.com/products/compute/ |

---

## Containers

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon ECS | AWS-native container orchestration; Fargate or EC2 launch types | **[High]** | https://aws.amazon.com/ecs/ |
| Amazon EKS | Managed Kubernetes; portable, industry-standard container orchestration | **[High]** | https://aws.amazon.com/eks/ |
| Amazon ECR | Container image registry; private, push/pull images for ECS/EKS | **[Med]** | https://aws.amazon.com/ecr/ |
| ECS Anywhere / EKS Anywhere / EKS Distro | Hybrid/on-premises container variants; recognise | **[Rec]** | https://aws.amazon.com/eks/eks-anywhere/ |

---

## Storage

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon S3 | Object storage; classes (Standard, IA, Glacier); lifecycle; versioning | **[High]** | https://aws.amazon.com/s3/ |
| Amazon EBS | Block storage for EC2; SSD (gp3, io2) and HDD (st1, sc1) types | **[High]** | https://aws.amazon.com/ebs/ |
| Amazon EFS | Elastic multi-AZ NFS file storage; shared across EC2 instances | **[High]** | https://aws.amazon.com/efs/ |
| Amazon FSx | Managed file systems (Windows, Lustre, NetApp, OpenZFS) | **[High]** | https://aws.amazon.com/fsx/ |
| Amazon S3 Glacier | Archive storage; Glacier, Glacier Deep Archive (cheap long-term) | **[High]** | https://aws.amazon.com/s3/storage-classes/glacier/ |
| AWS Storage Gateway | Hybrid on-premises ↔ AWS storage; file, volume, or tape gateway | **[High]** | https://aws.amazon.com/storagegateway/ |
| AWS Backup | Centralised cross-service backups; policies, vaults | **[High]** | https://aws.amazon.com/backup/ |

---

## Databases

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon RDS | Managed relational databases (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server); Multi-AZ, read replicas | **[High]** | https://aws.amazon.com/rds/ |
| Amazon Aurora | High-performance MySQL/PostgreSQL-compatible; 5x faster; auto-scaling read replicas | **[High]** | https://aws.amazon.com/rds/aurora/ |
| Amazon DynamoDB | Serverless NoSQL; millisecond latency; partition keys; on-demand or provisioned billing | **[High]** | https://aws.amazon.com/dynamodb/ |
| Amazon ElastiCache | In-memory cache (Redis, Memcached); sub-millisecond latency; session store, caching layer | **[High]** | https://aws.amazon.com/elasticache/ |
| Aurora Serverless | On-demand Aurora capacity; auto-pause when unused; pay per second | **[Med]** | https://aws.amazon.com/rds/aurora/serverless/ |
| Amazon Redshift | Columnar data warehouse; analytics at scale; 10x faster queries | **[Med]** | https://aws.amazon.com/redshift/ |
| DocumentDB / Neptune / Keyspaces / QLDB / Timestream | Specialised databases (document, graph, Cassandra, ledger, time-series); know the use case | **[Med]** | https://aws.amazon.com/products/databases/ |

---

## Networking and Content Delivery

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon VPC | Virtual network; subnets, routing tables, NAT, VPC endpoints, security | **[High]** | https://aws.amazon.com/vpc/ |
| Amazon Route 53 | DNS and routing/failover policies (geolocation, latency, weighted, failover, multi-value) | **[High]** | https://aws.amazon.com/route53/ |
| Amazon CloudFront | CDN; caches content at edge locations; reduces latency and origin load | **[High]** | https://aws.amazon.com/cloudfront/ |
| Elastic Load Balancing | ALB (L7, path/hostname routing) / NLB (L4, ultra-high throughput) / GWLB | **[High]** | https://aws.amazon.com/elasticloadbalancing/ |
| AWS Global Accelerator | AWS backbone routing; ideal for non-HTTP protocols; UDP, gaming, IoT | **[High]** | https://aws.amazon.com/global-accelerator/ |
| AWS PrivateLink | Private access to services without internet; endpoint services | **[High]** | https://aws.amazon.com/privatelink/ |
| AWS Direct Connect | Dedicated on-premises ↔ AWS link; consistent network performance | **[High]** | https://aws.amazon.com/directconnect/ |
| AWS VPN | Encrypted tunnel over internet; site-to-site or client-to-site | **[High]** | https://aws.amazon.com/vpn/ |
| AWS Transit Gateway | Hub for interconnecting VPCs and on-premises networks; simplified network management | **[High]** | https://aws.amazon.com/transit-gateway/ |

---

## Application Integration and Messaging

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon SQS | Message queues; decoupling (standard vs FIFO); visibility timeout; dead-letter queues | **[High]** | https://aws.amazon.com/sqs/ |
| Amazon SNS | Pub/sub messaging; fan-out to multiple subscribers (email, HTTP, Lambda, SQS) | **[High]** | https://aws.amazon.com/sns/ |
| Amazon EventBridge | Event bus; cross-service event routing; rules and targets | **[High]** | https://aws.amazon.com/eventbridge/ |
| Amazon API Gateway | REST/HTTP API gateway; throttling, caching, authentication, CORS | **[High]** | https://aws.amazon.com/api-gateway/ |
| AWS Step Functions | Workflow orchestration; state machines; visual monitoring | **[High]** | https://aws.amazon.com/step-functions/ |
| Amazon MQ | Managed ActiveMQ/RabbitMQ; for migrations from on-premises brokers | **[Med]** | https://aws.amazon.com/amazon-mq/ |
| AWS AppFlow | SaaS data integration; bi-directional sync with Salesforce, etc. | **[Med]** | https://aws.amazon.com/appflow/ |
| AWS AppSync | Managed GraphQL APIs; real-time subscriptions | **[Med]** | https://aws.amazon.com/appsync/ |

---

## Analytics and Data Ingestion

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| Amazon Kinesis | Real-time data streaming; shards, partition keys; Streams (millisecond), Firehose (near-real-time) | **[High]** | https://aws.amazon.com/kinesis/ |
| AWS Glue | Serverless ETL; data catalog; Spark jobs; schema discovery | **[High]** | https://aws.amazon.com/glue/ |
| Amazon Athena | SQL queries on S3; pay per GB scanned; interactive analytics | **[High]** | https://aws.amazon.com/athena/ |
| Amazon EMR | Managed big data (Spark, Hadoop, Presto); cluster-based analytics | **[Med]** | https://aws.amazon.com/emr/ |
| AWS Lake Formation | Build and govern data lakes; centralized catalog and access control | **[Med]** | https://aws.amazon.com/lake-formation/ |
| Amazon MSK | Managed Apache Kafka; streaming platform; high throughput | **[Med]** | https://aws.amazon.com/msk/ |
| Amazon OpenSearch Service | Search and log analytics; successor to Elasticsearch | **[Med]** | https://aws.amazon.com/opensearch-service/ |
| Amazon QuickSight | BI and visualisation; dashboards and embedded analytics | **[Med]** | https://aws.amazon.com/quicksight/ |
| AWS Data Exchange / AWS Data Pipeline | Third-party data and orchestration; recognise | **[Rec]** | https://aws.amazon.com/data-exchange/ |

---

## Security, Identity and Compliance

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| AWS IAM | Identity, policies, roles, STS, cross-account access; **exam-critical** | **[High]** | https://aws.amazon.com/iam/ |
| AWS IAM Identity Center (SSO) | Federated access across accounts; AD/SAML integration; multi-account SSO | **[High]** | https://aws.amazon.com/iam/identity-center/ |
| AWS KMS | Key management; encryption at rest; key policies, key rotation | **[High]** | https://aws.amazon.com/kms/ |
| AWS Certificate Manager (ACM) | TLS certificates; encryption in transit; auto-renewal; integrates with ALB/CloudFront | **[High]** | https://aws.amazon.com/certificate-manager/ |
| AWS Secrets Manager | Secrets management; automatic rotation; versioning; audit trail | **[High]** | https://aws.amazon.com/secrets-manager/ |
| Amazon Cognito | Identity for apps and users; user pools, identity pools; social login | **[High]** | https://aws.amazon.com/cognito/ |
| AWS WAF | Web application firewall; Layer 7 protection against SQL injection, XSS; integrates with ALB, CloudFront, API Gateway | **[High]** | https://aws.amazon.com/waf/ |
| AWS Shield | DDoS protection; Shield Standard (free, automatic) and Shield Advanced ($3K/month) | **[High]** | https://aws.amazon.com/shield/ |
| Amazon GuardDuty | Threat detection; ML-based analysis of logs and flows; findings + alerts | **[High]** | https://aws.amazon.com/guardduty/ |
| Amazon Macie | Sensitive-data discovery in S3; PII detection; automated alerts | **[High]** | https://aws.amazon.com/macie/ |
| AWS RAM | Resource sharing; share resources (RDS, Lambda, EC2, VPC) across accounts | **[Med]** | https://aws.amazon.com/ram/ |
| AWS Directory Service | Managed Active Directory; AD Connector (on-prem sync); Cloud Directory | **[Med]** | https://aws.amazon.com/directoryservice/ |
| AWS Firewall Manager / Network Firewall | Network management; distributed firewall rules; IDS/IPS | **[Med]** | https://aws.amazon.com/network-firewall/ |
| Amazon Inspector / Security Hub / Detective | Assessment, posture management, and investigation; know the differences | **[Med]** | https://aws.amazon.com/security-hub/ |
| AWS CloudHSM / Audit Manager / Artifact | Dedicated HSM, audit compliance, compliance artifact storage; recognise | **[Rec]** | https://aws.amazon.com/cloudhsm/ |

---

## Management and Governance

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| AWS Organizations | Multi-account management; SCPs, billing consolidation | **[High]** | https://aws.amazon.com/organizations/ |
| AWS Control Tower | Governed multi-account landing zone; automated setup, guardrails, compliance | **[High]** | https://aws.amazon.com/controltower/ |
| Amazon CloudWatch | Metrics, logs, alarms, dashboards; unified observability | **[High]** | https://aws.amazon.com/cloudwatch/ |
| AWS CloudTrail | API-call auditing; compliance logging; trail creation and analysis | **[High]** | https://aws.amazon.com/cloudtrail/ |
| AWS CloudFormation | Infrastructure as code; templates, stacks, nested stacks; drift detection | **[High]** | https://aws.amazon.com/cloudformation/ |
| AWS Config | Configuration inventory and compliance; rules engine; remediation | **[High]** | https://aws.amazon.com/config/ |
| AWS Trusted Advisor | Recommendations; cost, security, performance, fault tolerance, service quotas | **[High]** | https://aws.amazon.com/premiumsupport/technology/trusted-advisor/ |
| AWS Systems Manager | Operations and parameters; Parameter Store, Session Manager, Automation, Patch Manager | **[Med]** | https://aws.amazon.com/systems-manager/ |
| AWS Compute Optimizer | Rightsizing recommendations for EC2, EBS, Lambda, RDS | **[Med]** | https://aws.amazon.com/compute-optimizer/ |
| AWS Service Catalog / License Manager | Product catalog and licensing; recognise | **[Rec]** | https://aws.amazon.com/servicecatalog/ |
| Managed Grafana / Prometheus / Proton / Health Dashboard / CLI / WA Tool | Observability and utilities; recognise | **[Rec]** | https://aws.amazon.com/products/management-and-governance/ |

---

## Migration and Transfer

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| AWS DMS | Database migration; homogeneous and heterogeneous | **[High]** | https://aws.amazon.com/dms/ |
| AWS DataSync | Large-scale online data transfer; between on-prem and AWS, or between AWS services | **[High]** | https://aws.amazon.com/datasync/ |
| AWS Snow Family | Offline data transfer via physical device (Snowball, Snowmobile); petabyte-scale | **[High]** | https://aws.amazon.com/snow/ |
| AWS Transfer Family | Managed SFTP/FTPS/FTP to S3/EFS; legacy protocol support | **[High]** | https://aws.amazon.com/aws-transfer-family/ |
| Migration Hub / Application Migration / Application Discovery / SMS | Discovery and workload migration; know each role | **[Med]** | https://aws.amazon.com/migration-hub/ |

---

## Developer Tools, ML, Media, and Front-End

| Service | One-line description | Priority | Link |
|---------|---------------------|----------|------|
| AWS X-Ray | Distributed tracing; latency debugging; service map | **[Med]** | https://aws.amazon.com/xray/ |
| Machine Learning services | Rekognition (images), Polly (TTS), Transcribe (STT), Comprehend (NLP), Translate, Textract, Forecast, Kendra, Lex, SageMaker | **[Rec]** | https://aws.amazon.com/machine-learning/ |
| Elastic Transcoder / Kinesis Video Streams | Media processing; recognise | **[Rec]** | https://aws.amazon.com/media-services/ |
| Amplify / Pinpoint / Device Farm | Front-end and mobile; recognise | **[Rec]** | https://aws.amazon.com/products/front-end-web-mobile/ |

---

**Study Strategy:** Master all [High] services. Know what [Med] services do and when you'd use them. For [Rec] services, focus on *eliminating wrong answers* — you won't need to choose them, but you might need to recognise them in a distractor option.
