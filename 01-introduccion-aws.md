# 01 - Introducción a AWS

## Conceptos Básicos

### Regiones y Availability Zones

**Región (Region)**
- Ubicación geográfica donde AWS tiene infraestructura
- Ejemplos: `eu-west-1` (Irlanda), `us-east-1` (Virginia)
- Completamente independientes entre sí
- Selección basada en: latencia, compliance, servicios disponibles

**Availability Zone (AZ)**
- Datacenter aislado dentro de una región
- Una región tiene 3-6 AZs
- Físicamente separadas (varios km de distancia)
- Conectadas con red de alta velocidad
- Protección contra desastres

```
Region: eu-west-1 (Irlanda)
├─ eu-west-1a (Datacenter 1)
├─ eu-west-1b (Datacenter 2)
└─ eu-west-1c (Datacenter 3)
```

**Edge Locations**
- Puntos de presencia para CDN (CloudFront)
- Más de 400 edge locations globalmente
- Cachean contenido cerca de usuarios finales

### Infraestructura Global AWS

```
32+ Regiones
├─ 102+ Availability Zones  
└─ 400+ Edge Locations

Ejemplo multi-región:
Region eu-west-1 (primary)
└─ Aplicación principal

Region us-east-1 (DR)
└─ Disaster recovery
```

## Servicios AWS Principales

### Compute
- **EC2**: Servidores virtuales
- **Lambda**: Serverless compute
- **Elastic Beanstalk**: PaaS
- **ECS/EKS**: Contenedores

### Storage
- **S3**: Object storage
- **EBS**: Block storage (discos EC2)
- **EFS**: File system compartido
- **Glacier**: Archival

### Database
- **RDS**: Bases datos relacionales gestionadas
- **DynamoDB**: NoSQL
- **ElastiCache**: Redis/Memcached
- **Aurora**: DB propietaria AWS (PostgreSQL/MySQL compatible)

### Networking
- **VPC**: Red privada virtual
- **Route 53**: DNS
- **CloudFront**: CDN
- **Direct Connect**: Conexión dedicada on-premise

### Security & Identity
- **IAM**: Gestión usuarios y permisos
- **KMS**: Gestión claves cifrado
- **Secrets Manager**: Gestión secretos
- **WAF**: Web Application Firewall

## Modelos de Servicio

### IaaS (Infrastructure as a Service)
```
AWS proporciona: Hardware virtualizado
Tú gestionas: OS, runtime, aplicación

Ejemplo: EC2
```

### PaaS (Platform as a Service)
```
AWS proporciona: Hardware, OS, runtime
Tú gestionas: Aplicación

Ejemplo: Elastic Beanstalk, RDS
```

### SaaS (Software as a Service)
```
AWS proporciona: Todo
Tú usas: La aplicación

Ejemplo: AWS Console, WorkMail
```

## Consola AWS

### Acceso
1. **AWS Management Console**: Interfaz web
2. **AWS CLI**: Línea de comandos
3. **AWS SDK**: Programático (boto3, etc.)
4. **AWS CloudShell**: Terminal en navegador

### Navegación Console
```
Services → EC2 → Instances
Services → RDS → Databases  
Services → S3 → Buckets
Services → IAM → Users

Barra búsqueda: Buscar servicios rápidamente
```

## Facturación

### Free Tier
```
12 meses gratis (desde creación cuenta):
- EC2: 750 horas t2.micro/mes
- S3: 5 GB storage
- RDS: 750 horas db.t2.micro/mes
- CloudFront: 50 GB data transfer

Siempre gratis:
- Lambda: 1M requests/mes
- DynamoDB: 25 GB storage
- CloudWatch: 10 custom metrics
```

### Modelos de Pago
- **Pay-as-you-go**: Pagas solo lo que usas
- **Save when you reserve**: Reserved Instances (40-75% descuento)
- **Pay less when you use more**: Volume discounts
- **Pay even less as AWS grows**: Precios bajan con tiempo

### Calculadora de Costos
```
https://calculator.aws/

Ejemplo estimación Odoo:
- EC2 t3.large: ~$60/mes
- RDS db.t3.large: ~$120/mes
- EBS 100GB: ~$10/mes
- ALB: ~$20/mes
Total: ~$210/mes (aproximado)
```

## Soporte AWS

### Planes
1. **Basic**: Gratis (solo documentación)
2. **Developer**: $29/mes (email support)
3. **Business**: $100+/mes (24/7 phone/chat)
4. **Enterprise**: $15,000+/mes (TAM dedicado)

### Trusted Advisor
- Optimización de costos
- Performance
- Security
- Fault tolerance
- Service limits

Acceso según plan de soporte.

## Regiones para España/Europa

### Recomendadas
```
Primary: eu-west-1 (Irlanda)
- Más servicios disponibles
- Menor latency España (~20-30ms)
- Más barata que otras EU

Backup: eu-west-3 (París)
- Latency similar
- Sovereignty si necesitas data en Francia

DR: eu-central-1 (Frankfurt)
- Alternativa robusta
- Latency aceptable (~40ms)
```

## Mejores Prácticas Iniciales

```
✅ Activar MFA en root account
✅ Crear IAM users (no usar root)
✅ Configurar billing alerts
✅ Usar tags para organizar recursos
✅ Empezar con Free Tier
✅ Leer documentación AWS
✅ Practicar en sandbox/dev primero
✅ Implementar backups desde día 1
✅ Usar CloudWatch para monitoring

❌ No usar root account diariamente
❌ No exponer access keys en Git
❌ No dejar recursos corriendo sin usar
❌ No crear recursos sin tags
❌ No ignorar security groups
```

## Recursos de Aprendizaje

- **AWS Training**: https://www.aws.training/
- **AWS Documentation**: https://docs.aws.amazon.com/
- **AWS Well-Architected**: Framework mejores prácticas
- **AWS Blog**: Novedades y casos de uso
- **AWS re:Invent**: Conferencia anual (videos gratis)

---

**Siguiente**: [02 - IAM - Identity and Access Management](02-iam.md)
