# AWS Storage Notes (SAP-C02 Level)

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
| Glacier Instant | Archive + fast retrieval | Millisecond restore |
| Glacier Flexible | Archive | Minutes–hours restore |
| Glacier Deep Archive | Long-term archive | 12–48 hr restore |

Selection typically reflects access frequency and recovery time expectations.

> **EXAM TIP**  
> Archive + fast restore → Glacier Instant.  
> Very long-term archive → Deep Archive.

---

## Performance & Scaling

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html

- 3,500 PUT/COPY/POST/DELETE per prefix  
- 5,500 GET/HEAD per prefix  
- Automatic scaling  
- Multipart upload recommended for large objects  

Prefix distribution improves parallel access.

---

## Encryption

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html

Options include:

- SSE-S3  
- SSE-KMS  
- SSE-C  
- Client-side encryption  

With SSE-KMS, access depends on both IAM permissions and KMS key policy alignment.

> **EXAM TIP**  
> If access works but encryption fails, KMS key policy is often the cause.

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
> EBS protects within an AZ.  
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
| Multi-AZ | Yes | Yes | Yes (varies) | No |
| OS Support | Any | Linux | Windows/Linux | EC2 only |
| Typical Use | Data lake | Shared Linux | Enterprise file | Databases |

---

# Storage Selection Patterns

- Object storage scenarios often align with S3  
- Shared Linux file storage commonly aligns with EFS  
- Shared Windows file workloads typically align with FSx Windows  
- High-performance analytics frequently align with FSx Lustre  
- Enterprise NetApp environments often align with FSx ONTAP  
- Database storage commonly aligns with EBS  
- Archive requirements align with Glacier classes  
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
- Ignoring KMS key policy alignment  
- Using EFS for Windows workloads  
- Forgetting AD requirement for FSx Windows  
- Expecting DataSync to migrate databases  
- Confusing Glacier retrieval timelines  
- Assuming EBS is multi-AZ  
- Forgetting versioning for S3 replication  
- Treating asynchronous replication as zero RPO  

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