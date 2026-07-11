# 15 - Contenedores en AWS

Un contenedor empaqueta la aplicación; no resuelve por sí solo networking, identidad, secretos, almacenamiento, despliegue u observabilidad.

## Objetivos

- elegir entre ECS, EKS y App Runner según el caso;
- diferenciar task definition, task, service y cluster;
- ejecutar contenedores con roles y red seguros;
- construir y promover imágenes inmutables;
- desplegar, escalar, observar y recuperar un servicio.

## Elegir plataforma

| Plataforma | Encaja cuando |
|---|---|
| ECS con Fargate | Quieres orquestación AWS sin administrar hosts |
| ECS sobre EC2 | Necesitas control de instancias, capacidad especial o alta utilización estable |
| EKS | La organización ya opera Kubernetes o necesita su ecosistema y APIs |
| App Runner | Quieres publicar rápidamente un servicio web con poca configuración |

No elijas EKS solo por portabilidad: evalúa operación del cluster, upgrades, red, seguridad y skills del equipo.

## Conceptos de ECS

- **Task definition:** plantilla versionada de contenedores, CPU, memoria, puertos, roles y configuración.
- **Task:** ejecución de una revisión concreta.
- **Service:** mantiene el número deseado, integra balanceo, health checks y despliegues.
- **Cluster:** ámbito lógico de capacidad y servicios.
- **Capacity provider:** estrategia para Fargate, Fargate Spot o Auto Scaling Groups.

### Execution role y task role

- **Execution role:** ECS lo usa para descargar la imagen, escribir logs u obtener configuración inicial.
- **Task role:** la aplicación lo usa para llamar APIs AWS.

No concedas permisos de negocio al execution role ni reutilices un rol amplio para todos los servicios.

## Networking

Con `awsvpc`, cada task obtiene una interfaz de red y security groups. En producción:

```text
Internet → ALB público → tasks en subnets privadas
                         ├─ RDS mediante SG de aplicación
                         ├─ Secrets Manager por endpoint/NAT
                         └─ logs y métricas
```

El target group debe comprobar una ruta de salud que refleje preparación, no solo que el proceso existe. Configura deregistration delay y graceful shutdown para terminar peticiones antes de detener una task.

## Imágenes con Amazon ECR

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=eu-west-1
REPOSITORY=orders
TAG=$(git rev-parse --short HEAD)

aws ecr get-login-password --region "$REGION" |
  docker login --username AWS --password-stdin \
  "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com"

docker build --pull -t "$REPOSITORY:$TAG" .
docker tag "$REPOSITORY:$TAG" \
  "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPOSITORY:$TAG"
docker push "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPOSITORY:$TAG"
```

Usa tags inmutables o, mejor, promueve por digest. Activa análisis de vulnerabilidades, lifecycle y bloqueo de imágenes críticas. Evita `latest` en producción.

## Configuración y secretos

- variables no sensibles: task definition, AppConfig o Parameter Store;
- secretos: Secrets Manager o Parameter Store cifrado según ciclo de vida;
- nunca secretos en imagen, Dockerfile, argumentos de build o repositorio;
- rota y limita el acceso al secreto exacto;
- recuerda que una task puede necesitar reinicio para recibir una versión nueva.

## Estado y almacenamiento

Mantén las tasks reemplazables. Guarda sesiones y datos persistentes en servicios externos. EFS puede aportar almacenamiento compartido, pero añade latencia, coste y semántica de filesystem; no lo uses para evitar rediseñar estado sin medir.

## Autoscaling y despliegue

Escala con señales relacionadas con demanda: CPU/memoria, peticiones por target o backlog por task. Establece mínimos compatibles con disponibilidad multi-AZ.

Estrategias:

- rolling con porcentajes mínimo/máximo;
- blue/green para cambio y rollback rápidos;
- canary cuando routing y observabilidad permitan limitar impacto.

Una nueva revisión no está lista hasta superar health checks y métricas de negocio.

## Observabilidad y operación

- logs estructurados con correlation ID;
- métricas de tráfico, errores, latencia y saturación;
- Container Insights u OpenTelemetry cuando aporten valor;
- ECS Exec solo con autorización, auditoría y acceso temporal;
- alarmas enlazadas a runbooks;
- límites de CPU/memoria probados bajo carga.

## EKS: profundidad mínima

Si eliges EKS, debes diseñar además namespaces, RBAC, Pod Identity/IRSA, network policies, ingress, almacenamiento CSI, autoscaling, upgrades, add-ons y seguridad de admisión. El control plane administrado no elimina la operación de Kubernetes.

## Consideraciones para Odoo

- base de datos externa en RDS PostgreSQL;
- filestore persistente probado bajo la concurrencia esperada;
- sesiones compartidas solo mediante integración compatible con la versión;
- migraciones ejecutadas como job controlado, no por todas las replicas;
- workers, límites y timeouts medidos;
- imágenes propias parcheadas y versionadas.

## Laboratorio: servicio ECS

1. Construye una API sin secretos en la imagen.
2. Publica la imagen en ECR con tag de commit.
3. Despliega un ECS service en subnets privadas detrás de ALB.
4. Asigna task role de mínimo privilegio y logs centralizados.
5. Configura autoscaling y realiza una pequeña prueba de carga.
6. Despliega una versión defectuosa y demuestra rollback.

### Criterio de finalización

- [ ] Las tasks no tienen IP pública.
- [ ] Task role y execution role están separados.
- [ ] El servicio usa una imagen inmutable y analizada.
- [ ] Health checks, logs, métricas y rollback funcionan.
- [ ] El entorno puede recrearse y eliminarse mediante IaC.

## Preguntas de repaso

1. ¿Qué diferencia hay entre task role y execution role?
2. ¿Cuándo elegirías ECS sobre EC2 frente a Fargate?
3. ¿Por qué un health check de proceso puede ser insuficiente?
4. ¿Qué coste operativo adicional introduce EKS?

**Siguiente:** [16 - Buenas prácticas](16-best-practices.md)
