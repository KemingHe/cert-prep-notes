# Study Notes - AWS Identity and Access Management (IAM)

> **Last Updated**: 2026-02-25 by Keming He

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

## Identity Components

IAM is a global service, not tied to any specific AWS region.

- **Root user**: One per AWS account with full, unrestricted access
  - Use only for initial setup and account-level tasks (e.g., closing the account, changing the support plan)
  - Can set the account-level password policy; avoid using for any daily operations
- **IAM users**: Represent a physical person with long-term credentials
  - Can belong to zero or more groups; permissions accumulate across all group memberships
- **Groups**: A collection of IAM users; policies attached to a group apply to all members
  - Groups cannot contain other groups
- **Policies**: JSON documents defining Allow or Deny permissions for AWS services and resources
  - Can be attached to users, groups, or roles; always follow least privilege (grant only what is needed)
- **Roles**: Temporary permissions assumed by AWS services (e.g., EC2, Lambda) or federated users
  - Preferred over long-term access keys for granting permissions to AWS services

> [↑ Back to Table of Contents](#table-of-contents)

---

## Access Methods

- **AWS Management Console**: Browser-based; authenticate with username, password, and optional MFA
- **AWS CLI**: Command-line tool; authenticate via long-lived access keys using `aws configure`, or use modern keyless browser-based login via `aws login` (recommended, for standard IAM Users, [docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html)) or `aws sso login` (for IAM Identity Center/SSO environments)

> [↑ Back to Table of Contents](#table-of-contents)

---

## Shared Responsibility Model

### AWS Responsibilities

- Maintains the global IAM infrastructure, underlying network security, and vulnerability analysis
- Handles compliance validation and service-level agreements

### Customer Responsibilities

- Enforce MFA, set strong password policies, and rotate access keys regularly
- Apply least-privilege permissions and review access patterns to prevent privilege creep
- Manage users, groups, roles, and policies throughout their lifecycle

> [↑ Back to Table of Contents](#table-of-contents)

---

## Audit Tools

- **IAM Access Advisor**: _User-level_ list of services a user can access, showing when each was last accessed and the source policy; use this to identify and revoke unused permissions (least privilege)
- **IAM Credentials Report**: _Account-level_ CSV export listing all IAM users and the status of their passwords, access keys, and MFA devices

> [↑ Back to Table of Contents](#table-of-contents)

---

## Best Practices

- Avoid using the root account for daily tasks; create IAM users for all regular work
- Enforce MFA and strong password policies for all users
- Use roles to grant permissions to AWS services rather than embedding long-term access keys
- Assign users to groups and attach policies to groups rather than to individual users for scalable permission management
- Assign one IAM user per physical person to maintain individual accountability
- Never share IAM users or access keys between people or applications

> [↑ Back to Table of Contents](#table-of-contents)
