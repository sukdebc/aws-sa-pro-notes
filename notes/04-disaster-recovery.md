# Multi-Region & Disaster Recovery Notes (SAP-C02 Level)

These notes summarize disaster recovery strategies, regional failover models, replication patterns, and global architecture considerations across AWS services.

The focus is on recovery objectives (RTO/RPO), replication scope, automation, and operational trade-offs.

---

# Disaster Recovery Strategy Spectrum

📘 AWS DR Whitepaper  
https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html  

Disaster recovery strategies range from lowest cost to lowest recovery time.

---

## Backup & Restore

Characteristics:

- Backups stored in S3 or snapshots
- Cross-region snapshot copy
- No active infrastructure in DR region
- Infrastructure provisioned after disaster

Typical profile:

- RTO: Hours
- RPO: Hours

Cost-efficient but slowest recovery.

Common services:

- EBS snapshots
- RDS snapshots
- AWS Backup
- S3 Cross-Region Copy

---

## Pilot Light

Characteristics:

- Minimal core infrastructure running in DR region
- Database replication active
- Application tier scaled up during failover

Typical profile:

- RTO: Tens of minutes
- RPO: Minutes

Often used with:

- Cross-region read replicas
- Aurora Global
- Infrastructure as Code (CloudFormation/Terraform)

---

## Warm Standby

Characteristics:

- Fully functional but scaled-down stack
- Continuous data replication
- Traffic shifted during failover

Typical profile:

- RTO: Minutes
- RPO: Seconds to minutes

Balances cost and speed.

---

## Active-Active (Multi-Site)

Characteristics:

- Full production stack in multiple regions
- Traffic served from all regions
- Global data replication

Typical profile:

- RTO: Near zero
- RPO: Near zero (depending on database)

Highest cost and complexity.

Requires conflict resolution strategy.

---

# Route 53 & Global Traffic Management

📘 Route 53 Routing  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html  

---

## Failover Routing

- Health-check based
- Primary/Secondary
- Common for pilot light and warm standby

---

## Latency Routing

- Routes to lowest RTT region
- Improves performance, not DR directly

---

## Geolocation Routing

- Based on user location
- Used for compliance or traffic isolation

---

## Weighted Routing

- Gradual traffic shift
- Canary or migration scenarios
- Useful for partial failover

---

## Multi-Value Answer

- Returns multiple healthy endpoints
- Basic DNS-level load balancing

---

# Global Accelerator vs Route 53

📘 Documentation  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html  

| Feature | Route 53 | Global Accelerator |
|----------|-----------|-------------------|
| Failover Type | DNS-based | Anycast IP |
| Propagation Time | DNS TTL dependent | Immediate |
| Protocol | DNS | TCP/UDP |
| Path | Public internet | AWS backbone |

Global Accelerator provides faster failover for TCP/UDP workloads.

---

# Relational Database DR Patterns

---

## RDS Multi-AZ

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html  

- Synchronous replication
- Same region only
- Automatic failover
- No regional protection

Multi-AZ ≠ Cross-region DR.

---

## RDS Cross-Region Read Replica

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html  

- Asynchronous replication
- Manual promotion required
- DR and read scaling

Replication lag affects RPO.

---

## Aurora Global Database

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html  

- Dedicated cross-region replication
- Typically <1 sec lag
- Fast failover (~30 sec)
- Single-writer model

Common low-RPO relational DR solution.

---

# S3 Cross-Region Replication (CRR)

📘 Documentation  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html  

Requirements:

- Versioning enabled
- IAM replication role
- Asynchronous replication

Important:

- No atomic cross-region consistency
- Replication delay possible
- Delete marker replication configurable

---

# DynamoDB Global Tables

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html  

- Active-active multi-region writes
- Last-writer-wins conflict resolution
- Near-zero RPO/RTO under normal conditions
- Streams + TTL replicated

Conflict resolution must be considered in design.

---

# EBS & EFS Replication

---

## EBS Snapshots

📘 Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-snapshots.html  

- Incremental snapshots
- Stored in S3
- Cross-region copy supported
- Not real-time replication

Used in backup & restore or pilot light.

---

## EFS Replication

📘 Documentation  
https://docs.aws.amazon.com/efs/latest/ug/efs-replication.html  

- Asynchronous cross-region replication
- RPO typically minutes
- Managed replication process

Not suitable for zero-data-loss systems.

---

# ElastiCache Global Datastore

📘 Documentation  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Redis-Global-Datastore.html  

- Cross-region replication
- Low-latency reads
- Promotion required
- RTO typically minutes

Used when multi-region cache locality needed.

---

# AWS Backup (Cross-Region DR)

📘 Documentation  
https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html  

AWS Backup supports:

- Cross-region copy
- Cross-account backup
- Centralized compliance
- Lifecycle management

Used in backup & restore DR strategies.

---

# Stateless vs Stateful Layer Strategy

---

## Stateless Layer

Common patterns:

- Deploy compute in multiple regions
- Use Route 53 or Global Accelerator
- Container-based multi-region scaling
- Infrastructure as Code provisioning

Stateless components are easier to globalize.

---

## Stateful Layer

Replication strategies may include:

- Aurora Global
- RDS Cross-region replicas
- DynamoDB Global Tables
- S3 CRR
- EFS replication
- Redis Global Datastore

State replication complexity drives DR design.

---

# Automation & Orchestration

Failover automation may use:

- Route 53 health checks
- CloudWatch alarms
- Lambda automation
- EventBridge
- Step Functions
- Aurora Global failover API

Automation reduces RTO but increases architectural complexity.

---

# Multi-AZ vs Cross-Region Comparison

| Feature | Multi-AZ | Cross-Region |
|----------|------------|----------------|
| Scope | Within region | Across regions |
| Protects Against | AZ failure | Regional failure |
| Replication | Synchronous | Usually asynchronous |
| Failover | Automatic | Often manual or orchestrated |
| RPO | Near zero | Seconds to minutes |

---

# DR Strategy Comparison

| Strategy | Infra in DR | RTO | RPO | Cost | Complexity |
|-----------|--------------|------|------|-------|------------|
| Backup & Restore | None | Hours | Hours | Lowest | Low |
| Pilot Light | Core only | Tens of minutes | Minutes | Low–Moderate | Moderate |
| Warm Standby | Full stack (scaled) | Minutes | Seconds–Minutes | Moderate | Moderate |
| Active-Active | Full stack live | Near zero | Near zero* | Highest | High |

\*Depends on replication model and conflict resolution.

---

# Control Plane vs Data Plane Consideration

Important nuance:

- Some services replicate data, not control plane config
- IAM, Route 53, CloudFront are global
- VPC, subnets, security groups are regional
- Infrastructure must be recreated in DR region

Infrastructure as Code (CloudFormation/Terraform) is critical for DR.

---

# Common Pitfalls in SAP-C02

- Assuming Multi-AZ equals cross-region DR
- Treating asynchronous replication as real-time
- Ignoring replication lag in RPO calculation
- Underestimating conflict resolution in Global Tables
- Assuming CloudFront provides full DR
- Forgetting to replicate secrets/config
- Ignoring region-specific service limitations
- Designing active-active without write conflict model
- Not planning DNS TTL in failover scenarios

---

# DR Decision Framing Model

Always evaluate:

- RTO (Recovery Time Objective)
- RPO (Recovery Point Objective)
- Replication scope (AZ vs region vs global)
- Write locality
- Conflict resolution strategy
- Automation level
- Cost tolerance
- Operational complexity

Multi-region architecture clarity comes from aligning workload criticality with the appropriate recovery tier.