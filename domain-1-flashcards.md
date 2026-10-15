# Domain 1 Flashcards (from the course) with extra clarifications

| # | Question | Answer | Extra note |
|---|----------|--------|------------|
| 1 | Protect S3 objects from deletion or overwrite? | **Turn on versioning and MFA Delete** | For retention-style protection add **S3 Object Lock** |
| 2 | Secure sensitive data on EBS volumes? | **Enable EBS encryption when creating the volume** | Or turn on default encryption per Region; to encrypt an existing volume: snapshot → encrypted copy → new volume |
| 3 | Service for short-term/temporary credentials? | **AWS STS** | AssumeRole, federation, cross-account |
| 4 | Permissions of a new IAM user, group or role? | **None; everything must be explicitly granted** | Implicit deny |
| 5 | Two policy types on an IAM role? | **Trust policy and permission policy** | Trust = who can assume; permissions = what it can do |
| 6 | Two main uses of SCPs? | **Allow list** (block by default, allow some services) and **deny list** (allow by default, block some) | SCPs never grant permissions on their own |
| 7 | NACL vs security group? | NACLs: subnet level, explicit allow **and deny**, **stateless**, processed **lowest rule number first**. SGs: ENI level, **stateful**, allow only, anything not allowed is denied | The card says SGs have an "explicit deny"; precisely, unmatched traffic is **implicitly** denied and SGs can't have deny rules |
| 8 | Two VPC endpoint types? | **Gateway endpoints**: S3 and DynamoDB, use **routing** (route table entry), support endpoint **policies**. **Interface endpoints**: everything else, ENI with **security groups**, use DNS | Pick the right interface endpoint per service |
| 9 | Encrypt and rotate DB credentials, API keys, secrets? | **AWS Secrets Manager** | Parameter Store has no native rotation |
| 10 | Serve SSL for multiple domains on one IP for a web app? | **AWS Certificate Manager** | The underlying mechanism is **SNI** (Server Name Indication) on the load balancer/CloudFront |
| 11 | Directory Service to use on-prem credentials for on-prem and AWS resources? | **AWS Directory Service for Microsoft Active Directory (AWS Managed Microsoft AD)** | Trust to on-prem AD; AD Connector is a proxy alternative |
| 12 | Limit requests from a single IP in a WAF rule? | **Rate-based rule with a rate limit** | |
| 13 | Why does the OSI model matter for firewalls? | What a firewall can inspect depends on the **layer** it works at; **higher layers need more compute and cost more** | WAF = **layer 7**; security groups/NACLs = layers 3-4; Shield Standard = L3/L4 |
| 14 | (Course emphasis) Know your layers | See above | NLB = L4, ALB = L7, Gateway LB = L3 appliance traffic |
