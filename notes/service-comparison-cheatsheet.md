# Service Comparison Cheat Sheet

## Storage
| Need | Pick | Not |
|---|---|---|
| Object, unlimited, durable | S3 | EBS |
| Block for one instance | EBS | EFS |
| Shared Linux filesystem | EFS | EBS (except Multi-Attach io) |
| Shared Windows (SMB, AD) | FSx for Windows | EFS |
| HPC / ML scratch | FSx for Lustre | EFS |
| Ephemeral fastest | Instance store | EBS |
| Hybrid file → S3 | File Gateway | DataSync (one-off/scheduled transfer) |
| Hybrid block (iSCSI) | Volume Gateway | – |
| Tape replacement | Tape Gateway | – |

## Messaging
| Need | Pick |
|---|---|
| Queue / buffering | SQS |
| Strict order + dedup | SQS FIFO |
| Fan-out | SNS (+SQS) |
| Event routing, SaaS events, schedule | EventBridge |
| High-throughput stream, replay | Kinesis Data Streams |
| Stream → S3 w/ no code | Firehose |
| Existing JMS/AMQP broker | Amazon MQ |
| Kafka | MSK |
| Workflow | Step Functions |

## Load balancer
| Need | Pick |
|---|---|
| HTTP routing | ALB |
| Static IP, TCP/UDP, extreme perf | NLB |
| Network appliance | GWLB |

## Database
| Need | Pick |
|---|---|
| OLTP SQL | RDS / Aurora |
| NoSQL key-value | DynamoDB |
| Cache | ElastiCache / DAX |
| OLAP | Redshift |
| Graph | Neptune |
| Document (Mongo) | DocumentDB |
| Time series | Timestream |
| Search | OpenSearch |

## Networking
| Need | Pick |
|---|---|
| Private to AWS service via ENI | Interface endpoint / PrivateLink |
| Free private to S3/DynamoDB | Gateway endpoint |
| VPC ↔ VPC (few) | Peering |
| VPC hub | Transit Gateway |
| On-prem fast setup | Site-to-Site VPN |
| On-prem dedicated | Direct Connect |
| Global static IP entry | Global Accelerator |
| CDN | CloudFront |
| DNS | Route 53 |

## Security
| Need | Pick |
|---|---|
| Secrets rotation | Secrets Manager |
| Config values | Parameter Store |
| Keys managed | KMS |
| Dedicated HSM | CloudHSM |
| TLS certs | ACM |
| L7 filtering | WAF |
| DDoS | Shield (Advanced) |
| Threat detection | GuardDuty |
| Vulnerabilities | Inspector |
| PII in S3 | Macie |
| API audit | CloudTrail |
| Config compliance | Config |
| Aggregated findings | Security Hub |
| Org guardrails | SCP / Control Tower |
| Workforce SSO | IAM Identity Center |
| App-user auth | Cognito |

## Migration
| Need | Pick |
|---|---|
| Discover on-prem | Application Discovery Service, Migration Hub |
| Lift-and-shift servers | Application Migration Service (MGN) |
| DB migration | DMS (+SCT) |
| File transfer | DataSync / Transfer Family |
| Offline bulk | Snow Family |
| Disaster recovery | Elastic Disaster Recovery |
| The 7 Rs | Retire, Retain, Rehost, Relocate, Repurchase, Replatform, Refactor |

## Management
| Need | Pick |
|---|---|
| Metrics/alarms/logs | CloudWatch |
| Distributed tracing | X-Ray |
| IaC | CloudFormation / CDK |
| Patch/run commands/sessions | Systems Manager |
| Account events | Health Dashboard |
| Fleet governance | Organizations / Control Tower |
| Cost | Cost Explorer / Budgets |
