# Domain 1 Practice Assessment (5 questions)

**Result:** 5/5 (100%) | **Attempt:** 1 | **Time:** 3 min 29 s (about 42 s per question) | **Domain:** Design Secure Architectures
*Question gists are paraphrased and the explanations are mine. I don't paste AWS question text or answer options.*

## Summary table

| Q | Topic | Task | Answer (gist) | Core concept |
|---|-------|------|---------------|--------------|
| 1 | Find where malicious traffic from an EC2 instance is going | 1.2 | **VPC Flow Logs** | Network metadata (src/dst IP, ports, protocol, bytes, accept/reject) |
| 2 | Secure three-tier design, web tier reachable only over HTTPS (**select two**) | 1.2 | **IGW + public subnet for web, private subnets for app/db** and **chained security groups with HTTPS-only on the web SG** | Segmentation plus least privilege |
| 3 | HTTPS on an internet-facing ALB with minimal certificate maintenance | 1.3 | **ACM public certificate on an ALB HTTPS listener** | Managed, auto-renewing public certificates |
| 4 | Two EC2 instances, low latency in one Region, high fault tolerance | 2.2 link | **One instance in each Availability Zone** | AZs give fault isolation inside a Region |
| 5 | Store files of unknown format/size, highly available, protected from accidental deletion, **most cost-effective** | 1.3 | **S3 bucket with versioning** | Object storage plus versioning for recovery |

## Question breakdown

### Q1: Identifying malicious network traffic → VPC Flow Logs
- **Why right:** flow logs record IP traffic metadata for network interfaces, including the **destination IP**. They can go to CloudWatch Logs or S3 for filtering.
- **Why the others fail:** the **Health Dashboard** shows AWS service/account events; **CloudWatch Logs** is a destination, not the source of flow data by itself; **CloudTrail** records API calls, not network flows.
- **Memory hook:** *network traffic → Flow Logs; API activity → CloudTrail; metrics/alarms → CloudWatch; AWS service health → Health Dashboard.*

### Q2: Three-tier architecture (select two) → D and E
- **Network (internet gateway + subnets):** attach an **internet gateway**; **public subnet** for the web tier; **private subnets** for the app and database tiers. A subnet is public because its route table points to an IGW (and instances need a public IP).
- **Security groups (least privilege):**
  - Web SG: **HTTPS (443) from the internet** only.
  - App SG: allow only from the **web SG**.
  - DB SG: allow the DB port (3306 MySQL, 5432 PostgreSQL, 1433 SQL Server) only from the **app SG**.
- **Why the others fail:** API Gateway isn't needed for a plain EC2 three-tier app, and making every tier private leaves no internet entry point; "allow all traffic" breaks the HTTPS-only requirement; a **virtual private gateway is for VPN**, not internet access, and public subnets for the app tier over-expose it.
- **Pattern:** Internet → IGW → public web → private app → private DB.

### Q3: SSL/TLS certificate management → ACM public cert on the ALB
- **Why right:** ACM provisions public certs for free, attaches them to the ALB's HTTPS listener, and **renews automatically** when validation and association conditions are met. No manual purchase or replacement on instances.
- **Hidden catch:** an HTTPS listener encrypts **client → ALB** only. If the requirement is **all data in transit**, also use HTTPS between the ALB and the EC2 targets.
- **Why the others fail:** CloudFront forwarding to the ALB over **HTTP** leaves a segment unencrypted (configure HTTPS to the origin); third-party certs imported into IAM add manual renewal work; an **ACM private cert** is for private PKI, not publicly trusted for internet users.
- **Memory hook:** *public website + HTTPS + auto-renewal → ACM public certificate; private/internal PKI → ACM Private CA.*

### Q4: High availability and fault tolerance → separate AZs
- **Why right:** AZs are isolated infrastructure inside a Region with low-latency links; one instance per AZ survives an AZ failure while staying in the same Region (put an **ELB** with health checks in front).
- **Why the others fail:** different **Regions** adds latency/complexity (DR, not this requirement); different **subnets** may still be in the **same AZ**; different **VPCs** give isolation, not AZ diversity.
- **Memory hook:** *fault tolerance in a Region → multiple AZs; regional disaster → multiple Regions; segmentation → subnets; isolation → VPCs.*

### Q5: Cost-effective object storage with deletion protection → S3 + versioning
- **Why right:** S3 handles any file type and size, is highly durable and available, and **versioning** keeps previous versions (a delete adds a **delete marker**), so overwrites and accidental deletes are recoverable. It's also the cheapest fit.
- **Why the others fail:** **DynamoDB global tables** is a NoSQL database, not file storage; **RDS Multi-AZ** is a relational DB (HA, not cheap file storage); **CRR** adds cost, solves geographic redundancy, and doesn't by itself protect against accidental deletion.
- **Caveat:** versioning doesn't stop someone with permission from permanently deleting a **specific version**. For retention-based protection use **S3 Object Lock**; for extra protection, **MFA Delete**.
- **Check this fact before memorizing it:** the assessment's note about maximum object size is muddled. Classic exam facts: a single `PUT` handles up to **5 GB**, **multipart upload** is used for larger objects, and the maximum object size was **5 TB** (AWS has since announced a larger limit; check the current S3 docs).

## Revision table (from the assessment)

| Requirement | Service/feature |
|---|---|
| Identify network traffic destinations | VPC Flow Logs |
| Audit API activity | CloudTrail |
| Metrics, alarms, logs | CloudWatch |
| Public web tier HTTPS only | Security group allowing TCP 443 |
| Secure three-tier | Public web entry, private app/DB tiers |
| Public cert management | ACM |
| HTTPS termination at the load balancer | ALB HTTPS listener |
| HA within one Region | Multiple AZs |
| General object storage | S3 |
| Recover previous object versions | S3 Versioning |
| Retention-based protection | S3 Object Lock |
| Cross-Region copies | S3 CRR |

## What this score tells me
- Strong on: Flow Logs vs CloudTrail, SG chaining, ACM, AZ design, S3 versioning.
- Only 5 questions, so it's a spot check, not proof of readiness. Next test: the 20-question Official Practice Question Set and the full Practice Exam.
