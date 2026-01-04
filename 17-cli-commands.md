# 17 - Comandos AWS CLI Útiles

## IAM

```bash
# Listar usuarios
aws iam list-users

# Crear usuario
aws iam create-user --user-name new-developer

# Attach policy
aws iam attach-user-policy --user-name pablo     --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# Crear Access Keys
aws iam create-access-key --user-name pablo

# Listar roles
aws iam list-roles
```

## EC2

```bash
# Listar instancias
aws ec2 describe-instances

# Listar solo running
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Crear AMI
aws ec2 create-image --instance-id i-xxx --name "odoo-backup-$(date +%Y%m%d)"

# Crear snapshot
aws ec2 create-snapshot --volume-id vol-xxx --description "Daily backup"

# Listar Security Groups
aws ec2 describe-security-groups

# Crear tags
aws ec2 create-tags --resources i-xxx --tags Key=Name,Value=Odoo-Prod
```

## S3

```bash
# Listar buckets
aws s3 ls

# Listar objetos
aws s3 ls s3://my-bucket/

# Upload
aws s3 cp file.pdf s3://my-bucket/

# Download
aws s3 cp s3://my-bucket/file.pdf ./

# Sync directorio
aws s3 sync ./local-dir s3://my-bucket/backup/

# Delete
aws s3 rm s3://my-bucket/file.pdf

# Delete recursivo
aws s3 rm s3://my-bucket/folder/ --recursive

# Crear bucket
aws s3 mb s3://my-new-bucket --region eu-west-1

# Delete bucket
aws s3 rb s3://my-bucket --force
```

## RDS

```bash
# Listar DB instances
aws rds describe-db-instances

# Crear snapshot
aws rds create-db-snapshot     --db-instance-identifier mydb     --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# Listar snapshots
aws rds describe-db-snapshots

# Restore desde snapshot
aws rds restore-db-instance-from-db-snapshot     --db-instance-identifier mydb-restored     --db-snapshot-identifier mydb-snapshot-20240103

# Stop DB
aws rds stop-db-instance --db-instance-identifier mydb

# Start DB
aws rds start-db-instance --db-instance-identifier mydb
```

## CloudWatch

```bash
# Get metrics
aws cloudwatch get-metric-statistics     --namespace AWS/EC2     --metric-name CPUUtilization     --dimensions Name=InstanceId,Value=i-xxx     --start-time 2024-01-03T00:00:00Z     --end-time 2024-01-03T23:59:59Z     --period 3600     --statistics Average

# Crear alarm
aws cloudwatch put-metric-alarm     --alarm-name cpu-high     --alarm-description "CPU > 80%"     --metric-name CPUUtilization     --namespace AWS/EC2     --statistic Average     --period 300     --threshold 80     --comparison-operator GreaterThanThreshold     --dimensions Name=InstanceId,Value=i-xxx     --evaluation-periods 2
```

## Auto Scaling

```bash
# Listar ASGs
aws autoscaling describe-auto-scaling-groups

# Set desired capacity
aws autoscaling set-desired-capacity     --auto-scaling-group-name odoo-asg     --desired-capacity 5

# Update ASG
aws autoscaling update-auto-scaling-group     --auto-scaling-group-name odoo-asg     --min-size 2     --max-size 10
```

## Route 53

```bash
# Listar hosted zones
aws route53 list-hosted-zones

# Listar records
aws route53 list-resource-record-sets --hosted-zone-id Z123456

# Crear record
aws route53 change-resource-record-sets     --hosted-zone-id Z123456     --change-batch file://change-batch.json
```

## ELB

```bash
# Listar load balancers
aws elbv2 describe-load-balancers

# Listar target groups
aws elbv2 describe-target-groups

# Check target health
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...
```

## Systems Manager (Parameter Store)

```bash
# Put parameter
aws ssm put-parameter     --name /odoo/prod/db_password     --value "SecurePassword123"     --type SecureString

# Get parameter
aws ssm get-parameter     --name /odoo/prod/db_password     --with-decryption

# Get multiple parameters
aws ssm get-parameters     --names /odoo/prod/db_password /odoo/prod/admin_password     --with-decryption
```

## Cost Explorer

```bash
# Get cost and usage
aws ce get-cost-and-usage     --time-period Start=2024-01-01,End=2024-01-31     --granularity MONTHLY     --metrics BlendedCost     --group-by Type=DIMENSION,Key=SERVICE
```

## Useful Combinations

```bash
# Stop all instances con tag Environment=dev
aws ec2 describe-instances     --filters "Name=tag:Environment,Values=dev" "Name=instance-state-name,Values=running"     --query 'Reservations[].Instances[].InstanceId'     --output text | xargs aws ec2 stop-instances --instance-ids

# Backup all running instances
aws ec2 describe-instances     --filters "Name=instance-state-name,Values=running"     --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Name`].Value|[0]]'     --output text | while read id name; do
        aws ec2 create-image --instance-id $id --name "backup-$name-$(date +%Y%m%d)"
    done

# Delete old snapshots (>30 días)
aws ec2 describe-snapshots --owner-ids self     --query 'Snapshots[?StartTime<=`'$(date -d '30 days ago' -I)'`].[SnapshotId]'     --output text | xargs -n1 aws ec2 delete-snapshot --snapshot-id
```

---

**Siguiente**: [18 - Glosario](18-glosario.md)
