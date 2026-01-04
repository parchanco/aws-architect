# 04 - EC2 Instance Storage

## Tipos de Almacenamiento

### Comparación General

| Tipo | Persistencia | Scope | Multi-Attach | Uso |
|------|--------------|-------|--------------|-----|
| **EBS** | Sí | AZ | No* | Boot, DB, App |
| **Instance Store** | No | Instance | No | Cache temporal |
| **EFS** | Sí | Regional | Sí | Shared files |

*Excepto io1/io2 multi-attach

## EBS (Elastic Block Store)

### Características
- Disco de red (latencia mínima)
- Persiste aunque pares/termines instancia
- Bound to AZ (no puedes mover entre AZs directamente)
- Provisioned capacity (GB + IOPS)

### Tipos de Volúmenes

**gp3 (General Purpose SSD)** - Recomendado 99% casos
```
IOPS: 3,000-16,000
Throughput: 125-1,000 MB/s
Size: 1 GB - 16 TB
Price: ~$0.08/GB-month
Uso: Boot, apps, DBs pequeñas-medianas
```

**io2 (Provisioned IOPS SSD)** - DBs críticas
```
IOPS: 100-64,000 (256k con Block Express)
Size: 4 GB - 16 TB
Price: ~$0.125/GB-month + IOPS
Uso: DBs producción alto tráfico
```

**st1 (Throughput HDD)** - Big Data
```
Throughput: 500 MB/s
IOPS: 500
Price: ~$0.045/GB-month
Uso: Data warehouses, logs
```

**sc1 (Cold HDD)** - Archival
```
Throughput: 250 MB/s
IOPS: 250
Price: ~$0.015/GB-month (más barato)
Uso: Backups, archival
```

### Para Odoo
```
Root volume: gp3 20 GB
PostgreSQL: gp3 100-500 GB (o io2 si alto tráfico)
Filestore: gp3 50-200 GB
```

### EBS Snapshots
Backup point-in-time en S3.

**Características**:
- Incrementales (solo cambios)
- Regionales (cross-AZ automático)
- Copy a otras regiones
- Crear AMI desde snapshot

**Estrategia backup Odoo**:
```
Daily: Últimos 7 días (DLM policy)
Weekly: Últimos 4 semanas
Monthly: Últimos 12 meses → Archive tier
```

### EBS Encryption
- At-rest: AWS KMS (AES-256)
- In-transit: Cifrado automático
- Snapshots cifrados automáticamente
- **Enable by default** en cuenta

## AMI (Amazon Machine Image)

Template de instancia completa: OS + software + config.

### Golden Image Strategy
```
AMI contiene:
✅ Ubuntu 20.04
✅ Odoo 11 instalado
✅ Dependencies
✅ Nginx configurado
✅ CloudWatch agent
✅ Security hardening

User Data al launch:
- Pull código desde Git
- Fetch secrets (Parameter Store)
- Configure odoo.conf
- Start services

Launch time: 2-3 minutos (vs 15 min desde cero)
```

### Versionado
```
odoo-11-prod-v1.0-20240103
│      │    │  │   │
│      │    │  │   └─ Date
│      │    │  └───── Version
│      │    └──────── Environment
│      └───────────── App version
└──────────────────── App name
```

## Instance Store
Disco físico en servidor EC2.

**Características**:
- ✅ Performance extremo (millones IOPS)
- ❌ Datos se pierden al stop/terminate/fail
- ❌ No snapshots

**Para Odoo**:
```
❌ NO usar para PostgreSQL data
❌ NO usar para filestore
✅ Podrías usar para cache temporal
```

## EFS (Elastic File System)
Filesystem compartido NFS.

### Arquitectura Multi-Server
```
┌──────────┐  ┌──────────┐
│ Odoo-1   │  │ Odoo-2   │
│ AZ-1a    │  │ AZ-1b    │
└────┬─────┘  └────┬─────┘
     │            │
     └──────┬─────┘
            │
      ┌─────▼──────┐
      │    EFS     │
      │ /filestore │
      └────────────┘

Attachments compartidos instantáneamente
```

### Performance Modes
- **General Purpose**: Low latency (para Odoo)
- **Max I/O**: Alto throughput (Big Data)

### Storage Classes
- **Standard**: ~$0.30/GB-month (acceso frecuente)
- **Infrequent Access**: ~$0.025/GB-month (>30 días sin acceso)

### Cuándo Usar
```
✅ EFS: Múltiples servidores Odoo (shared filestore)
✅ EBS: Single servidor o DB (mejor performance)
```

### Setup Odoo con EFS
```bash
# Install EFS utils
sudo apt install -y amazon-efs-utils

# Mount
sudo mount -t efs -o tls fs-xxx:/ /var/lib/odoo/filestore

# Auto-mount /etc/fstab
fs-xxx:/ /var/lib/odoo/filestore efs _netdev,tls 0 0

# odoo.conf
data_dir = /var/lib/odoo/filestore
```

---

**Siguiente**: [05 - Load Balancing & Auto Scaling](05-elb-asg.md)
