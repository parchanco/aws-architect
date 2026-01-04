# 25 - IAM Advanced & Security

## IAM Policies Deep Dive

### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowS3ReadOdooAttachments",
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::odoo-attachments",
      "arn:aws:s3:::odoo-attachments/*"
    ],
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "10.0.0.0/8"
      }
    }
  }]
}
```

### Policy Types

**Identity-based**:
```
Attached to: Users, Groups, Roles
Who can do what

Example: User pablo can read S3
```

**Resource-based**:
```
Attached to: Resources (S3, SQS, Lambda)
Who can access this resource

Example: S3 bucket allows role X
```

**Permission Boundaries**:
```
Max permissions an identity can have
Even if policy grants more

Use: Delegate admin without full access
```

**SCPs (Service Control Policies)**:
```
AWS Organizations
Max permissions for accounts
Override: None (top-level control)
```

### Policy Evaluation Logic

```
1. Explicit DENY → DENY (always wins)
2. Explicit ALLOW → ALLOW
3. Default → DENY (implicit)

Example:
Policy A: Allow s3:*
Policy B: Deny s3:DeleteBucket
Result: Can do all except DeleteBucket
```

### Conditions

**IP-based**:
```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": ["203.0.113.0/24"]
  }
}
```

**Time-based**:
```json
"Condition": {
  "DateGreaterThan": {
    "aws:CurrentTime": "2024-01-01T00:00:00Z"
  },
  "DateLessThan": {
    "aws:CurrentTime": "2024-12-31T23:59:59Z"
  }
}
```

**MFA**:
```json
"Condition": {
  "BoolIfExists": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

**Tag-based**:
```json
"Condition": {
  "StringEquals": {
    "ec2:ResourceTag/Environment": "production"
  }
}
```

### Variables

```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::bucket/${aws:username}/*"
}

User pablo → Can access bucket/pablo/*
User maria → Can access bucket/maria/*
```

## IAM Roles Deep Dive

### Service Roles

```
EC2 Role:
Trust policy: ec2.amazonaws.com
Permissions: What EC2 can do

Lambda Role:
Trust policy: lambda.amazonaws.com
Permissions: What Lambda can do
```

### Cross-Account Roles

```
Account A (Production):
- Has resources
- Creates role "ProdAccessRole"
- Trust policy: Account B

Account B (Development):
- Users assume ProdAccessRole
- Temporary credentials
- Access Account A resources
```

**Setup**:
```bash
# Account A: Create role
aws iam create-role --role-name ProdAccessRole   --assume-role-policy-document '{
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::AccountB:root"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Account B: Assume role
aws sts assume-role   --role-arn arn:aws:iam::AccountA:role/ProdAccessRole   --role-session-name pablo-session
```

### Role Chaining

```
User → AssumeRole(RoleA)
RoleA → AssumeRole(RoleB)
RoleB → AssumeRole(RoleC)

Max: 1 hour session
Use case: Cross-account multi-hop
```

## AWS STS (Security Token Service)

Temporary credentials.

### AssumeRole

```python
import boto3

sts = boto3.client('sts')

response = sts.assume_role(
    RoleArn='arn:aws:iam::123:role/S3AccessRole',
    RoleSessionName='pablo-session',
    DurationSeconds=3600  # 1 hour
)

credentials = response['Credentials']
# AccessKeyId (temporary)
# SecretAccessKey (temporary)
# SessionToken
# Expiration
```

### AssumeRoleWithSAML

```
SAML 2.0 identity provider
SSO with corporate credentials
Google Workspace, Azure AD, Okta
```

### AssumeRoleWithWebIdentity

```
Web identity provider
Login with Amazon, Facebook, Google
Get temporary AWS credentials

⚠️ Prefer Cognito (easier)
```

### GetSessionToken

```
MFA for temporary credentials

Use case:
User has long-term credentials
Needs MFA for sensitive operations
GetSessionToken with MFA
Temporary credentials with MFA
```

### GetFederationToken

```
Federated user (no IAM user)
Temporary credentials
Use case: Proxy for on-premise users
```

## Identity Federation

### SAML 2.0

```
Corporate Login (SSO)
    ↓
SAML IdP (Google Workspace, Azure AD)
    ↓
AWS STS (AssumeRoleWithSAML)
    ↓
Temporary credentials
    ↓
Access AWS
```

**Setup**:
1. Configure IdP
2. Create SAML provider en AWS
3. Create role with SAML trust
4. Users login via IdP
5. Redirect to AWS console

### Custom Identity Broker

```
Corporate IdP
    ↓
Custom App (identity broker)
    ↓ STS AssumeRole/GetFederationToken
Temporary credentials
    ↓
Access AWS
```

### Web Identity Federation

```
User login (Google, Facebook, Amazon)
    ↓
Get token
    ↓
Exchange for AWS credentials (Cognito)
    ↓
Access AWS resources
```

## Amazon Cognito

User identity y access para apps.

### User Pools

```
Sign up / Sign in
User directory
MFA
Social login (Google, Facebook)
Email/phone verification
Custom attributes

Use case: App authentication
```

### Identity Pools

```
AWS credentials para users
Federated identities
Guest access
Role mapping

Use case: Access AWS services from app
```

### Example Flow

```
1. User sign in → User Pool
2. Get JWT token
3. Exchange token → Identity Pool
4. Get temporary AWS credentials
5. Access S3, DynamoDB, etc.
```

## AWS Directory Service

Managed Active Directory.

### Options

**AWS Managed Microsoft AD**:
```
Full Microsoft AD
Trust with on-premise AD
Multi-AZ
Automated backups
```

**AD Connector**:
```
Proxy to on-premise AD
No caching
MFA support
```

**Simple AD**:
```
Standalone directory
Samba 4
Small/Large
No trust relationships
```

## AWS Organizations

Manage múltiples AWS accounts.

### Structure

```
Root
├─ OU: Production
│  ├─ Account: Prod-Web
│  └─ Account: Prod-DB
├─ OU: Development
│  ├─ Account: Dev
│  └─ Account: Staging
└─ OU: Security
   └─ Account: Audit
```

### Features

**Consolidated Billing**:
```
Single bill para todas las accounts
Volume discounts
Reserved Instances shared
```

**Service Control Policies (SCPs)**:
```
Max permissions for accounts/OUs
Override todas las policies
Even root user limited

Example: Deny all except eu-west-1
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": "eu-west-1"
    }
  }
}
```

**AWS SSO**:
```
Single sign-on para múltiples accounts
SAML 2.0
Permission sets
```

## AWS Control Tower

Setup y govern multi-account environment.

### Features
```
Automated setup
Guardrails (SCPs + Config rules)
Account Factory
Dashboard
```

**Guardrails**:
```
Preventive: SCPs (deny actions)
Detective: Config rules (detect non-compliance)

Example:
- Prevent public S3 buckets
- Detect unencrypted EBS volumes
```

## IAM Best Practices Recap

```
✅ Root account: MFA, no uso diario
✅ Users: Individual IAM users, MFA
✅ Groups: Assign permissions a groups
✅ Roles: Para servicios AWS y cross-account
✅ Least Privilege: Mínimos permisos necesarios
✅ Policies: Use managed policies, create custom cuando necesario
✅ Credentials: Rotate Access Keys 90 días
✅ Conditions: IP, MFA, time restrictions
✅ Audit: CloudTrail, Access Advisor, Credentials Report
✅ Password Policy: Strong requirements
✅ Tags: Tag users, roles para cost allocation
✅ Permission Boundaries: Delegate safely

❌ No hardcodear credentials
❌ No compartir Access Keys
❌ No usar root account
❌ No dar permisos excesivos
```

---

**Certificación**: IAM policies, roles, federation son fundamentales.

**Siguiente**: [26 - Security & Encryption](26-security.md)
