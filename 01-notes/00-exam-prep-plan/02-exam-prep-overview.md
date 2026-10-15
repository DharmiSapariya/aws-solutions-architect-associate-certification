# Exam Prep Overview — SAA-C03

**Status:** ✅ Complete | **Course length:** 20 min | **Level:** Fundamental
*(My own summary of the course. Not official AWS text.)*

## Who the exam is for
- People in a solutions architect role, ideally with 1+ years designing highly available, cost-efficient, fault-tolerant, scalable distributed systems on AWS.
- No formal prerequisites, but you should understand the architect role and how to design using the **Well-Architected Framework**.

## What the certification validates
1. Designing solutions that meet current business needs and future projected needs
2. Designing architectures that are **secure, resilient, high-performing, cost-optimized**
3. Reviewing existing solutions and recommending improvements

## Domains and what to focus on

### Domain 1: Design Secure Architectures (30%)
- **Secure access:** IAM, IAM Identity Center (renamed from AWS SSO), STS, Control Tower, SCPs
- **Secure workloads:** Cognito, GuardDuty, Macie, Shield, WAF, Secrets Manager, VPC controls
- **Secure data:** KMS, ACM, key rotation, certificate renewal, access policies, backup/retention, all storage services

### Domain 2: Design Resilient Architectures (26%)
- **Scalable and loosely coupled:** API Gateway, Transfer Family, SQS, SNS, ALB, Lambda, Fargate, ECS, EKS (Kubernetes is appearing more across exams)
- Know **horizontal vs vertical scaling** (a common confusion), microservices and event-driven design
- **Highly available / fault tolerant:** Route 53, backup strategies, RDS Proxy, X-Ray, AI services like Comprehend/Polly as managed-service examples
- Concepts: immutable infrastructure, disaster recovery best practices, load balancing, distributed design patterns

### Domain 3: Design High-Performing Architectures (24%)
- New section vs. SAA-C02: **data ingestion and transformation**
- Services: CloudFront, Direct Connect, VPN, PrivateLink, S3, EFS, EBS, Batch, ElastiCache, Athena, Lake Formation, QuickSight, DataSync, Storage Gateway, Glue, EMR, Kinesis
- Questions to ask: How do I decouple so components scale independently? Which metrics/conditions trigger scaling? What are the data access patterns?
- Right-size resource type and size (compute, DB, storage); know serverless patterns (Lambda, Fargate)

### Domain 4: Design Cost-Optimized Architectures (20%)
- Cost tools: **Cost Explorer, tags, Budgets, Savings Plans, Reserved Instances, Cost and Usage Reports**
- **Right-sizing** for compute, database *and* storage; S3 lifecycle policies
- Lowest-cost data transfer; load balancing and connectivity costs
- Services: Transit Gateway, VPC peering, Route 53, ELB

## Fundamentals to know cold
- What cloud computing is and its five characteristics
- **Global infrastructure:** Regions, Availability Zones, Edge Locations
- Resilience scopes: **AZ-resilient** vs **Region-resilient** vs **globally resilient** services
- **Shared Responsibility Model**
- **Well-Architected Framework, six pillars**, including the new **Sustainability** pillar. Ways to learn it: documentation, Well-Architected Tool, Well-Architected Labs
- General IT basics: networking, virtualization, encryption, storage, security, DNS/DNSSEC, databases
- Mindset: *everything fails all the time*, so learn how AWS can fail and design around it

## Question types and test strategy
- **Multiple choice:** 1 correct + 3 distractors
- **Multiple response:** 2+ correct out of 5+ options
- Questions are long scenarios, and answers are long too. Read every word.
- Watch for answers that repeat the same sentence with **one or two words changed**. Those words decide the answer.
- Eliminate distractors early, then pick the *best* answer among the plausible ones.
- Use practice questions to build reading comprehension and spot keywords.

## Note to self
- The course names **AWS SSO** as renamed to **IAM Identity Center**. The exam guide (v1.0) still says "AWS SSO", so know both names.
