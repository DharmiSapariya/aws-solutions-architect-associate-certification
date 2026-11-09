# Domain 3 · Design High-Performing Architectures (24%)

Task statements: (3.1) high-performing and/or scalable **storage**, (3.2) high-performing and elastic **compute**, (3.3) high-performing **database** solutions, (3.4) high-performing and/or scalable **network** architectures, (3.5) **data ingestion and transformation** solutions.

---
## 3.1 Storage

### S3
- **Performance**: ≥ 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second **per prefix**; parallelize across prefixes.
- **Multipart upload**: recommended > 100 MB, required > 5 GB (max object 5 TB); use **byte-range fetches** for parallel downloads.
- **S3 Transfer Acceleration**: uploads over CloudFront edge → AWS backbone (long-distance clients).
- **S3 Select / Glacier Select**: server-side filtering of CSV/JSON/Parquet.
- **Athena** queries S3 with SQL; use Parquet/ORC + partitioning to reduce scan cost.
- **Storage classes**:

| Class | Use | Min duration | Retrieval |
|---|---|---|---|
| Standard | Frequent access | – | ms |
| Intelligent-Tiering | Unknown/changing access | – | ms (+ optional archive tiers) |
| Standard-IA | Infrequent, rapid access, multi-AZ | 30 days | ms, per-GB fee |
| One Zone-IA | Re-creatable infrequent data, single AZ | 30 days | ms |
| Glacier Instant Retrieval | Archive, quarterly access | 90 days | ms |
| Glacier Flexible Retrieval | Archive | 90 days | Expedited 1–5 min, Standard 3–5 h, Bulk 5–12 h |
| Glacier Deep Archive | Cheapest, long-term (7–10+ yrs) | 180 days | Standard 12 h, Bulk 48 h |
| Express One Zone | Single-digit-ms, high-perf, single AZ | – | ms |

### Block storage — EBS
| Type | Class | Notes |
|---|---|---|
| **gp3** | SSD general | 3,000 IOPS / 125 MB/s baseline, scale IOPS & throughput independently (up to 16,000 IOPS, 1,000 MB/s) |
| gp2 | SSD general | IOPS tied to size (3 IOPS/GB), burst credits |
| **io1 / io2 / io2 Block Express** | Provisioned IOPS SSD | Up to 64,000 (io1/io2) / 256,000 (Block Express) IOPS, **Multi-Attach**, latency-sensitive DBs |
| **st1** | Throughput HDD | Big sequential, streaming, logs, data warehouse — cannot be boot |
| **sc1** | Cold HDD | Lowest cost, infrequent — cannot be boot |
- **Instance store**: physically attached NVMe, highest IOPS, ephemeral (lost on stop/terminate/failure) — caches, buffers, scratch data.
- **EBS-optimized instances** for dedicated EBS bandwidth; **RAID 0** to increase performance across volumes.

### File storage
| Service | Use |
|---|---|
| **EFS** | Linux NFS shared across AZs/instances; modes: General Purpose vs Max I/O; throughput: Elastic / Provisioned / Bursting; classes Standard / IA / Archive; lifecycle |
| **FSx for Windows** | SMB, AD integration, DFS, Windows workloads |
| **FSx for Lustre** | HPC, ML, sub-ms latency, hundreds of GB/s, S3 integration (scratch vs persistent) |
| **FSx for NetApp ONTAP** | Multi-protocol (NFS, SMB, iSCSI), migrate NetApp |
| **FSx for OpenZFS** | Migrate ZFS workloads, low latency |
| **Storage Gateway** | Hybrid: **File Gateway** (NFS/SMB→S3), **Volume Gateway** (iSCSI; cached vs stored), **Tape Gateway** (VTL→S3/Glacier) |
| **DataSync** | Online transfer to/from on-prem NFS/SMB, S3, EFS, FSx; scheduled, bandwidth-optimised |
| **Transfer Family** | Managed SFTP/FTPS/FTP into S3/EFS |
| **Snow Family** | Snowcone (8–14 TB), **Snowball Edge** (80 TB storage-optimized / compute-optimized), Snowmobile (EB scale; retired for new use) — offline migration when network too slow |

---
## 3.2 Compute

### EC2 instance families
| Family | Optimized for |
|---|---|
| **T** (burstable) | Low baseline, CPU credits (T3 unlimited mode) |
| **M** | General purpose balance |
| **C** | Compute-intensive: batch, HPC, gaming, ML inference |
| **R / X / z1d** | Memory-intensive: in-memory DBs, caches, SAP HANA |
| **I / D / H** | Storage-optimized: high IOPS NoSQL, data warehouses |
| **P / G / Inf / Trn** | GPU / accelerators: ML training, graphics, inference |
| **Graviton (suffix g)** | ARM, better price-performance |
- **Nitro System**, **ENA** enhanced networking, **EFA** (HPC/MPI), **Elastic Fabric Adapter**.
- **Hibernate** saves RAM to encrypted EBS root for fast resume.

### Choosing a compute model
| Requirement | Pick |
|---|---|
| Full OS control, lift-and-shift | EC2 |
| Event-driven, < 15 min, no servers | Lambda |
| Containers without managing servers | ECS/EKS on **Fargate** |
| Kubernetes portability | EKS |
| Simple web app deployment | Elastic Beanstalk / App Runner |
| Batch jobs | AWS Batch |
| On-prem consistency | Outposts |
| Edge low latency | CloudFront Functions / Lambda@Edge / Local Zones / Wavelength |

- **Lambda perf tuning**: memory scales CPU; **Provisioned Concurrency** removes cold starts; avoid VPC cold-start issues (Hyperplane ENIs now mitigate); use **RDS Proxy** to pool DB connections; **Lambda@Edge** (CloudFront, up to 30 s origin events) vs **CloudFront Functions** (JS, sub-ms, viewer events only).
- **Auto Scaling** with target tracking on CPU/ALB request count/SQS depth.

---
## 3.3 Databases

### Pick the right database
| Need | Service |
|---|---|
| Relational OLTP, managed | **RDS** (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Db2) |
| Cloud-native relational, 5× MySQL / 3× PostgreSQL perf | **Aurora** (+ Serverless v2 for unpredictable loads) |
| Key-value/document, single-digit ms at any scale | **DynamoDB** |
| Microsecond reads for DynamoDB | **DAX** |
| In-memory cache / session store / leaderboards | **ElastiCache** (Redis: persistence, replication, data structures, pub/sub; Memcached: simple, multi-thread, no persistence) |
| Durable Redis-compatible primary DB | **MemoryDB** |
| Data warehouse OLAP, columnar | **Redshift** (Spectrum to query S3, RA3, concurrency scaling) |
| Graph | **Neptune** |
| MongoDB-compatible | **DocumentDB** |
| Cassandra-compatible | **Keyspaces** |
| Ledger, immutable, cryptographically verifiable | **QLDB** (being retired — consider Aurora PostgreSQL ledger patterns) |
| Time series | **Timestream** |
| Search / log analytics | **OpenSearch Service** |
| Custom engine/OS access | RDS Custom or EC2 |

### DynamoDB deep dive
- **Capacity modes**: Provisioned (RCU/WCU, auto scaling, reserved capacity) vs **On-Demand** (unpredictable traffic).
- 1 RCU = one strongly consistent read/s up to 4 KB (2 eventually consistent); 1 WCU = one write/s up to 1 KB. Transactions cost 2×.
- **Partition key design**: high cardinality, uniform access; avoid hot partitions (adaptive capacity helps).
- **Indexes**: LSI (same partition key, defined at table creation, 10 GB limit per partition key) vs **GSI** (any keys, own capacity, eventually consistent).
- **Streams** (24 h) → Lambda; **TTL** free deletion; **Global Tables**; **PITR**; **PartiQL**; **Export to S3**.
- Item size limit 400 KB — store large objects in S3 with pointer.

### RDS / Aurora performance
- Scale up (instance class), scale out (read replicas), **RDS Proxy**, **Performance Insights**, **Enhanced Monitoring**, storage autoscaling, Provisioned IOPS storage, parameter groups.
- Aurora: reader endpoint, custom endpoints, **Aurora Parallel Query**, **Serverless v2** (ACU scaling), **I/O-Optimized** configuration for heavy I/O.
- **Database Migration Service (DMS)** + **Schema Conversion Tool (SCT)**: heterogeneous migrations (e.g. Oracle→Aurora PostgreSQL); CDC for ongoing replication; homogeneous = DMS only.

### Caching strategies
- **Lazy loading** (cache-aside): only cached on miss → stale risk, miss penalty.
- **Write-through**: update cache on write → no stale data, cache churn.
- Add TTL to bound staleness.

---
## 3.4 Network

### Edge & global delivery
- **CloudFront**: CDN, caches static/dynamic content, TTLs, cache policies, invalidations, signed URLs/cookies, field-level encryption, geo-restriction, origin shield, HTTPS, Lambda@Edge. Cheaper egress than S3→internet.
- **Global Accelerator** (static anycast IPs, TCP/UDP, dynamic content, gaming/VoIP, fast regional failover) vs **CloudFront** (cacheable HTTP content).
- **Route 53** latency-based routing for multi-region performance.

### VPC connectivity
| Need | Service |
|---|---|
| Encrypted, quick, over internet | **Site-to-Site VPN** (2 tunnels, up to ~1.25 Gbps/tunnel; accelerated VPN via Global Accelerator) |
| Dedicated private, consistent latency, high bandwidth | **Direct Connect** (1/10/100 Gbps dedicated; hosted 50 Mbps–10 Gbps+); provisioning takes weeks |
| Connect many VPCs & on-prem | **Transit Gateway** (hub-and-spoke, multicast, inter-region peering) |
| 2 VPCs, simple | VPC Peering |
| Expose a service to many consumers | **PrivateLink** |
| Remote user access | **Client VPN** |
| Private access to AWS services | VPC endpoints |
| IPv6 outbound only | Egress-only Internet Gateway |
| Inspect/route 3rd-party appliances | Gateway Load Balancer |

- **ENA / EFA**, **Placement group (cluster)** for lowest latency, **jumbo frames** (MTU 9001 within VPC).
- **Elastic IP** (static public IPv4); **Direct Connect Gateway** (many VPCs across regions); **Direct Connect + VPN** for encryption.

---
## 3.5 Data ingestion, transformation & analytics

| Task | Service |
|---|---|
| Real-time stream ingestion | **Kinesis Data Streams** / MSK |
| Load streams into S3/Redshift/OpenSearch | **Kinesis Data Firehose** |
| Real-time analytics on streams | **Managed Service for Apache Flink** (formerly Kinesis Data Analytics) |
| Serverless ETL + Data Catalog | **AWS Glue** (crawlers, jobs, DataBrew for no-code prep, Glue Data Catalog) |
| SQL on S3, serverless | **Athena** |
| Big data (Spark/Hadoop/Hive) | **EMR** (EC2/EKS/Serverless) |
| BI dashboards | **QuickSight** (SPICE, ML insights) |
| Build a data lake with governance | **Lake Formation** (fine-grained access) on S3 + Glue |
| Move/replicate databases | **DMS** |
| Visual ETL | Glue Studio |
| Orchestrate pipelines | Step Functions, MWAA (Airflow), Glue workflows |
| Search/log analytics | OpenSearch |

- **Data formats**: columnar (Parquet/ORC) → cheaper/faster analytics; compress; partition by date/key.
- **Kinesis shard math**: producers 1 MB/s or 1,000 records/s per shard; consumers 2 MB/s per shard (shared) or 2 MB/s per consumer with enhanced fan-out. Hot shard → better partition key / split shard.

---
## Exam traps & quick answers
- "Lowest latency shared file system for HPC" → **FSx for Lustre**.
- "Windows shared storage with AD" → **FSx for Windows**.
- "Highest IOPS, ephemeral OK" → **Instance store**.
- "Database needs > 64,000 IOPS, mission-critical" → **io2 Block Express**.
- "Millions of reqs/s DynamoDB with microsecond reads" → **DAX**.
- "Unpredictable DB traffic, minimal ops" → **Aurora Serverless v2 / DynamoDB on-demand**.
- "Global users, static + dynamic content, TCP/UDP, fixed IPs" → **Global Accelerator**.
- "Cache static content close to users" → **CloudFront**.
- "Serverless SQL on S3 logs" → **Athena**.
- "Transform and deliver streaming data to S3 with minimal admin" → **Firehose**.
- "Migrate 100 TB with weak internet" → **Snowball Edge**.
- "On-prem needs local cache of S3 data via NFS" → **S3 File Gateway**.
