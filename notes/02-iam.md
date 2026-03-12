# AWS IAM Notes

IAM • STS • SCP • Permission Boundaries • Cross-Account Access • KMS • Session Policies • IRSA • Access Analyzer

These notes consolidate identity architecture, cross-account design, permission evaluation logic, and governance layering used in enterprise AWS environments.

---

## IAM Policy Evaluation Logic

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html  

Understanding evaluation order is critical for resolving complex access scenarios.

### Evaluation Model

Explicit Deny in any policy → Always Deny.

Permissions are evaluated across multiple policy layers:

- Service Control Policy (SCP) defines the maximum permissions available in the account
- Permission Boundaries limit the permissions an IAM identity can receive
- Session Policies can further restrict temporary role sessions
- Identity-based policies grant permissions to users or roles
- Resource-based policies can also grant permissions (if supported)

Effective permission is the **intersection of all applicable policy types**.

SCP ∩ Permission Boundary ∩ Identity Policy ∩ Session Policy ∩ Resource Policy

If any layer denies → final decision is Deny.

> **EXAM TIP**  
> If something “should work” but doesn’t →  
> Check SCP → Permission Boundary → Session Policy → Resource Policy.  
> Any explicit Deny anywhere wins.

---

## Cross-Account Access Pattern

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role.html  

Cross-account access requires three components:

### Caller Account
- IAM policy allowing `sts:AssumeRole`

### Target Account
- Role trust policy trusting caller principal  
- Role permission policy granting service actions  

All three must exist.

Missing any component results in `AccessDenied`.

> **EXAM TIP**  
> Cross-account failure? Check:  
> 1️⃣ Caller IAM policy  
> 2️⃣ Trust policy  
> 3️⃣ Target role permissions  
> 4️⃣ SCP restrictions  

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
Access can be granted through identity policies, resource policies, or both,
depending on the service and access pattern. Resource policies can grant access even if the caller identity policy doesn’t explicitly allow it. Example: cross-account S3 bucket access.

Trust policy is not required in this case.

Lambda invocation requires a resource-based policy.

CloudWatch Logs doesn't require it, but SNS/SQS/S3 do. This distinction matters in exam scenarios

> **EXAM TIP**  
> Lambda invoke permission = Resource policy.  
> Not trust policy.

---

## KMS Cross-Account Rules

**Documentation:**  
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

Cause: Key policy missing permission.

> **EXAM TIP**  
> KMS = IAM ∩ Key Policy.  
> If encryption fails, suspect key policy first.

---

### KMS Grants (Advanced)

**Documentation:**  
https://docs.aws.amazon.com/kms/latest/developerguide/grants.html  

KMS Grants allow temporary delegated permissions without modifying key policy.

Often used by:
- EBS  
- RDS  
- Lambda  

> **EXAM TIP**  
> Many AWS services use KMS Grants automatically — not visible in IAM policies.

---

## Permission Boundaries

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html  

Permission boundaries define the maximum permissions an identity can receive.

Permission boundaries:
- Do not grant permissions  
- Restrict identity-based policies  
- Do not affect root user  

> **EXAM TIP**  
> Delegated admin scenario → Think Permission Boundary.

---

## Service Control Policies (SCP)

**Documentation:**  
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html  

Applied at:
- Organization root  
- OU  
- Account level  

SCP:
- Defines maximum allowed actions  
- Cannot grant permissions  
- SCPs **apply** to all IAM identities in member accounts, including the **root user**.
- Explicit Deny overrides all  

SCP does not apply to:
- The management account root user
- Resource policies directly  
- External accounts outside the organization  

> **EXAM TIP**  
> Admin role blocked?  
> Very often → SCP. 
> Explicit denies in SCPs override all IAM permissions, including the root user of member accounts. 

---

## Session Policies

**Documentation:**  
https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html  

Used in:
- `AssumeRole`  
- `AssumeRoleWithSAML`  
- `AssumeRoleWithWebIdentity`  

Session policies:
- Restrict permissions further  
- Cannot grant new permissions  

> **EXAM TIP**  
> “Admin role but access denied” → Check session policy.

---

## IAM Condition Keys

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html  

Condition keys define advanced access control.

### Common Global Keys

| Condition Key | Purpose |
|---------------|---------|
| `aws:SourceIp` | Restrict by IP |
| `aws:PrincipalArn` | Restrict identity |
| `aws:PrincipalOrgID` | Restrict to AWS Organization |
| `aws:MultiFactorAuthPresent` | Require MFA |
| `aws:SourceVpc` | Restrict to VPC |
| `aws:SourceVpce` | Restrict to VPC endpoint |

---

### Service-Specific Examples

- `s3:prefix`  
- `kms:ViaService`  

> **EXAM TIP**  
> Organization-wide access restriction → `aws:PrincipalOrgID`.

---

## Service-Linked Roles

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html  

Service-linked roles:
- Created automatically  
- Managed by AWS  
- Required for service operation  

Examples:
- Auto Scaling  
- ECS  
- EKS  
- GuardDuty  

> **EXAM TIP**  
> Service-linked roles are AWS-managed — not typical IAM roles.

---

## IRSA (IAM Roles for Service Accounts – EKS)

**Documentation:**  
https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html  

IRSA enables per-pod IAM permissions.

Requirements:
- OIDC provider configured  
- Trust policy references OIDC provider  
- Condition includes service account subject  

> **EXAM TIP**  
> Least privilege in EKS → Use IRSA, not node IAM role.

---

## IAM Access Analyzer

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html  

Detects:
- Public access  
- Cross-account access  
- External principals in trust policies  

> **EXAM TIP**  
> Access Analyzer detects exposure.  
> It does not block access.

---

## STS Behavior

**Documentation:**  
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html  

STS:

- Public service endpoint
- Private subnets require **NAT Gateway** to reach the public STS endpoint
- Alternatively, create an **Interface VPC Endpoint (PrivateLink) for STS**
- Supports **regional endpoints** for improved availability and reduced latency

> **EXAM TIP**  
> Private subnet + AssumeRole failure →  
> Often missing NAT Gateway or STS Interface Endpoint.
---

## S3 Block Public Access Interaction

**Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html  

Block Public Access:
- Overrides bucket policy allowing public access  
- Enforced at account or bucket level  

> **EXAM TIP**  
> Bucket policy allows public but access denied → Check Block Public Access.

---

## IAM Identity Center (Enterprise Context)

**Documentation:**  
https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html  

IAM Identity Center:
- Centralized workforce access  
- Integrates with Organizations  
- Uses permission sets  

> **EXAM TIP**  
> Enterprise multi-account workforce access → IAM Identity Center.

---

## Common IAM Pitfalls

- Forgetting that explicit Deny overrides all allow permissions  
- Missing trust policy in cross-account role assumption  
- Ignoring KMS key policy requirements for encryption operations  
- Overlooking permission boundaries in delegated role creation  
- Assuming Lambda invocation uses a trust policy (it requires a resource-based policy)  
- Forgetting STS calls from private subnets require NAT or an STS interface endpoint  

---
# Design Considerations

IAM design becomes clearer when evaluated across:

- Identity layer  
- Resource layer  
- Organizational governance  
- Session restriction  
- Key management enforcement  