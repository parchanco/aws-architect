# 06 - RDS & Aurora

## RDS (Relational Database Service)

Base de datos gestionada por AWS.

### Ventajas
- ✅ Provisionamiento automático
- ✅ Backups automáticos (point-in-time recovery)
- ✅ Multi-AZ para DR
- ✅ Read Replicas para scaling
- ✅ Storage auto-scaling
- ✅ Monitoring integrado

### Engines Soportados
- PostgreSQL ← Para Odoo
- MySQL, MariaDB
- Oracle, SQL Server
- Aurora (PostgreSQL/MySQL compatible)

## RDS Multi-AZ

Alta disponibilidad con failover automático.

```
Primary (AZ-1a) ⟷ Standby (AZ-1b)
Replicación SÍNCRONA

Si Primary falla:
→ Promote Standby (1-2 min)
→ DNS actualizado automáticamente
→ App reconecta sin cambios
```

## Read Replicas

Scaling de lecturas.

```
Master (Read/Write)
  ↓ Replicación ASÍNCRONA
  ├─ Replica 1 (Read)
  ├─ Replica 2 (Read)
  └─ Replica 3 (Read)

Hasta 15 replicas
Pueden estar en otras regiones
```

### Para Odoo
```python
# Separar workloads
Escrituras → Master
Reportes pesados → Read Replica
Analytics → Read Replica
```

## Backups

**Automated Backups**:
- Diarios + transaction logs cada 5 min
- Retention: 1-35 días
- Point-in-time recovery

**Manual Snapshots**:
- Retención indefinida
- Copy a otras regiones

## Storage Auto Scaling

```python
Initial: 100 GB
Maximum: 1000 GB
Threshold: 90%

→ RDS aumenta automáticamente
→ Sin downtime
```

## Security

```python
Network:
- Deploy en private subnets
- SG: Port 5432 desde SG de app (no 0.0.0.0/0)

Encryption:
- At-rest: KMS (enable al crear)
- In-transit: SSL/TLS

IAM Auth:
- En vez de passwords, usa IAM tokens
- Tokens expiran en 15 min
```

## Aurora

DB propietaria AWS, compatible PostgreSQL/MySQL.

### Aurora vs RDS PostgreSQL

```
Aurora:
✅ 5x performance
✅ Storage auto-scaling (10GB → 128TB)
✅ 15 Read Replicas (vs 5 RDS)
✅ Failover < 30 seg (vs 60-120 RDS)
✅ 6 copias data cross 3 AZs
✅ Self-healing storage
❌ ~20% más caro

Para Odoo:
- Producción crítica: Aurora
- Producción normal: RDS PostgreSQL
```

### Aurora Serverless

Auto-scaling basado en carga.

```python
Min: 0.5 ACUs (Aurora Capacity Units)
Max: 64 ACUs
→ Escala automáticamente
→ Pagas solo por uso

Uso:
✅ Dev/test (pausa cuando no usas)
✅ Tráfico impredecible
❌ Producción 24/7 predecible
```

### Aurora Global Database

```
Primary (eu-west-1):
  - 1 Writer
  - 15 Readers

Secondary (us-east-1):
  - 16 Readers (read-only)
  - Replicación < 1 segundo

Si primary falla → Promote secondary (< 1 min)
```

## Costos (Aproximados eu-west-1)

```
RDS PostgreSQL db.t3.large:
- On-Demand: ~$120/mes
- Reserved 1yr: ~$70/mes
- Multi-AZ: x2

Aurora db.t3.large:
- ~$145/mes
- Multi-AZ incluido

+ Storage: ~$0.10/GB-month
+ Backups > retention: ~$0.02/GB-month
```

## Setup Odoo con RDS

```python
# odoo.conf
db_host = myodoo.xxx.rds.amazonaws.com
db_port = 5432
db_user = odoo
db_password = (usar Secrets Manager)
db_sslmode = require

# No hay db_name en conf (Odoo multi-DB)
```

---

**Siguiente**: [07 - ElastiCache](07-elasticache.md)
