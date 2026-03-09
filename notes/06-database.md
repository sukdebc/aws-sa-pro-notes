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
- Connection reuse 
- Failover transparency  
- Reduces connection exhaustion  
- Protection against connection storms (especially Lambda)

RDS Proxy acts as an intermediary connection pool between applications and the database.

During Multi-AZ failover, RDS Proxy automatically reconnects to the newly promoted primary instance.  
This reduces application-level connection failures and avoids reconnect storms during failover events.
RDS Proxy is particularly useful for serverless architectures where large numbers of short-lived connections (for example from Lambda) can overwhelm database connection limits.

> **EXAM TIP**  
> Lambda or highly concurrent workloads causing database connection exhaustion → RDS Proxy.

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

Hot partitions can cause throttling when a single partition key receives disproportionate traffic.<br>
DynamoDB can temporarily absorb spikes using burst capacity, which accumulates unused throughput for up to 300 seconds.<br>
Sustained uneven traffic distribution may still lead to throttling even when overall table capacity appears sufficient.

> **EXAM TIP**  
> Scaling issue in DynamoDB?  
> Often partition key design.
> Short traffic spikes may succeed due to DynamoDB burst capacity.  
> Sustained hot partitions still cause throttling even if table capacity appears adequate.  
> On-Demand mode simplifies scaling but trades cost for operational simplicity.

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

- Continuous backups - point-in-time recovery (PITR)  
- On-demand backups  
- Restore creates new table  

---

# Architectural Comparisons

## RDS vs Aurora

| Dimension | RDS | Aurora |
|-----------|-----|--------|
| Storage | EBS-backed | Distributed storage across AZs |
| Read Replicas | Yes | Up to 15 |
| Cross-region DR | Read replica | Aurora Global |
| Failover Speed | Minutes | Typically faster |
| Performance | Engine dependent | Higher throughput |
| Operational Complexity | Lower | Slightly higher |
| Cost | Lower baseline | Higher baseline |

When to choose RDS over Aurora:

- Cost-sensitive workloads
- Smaller applications
- Standard relational workloads
- When advanced Aurora features are unnecessary

When to choose Aurora over RDS:

- High read scaling required
- Faster failover required
- Multi-region DR with low RPO
- High throughput OLTP workloads

---

## Aurora Global vs RDS Cross-Region

| Dimension | Aurora Global Database | RDS Cross-Region Read Replica |
|-----------|------------------------|-------------------------------|
| Replication Method | Storage-level replication | Engine-level replication |
| Replication Lag | Typically < 1 second | Seconds to minutes |
| Failover Speed | Faster regional failover | Manual promotion required |
| RPO | Near zero | Higher |
| RTO | Lower | Higher |
| Write Capability | Single writer region | Read-only replica until promoted |
| Complexity | Higher architecture complexity | Simpler setup |
| Supported Engines | Aurora only | Multiple engines |

When to choose Aurora Global:

- Very low RPO (<1 sec) required
- Fast regional disaster recovery
- Global read scaling required 
- Latency-sensitive global applications 
- Mission-critical relational workloads

When to choose RDS Cross-Region Read Replicas:

- Moderate DR requirements 
- Replication lag of seconds is acceptable 
- Simpler architecture preferred 
- Cost-sensitive environments 
- Non-Aurora database engines

In many architectures, Aurora Global is selected for resilience, while RDS replicas are used for cost-efficient DR.
---

## Aurora vs DynamoDB

| Dimension | Aurora | DynamoDB |
|-----------|--------|----------|
| Data Model | Relational | Key-value / document |
| Joins | Supported | Not supported |
| Scaling | Vertical + replicas | Horizontal automatic |
| Transactions | Full ACID | Limited ACID |
| Global Writes | Complex | Native via Global Tables |
| Latency | Milliseconds | Single-digit milliseconds |

When to choose DynamoDB:

- Massive scale workloads
- Unpredictable traffic
- Serverless architectures
- Key-value access patterns

When to choose Aurora:

- Complex relational queries
- Transaction-heavy systems
- Applications requiring joins

---

# Common Pitfalls

- Assuming Multi-AZ provides regional DR  
- Confusing Aurora Global with read replicas  
- Ignoring DynamoDB partition key design  
- Expecting DAX to reduce write capacity  
- Using Redis as primary durable store  
- Forgetting eventual consistency in Global Tables  
- Not enabling point-in-time recovery (PITR) for critical workloads  
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