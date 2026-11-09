# Domain 1 · Design Secure Architectures (30%)

Task statements: (1.1) secure access to AWS resources, (1.2) secure workloads and applications, (1.3) determine appropriate data security controls.

---
## 1.1 Secure access to AWS resources

### Shared Responsibility Model
- **AWS**: security *of* the cloud (hardware, facilities, hypervisor, managed-service software).
- **Customer**: security *in* the cloud (IAM, data, OS patching on EC2, security groups, encryption choices).
- Responsibility shifts toward AWS as you move EC2 → RDS → Lambda → S3/DynamoDB.

### IAM fundamentals
- **Users, groups, roles, policies.** Prefer **roles** (temporary credentials via STS) for workloads and cross-account access.
- **Policy types**: identity-based, resource-based (S3 bucket policy, KMS key policy, SQS), permissions boundaries, SCPs, session policies, VPC endpoint policies.
- **Evaluation logic**: explicit **Deny** > explicit Allow > implicit deny. For cross-account: both the identity policy and resource policy must allow (unless resource policy grants the principal directly for same account).
- **Root account**: enable MFA, lock away, use only for tasks that require it (change account settings, close account, restore IAM permissions, change support plan).
- **MFA**: virtual, hardware, FIDO2. Use `aws:MultiFactorAuthPresent` condition.
- **IAM Access Analyzer**: finds resources shared externally, validates policies, generates least-privilege policies from CloudTrail activity.
- **Credential report / Access Advisor**: audit unused credentials and unused permissions.
- **Instance profile**: container that passes an IAM role to EC2.
- **PassRole**: permission `iam:PassRole` needed to hand a role to a service.

### Federation & workforce identity
| Need | Service |
|---|---|
| Workforce SSO across many accounts / SaaS apps | **IAM Identity Center** (successor to AWS SSO) |
| Corporate AD login to AWS | AD Connector, AWS Managed Microsoft AD, or SAML 2.0 federation |
| Web/mobile app end users (sign-up/sign-in, social login) | **Amazon Cognito** User Pools (authN) |
| Give app users temporary AWS credentials | Cognito **Identity Pools** (authZ) |
| Web identity (Google, Facebook) to AWS creds | Cognito Identity Pools |

### Multi-account governance
- **AWS Organizations**: consolidated billing, OUs, **SCPs** (guardrails — they *limit* max permissions; they never grant).
- **AWS Control Tower**: landing zone, guardrails (preventive via SCP, detective via Config), account factory.
- **AWS RAM** (Resource Access Manager): share subnets, Transit Gateway, License Manager configs across accounts without duplicating.
- Cross-account access: IAM role in target account trusted by source account; `sts:AssumeRole`.

---
## 1.2 Secure workloads and applications

### VPC network security
| Control | Level | Stateful? | Rules |
|---|---|---|---|
| **Security Group** | ENI / instance | Stateful | Allow only; can reference other SGs |
| **Network ACL** | Subnet | Stateless | Allow **and** deny; numbered rules; ephemeral ports needed |

- Public subnet = route to **Internet Gateway**. Private subnet = route to **NAT Gateway** (in a public subnet, per-AZ for HA) or no internet.
- **VPC Endpoints**: **Gateway** (S3, DynamoDB — free, route table entry) vs **Interface** (PrivateLink, ENI, most other services, hourly+data cost).
- **PrivateLink**: expose your service privately to other VPCs/accounts without peering, overlapping CIDRs OK.
- **VPC Peering**: non-transitive, no overlapping CIDRs. **Transit Gateway**: hub-and-spoke, transitive.
- **VPC Flow Logs**: IP traffic metadata (not packet content) to CloudWatch Logs / S3 / Kinesis Firehose.
- **Bastion / SSM Session Manager**: prefer Session Manager (no open port 22, IAM-controlled, logged).

### Edge & application protection
| Threat | Service |
|---|---|
| L3/L4 DDoS | **AWS Shield Standard** (free, automatic); **Shield Advanced** (24/7 DRT, cost protection, L7 with WAF) |
| SQLi, XSS, bad bots, geo-block, rate limit | **AWS WAF** (attach to CloudFront, ALB, API Gateway, AppSync, Cognito) |
| Central firewall policy across accounts | **AWS Firewall Manager** |
| Stateful network firewall inside VPC (IPS, domain filtering) | **AWS Network Firewall** |
| Block origin access except via CDN | CloudFront **OAC** (Origin Access Control; replaces OAI) for S3; custom header / managed prefix list for ALB |
| DNS-level threats | Route 53 Resolver DNS Firewall |

### Threat detection & compliance services
| Service | Purpose |
|---|---|
| **GuardDuty** | Threat detection from CloudTrail, VPC Flow, DNS logs, S3, EKS, malware |
| **Inspector** | Vulnerability scanning of EC2, ECR images, Lambda |
| **Macie** | Discovers sensitive data (PII) in S3 |
| **Security Hub** | Aggregates findings; CIS/PCI standards |
| **Detective** | Root-cause investigation using graph analysis |
| **Config** | Resource configuration history + compliance rules + auto-remediation |
| **CloudTrail** | API call audit log (management events default; data events optional); org trail; log file validation |
| **Trusted Advisor** | Best-practice checks (full checks need Business/Enterprise support) |
| **Artifact** | On-demand compliance reports / agreements |
| **Audit Manager** | Continuous audit evidence collection |

### Application-layer secrets & config
- **Secrets Manager**: automatic rotation (RDS, Redshift, DocumentDB natively; Lambda for others), cross-region replication, cost per secret.
- **SSM Parameter Store**: config + secrets (SecureString via KMS); cheaper; no native rotation.
- Never hard-code credentials; use roles + Secrets Manager.

---
## 1.3 Data security controls

### Encryption at rest
- **KMS**: symmetric (AES-256) & asymmetric keys.
  - Key types: **AWS owned**, **AWS managed** (`aws/s3`, rotates yearly automatically), **customer managed** (you control policy, optional auto rotation yearly, can disable/delete with 7–30 day wait).
  - **Envelope encryption**: data key encrypts data; KMS key encrypts data key. Direct KMS Encrypt limited to 4 KB.
  - Key policy is the primary access control; grants and IAM policies supplement it.
  - **Multi-Region keys** for cross-region encryption/DR (same key ID, independent management).
  - **Imported key material** & **custom key store (CloudHSM)** for compliance control.
- **CloudHSM**: dedicated single-tenant FIPS 140-2 Level 3 HSM; you manage keys (AWS cannot). Choose when regulations demand customer-exclusive key control.
- **S3 encryption options**: SSE-S3 (default on for all new objects), SSE-KMS (audit via CloudTrail; KMS request limits), **DSSE-KMS** (dual-layer), SSE-C (you supply the key each request), client-side encryption.
- **EBS**: encrypted snapshots/volumes via KMS; cannot directly encrypt an existing unencrypted volume — snapshot → copy as encrypted → new volume. Can enable "encryption by default" per region.
- **RDS/Aurora**: encryption chosen at creation; to encrypt existing, snapshot → copy encrypted → restore. Read replicas of encrypted DB are encrypted.
- **DynamoDB**: encrypted by default; **EFS**: enable at creation; **Redshift**, **SQS**, **SNS**, **Kinesis** all support KMS.

### Encryption in transit
- **ACM (AWS Certificate Manager)**: free public TLS certs for ELB, CloudFront, API Gateway (regional certs; CloudFront certs must be in **us-east-1**). Auto-renewal.
- Enforce HTTPS: ALB redirect rule, CloudFront viewer protocol policy, S3 bucket policy `aws:SecureTransport`.
- TLS termination: ALB (terminates), NLB (TLS listener or pass-through), end-to-end re-encrypt to targets if required.
- **VPN**: IPSec over internet; **Direct Connect** is private but **not encrypted by default** (add VPN over DX or MACsec).

### S3 security controls
- **Block Public Access** (account & bucket level) — turn on by default.
- **Bucket policies**, ACLs (legacy; disable with *Bucket owner enforced*), **Access Points** (per-application policies), **Presigned URLs** (temporary access).
- **Object Lock** (WORM): Governance vs Compliance mode, legal hold. **Glacier Vault Lock** for vault-level compliance policy.
- **Versioning + MFA Delete** to protect against accidental/malicious deletion.
- **S3 Object Lambda**, **VPC endpoint policies**, **Multi-Region Access Points**.

### Backup, retention & data lifecycle
- **AWS Backup**: centralized policy-based backups (EBS, RDS, DynamoDB, EFS, FSx, S3, EC2...), cross-region/account copy, **Vault Lock**.
- **Lifecycle policies** for retention/expiration; legal hold with Object Lock.
- **Data classification**: Macie. **Tokenization / masking**: handled in app or Glue/Lambda.

### Database security extras
- RDS IAM database authentication, SSL enforcement, security groups, private subnets.
- **RDS Proxy**: pooled connections + IAM auth + Secrets Manager.
- DynamoDB fine-grained access via IAM condition `dynamodb:LeadingKeys`.
- Redshift: column-level security, VPC enhanced routing.

---
## Exam traps & quick answers
- "Rotate DB credentials automatically with no code changes" → **Secrets Manager**.
- "Prevent S3 objects deleted/overwritten for N years (regulatory)" → **S3 Object Lock, Compliance mode**.
- "Block a specific IP from an app" → **NACL deny** (SG can't deny) or **WAF IP set**.
- "Restrict S3 access to a VPC only" → **Gateway endpoint + bucket policy `aws:sourceVpce`**.
- "Audit who called which API" → **CloudTrail**. "Config change history / compliance" → **AWS Config**.
- "Detect compromised EC2 / crypto-mining" → **GuardDuty**.
- "Find PII in S3" → **Macie**.
- "Restrict a whole member account from using a region/service" → **SCP**.
- "Cross-account S3 upload where owner must have full control" → bucket-owner-enforced / `bucket-owner-full-control`.
- "Only CloudFront can read from S3" → **OAC**.
- "Customer must hold keys exclusively" → **CloudHSM** (or KMS with external key store).
