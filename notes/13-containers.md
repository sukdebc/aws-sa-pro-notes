# AWS Containers Notes (SAP-C02 Level)

ECS • EKS • Fargate • Container Networking • IAM for Containers • Scaling • Storage • Service Mesh • Security

These notes summarize orchestration models, compute choices, networking, IAM integration, scaling strategies, and deployment patterns for containerized workloads on AWS.

The focus is on architectural trade-offs between ECS and EKS, serverless versus EC2-backed containers, and operational control versus simplicity.

---

# ECS vs EKS

Documentation

ECS Overview  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html  

EKS Overview  
https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html  

The primary distinction is orchestration model and operational complexity.

---

## Amazon ECS

ECS is AWS-native container orchestration.

Core characteristics:

- Native AWS orchestration model
- Deep integration with ALB, IAM, CloudWatch, and ECS service scaling
- Simpler operational model than Kubernetes
- Supports EC2 and Fargate launch types
- No Kubernetes control plane management
- Well suited for AWS-centric microservices

ECS is typically chosen when Kubernetes portability is not required and operational simplicity matters.

> **EXAM TIP**  
> AWS-centric containers with lower operational overhead → ECS.

---

## Amazon EKS

EKS provides managed Kubernetes control plane capabilities.

Core characteristics:

- Managed Kubernetes control plane
- Full Kubernetes API compatibility
- Supports EC2 worker nodes and Fargate
- Supports CRDs, operators, and Kubernetes-native tooling
- Higher operational complexity than ECS

EKS is commonly selected when Kubernetes portability, ecosystem tooling, or advanced control is required.

> **EXAM TIP**  
> Kubernetes compatibility or ecosystem requirement → EKS.

---

## ECS vs EKS Comparison

| Dimension | ECS | EKS |
|----------|-----|-----|
| Orchestration Model | AWS-native | Kubernetes |
| Operational Complexity | Lower | Higher |
| Portability | AWS-focused | Kubernetes standard |
| Custom Controllers | No | Yes |
| Ecosystem Flexibility | Limited | Extensive |
| Kubernetes API | No | Yes |

---

# Fargate vs EC2

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html  

This choice determines whether container infrastructure is serverless or instance-managed.

---

## AWS Fargate

Fargate is a serverless runtime for containers.

Core characteristics:

- No EC2 lifecycle management
- Task or pod level CPU and memory configuration
- Strong workload isolation
- Well suited for spiky or unpredictable workloads
- Lower operational overhead

Key limitations:

- No GPU support
- No privileged containers
- Limited host-level customization
- No DaemonSet support for EKS on Fargate

---

## EC2-backed Containers

EC2-backed containers provide full control over worker nodes.

Core characteristics:

- Full instance-level control
- Supports DaemonSets
- Supports GPU workloads
- More flexible networking customization
- More cost-efficient for high steady-state usage

EC2-backed compute is usually preferred when advanced customization or lower long-term cost is more important than simplicity.

---

## Fargate vs EC2 Comparison

| Dimension | Fargate | EC2 |
|----------|---------|-----|
| Instance Management | None | Required |
| DaemonSet Support | No | Yes |
| GPU Support | No | Yes |
| Custom Networking | Limited | Full control |
| Operational Overhead | Low | Higher |
| Cost Efficiency | Better for smaller or bursty workloads | Better at steady scale |

> **EXAM TIP**  
> DaemonSets, GPUs, or host customization required → EC2-backed containers.

---

# ECS Architecture

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html  

ECS supports two launch models:

- ECS on EC2
- ECS on Fargate

Capacity Providers allow ECS to manage infrastructure scaling more intelligently.

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html  

Capacity Providers support:

- Integration with Auto Scaling Groups
- Managed scaling for EC2-backed clusters
- Managed termination handling

Cluster Auto Scaling reduces manual infrastructure provisioning logic.

ECS networking best practice is usually `awsvpc` mode, which gives each task its own ENI.

---

# EKS Architecture

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html  

EKS separates the managed control plane from the data plane.

EKS worker options include:

- Managed node groups
- Self-managed nodes
- Fargate profiles

Managed node groups reduce operational burden compared with self-managed worker nodes.

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html  

---

## EKS Networking

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html  

EKS commonly uses the AWS VPC CNI.

Core characteristics:

- Pods receive IP addresses from VPC subnets
- ENI limits can constrain pod density
- Prefix delegation can increase IP density
- Subnet IP exhaustion is a common bottleneck

> **EXAM TIP**  
> EKS scaling issue with pods often points to subnet IP or ENI limits.

---

# IAM for Containers

Fine-grained IAM control is essential for container security and least privilege design.

---

## ECS Task Roles

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html  

Task roles are assigned at the ECS task level.

Benefits:

- Least privilege per service
- Avoids relying on the EC2 instance profile
- Cleaner permission boundaries across services

---

## IAM Roles for Service Accounts

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html  

IAM Roles for Service Accounts, often called IRSA, map Kubernetes service accounts to IAM roles.

Core characteristics:

- Uses OIDC federation
- Enables pod-level IAM isolation
- Reduces reliance on broad node IAM permissions

IRSA is the preferred permission model for EKS workloads needing AWS API access.

> **EXAM TIP**  
> Per-pod IAM permissions in EKS → IRSA.

---

# Container Networking

## ECS Networking Modes

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html  

| Mode | Characteristics |
|------|-----------------|
| awsvpc | ENI per task, VPC-native networking |
| bridge | Docker bridge networking |
| host | Shares host network namespace |

`awsvpc` mode allows security groups at the task level and is generally the preferred mode.

---

## EKS Networking Design

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/cni-network-policy.html  

Networking design must consider:

- Subnet sizing
- ENI limits
- Pod density
- Multi-AZ placement
- Security group strategy

Container networking decisions often become scaling decisions.

---

# Load Balancing

Documentation

Application Load Balancer  
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html  

Network Load Balancer  
https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html  

---

## ECS Load Balancing

ECS commonly uses:

- ALB for HTTP and HTTPS workloads
- NLB for TCP and UDP workloads

Dynamic port mapping is commonly used with ALB in ECS service designs.

---

## EKS Load Balancing

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html  

The AWS Load Balancer Controller manages:

- ALB for ingress-based HTTP routing
- NLB for Kubernetes Service type LoadBalancer
- Integration with Kubernetes ingress resources

---

## ALB vs NLB

| Dimension | ALB | NLB |
|----------|-----|-----|
| OSI Layer | Layer 7 | Layer 4 |
| Path-based Routing | Yes | No |
| TCP/UDP Support | No | Yes |
| gRPC Support | Limited | Yes |
| Latency | Moderate | Very low |

> **EXAM TIP**  
> Path-based routing → ALB.  
> TCP, UDP, or ultra-low latency → NLB.

---

# Scaling Models

## ECS Scaling

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html  

ECS scaling includes:

- Service Auto Scaling
- Scaling based on CPU, memory, SQS, or ALB metrics
- Capacity Providers for underlying EC2 capacity scaling

ECS scaling is simpler because service and infrastructure scaling are tightly integrated.

---

## EKS Scaling

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html  

EKS scaling typically involves:

- Horizontal Pod Autoscaler
- Cluster Autoscaler
- Vertical Pod Autoscaler

Pod scaling and node scaling must be coordinated carefully.

This separation adds flexibility but increases operational complexity.

---

# Storage for Containers

## ECS Storage

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/efs-volumes.html  

ECS supports:

- Amazon EFS for shared persistent storage
- Amazon EBS for EC2-backed tasks
- Ephemeral storage for Fargate tasks

EFS is common when multiple tasks require shared access.

---

## EKS Storage

Documentation  
https://docs.aws.amazon.com/eks/latest/userguide/storage.html  

EKS storage patterns include:

- CSI drivers for EBS and EFS
- PersistentVolumes
- PersistentVolumeClaims
- StatefulSets for stateful applications

Persistent state requires deliberate storage design rather than default container behavior.

---

# Service Mesh and Advanced Routing

Documentation  
https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html  

AWS App Mesh supports advanced service-to-service traffic management.

Capabilities include:

- mTLS
- Traffic shifting
- Canary releases
- Service-to-service observability

Service mesh can enable advanced routing and security controls, but introduces additional complexity.

---

# Container Security

Documentation

Amazon ECR Image Scanning  
https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html  

Bottlerocket  
https://docs.aws.amazon.com/bottlerocket/latest/userguide/what-is-bottlerocket.html  

Common security patterns include:

- ECR vulnerability scanning
- Inspector integration
- Least privilege through task roles or IRSA
- Secrets Manager for secret delivery
- KMS encryption for data protection
- Bottlerocket for hardened container hosts

Security architecture should separate image security, runtime permissions, and secret handling.

---

# Deployment Strategies

## ECS Deployment Patterns

Documentation  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-types.html  

ECS supports:

- Rolling deployments
- Blue/green deployments through CodeDeploy

---

## EKS Deployment Patterns

Documentation  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/  

EKS commonly uses:

- Rolling updates
- Canary deployments
- Blue/green deployments
- Traffic shifting through service mesh or ingress controls

Deployment strategy depends on operational tooling and blast radius tolerance.

---

# Cost Considerations

Container cost decisions often depend on workload shape.

Common patterns:

- Fargate reduces operational overhead
- EC2 is often more cost-efficient at steady scale
- Graviton can improve price-performance
- Over-provisioned EKS nodes create waste
- Spot worker nodes can reduce compute cost significantly

The cheapest runtime is not always the lowest total operating cost.

---

# Common Pitfalls

- Choosing EKS without a portability or Kubernetes requirement  
- Selecting Fargate for workloads requiring DaemonSets or GPUs  
- Ignoring subnet IP exhaustion  
- Granting node-level permissions instead of task roles or IRSA  
- Confusing ALB and NLB capabilities  
- Treating service mesh as mandatory  
- Ignoring EKS control plane logging and observability needs  
- Assuming stateful workloads fit naturally into container defaults  

---

# Design Considerations

Container platform decisions typically become clearer when evaluated across:

- Portability versus simplicity
- Operational control versus overhead
- Stateless versus stateful design
- Networking constraints
- IAM isolation model
- Scaling independence between services and infrastructure
- Cost profile over time

Clear container architectures usually come from choosing the simplest orchestration and compute model that still satisfies networking, security, and workload requirements.
