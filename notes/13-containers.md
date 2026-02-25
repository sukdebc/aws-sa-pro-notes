# AWS Containers Notes (SAP-C02 Level)

ECS • EKS • Fargate • Container Networking • IAM for Containers • Scaling • Storage • Service Mesh • Security

These notes summarize orchestration models, compute choices, networking, IAM integration, scaling strategies, and deployment patterns for containerized workloads on AWS.

The focus is on architectural trade-offs between ECS and EKS, serverless vs EC2-backed containers, and operational control vs simplicity.

---

# ECS vs EKS

## Documentation
- ECS Overview:  
  https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html  
- EKS Overview:  
  https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html  

The primary distinction is orchestration model and operational complexity.

---

## ECS (Elastic Container Service)

- AWS-native orchestration
- Deep AWS integration (ALB, IAM, CloudWatch)
- Simpler operational model
- Runs on EC2 or Fargate
- No Kubernetes control plane management
- Suitable when Kubernetes portability is not required

ECS is typically selected for AWS-centric microservices and cost-efficient operations.

---

## EKS (Elastic Kubernetes Service)

- Managed Kubernetes control plane
- Full Kubernetes API compatibility
- Runs on EC2 or Fargate
- Supports service mesh, CRDs, custom controllers
- Higher operational complexity

EKS is commonly chosen when Kubernetes ecosystem compatibility, portability, or advanced control is required.

---

## ECS vs EKS Comparison

| Dimension | ECS | EKS |
|------------|------|------|
| Orchestration | AWS-native | Kubernetes |
| Operational Complexity | Lower | Higher |
| Portability | AWS-focused | Kubernetes standard |
| Custom Controllers | No | Yes |
| Service Mesh Integration | Supported | Native ecosystem support |
| Ecosystem Flexibility | Limited | Extensive |

---

# Fargate vs EC2 (Applies to ECS and EKS)

## Documentation
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html  

This distinction determines whether container infrastructure is serverless or self-managed.

---

## Fargate

- Serverless container runtime
- No EC2 lifecycle management
- Per-task CPU and memory definition
- Strong isolation model
- Well-suited for spiky or unpredictable workloads

Limitations:

- No DaemonSets
- No privileged containers
- No GPUs
- Limited customization of networking stack

---

## EC2-backed Containers

- Full control over instances
- Supports DaemonSets
- Required for GPU workloads
- Supports custom CNI configurations
- More cost-effective at high steady-state usage

---

## Fargate vs EC2 Comparison

| Dimension | Fargate | EC2 |
|------------|----------|------|
| Instance Management | None | Required |
| DaemonSet Support | No | Yes |
| GPU Support | No | Yes |
| Custom Networking | Limited | Full control |
| Cost Efficiency | Better at low usage | Better at scale |
| Operational Overhead | Low | Higher |

---

# ECS Architecture

## Documentation
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html  

## ECS Launch Types

- ECS on EC2
- ECS on Fargate

Capacity Providers:
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html  

Allow ECS to manage EC2 scaling automatically and integrate with Auto Scaling Groups.

Cluster Auto Scaling improves resource provisioning without manual scaling logic.

ECS networking best practice is `awsvpc` mode, which assigns ENIs per task.

---

# EKS Architecture

## Documentation
https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html  

EKS separates:

- Managed control plane (AWS-managed)
- Worker nodes (EC2 or Fargate)

Node groups may be:

- Managed node groups  
- Self-managed  
- Fargate profiles  

Managed Node Groups:
https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html  

---

## EKS Networking

Documentation:  
https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html  

- Pods receive IPs from VPC subnets
- Uses AWS VPC CNI
- ENI limits affect pod scaling
- Prefix delegation increases IP density

Subnet IP exhaustion is a common scaling bottleneck.

---

# IAM for Containers

Fine-grained IAM control is critical.

---

## ECS Task Roles

Documentation:  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html  

- Assigned per task
- Enable least privilege per service
- Avoid reliance on EC2 instance profile

---

## IRSA (IAM Roles for Service Accounts)

Documentation:  
https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html  

- Maps Kubernetes service accounts to IAM roles
- Uses OIDC provider
- Enables per-pod permission isolation

IRSA eliminates the need to grant broad permissions to node IAM roles.

---

# Container Networking

## ECS Networking Modes

Documentation:  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html  

| Mode | Characteristics |
|------|-----------------|
| awsvpc | ENI per task, VPC-native |
| bridge | Legacy Docker bridge |
| host | Shares host network |

`awsvpc` enables security groups per task.

---

## EKS Networking

Documentation:  
https://docs.aws.amazon.com/eks/latest/userguide/cni-network-policy.html  

Design must consider:

- Subnet size
- ENI limits
- Pod density
- Multi-AZ spread

---

# Load Balancing

## Documentation
- ALB: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html  
- NLB: https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html  

---

## ECS

- ALB commonly used
- Supports dynamic port mapping
- NLB for TCP/UDP workloads

---

## EKS

AWS Load Balancer Controller:  
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html  

- Manages ALBs
- Service type LoadBalancer provisions NLB
- Ingress resources define routing rules

---

## ALB vs NLB Comparison

| Dimension | ALB | NLB |
|------------|------|------|
| Layer | 7 | 4 |
| Path-based routing | Yes | No |
| TCP/UDP support | No | Yes |
| gRPC | Limited | Yes |
| Latency | Moderate | Very low |

---

# Scaling Models

## ECS Scaling

Documentation:  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html  

- Service auto scaling
- CPU/memory/SQS/ALB metrics
- Capacity Providers for EC2 scaling

---

## EKS Scaling

- Horizontal Pod Autoscaler (HPA)  
- Cluster Autoscaler  
- Vertical Pod Autoscaler  

Cluster Autoscaler:  
https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html  

Pod and node scaling must be coordinated.

---

# Storage for Containers

## ECS

Documentation:  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/efs-volumes.html  

- EFS supported
- EBS for EC2 tasks
- Fargate supports ephemeral storage

---

## EKS

Documentation:  
https://docs.aws.amazon.com/eks/latest/userguide/storage.html  

- CSI drivers (EBS, EFS)
- PersistentVolumes (PV)
- PersistentVolumeClaims (PVC)
- StatefulSets require persistent backing

---

# Service Mesh & Advanced Routing

AWS App Mesh:
https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html  

Supports:

- mTLS
- Traffic shifting
- Canary deployments
- Service-to-service observability

Adds complexity but enables advanced routing.

---

# Container Security

## Documentation
- ECR Scanning:  
  https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html  
- Bottlerocket:  
  https://docs.aws.amazon.com/bottlerocket/latest/userguide/what-is-bottlerocket.html  

Security patterns:

- ECR vulnerability scanning
- AWS Inspector integration
- IRSA for least privilege
- Secrets Manager
- KMS encryption
- Bottlerocket OS for hardened nodes

---

# Deployment Strategies

## ECS

Documentation:  
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-types.html  

- Rolling updates
- Blue/green via CodeDeploy

---

## EKS

Documentation:  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/  

- RollingUpdate (default)
- Canary
- Blue/green
- Traffic shifting via App Mesh

---

# Cost Considerations

- Fargate reduces operational overhead
- EC2 improves cost efficiency at scale
- Graviton reduces compute cost
- Over-provisioned nodes increase waste in EKS
- Spot for container worker nodes can significantly reduce cost

---

# Common Pitfalls Observed in Container Architectures

- Selecting EKS without portability requirement
- Choosing Fargate for workloads requiring DaemonSets or GPU
- Ignoring subnet IP exhaustion
- Granting node-level permissions instead of IRSA/task roles
- Confusing ALB vs NLB capabilities
- Treating service mesh as mandatory
- Ignoring control plane logging for EKS

---

Container decisions become clearer when evaluated across:

- Portability vs simplicity
- Control vs operational overhead
- Stateless vs stateful design
- Networking constraints
- IAM isolation model
- Scaling independence (pods vs nodes)
- Cost profile over time