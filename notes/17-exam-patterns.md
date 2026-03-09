# Common Exam Scenario Patterns

These patterns summarize recurring architectural signals commonly seen in SAP-C02 scenarios.  
They are intended as quick heuristics and should always be validated against the full set of requirements in the question.

---

## High Availability

- Multi-AZ failover for relational database  
  → **RDS Multi-AZ** or **Aurora cluster**
- Near-zero RPO across regions  
  → **Aurora Global Database** or **DynamoDB Global Tables**
- Global traffic failover  
  → **Route 53** health checks with failover routing
- Application tier resilience  
  → **Load balancer** + **Auto Scaling** across multiple AZs

---

## Disaster Recovery

- RTO seconds, RPO near-zero  
  → **Aurora Global Database** (single-writer) or **DynamoDB Global Tables** (multi-active)
- RTO minutes  
  → **Warm Standby** (scaled-down environment always running)
- RTO tens of minutes  
  → **Pilot Light** (core services running, scale on failover)
- RTO hours acceptable  
  → **Backup and Restore** using snapshots or backups

---

## Compute Selection

- Short-lived event processing (<15 min)  
  → **Lambda**
- Containerized microservices  
  → **ECS** or **EKS**
- Kubernetes portability required  
  → **EKS**
- GPU or host customization required  
  → **EC2-based containers**
- Large batch workloads  
  → **AWS Batch**
- Traditional application servers  
  → **EC2 Auto Scaling Groups**

---

## Database Selection

- High-performance relational workloads  
  → **Aurora**
- Standard relational workloads  
  → **RDS**
- Massive or unpredictable scale  
  → **DynamoDB**
- Multi-region active-active database  
  → **DynamoDB Global Tables**
- Microsecond read latency / caching  
  → **ElastiCache (Redis)**
- Graph relationships  
  → **Neptune**

---

## Messaging & Event Architecture

- Multiple consumers must receive the same event  
  → **SNS** fan-out
- Durable buffering and backpressure control  
  → **SQS**
- Strict ordering required  
  → **SQS FIFO**
- Content-based routing or event bus pattern  
  → **EventBridge**
- High-throughput ordered streams  
  → **Kinesis Data Streams** or **MSK**

---

## Networking & Hybrid Connectivity

- Quick hybrid connectivity setup  
  → **Site-to-Site VPN**
- Predictable high-bandwidth hybrid connectivity  
  → **Direct Connect**
- Many VPCs connected together  
  → **Transit Gateway**
- Private service exposure across accounts  
  → **PrivateLink**
- Private access to S3 or DynamoDB  
  → **Gateway Endpoint**

---

## Storage

- Object storage with virtually unlimited scale  
  → **S3**
- Shared file system for Linux workloads  
  → **EFS**
- Windows file shares  
  → **FSx for Windows**
- Block storage for EC2 instances  
  → **EBS**
- Long-term archival with lowest cost  
  → **S3 Glacier Deep Archive**

---

## Global & Edge Services

- HTTP caching and CDN distribution  
  → **CloudFront**
- Static IP global endpoint with TCP/UDP acceleration  
  → **Global Accelerator**
- DNS-based routing and failover  
  → **Route 53**
- Single-digit millisecond latency near users  
  → **Local Zones**
- AWS infrastructure inside on-prem data center  
  → **Outposts**

---

## Multi-Account Governance

- Restrict maximum permissions across accounts  
  → **Service Control Policies (SCPs)**
- Deploy infrastructure across multiple accounts  
  → **CloudFormation StackSets**
- Centralized logging and governance  
  → **AWS Organizations** with centralized services
- Shared networking across accounts  
  → **VPC Sharing** via **AWS RAM**

---

## Configuration & Secrets

- Central configuration storage  
  → **SSM Parameter Store**
- Secrets management and automatic rotation  
  → **AWS Secrets Manager**
- Dynamic configuration rollout or feature flags  
  → **AWS AppConfig**

---
# Common Exam Traps

These are common misconceptions that frequently appear in SAP-C02 scenario questions.  
Many incorrect answer options are designed around these misunderstandings.

---

## Availability vs Disaster Recovery

- Multi-AZ does not equal multi-region DR  
  Multi-AZ protects against AZ failure within a region only.
- Read replicas do not provide automatic failover  
  Promotion must occur manually unless using managed failover solutions.
- Backups alone do not provide high availability

---

## Storage

- EBS volumes are AZ-scoped  
  They cannot be attached across AZs.
- EBS is not shared storage  
  Use EFS or FSx for multi-instance file systems.
- S3 provides object storage, not file system semantics
- S3 Cross-Region Replication requires versioning enabled on both buckets  
  Without versioning, CRR cannot be configured; this is a common setup failure.
---

## Messaging

- SNS does not provide durable buffering  
  Use SQS when backpressure handling is required.
- EventBridge is not designed for high-throughput streaming  
  Use Kinesis or MSK for streaming pipelines.
- SQS Standard may deliver duplicate messages  
  Applications must handle idempotency.
- EventBridge archive and replay are disabled by default  
  Must be explicitly enabled; event retention defaults to 0 (no replay).
---

## Serverless

- Lambda maximum execution time is 15 minutes
- Lambda concurrency can overwhelm downstream services  
  Reserved concurrency may be required.
- Lambda inside a VPC requires NAT or VPC endpoints for internet access
- Lambda reserved concurrency protects downstream services  
  Without it, a spike in function invocations can overwhelm database connection limits or queue consumers.
---

## Databases

- Aurora Global Database ≠ multi-writer global database  
  Only one region is writable.
- DynamoDB partition key design affects scalability  
  Poor key selection causes hot partitions.
- DAX improves read latency only; it does not reduce write capacity usage
- RDS Proxy handles reconnection during Multi-AZ failover automatically  
  Without RDS Proxy, applications may experience connection storms during failover.
- On-demand mode is 5–7× more expensive per request than provisioned capacity  
  Best for unpredictable spikes; provisioned capacity + reserved capacity better for sustained load.

---

## Networking

- VPC Peering does not support transitive routing
- Transit Gateway is preferred for many-VPC architectures
- PrivateLink exposes services privately but does not provide full network connectivity

---

## Edge & Global Services

- CloudFront only supports HTTP/HTTPS
- Global Accelerator does not cache content
- Route 53 performs DNS routing but does not accelerate traffic

---

## Security & Governance

- SCPs do not grant permissions  
  They only define maximum allowed permissions.
- Security Hub aggregates findings but does not perform scanning itself
- CloudTrail records activity but does not detect threats without other services
- KMS requires both IAM policy AND key policy for cross-account access  
  IAM policy alone is insufficient; the KMS key policy must explicitly allow the cross-account role ARN.
- This is a common failure pattern: AssumeRole succeeds, S3 access succeeds, but KMS Encrypt fails.

---

## Containers

- Fargate does not support GPUs or DaemonSets
- EKS requires Kubernetes operational knowledge
- Container orchestration does not eliminate the need for scaling and monitoring

---

## Analytics

- Athena is not optimized for high-concurrency BI workloads
- Firehose does not support replay of events
- Glue Data Catalog is the central metadata layer for many analytics services
- Kinesis burst capacity masks partition key problems temporarily  
  A poorly chosen partition key causes sustained throttling even after burst capacity is exhausted.
---