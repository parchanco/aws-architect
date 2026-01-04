# 02 - IAM (Identity and Access Management)

## Resumen Ejecutivo
Sistema de autenticación y autorización de AWS. Gestiona quién puede hacer qué en tu cuenta AWS.

## Componentes Principales

### 1. Users
Identidades para personas o aplicaciones.
- **Credenciales**: Password (console) + Access Keys (CLI/API)
- **Best practice**: Un usuario físico = un usuario IAM

### 2. Groups
Colecciones de usuarios con permisos similares.
```
Developers → Permisos EC2, S3, RDS
DevOps → Full infrastructure
Auditors → Read-only access
```

### 3. Policies
Documentos JSON que definen permisos.

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::bucket/*"
}
```

### 4. Roles
Permisos para servicios AWS (no credenciales permanentes).

**Uso principal**: EC2, Lambda accediendo a otros servicios.

```python
# ✅ BIEN - EC2 con Role
s3 = boto3.client('s3')  # Usa Role automáticamente

# ❌ MAL - Hardcodear keys
s3 = boto3.client('s3', 
    aws_access_key_id='AKIA...',
    aws_secret_access_key='...')
```

## MFA (Multi-Factor Authentication)
Autenticación de dos factores.
- **Obligatorio**: Root account, usuarios con permisos elevados
- **Tipos**: Virtual (Google Authenticator), Hardware, U2F (YubiKey)

## AWS CLI

### Instalación
```bash
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Configuración
```bash
aws configure
# Access Key ID: AKIA...
# Secret Access Key: ...
# Region: eu-west-1
# Output: json
```

### Comandos Básicos
```bash
aws s3 ls
aws ec2 describe-instances
aws rds describe-db-instances
```

## Herramientas de Seguridad

### IAM Credentials Report
CSV con estado de todos los usuarios:
- MFA activado
- Access Keys antiguas
- Último login

### IAM Access Advisor
Muestra qué permisos ha usado realmente cada usuario/role.
**Aplicar Least Privilege**: Quitar permisos no usados.

## Best Practices

```
✅ NO usar root account (solo setup inicial)
✅ Activar MFA en root y usuarios admin
✅ Un usuario físico = un IAM user
✅ Asignar usuarios a grupos
✅ Usar Roles para servicios AWS
✅ Rotar Access Keys cada 90 días
✅ Aplicar Least Privilege
✅ Auditar con Credentials Report

❌ No compartir credenciales
❌ No poner Access Keys en código/Git
❌ No dar más permisos de los necesarios
```

## Setup IAM para Odoo

```
Root Account
└─ MFA activado, solo emergencias

Groups:
├─ Developers (EC2, S3, RDS read/write)
├─ DevOps (Full infrastructure)
└─ Finance (Billing access)

Users:
├─ pablo → [Developers, DevOps]
└─ maria → [Developers]

Roles:
├─ OdooProductionRole → EC2 instances
│  └─ Policies: S3 access, SES email, CloudWatch
└─ LambdaExecutionRole → Lambda functions
```

---

**Siguiente**: [03 - EC2 - Elastic Compute Cloud](03-ec2.md)
