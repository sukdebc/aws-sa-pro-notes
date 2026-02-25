# Governance, Organizations & SCP Notes (SAP-C02 Level)

AWS Organizations • Service Control Policies • Permission Boundaries • CloudTrail • Config • Control Tower

These notes summarize governance models, multi-account structures, and permission boundaries across AWS Organizations.

The focus is on understanding effective permission evaluation, scope of control, layered guardrails, and enterprise-level governance patterns.

---

# 1️⃣ Service Control Policies (SCPs)

📘 Official Documentation:  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html

Service Control Policies (SCPs) operate at the AWS Organizations level and define the **maximum available permissions** for member accounts.

---

## Scope of Application

SCPs can be attached to:

- Organization Root
- Organizational Units (OUs)
- Individual member accounts

SCPs define the upper boundary of what identities within those accounts are allowed to perform.

---

## Key Characteristics

- SCPs **do not grant permissions**.
- They define a maximum permission boundary.
- Explicit `Deny` in an SCP overrides `Allow` in IAM.
- They apply to all IAM users and roles in member accounts.
- They **do not apply to the management (payer) account**.
- They do not directly modify resource policies.

Effective permissions = **Intersection of IAM policies and SCPs**.

If either layer denies an action, the request is denied.

---

# 2️⃣ Permission Boundaries

📘 Official Documentation:  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html

Permission boundaries operate within a single account and restrict what IAM identities (users or roles) can perform.

They are commonly used in delegated administration scenarios.

---

## Conceptual Comparison

| Feature | SCP | Permission Boundary |
|----------|------|---------------------|
| Scope | Organization-wide | Single account |
| Applies To | Accounts | IAM identities |
| Restricts Root User | Yes (member accounts) | No |
| Grants Permissions | No | No |
| Evaluation Layer | Outside IAM | Inside IAM |

Permission boundaries limit the maximum permissions an identity-based policy can grant.

---

# 3️⃣ AWS Organizations

📘 Official Documentation:  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html

AWS Organizations enables centralized governance and multi-account management.

## Core Capabilities

- Consolidated billing
- SCP enforcement
- Tag policies
- Delegated administrator support
- Organization-level CloudTrail
- Centralized security service management

Typical enterprise OU structure:

- Security OU
- Production OU
- Development OU
- Sandbox OU

Account separation reduces blast radius and supports least-privilege governance.

---

# 4️⃣ CloudTrail (Organization Trail)

📘 Official Documentation:  
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html

An Organization Trail enables centralized logging across all member accounts.

## Characteristics

- Captures activity from all accounts
- Logs stored in centralized S3 bucket
- Member accounts cannot disable it
- Required for centralized security visibility

Often integrated with:

- GuardDuty
- Security Hub
- Detective

Organization Trails improve audit consistency and forensic capability.

---

# 5️⃣ Tag Policies

📘 Official Documentation:  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html

Tag policies standardize tagging across the organization.

They define:

- Allowed tag keys
- Allowed values
- Case sensitivity enforcement

Tag policies **do not enforce remediation automatically**.  
They ensure tagging consistency but require AWS Config or automation for enforcement workflows.

---

# 6️⃣ AWS Config

📘 Official Documentation:  
https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html

AWS Config provides visibility into resource configuration and compliance state.

## Capabilities

- Detect configuration drift
- Evaluate compliance using rules
- Track resource history
- Support organization-wide aggregators

When combined with:

- Systems Manager Automation
- EventBridge
- Lambda

It can enable automated remediation.

---

# 7️⃣ IAM Access Analyzer (Organization Scope)

📘 Official Documentation:  
https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html

IAM Access Analyzer evaluates resource policies to detect unintended access.

## It Identifies:

- Public exposure
- Cross-account access
- External principals
- Risky trust relationships

Organization-wide analyzers provide centralized visibility across accounts.

---

# 8️⃣ AWS Control Tower

📘 Official Documentation:  
https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html

AWS Control Tower builds on Organizations and Config to provide structured enterprise governance.

## Guardrail Types

- **Preventive** – Implemented via SCPs
- **Detective** – Implemented via AWS Config rules

Control Tower provides an opinionated landing zone with pre-configured governance patterns.

---

# 9️⃣ Effective Permission Evaluation Model

Understanding layered evaluation is critical for SAP-C02.

Permission evaluation generally follows this logic:

1. SCP evaluation
2. Permission Boundary (if present)
3. IAM Identity-based Policy
4. Resource-based Policy
5. Explicit Deny anywhere → Denied

The effective permission is the intersection of all applicable layers.

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

## Governance Tool Roles

| Tool | Primary Function |
|-------|------------------|
| SCP | Define maximum allowed actions across accounts |
| IAM | Grant permissions to identities |
| Permission Boundary | Restrict identity permission scope |
| Tag Policy | Standardize tagging |
| AWS Config | Detect configuration drift |
| SSM Automation | Remediate configuration issues |
| Access Analyzer | Detect unintended access exposure |
| CloudTrail | Audit API activity |
| Control Tower | Structured landing zone governance |

---

# Common Pitfalls in Governance Scenarios

- Assuming SCPs grant permissions.
- Forgetting explicit Deny in SCP overrides IAM Allow.
- Confusing Permission Boundaries with SCPs.
- Assuming Tag Policies enforce remediation automatically.
- Overlooking Organization Trails for centralized logging.
- Assuming STS AssumeRole can bypass SCP restrictions.
- Forgetting SCPs do not apply to the management account.
- Misunderstanding the layered permission intersection model.

Governance design becomes clearer when evaluated across:

- Scope (organization vs account vs identity)
- Enforcement layer (preventive vs detective)
- Permission intersection logic
- Audit and compliance visibility
- Blast radius control

Understanding these dimensions clarifies how layered controls interact in enterprise AWS environments.