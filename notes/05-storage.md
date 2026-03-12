# AWS Storage Notes

S3 • EFS • FSx • EBS • Storage Gateway • DataSync

These notes capture key storage services across object, file, and block models, along with availability scope, encryption behavior, replication models, hybrid integration, and migration considerations.

The emphasis is on understanding how protocol, durability needs, operating system, and recovery scope influence storage selection.

---

# Amazon S3 (Object Storage)

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

S3 is AWS’s foundational object storage service.

- Region-scoped  
- 11 9’s durability  
- Strong read-after-write consistency  
- Virtually unlimited horizontal scaling  
- Not deployed inside a VPC  

---

## Storage Classes

| Class | Typical Use | Key Trait |
|--------|------------|-----------|
| Standard | Frequent access | Low latency |
| Standard-IA | Infrequent | 30-day minimum |
| One Zone-IA | Non-critical | Single AZ |
| Intelligent-Tiering | Unknown pattern | Auto tiering |
| S3 Glacier Instant Retrieval | Archive + fast retrieval | Millisecond restore |
| S3 Glacier Flexible Retrieval | Archive | Minutes–hours restore |
| S3 Glacier Deep Archive | Long-term archive | 12–48 hr restore |

Selection typically reflects access frequency and recovery time expectations.

> **EXAM TIP**
> Archive + fast restore → S3 Glacier Instant Retrieval.
> Very long-term archive → S3 Glacier Deep Archive.

---

## Performance & Scaling

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html

- At least **3,500 PUT/COPY/POST/DELETE requests per second per prefix**
- At least **5,500 GET/HEAD requests per second per prefix**
- S3 automatically scales as request rates increase
- Multiple prefixes allow horizontal scaling of request throughput
- Multipart upload recommended for large objects

Prefix distribution can improve parallel request throughput in extremely high-traffic workloads.

---

## Encryption

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html

Options include:

- **SSE-S3** – Encryption with S3-managed keys (simplest option)
- **SSE-KMS** – Encryption using AWS KMS keys with audit and access control
- **SSE-C** – Customer-provided encryption keys supplied with each request
- **Client-side encryption** – Data encrypted before upload; AWS never sees plaintext

With **SSE-KMS**, access depends on both IAM permissions and KMS key policy alignment.

> **EXAM TIP**  
> If access works but encryption fails, KMS key policy is often the cause.
> Audit logging or fine-grained key control → SSE-KMS

---

## Replication

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

- Same-Region Replication (SRR)  
- Cross-Region Replication (CRR)  
- Asynchronous  
- Versioning required  

Used for DR and compliance scenarios rather than strict zero RPO designs.

---

# Amazon EBS (Block Storage)

**Documentation:**  
https://docs.aws.amazon.com/ebs/

Persistent block storage for EC2.

- AZ-scoped  
- Low latency  
- Common for databases and OS volumes  

---

## Snapshots

- Incremental  
- Internally stored in S3  
- Cross-region copy supported  
- Cross-account sharing supported  

Snapshots form the basis of many backup & restore DR patterns.

> **EXAM TIP**  
> EBS volumes are AZ-scoped but internally replicated within the AZ for durability.
> Regional protection requires snapshot copy.

---

# Amazon EFS (Managed NFS File System)

**Documentation:**  
https://docs.aws.amazon.com/efs/

- Multi-AZ  
- Linux-only  
- POSIX compliant  
- Auto-scaling  

Commonly used for shared Linux workloads.

---

## Replication

**Documentation:**  
https://docs.aws.amazon.com/efs/latest/ug/efs-replication.html

- Asynchronous cross-region replication  
- RPO typically minutes  

> **EXAM TIP**  
> Shared Linux storage across instances → EFS.  
> Windows workloads generally point elsewhere.

---

# Amazon FSx Family (Specialized File Systems)

**Documentation:**  
https://docs.aws.amazon.com/fsx/

---

## FSx for Windows File Server

**Documentation:**  
https://docs.aws.amazon.com/fsx/latest/WindowsGuide/

- SMB protocol  
- Active Directory required  
- NTFS ACL support  

Often selected for Windows-native applications.

> **EXAM TIP**  
> Windows + SMB + AD integration → FSx Windows.

---

## FSx for Lustre

**Documentation:**  
https://docs.aws.amazon.com/fsx/latest/LustreGuide/

- High-performance  
- POSIX compliant  
- Native S3 integration  

Common in HPC, ML, and analytics workloads.

---

## FSx for NetApp ONTAP

**Documentation:**  
https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/

- Multi-protocol support  
- SnapMirror replication  
- Hybrid-friendly  

Often aligned with existing NetApp environments.

---

# Storage Gateway (Hybrid Integration)

**Documentation:**  
https://docs.aws.amazon.com/storagegateway/

Enables on-premises integration with AWS storage.

| Type | Purpose |
|------|---------|
| File Gateway | NFS/SMB → S3 |
| Volume Gateway | Block interface backed by S3 |
| Tape Gateway | Virtual tape library replacement |

Typically seen in hybrid modernization scenarios.

---

# AWS DataSync (Migration Service)

**Documentation:**  
https://docs.aws.amazon.com/datasync/

Accelerated data transfer service.

Supports:

- NFS  
- SMB  
- S3  
- EFS  
- FSx  

Optimized for file migration.

> **EXAM TIP**  
> File migration → DataSync.  
> Database migration → DMS.

---

# Object vs File vs Block

| Dimension | S3 | EFS | FSx | EBS |
|------------|----|-----|-----|-----|
| Storage Type | Object | File | File | Block |
| Protocol | REST | NFS | SMB/NFS | Mounted |
| Multi-AZ | Yes | Yes | Yes (varies by FSx type) | No |
| OS Support | Any | Linux | Windows/Linux | EC2 only |
| Typical Use | Data lake | Shared Linux | Enterprise file | Databases |

| FSx Type    | Multi-AZ          |
| ----------- | ----------------- |
| FSx Windows | Yes               |
| FSx ONTAP   | Yes               |
| FSx Lustre  | Usually Single-AZ |

---

# Storage Selection Patterns

- Object storage scenarios often align with S3  
- Shared Linux file storage commonly aligns with EFS  
- Shared Windows file workloads typically align with FSx Windows  
- High-performance analytics frequently align with FSx Lustre  
- Enterprise NetApp environments often align with FSx ONTAP  
- Database storage commonly aligns with EBS  
- Archive requirements align with S3 Glacier storage classes  
- Cross-region DR aligns with replication or snapshot copy  
- Hybrid file access aligns with Storage Gateway  

> **EXAM TIP**  
> Storage decisions are usually driven first by protocol (object, file, block), then by availability scope (AZ vs Region).

---

# Hybrid & Migration Patterns

| Scenario | Typical Service |
|-----------|----------------|
| On-prem NFS → AWS | DataSync |
| On-prem SMB → AWS | DataSync |
| Hybrid file interface | Storage Gateway |
| Large offline migration | Snowball |
| Database migration | DMS |

---

# Common Storage Pitfalls

- Assuming S3 resides inside a VPC  
- Ignoring KMS key policy alignment when using SSE-KMS  
- Using EFS for Windows workloads (Windows typically requires SMB → FSx for Windows)  
- Forgetting Active Directory requirement for FSx for Windows File Server  
- Expecting DataSync to migrate databases (it is optimized for file transfer)  
- Confusing Glacier storage class retrieval timelines  
- Assuming EBS volumes are multi-AZ (they are AZ-scoped)  
- Forgetting that S3 replication requires versioning enabled on both source and destination buckets  
- Treating asynchronous replication as zero-RPO protection 

---
# Design Considerations

When evaluating storage, considerations often include:

- Object vs file vs block model  
- AZ-level vs cross-region protection  
- Linux vs Windows workload  
- Throughput vs IOPS characteristics  
- DR expectations  
- Hybrid integration needs  
- Encryption scope and key management  

In most scenarios, storage selection naturally aligns with protocol and availability requirements rather than service familiarity.