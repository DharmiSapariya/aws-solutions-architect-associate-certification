# Domain 1 Self-Quiz / Flashcards

Cover the answers and test yourself. Questions come from the course's own prompts plus the exam guide.

## Task 1.1 Access
1. Is IAM zonal, regional or global? **Global.**
2. Why is the root user risky? **Full permissions that can't be limited; if compromised, the whole environment is.**
3. Two ways to protect the root user? **MFA, and use a limited IAM/Identity Center user instead for daily work.**
4. What permissions does a new IAM identity have? **None.**
5. What does least privilege do for blast radius? **Limits damage if an identity is compromised.**
6. User vs role? **User = long-term credentials; role = assumed, temporary credentials.**
7. What issues temporary credentials? **STS.**
8. How do federated corporate users get access? **An IAM role assumed via federation (SAML/Identity Center/AD).**
9. When should you hard-code credentials in an application? **Never.**
10. What is a policy? **An object that defines permissions when attached to an identity or resource.**
11. Identity vs resource policy? **Identity = what the identity can do; resource = who can access the resource.**
12. Which policy element is only in resource policies? **Principal.**
13. Which resources have resource policies? **S3, SQS, VPC endpoints, KMS keys (and more).**
14. How are overlapping allow/deny evaluated? **Explicit deny wins, otherwise explicit allow needed, default implicit deny.**
15. What gives traceability? **CloudTrail, CloudWatch, VPC Flow Logs (plus Config, GuardDuty).**
16. Multi-account services? **Organizations, SCPs, Control Tower.**

## Task 1.2 Workloads
17. Is a VPC zonal, regional or global? **Regional.**
18. Are subnets zonal, regional or global? **Zonal (AZ).**
19. What are the two VPC types? **Default and custom.**
20. SG: stateful or stateless? **Stateful, allow-only.**
21. NACL: stateful or stateless? **Stateless, allow and deny, ordered.**
22. What makes a subnet public? **A route to an internet gateway.**
23. How do private instances reach the internet? **NAT gateway in a public subnet.**
24. How to keep app servers private but reachable from on-prem via VPN? **No IGW route, route table to VPN, SG and NACL allow on-prem CIDR (and ephemeral ports).**
25. What is a VPC endpoint? **A gateway object for private access to AWS services without IGW/NAT.**
26. Why PrivateLink over peering? **Exposes just the service, scales to many VPCs/accounts, no peering/IGW/NAT.**
27. Name three external connection options. **Site-to-Site VPN, Client VPN, Direct Connect.**
28. Which service finds PII in S3? **Macie.**
29. Cognito user pool vs identity pool? **User pool authenticates users; identity pool grants temporary AWS credentials.**
30. GuardDuty does what? **Threat detection from logs using ML.**
31. Shield Standard vs Advanced? **Free automatic L3/L4 vs paid enhanced protection with response team and cost protection.**
32. Where can WAF be deployed? **ALB, API Gateway, CloudFront.**
33. Stop SQL injection? **WAF.**
34. Secrets with auto-rotation? **Secrets Manager.**

## Task 1.3 Data
35. At rest vs in transit? **Stored data protection vs data moving between parties.**
36. Define plaintext, algorithm, key, ciphertext. **Unencrypted data; code that encrypts; password-like secret; encrypted output.**
37. Symmetric vs asymmetric? **One key vs key pair.**
38. KMS vs CloudHSM? **Managed integrated vs dedicated single-tenant HSM; can combine with custom key store.**
39. Root key vs data key? **KMS key protects data keys; data keys encrypt data (envelope encryption).**
40. How are certificates managed for TLS? **ACM provisions and renews them.**
41. S3 client-side vs server-side? **Encrypt before upload vs S3 encrypts on arrival.**
42. Three SSE choices? **SSE-C, SSE-S3, SSE-KMS.**
43. Where do you get compliance reports? **AWS Artifact.**
44. Three principles of data security from the course? **Know the data/access, classify data, defense in depth (preventative + detective).**
45. EBS data needs protection and durability: easiest? **Encrypted EBS volume.**
46. Lifecycle rules vs Intelligent-Tiering? **Known patterns vs unknown/changing patterns.**
47. Four DR strategies? **Backup/restore, pilot light, warm standby, multi-site active-active.**
48. What determines recovery point? **Backup frequency.**
49. Which service centralizes backups? **AWS Backup.**
50. How to copy S3 objects to a DR Region? **Cross-Region Replication with versioning.**
51. Hybrid storage service? **Storage Gateway.**
