# Resilience & High Availability Patterns (SAP-C02 Level)

This document synthesizes compute, database, networking, storage, and serverless patterns into practical resilience architectures.

Focus areas:

- RTO / RPO alignment
- Multi-AZ vs Multi-Region
- Failure domains
- Stateless vs stateful tiers
- Automated failover
- Graceful degradation

Resilience is about eliminating single points of failure while aligning cost with business recovery objectives.

---

# 1️⃣ Failure Domains

Design begins by identifying blast radius.

Failure domains include:

- Instance-level
- Availability Zone (AZ)-level
- Region-level
- Account-level

High availability means removing single points of failure within each domain.

---

# 2️⃣ Multi-AZ Patterns

Multi-AZ design protects against Availability Zone failures.

---

## Compute

📘 Auto Scaling Docs:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html

📘 ALB Docs:  
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

Best practices:

- Deploy Auto Scaling Groups across multiple AZs
- Use Application Load Balancer to distribute traffic
- Avoid single-AZ EC2 dependencies
- Store sessions externally (Redis/DynamoDB)

---

## Database

📘 RDS Multi-AZ:  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html

📘 Aurora Replicas:  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html

Patterns:

- RDS Multi-AZ for synchronous standby failover
- Aurora replicas across AZs
- ElastiCache Multi-AZ (Redis cluster mode)

Multi-AZ improves availability within a region but does not protect against regional outages.

---

## Storage

📘 S3 Resilience:  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html

- S3 is multi-AZ by default
- EFS is regional (multi-AZ)
- EBS is single-AZ (requires snapshot for recovery)

Understanding storage failure scope is critical.

---

# 3️⃣ Stateless vs Stateful Design

Stateless tiers scale and fail independently.

Stateful tiers require replication or recovery strategies.

| Layer | Strategy |
|--------|----------|
| Web/API | Stateless, ASG + ALB |
| Session | Redis / DynamoDB |
| Database | Multi-AZ or Global replication |
| Files | S3 or EFS |

Externalizing state increases resilience.

---

# 4️⃣ Multi-Region Patterns

Multi-region protects against regional failure.

---

## Active-Passive

📘 Route 53 Failover:  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html

- Primary region serves traffic
- Secondary region is warm standby
- Route 53 failover routing
- Lower cost than active-active

RTO depends on automation and DNS TTL.

---

## Active-Active

📘 Global Accelerator:  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html

- Multiple regions serve traffic
- Latency-based routing or Global Accelerator
- Requires globally replicated data layer
- Higher cost and complexity

Active-active requires conflict resolution strategy.

---

# 5️⃣ Data Layer Resilience

Data replication often determines achievable RTO/RPO.

---

## Aurora Global Database

📘 Docs:  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html

- Sub-second cross-region replication
- Fast failover (~30 seconds)
- Primary-writer model

Suitable for low-RPO relational DR.

---

## DynamoDB Global Tables

📘 Docs:  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html

- Multi-master replication
- Near-zero RPO
- Last-writer-wins conflict resolution

Designed for active-active workloads.

---

## S3 Cross-Region Replication (CRR)

📘 Docs:  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html

- Asynchronous replication
- Compliance and DR use cases
- Not strongly consistent across regions

---

# 6️⃣ Serverless Resilience

---

## AWS Lambda

📘 Docs:  
https://docs.aws.amazon.com/lambda/latest/dg/welcome.html

- Automatically multi-AZ
- Use reserved concurrency to prevent exhaustion
- Configure DLQ or destinations
- Implement idempotency

---

## Step Functions

📘 Docs:  
https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html

- Built-in retry policies
- Catch and fallback states
- Standard vs Express trade-offs

Centralized workflow logic improves fault tolerance.

---

# 7️⃣ Queue-Based Resilience

📘 SQS Docs:  
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html

SQS buffers protect downstream services.

Pattern:

API → Lambda → SQS → Worker Lambda

Benefits:

- Smooths traffic spikes
- Prevents database overload
- Enables retry and DLQ handling
- Improves failure isolation

---

# 8️⃣ Circuit Breaker Pattern

Used when downstream service instability may cascade failures.

Approaches:

- Step Functions retries with exponential backoff
- Application-level timeout and fallback
- SQS buffering to decouple unstable dependencies
- Graceful fallback response

Prevents cascading system failure.

---

# 9️⃣ Graceful Degradation

When part of the system fails:

- Serve cached content via CloudFront
- Switch to read-only mode
- Disable non-critical features
- Use feature flags
- Serve static fallback page

Graceful degradation preserves partial availability.

---

# 🔟 Backup & Restore Strategy

📘 AWS Backup Docs:  
https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html

Backup is not high availability.

Examples:

- RDS automated backups
- EBS snapshots
- S3 versioning + lifecycle
- AWS Backup centralized policies

Backup improves durability but does not reduce failover time.

---

# 1️⃣1️⃣ Health Checks & Failover

| Layer | Tool |
|--------|------|
| DNS | Route 53 health checks |
| Compute | ALB target health checks |
| Multi-Region | Global Accelerator |
| Hybrid | BGP route failover |
| Application | Synthetic monitoring |

Failover should be automated rather than manual.

---

# 1️⃣2️⃣ Observability for Resilience

📘 CloudWatch Docs:  
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html

Components:

- CloudWatch alarms
- AWS Health events
- X-Ray tracing
- Structured logging
- Synthetic canaries

Resilience without observability cannot be validated.

---

# 1️⃣3️⃣ Cost vs Availability Tradeoff

| Strategy | Cost | RTO | RPO |
|----------|------|------|------|
| Backup & Restore | Low | High | High |
| Pilot Light | Medium | Medium | Medium |
| Warm Standby | High | Low | Low |
| Active-Active | Highest | Near Zero | Near Zero |

Architecture must align with business objectives rather than technical preference.

---

# Common Design Mistakes

- Confusing backup with high availability
- Deploying single-AZ databases in production
- Ignoring data replication scope
- Storing session state locally
- Relying on manual failover
- Overlooking DNS TTL impact
- Not testing disaster recovery procedures
- Failing to externalize configuration and secrets

---

# Resilience Achieved When

- Compute is stateless and horizontally scalable
- Data is replicated appropriately
- Networking avoids single points of failure
- Failover is automated
- Recovery is tested regularly
- Observability validates assumptions

Resilience is not a single service choice — it is a layered architectural property.