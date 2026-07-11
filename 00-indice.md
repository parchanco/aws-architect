# Índice y rutas de aprendizaje

Usa el [README](README.md) como guion principal y checklist. Este índice sirve para localizar contenidos o elegir una ruta según tu objetivo.

## Temario completo

| Bloque | Capítulos | Resultado |
|---|---|---|
| Fundamentos | [01](01-introduccion-aws.md) · [02](02-iam.md) | Entender la nube, responsabilidad compartida e identidad |
| Cómputo y alta disponibilidad | [03](03-ec2.md) · [04](04-ec2-storage.md) · [05](05-elb-asg.md) | Diseñar una capa de aplicación escalable y multi-AZ |
| Bases de datos y caché | [06](06-rds-aurora.md) · [07](07-elasticache.md) · [21](21-databases.md) | Elegir motores, modelos, réplicas y patrones de caché |
| DNS, red y entrega | [08](08-route53.md) · [12](12-cloudfront.md) · [27](27-vpc.md) | Conectar y publicar cargas de forma segura |
| Almacenamiento | [09](09-s3-intro.md) · [10](10-s3-advanced.md) · [11](11-s3-security.md) · [13](13-storage-extras.md) | Elegir almacenamiento y proteger su ciclo de vida |
| Integración y contenedores | [14](14-messaging.md) · [15](15-containers.md) | Desacoplar cargas y elegir una plataforma de ejecución |
| Arquitectura y herramientas | [16](16-best-practices.md) · [17](17-cli-commands.md) · [18](18-glosario.md) | Aplicar buenas prácticas y trabajar por CLI |
| Serverless | [19](19-serverless.md) · [20](20-serverless-architectures.md) | Diseñar aplicaciones dirigidas por eventos |
| Datos e IA/ML | [22](22-data-analytics.md) · [23](23-machine-learning.md) | Construir flujos de datos y reconocer casos de uso de ML |
| Operación y seguridad | [24](24-monitoring.md) · [25](25-iam-advanced.md) · [26](26-security.md) | Observar, auditar y proteger cargas |
| Continuidad | [28](28-disaster-recovery.md) | Definir RTO/RPO, recuperación y migraciones |
| Economía cloud | [29](29-finops-economia-cloud.md) | Controlar costes y tomar decisiones económicas |
| Automatización | [30](30-infraestructura-como-codigo.md) · [31](31-cicd-devops.md) | Desplegar infraestructura y software repetiblemente |
| Operaciones y gobierno | [32](32-operaciones-systems-manager.md) · [33](33-gobierno-multicuenta.md) | Operar flotas y organizar múltiples cuentas |
| Proyecto final | [34](34-proyecto-final.md) | Integrar diseño, despliegue, operación y documentación |

## Rutas por objetivo

### Cloud Practitioner / iniciación

01 → 02 → 03 → 06 → 08 → 09 → 14 → 16 → 24 → 29

Entrega: mapa de servicios, presupuesto, diagrama básico y explicación del modelo de responsabilidad compartida.

### Solutions Architect Associate

01–16 → 19–21 → 24–30 → 33–34

Prioriza seguridad, resiliencia, rendimiento y costes. La guía oficial SAA-C03 recomienda experiencia práctica; este repositorio es el mapa, no un sustituto de esa experiencia.

### Developer / aplicaciones cloud

01–02 → 09–11 → 14–15 → 17 → 19–20 → 24–26 → 30–31 → 34

Entrega: API event-driven con autenticación, observabilidad, pruebas y despliegue automatizado.

### SysOps / Cloud Operations

01–17 → 24–29 → 32–34

Entrega: panel, alarmas, inventario, parcheado, backup, runbook de incidente y simulacro de recuperación.

### DevOps

02 → 14–17 → 19–20 → 24–26 → 28–34

Entrega: infraestructura como código, pipeline con controles, despliegue progresivo, rollback y métricas DORA básicas.

### Security

02 → 09–11 → 24–27 → 30 → 32–34

Entrega: modelo de amenazas, mínimo privilegio, cifrado, registro centralizado y respuesta a incidentes.

### Data / Machine Learning

01–02 → 06–07 → 09–11 → 14 → 19 → 21–24 → 26 → 29–30 → 34

Entrega: ingesta, catálogo, transformación, consulta, controles de acceso y observabilidad de datos.

## Método de evaluación

Al terminar cada bloque, puntúate de 0 a 2:

- **0:** no puedo explicarlo;
- **1:** puedo explicarlo o seguir una guía;
- **2:** puedo implementarlo, probarlo y justificar alternativas.

No avances al proyecto final hasta obtener al menos 1 en todos los bloques principales. Para considerar dominada una ruta, alcanza 2 en sus bloques y presenta sus evidencias.

## Referencia de certificación

La versión SAA-C03 organiza el examen en cuatro dominios: arquitecturas seguras (30 %), resilientes (26 %), de alto rendimiento (24 %) y optimizadas en coste (20 %). Verifica estos datos y el listado vigente de servicios en la [guía oficial](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html).

---

**Última actualización:** julio de 2026 · **Versión:** 3.0
