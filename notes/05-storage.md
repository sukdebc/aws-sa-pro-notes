# AWS Storage Notes (SAP-C02 Level)

S3 • EFS • FSx • EBS • Storage Gateway • DataSync

These notes summarize object, file, and block storage services in AWS, including cross-account access, encryption models, hybrid integration, and migration patterns.

The emphasis is on selecting the correct storage architecture based on protocol, performance, operating system, availability scope, and multi-region strategy.

---

# 1️⃣ Amazon S3

📘 Docs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

S3 is the foundational object storage service in AWS.

- Region-scoped
- 11 9’s durability
- Horizontally scalable
- Strong read-after-write consistency (for PUT and DELETE)

S3 is not inside a VPC.

---

## S3 Storage Classes

| Storage Class | Typical Use Case | Key Characteristics |
|---------------|------------------|--------------------|
| Standard | Frequently accessed data | Low latency, high durability |
| Standard-IA | Infrequent access | 30-day minimum, retrieval fee |
| One Zone-IA | Non-critical infrequent data | Single AZ |
| Intelligent-Tiering | Unknown access patterns | Auto tier movement |
| Glacier Instant Retrieval | Archive with ms retrieval | Lower cost than IA |
| Glacier Flexible Retrieval | Archive, minutes-hours retrieval | Restore required |
| Glacier Deep Archive | Rare access | 12–48 hour retrieval |

Storage class selection is driven by access frequency and RTO requirements.

---

## Performance & Scaling

📘 Performance Docs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html

- 3,500 PUT/COPY/POST/DELETE per prefix
- 5,500 GET/HEAD per prefix
- Automatic scaling
- Multipart upload recommended for >100 MB
- S3 Select allows partial object retrieval

Prefix distribution improves parallel performance.

---

## Consistency Model

- Strong consistency for new objects
- Strong consistency for overwrite and delete
- No eventual consistency model anymore

Important for data lake and analytics design.

---

## Encryption Options

📘 Encryption Docs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html

- Client-side encryption
- SSE-S3 (AWS-managed)
- SSE-KMS (KMS-managed)
- SSE-C (customer-provided key)

SSE-KMS requires:

- IAM permission
- KMS key policy permission

---

## Access & Networking

📘 VPC Endpoint Docs: https://docs.aws.amazon.com/vpc/latest/privatelink/

Important facts:

- Gateway Endpoint enables private access
- S3 Access Points simplify multi-tenant access
- Block Public Access overrides bucket policy
- Cross-account access requires:
  - Bucket policy allow
  - IAM allow
  - KMS key policy (if encrypted)

---

## Replication

📘 Replication Docs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

- Cross-Region Replication (CRR)
- Same-Region Replication (SRR)
- Asynchronous
- Versioning required
- Replicates new objects only

Used for DR and compliance, not strong consistency.

---

# 2️⃣ Amazon EFS

📘 Docs: https://docs.aws.amazon.com/efs/

EFS provides fully managed NFS-based file storage.

- Multi-AZ
- POSIX-compliant
- Auto scaling
- Designed for Linux workloads

---

## Performance Modes

| Mode | Use Case |
|------|----------|
| General Purpose | Low-latency apps |
| Max I/O | Highly parallel workloads |

---

## Throughput Modes

- Bursting (default)
- Provisioned
- Elastic throughput

---

## EFS Replication

📘 Docs: https://docs.aws.amazon.com/efs/latest/ug/efs-replication.html

- Asynchronous cross-region replication
- Typical RPO ~15 minutes

Used for DR scenarios.

---

# 3️⃣ Amazon FSx Family

📘 Docs: https://docs.aws.amazon.com/fsx/

FSx provides managed file systems optimized for specific workloads.

---

## FSx for Windows File Server

📘 Docs: https://docs.aws.amazon.com/fsx/latest/WindowsGuide/

- SMB protocol
- Active Directory integration required
- NTFS ACL support
- Multi-AZ deployment option

Best for Windows-native workloads.

---

## FSx for Lustre

📘 Docs: https://docs.aws.amazon.com/fsx/latest/LustreGuide/

- High-performance HPC workloads
- POSIX compliant
- Native S3 integration
- Scratch and Persistent deployment options

Common in ML and analytics.

---

## FSx for NetApp ONTAP

📘 Docs: https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/

- Multi-protocol (NFS, SMB, iSCSI)
- SnapMirror support
- Enterprise hybrid integration
- Storage efficiency features

Ideal for NetApp-heavy environments.

---

# 4️⃣ Amazon EBS

📘 Docs: https://docs.aws.amazon.com/ebs/

EBS provides persistent block storage for EC2.

- AZ-scoped
- Low latency
- Suitable for databases and OS volumes

---

## Volume Types

| Type | Use Case |
|------|----------|
| gp3 | General-purpose SSD |
| io1/io2 | High IOPS workloads |
| st1 | Throughput-intensive HDD |
| sc1 | Cold HDD |

---

## Multi-Attach

- Supported for io1/io2
- Allows attachment to multiple instances
- Limited use cases (clustered apps)

---

## Snapshots

- Incremental
- Stored in S3 internally
- Cross-account sharing supported
- Cross-region copy supported

Snapshots enable DR and backup strategy.

---

# 5️⃣ Storage Gateway

📘 Docs: https://docs.aws.amazon.com/storagegateway/

Enables hybrid integration.

---

## Gateway Types

| Type | Purpose |
|------|---------|
| File Gateway | NFS/SMB → S3 |
| Volume Gateway | Block interface backed by S3 |
| Tape Gateway | Virtual tape library |

Used in hybrid modernization.

---

# 6️⃣ AWS DataSync

📘 Docs: https://docs.aws.amazon.com/datasync/

Accelerated data transfer service.

Supports:

- NFS
- SMB
- S3
- EFS
- FSx
- On-prem → AWS
- AWS → AWS

Optimized for file migration, not database replication.

---

# 7️⃣ Object vs File vs Block

| Dimension | S3 | EFS | FSx | EBS |
|------------|----|-----|-----|-----|
| Storage Type | Object | File | File | Block |
| Protocol | REST | NFS | SMB/NFS | Mounted volume |
| Multi-AZ | Yes | Yes | Yes (varies) | No |
| OS Support | Any | Linux | Windows/Linux | EC2 only |
| Best For | Data lake, static content | Shared Linux storage | Specialized file workloads | Databases |

---

# 8️⃣ Hybrid & Migration Patterns

| Scenario | Recommended Service |
|-----------|---------------------|
| On-prem NFS → AWS | DataSync |
| On-prem SMB → AWS | DataSync |
| Hybrid file interface | Storage Gateway |
| Massive offline migration | Snowball |
| Database migration | DMS (not DataSync) |

---

# 9️⃣ Storage Selection Patterns

- Shared Linux storage → EFS
- Shared Windows storage → FSx Windows
- HPC → FSx Lustre
- NetApp hybrid → FSx ONTAP
- Object storage → S3
- Database volumes → EBS
- Archive → Glacier classes
- Cross-region DR → Replication or Snapshot copy

Selection depends on:

- Protocol
- Latency
- Multi-AZ scope
- Cross-region requirements
- OS compatibility
- Hybrid integration

---

# 🔟 Common Pitfalls

- Assuming S3 is inside a VPC
- Ignoring KMS key policy alignment
- Using EFS for Windows workloads
- Forgetting Active Directory for FSx Windows
- Expecting DataSync to migrate databases
- Confusing Glacier retrieval times
- Assuming EBS is multi-AZ
- Ignoring S3 prefix performance patterns
- Forgetting versioning for S3 replication

---

# Architectural Lens

Before choosing storage:

1. Object vs file vs block?
2. Multi-AZ required?
3. Cross-region required?
4. Linux or Windows workload?
5. Throughput vs IOPS priority?
6. Hybrid integration needed?
7. Encryption scope?
8. Recovery objective?

Correct storage choice is almost always protocol + availability driven.