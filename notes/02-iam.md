# AWS IAM Notes (SAP-C02 Level)

IAM • STS • SCP • Permission Boundaries • Cross-Account Access • KMS • Session Policies • IRSA • Access Analyzer

These notes consolidate identity architecture, cross-account design, permission evaluation logic, and governance layering used in enterprise AWS environments.

---

## IAM Policy Evaluation Logic

**AWS Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html  

Understanding evaluation order is critical for resolving complex access scenarios.

### Evaluation Flow

1. Explicit Deny in any policy → Always Deny  
2. Service Control Policy (SCP) limits maximum permission  
3. Permission Boundary limits identity permissions  
4. Session Policy further restricts permissions  
5. Identity-based Policy grants permission  
6. Resource-based Policy grants permission (if applicable)  

Final effective permission is the **intersection** of all applicable policies.

If any layer denies → final decision is Deny.

---

## Cross-Account Access Pattern

**AssumeRole Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role.html  

Cross-account access requires three components:

### Caller Account

- IAM policy allowing:
  - `sts:AssumeRole`

### Target Account

- Role trust policy trusting caller principal  
- Role permission policy granting actual service actions  

All three must exist:

- Caller IAM policy  
- Trust policy  
- Role permissions  

Missing any component results in `AccessDenied`.

---

## Resource-Based vs Identity-Based Policies

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html  

### Identity-Based Policies

Attached to:

- Users  
- Roles  
- Groups  

Used for services that do not support resource policies (e.g., EC2, RDS).

---

### Resource-Based Policies

Supported by:

- S3  
- SNS  
- SQS  
- Lambda  
- KMS  
- API Gateway  
- EventBridge  

Pattern:

- Caller identity policy allows action  
- Resource policy allows caller ARN  

Trust policy is not required in this case.

Lambda invocation requires a resource-based policy, not a trust policy.

---

## KMS Cross-Account Rules

**KMS Key Policy Documentation:**  
https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html  

KMS enforces both:

- IAM policy  
- KMS key policy  

The key policy ultimately controls access.

For cross-account usage:

- Key policy must explicitly include the target role ARN  
- IAM policy alone is insufficient  

Common failure pattern:

- AssumeRole succeeds  
- S3 PutObject succeeds  
- KMS Encrypt fails  

Cause: Key policy missing target role permission.

---

### KMS Grants (Advanced)

**Documentation:**  
https://docs.aws.amazon.com/kms/latest/developerguide/grants.html  

KMS Grants allow temporary delegated permissions without modifying key policy.

Often used by:

- EBS  
- RDS  
- Lambda  

---

## Permission Boundaries

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html  

Permission boundaries define the maximum permissions an identity can receive.

Effective permission:  

SCP  
∩ Permission Boundary  
∩ Identity Policy  
∩ Session Policy  
∩ Resource Policy  

Permission boundaries:

- Do not grant permissions  
- Restrict identity-based policies  
- Do not affect root user  

---

## Service Control Policies (SCP)

**Documentation:**  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html  

Applied at:

- Organization root  
- Organizational Units (OU)  
- Account level  

SCP:

- Defines maximum allowed actions  
- Cannot grant permissions  
- Affects all identities including root user  
- Explicit Deny overrides all  

SCP does not apply to:

- Resource policies directly  
- External accounts outside the organization  

---

## Session Policies

**STS API Documentation:**  
https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html  

Used in:

- `AssumeRole`  
- `AssumeRoleWithSAML`  
- `AssumeRoleWithWebIdentity`  

Session policies:

- Restrict permissions further  
- Cannot grant new permissions  
- Often explain reduced access despite Admin role  

---

## Role Chaining

When assuming a role from another assumed role:

- Maximum session duration = 1 hour  
- Even if target role allows longer duration  

Role chaining can impact federation designs.

---

## IAM Condition Keys

**Global Condition Keys Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html  

Condition keys frequently define advanced access control.

### Common Global Keys

| Condition Key | Purpose |
|---------------|---------|
| `aws:SourceIp` | Restrict by IP |
| `aws:PrincipalArn` | Restrict specific identity |
| `aws:PrincipalOrgID` | Restrict to AWS Organization |
| `aws:MultiFactorAuthPresent` | Require MFA |
| `aws:SourceVpc` | Restrict to specific VPC |
| `aws:SourceVpce` | Restrict to specific VPC endpoint |

---

### Service-Specific Examples

- `s3:prefix` → Restrict object key prefixes  
- `kms:ViaService` → Restrict KMS use to specific service  

Conditions are often used to enforce least privilege without modifying architecture.

---

## Service-Linked Roles

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html  

Service-linked roles are:

- Created automatically by AWS services  
- Managed by AWS  
- Required for service operation  

Examples:

- Auto Scaling  
- ECS  
- EKS  
- GuardDuty  

These roles cannot be modified like normal IAM roles.

---

## IRSA (IAM Roles for Service Accounts – EKS)

**Documentation:**  
https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html  

IRSA allows fine-grained IAM permissions at the pod level.

Requirements:

- OIDC provider configured for cluster  
- IAM role trust policy references OIDC provider  
- Condition on service account subject  

IRSA replaces node-level IAM permissions and enables least-privilege Kubernetes design.

---

## IAM Access Analyzer

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html  

Access Analyzer:

- Detects unintended public or cross-account access  
- Evaluates resource-based policies  
- Works for S3, KMS, SQS, IAM roles, Lambda, and others  
- Can operate at account or organization level  

Useful for identifying external exposure.

---

## STS Behavior

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html  

Security Token Service (STS):

- Public endpoint  
- Requires NAT from private subnet (unless interface endpoint exists)  
- Supports regional endpoints  
- Regional endpoints improve latency and resilience  

STS is not VPC-scoped.

---

## S3 Block Public Access Interaction

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html  

Block Public Access:

- Overrides bucket policy allowing public access  
- Enforced at account or bucket level  
- Explicit deny behavior  

---

## IAM Identity Center (Enterprise Context)

**Documentation:**  
https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html  

IAM Identity Center:

- Centralized workforce access  
- Integrates with Organizations  
- Federates to multiple accounts  
- Supports permission sets  

Used in enterprise multi-account environments.

---

## Common IAM Pitfalls Observed

- Forgetting explicit Deny precedence  
- Missing trust policy in cross-account role  
- Ignoring KMS key policy requirements  
- Overlooking permission boundaries  
- Misconfiguring SCP at OU level  
- Assuming Lambda invoke uses trust policy  
- Forgetting STS requires internet/NAT  
- Ignoring role chaining duration limits  
- Missing condition keys in advanced restriction scenarios  

IAM design becomes clearer when evaluated across:

- Identity layer  
- Resource layer  
- Organizational governance  
- Session restriction  
- Key management enforcement  