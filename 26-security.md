# 26 - AWS Security & Encryption

## AWS KMS (Key Management Service)

Managed encryption keys.

### Conceptos

**CMK (Customer Master Key)**:
```
AWS Managed: aws/s3, aws/rds
Customer Managed: Tu control completo
```

**Data Keys**:
```
Generated from CMK
Encrypt actual data
Envelope encryption
```

### Key Types

**Symmetric (AES-256)**:
```
Single key (encrypt/decrypt)
AWS services integration
Never exposed (API only)
```

**Asymmetric (RSA, ECC)**:
```
Public key (encrypt)
Private key (decrypt)
Can download public key
Use: Encryption outside AWS
```

### Key Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "Allow use of key",
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123:role/OdooRole"
    },
    "Action": [
      "kms:Decrypt",
      "kms:DescribeKey"
    ],
    "Resource": "*"
  }]
}
```

### Encryption Operations

```python
import boto3
import base64

kms = boto3.client('kms')

# Encrypt
response = kms.encrypt(
    KeyId='alias/odoo-key',
    Plaintext=b'my secret data'
)
ciphertext = response['CiphertextBlob']

# Decrypt
response = kms.decrypt(
    CiphertextBlob=ciphertext
)
plaintext = response['Plaintext']
```

### Envelope Encryption

```
1. Generate data key from CMK
2. Encrypt data with data key
3. Encrypt data key with CMK
4. Store encrypted data + encrypted data key

Decrypt:
1. Decrypt data key with CMK
2. Decrypt data with data key
3. Discard data key

Benefits:
✓ Better performance
✓ CMK never leaves AWS
✓ Network efficiency
```

### KMS Key Rotation

```
Automatic rotation: Every year
Manual rotation: Create new key, update alias

Automatic rotation:
✓ Same key ID
✓ Transparent
✓ Old keys still valid (decrypt old data)
```

### KMS Limits

```
API requests: 5,500-30,000 req/s (varies by region)
Symmetric: 4 KB data limit
Asymmetric: Varies by algorithm

Solution: Envelope encryption
```

## AWS CloudHSM

Hardware Security Module.

### vs KMS

```
KMS:
- Multi-tenant
- AWS managed
- Integrated services
- $1/key/month
- API-based

CloudHSM:
- Single-tenant
- You manage
- FIPS 140-2 Level 3
- $1.60/hour per HSM
- Industry-standard APIs (PKCS#11, JCE)
```

### Use Cases

```
✓ Regulatory compliance (FIPS 140-2 Level 3)
✓ Contractual obligations
✓ Need full control of keys
✓ Custom crypto algorithms
```

## AWS Certificate Manager (ACM)

Managed SSL/TLS certificates.

### Features

```
Free certificates
Automatic renewal
Integration: ELB, CloudFront, API Gateway
Wildcard certificates
```

### Request Certificate

```bash
aws acm request-certificate   --domain-name odoo.tuempresa.com   --subject-alternative-names *.tuempresa.com   --validation-method DNS
```

**Validation Methods**:
- DNS: Add CNAME record
- Email: Confirm via email

### Certificate Renewal

```
Public certificates: Auto-renew (60 days before expiry)
Private certificates: Auto-renew
```

### Import Certificate

```bash
aws acm import-certificate   --certificate fileb://cert.pem   --private-key fileb://key.pem   --certificate-chain fileb://chain.pem
```

## AWS Secrets Manager

Managed secrets storage.

### Features

```
Encryption: KMS
Rotation: Automatic
Versioning: Multiple versions
Replication: Cross-region
Integration: RDS, Redshift, DocumentDB
```

### Store Secret

```python
import boto3

secrets = boto3.client('secretsmanager')

secrets.create_secret(
    Name='odoo/prod/db_password',
    SecretString='MySecurePassword123',
    KmsKeyId='alias/odoo-key'
)
```

### Retrieve Secret

```python
response = secrets.get_secret_value(
    SecretId='odoo/prod/db_password'
)
secret = response['SecretString']
```

### Automatic Rotation

```python
# Lambda rotation function
def lambda_handler(event, context):
    token = event['Token']
    step = event['Step']
    
    if step == "createSecret":
        # Generate new password
        new_password = generate_password()
        # Store pending version
        secrets.put_secret_value(
            SecretId=event['SecretId'],
            ClientRequestToken=token,
            SecretString=new_password,
            VersionStages=['AWSPENDING']
        )
    
    elif step == "setSecret":
        # Update database password
        update_db_password(new_password)
    
    elif step == "testSecret":
        # Test new password
        test_connection(new_password)
    
    elif step == "finishSecret":
        # Mark as current
        secrets.update_secret_version_stage(...)
```

### Secrets Manager para Odoo

```python
# odoo.conf no tiene password hardcodeado
# Fetch al start

import boto3
import json

secrets = boto3.client('secretsmanager')

response = secrets.get_secret_value(SecretId='odoo/prod/config')
config = json.loads(response['SecretString'])

db_password = config['db_password']
admin_password = config['admin_password']
smtp_password = config['smtp_password']
```

## AWS Shield

DDoS protection.

### Shield Standard

```
Free
Automatic
Protection: Layer 3/4 (network/transport)
All AWS customers
```

### Shield Advanced

```
$3,000/month
Enhanced protection
Layer 7 (application)
24/7 DDoS Response Team (DRT)
Cost protection (scaling costs during attack)
Integration: CloudFront, Route 53, ELB
```

## AWS WAF (Web Application Firewall)

Protect web applications.

### Features

```
Layer 7 (HTTP/HTTPS)
Deploy on: CloudFront, ALB, API Gateway
Rules: IP, geo, rate limiting, SQL injection, XSS
```

### Web ACL

```
Rules:
1. Allow all except
2. Block all except
3. Count (monitor mode)

Example rules:
- Block IPs from specific countries
- Rate limit: 2000 req/5min per IP
- Block SQL injection patterns
- Block XSS patterns
- Size constraints
```

### Rule Example

```json
{
  "Name": "RateLimitRule",
  "Priority": 1,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 2000,
      "AggregateKeyType": "IP"
    }
  },
  "Action": {"Block": {}},
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "RateLimit"
  }
}
```

### Managed Rules

```
AWS Managed:
- Core rule set
- Known bad inputs
- SQL injection
- Linux/Windows exploits

3rd Party:
- F5
- Fortinet
- Imperva
```

## AWS Firewall Manager

Centralize WAF management.

```
AWS Organizations integration
Deploy WAF rules across accounts
Automatic compliance
Security policies
```

## GuardDuty

Intelligent threat detection.

### How it Works

```
Data sources:
- CloudTrail logs
- VPC Flow Logs
- DNS logs

Machine Learning
    ↓
Anomaly detection
    ↓
Findings (threats)
    ↓
EventBridge → Lambda/SNS
```

### Findings

```
Severity: Low, Medium, High
Types:
- Backdoor (compromised instance)
- Behavior (unusual API calls)
- Cryptocurrency (mining)
- Pentest (port scanning)
- Persistence (IAM abuse)
- Recon (reconnaissance)
- ResourceConsumption
- Stealth
- Trojan
- UnauthorizedAccess
```

### Enable

```
30-day free trial
Per million events pricing
Multi-account (via Organizations)
```

## Amazon Inspector

Automated security assessment.

### Scans

**Network Assessments**:
```
Network reachability
Unintended network accessibility
```

**Host Assessments**:
```
Vulnerabilities
CIS benchmarks
Security best practices

Requires: SSM Agent
```

### How it Works

```
1. Define target (EC2 instances)
2. Install agent (SSM)
3. Define rules package
4. Run assessment
5. Review findings
6. Remediate
```

## Amazon Macie

Discover y protect sensitive data en S3.

### Features

```
Machine learning
Identify PII (Personal Identifiable Information)
Sensitive data: Credit cards, IDs, secrets
Security and access control issues
Compliance (GDPR, HIPAA)
```

### Findings

```
Policy findings:
- Public bucket
- Unencrypted bucket
- Shared with external account

Sensitive data findings:
- Credit card numbers
- API keys
- Personal data
```

## Security Hub

Centralized security findings.

```
Aggregates findings:
- GuardDuty
- Inspector
- Macie
- IAM Access Analyzer
- Systems Manager
- Firewall Manager

Compliance checks:
- CIS AWS Foundations
- PCI DSS
- AWS Best Practices

→ Single pane of glass
```

## Detective

Analyze y investigate security issues.

```
Data sources:
- VPC Flow Logs
- CloudTrail
- GuardDuty

Visualizations
Root cause analysis
Security investigations
```

## Security Best Practices

### Data Encryption

```
At Rest:
✅ EBS: Encrypted
✅ S3: SSE-S3/SSE-KMS
✅ RDS: Encrypted
✅ Snapshots: Encrypted

In Transit:
✅ SSL/TLS everywhere
✅ VPN to VPC
✅ Direct Connect + VPN
```

### Network Security

```
✅ VPC: Public/Private subnets
✅ Security Groups: Restrictive
✅ NACLs: Additional layer
✅ VPC Flow Logs: Monitor traffic
✅ No public databases
✅ Use VPC endpoints (S3, DynamoDB)
```

### Identity & Access

```
✅ IAM: Least privilege
✅ MFA: Everywhere
✅ Roles: Not Access Keys
✅ Rotate credentials: 90 days
✅ CloudTrail: All regions
✅ AWS Config: Compliance
```

### Monitoring

```
✅ CloudWatch: Alarms
✅ GuardDuty: Threat detection
✅ Security Hub: Centralized view
✅ Inspector: Vulnerability scanning
✅ Macie: Sensitive data
```

### Compliance

```
✅ AWS Artifact: Compliance reports
✅ AWS Config: Config rules
✅ AWS Audit Manager: Compliance audits
✅ Security Hub: Compliance checks
```

---

**Certificación**: Encryption (KMS), WAF, Shield, GuardDuty son importantes.

**Siguiente**: [27 - VPC Networking](27-vpc.md)
