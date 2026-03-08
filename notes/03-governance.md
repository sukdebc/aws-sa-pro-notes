# Governance, Organizations & SCP Notes (SAP-C02 Level)

AWS Organizations • Service Control Policies • Permission Boundaries • CloudTrail • Config • Control Tower • Access Analyzer

These notes summarize governance models, multi-account structures, layered permission controls, and enterprise guardrail patterns used in large-scale AWS environments.

The emphasis is on:

- Permission intersection logic  
- Preventive vs detective controls  
- Centralized visibility  
- Delegated administration  
- Enterprise-scale governance patterns  

---

# 1️⃣ Service Control Policies (SCPs)

**Documentation:**  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html

Service Control Policies (SCPs) operate at the AWS Organizations level and define the **maximum available permissions** for member accounts.

SCPs establish a permission ceiling — they do not grant permissions.

---

## Scope of Application

SCPs can be attached to:

- Organization Root  
- Organizational Units (OUs)  
- Individual member accounts  

All IAM users and roles within member accounts are affected.

SCPs apply to:

- IAM users  
- IAM roles  
- Federated identities  
- Root user (in member accounts)  

SCPs do NOT apply to:

- The management (payer) account  
- External accounts outside the organization  

---

## Key Characteristics

- SCPs **do not grant permissions**  
- They define a maximum permission boundary  
- Explicit `Deny` in SCP overrides IAM `Allow`  
- Evaluated before IAM identity policies  
- Cannot directly modify resource policies  

---

> **EXAM TIP**  
> Admin role blocked across multiple accounts?  
> Often an SCP at OU or Root level.

---

## Common SCP Design Patterns

- Deny disabling CloudTrail or GuardDuty  
- Deny actions outside approved regions  
- Deny IAM privilege escalation actions  
- Enforce encryption standards  

SCPs are typically used for **guardrails**, not granular access control.

---

# 2️⃣ Permission Boundaries

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html

Permission boundaries operate at the IAM identity level within a single account.

They define the **maximum permissions an identity policy can grant**.

---

## Evaluation Model

Effective permissions = `SCP ∩ Permission Boundary ∩ Identity Policy ∩ Session Policy ∩ Resource Policy`


Permission boundaries:

- Do not grant permissions  
- Restrict identity-based policies  
- Do not affect the root user  

---

## Typical Use Case

Delegated role creation:

- Developers can create IAM roles  
- Permission boundary prevents privilege escalation  
- Roles cannot exceed defined maximum scope  

---

> **EXAM TIP**  
> Scenario: “Allow teams to create roles but prevent admin access.”  
> → Permission Boundary.

---

# 3️⃣ AWS Organizations

**Documentation:**  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html

AWS Organizations enables centralized governance and multi-account management.

---

## Core Capabilities

- Consolidated billing  
- SCP enforcement  
- Tag policies  
- Delegated administrator support  
- Organization-level CloudTrail  
- Centralized security services  

---

## Enterprise OU Structure Example

- Root  
  - Security OU  
  - Production OU  
  - Development OU  
  - Sandbox OU  

Segmentation supports:

- Blast radius reduction  
- Environment isolation  
- Policy differentiation  
- Compliance tiering  

---

## Delegated Administrator Model

Security services (GuardDuty, Config, Security Hub, Access Analyzer) can be centrally managed from a designated security account.

---

> **EXAM TIP**  
> Enterprise-wide governance + centralized security visibility  
> → Organizations + delegated admin pattern.

---

# 4️⃣ CloudTrail (Organization Trail)

**Documentation:**  
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html

Organization Trails capture API activity across all member accounts.

---

## Characteristics

- Logs activity from all accounts  
- Stored in centralized S3 bucket  
- Member accounts cannot disable it  
- Supports multi-region logging  

---

## Integration with Security Services

CloudTrail feeds:

- GuardDuty  
- Detective  
- Security Hub  
- SIEM tools  

---

> **EXAM TIP**  
> Requirement: “Security team must monitor activity across all accounts.”  
> → Organization Trail.

---

# 5️⃣ Tag Policies

**Documentation:**  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html

Tag policies standardize tagging across accounts.

They define:

- Approved tag keys  
- Allowed tag values  
- Case enforcement  

They do not automatically remediate resources.

Automation via Config + Lambda/SSM is required for enforcement.

---

> **EXAM TIP**  
> Tag standardization → Tag Policy  
> Tag enforcement → Config + automation

---

# 6️⃣ AWS Config

**Documentation:**  
https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html

AWS Config provides visibility into configuration state and compliance posture.

---

## Capabilities

- Track resource configuration history  
- Evaluate compliance via rules  
- Detect drift  
- Support organization-wide aggregators  
- Enable conformance packs  

---

## Automated Remediation Pattern

Config Rule → EventBridge → SSM Automation / Lambda

Used for:

- Enforcing encryption  
- Preventing public S3 buckets  
- Ensuring logging remains enabled  

---

> **EXAM TIP**  
> Detect non-compliance → Config  
> Auto-remediate → Config + SSM Automation

---

# 7️⃣ IAM Access Analyzer (Organization Scope)

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html

Access Analyzer evaluates resource policies for unintended exposure.

---

## Identifies

- Public access  
- Cross-account access  
- External principals  
- Risky trust policies  

Can operate at account or organization level.

---

> **EXAM TIP**  
> Access Analyzer detects exposure.  
> It does not enforce access control.

---

# 8️⃣ AWS Control Tower

**Documentation:**  
https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html

Control Tower builds on Organizations and Config to provide structured landing zones.

---

## Guardrail Types

- **Preventive** → Implemented via SCP  
- **Detective** → Implemented via Config rules  

---

## What Control Tower Provides

- Pre-configured multi-account structure  
- Centralized logging  
- Baseline security configuration  
- Governance dashboard  
- Identity integration  

---

> **EXAM TIP**  
> “Quickly establish secure, compliant multi-account environment.”  
> → Control Tower.

---

# 9️⃣ Effective Permission Evaluation Model

Layered evaluation:

1. SCP  
2. Permission Boundary (if present)  
3. IAM Identity-based Policy  
4. Resource-based Policy  
5. Explicit Deny anywhere → Denied  

Effective permission is always the intersection.

Understanding this layered model is critical for complex SAP-C02 governance scenarios.

---

# Comparison Summary

## SCP vs IAM vs Permission Boundary

| Dimension | SCP | IAM Policy | Permission Boundary |
|------------|------|------------|---------------------|
| Grants Permissions | No | Yes | No |
| Restricts Maximum Scope | Yes | No | Yes |
| Scope | Organization | Identity | Identity |
| Applies Across Accounts | Yes | No | No |
| Evaluated Before IAM | Yes | N/A | Inside IAM flow |
| Affects Root User | Yes (member accounts) | Yes | No |

---

# Governance Tool Roles

| Tool | Primary Function |
|-------|------------------|
| SCP | Define maximum allowed actions across accounts |
| IAM | Grant permissions to identities |
| Permission Boundary | Restrict identity permission scope |
| Tag Policy | Standardize tagging |
| AWS Config | Detect configuration drift |
| SSM Automation | Remediate configuration issues |
| Access Analyzer | Detect unintended exposure |
| CloudTrail | Audit API activity |
| Control Tower | Enterprise landing zone governance |

---

# Common Governance Pitfalls

- Assuming SCPs grant permissions  
- Forgetting explicit Deny precedence  
- Confusing Permission Boundaries with SCPs  
- Expecting Tag Policies to auto-remediate  
- Overlooking Organization Trails  
- Assuming AssumeRole bypasses SCP  
- Forgetting SCP does not apply to management account  
- Misunderstanding layered permission intersection logic  

---
# Design Considerations

Governance decisions become clearer when evaluated across:

- Scope (organization vs account vs identity)  
- Preventive vs detective enforcement  
- Centralized visibility requirements  
- Delegated control models  
- Compliance and audit readiness  
- Blast radius containment  

Enterprise governance in AWS is fundamentally about layered control — not single policies.

