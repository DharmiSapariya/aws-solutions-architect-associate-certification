# Common Scenarios & Reference Architectures

## 1. Highly available 3-tier web app
Route 53 → CloudFront (+WAF) → ALB (public subnets, ≥ 2 AZs) → ASG of EC2 / Fargate (private subnets) → Aurora Multi-AZ (isolated subnets) + ElastiCache. Static assets in S3 via CloudFront OAC. Sessions in ElastiCache/DynamoDB. NAT per AZ. Secrets Manager for DB creds.

## 2. Serverless REST API
CloudFront → API Gateway (Cognito authorizer, WAF, throttling) → Lambda → DynamoDB (DAX if needed). Async: SQS/SNS/EventBridge. Logs: CloudWatch; tracing: X-Ray.

## 3. Event-driven image pipeline
S3 upload → S3 event → SQS → Lambda (resize) → S3 output + DynamoDB metadata. DLQ for failures. Fan-out via SNS for notifications.

## 4. Real-time analytics
Producers → Kinesis Data Streams → (Flink for real-time aggregations, Lambda for custom) → Firehose → S3 (Parquet) → Glue Catalog → Athena → QuickSight. Alternatively Firehose → Redshift/OpenSearch.

## 5. Data lake
S3 (raw/curated zones) + Glue crawlers/ETL + Lake Formation (permissions) + Athena/Redshift Spectrum/EMR + QuickSight. Encrypt with KMS; Macie for PII.

## 6. Hybrid connectivity
On-prem ↔ Direct Connect (primary) + Site-to-Site VPN (backup) → Transit Gateway → VPCs. DNS via Route 53 Resolver endpoints. Storage via File/Volume Gateway or DataSync.

## 7. Multi-account landing zone
Organizations + Control Tower; OUs (Security, Infrastructure, Workloads-Prod/Dev, Sandbox); SCP guardrails; org CloudTrail to log-archive account; Config aggregator; GuardDuty/Security Hub delegated admin; IAM Identity Center for access; RAM for shared subnets/TGW.

## 8. Multi-region DR (active-passive)
Route 53 failover + health checks; Aurora Global Database or cross-region read replica; S3 CRR; DynamoDB Global Tables; AMIs/snapshots copied; IaC templates deployed to DR region; Pilot Light/Warm Standby.

## 9. Lift-and-shift migration
Discovery (Application Discovery Service) → MGN (rehost) → DMS/SCT for DB → DataSync/Snow for bulk data → cutover via Route 53 weighted → optimise later (replatform/refactor).

## 10. Decoupling a monolith
Break out components behind SQS; replace polling with events; move session state out; containerize to ECS/Fargate; strangler-fig pattern with ALB path routing.

## 11. Secure private workload
No public IPs; SSM Session Manager; VPC endpoints for S3/DynamoDB/SSM/ECR/Secrets Manager; NACL + SG; KMS CMKs; Flow Logs; GuardDuty; Config rules; IAM roles only.

## 12. Static website
S3 (private) + CloudFront (OAC, ACM cert in us-east-1) + Route 53 alias + WAF optional; Lambda@Edge/CloudFront Functions for rewrites.

## Anti-patterns the exam punishes
- Storing state/sessions on instance disks in an ASG.
- Long-lived IAM user keys on EC2 (use roles).
- Single AZ for production.
- Tightly coupled synchronous chains without queues.
- Over-provisioning for peak when auto scaling/serverless fits.
- Public S3 buckets / open 0.0.0.0/0 security groups for admin ports.
