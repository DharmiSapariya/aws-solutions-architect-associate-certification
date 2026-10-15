# Domain 1 Walkthrough Questions (3 from the course) and the Method

*Paraphrased gists and my own breakdown. Not AWS's wording.*

## The method the instructor uses (use it on the exam)
1. Read the **stem** (the question) fully.
2. Identify **key words and phrases**.
3. Decide **exactly what is being asked**; for "select two" there is **no partial credit**.
4. Predict an answer **before** reading the options.
5. Read all options; **eliminate distractors**.
6. **Re-read the plausible answers** and choose the best one. Any correct answer must meet **all** requirements in the stem.

## Walkthrough 1: root user secure login (select two) → MFA and strong passwords
- **Key words:** root user, best practices, secure login.
- **Correct:** (B) **enforce MFA** for root logins; (D) **enforce complex passwords** for the root user.
- **Distractors:**
  - *Use access keys for root:* programmatic root keys give **full access to everything including billing**, can't be reduced in permissions, and shouldn't be used.
  - *Root assumes a role in its own account:* **the root user can't assume a role** in its own account; roles are for services and cross-account/third-party access.
  - *Delete the root account:* **deleting it deletes all resources**, so you are no longer running in AWS.
- **Lesson:** root = MFA + strong password + no access keys + not for daily use.

## Walkthrough 2: two apps in the same subnet, different inbound rules → security groups
- **Key words:** **same subnet**, allow and block inbound, HTTP for app A, HTTPS only for app B.
- **Correct:** (B) **Security groups**: instance/ENI level, **allow-only**, so each instance gets its own rules and unmatched traffic is denied.
- **Distractors:**
  - *NACLs:* work at the **subnet boundary**; both apps are in the same subnet so they'd share one rule set.
  - *VPC peering:* connects VPCs, no traffic filtering.
  - *Route tables:* decide where traffic goes, not which traffic is blocked.
- **Lesson:** "same subnet" + different rules per instance → security groups.

## Walkthrough 3: regulatory key generation, storage and control → KMS
- **Key words:** **regulatory requirements**; **security and durability** in generating, **storing** and **controlling** cryptographic data keys; **most secure**.
- **Correct:** (A) use **AWS KMS** to generate KMS keys and data keys, and control access with **KMS key policies**.
- **Distractors:**
  - *Import KMS keys into ACM:* ACM is for **SSL/TLS certificates**, not data keys.
  - *Third-party keys on an instance store volume with IAM control:* instance store is **ephemeral**, so durability fails.
  - *OpenSSL keys uploaded to S3 with KMS key policies:* S3 can't do key management, and KMS key policies only apply to **keys held in KMS**.
- **Instructor's KMS vs CloudHSM point:** KMS uses HSMs but **AWS manages the encryption software**; with CloudHSM AWS provisions the hardware and **you use your own client** to do the encryption. Both are FIPS-validated; the course says **KMS level 2, CloudHSM level 3**. *AWS has since updated KMS HSM validation, so check current docs.*
- **Lesson:** "generate + store + control keys, durable, regulated" → KMS (CloudHSM if dedicated hardware is demanded).

## Next step noted by the course
After Domain 1, go to the first hands-on lab, **Design Secure Architectures** (SimuLearn), then Domain 2.
