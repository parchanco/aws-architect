# 11 - S3 Security

## Encryption

### Server-Side Encryption (SSE)

**SSE-S3**:
```
AWS gestiona keys
AES-256
Header: x-amz-server-side-encryption: AES256
Más simple, menos control
```

**SSE-KMS**:
```
AWS KMS gestiona keys
Más control (audit trail en CloudTrail)
Rotation automática
Header: x-amz-server-side-encryption: aws:kms
```

**SSE-C**:
```
Tú gestionas keys
Envías key en cada request
AWS no almacena key
HTTPS obligatorio
```

### Client-Side Encryption
```
Cifras antes de upload
Descifras después de download
Tú gestionas todo
```

### Encryption in Transit
```
HTTPS/TLS
SSL/TLS endpoints:
- s3.amazonaws.com
- bucket.s3.amazonaws.com
```

### Default Encryption
```
Enable en bucket settings
→ Todos los nuevos objects cifrados automáticamente
→ Aunque client no especifique encryption
```

## Access Control

### Bucket Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123:role/OdooRole"},
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### IAM Policies
```json
{
  "Effect": "Allow",
  "Action": ["s3:ListBucket", "s3:GetObject"],
  "Resource": [
    "arn:aws:s3:::my-bucket",
    "arn:aws:s3:::my-bucket/*"
  ]
}
```

### ACLs (Access Control Lists)
```
Legacy, evitar usar
Usar Bucket Policies o IAM Policies
```

### Block Public Access

```
Settings:
☑ Block all public access
☑ Block public ACLs
☑ Ignore public ACLs
☑ Block public bucket policies
☑ Restrict public buckets

Enable siempre (salvo website público)
```

## S3 Access Points

Simplifica access management.

```
En vez de 1 bucket policy gigante:

Access Point 1: /finance
- Only Finance team
- Read/Write

Access Point 2: /marketing
- Only Marketing team
- Read-only

Access Point 3: /apps
- Odoo application
- Read/Write /attachments
```

## S3 Object Lock

WORM (Write Once Read Many).

```
Modes:
- Governance: Usuarios con permisos pueden delete
- Compliance: NADIE puede delete (ni root)

Retention:
- Retain until date: 2030-01-01
- Legal hold: Indefinido hasta remove hold

Uso: Compliance (regulatorio)
```

## Pre-Signed URLs

Acceso temporal a objects privados.

```python
import boto3
s3 = boto3.client('s3')

url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'file.pdf'},
    ExpiresIn=3600  # 1 hora
)

# URL temporal válida 1 hora
# Después expira
```

### Para Odoo
```python
# Generar link descarga temporal para cliente
def get_attachment_url(self):
    s3 = boto3.client('s3')
    url = s3.generate_presigned_url(
        'get_object',
        Params={
            'Bucket': 'odoo-attachments',
            'Key': self.s3_key
        },
        ExpiresIn=300  # 5 minutos
    )
    return url
```

## CORS (Cross-Origin Resource Sharing)

Permite access desde web browsers.

```json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["https://odoo.tuempresa.com"],
      "AllowedMethods": ["GET", "PUT", "POST"],
      "AllowedHeaders": ["*"],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

## MFA Delete

Require MFA para delete objects.

```
Activar:
- Versioning: Enabled
- MFA Delete: Enabled

Para delete object → Necesitas MFA code
Extra protección contra delete accidental
```

## Access Logs

Log todas las requests a bucket.

```
Source bucket: my-bucket
Target bucket: my-bucket-logs

Logs contienen:
- Timestamp
- IP source
- Operation (GET, PUT, DELETE)
- Object key
- HTTP status
- Error code
```

## S3 Security Best Practices

```
✅ Enable encryption by default
✅ Block public access (salvo necesario)
✅ Use IAM roles (no Access Keys)
✅ Enable versioning (protection contra delete)
✅ Enable MFA delete (critical buckets)
✅ Use S3 Access Points (simplifica policies)
✅ Monitor con CloudWatch y CloudTrail
✅ Lifecycle policies (delete old data)
✅ Pre-signed URLs para temporary access
✅ CORS solo origins específicos

❌ No hacer buckets públicos sin razón
❌ No poner secrets en objects names/metadata
❌ No usar ACLs (legacy)
```

---

**Siguiente**: [12 - CloudFront](12-cloudfront.md)
