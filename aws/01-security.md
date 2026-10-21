# Domain 1: Design Secure Architectures (30%)

This domain focuses on **access control, network security, and data protection**. The exam weight (30%) reflects that security is foundational; every architecture starts with "who can access what."

---

## Focus Areas

**Identity & Access (IAM, STS, cross-account, federation)** ← Exam-critical
- Identity vs resource policies (both needed for cross-account access)
- Roles vs users vs groups; when to use each
- STS for temporary credentials
- Cross-account role assumption
- Permission boundaries (delegate admin without giving all permissions)
- IAM Identity Center (federated access, multi-account)
- Organizations & Service Control Policies (SCPs) — prevent risky actions org-wide

**Network Security (VPC, security groups, NACLs)**
- Security groups (stateful, per-instance) vs NACLs (stateless, per-subnet)
- Subnets, NAT gateway (egress only, not for inbound), NAT instances
- VPC endpoints (PrivateLink) — access AWS services without internet
- VPC Flow Logs, GuardDuty

**Application & Data Protection**
- WAF (Layer 7, web app firewall) vs Shield (DDoS, freemium)
- Secrets Manager (rotation, versioning) vs Parameter Store (free, no rotation)
- Cognito (app user identity) vs IAM Identity Center (org federation)
- KMS (key management, encryption at rest), key policies, key rotation
- ACM (TLS certificates, encryption in transit), auto-renewal
- Macie (sensitive data discovery in S3), GuardDuty (threat detection)

---

## Documentation Links

### IAM in Depth
- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) — critical for multi-policy scenarios
- [STS API Reference](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)
- [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [Organizations & SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html)

### Network Security
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

### Encryption & Secrets
- [KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- [Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)

### Application Protection
- [WAF](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html)
- [Shield](https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html)

---

## Key Decision Points

### "Cross-account access needed — which approach?"
**Use IAM role assumption** (role in target account, trust policy in role). The principal assumes a role in the other account, receives temporary credentials via STS, and acts with role permissions. Compare with resource-based policies — both are correct, but role assumption is the standard pattern for cross-account, especially with multiple principals.

### "Should I use Secrets Manager or Parameter Store?"
**Secrets Manager** if secrets must rotate (database passwords, API keys) or you need versioning/audit trail.
**Parameter Store** for configuration values and secrets that don't rotate; free tier is generous. AWS charges per secret for Secrets Manager.

### "WAF or Shield? Do I need both?"
**Shield Standard** (free, automatic) defends against Layer 3/4 DDoS.
**Shield Advanced** ($3K/month) adds Layer 7 DDoS protection and 24/7 DDoS Response Team.
**WAF** is a separate Layer 7 firewall; deploy it if you need app-level protection (SQL injection, XSS, rate limiting). Shield Advanced + WAF is the enterprise option.

### "Permission boundary — when does it appear in exams?"
Permission boundaries limit what a role/user can do, even if granted broader permissions elsewhere. Exam scenario: "I need to delegate permissions to a team lead, but ensure they can't access the database." Answer: Attach a permission boundary that excludes RDS actions.

---

## Exam Scenarios

**Scenario 1: Cross-account access for a third-party vendor**
Create a role in your account with least-privilege permissions for that vendor's specific tasks. Add an external ID (secret) in the trust policy to prevent role hijacking. Vendor assumes the role from their account, receives temporary STS credentials, and operates within role permissions.

**Scenario 2: Encrypt data in S3 with customer-managed KMS keys**
Create a KMS key, configure the key policy to allow the principal to use the key (not just the key manager). Upload objects with `x-amz-server-side-encryption: aws:kms` and the key ID. Verify in KMS console that the key policy permits the principal; if not, the operation silently fails or returns a key access error.

**Scenario 3: Federate with on-prem Active Directory**
Use IAM Identity Center connected to your AD via AWS Directory Service, or use SAML federation with a third-party IdP. IAM Identity Center is the modern approach; it handles multi-account SSO and permission assignment. Exam question: "How to give on-prem users access to AWS without creating IAM users?" Answer: Identity Center + AD Connector (or AD Sync).

---

## Common Exam Pitfalls

- **Confusing security groups and NACLs:** Security groups are stateful (no need to allow return traffic); NACLs are stateless (must explicitly allow return traffic). Both must allow the traffic for it to flow.
- **Forgetting KMS key policy:** KMS key policy is separate from IAM policy; the principal needs permission in *both* places to use the key.
- **Misunderstanding "least privilege":** The exam tests whether you'll grant only the specific actions needed (e.g., `s3:GetObject` not `s3:*`).
- **Resource-based vs identity-based policies:** Cross-account access requires *both* a trust relationship (resource policy) and permission (identity policy).

---

**Next:** [Domain 2 — Resilience](./02-resilience.md)
