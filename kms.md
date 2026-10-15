# AWS Key Management Service (KMS)
*Category: Security | Scope: Regional (multi-Region keys available) | Domain 1 (Task 1.3)*

## What it is
Managed service to **create and control cryptographic keys** and use them to encrypt data across AWS services and your applications.

## Key features and how it works
- **Key types**: **AWS owned keys** (invisible), **AWS managed keys** (`aws/s3` and so on; auto-rotated on a fixed schedule), **customer managed keys** (you control policy, rotation, enable/disable, deletion).
- **Symmetric** (AES-256-GCM, most services) and **asymmetric** (RSA/ECC for encrypt or sign) KMS keys; **HMAC** keys.
- **Envelope encryption**: the KMS key never leaves KMS; it generates/encrypts **data keys**, and data keys encrypt the data. `Encrypt` directly handles only small payloads (4 KB).
- **Key policy** is the primary access control (every key has one); can also use **IAM policies** and **grants**. Separate **key administrators** from **key users**.
- **Rotation**: customer managed symmetric keys support optional automatic rotation (yearly by default, configurable) and on-demand rotation; **asymmetric, HMAC and imported-material keys rotate manually**. (Verify current docs.)
- **Multi-Region keys** share key material/ID across Regions for cross-Region encryption.
- **Custom key store** backed by **CloudHSM** (or external key manager). **Imported key material** supported.
- Deleting a key needs a **7 to 30 day waiting period**. Every use is logged in **CloudTrail**. FIPS 140-validated HSMs.
- Integrated with S3, EBS, RDS, DynamoDB, EFS, Redshift, Secrets Manager, and more. Per-Region request quotas apply.

## When the exam wants it
- "Encrypt data at rest", "control and audit key usage", "encrypt EBS/S3/RDS with your own key", "rotate keys".

## Don't confuse with
- **CloudHSM** (dedicated HSM), **ACM** (certificates for TLS), **Secrets Manager** (secrets).

## Gotchas
Keys are regional (copying encrypted snapshots across Regions needs re-encryption with a key in the target Region); a deleted key makes data unrecoverable; SSE-KMS at very high request rates can hit KMS quotas.

## From the Domain 1 course
"Why KMS instead of CloudHSM, and how can you use them together? How do you manage keys across Regions? What types of keys exist and how often can each rotate? How do you handle root keys vs data keys?" Exam guide skills: encrypt data at rest with KMS, implement access policies for keys, rotate keys.

## Added from Domain 1 practice (walkthrough 3)
- Course framing: with **KMS**, AWS manages the encryption software on HSMs; with **CloudHSM**, AWS provisions the hardware and **you use your own client**. The course says KMS is **FIPS 140-2 level 2** and CloudHSM **level 3**; AWS has since updated KMS HSM validation, so verify the current level in the docs.
- Regulatory + durable generation, storage and control of data keys → **KMS with key policies**.
