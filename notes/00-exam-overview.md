# 00 · Exam Overview — AWS Certified Solutions Architect – Associate (SAA-C03)

## Exam facts
| Item | Detail |
|---|---|
| Code | SAA-C03 |
| Format | 65 questions (50 scored + 15 unscored), multiple choice / multiple response |
| Duration | 130 minutes |
| Scoring | Scaled 100–1000, **720 to pass**, compensatory (no per-domain pass mark) |
| Level | Associate — ≥ 1 year designing solutions on AWS recommended |

## Domain weighting
| Domain | Weight |
|---|---|
| 1. Design Secure Architectures | 30% |
| 2. Design Resilient Architectures | 26% |
| 3. Design High-Performing Architectures | 24% |
| 4. Design Cost-Optimized Architectures | 20% |

## The mindset the exam tests
Every question is a trade-off among **security, resilience, performance, cost, and operational overhead**. Look for keywords:

| Keyword in question | Usually points to |
|---|---|
| "least operational overhead" | Managed / serverless (Lambda, Fargate, Aurora Serverless, DynamoDB) |
| "most cost-effective" | Spot, Savings Plans, S3 tiering, right-sizing, serverless for spiky loads |
| "highly available" | Multi-AZ, ELB, Auto Scaling |
| "disaster recovery / region failure" | Multi-Region, cross-region replication, Route 53 failover |
| "decouple" | SQS, SNS, EventBridge |
| "least privilege" | Narrow IAM policies, roles (never long-lived access keys) |
| "real-time streaming" | Kinesis Data Streams / Firehose |
| "millisecond latency, read-heavy" | ElastiCache / DAX / CloudFront |
| "on-premises hybrid, low latency to AWS" | Direct Connect (VPN as backup / quick start) |

## Elimination strategy
1. Remove answers that break a stated requirement (region, compliance, latency).
2. Remove answers that add needless infrastructure to manage.
3. Between two valid answers, pick the one using the **native managed service**.
4. Watch absolutes: "always", "only", "never" are often wrong.

## Repo map
- [Domain 1 – Secure](domain-1-secure-architectures.md)
- [Domain 2 – Resilient](domain-2-resilient-architectures.md)
- [Domain 3 – High-Performing](domain-3-high-performing-architectures.md)
- [Domain 4 – Cost-Optimized](domain-4-cost-optimized-architectures.md)
- [Service comparison cheat sheet](service-comparison-cheatsheet.md)
- [Well-Architected Framework](well-architected-framework.md)
- [Common scenarios & patterns](scenarios-and-patterns.md)
- [Study plan & exam-day tips](study-plan-and-exam-tips.md)
