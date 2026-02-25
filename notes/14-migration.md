# AWS Migration Notes (SAP-C02 Level)

DMS • SCT • Snowball • Snowmobile • MGN • VM Import/Export • Hybrid Networking • Cutover Strategies • 7 R’s

These notes summarize migration strategies, database and application migration services, hybrid connectivity models, large-scale data transfer options, and cutover approaches.

The emphasis is on choosing the correct strategy based on downtime tolerance, data size, compatibility constraints, and network bandwidth.

---

# 1️⃣ Migration Strategies – The 7 R’s

📘 AWS Migration Strategy Docs:  
https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-selection/what-is-migration.html

Migration decisions begin with selecting the appropriate transformation strategy.

## Rehost (Lift & Shift)

- Move workloads as-is
- Uses EC2, VM Import/Export, or AWS MGN
- Minimal application changes
- Fastest migration approach
- Common for large, time-sensitive migrations

---

## Replatform (Lift & Reshape)

- Minor optimizations without full redesign
- Examples:
  - Move database from EC2 to RDS
  - Replace self-managed queue with SQS
  - Move app to Elastic Beanstalk
- Improves operational efficiency while limiting risk

---

## Repurchase

- Replace application with SaaS
- Example: CRM, HR systems
- Reduces infrastructure management

---

## Refactor (Re-architect)

- Significant architectural redesign
- Often involves microservices or serverless
- Highest complexity and long-term benefit

---

## Retire

- Decommission unused systems
- Reduces migration scope

---

## Retain

- Keep certain workloads on-prem temporarily
- Used when regulatory or technical constraints prevent migration

---

## Relocate

- Hypervisor-level migration
- Example: VMware → VMware Cloud on AWS
- No application-level changes

---

# 2️⃣ Database Migration

## AWS Database Migration Service (DMS)

📘 Docs:  
https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html

DMS is responsible for **data movement**, not schema transformation.

### Capabilities

- Homogeneous migrations (MySQL → MySQL)
- Heterogeneous migrations (Oracle → PostgreSQL)
- Full load
- Change Data Capture (CDC)
- Full load + CDC for minimal downtime

### Architecture Considerations

- Requires replication instance inside a VPC
- Needs network connectivity to source and target
- Instance size affects throughput
- Supports ongoing replication

DMS does not:

- Convert schema
- Convert stored procedures completely
- Automatically fix incompatible data types

---

## AWS Schema Conversion Tool (SCT)

📘 Docs:  
https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/Welcome.html

SCT converts database schema during heterogeneous migrations.

### Capabilities

- Schema conversion
- Stored procedure translation (partial support)
- Assessment reports
- Compatibility analysis

SCT is typically paired with DMS for heterogeneous migrations.

---

## DMS vs SCT Comparison

| Dimension | DMS | SCT |
|------------|------|------|
| Moves Data | Yes | No |
| Converts Schema | No | Yes |
| Supports CDC | Yes | No |
| Used for Homogeneous Migration | Yes | Optional |
| Used for Heterogeneous Migration | Yes | Required |

---

# 3️⃣ Database Cutover Patterns

## Full Load Only

- Requires downtime
- Suitable for non-critical systems
- Simpler migration approach

---

## Full Load + CDC

- Minimal downtime
- Source continues accepting writes
- Cutover when replication lag approaches zero

This is the most common pattern for production migrations.

---

## Blue/Green Pattern

- Deploy parallel environment
- Synchronize data
- Switch DNS or routing after validation
- Enables rollback if necessary

Cutover strategy depends on RTO/RPO tolerance.

---

# 4️⃣ Large-Scale Data Transfer (Snow Family)

📘 Snowball Docs:  
https://docs.aws.amazon.com/snowball/latest/ug/whatissnowball.html  
📘 Snowmobile Docs:  
https://docs.aws.amazon.com/snowball/latest/ug/whatissnowmobile.html

## Snowball Edge

- 10–80 TB usable capacity
- Offline migration
- Automatic encryption
- Edge compute support
- Suitable when bandwidth is constrained

---

## Snowmobile

- Up to 100 PB per device
- Designed for data center migrations
- Physical secure transport

---

## Data Transfer Decision Matrix

| Data Volume | Recommended Approach |
|-------------|----------------------|
| < 1 TB | Internet / VPN / Direct Connect |
| 1–50 TB | Snowball Edge |
| 50+ PB | Snowmobile |

Bandwidth, security, and timeline influence selection.

---

# 5️⃣ Application & VM Migration

## AWS Application Migration Service (MGN)

📘 Docs:  
https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html

- Continuous block-level replication
- Near-zero downtime
- Test cutovers supported
- Converts servers into EC2 instances
- Supports large-scale lift-and-shift

---

## VM Import/Export

📘 Docs:  
https://docs.aws.amazon.com/vm-import/latest/userguide/what-is-vmimport.html

- Converts VM images into AMIs
- No continuous replication
- Suitable for one-time migration

---

## MGN vs VM Import/Export

| Feature | MGN | VM Import/Export |
|----------|------|-----------------|
| Continuous Replication | Yes | No |
| Cutover Testing | Yes | Limited |
| Downtime | Minimal | Higher |
| Best For | Large-scale rehost | Single VM import |

---

# 6️⃣ Hybrid & Network Migration

Migration often requires secure hybrid connectivity.

📘 Direct Connect Docs:  
https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html  
📘 VPN Docs:  
https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html

Connectivity options:

- Site-to-Site VPN
- Direct Connect
- Transit Gateway
- Hybrid DNS configuration

Considerations:

- Latency
- Throughput
- Security
- Data transfer cost

Direct Connect is often preferred for sustained high-throughput database replication.

---

# 7️⃣ File System Migration

## AWS DataSync

📘 Docs:  
https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html

Used for:

- NFS → EFS
- SMB → FSx for Windows
- On-prem → S3
- AWS → AWS file transfers

Capabilities:

- Parallel transfer
- Built-in encryption
- Scheduling support
- Incremental sync

DataSync is optimized for file-based migration, not database migration.

---

## File Migration Comparison

| Scenario | Recommended Service |
|----------|---------------------|
| NFS to EFS | DataSync |
| SMB to FSx | DataSync |
| Hybrid file gateway | Storage Gateway |
| Database migration | DMS |

---

# 8️⃣ Migration Decision Matrix

| Scenario | Recommended Approach |
|-----------|----------------------|
| Oracle → PostgreSQL | SCT + DMS |
| 5 TB DB minimal downtime | DMS Full Load + CDC |
| 100 TB data transfer | Snowball |
| Massive data center exit | Snowmobile |
| 50 VMs lift-and-shift | MGN |
| File share migration | DataSync |
| Hybrid database replication | DMS + Direct Connect |

Decision clarity depends on:

- Downtime tolerance
- Data size
- Schema compatibility
- Bandwidth constraints
- Application dependencies

---

# 9️⃣ Common Pitfalls in Migration Projects

- Assuming DMS converts schema automatically.
- Ignoring stored procedure incompatibilities in heterogeneous migrations.
- Underestimating bandwidth limitations.
- Choosing internet transfer for large datasets without evaluating Snow devices.
- Forgetting replication instances require correct VPC routing and security groups.
- Using DataSync for database migration.
- Assuming all database engines are fully supported as DMS targets.
- Skipping cutover rehearsal before production migration.
- Ignoring rollback planning.

Migration planning improves when evaluating:

- Downtime tolerance (RTO/RPO)
- Compatibility gaps
- Data size vs bandwidth
- Cutover rollback strategy
- Operational complexity post-migration

Understanding these dimensions ensures smoother transitions and reduced migration risk.