# 01 - Introducción a AWS

## Objetivos

- explicar regiones, Availability Zones y edge locations;
- reconocer el modelo de responsabilidad compartida;
- elegir región con criterios técnicos, legales y económicos;
- preparar una cuenta de laboratorio con controles de coste y seguridad;
- distinguir servicio administrado de responsabilidad delegada.

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

### AWS Free Tier

El programa depende de la fecha de creación de la cuenta. Las cuentas nuevas creadas después del 15 de julio de 2025 pueden elegir un plan gratuito o de pago y reciben créditos promocionales; las cuentas anteriores conservan las reglas heredadas.

> [!WARNING]
> “Free Tier” no significa que toda la cuenta sea gratuita. Algunos servicios o configuraciones no están incluidos y el exceso de uso se factura. Comprueba siempre la [documentación vigente](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html).

Antes del primer laboratorio:

1. configura un AWS Budget y alertas;
2. revisa los créditos y su caducidad;
3. etiqueta recursos con `Environment=lab` y `Owner`;
4. elimina los recursos al terminar;
5. comprueba Cost Explorer durante los días posteriores.

### Modelos de Pago
- **Pay-as-you-go**: Pagas solo lo que usas
- **Compromisos de uso**: Savings Plans y Reserved Instances pueden reducir costes cuando la demanda es estable
- **Pay less when you use more**: Volume discounts
- **Pay even less as AWS grows**: Precios bajan con tiempo

### Calculadora de Costos
Usa [AWS Pricing Calculator](https://calculator.aws/) antes de desplegar. Una estimación debe indicar región, fecha, horas de uso, transferencia, almacenamiento, copias de seguridad y crecimiento esperado. Evita memorizar precios: cambian por región, arquitectura y fecha.

## Soporte AWS

### Planes

AWS ofrece distintos planes y niveles de respuesta. Como nombres, precios y prestaciones pueden cambiar, compara requisitos de producción, tiempo de respuesta, canales y acompañamiento técnico en la [página oficial de AWS Support](https://aws.amazon.com/premiumsupport/plans/).

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
✅ Centralizar acceso humano con IAM Identity Center y credenciales temporales
✅ Configurar billing alerts
✅ Usar tags para organizar recursos
✅ Confirmar créditos, límites y presupuesto antes de practicar
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

## Laboratorio: preparar la cuenta

1. Protege el usuario raíz con MFA y revisa sus métodos de recuperación.
2. Configura acceso humano temporal mediante IAM Identity Center.
3. Crea un presupuesto y alertas al 50 %, 80 % y 100 %.
4. Elige región y documenta latencia, servicios, cumplimiento y coste.
5. Crea un recurso etiquetado, localízalo en facturación y elimínalo.
6. Comprueba que no quedan recursos activos en otras regiones.

### Criterio de finalización

- [ ] Root no tiene access keys y no se usa para trabajo diario.
- [ ] El acceso de laboratorio utiliza credenciales temporales.
- [ ] Presupuesto, alertas, región y etiquetas están documentados.
- [ ] La limpieza de recursos se ha verificado.

## Preguntas de repaso

1. ¿Qué diferencia hay entre región y Availability Zone?
2. ¿Qué responsabilidades mantiene el cliente al usar un servicio administrado?
3. ¿Por qué la región más cercana no siempre es la mejor elección?
4. ¿Qué controles aplicarías antes de crear el primer recurso?

---

**Siguiente**: [02 - IAM - Identity and Access Management](02-iam.md)
