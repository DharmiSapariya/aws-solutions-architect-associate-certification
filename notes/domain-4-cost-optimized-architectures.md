# Domain 4 · Design Cost-Optimized Architectures (20%)

Task statements: (4.1) cost-optimized **storage**, (4.2) cost-optimized **compute**, (4.3) cost-optimized **database**, (4.4) cost-optimized **network** architectures.

---
## Core principles
- Pay only for what you use; right-size; match purchasing model to workload predictability; use managed/serverless to cut ops cost; delete idle resources; measure with tags.

## Cost management tools
| Tool | Purpose |
|---|---|
| **Cost Explorer** | Visualize and forecast spend, RI/SP recommendations |
| **AWS Budgets** | Alerts + actions when cost/usage/coverage exceeds threshold |
| **Cost and Usage Report (CUR)** | Most detailed data → S3/Athena |
| **Cost Allocation Tags** | User-defined & AWS-generated tags for chargeback |
| **Cost Anomaly Detection** | ML-based unexpected-spend alerts |
| **Compute Optimizer** | Right-sizing for EC2, EBS, Lambda, ASG |
| **Trusted Advisor** | Cost checks (idle LB, underused EC2, unattached EBS/EIP) |
| **Pricing Calculator** | Estimate before deploy |
| **Organizations consolidated billing** | Aggregate usage for volume discounts, share RIs/SPs |
| **Savings Plans / RI recommendations** | In Cost Explorer |

---
## 4.1 Storage cost optimization

### S3
- **Lifecycle rules**: transition Standard → IA → Glacier tiers, expire old versions, abort incomplete multipart uploads, expire delete markers.
- **Intelligent-Tiering** when access pattern unknown (small monitoring fee per object; objects < 128 KB not auto-tiered).
- Compress objects, use **Parquet**, avoid many tiny files in IA/Glacier (minimum billable size 128 KB for IA classes).
- **Requester Pays** buckets shift transfer cost to the downloader.
- **S3 Storage Lens** for usage analytics; **S3 Storage Class Analysis** to decide on IA transition.
- Delete retired versions when versioning is on.

### EBS / EFS / backups
- Delete **unattached volumes** and old snapshots (snapshots are incremental; use **Data Lifecycle Manager**; **Snapshot Archive** is ~75% cheaper for long retention).
- **gp3** is ~20% cheaper than gp2 and decoupled IOPS; st1/sc1 for throughput/cold workloads.
- **EFS Infrequent Access / Archive** with lifecycle management; **One Zone** classes for non-critical data (~47% cheaper).
- Use **AWS Backup** lifecycle to move backups to cold storage (≥ 90-day retention minimum).
- **FSx** data deduplication and compression; use HDD for less-demanding Windows file shares.

### Data transfer into AWS
- Transfer **in** is free. Choose DataSync/Snow Family when cost/time of network is high.

---
## 4.2 Compute cost optimization

### EC2 purchasing options
| Option | Discount | Commitment | Best for |
|---|---|---|---|
| **On-Demand** | 0 | None | Short, unpredictable, dev/test |
| **Reserved Instances** – Standard | up to ~72% | 1 or 3 yr (No/Partial/All upfront), specific family/region | Steady-state, predictable |
| RI – Convertible | up to ~54% | 1 or 3 yr | Flexibility to change family/OS |
| **Compute Savings Plans** | up to ~66% | $/hr 1 or 3 yr | EC2 (any family/region/OS), **Fargate, Lambda** |
| **EC2 Instance Savings Plans** | up to ~72% | $/hr, specific family in a region | Steady EC2 in known family |
| **Spot Instances** | up to ~90% | None (2-minute interruption notice) | Fault-tolerant, stateless, batch, CI, big data, containers workers |
| **Dedicated Hosts** | – | Per host (On-Demand or Reserved) | BYOL (per-socket/core), compliance, host affinity |
| **Dedicated Instances** | – | Hardware dedicated to you | Compliance isolation |
| **Capacity Reservations** | none (pay On-Demand) | Any duration | Guarantee capacity in an AZ (combine with Savings Plans for discount) |
| **Scheduled / On-Demand Capacity Reservation** | – | – | Predictable windows |

- **Spot Fleet / EC2 Fleet / ASG mixed instances**: diversify instance types & AZs; **capacity-optimized** allocation strategy lowers interruption.
- **Spot Block** is discontinued. **Spot is not suitable** for databases or anything that can't tolerate interruption (unless checkpointed).
- **Standard RIs can be sold on RI Marketplace**; Convertible cannot.
- Regional RI = AZ-flexible, no capacity reservation; Zonal RI = capacity reserved.

### Reduce compute spend
- **Right-size** with Compute Optimizer; stop dev/test at night (**Instance Scheduler**); **Auto Scaling** to follow demand.
- **Graviton** instances ~20% cheaper / up to 40% better price-performance.
- **Serverless**: Lambda (pay per ms; ARM saves ~20%), Fargate (pay per vCPU/GB-sec), Fargate Spot (~70% off for interruptible tasks), Aurora Serverless, DynamoDB on-demand.
- **Lambda cost tuning**: right-size memory (**Lambda Power Tuning**), avoid idle waiting (use Step Functions), batch SQS events.
- **Containers**: ECS on Spot capacity providers; consolidate with bin packing.
- **T-family burstable** for low steady use; avoid unlimited-mode surprise charges.
- **Elastic Beanstalk/App Runner** reduce operational cost.
- Savings Plans > RIs when you want flexibility across services.

---
## 4.3 Database cost optimization

- **Right-size** instances (Performance Insights, Compute Optimizer); stop non-prod RDS (auto-restart after 7 days) and use snapshots.
- **Reserved Instances** for RDS / ElastiCache / Redshift / OpenSearch / DynamoDB reserved capacity.
- **Aurora Serverless v2** (pay per ACU-second) for variable workloads; **Aurora I/O-Optimized** when I/O > ~25% of cost; Aurora Standard otherwise.
- **DynamoDB**: On-demand for unpredictable / low-volume, Provisioned + auto scaling for steady; **Standard-IA table class** (≈60% lower storage cost) for rarely read data; use **TTL** to delete expired items free; eventually consistent reads (½ cost); batch ops; avoid scans, use projections.
- **Caching** (ElastiCache, DAX) reduces read load → smaller DB.
- **Read replicas** only when needed; Multi-AZ only for production.
- **Redshift**: pause/resume, RA3 managed storage, Spectrum for cold data on S3, Reserved nodes; **Serverless** for intermittent.
- **Backup retention** tuned; clean old manual snapshots.
- **DMS** for cheap migration; consider **open-source engines** (PostgreSQL/MySQL/Aurora) over commercial licensing (Oracle/SQL Server) — **License Included vs BYOL**.
- **Athena** (pay per TB scanned) beats running a cluster for ad-hoc queries → store as Parquet + partition.

---
## 4.4 Network cost optimization

### Data transfer pricing rules of thumb
| Transfer | Cost |
|---|---|
| Internet → AWS (inbound) | **Free** |
| Within same AZ using **private IP** | Free |
| Between AZs in same Region | ~$0.01/GB each direction |
| Between Regions | ~$0.02/GB (varies) |
| AWS → Internet | Charged per GB (tiered); CloudFront egress cheaper |
| Via **public IP / Elastic IP** between instances | Charged even in same AZ |
| S3 → CloudFront | Free (origin fetch) |
| S3 → same-region EC2 | Free |

### Reduce network cost
- Use **CloudFront** to cut egress and origin load (and cheaper than direct internet egress).
- **VPC Gateway endpoints** (S3, DynamoDB) are **free** and avoid NAT charges; NAT Gateway charges hourly + per GB processed → route AWS-service traffic via endpoints.
- Co-locate chatty tiers in the same AZ where HA allows; understand NLB cross-zone charges.
- **Direct Connect** reduces egress per GB vs internet for steady large volumes (but has port-hour fees); **Transit Gateway** has per-attachment + per-GB fees — use **VPC peering** for a few VPCs, TGW for many.
- **S3 Transfer Acceleration** costs extra — only when needed.
- **Release unused Elastic IPs** (idle public IPv4 now billed hourly, all public IPv4 addresses are charged); consider IPv6.
- Right-size NAT: one NAT per AZ for HA but a single NAT for dev/test; consider **NAT instance** for tiny workloads.
- **Compress** responses, use caching headers.
- **Use ALB instead of multiple CLBs** (host/path routing consolidates).

---
## Cost-reduction quick matrix
| Situation | Cheapest sensible answer |
|---|---|
| Batch image processing, interruption tolerated | **Spot** + SQS |
| 24/7 steady DB 3 years | **Reserved Instance / Savings Plan** |
| Mixed EC2 + Lambda + Fargate steady commit | **Compute Savings Plan** |
| Infrequent S3 access, unpredictable | **Intelligent-Tiering** |
| Archive rarely retrieved, hours OK | **Glacier Flexible / Deep Archive** |
| Spiky web traffic | **Auto Scaling + ALB / Lambda + API Gateway** |
| Bursty DB, unpredictable | **Aurora Serverless v2 / DynamoDB on-demand** |
| Private S3 access from VPC, reduce NAT cost | **Gateway endpoint** |
| Ad-hoc S3 queries | **Athena** |
| Cost visibility per team | **Cost allocation tags + Budgets** |
| Licensing per core compliance | **Dedicated Hosts** |
| Dev environments off-hours | **Instance Scheduler / stop** |

## Exam traps
- Savings Plans ≠ capacity reservation. Zonal RI or Capacity Reservation is needed to guarantee capacity.
- Spot with persistent request + `stop`/`hibernate` behaviour; Spot cannot run databases that cannot tolerate interruption.
- "Least operational AND cost" typically favors serverless managed services.
- Don't pick multi-region architectures when requirements don't demand it — they multiply cost.
