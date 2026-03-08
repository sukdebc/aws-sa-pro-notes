# Compute, Auto Scaling & Spot Notes (SAP-C02 Level)

EC2 • Auto Scaling • Spot • Placement Groups • Savings Plans • Capacity Rebalancing • Lifecycle Hooks • Instance Refresh • Capacity Reservations

These notes capture architectural patterns, failure-domain awareness, elasticity control, interruption trade-offs, and cost alignment themes that repeatedly appear in SAP-C02 scenarios.

The emphasis is on understanding availability scope, scaling behavior, interruption tolerance, and workload alignment — not memorizing features.

---

# EC2 – Architectural Baseline

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html  

EC2 provides raw compute within an Availability Zone.

Core architectural properties:

- AZ-scoped failure domain  
- Instance lifecycle management  
- Capacity type selection (On-Demand, Spot, Reserved, Savings Plans)  
- AMI-driven immutability patterns  
- Networking + security boundary alignment  

A single EC2 instance is never highly available.

High availability requires:

- Multi-AZ deployment  
- Load balancing  
- Automated replacement  

> **EXAM TIP**  
> EC2 alone is never the HA solution.  
> Look for Multi-AZ + ELB + Auto Scaling in resilient architectures.

---

# Auto Scaling – Elastic Control Plane

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html  

Auto Scaling provides:

- Elasticity  
- Self-healing  
- Cost control via scale-in  
- Rolling updates  

Elasticity ≠ Availability.  
Availability depends on AZ distribution and health checks.

---

## Health Checks

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html  

Supported checks:

- EC2 status checks  
- ELB health checks  

ELB health checks detect application-level failures.

Without ELB health checks, a crashed application may not be replaced.

> **EXAM TIP**  
> Application failure detection → Use ELB health checks, not just EC2 checks.

---

## Scaling Policies – Behavioral Fit

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html  

| Policy | Typical Fit | Observations |
|---------|------------|--------------|
| Target Tracking | Most workloads | Simplest and default choice |
| Step Scaling | Fine control required | Threshold-based scaling |
| Scheduled Scaling | Predictable traffic | Known patterns |
| Predictive Scaling | Forecast-based | ML-driven growth modeling |

Target Tracking is usually correct unless precise threshold control is required.

> **EXAM TIP**  
> If no special requirement is stated, Target Tracking is typically sufficient.

---

## Launch Template vs Launch Configuration

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html  

| Feature | Launch Template | Launch Config |
|----------|----------------|---------------|
| Mixed Instances | Supported | Not supported |
| Spot Integration | Full | Limited |
| Versioning | Yes | No |
| Advanced Networking | Yes | Limited |
| Capacity Reservations | Supported | Limited |

Launch Templates are the modern standard.

---

## Instance Refresh

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html  

Instance Refresh:

- Gradual instance replacement  
- Rolling AMI updates  
- Maintains minimum healthy percentage  
- Safer than manual termination  

Preferred approach for production AMI updates.

---

## Lifecycle Hooks

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html  

Lifecycle Hooks allow:

- Custom launch actions  
- Graceful termination  
- Log export  
- Data draining  
- Checkpoint saving  

Often used for Spot interruption handling.

---

# Mixed Instance Policies

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html  

Allows blending:

- Multiple instance types  
- On-Demand + Spot capacity  
- Allocation strategies  

Improves:

- Cost efficiency  
- Capacity diversification  
- Interruption resilience  

Diversification reduces capacity risk.

---

# Spot Instances – Interruption Awareness

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html  

Spot provides cost savings with interruption risk.

Architectural question:  
Is the workload interruption-tolerant?

---

## Spot Allocation Strategies

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-allocation-strategies.html  

| Strategy | Characteristics |
|-----------|----------------|
| capacity-optimized | Lowest interruption risk |
| price-capacity-optimized | Balanced |
| lowest-price | Highest interruption risk |

Lowest-price is rarely correct for production HA workloads.

> **EXAM TIP**  
> Production + Spot → capacity-optimized allocation.

---

## Interruption Handling

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-instance-termination-notices.html  

- 2-minute termination notice  
- Lifecycle hooks  
- Checkpointing  
- Graceful shutdown logic  

Workloads must tolerate termination.

---

## Capacity Rebalancing

Documentation  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-capacity-rebalancing.html  

When interruption risk increases:

- ASG proactively launches replacement  
- Reduces disruption window  
- Improves production safety  

Critical when running Spot in HA systems.

---

## Suitable Workloads for Spot

- Stateless services  
- Batch processing  
- CI/CD runners  
- Big data  
- Container worker nodes  

---

## Unsuitable Workloads for Spot

- Primary databases  
- Stateful single-instance systems  
- Single-AZ clusters  
- Non-checkpointable long-running jobs  

---

# EC2 Placement Groups

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html  

| Type | Use Case | Characteristic |
|------|----------|----------------|
| Cluster | HPC / ML | Low latency within AZ |
| Spread | Small critical systems | Instance-level isolation |
| Partition | Large distributed systems | Partition-level fault isolation |

Placement Groups operate within a single AZ.

---

# Elastic Fabric Adapter

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html  

Used for:

- HPC  
- Distributed ML  
- MPI workloads  

Typically paired with:

- Cluster Placement Groups  
- Specialized instance types  

---

# Capacity Reservations

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-reservations.html  

Used when:

- Guaranteed AZ capacity required  
- Regulatory constraints  
- High-demand regions  

Capacity Reservations guarantee physical capacity.

Different from Savings Plans, which are financial commitments only.

> **EXAM TIP**  
> Guaranteed capacity requirement → Capacity Reservation, not Savings Plan.

---

# EC2 Fleet

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet.html  

EC2 Fleet:

- Launches On-Demand + Spot  
- Multiple instance types  
- Capacity diversification  

Often used for distributed or large-scale compute.

---

# Cost Optimization Patterns

Documentation  

Savings Plans  
https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html  

Reserved Instances  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html  

Graviton  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/graviton.html  

| Option | Fit |
|---------|-----|
| Savings Plans | Flexible steady baseline |
| Reserved Instances | Fixed predictable workloads |
| Spot | Interruption-tolerant workloads |
| Graviton | Better price/performance |

Savings Plans are generally more flexible than Reserved Instances.

---

# Dedicated Instances vs Dedicated Hosts

Documentation  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html  

| Feature | Dedicated Instance | Dedicated Host |
|----------|------------------|----------------|
| Hardware Isolation | Yes | Yes |
| Host Visibility | No | Yes |
| License Control | Limited | Full |
| BYOL | Limited | Strong support |

Dedicated Hosts are commonly used for:

- BYOL licensing  
- Compliance-sensitive workloads  

---

# Load Balancer Interaction

Documentation  
https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html  

ALB:

- HTTP/HTTPS  
- Path-based routing  

NLB:

- TCP/UDP  
- Ultra-low latency  

ELB health checks integrate with Auto Scaling for replacement automation.

---

# Common Compute Themes in SAP-C02

- Elasticity does not imply availability  
- Spot requires interruption tolerance  
- Launch Templates over Launch Configurations  
- Multi-AZ required for resilience  
- ELB health checks critical  
- Diversify instance types for Spot  
- Handle AMI updates via Instance Refresh  

---

# Design Considerations

Compute architecture decisions usually align with:

- Failure domain scope (AZ vs Region)  
- Interruption tolerance  
- Startup latency sensitivity  
- Traffic predictability  
- State management strategy  
- Cost baseline vs burst model  
- Capacity guarantees required  

Resilience is achieved through distribution and automation.  
Cost efficiency comes from matching workload behavior to capacity type.