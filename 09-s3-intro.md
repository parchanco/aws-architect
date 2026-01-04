# 09 - S3 (Simple Storage Service)

Object storage para la nube.

## Conceptos Básicos

### Buckets
Container para objects (archivos).
```
Nombre: Globalmente único
Ejemplo: scalpers-odoo-prod-attachments
Region-specific: eu-west-1
```

### Objects
Archivos almacenados.
```
Key (path): invoices/2024/INV-001.pdf
Value: Binary data (archivo)
Max size: 5 TB
```

### Estructura
```
my-bucket/
├─ invoices/
│  ├─ 2024/
│  │  ├─ INV-001.pdf
│  │  └─ INV-002.pdf
│  └─ 2023/
├─ attachments/
└─ documents/
```

## Características

```
✅ Durability: 99.999999999% (11 nines)
✅ Availability: 99.99%
✅ Unlimited storage
✅ Versioning
✅ Encryption
✅ Lifecycle policies
✅ Static website hosting
```

## Storage Classes

### S3 Standard
```
Uso: Datos frecuentemente accedidos
Durability: 11 nines
Availability: 99.99%
Precio: ~$0.023/GB-month
```

### S3 Intelligent-Tiering
```
Auto-mueve entre tiers según acceso
Precio: ~$0.023/GB + $0.0025/1000 objects
```

### S3 Standard-IA (Infrequent Access)
```
Uso: Datos accedidos < 1 vez/mes
Retrieval cost: Por GB
Precio: ~$0.0125/GB-month
```

### S3 Glacier
```
Uso: Archival, backups long-term
Retrieval: Minutos a horas
Precio: ~$0.004/GB-month
```

### S3 Glacier Deep Archive
```
Uso: Compliance (7-10 años)
Retrieval: 12-48 horas
Precio: ~$0.00099/GB-month (más barato)
```

## Comandos Básicos

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
aws s3 sync ./local-dir s3://my-bucket/prefix/

# Delete
aws s3 rm s3://my-bucket/file.pdf
```

## S3 para Odoo

### Attachments en S3

```python
# Custom addon: s3_storage
import boto3

class IrAttachment(models.Model):
    _inherit = 'ir.attachment'
    
    def _file_write(self, value, checksum):
        s3 = boto3.client('s3')
        bucket = 'odoo-prod-attachments'
        key = f'attachments/{self.id}/{self.name}'
        
        s3.put_object(
            Bucket=bucket,
            Key=key,
            Body=value,
            ServerSideEncryption='AES256'
        )
        return key
    
    def _file_read(self, fname):
        s3 = boto3.client('s3')
        bucket = 'odoo-prod-attachments'
        
        response = s3.get_object(Bucket=bucket, Key=fname)
        return response['Body'].read()
```

### Ventajas S3 para Odoo
```
✅ Almacenamiento ilimitado
✅ No necesitas resize volúmenes
✅ Backups automáticos (durability 11 nines)
✅ Lifecycle policies (mover a Glacier)
✅ Más barato que EBS para archival
✅ CDN fácil con CloudFront

vs EFS:
S3: $0.023/GB-month
EFS: $0.30/GB-month
```

## Versioning

Mantiene múltiples versiones de objects.

```python
Enable versioning:
aws s3api put-bucket-versioning     --bucket my-bucket     --versioning-configuration Status=Enabled

Subir file.txt → Version 1
Subir file.txt → Version 2
Subir file.txt → Version 3

Puedes restaurar cualquier versión
Delete no borra (marca delete marker)
```

## Lifecycle Policies

Automatiza transición entre storage classes.

```python
Rule: Move old attachments
- After 30 days → S3 Standard-IA
- After 90 days → S3 Glacier
- After 365 days → S3 Glacier Deep Archive
- After 2555 days (7 años) → Delete

Ahorro: ~70-80% en storage costs
```

---

**Siguiente**: [10 - S3 Advanced](10-s3-advanced.md)
