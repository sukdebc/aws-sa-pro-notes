# AWS Security Services Notes

GuardDuty • Macie • Inspector • Detective • Security Hub • WAF • Shield • IAM Access Analyzer • CloudTrail • KMS

These notes summarize AWS-native threat detection, compliance posture management, vulnerability scanning, investigation tooling, encryption controls, and edge protection services.

The focus is on understanding service roles, scope boundaries, and how detection, prevention, investigation, and aggregation layers interact.

---

# Amazon GuardDuty

Documentation  
https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html

GuardDuty provides continuous threat detection across AWS accounts and workloads.

It analyzes:

- VPC Flow Logs
- DNS logs
- CloudTrail management events
- EKS audit logs
- S3 data events (optional)
- RDS login activity (optional)

GuardDuty detects:

- Compromised EC2 instances
- Stolen IAM credentials
- Crypto-mining activity
- Reconnaissance behavior
- Anomalous API usage
- DNS data exfiltration
- Suspicious S3 access

GuardDuty:

- Is agentless
- Does not block traffic
- Generates findings
- Integrates with Security Hub and EventBridge

It is a detection service, not a prevention service.

> **EXAM TIP**  
> GuardDuty detects threats but does not block them.

---

# Amazon Macie

Documentation  
https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html

Macie discovers and classifies sensitive data stored in Amazon S3.

It identifies:

- Personally Identifiable Information (PII)
- Financial information
- Credentials
- API keys
- Sensitive logs

Macie characteristics:

- Operates only on S3
- Performs classification and risk scoring
- Generates findings for data exposure

Macie focuses on data security posture rather than runtime security.

> **EXAM TIP**  
> Sensitive data discovery in S3 → Macie.

---

# Amazon Inspector

Documentation  
https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html

Inspector provides automated vulnerability management.

Inspector scans:

- EC2 instances for software vulnerabilities
- ECR container images for CVEs
- Lambda dependencies

Inspector characteristics:

- Uses SSM agent for EC2 scanning
- Continuously evaluates newly discovered CVEs
- Focuses on vulnerability detection

Inspector answers the question:  
Is this workload vulnerable?

---

# Amazon Detective

Documentation  
https://docs.aws.amazon.com/detective/latest/userguide/what-is-detective.html

Detective supports post-incident investigation.

It builds graph-based relationships across:

- CloudTrail logs
- VPC Flow Logs
- GuardDuty findings
- IAM activity

Detective enables:

- Timeline reconstruction
- Entity behavior analysis
- Root cause investigation

Detective is investigative rather than preventative.

> **EXAM TIP**  
> Security investigation and root cause analysis → Detective.

---

# AWS Security Hub

Documentation  
https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html

Security Hub aggregates findings from multiple AWS security services.

It integrates with:

- GuardDuty
- Inspector
- Macie
- IAM Access Analyzer
- AWS Foundational Security Best Practices
- CIS benchmarks
- PCI DSS controls

Security Hub:

- Centralizes security posture visibility
- Normalizes findings
- Supports cross-account aggregation

Security Hub does not perform scanning itself.

> **EXAM TIP**  
> Centralized security findings across accounts → Security Hub.

---

# AWS WAF

Documentation  
https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html

AWS WAF protects HTTP and HTTPS workloads at the application layer.

It protects against:

- SQL injection
- Cross-site scripting (XSS)
- Malicious bots
- Rate-based abuse
- Geographic filtering

WAF integrates with:

- CloudFront
- Application Load Balancer
- API Gateway
- AppSync

WAF performs application-layer request filtering.

---

# AWS Shield

Documentation  
https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html

Shield provides protection against distributed denial-of-service attacks.

Shield Standard:

- Automatically enabled
- Protects against common network-layer attacks

Shield Advanced:

- Enhanced detection
- Access to DDoS Response Team
- Cost protection during attacks

Shield mitigates volumetric network attacks.

---

## WAF vs Shield

| Dimension | WAF | Shield |
|-----------|-----|--------|
| OSI Layer | Layer 7 | Layer 3/4 |
| Protects Against | Application attacks | Network DDoS |
| HTTP Filtering | Yes | No |
| Rate Limiting | Yes | Infrastructure level |
| Cost Protection | No | Yes (Advanced) |

WAF filters malicious HTTP requests.  
Shield mitigates volumetric DDoS attacks.

---

# IAM Access Analyzer

Documentation  
https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html

IAM Access Analyzer detects unintended external access.

It evaluates resource policies for:

- S3 buckets
- KMS keys
- IAM roles
- SQS queues
- Lambda functions
- Secrets Manager
- ECR repositories

It identifies:

- Public exposure
- Cross-account access
- External principal trust relationships

Access Analyzer highlights exposure but does not block access automatically.

> **EXAM TIP**  
> Detect unintended public or cross-account access → Access Analyzer.

---

# AWS CloudTrail

Documentation  
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html

CloudTrail records API activity across AWS services.

Key architectural considerations:

- Enable multi-region trails
- Use organization trails for centralized logging
- CloudTrail Lake supports SQL analysis

CloudTrail provides audit visibility.

It is often used as a data source for:

- GuardDuty
- Detective
- Security analysis workflows

> **EXAM TIP**  
> CloudTrail records activity but does not detect threats itself.

---

# AWS KMS

Documentation  
https://docs.aws.amazon.com/kms/latest/developerguide/overview.html

KMS manages encryption keys used by AWS services.

Key types:

- AWS-managed keys
- Customer-managed keys
- Custom key stores backed by CloudHSM

KMS permission model includes:

- IAM policies
- Key policies
- Grants

Key policies are authoritative.

KMS keys are region-scoped.

Automatic rotation is available for supported customer-managed symmetric keys.

> **EXAM TIP**  
> Even if IAM allows access, the key policy must also permit it.

---

# Security Service Roles

## Detection vs Investigation vs Aggregation

| Service | Detection | Investigation | Aggregation |
|--------|-----------|--------------|-------------|
| GuardDuty | Yes | No | No |
| Macie | Yes | No | No |
| Inspector | Yes | No | No |
| Detective | No | Yes | No |
| Security Hub | No | No | Yes |

---

## Security Focus Areas

| Service | Primary Focus |
|--------|---------------|
| GuardDuty | Behavioral threat detection |
| Macie | Sensitive data discovery |
| Inspector | Vulnerability management |
| Detective | Incident investigation |
| WAF | Application-layer protection |
| Shield | Network-layer DDoS protection |
| CloudTrail | API audit logging |
| KMS | Encryption key management |

---

# Layered Security Model

AWS security services operate across layers.

Preventive controls include:

- IAM
- Service Control Policies
- WAF
- Shield

Detective controls include:

- GuardDuty
- Inspector
- Macie

Investigative tools include:

- Detective

Aggregation and governance layer:

- Security Hub

Audit and visibility layer:

- CloudTrail

Encryption and key management layer:

- KMS

Understanding these layers clarifies how security services complement each other.

---

# Common Pitfalls

- Expecting GuardDuty to block or mitigate attacks (it provides threat detection only)  
- Assuming Security Hub performs vulnerability scanning  
- Confusing vulnerability scanning with behavioral threat detection  
- Using AWS WAF when infrastructure-level DDoS protection is required  
- Treating CloudTrail as a threat detection system instead of an audit log service  
- Assuming Amazon Macie scans services beyond S3  
- Forgetting the integration between GuardDuty, Detective, and Security Hub  
- Ignoring KMS key policy requirements when using SSE-KMS  
- Assuming AWS Shield protects against application-layer exploits

---

# Design Considerations

Security architecture decisions typically align with:

- Whether the goal is prevention, detection, investigation, or aggregation  
- Whether protection is needed at network or application layer  
- Whether the concern is data exposure or runtime compromise  
- Whether centralized multi-account visibility is required  
- Whether encryption control must be customer-managed  

Clarity usually comes from identifying the security layer where protection or visibility is required.
