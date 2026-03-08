# AWS Networking Notes (SAP-C02 Level)

VPC Peering • Transit Gateway • Direct Connect • VPN • PrivateLink • Route 53 • Hybrid DNS • Endpoints • Inspection Patterns • Global Connectivity

These notes consolidate core AWS networking constructs, hybrid architectures, routing behavior, service exposure models, and enterprise-scale connectivity patterns relevant for architect-level design decisions.

---

# VPC Connectivity Models

## VPC Peering

**Documentation:**  
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

> **EXAM TIP**  
> VPC Peering is **not transitive** and does not scale well in large multi-account enterprises.  
> Enterprise-wide connectivity → Think **Transit Gateway**.

---

## Transit Gateway (TGW)

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html  

Transit Gateway acts as a scalable hub for VPC and hybrid connectivity.

### Characteristics

- Transitive routing  
- Hub-and-spoke topology  
- Connects VPCs, VPNs, Direct Connect  
- Multiple route tables (segmentation capability)  
- High throughput  

> **EXAM TIP**  
> Many VPCs + Hybrid connectivity + Central control → **Transit Gateway**.

---

## TGW Route Table Mechanics

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html  

Two separate controls exist:

### Association
- Each attachment must be associated with exactly one TGW route table  
- Determines which routing domain the attachment uses  

### Propagation
- Attachments can propagate routes into one or more TGW route tables  
- Missing propagation causes silent traffic drops  

Blackhole routes drop traffic without explicit logging.

> **EXAM TIP**  
> Association ≠ Propagation.  
> Missing propagation → Traffic silently fails.

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

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html  

- Enables internet access for public subnets  
- Requires public or Elastic IP  
- Route: `0.0.0.0/0 → IGW`

---

## NAT Gateway

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html  

- Enables private subnets to access internet  
- Required for:
  - Public AWS endpoints (STS, etc.)  
  - External APIs  
- Deploy per AZ for high availability  

> **EXAM TIP**  
> Private subnet + No NAT + No VPC Endpoint →  
> STS / public API calls will fail.

---

# VPC Endpoints

**Overview Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html  

---

## Gateway Endpoint

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html  

- Only for S3 and DynamoDB  
- Free  
- Route-table based  
- No security groups  

> **EXAM TIP**  
> Gateway Endpoint alone does **not** solve cross-account S3 access.  
> IAM + bucket policy still required.

---

## Interface Endpoint (PrivateLink)

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

## PrivateLink (Service Exposure Pattern)

PrivateLink enables private service consumption across VPCs or accounts.

### Key Characteristics

- Supports overlapping CIDRs  
- Not transitive  
- Requires endpoint creation in every consumer VPC  
- Cannot be forwarded via TGW or peering  

> **EXAM TIP**  
> Overlapping CIDR + Private service exposure → **PrivateLink**.  
> Requires **Network Load Balancer (NLB)** on provider side.

---

### PrivateLink Structural Requirements

#### Service Provider VPC
- Network Load Balancer (required)  
- VPC Endpoint Service  

#### Consumer VPC
- Interface Endpoint  
- Security groups control access  

---

# Hybrid Connectivity

## Site-to-Site VPN

**Documentation:**  
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

**Documentation:**  
https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html  

Provides dedicated connectivity between on-prem and AWS.

### Characteristics

- Not encrypted by default  
- Predictable latency  
- High throughput  

> **EXAM TIP**  
> DX = **NOT encrypted**  
> VPN = **Encrypted**  
> DX + VPN together = Recommended hybrid resilience pattern.

---

## Asymmetric Routing Risk

When multiple hybrid paths exist (DX + VPN):

- Traffic may enter via DX  
- Return via VPN or IGW  
- Stateful firewalls may drop return traffic  

> **EXAM TIP**  
> Hybrid failover requires routing symmetry validation.

---

# Transit Gateway Advanced Patterns

## Centralized Inspection VPC

**AWS Network Firewall Documentation:**  
https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html  

Enterprise pattern:

- TGW routes traffic through inspection VPC  
- AWS Network Firewall performs L3/L7 inspection  

Flow:  
VPC → TGW → Inspection VPC → TGW → Destination  

> **EXAM TIP**  
> Centralized egress inspection → Route `0.0.0.0/0` via inspection VPC.  
> Avoid NAT in every VPC.

---

## TGW Peering (Cross-Region)

**Documentation:**  
https://docs.aws.amazon.com/vpc/latest/tgw/tgw-peering.html  

- Connects Transit Gateways across regions  
- Not transitive across multiple peered TGWs  
- Requires routing configuration on both sides  

> **EXAM TIP**  
> TGW peering is **not globally transitive**.

---

# Route 53

## Routing Policies

**Documentation:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html  

| Policy | Use Case |
|--------|----------|
| Weighted | Gradual traffic shifting |
| Latency | Lowest RTT region |
| Geolocation | Country-based routing |
| Geoproximity | Traffic bias adjustment |
| Failover | Active-passive |
| Multi-value | Health-checked round robin |

Failover routing requires health checks unless using ALIAS targets.

> **EXAM TIP**  
> Route 53 = DNS routing only.  
> Does **not** accelerate traffic.

---

## Hybrid DNS (Route 53 Resolver)

**Documentation:**  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html  

To resolve private hosted zones from on-prem:

- Create inbound resolver endpoint in VPC  
- Forward DNS queries from on-prem  
- Use outbound endpoint for AWS → on-prem resolution  

> **EXAM TIP**  
> Private hosted zones are **not automatically resolvable** from on-prem.  
> Resolver endpoints are required.

---

# Security Boundaries

## Security Groups vs NACL

| Feature | Security Group | NACL |
|----------|---------------|------|
| Stateful | Yes | No |
| Applied To | ENI | Subnet |
| Return Traffic | Automatic | Must allow explicitly |

> **EXAM TIP**  
> Unexpected return traffic issues → Often NACL misconfiguration.

---

# Global Connectivity

## Global Accelerator vs Route 53

**Global Accelerator Documentation:**  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html  

| Feature | Route 53 | Global Accelerator |
|----------|----------|-------------------|
| Type | DNS-based | Anycast static IP |
| Failover | TTL-based | Immediate |
| Path | Internet | AWS backbone |
| Use Case | Regional routing | Fast L4 failover + static IP |

> **EXAM TIP**  
> Need static IP + fast L4 failover → Global Accelerator.  
> Need DNS-based regional routing → Route 53.

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

---