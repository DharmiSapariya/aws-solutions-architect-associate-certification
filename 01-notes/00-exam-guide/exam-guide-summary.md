# SAA-C03 Exam Guide — Quick Reference

*Summary of the official Exam Guide v1.0. Always check the latest guide on the AWS site.*

## Exam facts
| Item | Detail |
|------|--------|
| Question types | Multiple choice, multiple response |
| Total questions | 65 (50 scored + 15 unscored, unmarked) |
| Time | 130 minutes |
| Scoring | Scaled 100–1,000, **passing = 720** |
| Model | Compensatory: pass the overall exam, not each section |
| Guessing | No penalty, unanswered = incorrect |
| Target candidate | 1+ year hands-on designing AWS solutions |

## Domains and task statements

| Domain | Weight |
|--------|--------|
| 1. Design Secure Architectures | 30% |
| 2. Design Resilient Architectures | 26% |
| 3. Design High-Performing Architectures | 24% |
| 4. Design Cost-Optimized Architectures | 20% |

### Domain 1: Secure
- 1.1 Design secure access to AWS resources (IAM, MFA, STS, cross-account, Control Tower, SCPs, federation)
- 1.2 Design secure workloads and applications (VPC security, SG/NACL, NAT, Shield, WAF, Secrets Manager, VPN/Direct Connect)
- 1.3 Determine appropriate data security controls (KMS, ACM/TLS, key policies, backups, lifecycle, rotation)

### Domain 2: Resilient
- 2.1 Design scalable and loosely coupled architectures (API Gateway, SQS/SNS, Lambda/Fargate, ECS/EKS, Step Functions, caching, read replicas)
- 2.2 Design highly available and/or fault-tolerant architectures (Multi-AZ/Region, Route 53, DR: backup-restore / pilot light / warm standby / active-active, RPO/RTO, RDS Proxy, X-Ray)

### Domain 3: High-Performing
- 3.1 High-performing and/or scalable **storage** (S3, EFS, EBS, hybrid)
- 3.2 High-performing and elastic **compute** (Batch, EMR, Fargate, Auto Scaling, Lambda memory sizing)
- 3.3 High-performing **database** solutions (engine choice, Aurora vs DynamoDB, ElastiCache, read replicas, capacity planning)
- 3.4 High-performing and/or scalable **network** architectures (CloudFront, Global Accelerator, ALB, VPN/Direct Connect/PrivateLink)
- 3.5 High-performing **data ingestion and transformation** (Athena, Lake Formation, QuickSight, DataSync, Storage Gateway, Glue, Kinesis, EMR, csv→parquet)

### Domain 4: Cost-Optimized
- 4.1 Cost-optimized **storage** (S3 tiers/lifecycle, Requester Pays, HDD vs SSD, FSx/EFS/EBS, data transfer)
- 4.2 Cost-optimized **compute** (Spot, Reserved, Savings Plans, instance family/size, ALB vs NLB vs GWLB, hibernation, Outposts/Snowball Edge)
- 4.3 Cost-optimized **database** (DynamoDB vs RDS, serverless, backup/retention, DMS migrations)
- 4.4 Cost-optimized **network** (NAT gateway vs instance, one NAT vs per-AZ, Direct Connect vs VPN, Transit Gateway, VPC endpoints, CDN)

## Scope notes
- In scope: broad list across analytics, app integration, compute, containers, database, ML (Comprehend, Polly, Rekognition, etc.), governance, migration, networking, security, serverless, storage.
- Out of scope (examples): all IoT, CDK, CodeBuild/CodeDeploy/CodeCommit, Lightsail, App Mesh, Cloud Map, Elemental media services, most niche ML services.
