# Task 1.1: Design Secure Access to AWS Resources

## 1. Big picture and fundamentals
- Security is one of the **earliest and biggest design decisions**: define how **people, tools and applications you build** access AWS services and your data.
- "Access" covers: who/what can **launch or terminate** resources, **how and when** access is granted, **operational permissions**, and basically any **API call** to a service.
- Fundamentals the course says you must know first:
  - **Shared Responsibility Model**: AWS secures the cloud itself (hardware, facilities, global infrastructure, managed service software); you secure what you put *in* the cloud (IAM config, data, OS on EC2, security groups, encryption choices). Responsibility shifts toward AWS as services become more managed.
  - **AWS global infrastructure**: Regions, Availability Zones, Edge Locations.
  - **Service resilience scope**: some services are **global** (survive a Region failure), some **regional** (survive an AZ failure), some **AZ-scoped**.
- Know the differences between **public, private, hybrid and multi-cloud** environments and how to design secure access to each.

## 2. AWS accounts (often overlooked, must know)
- An AWS account is the basic container and **security boundary** for resources and identities. Understand what accounts are and how they work before designing access.
- A new account begins with a single **account root user** with full permissions.
- **Extra depth:** multi-account is the recommended pattern (separate prod/dev/security/log-archive accounts) to limit blast radius, which leads into Organizations and Control Tower (section 8).

## 3. IAM fundamentals and best practices
- **IAM is a global service.** Its data (users, groups, roles, policies) is stored once and applies across **all Regions**. Not zonal, not regional.
- **Root user risks**
  - Full permissions that cannot be changed or reduced from within the account (the course's framing).
  - If compromised, the **whole AWS environment is compromised**.
- **Securing the root user**
  - Enable **MFA** on it.
  - Do **not** use it for day-to-day work. Create another user with **limited permissions** instead.
  - **Extra depth:** no root access keys; strong unique password; store credentials securely; use root only for the few tasks that require it (for example changing account settings or closing the account).
- **New IAM identities start with NO permissions.** Permissions must be granted (default is implicit deny).
- **Principle of least privilege**: grant only the permissions needed and no more. It **limits the blast radius** if an identity is compromised.
- **MFA** for root and privileged IAM users is a core best practice.

## 4. IAM users, groups, roles (and when to use each)
| Identity | What it is | Credentials | Typical use |
|---|---|---|---|
| **User** | A person or application identity inside one account | Long-term (password, access keys) | A specific person or legacy app that needs persistent credentials |
| **Group** | Collection of users; attach policies once for many users | None itself | Give a team the same permissions (developers, admins) |
| **Role** | An identity **assumed** by a trusted entity; no permanent credentials | **Temporary** via STS | AWS services (EC2, Lambda), cross-account access, federated users, temporary elevated access |

- **Extra depth:** groups cannot be nested and a group cannot be a Principal in a resource policy. Prefer roles with temporary credentials over IAM users wherever possible.
- The course asks you to understand: how to **create** users/groups/roles, their **strengths and limits**, and scenarios where you would **switch** between user-, group- and role-based permissions.
- **Combining them:** users go in groups (policy per group); roles are assumed for specific tasks or by services.

## 5. Roles, STS and role-based access control
- **Ways to assume a role**: a user/application calls STS; an AWS service is assigned a role (for example an **EC2 instance profile** or **Lambda execution role**); federated identities assume roles; another account assumes a role (cross-account).
- **AWS Security Token Service (STS)** issues **temporary, limited-privilege credentials**. Know its use **within and across accounts** and with **federation**.
- **Role switching** (console) and **cross-account access** are named in the exam guide:
  - Cross-account role: account B creates a role with a **trust policy** naming account A; a principal in A needs permission to call `sts:AssumeRole`.
- **Credential best practices**: never embed long-term keys; use roles for services; rotate and protect any credentials that must exist.

## 6. Federation: when and how
- Users often already have identities **outside AWS** (corporate directory, third-party IdP). They need credentials to use AWS or apps that call AWS.
- Use an **IAM role** to give permissions to users whose identity is **federated** from your organization or a third-party identity provider.
- Know how to **design and configure Active Directory** federation to **IAM roles or users**.
- Services and options to know:
  - **AWS IAM Identity Center (formerly AWS SSO)**: central workforce access across multiple accounts and apps, integrates with corporate directories.
  - **AWS Directory Service**: **AWS Managed Microsoft AD**, **AD Connector** (proxy to on-prem AD), **Simple AD**.
  - **SAML 2.0** federation with an IdP (for example AD FS).
  - **Amazon Cognito** for **customer/app users** (web identity, social login); see Task 2.
- **Decision rule:** workforce with a corporate directory → federate (Identity Center or SAML) to roles; app customers → Cognito.

## 7. IAM policies (read and interpret, not write advanced ones)
- A **policy** is an **object in AWS** that, when associated with an identity or resource, **defines its permissions**.
- **Never hard-code credentials in an application.** Alternatives: IAM roles for services, instance profiles, Secrets Manager, Systems Manager Parameter Store, temporary credentials from STS.
- **Two main types**
  - **Identity-based policies**: attached to an IAM **user, group or role**. They define **what that identity can do** (which resources it can access).
  - **Resource-based policies**: attached to a **resource** (S3 bucket, SQS queue, VPC endpoint, KMS key). They define **who can access the resource** and which actions.
- **Policy statement parts** (know what is required and how they add granularity):
  - `Effect` (Allow/Deny), `Action`, `Resource` (required for identity policies)
  - `Principal`: **required in resource-based policies**, absent in identity policies. This is the structural difference between bucket policies and identity policies. It says *which principals the policy applies to*.
  - `Condition`: adds granularity (source IP, MFA present, time, tags, encryption required, and more)
  - `Sid`, `Version`
  - **Extra depth:** wildcards and ARNs; `NotAction`/`NotResource`.
- **Other policy types to be able to name (wrap-up asks you to differentiate)**
  - **Identity (permissions) policy**: grants permissions to an identity.
  - **Resource policy**: grants access to a resource, can be used for **cross-account** access.
  - **Service control policy (SCP)**: Organizations guardrail that sets the **maximum** permissions for accounts; does not grant by itself.
  - **Extra depth:** permissions boundary (max for a user/role), session policy (limits an assumed-role session), trust policy (who can assume a role).
- **Policy evaluation logic** (must know, especially with multiple policies or overlapping allow/deny):
  1. Everything starts as **implicit deny**.
  2. An **explicit Deny always wins**, regardless of any allow.
  3. Otherwise an explicit **Allow** is needed.
  4. Multiple attached policies are combined; permissions are effectively the **union of allows**, minus any explicit deny.
  5. **Extra depth:** SCPs, permission boundaries and session policies act as **filters** (the effective permission is the intersection). Within one account either an identity policy or a resource policy allow can be enough; **cross-account** needs allow on **both** sides.

## 8. Multi-account security strategy
You will be tested on designing security for **multiple AWS accounts**: gain visibility, enforce standards, alert and automate.
- **AWS Organizations**: group accounts into OUs, apply **SCPs**, consolidated billing.
- **Service control policies**: guardrails that limit what accounts/OUs can do (for example deny leaving approved Regions). They never grant permissions on their own.
- **AWS Control Tower**: automated landing zone with guardrails and an account factory for governed multi-account setup, built on Organizations.
- **Extra depth:** AWS Service Catalog standardizes approved products; AWS Resource Access Manager (RAM) shares resources across accounts; Identity Center gives single sign-on across accounts.

## 9. Traceability: monitor, alert, audit
- Goal: know **who and what has access** to your account resources and data, and be able to **monitor, alert and audit** actions and changes in near real time, and **integrate logs and metrics** with systems that investigate and respond automatically.
- Tools:
  - **AWS CloudTrail**: records API calls (who did what, when, from where).
  - **Amazon CloudWatch**: metrics, logs, alarms.
  - **VPC Flow Logs**: network traffic metadata.
  - **Extra depth:** AWS Config (resource configuration history and compliance rules), GuardDuty, Security Hub, EventBridge + Lambda for automated remediation.

## 10. Practice questions the course says to ask yourself
- How do you create IAM users, groups and roles? What are their strengths and limits?
- When would you switch between user, group and role permissions?
- Can you read an IAM policy and say what it allows or denies?
- How is access evaluated when a user has multiple policies?
- When should an account use federation instead of IAM users?
- How would you see and enforce security across 50 accounts?
