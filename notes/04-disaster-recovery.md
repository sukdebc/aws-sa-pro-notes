# Multi-Region & Disaster Recovery Notes

These notes summarize disaster recovery strategies, regional failover models, replication patterns, automation considerations, and global architecture trade-offs across AWS services.

The emphasis is on:

- RTO / RPO alignment  
- Replication scope (AZ vs Region vs Global)  
- Automation level  
- Data consistency model  
- Operational complexity vs cost  

---

# Disaster Recovery Strategy Spectrum

**Documentation (AWS DR Whitepaper):**  
https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html

DR strategies exist on a spectrum — from lowest cost to lowest recovery time.

---

## 1️⃣ Backup & Restore

Characteristics:

- Backups stored in S3 or snapshots  
- Cross-region snapshot copy  
- No running infrastructure in DR region  
- Infrastructure provisioned after disaster  

Typical profile:

- **RTO:** Hours  
- **RPO:** Hours  

Common services:

- EBS snapshots  
- RDS snapshots  
- AWS Backup  
- S3 replication or backup copy  

Cost-efficient but slowest recovery model.

> **EXAM TIP**  
> Low cost + longer recovery acceptable → Backup & Restore.

---

## 2️⃣ Pilot Light

Characteristics:

- Core infrastructure running in DR region  
- Database replication active  
- Application tier scaled up during failover  

Typical profile:

- **RTO:** Tens of minutes  
- **RPO:** Minutes  

Common components:

- Cross-region read replicas  
- Aurora Global Database  
- Infrastructure as Code  

> **EXAM TIP**  
> Minimal always-on DR footprint + faster recovery → Pilot Light.

---

## 3️⃣ Warm Standby

Characteristics:

- Fully functional but scaled-down stack  
- Continuous replication  
- Traffic switched during failover  

Typical profile:

- **RTO:** Minutes  
- **RPO:** Seconds to minutes  

Balanced cost vs recovery speed.

> **EXAM TIP**  
> “Must recover in minutes” but not near-zero RTO → Warm Standby.

---

## 4️⃣ Active-Active (Multi-Site)

Characteristics:

- Full production stack in multiple regions  
- Traffic served from all regions  
- Continuous data replication  

Typical profile:

- **RTO:** Near zero  
- **RPO:** Near zero (depends on database model)  

Highest cost and operational complexity.

Requires:

- Conflict resolution model  
- Write locality planning  
- Consistency trade-offs  

> **EXAM TIP**  
> Near-zero RTO + global traffic → Active-Active.  
> Always check write-conflict handling.

---

# Route 53 & Global Traffic Management

**Documentation:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

---

## Failover Routing

- Primary / Secondary  
- Health check based  
- Common in pilot light and warm standby  

> **EXAM TIP**  
> DR failover via DNS → Route 53 Failover Policy.

---

## Weighted Routing

- Gradual traffic shift  
- Canary or partial failover  

Useful during DR testing or migration.

---

## Latency Routing

- Routes to lowest RTT region  
- Performance optimization, not full DR strategy  

---

## Geolocation Routing

- Based on user location  
- Often used for compliance requirements  

---

# Global Accelerator vs Route 53

**Documentation:**  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html

| Feature | Route 53 | Global Accelerator |
|----------|-----------|-------------------|
| Failover | DNS-based | Anycast IP |
| Propagation | DNS TTL dependent | Immediate |
| Protocol | DNS | TCP/UDP |
| Path | Internet | AWS backbone |

Global Accelerator provides faster failover for TCP/UDP workloads.

> **EXAM TIP**  
> Need static IP + fast failover → Global Accelerator.  
> DNS-only switching → Route 53.

---

# Relational Database DR Patterns

---

## RDS Multi-AZ

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html

- Synchronous replication  
- Same region only  
- Automatic failover  

Does NOT protect against regional failure.

> **EXAM TIP**  
> Multi-AZ ≠ Cross-region DR.

---

## RDS Cross-Region Read Replica

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html

- Asynchronous replication  
- Manual promotion required  
- Replication lag affects RPO  

Used for DR + read scaling.

---

## Aurora Global Database

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html

- Dedicated cross-region replication  
- Typically <1 second lag  
- Fast failover (~30 seconds)  
- Single writer model  

Common low-RPO relational DR design.

> **EXAM TIP**  
> Low RPO + fast cross-region recovery → Aurora Global.

---

# DynamoDB Global Tables

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html

- Active-active multi-region writes  
- Last-writer-wins conflict resolution  
- Near-zero RPO/RTO under normal operation  

Conflict resolution must be understood.

> **EXAM TIP**  
> Multi-region writes → DynamoDB Global Tables.  
> Always consider conflict model.

---

# S3 Cross-Region Replication (CRR)

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

Requirements:

- Versioning enabled  
- IAM replication role  
- Asynchronous replication  

Important:

- Not atomic across regions  
- Replication delay possible  
- Delete marker replication configurable  

> **EXAM TIP**  
> S3 CRR = Asynchronous.  
> Not guaranteed zero RPO.

---

# EBS & EFS Replication

---

## EBS Snapshots

**Documentation:**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-snapshots.html

- Incremental  
- Cross-region copy supported  
- Not real-time replication  

Used in backup & restore or pilot light strategies.

---

## EFS Replication

**Documentation:**  
https://docs.aws.amazon.com/efs/latest/ug/efs-replication.html

- Asynchronous cross-region replication  
- RPO typically minutes  

Not suitable for zero data loss systems.

---

# ElastiCache Global Datastore

**Documentation:**  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Redis-Global-Datastore.html

- Cross-region replication  
- Promotion required  
- RTO typically minutes  

Primarily for read locality + regional DR.

---

# AWS Backup (Cross-Region Strategy)

**Documentation:**  
https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html

Supports:

- Cross-region copy  
- Cross-account backup  
- Lifecycle management  
- Centralized compliance  

Used mainly for Backup & Restore strategy.

---

# Stateless vs Stateful DR Strategy

---

## Stateless Layer

- Deploy compute in multiple regions  
- Use Route 53 or Global Accelerator  
- Use IaC for consistent provisioning  

Stateless components are easier to globalize.

---

## Stateful Layer

Replication options:

- Aurora Global  
- RDS replicas  
- DynamoDB Global Tables  
- S3 CRR  
- EFS replication  
- Redis Global Datastore  

State replication drives DR complexity.

> **EXAM TIP**  
> DR complexity = Data replication complexity.

---

# Automation & Orchestration

Automation tools:

- Route 53 health checks  
- CloudWatch alarms  
- Lambda  
- EventBridge  
- Step Functions  
- Aurora failover API  

Automation reduces RTO but increases architecture complexity.

> **EXAM TIP**  
> Fully automated failover → Higher complexity but lower RTO.

---

# Multi-AZ vs Cross-Region

| Feature | Multi-AZ | Cross-Region |
|----------|------------|----------------|
| Scope | Within region | Across regions |
| Protects Against | AZ failure | Regional failure |
| Replication | Synchronous | Usually asynchronous |
| Failover | Automatic | Often manual |
| RPO | Near zero | Seconds–Minutes |

---

# DR Strategy Comparison

| Strategy | Infra in DR | RTO | RPO | Cost | Complexity |
|-----------|--------------|------|------|-------|------------|
| Backup & Restore | None | Hours | Hours | Low | Low |
| Pilot Light | Core only | Tens of minutes | Minutes | Low–Moderate | Moderate |
| Warm Standby | Scaled-down full stack | Minutes | Seconds–Minutes | Moderate | Moderate |
| Active-Active | Full stack | Near zero | Near zero* | High | High |

\*Depends on replication model.

---

# Control Plane vs Data Plane Considerations

Important nuance:

- IAM, Route 53, CloudFront are global services  
- VPC, Subnets, Security Groups are regional  
- Data replication ≠ Infrastructure replication  

Infrastructure must be recreated in DR region.

Infrastructure as Code is critical for reliable DR execution.

> **EXAM TIP**  
> Replicated data alone is not DR.  
> Infrastructure automation is required.

---

# Common Pitfalls

- Assuming Multi-AZ equals regional protection  
- Ignoring replication lag in RPO calculation  
- Designing active-active without conflict resolution  
- Forgetting DNS TTL impact during failover  
- Assuming CloudFront equals DR  
- Ignoring secrets/config replication  
- Not planning failover automation  

---

# DR Decision Framing Model

When evaluating DR:

- What is required RTO?  
- What is acceptable RPO?  
- Is regional failure in scope?  
- Is write conflict possible?  
- What is cost tolerance?  
- How much automation is required?  

Multi-region design should match the business impact and recovery needs — not automatically default to active-active.