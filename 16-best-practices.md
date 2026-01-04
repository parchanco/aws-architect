# 16 - Best Practices & Cost Optimization

## Security Best Practices

```
✅ Enable MFA en root y usuarios críticos
✅ Usar IAM Roles (no Access Keys en código)
✅ Least Privilege (mínimos permisos)
✅ Encrypt data (at-rest y in-transit)
✅ Security Groups restrictivos
✅ VPC: Public/Private subnets
✅ CloudTrail enabled (audit trail)
✅ AWS Config (compliance)
✅ Regular security audits
✅ Patch management automático
```

## High Availability

```
✅ Multi-AZ deployments
✅ Auto Scaling Groups
✅ Load Balancers
✅ RDS Multi-AZ
✅ Read Replicas (DB)
✅ Route 53 health checks
✅ Backups automáticos
✅ Disaster Recovery plan
```

## Cost Optimization

### Compute

```
✅ Reserved Instances (prod 24/7) → 40-75% ahorro
✅ Savings Plans (flexible)
✅ Spot Instances (batch jobs) → 70-90% ahorro
✅ Right-sizing (monitor y ajustar)
✅ Stop/Start dev instances (noches/fines semana)
✅ Use Fargate para containers (no over-provision)
```

### Storage

```
✅ S3 Lifecycle policies:
   - Standard → IA (30 días)
   - IA → Glacier (90 días)
✅ Delete old snapshots
✅ EBS gp3 (mejor que gp2)
✅ Delete unattached EBS volumes
✅ S3 Intelligent-Tiering
```

### Networking

```
✅ CloudFront (reduce data transfer)
✅ VPC endpoints (avoid NAT Gateway costs)
✅ Consolidate data transfer
✅ Monitor data transfer costs
```

### Database

```
✅ RDS Reserved Instances
✅ Aurora Serverless v2 (dev/test)
✅ Right-size DB instances
✅ Delete old automated backups
✅ Use Read Replicas (reduce load master)
```

## Monitoring & Alerting

```
✅ CloudWatch Alarms:
   - CPU > 80%
   - Disk space > 85%
   - HTTP 5xx errors
   - DB connections > 80%
✅ SNS notifications
✅ CloudWatch Dashboards
✅ AWS Cost Explorer
✅ AWS Budgets (alerts gastos)
```

## Tagging Strategy

```
Tags obligatorios:
- Name: odoo-prod-web-01
- Environment: production
- Application: odoo
- Owner: devops-team
- CostCenter: engineering
- BackupPolicy: daily

Beneficios:
✅ Cost allocation
✅ Automation
✅ Access control
✅ Compliance
```

## Backup Strategy

```
3-2-1 Rule:
- 3 copias de data
- 2 diferentes storage types
- 1 offsite

Odoo example:
1. RDS automated backups (7 días)
2. RDS manual snapshots (weekly → 30 días)
3. Snapshot copy a otra región (monthly → 12 meses)
4. EBS snapshots (daily → 7 días)
5. S3 attachments con versioning
6. Critical data → Glacier (long-term)
```

## Arquitectura Odoo Production

```
Route 53
    ↓
CloudFront (static assets)
    ↓
WAF (protection)
    ↓
ALB (HTTPS, Multi-AZ)
    ↓
Auto Scaling Group (2-10 instances)
    ├─ EC2 Odoo (from Golden AMI)
    ├─ Security Groups
    └─ Private Subnets
    ↓
RDS PostgreSQL Multi-AZ
    ├─ Automated backups
    ├─ Encryption
    └─ Private Subnets
    
ElastiCache Redis (sessions)
EFS (shared filestore)
S3 (attachments, backups)
CloudWatch (monitoring)
SNS (alerts)
```

## Disaster Recovery

```
Strategies:

Backup & Restore (RPO: hours, RTO: 24h):
- Cheapest
- Regular backups to S3

Pilot Light (RPO: minutes, RTO: hours):
- Critical core (DB) always running en DR region
- Scale up when needed

Warm Standby (RPO: seconds, RTO: minutes):
- Scaled-down version running en DR region
- Scale up cuando falla primary

Multi-Site (RPO: 0, RTO: 0):
- Full production en múltiples regiones
- Active-Active
- Most expensive
```

---

**Siguiente**: [17 - CLI Commands](17-cli-commands.md)
