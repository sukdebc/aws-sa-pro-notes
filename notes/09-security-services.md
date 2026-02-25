# AWS Security Services Notes (SAP-C02 Level)

GuardDuty • Macie • Inspector • Detective • Security Hub • WAF • Shield • IAM Access Analyzer • CloudTrail • KMS

These notes summarize AWS-native threat detection, compliance posture management, vulnerability scanning, investigation tooling, encryption controls, and edge protection services.

The focus is on understanding service roles, scope boundaries, and how detection, prevention, investigation, and aggregation layers interact.

---

# 1️⃣ Amazon GuardDuty

📘 Docs:  
https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html

GuardDuty provides continuous threat detection across AWS accounts and workloads.

It analyzes:

- VPC Flow Logs
- DNS logs
- CloudTrail management events
- EKS audit logs
- S3 data events (optional)
- RDS login activity (optional)

## Detects

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
- Integrates with Security Hub, EventBridge, and automation workflows

It is a **detection service**, not a prevention service.

---

# 2️⃣ Amazon Macie

📘 Docs:  
https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html

Macie discovers and classifies sensitive data stored in Amazon S3.

## Detects

- Personally Identifiable Information (PII)
- Financial data
- Credentials
- API keys
- Sensitive logs

Macie:

- Operates only on S3
- Performs classification and risk scoring
- Does not encrypt or remediate automatically

It is focused on **data security posture**, not runtime security.

---

# 3️⃣ Amazon Inspector

📘 Docs:  
https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html

Inspector provides automated vulnerability management.

## Scans

- EC2 instances (software vulnerabilities)
- ECR container images (CVE detection)
- Lambda functions (dependency vulnerabilities)

Inspector:

- Relies on SSM agent for EC2 scanning
- Continuously evaluates new CVEs
- Focuses on vulnerability detection, not anomaly detection

Inspector answers: *“Is this workload vulnerable?”*

---

# 4️⃣ Amazon Detective

📘 Docs:  
https://docs.aws.amazon.com/detective/latest/userguide/what-is-detective.html

Detective supports post-incident investigation.

It builds graph-based relationships across:

- CloudTrail logs
- VPC Flow Logs
- GuardDuty findings
- IAM activity patterns

Detective enables:

- Timeline reconstruction
- Entity behavior analysis
- Root cause investigation

Detective is investigative, not preventative.

---

# 5️⃣ AWS Security Hub

📘 Docs:  
https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html

Security Hub aggregates findings from:

- GuardDuty
- Inspector
- Macie
- IAM Access Analyzer
- AWS Foundational Security Best Practices
- CIS benchmarks
- PCI controls

Security Hub:

- Centralizes security posture visibility
- Normalizes findings into a common format
- Supports cross-account aggregation

It does not perform scanning itself.

---

# 6️⃣ AWS WAF

📘 Docs:  
https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html

AWS WAF protects HTTP/HTTPS workloads at **Layer 7**.

## Protects Against

- SQL injection
- Cross-site scripting (XSS)
- Malicious bots
- Rate-based abuse
- Geographic filtering

## Integrations

- CloudFront
- Application Load Balancer
- API Gateway
- AppSync

WAF performs application-layer filtering.

---

# 7️⃣ AWS Shield

📘 Docs:  
https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html

Shield provides DDoS protection at **Layer 3/4**.

## Shield Standard

- Automatically enabled
- Protects against common network attacks

## Shield Advanced

- Enhanced detection
- 24/7 DDoS Response Team
- Cost protection during DDoS events

Shield mitigates volumetric attacks.

---

## WAF vs Shield Comparison

| Dimension | WAF | Shield |
|------------|------|--------|
| OSI Layer | Layer 7 | Layer 3/4 |
| Protects Against | App-layer attacks | DDoS attacks |
| HTTP Filtering | Yes | No |
| Rate Limiting | Yes | Infrastructure-level |
| Cost Protection | No | Yes (Advanced) |

WAF filters malicious requests.  
Shield protects against volumetric DDoS.

---

# 8️⃣ IAM Access Analyzer

📘 Docs:  
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

It does not automatically block access.

---

# 9️⃣ AWS CloudTrail

📘 Docs:  
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html

CloudTrail records API activity across AWS services.

## Key Considerations

- Enable multi-region trails
- Use Organization Trail for centralized logging
- CloudTrail Lake supports SQL-based log analysis

CloudTrail provides audit visibility but does not detect threats independently.

It is a data source for GuardDuty and Detective.

---

# 🔟 AWS KMS (Key Management Service)

📘 Docs:  
https://docs.aws.amazon.com/kms/latest/developerguide/overview.html

KMS manages encryption keys for AWS services.

## Key Types

- AWS-managed keys
- Customer-managed keys (CMKs)
- Custom key stores (CloudHSM-backed)

## Permission Model

- IAM policies
- Key policies
- Grants

Key policies must allow access even if IAM allows it.

KMS keys are region-scoped.

Automatic rotation:

- Available for supported customer-managed symmetric keys (yearly)

---

# 1️⃣1️⃣ Security Services Comparison

## Detection vs Investigation vs Aggregation

| Service | Detection | Investigation | Aggregation |
|-----------|------------|--------------|-------------|
| GuardDuty | Yes | No | No |
| Macie | Yes (Data classification) | No | No |
| Inspector | Yes (Vulnerabilities) | No | No |
| Detective | No | Yes | No |
| Security Hub | No | No | Yes |

---

## Data vs Runtime vs Network Protection

| Service | Focus Area |
|-----------|------------|
| Macie | Sensitive data discovery |
| Inspector | Vulnerability management |
| GuardDuty | Behavioral threat detection |
| Detective | Incident investigation |
| WAF | Application-layer filtering |
| Shield | Network-layer DDoS mitigation |
| KMS | Encryption key management |
| CloudTrail | Audit logging |

---

# 1️⃣2️⃣ Layered Security Model

Security services operate in layers:

1. **Preventive Controls** – IAM, SCPs, WAF, Shield
2. **Detective Controls** – GuardDuty, Inspector, Macie
3. **Investigative Controls** – Detective
4. **Aggregation & Governance** – Security Hub
5. **Audit & Visibility** – CloudTrail
6. **Encryption & Key Control** – KMS

Understanding these layers clarifies how services complement one another.

---

# Common Pitfalls in Security Architecture

- Expecting GuardDuty to block attacks.
- Assuming Security Hub performs scanning.
- Confusing vulnerability scanning (Inspector) with behavioral detection (GuardDuty).
- Using WAF when volumetric DDoS protection is required.
- Treating CloudTrail as real-time threat detection.
- Assuming Macie scans services beyond S3.
- Overlooking GuardDuty + Detective integration.
- Forgetting KMS key policy requirements.
- Assuming Shield protects against application-layer exploits.

Security decisions become clearer when asking:

- Is the goal detection, prevention, investigation, or aggregation?
- Is the protection needed at network or application layer?
- Is the concern data exposure or runtime compromise?
- Is centralized multi-account visibility required?

Understanding these dimensions clarifies how AWS security services interact in layered architectures.