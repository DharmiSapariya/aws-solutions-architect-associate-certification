# Task 1.2: Design Secure Workloads and Applications

**Focus:** how the people, tools and applications you build **securely access** AWS services, and how you secure the networks, tiers and applications themselves. This includes who/what can launch or terminate resources, how and when access is given, and operational permissions.
*(The transcript calls this "cure workloads", which is a transcription error for "secure".)*

## 1. VPC fundamentals
- You must know how to **design, build and secure** an Amazon VPC.
- **Two types of VPC: default and custom.** Know how security is **initially configured** for each.
  | | Default VPC | Custom VPC |
  |---|---|---|
  | Created by | AWS, one per Region, automatically | You |
  | Subnets | One **public** subnet per AZ, pre-created | None until you create them |
  | Internet access | Internet gateway attached, route to it exists | **None by default**. You add IGW, routes |
  | Security group | Default SG: allows inbound from members of itself, all outbound | A default SG is created too (same rules) |
  | Network ACL | Default NACL: **allows all** inbound and outbound | Default NACL allows all; a **new custom NACL denies all** until you add rules |
  | Use | Quick start, not best practice for production | Production, full control over CIDR, tiers, routing |
  *(Extra depth: default VPC CIDR is 172.31.0.0/16.)*
- **Resilience scope**
  - A **VPC is regional**: created in **one Region and one AWS account**.
  - A **subnet is Availability Zone resilient** (lives in exactly one AZ). Subnets are where your services sit and run, and they add structure and functionality to the VPC.
  - So a multi-AZ design needs **subnets in multiple AZs**.
- **Networking fundamentals expected:** protocols, **CIDR**, **subnetting**, routing, security filters, gateways.

## 2. Security components for multi-tier VPC designs
Many applications are **multi-tiered** (web / app / database). Know the use and functionality of these four, **how they work individually and together**, and how to build their rules:

| Component | Level | State | Rules | Key points |
|---|---|---|---|---|
| **Security group (SG)** | Instance / ENI | **Stateful** (return traffic automatic) | **Allow only** | All rules evaluated; can reference other SGs; default denies all inbound |
| **Network ACL (NACL)** | **Subnet** | **Stateless** (must allow return traffic) | **Allow and Deny** | Rules processed in **number order**, lowest first, first match wins; includes ephemeral ports |
| **Route table** | Subnet | n/a | Routes | Decides where traffic goes (local, IGW, NAT, VGW, TGW, peering, endpoint) |
| **NAT gateway** | Subnet (placed in a **public** subnet) | n/a | n/a | Lets **private** subnet resources reach the internet **outbound only** |

- **Internet gateway (IGW)**: provides internet access to a VPC; a subnet is "public" because its route table has a route to an IGW.
- **NAT gateway notes (Extra depth):** is AZ-scoped (use one per AZ for HA); needs an Elastic IP; NAT gateway vs NAT instance is also a cost topic in Domain 4.

## 3. Rule processing logic and pitfalls
- **SGs**: stateful, allow-only, **no deny rules**. If inbound is allowed, the response goes back automatically.
- **NACLs**: stateless, so allow **both directions**, including **ephemeral ports (1024-65535)** for return traffic.
- NACL rules are checked **lowest number first**; **first match wins**; an unmatched packet hits the final `*` deny rule.
- Use SGs for fine-grained instance-level control and NACLs as a coarse subnet-level layer, for example to **block a specific IP range** (only NACLs can deny).
- **Layered together** = defense in depth: NACL at the subnet edge, SG at the instance, routes controlling reachability.
- Know the **base configuration** of the security filters for both the default and custom VPC (table in section 1).
- **Common pitfalls:** forgetting ephemeral ports in NACLs; forgetting the NAT gateway must sit in a public subnet with its own route to the IGW; private subnet route table missing the route to the NAT; overly broad rules like 0.0.0.0/0 on SSH.

## 4. Network segmentation: public vs private subnets
- **Public subnet**: route table has a route to an **internet gateway**; resources typically have public IPs. Common residents: load balancers, bastion hosts, NAT gateways.
- **Private subnet**: **no direct route** to the IGW; common residents: application servers, databases. Outbound internet through a **NAT gateway** if needed.
- Typical tiers: **web (public or behind a public ALB) → app (private) → database (private/isolated)**.
- Understand the strategy for **when to use** each and common practices around them.

### Scenario from the course: on-prem to private app servers over VPN
*Allow specific traffic from on-premises, over a VPN, to application servers in a private subnet, while keeping them safe from the public internet.*
- No IGW route for the private subnet; app servers have **no public IPs**.
- Create the VPN connection to the VPC (virtual private gateway or transit gateway) and ensure the subnet **route table** sends the on-prem CIDR to it (route propagation).
- **Security group**: allow inbound only the required ports from the **on-prem CIDR**.
- **NACL**: allow the on-prem CIDR in and the **ephemeral return traffic** out.
- Result: reachable from on-prem, invisible to the public internet, and no issues at VPC, subnet or instance level.

## 5. Service endpoints, PrivateLink, peering
- **VPC endpoints**: let resources reach AWS services **without** an internet gateway or NAT gateway, keeping traffic on the AWS network.
  - **Gateway endpoints**: for **S3 and DynamoDB**, implemented as a **route table target**, no hourly charge.
  - **Interface endpoints (powered by AWS PrivateLink)**: an **ENI with a private IP** in your subnet for most other services.
- **AWS PrivateLink** solves *exposing an application securely*:
  - Making an app public exposes it to the internet.
  - **VPC peering** adds **management overhead as you scale** and **exposes other applications in the peered VPCs** to each other. *(Extra depth: peering is non-transitive and requires non-overlapping CIDRs.)*
  - **PrivateLink** is a **secure, scalable** way to expose a service to **tens or hundreds of VPCs** (including other accounts) with **no peering, internet gateway or NAT gateway**.
- Also know **transit gateway** (hub-and-spoke connectivity) as a network connection tool deployed with VPCs.

## 6. Securing external network connections (to and from AWS)
| Option | What it is | Security / capacity / resilience notes |
|---|---|---|
| **Site-to-Site VPN** | IPsec tunnels between your network and AWS over the internet | Encrypted; two tunnels per connection for redundancy; limited bandwidth per tunnel (about 1.25 Gbps) |
| **Client VPN** | Managed VPN for **individual remote users** (OpenVPN-based) | User-level access into the VPC/on-prem |
| **AWS Direct Connect** | **Dedicated private** physical connection to AWS | Consistent performance; **not encrypted by default** (add VPN over Direct Connect or MACsec); resilience needs redundant connections/locations |

- Understand the **capacity, security and resilience options** for configuring each. Common pattern: Direct Connect primary with VPN backup.
- Also tied to Domain 3 and 4 (performance and cost choices between VPN, Direct Connect and internet).

## 7. Application-level security services and use cases
- **Amazon Macie**: uses **machine learning** to **discover, classify and protect sensitive data in Amazon S3**. This is the answer for **PII** (names, addresses, emails, SSN, driver license, passport, date of birth, bank and credit card details).
- **Amazon Cognito**: adds sign-up/sign-in and access control for apps. Know:
  - **User pools**: user directory and **authentication** (sign-up/sign-in, tokens), social and SAML IdPs.
  - **Identity pools**: grant **temporary AWS credentials** (authorization to AWS services), including guest users.
  - Cognito **brokers single sign-on / identity federation**.
- **Amazon GuardDuty**: **threat detection** by analyzing CloudTrail, VPC Flow Logs and DNS logs with ML and threat intelligence; no agents.
- Expect **scenario questions** on the use cases of these three.
- **Secure application access** also means: put apps behind load balancers, use private subnets, restrict SG sources, use IAM roles not keys.

## 8. Firewalls, proxies, DDoS and web attacks
- **Threat vectors external to AWS** named in the guide: **DDoS** and **SQL injection**.
- **AWS Shield**
  - **Shield Standard**: free, automatic, always on, protects against common network and transport layer (L3/L4) DDoS attacks.
  - **Shield Advanced**: paid, enhanced detection and mitigation, **24/7 DDoS response team** access, cost protection, extra protection for resources like ELB, CloudFront, Route 53, Global Accelerator, and Elastic IPs.
  - Know the **difference** between them.
- **AWS WAF**: a **layer 7 web application firewall** that filters HTTP(S) requests with rules (SQL injection, cross-site scripting, rate limiting, geo, IP lists).
  - Can **only be deployed on certain services**: **Application Load Balancer, Amazon API Gateway, Amazon CloudFront** (the course's list; AppSync and others have since been added).
  - Not directly on NLB or EC2.
- **Selection logic:** volumetric/network DDoS → **Shield**; malicious web requests/SQLi/XSS → **WAF**; both layered for defense in depth.
- Fundamentals to know: **firewalls and proxy servers**.

## 9. Secrets and application credentials
- **Never hard-code credentials**; use IAM roles, Secrets Manager or Parameter Store.
- **AWS Secrets Manager vs Systems Manager Parameter Store**
  | | Secrets Manager | Parameter Store |
  |---|---|---|
  | Purpose | Designed specifically for **secrets** | General configuration plus secrets (SecureString) |
  | **Automatic rotation** | **Yes**, forced at a chosen interval of days | No native rotation |
  | Integrations | Native database (RDS) rotation, cross-Region replication | Simple hierarchy of parameters |
  | Cost | Per secret + API calls | Standard tier is free |
  - Course rule: need **high-volume access with automatic credential rotation** → **Secrets Manager**.
- **AWS IAM Identity Center (SSO)** also integrates into securing application access (workforce SSO).
- Pick the best service for the stated requirement; know when one is better than the other.

## 10. Questions the course says to ask yourself
- How do you build **security into each networking tier**?
- How do you secure **application use across tiers**?
- What does **managing those security components** look like (rules, logging, change control)?
- Which service protects **PII**? (Macie)
- Which service stops **DDoS vs SQL injection**? (Shield vs WAF)
- Which service for **rotating secrets**? (Secrets Manager)
