# AWS Database Notes (SAP-C02 Level)

Aurora • RDS • DynamoDB • ElastiCache

These notes capture architectural patterns, failover behavior, replication strategies, scaling models, durability considerations, and workload alignment trade-offs across AWS database services.

The focus is on understanding durability, consistency, and operational design rather than memorizing features.

---

# Aurora

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html  

Aurora is a cloud-native relational engine built on a distributed storage system.

---

## Aurora Storage Architecture

- Six copies of data across three Availability Zones
- Quorum-based writes
- Self-healing storage
- Storage auto-scaling up to 128 TB
- No replication lag between writer and replicas (shared storage layer)

This architecture improves durability and reduces failover complexity.

---

## Aurora Replicas

- Up to 15 replicas per cluster
- Share same storage layer
- No storage replication lag
- Failover priority tiers
- Reader promotion to writer during failover

Failover typically faster than RDS.

---

## Aurora Global Database

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html  

Characteristics:

- Cross-region replication via dedicated infrastructure
- Typically <1 second replication lag
- Writes only in primary region
- Secondary regions read-only until promoted
- Fast regional failover (~30 seconds)

Used when low RPO across regions is required for relational workloads.

---

## Aurora Serverless v2

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html  

- Fine-grained capacity scaling
- No cold starts (unlike v1)
- Scales based on ACUs
- Compatible with provisioned clusters

Useful for unpredictable or burst-heavy workloads.

---

## Backtrack

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html  

- Rewinds cluster to previous time
- No full restore required
- Useful for logical corruption
- Not supported for Aurora Global Database

---

# Amazon RDS

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html  

RDS provides managed relational databases without Aurora’s distributed storage layer.

---

## Multi-AZ Deployment

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html  

- Synchronous standby replica
- Automatic failover
- Same region only
- No read scaling benefit (standby not readable)

Multi-AZ = High Availability, not Disaster Recovery.

---

## Read Replicas

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html  

- Asynchronous replication
- Read scaling
- Cross-region replication supported
- Manual promotion required

Used for:

- Disaster recovery
- Regional read scaling

---

## RDS Storage Auto Scaling

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIOPS.Autoscaling.html  

Automatically increases storage when threshold exceeded.

Prevents outage due to storage exhaustion.

---

## RDS Proxy

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html  

- Connection pooling
- Handles failover more gracefully
- Reduces connection exhaustion
- Useful for Lambda/ECS/EKS workloads

Common pattern in serverless architectures.

---

## Performance Insights

📘 Documentation  
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html  

Helps analyze:

- Query latency
- CPU bottlenecks
- Wait events
- Load patterns

---

# DynamoDB

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html  

Fully managed NoSQL database optimized for scale and predictable performance.

---

## Capacity Modes

| Mode | Use Case |
|------|----------|
| Provisioned | Predictable workload |
| On-Demand | Unpredictable traffic |

On-demand automatically scales but costs more at high sustained throughput.

---

## Partition Model

Partition count influenced by:

- WCU
- RCU
- Item size
- Partition key distribution

Hot partitions cause throttling and uneven scaling.

---

## Consistency Model

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html  

- Eventually consistent reads (default)
- Strongly consistent reads (single region only)
- Global Tables use eventual consistency across regions

---

## DynamoDB Transactions

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transactions.html  

- ACID transactions across multiple items
- Limited to 25 items per transaction
- Increased cost and latency

---

## Adaptive Capacity

Automatically rebalances throughput for hot partitions.

Reduces manual partition tuning.

---

## DynamoDB Streams

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html  

- Change log of table modifications
- Integrates with Lambda
- Used for audit, replication, materialized views

---

## Global Tables

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html  

- Active-active replication
- Multi-region writes
- Last-writer-wins conflict resolution
- Near-zero RPO/RTO

Used for global write locality requirements.

---

## DAX (DynamoDB Accelerator)

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html  

- In-memory cache
- Microsecond read latency
- Read-through cache
- Does not reduce write capacity usage

Used for read-heavy workloads.

---

## TTL (Time to Live)

- Automatic item expiration
- Asynchronous deletion
- Useful for sessions or temporary data

---

# ElastiCache

📘 Documentation  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html  

Managed in-memory caching.

---

## Redis

📘 Documentation  
https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.html  

Supports:

- Replication
- Automatic failover
- Cluster mode (sharding)
- Persistence (optional)
- Global Datastore (cross-region)

Used for:

- Session management
- Leaderboards
- Distributed locking
- High-speed caching

---

## Redis Cluster Mode vs Non-Cluster Mode

| Mode | Scaling | Use Case |
|------|--------|----------|
| Non-cluster | Vertical | Simpler workloads |
| Cluster mode | Horizontal sharding | Large-scale cache |

---

## Memcached

- No replication
- No persistence
- Multi-node horizontal scaling
- Simpler caching use cases

---

# Backup & Restore Patterns

## RDS / Aurora

- Automated backups
- Point-in-time recovery (PITR)
- Manual snapshots
- Cross-region snapshot copy

---

## DynamoDB

📘 Documentation  
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html  

- Continuous backups (PITR)
- On-demand backups
- No restore into same table (must create new table)

---

# Architectural Comparisons

## RDS vs Aurora

| Feature | RDS | Aurora |
|----------|------|--------|
| Storage | EBS-backed | Distributed storage |
| Read Replicas | Yes | Yes (up to 15) |
| Cross-Region DR | Read replica | Aurora Global |
| Failover Speed | Minutes | Faster |
| Performance | Engine dependent | Higher throughput |

---

## Aurora Global vs RDS Cross-Region Replica

| Feature | Aurora Global | RDS Cross-Region |
|----------|----------------|-----------------|
| Replication Lag | <1 sec typical | Seconds–minutes |
| Failover | Faster | Manual |
| RPO | Near zero | Higher |
| RTO | Low | Higher |

---

## Relational vs DynamoDB

| Feature | RDS / Aurora | DynamoDB |
|----------|---------------|------------|
| Data Model | Relational | Key-value |
| Joins | Yes | No |
| Schema | Structured | Flexible |
| Scaling | Vertical + replicas | Horizontal automatic |
| Transactions | Strong ACID | Limited ACID |
| Global Writes | Complex | Native |

---

# Common Pitfalls in SAP-C02

- Assuming Multi-AZ provides multi-region DR
- Confusing Aurora Global with read replicas
- Ignoring DynamoDB partition key design
- Expecting DAX to reduce write capacity
- Using Redis as primary durable database
- Forgetting eventual consistency in Global Tables
- Not enabling PITR for critical workloads
- Underestimating connection scaling without RDS Proxy

---

# Decision Framing Model

Database selection becomes clearer when evaluated across:

- Consistency requirements
- Failover scope (AZ vs region vs global)
- Read/write ratio
- Access pattern predictability
- Multi-region write requirement
- Operational complexity tolerance
- Latency expectations

Architectural clarity comes from aligning workload behavior with the consistency and scaling model of the database engine.