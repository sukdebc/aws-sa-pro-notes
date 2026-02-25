# AWS Networking Notes (SAP-C02 Level)

VPC Peering • Transit Gateway • Direct Connect • VPN • PrivateLink • Route 53 • Hybrid DNS • Endpoints • Inspection Patterns • Global Connectivity

These notes consolidate core AWS networking constructs, hybrid architectures, routing behavior, service exposure models, and enterprise-scale connectivity patterns relevant for architect-level design decisions.

---

# VPC Connectivity Models

## VPC Peering

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html  

VPC Peering provides private connectivity between two VPCs.

### Characteristics

- One-to-one relationship  
- Non-transitive  
- No overlapping CIDRs  
- Supports inter-region peering  
- Requires route updates on both VPCs  
- No centralized routing control  

### Suitable For

- Small number of VPCs  
- Simple connectivity  
- Low operational complexity  

---

## Transit Gateway (TGW)

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html  

Transit Gateway acts as a scalable hub for VPC and hybrid connectivity.

### Characteristics

- Transitive routing  
- Hub-and-spoke topology  
- Connects VPCs, VPNs, Direct Connect  
- Multiple route tables (segmentation capability)  
- High throughput  

---

## TGW Route Table Mechanics

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html  

Two separate controls exist:

### Association
- Each attachment must be associated with exactly one TGW route table.
- Determines which routing domain the attachment uses.

### Propagation
- Attachments can propagate routes into one or more TGW route tables.
- Missing propagation causes silent traffic drops.

Blackhole routes in TGW drop traffic without explicit logging.

---

## Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|----------|-------------|----------------|
| Routing Model | Mesh (manual) | Hub-and-spoke |
| Transitive | No | Yes |
| Scale | Small | Large (100s of VPCs) |
| Centralized Control | No | Yes |
| Hybrid Connectivity | No | Yes |
| Overlapping CIDR Allowed | No | No |

---

# Internet & Private Connectivity

## Internet Gateway (IGW)

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html  

- Enables internet access for public subnets  
- Requires public IP or Elastic IP  
- Route: `0.0.0.0/0 → IGW`  

---

## NAT Gateway

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html  

- Enables private subnets to access internet  
- Required for:
  - Public AWS endpoints (STS, etc.)
  - Cross-account S3 access without interface endpoint
  - External APIs  
- Deployed per AZ for high availability  

---

## VPC Endpoints

**Overview Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html  

---

### Gateway Endpoint

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html  

- Only for S3 and DynamoDB  
- Free  
- Route-table based  
- Same-account S3 access  
- No security groups  

---

### Interface Endpoint (PrivateLink)

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html  

- ENI-based  
- Requires security groups  
- Supports most AWS services  
- Works across accounts  
- Not transitive  

---

## Endpoint Comparison

| Feature | Gateway Endpoint | Interface Endpoint |
|----------|------------------|-------------------|
| Services | S3, DynamoDB | Most AWS APIs |
| Cost | Free | Per endpoint + data |
| Cross-Account | Limited | Yes |
| Security Groups | No | Yes |
| Transitive | No | No |

---

# PrivateLink (Service Exposure Pattern)

**AWS Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html  

PrivateLink enables private service consumption across VPCs or accounts.

---

## Architecture Model

### Service Provider VPC

- Network Load Balancer (required)  
- VPC Endpoint Service  

**Endpoint Service Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/create-endpoint-service.html  

### Consumer VPC

- Interface Endpoint  
- Security groups control access  

PrivateLink:

- Supports overlapping CIDRs  
- Is not transitive  
- Requires endpoint creation in every consumer VPC  
- Cannot be forwarded via TGW or peering  

---

# Hybrid Connectivity

## Site-to-Site VPN

**AWS Documentation:**  
https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html  

- Encrypted  
- Two tunnels  
- Supports BGP  
- Route propagation required in VPC/TGW route tables  

Used for:

- Fast hybrid setup  
- Backup to Direct Connect  

---

## Direct Connect (DX)

**AWS Documentation:**  
https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html  

Provides dedicated connectivity between on-prem and AWS.

---

### Virtual Interface Types

**Documentation:**  
https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html  

| Type | Purpose |
|------|--------|
| Private VIF | VPC connectivity |
| Public VIF | Public AWS services |
| Transit VIF | TGW connectivity |

---

### Characteristics

- Not encrypted by default  
- Predictable latency  
- High throughput  
- Can combine with VPN for encryption and failover  

---

## DX + VPN Hybrid Pattern

**Hybrid Connectivity Whitepaper:**  
https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/hybrid-connectivity.html  

Common enterprise design:

- DX for primary connectivity  
- VPN attached to same TGW  
- Automatic failover via BGP route priority  

---

## Asymmetric Routing Risk

When multiple hybrid paths exist (DX + VPN):

- Traffic may enter via DX  
- Return via VPN or IGW  

Stateful firewalls may drop return traffic.

Ensure routing symmetry when designing hybrid failover.

---

# Transit Gateway Advanced Patterns

## Centralized Inspection VPC

**AWS Network Firewall Documentation:**  
https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html  

Enterprise pattern:

- TGW routes traffic through inspection VPC  
- AWS Network Firewall performs L3/L7 inspection  
- Used for segmentation and compliance  

Flow:

VPC → TGW → Inspection VPC → TGW → Destination  

---

## TGW Peering (Cross-Region)

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/tgw-peering.html  

- Connects Transit Gateways across regions  
- Enables global hub architecture  
- Not transitive across multiple peered TGWs  
- Requires routing configuration on both sides  

---

# Route 53

## Routing Policies

**Documentation:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html  

| Policy | Use Case |
|--------|----------|
| Simple | Single record |
| Weighted | Gradual traffic shifting |
| Latency | Lowest RTT region |
| Geolocation | Country-based routing |
| Geoproximity | Traffic bias adjustment |
| Failover | Active-passive |
| Multi-value | Health-checked round robin |

Failover routing requires health checks unless using supported ALIAS targets (e.g., ALB, CloudFront).

---

## Hybrid DNS (Route 53 Resolver)

**Documentation:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html  

To resolve private hosted zones from on-prem:

- Create inbound resolver endpoint in VPC  
- Forward DNS queries from on-prem to inbound endpoint  
- Use outbound endpoint for AWS → on-prem resolution  

Private hosted zones resolve only inside associated VPCs unless resolver endpoints are configured.

---

# Security Boundaries

## Security Groups vs NACL

**Security Groups Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html  

**NACL Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html  

| Feature | Security Group | NACL |
|----------|---------------|------|
| Stateful | Yes | No |
| Applied To | ENI | Subnet |
| Default Action | Deny | Allow |
| Return Traffic | Automatic | Must allow explicitly |

Unexpected return traffic issues often involve NACL configuration.

---

# Global Connectivity

## Global Accelerator vs Route 53

**Global Accelerator Documentation:**  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html  

| Feature | Route 53 | Global Accelerator |
|----------|----------|-------------------|
| Type | DNS-based | Anycast static IP |
| Failover | DNS update | Immediate |
| Path | Internet | AWS backbone |
| Use Case | Regional routing | Fast failover + static IP |

Global Accelerator provides static anycast IPs and faster regional failover for TCP/UDP workloads.

---

# Common Pitfalls

- Assuming VPC peering supports transitive routing  
- Forgetting TGW route propagation configuration  
- Overlooking asymmetric routing in hybrid designs  
- Expecting PrivateLink to be transitive  
- Attempting to attach overlapping CIDRs to TGW  
- Assuming Direct Connect is encrypted  
- Forgetting NAT requirement for STS in private subnets  
- Using gateway endpoint expecting cross-account S3 support  
- Misunderstanding Route 53 latency vs geolocation routing  
- Forgetting PrivateLink requires NLB on provider side  