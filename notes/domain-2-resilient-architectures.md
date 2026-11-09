# Domain 2 · Design Resilient Architectures (26%)

Task statements: (2.1) design scalable and loosely coupled architectures, (2.2) design highly available and/or fault-tolerant architectures.

---
## 2.1 Scalable & loosely coupled architectures

### Global infrastructure
- **Region** = geographic area; **AZ** = one or more discrete data centres with independent power/network; **Edge locations / PoPs** for CloudFront, Route 53, Global Accelerator; **Local Zones**, **Wavelength**, **Outposts** for low latency/hybrid.
- Region choice factors: compliance/data residency, latency, service availability, price.

### Decoupling & messaging
| Service | Model | Key facts |
|---|---|---|
| **SQS Standard** | Queue (pull) | Unlimited throughput, at-least-once, best-effort ordering, retention 1 min–14 days (default 4 days), 256 KB message, visibility timeout (default 30 s) |
| **SQS FIFO** | Queue | Exactly-once, strict ordering per **message group ID**, 300 msg/s (3,000 with batching; higher with high-throughput mode), name must end `.fifo` |
| **SQS DLQ** | Failure handling | Redrive messages that exceed `maxReceiveCount` |
| **SQS long polling** | Cost/latency | `WaitTimeSeconds` up to 20 s reduces empty receives |
| **SNS** | Pub/sub (push) | Fan-out to SQS, Lambda, HTTP, email, SMS, Firehose; filter policies; FIFO topics |
| **EventBridge** | Event bus | Rules, schedules, SaaS + AWS service events, schema registry, archive/replay, cross-account |
| **Kinesis Data Streams** | Streaming | Shards (1 MB/s in, 2 MB/s out), retention 24 h–365 d, replay, ordering per partition key, **enhanced fan-out** |
| **Kinesis Data Firehose** | Delivery | Near real-time load to S3/Redshift/OpenSearch/Splunk, optional Lambda transform, fully managed |
| **Amazon MQ** | Managed ActiveMQ/RabbitMQ | Migrating apps using JMS/AMQP/MQTT without rewrite |
| **Step Functions** | Orchestration | Standard (long, exactly-once) vs Express (high-volume, short) workflows, retries, error handling |
| **AppFlow / MSK** | SaaS integration / managed Kafka | |

**Fan-out pattern**: SNS topic → multiple SQS queues → independent consumers.
**Buffering pattern**: API/ingest → SQS → Auto Scaling consumers (scale on `ApproximateNumberOfMessages / instance`).

### Serverless & microservices
- **Lambda**: up to 15 min, 10 GB memory, 10 GB `/tmp` (configurable), concurrency limits (reserved vs provisioned), event source mappings (SQS, Kinesis, DynamoDB Streams), **SnapStart**, VPC attach, layers, container images.
- **API Gateway**: REST / HTTP / WebSocket APIs; throttling, caching, usage plans & API keys, Cognito/Lambda authorizers, edge-optimized vs regional vs private endpoints; 29 s integration timeout.
- **AppSync**: managed GraphQL.
- **ECS** (EC2 or **Fargate**), **EKS** (managed Kubernetes), **App Runner**, **ECR**. Fargate = no servers to manage.
- **Elastic Beanstalk**: PaaS for web apps; **Lightsail**: simple VPS.
- **Batch**: queued batch jobs on managed compute (can use Spot).

### Scaling compute
- **Auto Scaling groups**: min/desired/max, health checks (EC2 + ELB), cooldown, lifecycle hooks, warm pools, instance refresh, mixed instances policy (On-Demand + Spot).
- **Scaling policies**: target tracking (simplest), step, simple, **scheduled** (known patterns), **predictive**.
- **Launch templates** (versioned; replace launch configurations).
- Stateless web tier: store sessions in **ElastiCache / DynamoDB**, uploads in **S3**, not on instance.

### Load balancers
| | ALB | NLB | GWLB | CLB |
|---|---|---|---|---|
| Layer | 7 (HTTP/HTTPS/gRPC/WebSocket) | 4 (TCP/UDP/TLS) | 3 (GENEVE) | legacy |
| Features | path/host/header/query routing, redirect, fixed response, Lambda targets, OIDC/Cognito auth, WAF | Static IP / Elastic IP per AZ, millions req/s, ultra-low latency, preserves client IP, PrivateLink backend | Insert 3rd-party virtual appliances (firewall/IDS) transparently | avoid |
- Cross-zone load balancing: on by default for ALB (free), off by default for NLB (inter-AZ data charges).
- **Connection draining / deregistration delay**, **sticky sessions** (cookie), **SNI** for multiple certs.

### Storage as a decoupling/scale layer
- **S3** (11 9's durability, strong read-after-write consistency, unlimited scale), **EFS** (shared POSIX NFS across AZs, auto-scaling), **FSx** (Windows File Server, Lustre, NetApp ONTAP, OpenZFS).

### Caching to reduce load
- CloudFront, ElastiCache (Redis / Memcached), DAX, API Gateway cache, Aurora read replicas.

---
## 2.2 Highly available & fault-tolerant architectures

### Terminology
- **High availability** = minimize downtime (Multi-AZ). **Fault tolerance** = continue operating through failure with zero interruption. **Durability** = data not lost.
- **RPO** (max data loss) and **RTO** (max downtime) drive strategy.

### Disaster recovery strategies (cheapest → fastest)
| Strategy | RPO/RTO | Description | Cost |
|---|---|---|---|
| **Backup & Restore** | Hours | Backups to S3 / cross-region copy; rebuild with IaC | $ |
| **Pilot Light** | Tens of minutes | Core data replicated + minimal always-on core; scale up on failover | $$ |
| **Warm Standby** | Minutes | Scaled-down full copy running | $$$ |
| **Multi-Site Active/Active** | Near-zero | Full capacity in 2+ Regions, Route 53 / Global Accelerator routing | $$$$ |

### Database resilience
| Service | HA / DR feature |
|---|---|
| **RDS Multi-AZ** | Synchronous standby, automatic failover (60–120 s), same region, *not* for read scaling; backups from standby |
| **RDS Multi-AZ DB cluster** | 1 writer + 2 readable standbys, faster failover |
| **RDS Read Replicas** | Async, up to 15 (5 for non-Aurora engines like MySQL/MariaDB/PostgreSQL), cross-region possible, promotable, read scaling |
| **Aurora** | 6 copies across 3 AZs, storage auto-grows to 128 TiB, up to 15 replicas, failover < 30 s typically, **Aurora Global Database** (RPO ~1 s, cross-region RTO < 1 min), **Aurora Serverless v2**, Backtrack (MySQL) |
| **DynamoDB** | Multi-AZ by default, **Global Tables** (multi-region active-active), PITR 35 days, on-demand backups, Streams, DAX |
| **ElastiCache Redis** | Multi-AZ with auto-failover, replicas, Global Datastore |
| **Redshift** | Snapshots cross-region, Multi-AZ RA3 |
| **RDS Backups** | Automated (1–35 days, PITR), manual snapshots (retained until deleted), cross-region snapshot copy |

### Compute resilience
- Spread ASG across ≥ 2 AZs; ELB health checks; replace unhealthy instances automatically.
- **EC2 Auto Recovery** (single instance, same IP/ENI/EBS), **Placement groups**: Cluster (low latency, single AZ), Spread (max 7 per AZ, separate hardware), Partition (large distributed systems like HDFS/Kafka/Cassandra).
- **EBS**: single AZ — protect with snapshots (incremental, stored in S3, copy across region), **Multi-Attach** (io1/io2, same AZ), **Data Lifecycle Manager** automates snapshots.
- **EFS**: Regional (Multi-AZ) vs One Zone; replication to another region.
- **S3**: Cross-Region Replication (CRR) / Same-Region Replication (SRR) — versioning required on both buckets, only new objects (Batch Replication for existing), **Replication Time Control** (15 min SLA), Multi-Region Access Points.
- **AWS Elastic Disaster Recovery (DRS)**: continuous block-level replication of on-prem/other-cloud servers to AWS staging area.

### Networking resilience
- **Route 53 routing policies**:
  - Simple, **Weighted** (A/B, canary), **Latency**, **Failover** (active-passive with health checks), **Geolocation** (by user country), **Geoproximity** (bias, via Traffic Flow), **Multi-value answer** (up to 8 healthy records), IP-based.
  - Alias records (free, root/apex domain, AWS resources) vs CNAME (not at apex).
  - Health checks: endpoint, calculated, CloudWatch alarm (for private resources).
- **Global Accelerator**: 2 static anycast IPs, AWS backbone, health-based failover in seconds, no DNS-cache issue — good for TCP/UDP non-HTTP and fast failover.
- **CloudFront**: **origin groups** for origin failover.
- **Connectivity redundancy**: 2 Direct Connect connections at separate locations, or DX + Site-to-Site VPN backup; VPN with 2 tunnels; **Transit Gateway** + ECMP; **DX Gateway** for multi-region/VPC.
- **NAT Gateway**: AZ-scoped → one per AZ.

### Infrastructure as code & operations
- **CloudFormation** (templates, stacks, StackSets, change sets, drift detection, rollbacks, `DeletionPolicy`), **CDK**, **Service Catalog**.
- Immutable / blue-green deployments: Elastic Beanstalk, CodeDeploy (in-place vs blue/green; canary/linear), Route 53 weighted, ALB target group weights.
- Monitoring: **CloudWatch** (metrics, alarms, logs, dashboards, composite alarms, Logs Insights, EventBridge), **X-Ray** tracing, **Health Dashboard** (account-specific events).

### Resilience service limits to remember
- SQS FIFO ordering is per message group; Standard queue may deliver duplicates → make consumers idempotent.
- Lambda async invocations retry twice; use DLQ / on-failure destination.
- S3 doesn't fail over automatically across regions; combine CRR + Route 53 / MRAP.

---
## Exam traps & quick answers
- "Application needs to survive AZ failure with automatic failover, relational DB" → **RDS Multi-AZ / Aurora**.
- "Scale reads" → **Read replicas / ElastiCache**; "Scale writes" → sharding / Aurora Limitless / DynamoDB.
- "Cross-region DR for DynamoDB, active-active" → **Global Tables**.
- "Cross-region DR for Aurora, low RPO" → **Aurora Global Database**.
- "Decouple order processing, ensure no message lost, ordering needed" → **SQS FIFO**.
- "One event, many subscribers" → **SNS fan-out (+SQS)**.
- "Lowest RTO for non-HTTP traffic, static IPs" → **Global Accelerator**.
- "Failover between primary region and secondary" → **Route 53 failover routing with health checks**.
- "Minimal cost DR with RTO of hours" → **Backup & restore**.
