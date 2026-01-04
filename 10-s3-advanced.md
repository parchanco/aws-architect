# 10 - S3 Advanced Features

## Replication

### Cross-Region Replication (CRR)
```
Bucket eu-west-1 → Bucket us-east-1
Async replication

Uso:
- Disaster recovery
- Compliance
- Lower latency access
```

### Same-Region Replication (SRR)
```
Bucket-prod → Bucket-backup (misma región)

Uso:
- Log aggregation
- Replicación entre accounts
```

**Requisitos**:
- Versioning enabled (source y destination)
- IAM permissions
- Solo replica nuevos objects (no retroactivo)

## S3 Event Notifications

Trigger acciones cuando hay cambios en bucket.

```python
Events:
- s3:ObjectCreated:*
- s3:ObjectRemoved:*
- s3:ObjectRestore:*

Destinations:
- SNS Topic
- SQS Queue
- Lambda Function

Ejemplo: Upload PDF → Lambda procesa → Save to DB
```

### Para Odoo
```python
# Odoo upload invoice PDF a S3
# → S3 Event → Lambda
# → Lambda OCR extrae datos
# → Lambda llama API Odoo con datos
```

## S3 Performance

### Optimización Upload

**Multipart Upload**:
```
File > 100 MB → Split en partes
Upload partes en paralelo
S3 ensambla

Ventajas:
- Faster (paralelo)
- Recovery (retry solo partes fallidas)
- Obligatorio para files > 5 GB
```

**Transfer Acceleration**:
```
Upload a Edge Location (cerca usuario)
Edge → S3 por red privada AWS
Más rápido para uploads internacionales

URL: bucket.s3-accelerate.amazonaws.com
Extra cost: $0.04/GB
```

### Optimización Download

**Byte-Range Fetches**:
```python
# Download solo parte del archivo
# Header: Range: bytes=0-1023

Ventajas:
- Parallel downloads (diferentes ranges)
- Resume failed downloads
- Get solo metadata (primeros bytes)
```

### Performance Limits

```
Requests:
- 3,500 PUT/COPY/POST/DELETE por segundo por prefix
- 5,500 GET/HEAD por segundo por prefix

Prefix: Parte del key antes del filename
my-bucket/prefix1/file.txt → prefix1
my-bucket/prefix2/file.txt → prefix2

Para más performance: Usa múltiples prefixes
```

## S3 Select & Glacier Select

Query CSV/JSON directo en S3.

```python
# En vez de download completo → filter local
# Usa SQL para filtrar en S3

import boto3
s3 = boto3.client('s3')

response = s3.select_object_content(
    Bucket='my-bucket',
    Key='large-file.csv',
    Expression='SELECT * FROM S3Object WHERE amount > 1000',
    ExpressionType='SQL',
    InputSerialization={'CSV': {'FileHeaderInfo': 'USE'}},
    OutputSerialization={'CSV': {}}
)

# Retorna solo rows que cumplen condición
# Ahorro: Bandwidth, tiempo, costo
```

## S3 Batch Operations

Ejecutar operaciones en billones de objects.

```
Operations:
- Copy objects
- Invoke Lambda en cada object
- Replace tags
- Change ACLs
- Restore from Glacier

Ejemplo:
- Encrypt todos los objetos sin encryption
- Copy bucket-a → bucket-b (millones de files)
```

## S3 Storage Lens

Analytics y recommendations para usage.

```
Dashboard muestra:
- Storage metrics
- Cost optimization
- Data protection
- Access patterns

Free tier: Basic metrics
Paid: Advanced metrics + recommendations
```

---

**Siguiente**: [11 - S3 Security](11-s3-security.md)
