# AWS Well-Architected Framework (6 Pillars)

| Pillar | Goal | Design principles | Key services |
|---|---|---|---|
| **Operational Excellence** | Run and monitor systems, improve continuously | Operations as code, small reversible changes, anticipate failure, learn from events | CloudFormation, CloudWatch, Config, Systems Manager, EventBridge |
| **Security** | Protect data, systems, assets | Strong identity foundation, traceability, security at all layers, automate, protect data in transit/at rest, prepare for events | IAM, KMS, GuardDuty, WAF, Shield, CloudTrail |
| **Reliability** | Recover from failure, meet demand | Test recovery, automatic recovery, scale horizontally, stop guessing capacity, manage change via automation | Multi-AZ, Auto Scaling, Route 53, Backup, SQS |
| **Performance Efficiency** | Use resources efficiently | Democratize advanced tech, go global in minutes, serverless, experiment more, mechanical sympathy | CloudFront, Lambda, ElastiCache, Compute Optimizer |
| **Cost Optimization** | Avoid unnecessary cost | Cloud financial management, consumption model, measure efficiency, stop undifferentiated heavy lifting, analyze spend | Cost Explorer, Budgets, Savings Plans, Spot |
| **Sustainability** | Minimize environmental impact | Understand impact, set goals, maximize utilization, adopt efficient hardware, use managed services, reduce downstream impact | Graviton, serverless, right-sizing, lifecycle policies |

## Tools
- **Well-Architected Tool** (workload reviews), **Well-Architected Lenses** (serverless, SaaS, ML...), **Trusted Advisor**, **Architecture Center**.

## Mapping pillars to exam domains
- Domain 1 ↔ Security · Domain 2 ↔ Reliability (+ Operational Excellence) · Domain 3 ↔ Performance Efficiency · Domain 4 ↔ Cost Optimization.

## Shared responsibility reminder
AWS secures the infrastructure; customers secure what they put on it.
