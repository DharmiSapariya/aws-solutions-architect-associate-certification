# Domain 1 Practice: Takeaways and Traps (Day 3)

Sources: 5-question practice assessment (5/5), 3 walkthrough questions, 14 flashcards. Details are in `02-practice-questions/`.

## Patterns the exam keeps using
| If you see... | Answer pattern |
|---|---|
| "Same subnet, different rules per instance" | **Security groups**, not NACLs |
| "Block a specific IP" | **NACL** (or WAF at layer 7) |
| "Destination of suspicious traffic" | **VPC Flow Logs** |
| "Who made this API call" | **CloudTrail** |
| "Root user best practice" | **MFA + strong password, no access keys, no daily use** |
| "Regulatory + generate/store/control keys" | **KMS** (CloudHSM if dedicated HSM is required) |
| "Public HTTPS with auto-renewing certs" | **ACM public cert on ALB/CloudFront** |
| "Three-tier, internet only on HTTPS" | **IGW + public web subnet, private app/DB, chained SGs, 443 only** |
| "Fault tolerant within a Region" | **Multiple AZs** (plus ELB) |
| "Recover accidentally deleted objects, cheapest" | **S3 versioning** |
| "Can't be deleted for N years" | **S3 Object Lock** |
| "Rate-limit one IP" | **WAF rate-based rule** |
| "Rotate credentials automatically" | **Secrets Manager** |

## Traps worth remembering
- **HTTPS listener ≠ end-to-end encryption.** The ALB → target hop needs its own HTTPS if "all data in transit" is required. Same for CloudFront → origin.
- **Two subnets ≠ two AZs.** Subnets can share an AZ. HA needs separate AZs.
- **Virtual private gateway ≠ internet gateway.** VGW is VPN; IGW is internet.
- **Root user can't assume a role** in its own account; deleting the root account deletes everything; never use root access keys.
- **Security groups: allow-only, stateful, instance-level.** NACLs: stateless, subnet-level, ordered rules, allow and deny.
- **ACM is for certificates, not data keys.** KMS holds data keys.
- **Instance store is ephemeral**, so it never satisfies a durability requirement.
- **KMS key policies only govern keys held in KMS.**
- **S3 versioning doesn't block permanent deletion of a specific version** (needs permissions control, MFA Delete or Object Lock).
- **CRR is not an accidental-delete protection** and adds cost; only choose it for geographic redundancy.
- "Select two" questions have **no partial credit**: verify each choice meets every requirement.
- "**Most cost-effective**" in the stem usually rules out the multi-Region/replication-heavy option.

## Gaps to patch before the real exam
- Re-check exact current numbers (S3 object size limits, FIPS validation levels for KMS and CloudHSM) in AWS docs.
- Practice more questions mixing **NACL vs SG** and **KMS vs CloudHSM vs ACM vs Secrets Manager**.
- Do the Domain 1 SimuLearn lab to build the three-tier design in the console.
