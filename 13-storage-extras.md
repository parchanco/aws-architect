# 13 - AWS Storage Extras

## Snow Family

Dispositivos físicos para migrar data offline.

### Snowcone
```
Storage: 8-14 TB
Uso: Edge computing, data transfer
Tamaño: Pequeño (4.5 lbs)
```

### Snowball Edge
```
Storage: 80-210 TB
Compute: Opcional (EC2, Lambda)
Uso: Data migration, edge computing
```

### Snowmobile
```
Storage: 100 PB
Truck físico
Uso: Datacenter complete migration
```

**Cuándo usar**:
```
> 10 TB data + bandwidth limitado
Network migration = semanas/meses
Snow device = días
```

## FSx

Managed file systems.

### FSx for Windows File Server
```
SMB protocol
Active Directory integration
Windows-native
```

### FSx for Lustre
```
High Performance Computing
Linux
ML, analytics, video processing
Integrates with S3
```

**Para Odoo**: Normalmente usa EFS (Linux NFS).

## Storage Gateway

Hybrid cloud storage (on-premise + AWS).

### File Gateway
```
On-premise file shares backed by S3
NFS/SMB protocol
```

### Volume Gateway
```
iSCSI protocol
Block storage backed by S3
```

### Tape Gateway
```
Virtual tape library
Backup to S3/Glacier
```

---

**Siguiente**: [14 - Messaging](14-messaging.md)
