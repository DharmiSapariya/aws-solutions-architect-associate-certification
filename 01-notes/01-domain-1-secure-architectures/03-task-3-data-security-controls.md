# Task 1.3: Determine Appropriate Data Security Controls

**Core idea:** protecting data is one of the most important parts of security. Evaluate security for data **in transit and at rest**, apply **least privilege** at every layer, and secure access to your **encryption keys** too. *The best exam answer is often the most secure one.*

## 1. Encryption: at rest vs in transit
| | At rest | In transit |
|---|---|---|
| Protects against | Unauthorized access and **theft** of stored data | Interception while data moves between places |
| Parties | Usually **one** party | **Two or more** parties |
| Idea | Data stored encrypted on disk/storage | Adds a **tunnel** around data so outsiders can't read it |
| AWS examples | KMS with S3/EBS/RDS/DynamoDB | TLS/HTTPS with ACM, VPN, Direct Connect with encryption |

## 2. Encryption terms and key types
- **Plaintext**: unencrypted data. Not always text (docs, images, applications).
- **Algorithm**: code that takes **plaintext + key** and produces encrypted output.
- **Key**: like a password, used with the algorithm. Different encryption types use different keys.
- **Ciphertext**: the encrypted data.
- **Symmetric**: the **same key** encrypts and decrypts (fast, used for bulk data).
- **Asymmetric**: a **key pair** (public encrypts/verifies, private decrypts/signs).

## 3. Key management: KMS vs CloudHSM, key types, policies, rotation
- **AWS KMS**: managed service to create and control encryption keys, integrated with most AWS services, with auditing in CloudTrail.
- **AWS CloudHSM**: **dedicated hardware security module** (single-tenant) in your VPC where **you** control the keys.
- **Why KMS instead of CloudHSM?** Simplicity, deep integration, lower cost, fully managed. **Why CloudHSM?** Dedicated/single-tenant HSM, strict compliance or customer-exclusive key control.
- **Using them together:** KMS **custom key store** backed by CloudHSM.
- **Key types** (know the differences in capabilities, **Extra depth**):
  - **AWS owned keys**: used by AWS services, invisible to you.
  - **AWS managed keys** (for example `aws/s3`): created by AWS in your account, **rotate automatically** on a fixed schedule you cannot change.
  - **Customer managed keys (CMKs)**: you control the key policy, enable/disable, and rotation.
  - **Symmetric vs asymmetric KMS keys**; HMAC keys.
- **Root keys vs data keys (envelope encryption):** a KMS key (the "root" key) never leaves KMS and is used to generate and encrypt **data keys**; the data key encrypts your data. Handling differs because the master key is protected inside KMS while data keys are used by the service/client and stored only in encrypted form.
- **Multi-Region keys** help manage keys **across Regions** (same key ID/material replicated).
- **Access policies for keys**
  - **Key policy** is the primary control (resource-based policy on the key).
  - Can combine with **IAM policies** and **grants**.
  - Apply **least privilege**: separate key administrators from key users.
- **Rotation**: know **how often each key type can rotate** (**Extra depth**, verify current docs): AWS managed keys rotate automatically on a fixed cycle; customer managed symmetric keys support **optional automatic rotation** (default yearly, configurable) plus on-demand; **asymmetric keys, HMAC keys and imported key material need manual rotation**.

## 4. Encryption in transit: ACM and TLS
- **AWS Certificate Manager (ACM)** provisions, manages and **renews TLS certificates** for use with **ELB, CloudFront, API Gateway** and similar services (public certs are free). Know **how certificates are renewed** (ACM-issued certificates auto-renew when validation remains valid; **imported certificates must be renewed by you**).
- **Extra depth:** ACM public certs can't be installed directly on EC2; use a load balancer/CloudFront, or a Private CA for internal certs.
- **Enforce HTTPS** (for example S3 bucket policy condition `aws:SecureTransport`).

## 5. Amazon S3 encryption (and other storage)
- S3 supports encryption **at rest and in transit**.
- **Client-side encryption**: data is encrypted **before it leaves the source**; you control everything (keys and process).
- **Server-side encryption (SSE)**: data travels over **HTTPS** and S3 encrypts it on arrival. Three choices to know:
  1. **SSE-C**: server-side with **customer-provided keys** (you supply the key on each request; AWS doesn't store it).
  2. **SSE-S3**: server-side with **S3-managed keys**.
  3. **SSE-KMS**: server-side with **keys stored in AWS KMS** (course calls them "customer master keys in KMS"); gives key policy control and an **audit trail** in CloudTrail.
- **Extra depth:** SSE-S3 is the default for new objects; SSE-KMS can hit KMS request quotas at very high request rates.
- **EBS**: encrypted volumes protect data at rest, data moving between instance and volume, snapshots, and volumes created from them, with minimal performance impact.

## 6. Compliance
- Security and compliance is a **shared responsibility** between AWS and the customer.
- **AWS Artifact**: **self-service central repository** of AWS **security and compliance reports** (SOC, ISO, PCI and others) and **online agreements**.
- Skill: align AWS technologies to compliance needs. For example: encryption with KMS/CloudHSM, S3 Object Lock for WORM retention, Config rules and Audit Manager for evidence, Region choice for data residency.

## 7. Retention, classification, defense in depth
- Follow best practices for securing sensitive data in AWS data stores: understand **general data security patterns** and map them to **cloud security controls**.
- **AWS Cloud Adoption Framework (CAF), security perspective.** The course lists five capabilities: **IAM, detective controls, infrastructure security, data protection, incident response**. *(Newer CAF versions list more capabilities.)*
- Three principles from the course:
  1. Think about **the data you protect, how it is stored, and who has access**.
  2. **Not all data is equal**: **classify data** properly to enforce the right security.
  3. **Defense in depth**: **layer multiple controls** for redundancy; two categories, **preventative** and **detective**.
- **Retention** policies: how long to keep data and when to delete (lifecycle rules, backups, Object Lock).

## 8. Designing data protection: scenarios and questions
- How do you protect data in transit over a **VPN over the internet**, over **Direct Connect**, **between VPCs**, and between **S3 and your VPC**? *(Extra depth: IPsec encrypts the VPN; Direct Connect is not encrypted by default; use TLS everywhere; gateway endpoints keep S3 traffic on the AWS network.)*
- How do you protect data for users reaching you over the **public internet**? (TLS/HTTPS, ACM, CloudFront.)
- How do storage services differ in protection, for example **S3 vs EBS**, and does protection affect performance?
- **Scenario: data generated on an EC2 instance using EBS that must be protected while staying durable.** Encrypted **EBS volume** or transfer to an encrypted S3 bucket? **The least effort is the encrypted EBS volume.**
- **Performance impact of encryption:** know which services have **no impact** and which have **slight** impact, for example retrieving data from **RDS with KMS** or reading from **S3**.
- **Managed services that help secure, evaluate and audit data**: **KMS** (course emphasis), plus Macie, Config, Security Hub, CloudTrail.

## 9. Access governance and protection by access pattern
- **Data access and governance**: least privilege, classify data, control who can read/write/decrypt.
- S3 lets you manage security for **entire buckets** and add control by **specific paths (prefixes) or objects**.
- Know which services offer that granularity, **read and build policies** for different access patterns, and understand **how the service evaluates them**.
- **Extra depth S3 controls:** bucket policies, IAM policies, **Block Public Access**, access points, ACLs (legacy), Object Lock, versioning, MFA delete.

## 10. Automated data lifecycle
- Capabilities exist for **automatically managing the data lifecycle**.
- **S3 lifecycle configurations** vs **S3 Intelligent-Tiering**:
  - **Lifecycle rules**: when the **access pattern is known/predictable**, transition or expire objects after set days.
  - **Intelligent-Tiering**: when access patterns are **unknown or changing**; it moves objects automatically (monitoring fee, no retrieval fee).
- Lifecycle also supports **expiration and retention** goals.

## 11. Cloud storage fundamentals and disaster recovery
- **Cloud storage**: a model where a provider stores your data and operates storage **as a service**.
- **Benefits**: no hardware to buy, no storage to provision, no capital spending, faster deployment; **lifecycle management policies** add automation, savings, and can **lock data** for compliance.
- **Requirements to plan for**: **durability, availability, security, regulatory/governance, functional**.
- **Data types**: **object, file, block**.
- **Five uses**: **backup and recovery, software test and development, data migration, compliance, big data and data lakes**.
- **Disaster recovery (DR)** is a big exam topic and part of resilient design. Four approaches from low cost/low complexity to complex multi-Region:
  | Strategy | Idea | Typical RTO/RPO | Cost |
  |---|---|---|---|
  | **Backup and restore** | Periodic backups, rebuild on disaster | Hours | Lowest |
  | **Pilot light** | Core components always on in DR site, scale out on failover | Tens of minutes | Low |
  | **Warm standby** | Scaled-down full copy running | Minutes | Medium |
  | **Multi-site active-active** | Full capacity in multiple sites serving traffic | Near zero | Highest |
- **Active-passive**: an active site (for example one Region) serves traffic; a **passive** site (another Region) is used for recovery and **doesn't serve traffic until failover**.
- **Choosing**: for the loss of **one data center** in a well-architected highly available workload, **backup and restore** may be enough. If disaster means a **whole Region** or regulation demands it, consider **pilot light, warm standby or multi-site active-active**.
- Terms: **RPO** (how much data loss is acceptable) and **RTO** (how long to recover).

## 12. Backups and replication
- Understand services that use a **backup strategy** that runs **periodically or continuously**. **Backup frequency determines achievable recovery point (RPO)**. A backup should allow **point-in-time restore**.
- Backup features to know: **EBS snapshots; DynamoDB backups (and point-in-time recovery); RDS snapshots; Aurora snapshots; EFS backup (via AWS Backup); Redshift snapshots; Neptune snapshots; DocumentDB backups**.
- **Amazon S3 Cross-Region Replication (CRR)**: asynchronously and continuously copies objects to a bucket in the DR Region; **versioning** lets you choose a restore point (CRR requires versioning).
- **AWS Backup**: centralized place to **configure, schedule and monitor** backups. Know its capabilities for: **EBS volumes, EC2 instances, RDS and Aurora, DynamoDB tables, EFS file systems, Storage Gateway volumes, FSx for Windows File Server and FSx for Lustre**. It supports **cross-Region copy** to a DR Region.
- **Hybrid environments**: **AWS Storage Gateway**.
- Focus for the exam: **data security patterns and the AWS controls that protect data**.

## 13. Questions the course says to ask yourself
- Why KMS instead of CloudHSM, and how can they work together?
- How do you manage keys across Regions? What are the key types and how often can each rotate?
- How do you use ACM, and how are certificates renewed?
- Which S3 server-side encryption option fits which requirement?
- Will encryption affect performance, and where?
- When use lifecycle configurations vs Intelligent-Tiering?
- Which DR strategy fits the given RPO/RTO and definition of "disaster"?
