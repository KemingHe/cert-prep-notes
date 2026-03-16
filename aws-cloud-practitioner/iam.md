# Study Notes - AWS Identity and Access Management (IAM)

> **Last Updated**: 2026-03-16 by Keming He

## Table of Contents

- [Study Notes - AWS Identity and Access Management (IAM)](#study-notes---aws-identity-and-access-management-iam)
  - [Table of Contents](#table-of-contents)
  - [Identity Components](#identity-components)
  - [Access Methods](#access-methods)
  - [Shared Responsibility Model](#shared-responsibility-model)
    - [AWS Responsibilities](#aws-responsibilities)
    - [Customer Responsibilities](#customer-responsibilities)
  - [Audit Tools](#audit-tools)
  - [Best Practices](#best-practices)
  - [References](#references)

## Identity Components

[IAM is a global service](https://docs.aws.amazon.com/general/latest/gr/iam-service.html), not tied to any specific AWS region.

- **[Root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html)**: One per AWS account with full, unrestricted access
  - Use only for [specific root-only tasks](https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-tasks.html) such as closing the account, changing account settings, restoring IAM permissions, activating billing access, configuring S3 MFA delete, or fixing S3/SQS policies that deny all principals
  - Can set an [account-level password policy for IAM users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html), but this policy does NOT apply to the root user password itself; avoid using root for any daily operations
- **[IAM users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)**: Identity representing a person or application with long-term credentials (password, access keys)
  - AWS recommends using federation with temporary credentials for human users instead of creating IAM users
  - Can belong to zero or more [groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html); policies from all groups are evaluated together according to [AWS policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- **Groups**: A collection of IAM users; policies attached to a group apply to all members
  - [Groups cannot contain other groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html)
- **[Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)**: JSON documents defining Allow or Deny permissions for AWS services and resources
  - Can be attached to users, groups, or roles; always follow least privilege (grant only what is needed)
- **[Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)**: Temporary permissions assumed by AWS services (e.g., EC2, Lambda) or federated users
  - Preferred over long-term access keys for granting permissions to AWS services

> [↑ Back to Table of Contents](#table-of-contents)

---

## Access Methods

- **[AWS Management Console](https://docs.aws.amazon.com/signin/latest/userguide/how-to-sign-in.html)**: Browser-based; authenticate with username, password, and optional MFA
- **AWS CLI**: Command-line tool with multiple [authentication methods](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html)
  - [Long-lived access keys](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html) using `aws configure` (not recommended by AWS)
  - [Keyless browser-based login](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html) via `aws login` (recommended for standard IAM users, requires CLI v2.32.0+)
  - [IAM Identity Center/SSO](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html) via `aws sso login` for centralized enterprise access management

> [↑ Back to Table of Contents](#table-of-contents)

---

## Shared Responsibility Model

[AWS and customers share security responsibilities](https://aws.amazon.com/compliance/shared-responsibility-model/) for IAM.

### AWS Responsibilities

- Maintains the [global IAM infrastructure](https://docs.aws.amazon.com/IAM/latest/UserGuide/security.html), underlying network security, and vulnerability analysis through third-party audits
- Handles [compliance validation](https://aws.amazon.com/compliance/shared-responsibility-model/) and service-level agreements

### Customer Responsibilities

- Enforce [multi-factor authentication (MFA)](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html), set [strong password policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html), and [rotate access keys regularly](https://docs.aws.amazon.com/wellarchitected/latest/framework/sec_identities_audit.html) (maximum 90-day interval)
- Apply [least-privilege permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) and review access patterns to prevent privilege creep
- Manage users, groups, roles, and policies throughout their lifecycle

> [↑ Back to Table of Contents](#table-of-contents)

---

## Audit Tools

- **[IAM Access Advisor](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor.html)**: Shows [service access information](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor-view-data.html) for IAM identities (users, groups, roles, or policies), including when each service was last accessed; use this to identify and revoke unused permissions (least privilege)
  - Source policy information requires using the `ListPoliciesGrantingServiceAccess` API/CLI command (not shown in basic console view)
- **[IAM Credentials Report](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html)**: Account-level CSV export listing all IAM users and the status of their passwords, access keys, and MFA devices

> [↑ Back to Table of Contents](#table-of-contents)

---

## Best Practices

AWS [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) emphasize temporary credentials and federation:

- Avoid using the root account for daily tasks; protect root user credentials and minimize root usage
- **For human users**: [Require federation with an identity provider](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) to access AWS using temporary credentials; use [AWS IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) for centralized workforce access management instead of creating individual IAM users
- **For workloads**: [Require temporary credentials with IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) to access AWS rather than embedding long-term access keys
- Enforce [phishing-resistant MFA](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) (passkeys, security keys) and [strong password policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html)
- When IAM users are unavoidable (legacy systems, third-party tools), assign users to groups and attach policies to groups for scalable permission management
- Never share IAM users or access keys between people or applications

> [↑ Back to Table of Contents](#table-of-contents)

---

## References

- [AWS IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) - comprehensive IAM documentation
- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/) - security responsibilities framework
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/framework/security.html) - architectural security guidance
- [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) - how AWS evaluates permissions

> [↑ Back to Table of Contents](#table-of-contents)
