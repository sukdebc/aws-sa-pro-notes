# AWS Architecture Decision Trees

Compact decision trees for common SAP-C02 architecture decisions.

Use as quick heuristics only. Final choices should always align with workload non-functional requirements (RTO/RPO, compliance, cost, operational capability).

---

## Color scheme

| Type | Color | Meaning |
|------|--------|---------|
| Start / Entry | Light Blue | Beginning of flow |
| Decision | Light Orange | Branching logic |
| Service / Action | Light Green | AWS service recommendation |
| Neutral | Light Gray | Optional routing nodes |

---

## 1. Networking — Hybrid & VPC connectivity

```mermaid
flowchart TD

  %% Nodes
  A[Need network connectivity] --> B{On premises involved}

  B -->|Yes| C{Traffic type}
  C -->|High throughput predictable| D[Direct Connect]
  C -->|Quick setup or low bandwidth| E[Site to Site VPN]

  D --> F{Multiple VPCs}
  F -->|Yes| G[Direct Connect Gateway plus Transit Gateway]
  F -->|No| H[Direct Connect plus Virtual Private Gateway]

  E --> I{Multiple VPCs}
  I -->|Yes| J[VPN plus Transit Gateway]
  I -->|No| K[VPN plus Virtual Private Gateway]

  B -->|No AWS only| L{Need VPC to VPC connectivity}

  L -->|Yes| M{Number of VPCs}
  M -->|Two VPCs simple| N[VPC Peering]
  M -->|Many VPCs hub spoke| O[Transit Gateway]

  O --> P{Cross region needed}
  P -->|Yes| Q[TGW Peering]
  P -->|No| R[Regional TGW routing]

  L -->|No service exposure only| S{Expose service privately}
  S -->|Yes| T[PrivateLink via NLB plus Interface Endpoint]
  S -->|No| U{Access AWS services privately}

  U -->|Yes| V{Service type}
  V -->|S3 or DynamoDB| W[Gateway Endpoint]
  V -->|Other AWS services| X[Interface Endpoint]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px

  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px
  classDef neutral fill:#e2e3e5,stroke:#6c757d,stroke-width:2px

  class B,C,F,I,L,M,P,S,U,V decision
  class D,E,G,H,J,K,N,O,Q,R,T,W,X service

```

## Decision checklist
- **Direct Connect** → predictable high-throughput, low-latency hybrid connectivity.
- **Site-to-Site VPN** → quick setup or temporary hybrid connectivity.
- **Transit Gateway** → many-VPC connectivity or hub-and-spoke network designs.
- **PrivateLink** → private service exposure across accounts or overlapping CIDR ranges.
- **Gateway / Interface Endpoints** → private access to AWS services without internet routing.

---

## 2. Database selection

```mermaid
flowchart TD

  A[Start: Need a database] --> B{Primary data model}

  B -->|Relational or ACID transactions| C{Performance or scale requirements}
  C -->|High performance or high read scaling| D[Aurora]
  C -->|Standard relational workload| E[RDS]

  D --> F{Multi-region DR required}
  F -->|Yes| G[Aurora Global Database]
  F -->|No| H[Aurora cluster]

  E --> I{Many short lived connections}
  I -->|Yes| J[RDS plus RDS Proxy]
  I -->|No| K[Standard RDS]

  B -->|Key value or flexible schema| L{Traffic scale}
  L -->|Massive or unpredictable scale| M[DynamoDB]
  L -->|MongoDB API compatibility| N[Amazon DocumentDB]

  M --> O{Multi-region writes required}
  O -->|Yes| P[DynamoDB Global Tables]
  O -->|No| Q[DynamoDB single region]

  B -->|Microsecond read latency required| R[ElastiCache Redis]
  B -->|Graph relationships| S[Amazon Neptune]
  B -->|Time series data| T[Amazon Timestream]
  B -->|Immutable ledger or audit trail| U[Amazon QLDB]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px
  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px

  class B,C,F,I,L,O decision
  class D,E,G,H,J,K,M,N,P,Q,R,S,T,U service

```

## Decision checklist
- **Aurora** → high-performance relational workloads, large read scaling.
- **RDS** → standard relational workloads.
- **RDS Proxy** → useful for workloads with many short-lived connections (for example Lambda).
- **DynamoDB** → massive scale or unpredictable traffic.
- **DynamoDB Global Tables** → multi-region active-active writes.
- **ElastiCache (Redis)** → microsecond read latency or caching layer.
- Specialized databases:
  - **Neptune** → graph workloads
  - **Timestream** → time-series data
  - **QLDB** → immutable ledger workloads

---

## 3. Disaster recovery (RTO/RPO)

```mermaid
flowchart TD

  A[Start: Define RTO and RPO requirements] --> B{RTO seconds or near-zero RPO}

  B -->|Yes| C{Primary data model}

  C -->|Relational| D[Aurora Global Database<br/>Single writer, multi-region hot standby]

  C -->|NoSQL| E{Need multi-region writes}

  E -->|Yes| F[DynamoDB Global Tables<br/>Multi-active multi-region writes<br/>Last-writer-wins conflict resolution]

  E -->|No| G[Single-region DynamoDB<br/>Use DR pattern based on recovery target]

  B -->|No| H{RTO minutes}

  H -->|Yes| I[Warm Standby<br/>Scaled-down environment always running]

  H -->|No| J{RTO tens of minutes}

  J -->|Yes| K[Pilot Light<br/>Core services running, scale on failover]

  J -->|No| L[Backup & Restore<br/>Snapshots and cross-region backups]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px
  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px

  class B,C,E,H,J decision
  class D,F,G,I,K,L service

```

## Decision checklist
- **Active-Active** → sub-second RTO and near-zero RPO.
- **Warm Standby** → minute-level RTO.
- **Pilot Light** → tens-of-minutes RTO.
- **Backup & Restore** → hours-level RTO.
- **Operational practice** → automate failover where possible and regularly test recovery procedures.

---

## 4. Storage selection

```mermaid
flowchart TD

  A[Start: What access pattern and protocol] -->|Object storage via REST API| B[S3]

  A -->|File system access| C{Workload type}
  C -->|Linux NFS workloads| D[EFS]
  C -->|Windows SMB workloads| E[FSx for Windows File Server]
  C -->|High performance compute or POSIX or HPC| F[FSx for Lustre]

  A -->|Block storage required| G[EBS volume]

  B --> H{Archive or long term retention needed}
  H -->|Immediate retrieval required| I[S3 Glacier Instant Retrieval]
  H -->|Minutes to hours retrieval acceptable| J[S3 Glacier Flexible Retrieval]
  H -->|Very long term archive lowest cost| K[S3 Glacier Deep Archive]

  G --> L{Need storage across AZs or instances}
  L -->|Single instance in one AZ| M[EBS standard usage]
  L -->|High availability required| N[Use managed service replication - example RDS Multi AZ or Aurora cross-AZ replication]

  N --> O{Need cross region durability}
  O -->|Yes| P[EBS snapshot copy or service replication]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px
  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px

  class C,H,L,O decision
  class B,D,E,F,G,I,J,K,M,N,P service

```

## Decision checklist
- **S3** → object storage. **EFS / FSx** → file systems. **EBS** → block storage.
- **Glacier storage classes** → selected based on retrieval latency and cost requirements.
- **EBS** → AZ-scoped; use replication or managed services for high availability.
- **Cross-region durability** → snapshot copy or service-level replication.
- **Lifecycle policies** → move data automatically to lower-cost storage classes.

---

## 5. Compute selection

```mermaid
flowchart TD

  A[Start: Workload type] --> B{Event driven or short lived tasks}
  B -->|Yes| C{Execution typically under 15 minutes}
  C -->|Yes| D[Lambda]
  C -->|No| E[Fargate or EC2 containers]

  D --> F{Low latency required}
  F -->|Yes| G[Lambda with Provisioned Concurrency]
  F -->|No| H[Standard Lambda]

  A -->|No| I{Workload packaged as containers}
  I -->|Yes| J{Need Kubernetes ecosystem or portability}
  J -->|Yes| K[EKS]
  J -->|No| L[ECS]

  K --> M{Need host control or GPU or DaemonSets}
  M -->|Yes| N[EKS with EC2 workers]
  M -->|No| O[EKS with Fargate]

  L --> P{Need host customization or GPU}
  P -->|Yes| Q[ECS on EC2]
  P -->|No| R[ECS on Fargate]

  I -->|No| S{Traditional application or OS control needed}
  S -->|Yes| T[EC2 Auto Scaling Groups]
  S -->|Simplified platform deployment| U[Elastic Beanstalk]

  T --> V{Large scale batch jobs}
  V -->|Yes| W[AWS Batch]
  V -->|No| X[Standard EC2 workloads]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px
  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px

  class B,C,F,I,J,M,P,S,V decision
  class D,E,G,H,K,L,N,O,Q,R,T,U,W,X service

```

## Decision checklist
- **Lambda** → sub-15-minute event-driven tasks.
- **Provisioned Concurrency** → low-latency Lambda workloads.
- **EKS** → Kubernetes ecosystem and portability.
- **ECS** → simpler container orchestration.
- **EC2-based containers (EKS/ECS on EC2)** → GPU workloads or host customization.
- **EC2 Auto Scaling Groups / Elastic Beanstalk** → traditional application deployments.
- **AWS Batch** → large-scale batch processing workloads.

---

## 6. Messaging / Event Architecture

```mermaid
flowchart TD

  A[Start: What messaging pattern is required] --> B{High throughput ordered event stream}
  B -->|Yes| C[Kinesis Data Streams or MSK]

  B -->|No| D{Do multiple independent consumers need the same event}
  D -->|Yes| E[SNS fan out to SQS queues per consumer]

  D -->|No| F{Need durable buffering or backpressure control}
  F -->|Yes| G[SQS queue]

  G --> H{Strict ordering required}
  H -->|Yes| I[SQS FIFO]
  H -->|No| J[SQS Standard]

  F -->|No| K{Need content based routing or event bus}
  K -->|Yes| L[EventBridge]

  K -->|No| M[Direct invocation - Lambda or Step Functions]

  %% Styles
  style A fill:#cce5ff,stroke:#004085,stroke-width:2px
  classDef decision fill:#ffe5cc,stroke:#cc7000,stroke-width:2px
  classDef service fill:#d4edda,stroke:#155724,stroke-width:2px

  class B,D,F,H,K decision
  class C,E,G,I,J,L,M service

```

## Decision checklist
- **Kinesis / MSK** → high-throughput, ordered event streams.
- **SNS → SQS fan-out** → multiple independent consumers need the same event.
- **SQS** → durable buffering and backpressure.
- **SQS FIFO** → strict ordering requirements.
- **EventBridge** → content-based routing and event bus architectures.
- **Direct Lambda / Step Functions invocation** → when buffering or queueing is not required.
- **Operational settings** → configure DLQs, visibility timeouts, and message retention appropriately.
