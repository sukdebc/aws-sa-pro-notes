# Resilience & High Availability Patterns

This document brings together compute, database, networking, storage, and serverless patterns into practical resilience architectures.

Focus areas:

- RTO / RPO alignment  
- Multi-AZ vs Multi-Region scope  
- Failure domains  
- Stateless vs stateful tiers  
- Automated failover  
- Graceful degradation  

Resilience is generally about removing single points of failure while keeping recovery objectives aligned with business impact.

---

# Failure Domains

Resilience thinking often starts with blast radius.

Common failure domains:

- Instance-level  
- Availability Zone (AZ)-level  
- Region-level  
- Account-level  

High availability typically means ensuring that a single failure in any one of these domains does not cause total service disruption.

---

# Multi-AZ Patterns

Multi-AZ design protects against Availability Zone failure within a region.

---

## Compute

**Auto Scaling Documentation:**  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html  

**ALB Documentation:**  
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html  

Common patterns:

- Auto Scaling Groups spanning multiple AZs  
- Application Load Balancer distributing traffic  
- Avoiding single-AZ dependencies  
- Externalizing session state (Redis / DynamoDB)  

> **EXAM TIP**  
> Single-AZ EC2 in production is usually a red flag.

---

## Database

**RDS Multi-AZ:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html  

**Aurora Replication:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html  

Patterns:

- RDS Multi-AZ for synchronous standby  
- Aurora replicas across AZs  
- ElastiCache Multi-AZ (Redis)  

Multi-AZ improves availability within a region but does not provide regional disaster protection.

> **EXAM TIP**  
> Multi-AZ = AZ protection, not regional DR.

---

## Storage

**S3 Durability:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html  

- S3 is multi-AZ by default  
- EFS is regional (multi-AZ)  
- EBS is single-AZ  

Understanding storage failure scope influences recovery design.

---

# Stateless vs Stateful Design

Stateless tiers generally scale and fail independently.

Stateful tiers usually require replication or recovery strategies.

| Layer | Typical Pattern |
|--------|----------------|
| Web/API | Stateless, ASG + ALB |
| Session | Redis / DynamoDB |
| Database | Multi-AZ or global replication |
| Files | S3 or EFS |

Externalizing state typically improves resilience.

> **EXAM TIP**  
> If sessions are stored locally, failover usually breaks user experience.

---

# Multi-Region Patterns

Multi-region designs address regional failure scenarios.

---

## Active-Passive

**Route 53 Failover:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html  

- Primary region serves traffic  
- Secondary region stands by  
- DNS-based failover  
- Lower cost than active-active  

RTO often depends on automation and DNS TTL.

---

## Active-Active

**Global Accelerator:**  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html  

- Multiple regions serve traffic  
- Latency-based routing or Anycast IP  
- Requires replicated data layer  
- Higher complexity  

Active-active usually requires clear write conflict handling.

> **EXAM TIP**  
> Near-zero RTO + global traffic → Active-active, but data model must support it.

---

# Data Layer Resilience

The data layer often determines achievable RTO and RPO.

---

## Aurora Global Database

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html  

- Sub-second cross-region replication  
- Fast failover  
- Single-writer model  

Common in low-RPO relational DR designs.

---

## DynamoDB Global Tables

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html  

- Multi-region writes  
- Last-writer-wins conflict resolution  
- Near-zero RPO under normal operation  

Conflict resolution considerations influence design.

---

## S3 Cross-Region Replication

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html  

- Asynchronous  
- Used for DR and compliance  
- Not strongly consistent across regions  

---

# Serverless Resilience

---

## AWS Lambda

**Documentation:**  
https://docs.aws.amazon.com/lambda/latest/dg/welcome.html  

- Multi-AZ by default  
- Reserved concurrency can protect downstream services  
- DLQ or failure destinations supported  
- Idempotency patterns reduce retry impact  

> **EXAM TIP**  
> Async Lambda without DLQ can hide failure until too late.

---

## Step Functions

**Documentation:**  
https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html  

- Built-in retry and backoff  
- Catch and fallback states  
- Centralized workflow orchestration  

Workflow-level retries often simplify resilience logic.

---

# Queue-Based Resilience

**SQS Documentation:**  
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html  

Pattern:

API → Lambda → SQS → Worker Lambda  

Common effects:

- Traffic smoothing  
- Failure isolation  
- Controlled retry  
- DLQ integration  

> **EXAM TIP**  
> When protecting a database from spikes, queue buffering is often the simplest resilience layer.

---

# Circuit Breaker Pattern

Applied when downstream instability may cascade.

Approaches include:

- Exponential backoff retries  
- Timeout + fallback logic  
- Queue-based decoupling  
- Graceful fallback responses  

Helps prevent system-wide failure propagation.

---

# Graceful Degradation

When part of the system fails:

- Cached content served via CloudFront  
- Read-only mode activated  
- Non-critical features disabled  
- Static fallback responses used  

Partial availability is often preferable to total outage.

---

# Backup & Restore

**AWS Backup Documentation:**  
https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html  

Examples:

- RDS automated backups  
- EBS snapshots  
- S3 versioning  
- Centralized AWS Backup policies  

Backup improves durability but does not guarantee fast recovery.

> **EXAM TIP**  
> Backup ≠ High Availability.

---

# Health Checks & Failover

| Layer | Mechanism |
|--------|-----------|
| DNS | Route 53 health checks |
| Compute | ALB target health |
| Multi-Region | Global Accelerator |
| Hybrid | BGP failover |
| Application | Synthetic monitoring |

Automated failover generally reduces recovery time compared to manual processes.

---

# Observability for Resilience

**CloudWatch Documentation:**  
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html  

Components often include:

- CloudWatch alarms  
- AWS Health events  
- X-Ray tracing  
- Structured logs  
- Synthetic canaries  

Resilience assumptions are usually validated through monitoring and testing.

---

# Cost vs Availability Trade-Off

| Strategy | Cost | RTO | RPO |
|----------|------|------|------|
| Backup & Restore | Low | High | High |
| Pilot Light | Medium | Medium | Medium |
| Warm Standby | High | Low | Low |
| Active-Active | Highest | Near Zero | Near Zero |

Availability level typically reflects business impact tolerance.

---

# Common Pitfalls

- Confusing backup with high availability  
- Running production databases in a single Availability Zone  
- Ignoring replication scope (AZ vs Region)  
- Storing session state locally on application servers  
- Relying on manual failover instead of automation  
- Overlooking DNS TTL impact during failover  
- Not regularly testing recovery procedures
---

# Design Considerations

Resilience design is often evaluated across:

- Required RTO and RPO  
- Failure domain scope (AZ vs Region)  
- Stateless vs stateful tier behavior  
- Data replication model  
- Automation level  
- Observability coverage  
- Cost tolerance  

Resilience is usually achieved through layered decisions rather than a single service choice.