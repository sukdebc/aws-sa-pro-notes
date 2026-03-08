# AWS Database Notes (SAP-C02 Level)

Aurora • RDS • DynamoDB • ElastiCache

These notes capture architectural patterns, failover behavior, replication strategies, scaling models, durability considerations, and workload alignment trade-offs across AWS database services.

The emphasis is on understanding consistency, availability scope, scaling behavior, and operational impact rather than memorizing features.

---

# Aurora

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html  

Aurora is a cloud-native relational engine built on distributed storage.

- Six copies of data across three AZs  
- Quorum-based writes  
- Self-healing storage  
- Storage auto-scaling  
- Shared storage layer across writer and replicas  

This architecture improves durability and simplifies failover.

---

## Aurora Replicas

- Up to 15 replicas  
- Share the same storage layer  
- Minimal replication lag  
- Reader promotion during failover  

Failover is generally faster than traditional RDS.

> **EXAM TIP**  
> High read scaling + fast failover → Aurora.

---

## Aurora Global Database

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html  

- Dedicated cross-region replication  
- Typically <1 second lag  
- Single writer region  
- Secondary regions read-only until promoted  
- Fast regional failover  

Common when low RPO is required across regions for relational workloads.

> **EXAM TIP**  
> Low RPO across regions (relational) → Aurora Global.

---

## Aurora Serverless v2

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html  

- Fine-grained capacity scaling  
- No cold starts  
- Compatible with provisioned clusters  

Often aligned with unpredictable or burst-heavy workloads.

---

## Backtrack

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html  

- Rewinds cluster to a previous point in time  
- No full restore required  
- Useful for logical errors  

Not supported for Aurora Global Database.

---

# Amazon RDS

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html  

Managed relational databases without Aurora’s distributed storage architecture.

---

## Multi-AZ Deployment

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html  

- Synchronous standby replica  
- Automatic failover  
- Same region only  
- Standby not readable  

Multi-AZ provides high availability within a region.

> **EXAM TIP**  
> Multi-AZ = High Availability (same region).  
> Not cross-region disaster recovery.

---

## Read Replicas

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html  

- Asynchronous replication  
- Cross-region supported  
- Manual promotion required  

Used for read scaling and DR.

---

## RDS Proxy

**Documentation:**  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html  

- Connection pooling  
- Handles failover more gracefully  
- Reduces connection exhaustion  

Common in Lambda, ECS, and EKS architectures.

> **EXAM TIP**  
> High concurrent connections (serverless) → RDS Proxy.

---

# DynamoDB

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html  

Fully managed NoSQL database optimized for horizontal scale.

---

## Capacity Modes

| Mode | Typical Use |
|------|-------------|
| Provisioned | Predictable workload |
| On-Demand | Unpredictable traffic |

On-demand simplifies scaling but may cost more under sustained load.

---

## Partition Model

Partition distribution depends on:

- RCU/WCU  
- Item size  
- Partition key design  

Hot partitions can cause throttling.

> **EXAM TIP**  
> Scaling issue in DynamoDB?  
> Often partition key design.

---

## Consistency

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html  

- Eventually consistent reads (default)  
- Strongly consistent reads (single region)  
- Global Tables use eventual consistency across regions  

---

## Global Tables

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html  

- Active-active multi-region writes  
- Last-writer-wins conflict resolution  
- Near-zero RPO/RTO under normal conditions  

Conflict resolution must be considered in design.

> **EXAM TIP**  
> Multi-region writes required → DynamoDB Global Tables.

---

## DynamoDB Streams

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html  

- Change log of table modifications  
- Integrates with Lambda  
- Used for replication or event-driven patterns  

---

## DAX

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html  

- In-memory read cache  
- Microsecond read latency  
- Does not reduce write capacity usage  

> **EXAM TIP**  
> DAX improves read latency only.

---

# ElastiCache

**Documentation:**  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html  

Managed in-memory caching layer.

---

## Redis

**Documentation:**  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.html  

- Replication + failover  
- Optional persistence  
- Cluster mode (sharding)  
- Global Datastore (cross-region)  

Common for caching, sessions, distributed locking.

> **EXAM TIP**  
> Redis is typically cache-first, not primary durable database.

---

## Memcached

- No replication  
- No persistence  
- Horizontal scaling  
- Simpler caching scenarios  

---

# Backup & Restore Patterns

## RDS / Aurora

- Automated backups  
- Point-in-time recovery  
- Manual snapshots  
- Cross-region snapshot copy  

---

## DynamoDB

**Documentation:**  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html  

- Continuous backups (PITR)  
- On-demand backups  
- Restore creates new table  

---

# Architectural Comparisons

## RDS vs Aurora

| Feature | RDS | Aurora |
|----------|------|--------|
| Storage | EBS-backed | Distributed |
| Read Replicas | Yes | Yes (up to 15) |
| Cross-Region DR | Read replica | Aurora Global |
| Failover Speed | Minutes | Faster |
| Throughput | Engine dependent | Higher |

---

## Aurora Global vs RDS Cross-Region

| Feature | Aurora Global | RDS Cross-Region |
|----------|----------------|-----------------|
| Replication Lag | <1 sec typical | Seconds–minutes |
| Failover | Faster | Manual |
| RPO | Near zero | Higher |
| RTO | Lower | Higher |

---

## Relational vs DynamoDB

| Feature | RDS / Aurora | DynamoDB |
|----------|---------------|------------|
| Data Model | Relational | Key-value |
| Joins | Supported | Not supported |
| Scaling | Vertical + replicas | Horizontal automatic |
| Transactions | Full ACID | Limited ACID |
| Global Writes | Complex | Native |

---

# Common Pitfalls

- Assuming Multi-AZ provides regional DR  
- Confusing Aurora Global with read replicas  
- Ignoring DynamoDB partition key design  
- Expecting DAX to reduce write capacity  
- Using Redis as primary durable store  
- Forgetting eventual consistency in Global Tables  
- Not enabling PITR for critical workloads  
- Ignoring connection scaling limits without RDS Proxy  

---

# Design Considerations

Database choices typically align with:

- Consistency requirements  
- Failover scope (AZ vs Region vs Global)  
- Read/write ratio  
- Access pattern predictability  
- Multi-region write needs  
- Latency expectations  
- Operational complexity tolerance  

Clarity usually comes from aligning workload behavior with the database engine’s consistency and scaling model.