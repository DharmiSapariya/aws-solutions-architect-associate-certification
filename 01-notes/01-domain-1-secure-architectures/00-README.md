# Domain 1: Design Secure Architectures (30% of the exam)

**Status:** Course reviewed | **Course:** Domain 1 Review (SAA-C03), 45 min
*My in-depth notes, combining the Skill Builder course with the official Exam Guide v1.0. Not official AWS text. Items marked **Extra depth** are things I added from AWS docs knowledge to fill in what the exam expects. Verify details against current AWS docs.*

## Why this domain matters most
- It is the **largest domain (30%)** and security is a theme in *every* domain.
- AWS says security should be considered at **every stage, level and tier** of an application, not only in "security" topics. Ask "how do I secure this?" for every service you study.
- Exam tip from the course: the **best answer is very often the most secure one** that still meets the requirements.
- You need to know the security services *and* the concepts behind them, because the exam makes you choose between two similar services based on the scenario wording.

## The 3 task statements

| # | Task statement | Notes file |
|---|----------------|-----------|
| 1.1 | Design secure access to AWS resources | [01-task-1-secure-access-to-aws-resources.md](01-task-1-secure-access-to-aws-resources.md) |
| 1.2 | Design secure workloads and applications | [02-task-2-secure-workloads-and-applications.md](02-task-2-secure-workloads-and-applications.md) |
| 1.3 | Determine appropriate data security controls | [03-task-3-data-security-controls.md](03-task-3-data-security-controls.md) |
| | Wrap-up, comparison tables, cheat sheet | [04-wrap-up-comparisons-and-cheatsheet.md](04-wrap-up-comparisons-and-cheatsheet.md) |
| | Self-quiz / flashcards | [05-self-quiz-flashcards.md](05-self-quiz-flashcards.md) |

## Exam Guide coverage map (every knowledge/skill bullet)

### Task 1.1: Secure access to AWS resources
| Exam guide bullet | Where covered |
|---|---|
| Access controls and management across multiple accounts | Task 1 §8 (Organizations, Control Tower, SCPs) |
| Federated access and identity services (IAM, AWS SSO) | Task 1 §6 |
| AWS global infrastructure (AZs, Regions) | Task 1 §1, Task 2 §1 |
| Security best practices (least privilege) | Task 1 §3, §4 |
| Shared responsibility model | Task 1 §1 |
| Skill: best practices for IAM users and root users (MFA) | Task 1 §3 |
| Skill: flexible authorization model (users, groups, roles, policies) | Task 1 §4, §5 |
| Skill: role-based access control (STS, role switching, cross-account) | Task 1 §5 |
| Skill: multi-account security strategy (Control Tower, SCPs) | Task 1 §8 |
| Skill: appropriate use of resource policies | Task 1 §7 |
| Skill: when to federate a directory service with IAM roles | Task 1 §6 |

### Task 1.2: Secure workloads and applications
| Exam guide bullet | Where covered |
|---|---|
| Application configuration and credentials security | Task 2 §9 |
| AWS service endpoints | Task 2 §5 |
| Control ports, protocols, and network traffic | Task 2 §2, §3 |
| Secure application access | Task 2 §7 |
| Security services and use cases (Cognito, GuardDuty, Macie) | Task 2 §7 |
| Threat vectors external to AWS (DDoS, SQL injection) | Task 2 §8 |
| Skill: VPC architectures with security components | Task 2 §2, §3 |
| Skill: network segmentation (public/private subnets) | Task 2 §4 |
| Skill: integrate services (Shield, WAF, SSO, Secrets Manager) | Task 2 §8, §9 |
| Skill: secure external connections (VPN, Direct Connect) | Task 2 §6 |

### Task 1.3: Data security controls
| Exam guide bullet | Where covered |
|---|---|
| Data access and governance | Task 3 §9 |
| Data recovery | Task 3 §11, §12 |
| Data retention and classification | Task 3 §7 |
| Encryption and appropriate key management | Task 3 §1 to §4 |
| Skill: align AWS technologies to compliance requirements | Task 3 §6 |
| Skill: encrypt data at rest (KMS) | Task 3 §3, §5 |
| Skill: encrypt data in transit (ACM, TLS) | Task 3 §4 |
| Skill: access policies for encryption keys | Task 3 §3 |
| Skill: data backups and replications | Task 3 §12 |
| Skill: policies for data access, lifecycle, protection | Task 3 §9, §10 |
| Skill: rotate keys and renew certificates | Task 3 §3, §4 |

## Official resources listed by the course
Managing Your AWS Account; What is IAM?; What is Amazon VPC?; Security in Amazon VPC; Security, Identity, and Compliance; Data Encryption; Amazon EBS Snapshots; Point-in-Time Recovery for DynamoDB; Backing up and Restoring an RDS DB Instance; Creating a DB Cluster Snapshot; Using AWS Backup with EFS; Amazon Redshift Snapshots and Backups; Backing up and Restoring a Neptune DB Cluster; Backing up and Restoring in DocumentDB; S3 Cross-Region Replication; AWS Backup; plus the product pages for EBS, EC2, RDS/Aurora, DynamoDB, EFS, Storage Gateway, and FSx for Windows/Lustre.

## Transcript quirks to remember
- The speaker says "principle" in places where the correct term is **Principal** (policy element) and **principle** of least privilege. Learn the correct spelling.
- The course still says **AWS Single Sign-On (SSO)**; the current name is **AWS IAM Identity Center**. The exam may use either.
- The three **walkthrough questions** in the course are not in the transcript. Add your own notes on them in `02-practice-questions/`.
