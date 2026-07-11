# 28 - Disaster Recovery & Migrations

## Disaster Recovery Strategies

### RPO y RTO

```
RPO (Recovery Point Objective):
- Cuánta data puedes perder
- Tiempo entre backups
- Ejemplo: RPO 1 hora = backups hourly

RTO (Recovery Time Objective):
- Cuánto downtime toleras
- Tiempo para recuperar
- Ejemplo: RTO 4 horas = max 4h down
```

### DR Strategies

**1. Backup & Restore**
```
RPO: Hours
RTO: 24+ hours
Cost: Lowest

Setup:
- Regular backups a S3
- RDS snapshots
- AMIs
- Scripts to restore

Odoo example:
- Daily RDS snapshots → S3
- Daily AMI
- Filestore → S3
- Restore on demand
```

**2. Pilot Light**
```
RPO: Minutes
RTO: Hours
Cost: Low

Setup:
- Core running en DR region
- Database replicating
- Scale up when needed

Odoo example:
- RDS Read Replica en us-east-1 (standby)
- AMIs en us-east-1
- Disaster: Promote replica, launch EC2s
```

**3. Warm Standby**
```
RPO: Seconds
RTO: Minutes
Cost: Medium

Setup:
- Scaled-down version running
- Database replicating
- Scale up when needed

Odoo example:
- 1 small EC2 running (vs 10 prod)
- RDS Read Replica
- Disaster: Update DNS, scale up ASG
```

**4. Multi-Site (Active-Active)**
```
RPO: None (real-time)
RTO: Automatic
Cost: Highest

Setup:
- Full production en múltiples regiones
- Route 53 weighted/latency routing
- Database multi-region

Odoo example:
- Full stack eu-west-1
- Full stack us-east-1
- Route 53: 50% traffic each
- Aurora Global Database
```

## Backup Strategies

### RDS Backups

```
Automated:
- Daily full + transaction logs
- Point-in-time recovery
- Retention: 1-35 days
- Cross-region copy

Manual Snapshots:
- On-demand
- Indefinite retention
- Share with other accounts
- Copy to other regions
```

**Script automatización**:
```bash
#!/bin/bash
# Daily snapshot con retention

DATE=$(date +%Y%m%d)
INSTANCE="odoo-prod-db"

# Create snapshot
aws rds create-db-snapshot   --db-instance-identifier $INSTANCE   --db-snapshot-identifier "${INSTANCE}-${DATE}"

# Delete snapshots > 30 days
aws rds describe-db-snapshots   --db-instance-identifier $INSTANCE   --query "DBSnapshots[?SnapshotCreateTime<='$(date -d '30 days ago' -I)'].DBSnapshotIdentifier"   --output text | xargs -n1 aws rds delete-db-snapshot --db-snapshot-identifier
```

### EBS Snapshots

```
Data Lifecycle Manager:
- Automated snapshot creation
- Retention policies
- Cross-region copy
- Tags
```

**Policy**:
```
Target: Volumes tagged Backup=true
Schedule: Daily @ 02:00 UTC
Retention: 7 daily, 4 weekly, 12 monthly
Cross-region: Copy to us-east-1
```

### S3 Backup

```
Versioning: Keep múltiples versions
Cross-Region Replication: S3 → S3 (otra región)
Glacier: Long-term archival

Lifecycle:
- 0 días: S3 Standard
- 30 días: S3 Standard-IA
- 90 días: S3 Glacier
- 365 días: S3 Glacier Deep Archive
- 2555 días (7 años): Delete
```

## Database Migration

### AWS DMS (Database Migration Service)

Migrar bases de datos con minimal downtime.

**Types**:
```
Homogeneous: PostgreSQL → RDS PostgreSQL
Heterogeneous: Oracle → Aurora PostgreSQL (requires SCT)
```

**Migration Types**:
```
Full Load: One-time complete migration
Full Load + CDC: Ongoing replication
CDC Only: Replicate changes only
```

**Example: On-premise PostgreSQL → RDS**
```
1. Setup DMS replication instance
2. Create source endpoint (on-premise)
3. Create target endpoint (RDS)
4. Create migration task
   - Full load + CDC
5. Start migration
6. Monitor progress
7. Cutover:
   - Stop writes to source
   - Wait for replication lag = 0
   - Switch app to RDS
   - Stop task
```

### Schema Conversion Tool (SCT)

Convert schemas entre engines diferentes.

```
Source: Oracle
Target: PostgreSQL

SCT converts:
- Tables
- Views
- Stored procedures
- Functions
- Triggers

Manual review needed for:
- Complex PL/SQL
- Proprietary features
```

## AWS Backup

Centralized backup service.

### Features

```
Backup Plans:
- Resources to backup
- Schedule
- Retention
- Lifecycle
- Cross-region copy

Supports:
- EC2, EBS
- RDS, Aurora, DynamoDB
- EFS, FSx
- Storage Gateway
- DocumentDB
```

**Backup Plan Example**:
```
Name: Daily-Production-Backup
Resources: Tag Environment=production

Rules:
1. Daily @ 02:00 UTC
   Retention: 7 days
   Copy to: us-east-1
   
2. Weekly (Sunday)
   Retention: 4 weeks
   Move to Cold Storage: 30 days
```

### Backup Vault

```
Container para backups
Encryption con KMS
Access policies
Lock (WORM)
```

## Application Migration

### AWS Application Discovery Service

Discover on-premise servers.

```
Agentless discovery:
- VMware environment
- Collect configuration

Agent-based discovery:
- Install agent
- Detailed performance data
```

### AWS Migration Hub

Centralized tracking.

```
Track migrations:
- Application Discovery
- Server Migration Service
- Database Migration Service

Dashboard:
- Migration progress
- Status
- Issues
```

### AWS Server Migration Service (SMS)

Migrate VMs a AWS.

```
Supports:
- VMware vSphere
- Microsoft Hyper-V
- Azure VMs

Process:
1. Install SMS Connector (on-premise)
2. Configure replication
3. Test migrations
4. Cutover
```

## DataSync

Transfer data a/desde AWS.

```
On-premise → AWS (S3, EFS, FSx)
AWS → AWS (cross-region)

Features:
- Incremental transfers
- Bandwidth throttling
- Data validation
- Scheduling

10x faster than open-source tools
```

**Example: NFS → EFS**
```bash
# Create task
aws datasync create-task   --source-location-arn arn:aws:datasync:...:location/nfs-xxx   --destination-location-arn arn:aws:datasync:...:location/efs-xxx   --schedule ScheduleExpression="cron(0 2 * * ? *)"
```

## Transfer Family

Managed file transfer (SFTP, FTPS, FTP).

```
Clients → Transfer Family → S3/EFS

Use cases:
- Legacy file transfers
- Partner integrations
- Compliance requirements

Pricing:
- $0.30/hour per endpoint
- $0.04/GB transferred
```

## Disaster Recovery - Odoo Complete

```
Strategy: Pilot Light
RPO: 5 minutes
RTO: 30 minutes

Primary: eu-west-1
├─ VPC, Subnets, SGs
├─ ALB
├─ ASG (2-10 EC2)
├─ RDS Multi-AZ
├─ ElastiCache
└─ EFS

DR: us-east-1
├─ VPC, Subnets, SGs (pre-configured)
├─ RDS Read Replica (continuous replication)
├─ AMIs (synced weekly)
├─ S3 bucket (CRR from eu-west-1)
└─ Route 53 health check

Disaster procedure:
1. Detect failure (Route 53 health check)
2. Promote RDS Read Replica (5 min)
3. Launch EC2 from AMI via ASG (2 min)
4. Update Route 53 (failover policy)
5. Verify application (5 min)
6. Notify users

Total RTO: ~15-30 minutes
```

## Chaos Engineering

Test DR regularly.

```
AWS Fault Injection Simulator:
- Inject failures
- Test resilience
- Validate DR procedures

Scenarios:
- Terminate random EC2s
- Throttle network
- Inject CPU stress
- Fail AZ
```

## Cost Optimization DR

```
Pilot Light:
✅ RDS Read Replica: Running cost
✅ Snapshots: Storage cost only
✅ AMIs: Storage cost only
✅ No EC2 running (pay on launch)

Savings vs Active-Active: 70-80%
```

## DR Checklist

```
✅ Document RTO/RPO requirements
✅ Choose appropriate strategy
✅ Automate backups
✅ Test restore procedures monthly
✅ Cross-region backups
✅ Playbook for disaster scenarios
✅ Monitor health checks
✅ Practice DR drills quarterly
✅ Update documentation
✅ Review and improve
```

## Multi-Region Failover Example

```bash
#!/bin/bash
# DR Failover Script

# 1. Promote RDS Read Replica
aws rds promote-read-replica   --db-instance-identifier odoo-dr   --region us-east-1

# 2. Wait for promotion
aws rds wait db-instance-available   --db-instance-identifier odoo-dr   --region us-east-1

# 3. Update ASG desired capacity
aws autoscaling set-desired-capacity   --auto-scaling-group-name odoo-asg-dr   --desired-capacity 4   --region us-east-1

# 4. Update Route 53 (failover)
# Manual or use API to update health check

# 5. Notify team
aws sns publish   --topic-arn arn:aws:sns:us-east-1:123:alerts   --message "DR activated in us-east-1"
```

---

**Certificación**: DR strategies, RPO/RTO, backup best practices son importantes.

**Siguiente**: [29 - FinOps y economía cloud](29-finops-economia-cloud.md)
