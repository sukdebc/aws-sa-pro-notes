# Compute, Auto Scaling & Spot Notes (SAP-C02 Level)

EC2 • Auto Scaling • Spot • Placement Groups • Savings • Capacity Rebalancing • Lifecycle Hooks • Instance Refresh • Capacity Reservations

These notes capture architectural patterns, trade-offs, scaling behavior, and resilience themes that repeatedly appear in SAP-C02 scenarios.

---

# Auto Scaling – Architectural Context

📘 AWS Documentation  
- Auto Scaling Overview:  
  https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html  
- Scaling Policies:  
  https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html  
- Launch Templates:  
  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html  

Auto Scaling contributes to three core properties:

- Elasticity (capacity adjusts dynamically)
- Resilience (when deployed across multiple AZs)
- Cost control (scale-in during low demand)

⚠️ Scaling alone does not imply high availability.  
High availability requires:

- Multi-AZ deployment
- Load balancer health checks
- Proper termination policies

---

## Health Checks (Critical Detail)

📘 Documentation:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html  

Auto Scaling supports:

- EC2 status checks
- ELB health checks (recommended)

ELB health checks detect application-level failure.  
Without ELB health checks, failed applications may not be replaced.

---

## Scaling Policies – When They Fit

| Policy | Typical Fit | Observations |
|---------|------------|--------------|
| Target Tracking | Most workloads | Simplest and commonly sufficient |
| Step Scaling | Fine-grained thresholds | Used when precise control required |
| Scheduled Scaling | Predictable traffic | Known daily/seasonal patterns |
| Predictive Scaling | Pattern-based growth | ML-based forecast |

In SAP scenarios, Target Tracking is usually correct unless more control is explicitly required.

---

## Instance Refresh

📘 Documentation:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html  

Instance Refresh:

- Gradually replaces instances
- Supports rolling updates
- Maintains minimum healthy percentage
- Useful for AMI upgrades

Preferred over manual termination during production updates.

---

## Lifecycle Hooks

📘 Documentation:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html  

Lifecycle Hooks allow:

- Custom actions during launch or termination
- Graceful shutdown
- Data draining
- Log export
- Checkpoint saving

Frequently used in Spot interruption handling.

---

## Launch Template vs Launch Configuration

| Feature | Launch Template | Launch Config |
|----------|----------------|---------------|
| Mixed Instance Types | Supported | Not supported |
| Spot Integration | Full support | Limited |
| Versioning | Yes | No |
| Advanced Networking | Yes | Limited |
| Capacity Reservations | Supported | Limited |

Launch Templates are preferred in modern architectures.

---

# Mixed Instance Policies (MIP)

📘 Documentation:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html  

Mixed Instance Policies allow blending:

- Instance families and sizes
- On-Demand and Spot capacity
- Flexible allocation strategies

Improves both cost efficiency and capacity resilience.

---

## Spot Allocation Strategies

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-allocation-strategies.html  

| Strategy | Characteristics |
|-----------|----------------|
| capacity-optimized | Lowest interruption risk |
| price-capacity-optimized | Balanced |
| lowest-price | Highest interruption risk |

For production HA workloads, capacity-optimized is typically safer than lowest-price.

---

# Spot Instances – Trade-Off Awareness

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html  

Spot provides cost savings but introduces interruption risk.

Architectural question:
Is this workload interruption-tolerant?

---

## Interruption Handling

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-instance-termination-notices.html  

- 2-minute termination notice
- Lifecycle hooks
- Checkpointing
- Capacity Rebalancing

---

## Capacity Rebalancing

📘 Documentation:  
https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-capacity-rebalancing.html  

When interruption risk increases:

- ASG launches replacement before termination
- Reduces service disruption
- Critical for production Spot usage

---

## Workloads Suitable for Spot

- Stateless services
- Batch jobs
- CI/CD workers
- Big data processing
- Container hosts

---

## Workloads Unsuitable for Spot

- Databases
- Stateful single-instance workloads
- Single-AZ critical clusters
- Non-checkpointable long-running jobs

---

# EC2 Placement Groups

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html  

| Type | Use Case | Characteristic |
|------|----------|----------------|
| Cluster | HPC / ML | Low latency within AZ |
| Spread | Small critical systems | Instance-level isolation |
| Partition | Large distributed systems | Partition-level isolation |

---

# Elastic Fabric Adapter (EFA)

📘 Documentation:  
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

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-reservations.html  

Used when:

- Guaranteed capacity required
- Regulatory or critical workloads
- AZ capacity constraints

Different from Savings Plans (financial) — this guarantees physical capacity.

---

# EC2 Fleet

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet.html  

EC2 Fleet:

- Launches On-Demand + Spot
- Multiple instance types
- Capacity diversification

Often used for large-scale distributed workloads.

---

# Cost Optimization

📘 Documentation  
- Savings Plans:  
  https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html  
- Reserved Instances:  
  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html  
- Graviton:  
  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/graviton.html  

| Option | Fit |
|---------|-----|
| Savings Plans | Flexible steady usage |
| Reserved Instances | Fixed steady workloads |
| Spot | Interruption-tolerant |
| Graviton | Better price/performance |

Savings Plans are generally more flexible than RIs.

---

# Dedicated Hosts vs Dedicated Instances

📘 Documentation:  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html  

| Feature | Dedicated Instance | Dedicated Host |
|----------|------------------|----------------|
| Hardware Isolation | Yes | Yes |
| Visibility of host | No | Yes |
| License control | Limited | Full |
| Compliance workloads | Moderate | Strong |

Dedicated Hosts used for:

- BYOL licensing
- Regulatory compliance

---

# Load Balancer Interaction (Important)

📘 Documentation:  
https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html  

ALB:

- HTTP/HTTPS
- Path-based routing

NLB:

- TCP/UDP
- Ultra-low latency

ELB health checks integrate with Auto Scaling.

---

# Common Compute Themes in SAP-C02

- Scaling ≠ high availability  
- Spot requires interruption tolerance  
- Launch Templates over Launch Configurations  
- Multi-AZ required for resilience  
- Use ELB health checks  
- Consider startup latency  
- Diversify instance types for Spot  

---

# Mental Model Summary

Resilience:
- Multi-AZ
- Health checks
- Load balancing
- Replacement automation

Elasticity:
- Auto Scaling
- Scaling policy
- Mixed instance diversification

Cost Optimization:
- Spot where safe
- Savings Plans for steady baseline
- Graviton for efficiency
- Avoid over-provisioning

---

# Common Pitfalls Observed in SAP-C02

- Confusing elasticity with availability  
- Choosing lowest-price Spot allocation  
- Ignoring ELB health checks  
- Deploying production in single AZ  
- Not handling Spot interruption  
- Overcomplicating scaling policy  
- Forgetting instance refresh during AMI updates  
- Ignoring capacity constraints in busy regions  

Architectural decisions should always align with workload characteristics, not just pricing or simplicity.