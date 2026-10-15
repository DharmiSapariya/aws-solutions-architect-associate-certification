# Domain 1: Wrap-up, Comparison Tables and Cheat Sheet

## Course wrap-up: what to study deeper
The Domain 1 wrap-up says to dive deeper into protecting **resources, applications and data**, and to be able to:
1. Apply **least privilege** (only needed permissions, no more) in a **multi-account** setup. Candidates: **AWS Control Tower, AWS Service Catalog, AWS Organizations**.
   - **Organizations + SCPs** set the permission ceiling per account/OU; **Control Tower** automates a governed multi-account environment with guardrails built on Organizations. **Service Catalog** distributes approved, pre-configured products rather than controlling permissions. For "least privilege across accounts" think **Organizations (SCPs), with Control Tower to set it up and govern it**.
2. Know **when to use an IAM role vs an IAM user**.
3. Know the difference between **identity policy, resource policy, permissions policy, SCP**, and how **overlapping allow/deny** rules are evaluated (explicit deny wins).
4. Know **different ways to federate** into AWS, **AWS SSO (IAM Identity Center)** and use cases for **AWS Directory Service**.
5. Know monitoring services beyond **CloudTrail, CloudWatch and VPC Flow Logs** (for example AWS Config, GuardDuty, Security Hub, Detective, Macie, Inspector).
6. Set up your own **VPCs with appropriate security controls**.
7. Know the use cases and capabilities of **Shield, WAF, Secrets Manager and Systems Manager Parameter Store**.
8. Know how to protect data **in transit and at rest**, and when to choose **KMS vs CloudHSM**.
9. Be able to **choose between two services based on the question stem**. This is what passes the exam.

## Comparison tables

### IAM user vs group vs role
| | User | Group | Role |
|---|---|---|---|
| Credentials | Long-term | None | Temporary (STS) |
| Used by | One person/app | Set of users | Services, other accounts, federated users |
| Can be a Principal in resource policy | Yes | No | Yes |
| Prefer when | Truly need persistent identity | Managing permissions for many users | Almost everything else |

### Policy types
| Policy | Attached to | Answers |
|---|---|---|
| Identity-based | User, group, role | What can this identity do? |
| Resource-based | S3, SQS, KMS key, VPC endpoint | Who can access this resource? (has Principal) |
| SCP | Org, OU, account | What is the maximum allowed? (no grants) |
| Permissions boundary | User/role | Maximum a user/role can get |
| Trust policy | Role | Who can assume this role? |

### Evaluation order in one line
Implicit deny → any **explicit deny** wins → need an **allow** → SCP/boundary/session policies must also allow.

### Security group vs NACL
| | Security group | NACL |
|---|---|---|
| Level | Instance/ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and deny |
| Evaluation | All rules | In number order |
| Default (custom) | Deny inbound, allow outbound | Custom NACL denies all |

### Ways to reach AWS services and other networks
| Need | Use |
|---|---|
| Private access to S3/DynamoDB from VPC | **Gateway endpoint** |
| Private access to other AWS services/SaaS | **Interface endpoint (PrivateLink)** |
| Expose one service to many VPCs/accounts | **PrivateLink** |
| Connect two VPCs simply | **VPC peering** (non-transitive) |
| Hub for many VPCs/on-prem | **Transit Gateway** |
| Encrypted tunnel to on-prem over internet | **Site-to-Site VPN** |
| Remote users into VPC | **Client VPN** |
| Dedicated private line | **Direct Connect** |
| Private subnet outbound to internet | **NAT gateway** |

### Shield vs WAF
| | Shield Standard | Shield Advanced | WAF |
|---|---|---|---|
| Layer | L3/L4 | L3/L4 plus L7 with WAF | L7 |
| Cost | Free | Paid | Per web ACL/rule/request |
| Stops | Common DDoS | Larger/complex DDoS, response team | SQLi, XSS, bad bots, rate limiting |
| Deploys on | Automatic | ELB, CloudFront, Route 53, Global Accelerator, EIP | ALB, API Gateway, CloudFront |

### Secrets Manager vs Parameter Store
Secrets Manager = built for secrets, **automatic rotation**, higher cost. Parameter Store = config + secrets, **no native rotation**, standard tier free.

### KMS vs CloudHSM
| | KMS | CloudHSM |
|---|---|---|
| Model | Managed, multi-tenant | Dedicated single-tenant HSM |
| Control | AWS manages hardware; you manage keys/policies | You manage everything, AWS has no access to keys |
| Integration | Native with most services | Custom/app integration, or KMS custom key store |
| Pick when | Default choice | Strict compliance, exclusive control |

### S3 server-side encryption
| Option | Keys | Notes |
|---|---|---|
| SSE-S3 | Managed by S3 | Simplest, default |
| SSE-KMS | KMS keys | Key policy control, CloudTrail audit |
| SSE-C | You provide each request | AWS doesn't store key |
| Client-side | You, before upload | You control everything |

### Disaster recovery
Backup & restore (cheapest, slowest) → Pilot light → Warm standby → Multi-site active-active (fastest, most expensive).

## Service lookup: scenario keyword → service
| If the question says... | Think... |
|---|---|
| PII / sensitive data in S3 | **Macie** |
| Threat detection, compromised instance, unusual API calls | **GuardDuty** |
| DDoS | **Shield (Advanced)** |
| SQL injection, XSS, block malicious web requests | **WAF** |
| Rotate database credentials automatically | **Secrets Manager** |
| Mobile/web app user sign-in, social login | **Cognito user pool** |
| App users need temporary AWS credentials | **Cognito identity pool** |
| Workforce SSO across accounts | **IAM Identity Center** |
| Corporate AD + AWS | **Directory Service / AD federation** |
| Restrict what accounts can do | **SCPs (Organizations)** |
| Set up governed multi-account environment | **Control Tower** |
| Audit API activity | **CloudTrail** |
| Network traffic logs | **VPC Flow Logs** |
| Compliance reports (SOC, ISO) | **AWS Artifact** |
| Manage encryption keys | **KMS** (or **CloudHSM** for dedicated HSM) |
| Certificates for HTTPS on ALB/CloudFront | **ACM** |
| Centralized backup across services | **AWS Backup** |
| Copy S3 objects to another Region | **S3 CRR (needs versioning)** |
| Hybrid file/volume access to AWS | **Storage Gateway** |
| Hard-coded credentials in app | **IAM role / Secrets Manager** |

## Exam traps from this domain
- **Root user** is for emergencies only; MFA on it; never for daily work.
- **IAM is global**, **VPC is regional**, **subnet is AZ-scoped**.
- **Explicit deny beats allow**, always.
- **NACLs are stateless**: remember ephemeral ports. **SGs cannot deny**.
- **WAF is not on NLB/EC2 directly**; it fronts ALB, API Gateway, CloudFront.
- **Resource policy has a Principal element**, identity policy does not.
- **Never hard-code credentials**.
- **Peering doesn't scale and exposes more**; PrivateLink exposes only the service.
- **Direct Connect is not encrypted by default**.
- Least effort to protect EBS-resident data: **encrypted EBS volume**.
- "Most secure that meets the requirement" is usually the right answer.
