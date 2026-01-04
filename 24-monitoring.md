# 24 - Monitoring, Audit & Compliance

## Amazon CloudWatch

Monitoring y observability.

### CloudWatch Metrics

**Métricas por defecto (EC2)**:
```
CPUUtilization
NetworkIn/NetworkOut
DiskReadOps/DiskWriteOps
StatusCheckFailed

⚠️ NO incluye: Memory, Disk space
→ Necesitas CloudWatch Agent
```

**Métricas custom**:
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='Odoo/Production',
    MetricData=[{
        'MetricName': 'ActiveUsers',
        'Value': 150,
        'Unit': 'Count',
        'Timestamp': datetime.utcnow()
    }]
)
```

**Métricas detalladas EC2**:
```
Basic monitoring: 5 minutos (gratis)
Detailed monitoring: 1 minuto ($)
```

### CloudWatch Logs

**Log Groups y Streams**:
```
Log Group: /aws/lambda/my-function
├─ Log Stream: 2024/01/03/[$LATEST]abc123
└─ Log Stream: 2024/01/03/[$LATEST]def456

Retention: 1 día - indefinido
```

**CloudWatch Logs Insights**:
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
| sort @timestamp desc
```

**Exports**:
```
Logs → S3 (batch export)
Logs → Kinesis Data Streams (real-time)
Logs → Lambda (subscription filter)
```

### CloudWatch Alarms

**States**:
```
OK: Metric dentro threshold
ALARM: Metric fuera threshold
INSUFFICIENT_DATA: No hay datos
```

**Example Alarm**:
```bash
aws cloudwatch put-metric-alarm   --alarm-name cpu-high   --alarm-description "CPU > 80%"   --metric-name CPUUtilization   --namespace AWS/EC2   --statistic Average   --period 300   --threshold 80   --comparison-operator GreaterThanThreshold   --evaluation-periods 2   --dimensions Name=InstanceId,Value=i-xxx   --alarm-actions arn:aws:sns:eu-west-1:123:alerts
```

**Alarm Actions**:
```
SNS: Send notification
Auto Scaling: Scale up/down
EC2: Stop, terminate, reboot, recover
Systems Manager: Run automation
```

**Composite Alarms**:
```
Alarm combinando múltiples alarms:

HighCPU AND HighMemory → Critical alert
HighCPU OR HighDisk → Warning alert
```

### CloudWatch Dashboards

```
Visualización de métricas
Cross-region, cross-account
Widgets: Line, number, gauge, etc.
Auto-refresh
Sharable

Costo: $3/dashboard/month
```

### CloudWatch Agent

Recopila métricas adicionales.

**Install**:
```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
```

**Config (config.json)**:
```json
{
  "metrics": {
    "namespace": "Odoo/Production",
    "metrics_collected": {
      "mem": {
        "measurement": [{
          "name": "mem_used_percent",
          "rename": "MemoryUtilization"
        }],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [{
          "name": "used_percent",
          "rename": "DiskUtilization"
        }],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [{
          "file_path": "/var/log/odoo/odoo.log",
          "log_group_name": "/aws/odoo/production",
          "log_stream_name": "{instance_id}"
        }]
      }
    }
  }
}
```

**Start**:
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl   -a fetch-config   -m ec2   -c file:/opt/aws/amazon-cloudwatch-agent/config.json   -s
```

### CloudWatch Container Insights

Métricas de containers (ECS, EKS).

```
Automatic discovery
Pod/node metrics
Performance dashboards
```

### CloudWatch ServiceLens

Observability para aplicaciones distribuidas.

```
X-Ray traces
CloudWatch metrics
CloudWatch Logs

→ Unified view
```

## AWS X-Ray

Distributed tracing.

### Conceptos

```
Segment: Request path through app
Subsegment: Remote call (DB, API)
Trace: Collection of segments
Service Map: Visual representation
```

### Integration

**Lambda**:
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()  # Auto-instrument boto3, requests, etc.

@xray_recorder.capture('process_order')
def lambda_handler(event, context):
    # Automatically traced
    pass
```

**EC2/ECS**:
```
1. Install X-Ray daemon
2. Instrument code
3. Send segments to daemon
4. Daemon → X-Ray service
```

## AWS CloudTrail

Audit log de API calls.

### Características

```
Who: IAM user/role
What: API call
When: Timestamp
Where: Source IP, region
Result: Success/failure
```

**Events**:
```
Management events: Create EC2, Delete S3
Data events: GetObject, PutObject (S3)
Insights events: Unusual activity
```

### Setup

```
Create trail:
- Apply to all regions: ✓
- Management events: ✓
- Data events: S3, Lambda (optional)
- Log file validation: ✓
- SNS notification: (optional)
- CloudWatch Logs: ✓

Storage: S3 bucket
Encryption: SSE-S3 or SSE-KMS
```

### Use Cases

```
✓ Compliance
✓ Security analysis
✓ Troubleshooting
✓ Operational auditing
✓ Risk auditing
```

### Query CloudTrail Logs

**Athena**:
```sql
SELECT
  useridentity.principalid,
  eventname,
  sourceipaddress,
  eventtime
FROM cloudtrail_logs
WHERE eventname = 'RunInstances'
  AND eventtime > '2024-01-01'
ORDER BY eventtime DESC;
```

**CloudWatch Logs Insights**:
```
fields eventTime, eventName, userIdentity.principalId
| filter eventName = "DeleteBucket"
| sort eventTime desc
```

## AWS Config

Assess, audit, evaluate configurations.

### Funcionamiento

```
1. Record configurations
   EC2, S3, IAM, RDS, etc.

2. Evaluate against rules
   Is S3 bucket encrypted?
   Is SSH open to 0.0.0.0/0?

3. Remediation
   Automatic fix (optional)
   
4. Notifications
   SNS
```

### Config Rules

**AWS Managed Rules** (90+):
```
s3-bucket-public-read-prohibited
s3-bucket-ssl-requests-only
ec2-instance-managed-by-systems-manager
rds-storage-encrypted
iam-password-policy
```

**Custom Rules**:
```
Lambda function
Evaluate config
Return compliant/non-compliant
```

### Remediation

```
Non-compliant resource detected
    ↓
Automatic remediation (SSM Automation)
    ↓
Fix configuration
    ↓
Re-evaluate
```

**Example**:
```
Rule: S3 bucket must be encrypted
Trigger: Bucket created without encryption
Remediation: Lambda → Enable encryption
```

## AWS Systems Manager (SSM)

Operational hub para AWS resources.

### Session Manager

Secure shell sin SSH keys.

```
✅ No SSH port open
✅ No bastion host needed
✅ IAM authentication
✅ Audit trail
✅ Session logging
```

**Connect**:
```bash
aws ssm start-session --target i-xxx
```

### Parameter Store

Secure storage para configs/secrets.

```
Types:
- String: Plain text
- StringList: Comma-separated
- SecureString: KMS encrypted

Hierarchy:
/odoo/prod/db_password
/odoo/staging/db_password
/odoo/dev/db_password
```

**Get Parameter**:
```bash
aws ssm get-parameter   --name /odoo/prod/db_password   --with-decryption
```

**Python**:
```python
import boto3

ssm = boto3.client('ssm')

response = ssm.get_parameter(
    Name='/odoo/prod/db_password',
    WithDecryption=True
)

password = response['Parameter']['Value']
```

### Secrets Manager vs Parameter Store

```
Parameter Store:
✅ Free (Standard)
✅ 10,000 parameters
✅ Simple
❌ No auto rotation

Secrets Manager:
✅ Auto rotation
✅ RDS integration
✅ Cross-region replication
❌ $0.40/secret/month + API calls
```

### Run Command

Execute commands en instances.

```bash
aws ssm send-command   --document-name "AWS-RunShellScript"   --targets "Key=tag:Environment,Values=production"   --parameters 'commands=["sudo systemctl restart odoo"]'
```

### Patch Manager

Automate OS patching.

```
Patch Baseline: Define which patches
Maintenance Window: When to patch
Patch Group: Which instances
```

### Automation

Runbooks para tasks comunes.

```
Documents:
- AWS-UpdateLinuxAmi
- AWS-CreateSnapshot
- AWS-RestartEC2Instance

Custom documents (YAML/JSON)
```

## EventBridge (CloudWatch Events)

Event-driven architecture.

### Default Event Bus

AWS service events:
```
EC2 State Change
RDS Maintenance
Auto Scaling events
etc.
```

### Custom Event Bus

Your application events.

### Rules

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

**Targets**:
- Lambda
- SNS
- SQS
- Step Functions
- ECS Task
- CodePipeline
- SSM Automation

## Monitoring Best Practices Odoo

```
CloudWatch Metrics:
✅ EC2: CPU, Memory, Disk, Network
✅ RDS: CPU, Connections, FreeSpace, IOPS
✅ ALB: TargetResponseTime, HTTPCode_Target_5XX
✅ Custom: Active users, Request latency

CloudWatch Alarms:
✅ CPU > 80% (scale up)
✅ Memory > 85%
✅ Disk > 85%
✅ DB connections > 80%
✅ HTTP 5xx errors
✅ Target response time > 2s

CloudWatch Logs:
✅ Odoo application logs
✅ Nginx access logs
✅ PostgreSQL slow queries
✅ ALB access logs

CloudTrail:
✅ Enabled en todas las regiones
✅ Log file validation
✅ S3 + CloudWatch Logs

AWS Config:
✅ Track configuration changes
✅ S3 encryption rules
✅ Security Group rules

X-Ray:
✅ Trace requests end-to-end
✅ Identify bottlenecks
✅ Debug performance issues
```

---

**Certificación**: CloudWatch, CloudTrail, Config son muy importantes.

**Siguiente**: [25 - IAM Advanced](25-iam-advanced.md)
