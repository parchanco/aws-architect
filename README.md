# AWS Architect: ruta práctica de aprendizaje

Temario abierto en español para aprender AWS desde los fundamentos hasta arquitectura, desarrollo, operaciones, DevOps, seguridad y datos. El objetivo no es memorizar servicios: es aprender a elegirlos, desplegarlos, operarlos y justificar las decisiones.

> [!IMPORTANT]
> Crear recursos en AWS puede generar costes. Usa presupuestos y alertas, revisa el precio de cada servicio y elimina los recursos al terminar cada práctica. No subas credenciales al repositorio.

## Qué vas a conseguir

Al completar el itinerario podrás:

- diseñar cargas seguras, resilientes, eficientes y optimizadas en coste;
- desplegar infraestructura reproducible y aplicaciones con entrega continua;
- observar, operar y recuperar sistemas ante fallos;
- elegir servicios de cómputo, red, almacenamiento, bases de datos, integración y datos;
- explicar compromisos técnicos con diagramas y registros de decisiones;
- preparar AWS Certified Solutions Architect - Associate con práctica real.

El temario toma como marco los seis pilares de [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html): excelencia operativa, seguridad, fiabilidad, eficiencia del rendimiento, optimización de costes y sostenibilidad.

## Cómo seguir el curso

Trabaja en ciclos cortos: **leer → practicar → explicar → limpiar**.

1. Lee el capítulo y anota las decisiones, no solo las definiciones.
2. Completa una práctica pequeña en una cuenta de laboratorio.
3. Explica el resultado con tus propias palabras o mediante un diagrama.
4. Comprueba costes, elimina recursos y marca la casilla.
5. Al terminar cada etapa, completa su reto y revisa tus lagunas.

Ritmo sugerido: 4–6 horas semanales durante 12–16 semanas. Si ya trabajas con cloud, usa las rutas por rol de [00-indice.md](00-indice.md).

## Checklist de progreso

### Etapa 0 — Preparar un entorno seguro

- [ ] Crear o elegir una cuenta de laboratorio; no usar la cuenta de producción.
- [ ] Proteger el usuario raíz con MFA y no crear access keys para él.
- [ ] Configurar AWS CLI con credenciales temporales o IAM Identity Center.
- [ ] Crear un AWS Budget y una alerta de facturación.
- [ ] Elegir una región principal y documentar por qué.
- [ ] Preparar un repositorio privado para los ejercicios, sin secretos.

### Etapa 1 — Fundamentos y arquitectura base

- [ ] [01 · Introducción a AWS](01-introduccion-aws.md)
- [ ] [02 · IAM](02-iam.md)
- [ ] [03 · EC2](03-ec2.md)
- [ ] [04 · Almacenamiento de EC2](04-ec2-storage.md)
- [ ] [05 · ELB y Auto Scaling](05-elb-asg.md)
- [ ] [06 · RDS y Aurora](06-rds-aurora.md)
- [ ] [07 · ElastiCache](07-elasticache.md)
- [ ] [08 · Route 53](08-route53.md)
- [ ] **Reto:** dibujar una aplicación web en dos AZ y justificar cada componente.

### Etapa 2 — Almacenamiento, integración y entrega

- [ ] [09 · Fundamentos de S3](09-s3-intro.md)
- [ ] [10 · S3 avanzado](10-s3-advanced.md)
- [ ] [11 · Seguridad en S3](11-s3-security.md)
- [ ] [12 · CloudFront y Global Accelerator](12-cloudfront.md)
- [ ] [13 · Servicios de almacenamiento adicionales](13-storage-extras.md)
- [ ] [14 · SQS, SNS y streaming](14-messaging.md)
- [ ] [15 · Contenedores](15-containers.md)
- [ ] **Reto:** publicar contenido privado mediante CloudFront y desacoplar una tarea con SQS.

### Etapa 3 — Diseñar y automatizar

- [ ] [16 · Buenas prácticas y costes](16-best-practices.md)
- [ ] [17 · AWS CLI](17-cli-commands.md)
- [ ] [19 · Serverless](19-serverless.md)
- [ ] [20 · Arquitecturas serverless](20-serverless-architectures.md)
- [ ] [29 · FinOps y economía cloud](29-finops-economia-cloud.md)
- [ ] [30 · Infraestructura como código](30-infraestructura-como-codigo.md)
- [ ] [31 · CI/CD para aplicaciones AWS](31-cicd-devops.md)
- [ ] **Reto:** desplegar por código una API pequeña con pipeline y rollback documentado.

### Etapa 4 — Datos, seguridad y operaciones

- [ ] [21 · Bases de datos](21-databases.md)
- [ ] [22 · Datos y analítica](22-data-analytics.md)
- [ ] [23 · Machine learning](23-machine-learning.md)
- [ ] [24 · Observabilidad y auditoría](24-monitoring.md)
- [ ] [25 · IAM avanzado](25-iam-advanced.md)
- [ ] [26 · Seguridad y cifrado](26-security.md)
- [ ] [27 · Redes VPC](27-vpc.md)
- [ ] [28 · Recuperación y migraciones](28-disaster-recovery.md)
- [ ] [32 · Operaciones con Systems Manager](32-operaciones-systems-manager.md)
- [ ] [33 · Gobierno multi-cuenta](33-gobierno-multicuenta.md)
- [ ] **Reto:** detectar un fallo, responder con un runbook y demostrar la recuperación.

### Etapa 5 — Demostrar lo aprendido

- [ ] [34 · Proyecto final](34-proyecto-final.md)
- [ ] Completar el [glosario](18-glosario.md) con al menos 20 términos propios.
- [ ] Realizar una revisión Well-Architected del proyecto.
- [ ] Entregar diagrama, ADR, repositorio, panel operativo, runbook y estimación de costes.
- [ ] Poder explicar tres decisiones y sus alternativas sin consultar apuntes.

## Evidencias de aprendizaje

No marques una etapa solo por haberla leído. Guarda para cada reto:

- un diagrama de arquitectura;
- el código o los comandos reproducibles;
- una captura o consulta que pruebe que funciona;
- una estimación de coste y una lista de recursos eliminados;
- un breve ADR: contexto, decisión, alternativas y consecuencias.

## Recursos oficiales

- [AWS Skill Builder](https://aws.amazon.com/training/digital/) para formación y laboratorios oficiales.
- [Guía oficial SAA-C03](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html) para comprobar el alcance actual del examen.
- [AWS Architecture Center](https://aws.amazon.com/architecture/) para patrones y arquitecturas de referencia.
- [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) para evaluar cargas.
- [AWS Pricing Calculator](https://calculator.aws/) para estimar costes antes de desplegar.

## Alcance y contribuciones

Los ejemplos parten de aplicaciones web y, cuando aporta valor, de Odoo, pero los patrones son generales. Los servicios, límites y exámenes de AWS cambian: contrasta siempre la documentación oficial y abre una propuesta si encuentras contenido desactualizado.

Este repositorio ayuda a aprender y practicar; no garantiza aprobar una certificación ni sustituye experiencia real.

---

**Versión:** 3.0 · **Última revisión del itinerario:** julio de 2026
