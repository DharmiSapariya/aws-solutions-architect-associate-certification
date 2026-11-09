# Study Plan & Exam-Day Tips

## 6-week plan (≈ 8–10 hrs/week)
| Week | Focus | Hands-on |
|---|---|---|
| 1 | IAM, Organizations, VPC fundamentals, EC2 basics | Build custom VPC, public/private subnets, NAT, SG/NACL |
| 2 | Storage (S3, EBS, EFS, FSx), security (KMS, WAF, GuardDuty) | S3 policies/lifecycle/replication, encrypted EBS |
| 3 | Databases (RDS, Aurora, DynamoDB, ElastiCache), caching | Multi-AZ RDS, read replica, DynamoDB GSI/streams |
| 4 | ELB, ASG, Route 53, CloudFront, serverless (Lambda, API GW, SQS, SNS) | ALB + ASG, CloudFront + S3 OAC, SQS→Lambda |
| 5 | DR, migration, analytics (Kinesis, Glue, Athena), cost, Well-Architected | Route 53 failover, Kinesis→Firehose→S3→Athena |
| 6 | Practice exams, weak-area review | Review wrong answers; re-read cheat sheet |

## Resources
- AWS Exam Guide & sample questions (SAA-C03), AWS Skill Builder, AWS Well-Architected whitepaper, service FAQs (S3, EC2, VPC, RDS, Aurora, DynamoDB, ELB), re:Invent talks, hands-on labs, reputable practice exams.

## Exam-day tactics
- 65 questions / 130 min ≈ 2 min each; flag and move on, return if time remains.
- Read the **last sentence** (what's asked) first, then scan for constraints.
- Multiple-response questions: count how many to select; each must be independently correct.
- Prefer managed/native/serverless; avoid custom code when a service exists.
- Unanswered = wrong; always guess.
- Remember: unscored questions (15) are mixed in — don't panic at unfamiliar topics.

## Last-day review checklist
- [ ] SG vs NACL · Gateway vs Interface endpoints · NAT per AZ
- [ ] RDS Multi-AZ vs read replica · Aurora Global · DynamoDB Global Tables
- [ ] SQS Standard vs FIFO · SNS fan-out · Kinesis shards
- [ ] Route 53 policies · Global Accelerator vs CloudFront
- [ ] S3 storage classes · lifecycle · Object Lock · CRR requirements
- [ ] KMS key types · envelope encryption · CloudHSM
- [ ] EC2 purchasing options · Spot use cases · Savings Plans types
- [ ] DR strategies (RPO/RTO) · Direct Connect vs VPN
- [ ] Well-Architected 6 pillars
