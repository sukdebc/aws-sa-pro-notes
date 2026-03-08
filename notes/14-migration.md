# AWS Migration Notes (SAP-C02 Level)

DMS • SCT • Snowball • Snowmobile • MGN • VM Import/Export • Hybrid Networking • Cutover Strategies • 7 R’s

These notes summarize migration strategies, database and application migration services, hybrid connectivity models, large-scale data transfer options, and cutover approaches.

The emphasis is on choosing the correct strategy based on downtime tolerance, data size, compatibility constraints, and network bandwidth.

---

# Migration Strategies – The 7 R’s

Documentation  
https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-selection/what-is-migration.html

Migration decisions usually begin by selecting the right transformation strategy.

---

## Rehost

Rehost is lift-and-shift migration.

Characteristics:

- Move workloads largely as-is
- Minimal application changes
- Fastest migration approach
- Often uses EC2, VM Import/Export, or AWS MGN

Common when migration speed matters more than modernization.

---

## Replatform

Replatform introduces limited optimization without redesign.

Examples include:

- Moving a database from EC2 to RDS
- Replacing a self-managed queue with SQS
- Moving an application to Elastic Beanstalk

This improves operations while keeping migration risk moderate.

---

## Repurchase

Repurchase replaces the existing application with SaaS.

Typical examples:

- CRM systems
- HR platforms
- Collaboration tools

This reduces infrastructure management but may require business process change.

---

## Refactor

Refactor involves significant architectural redesign.

Typical patterns include:

- Microservices
- Serverless
- Event-driven decomposition

This has the highest complexity but often gives the greatest long-term benefit.

---

## Retire

Retire means decommissioning systems that are no longer needed.

Benefits include:

- Reduced migration scope
- Lower cost
- Less operational overhead

---

## Retain

Retain means keeping some workloads on premises temporarily.

This is used when:

- Regulatory constraints exist
- Technical dependencies remain
- Migration is not yet practical

---

## Relocate

Relocate is hypervisor-level migration with minimal application change.

Typical example:

- VMware workloads moved to VMware Cloud on AWS

This preserves the existing virtualization model.

> **EXAM TIP**  
> Large-scale lift-and-shift with minimal change → Rehost.  
> Hypervisor-level VMware move → Relocate.

---

# Database Migration

## AWS Database Migration Service

Documentation  
https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html

DMS is responsible for moving data rather than transforming schema.

Capabilities include:

- Homogeneous migrations
- Heterogeneous migrations
- Full load
- Change Data Capture
- Full load plus CDC

Architecture considerations:

- Requires a replication instance in a VPC
- Needs connectivity to source and target
- Replication instance size affects throughput
- Supports ongoing replication for minimal downtime cutovers

DMS does not:

- Convert schema
- Fully convert stored procedures
- Automatically resolve incompatible data types

> **EXAM TIP**  
> DMS moves data.  
> It does not perform schema conversion.

---

## AWS Schema Conversion Tool

Documentation  
https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/Welcome.html

SCT is used for schema conversion during heterogeneous database migrations.

Capabilities include:

- Schema conversion
- Partial stored procedure translation
- Compatibility assessment reports
- Migration complexity analysis

SCT is commonly paired with DMS when source and target engines differ.

---

## DMS vs SCT

| Dimension | DMS | SCT |
|----------|-----|-----|
| Moves Data | Yes | No |
| Converts Schema | No | Yes |
| Supports CDC | Yes | No |
| Homogeneous Migration | Yes | Optional |
| Heterogeneous Migration | Yes | Usually required |

---

# Database Cutover Patterns

## Full Load Only

Characteristics:

- Requires downtime
- Simpler migration process
- Suitable for non-critical workloads

---

## Full Load plus CDC

Characteristics:

- Supports minimal downtime migration
- Source system continues accepting writes
- Cutover occurs when replication lag is near zero

This is the most common production migration pattern.

---

## Blue/Green Cutover

Characteristics:

- Parallel environments
- Data synchronization before switch
- Validation before traffic shift
- Easier rollback path

Cutover design should always align with RTO and RPO expectations.

> **EXAM TIP**  
> Minimal downtime database migration → Full load + CDC.

---

# Large-Scale Data Transfer

Documentation

Snowball  
https://docs.aws.amazon.com/snowball/latest/ug/whatissnowball.html  

Snowmobile  
https://docs.aws.amazon.com/snowball/latest/ug/whatissnowmobile.html

---

## Snowball Edge

Snowball Edge is used for offline data transfer where network bandwidth is limited.

Characteristics:

- Tens of terabytes of usable capacity per device
- Automatic encryption
- Chain-of-custody controls
- Edge compute capability on some device types

Used when transferring data over the network is too slow or impractical.

---

## Snowmobile

Snowmobile is designed for extremely large migrations.

Characteristics:

- Up to 100 PB per device
- Physical secure transport
- Suitable for data center-scale migrations

Used for very large-scale data transfers that are impractical over standard networks.

---

## Data Transfer Decision Matrix

| Data Volume | Recommended Approach |
|-------------|----------------------|
| Small to moderate | Internet, VPN, or Direct Connect |
| Tens of TB | Snowball Edge |
| Massive PB-scale migration | Snowmobile |

Bandwidth, security, and migration timeline usually drive the choice.

> **EXAM TIP**  
> Large dataset with constrained bandwidth → Snow family, not internet transfer.

---

# Application and VM Migration

## AWS Application Migration Service

Documentation  
https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html

AWS MGN supports lift-and-shift server migration.

Core characteristics:

- Continuous block-level replication
- Minimal downtime cutover
- Test cutovers supported
- Converts source servers into EC2 instances
- Suitable for large-scale rehost migrations

MGN is commonly chosen for server migration with minimal application change.

---

## VM Import/Export

Documentation  
https://docs.aws.amazon.com/vm-import/latest/userguide/what-is-vmimport.html

VM Import/Export converts VM images into AMIs.

Characteristics:

- One-time image-based migration
- No ongoing replication
- More suitable for smaller or one-off migrations

It is not the best option for large-scale near-zero downtime migration.

---

## MGN vs VM Import/Export

| Feature | MGN | VM Import/Export |
|--------|-----|------------------|
| Continuous Replication | Yes | No |
| Cutover Testing | Yes | Limited |
| Downtime | Minimal | Higher |
| Best Fit | Large-scale rehost | One-time VM image migration |

---

# Hybrid and Network Migration

Documentation

Direct Connect  
https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html  

Site-to-Site VPN  
https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html

Migration often requires temporary or long-term hybrid connectivity.

Common connectivity options include:

- Site-to-Site VPN
- Direct Connect
- Transit Gateway
- Hybrid DNS integration

Design considerations include:

- Latency
- Throughput
- Security
- Transfer cost
- Reliability

Direct Connect is often preferred for sustained high-throughput replication workloads such as database migration.

---

# File System Migration

## AWS DataSync

Documentation  
https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html

DataSync is optimized for file and object transfer.

Common use cases include:

- NFS to EFS
- SMB to FSx for Windows File Server
- On-premises to S3
- AWS to AWS file transfer

Capabilities include:

- Parallelized transfer
- Encryption in transit
- Scheduling
- Incremental sync

DataSync is designed for file migration, not relational database migration.

> **EXAM TIP**  
> File share migration → DataSync.  
> Database migration → DMS.

---

## File Migration Comparison

| Scenario | Recommended Service |
|----------|---------------------|
| NFS to EFS | DataSync |
| SMB to FSx | DataSync |
| Hybrid cached file access | Storage Gateway |
| Database migration | DMS |

---

# Migration Decision Matrix

| Scenario | Recommended Approach |
|----------|----------------------|
| Oracle to PostgreSQL | SCT + DMS |
| Multi-TB database with minimal downtime | DMS full load + CDC |
| Large offline data transfer | Snowball Edge |
| Massive data center exit | Snowmobile |
| Large VM lift-and-shift migration | MGN |
| File share migration | DataSync |
| Hybrid replication with high throughput | DMS + Direct Connect |

Decision clarity usually depends on:

- Downtime tolerance
- Data volume
- Schema compatibility
- Available bandwidth
- Dependency complexity

---

# Common Pitfalls

- Assuming DMS converts schema automatically  
- Ignoring stored procedure incompatibilities in heterogeneous migration  
- Underestimating bandwidth limits  
- Choosing internet transfer for very large datasets without evaluating Snow devices  
- Forgetting DMS replication instances need correct VPC routing and security groups  
- Using DataSync for database migration  
- Assuming every database target is equally supported by DMS  
- Skipping cutover rehearsal before production migration  
- Ignoring rollback planning  

---

# Design Considerations

Migration decisions typically become clearer when evaluated across:

- Downtime tolerance and recovery objectives
- Schema and application compatibility
- Data size versus available bandwidth
- Need for replication versus one-time copy
- Rollback and cutover strategy
- Post-migration operational complexity

The right migration pattern usually comes from aligning business downtime tolerance with data movement method, compatibility constraints, and target-state architecture.
