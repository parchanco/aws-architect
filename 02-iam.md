# 02 - IAM: identidades y permisos

IAM responde a dos preguntas: **quién o qué realiza una solicitud** y **qué puede hacer sobre qué recursos bajo qué condiciones**.

## Objetivos

- diferenciar identidad humana, workload, rol y recurso;
- usar credenciales temporales para consola, CLI y aplicaciones;
- interpretar una policy y su evaluación;
- aplicar mínimo privilegio y probar permisos;
- diseñar acceso seguro para una aplicación web.

## Identidades

### Personas

Para equipos, centraliza el acceso con IAM Identity Center y, si existe, un proveedor corporativo. Los usuarios reciben sesiones temporales mediante permission sets. Los IAM users con access keys permanentes deben ser una excepción heredada, no el punto de partida.

### Workloads

EC2, Lambda, ECS y pipelines asumen roles. El SDK obtiene y renueva credenciales automáticamente desde el entorno de ejecución.

```python
import boto3

# El SDK usa el role del workload; no hay claves en el código.
s3 = boto3.client("s3")
```

### Usuario raíz

Protege el root con MFA, no crees access keys y úsalo solo para tareas que lo exigen. Define un procedimiento de acceso de emergencia.

## Cómo se evalúa una autorización

```text
solicitud
  ├─ identidad y sesión
  ├─ identity policies
  ├─ resource policies
  ├─ permission boundary
  ├─ SCP de Organizations
  └─ condiciones y contexto

Denegación explícita > autorización explícita > denegación implícita
```

Una SCP o permission boundary establece un máximo: no concede acceso por sí sola.

## Anatomía de una policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "ReadReports",
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::company-reports/*",
    "Condition": {
      "Bool": {"aws:SecureTransport": "true"}
    }
  }]
}
```

- `Action`: operaciones permitidas o denegadas.
- `Resource`: ARN exacto cuando el servicio lo admite.
- `Condition`: restringe por contexto, etiquetas, red, organización o MFA.
- `Principal`: aparece en policies basadas en recursos y trust policies.

Evita `Action: "*"` y `Resource: "*"` salvo que la operación lo requiera y exista una justificación.

## Roles y trust policies

La permissions policy dice qué hace el rol. La trust policy dice quién puede asumirlo.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
```

Para acceso entre cuentas, limita el principal de origen y añade condiciones apropiadas. No confíes en toda una cuenta sin revisar qué identidades pueden asumir el rol.

## AWS CLI con credenciales temporales

```bash
# Configuración inicial con IAM Identity Center
aws configure sso

# Iniciar o renovar una sesión
aws sso login --profile sandbox

# Verificar la identidad activa antes de ejecutar cambios
aws sts get-caller-identity --profile sandbox

# Usar el perfil de forma explícita
aws s3 ls --profile sandbox
```

No pegues credenciales en scripts, historial del shell, tickets o archivos del repositorio.

## Mínimo privilegio

1. empieza por el caso de uso y los recursos exactos;
2. concede lectura y escritura por separado;
3. usa condiciones y etiquetas cuando aporten un límite real;
4. prueba accesos permitidos y denegados;
5. revisa CloudTrail, Access Advisor e IAM Access Analyzer;
6. elimina permisos que ya no se necesitan.

## Diseño para una aplicación web

```text
Personas ── IAM Identity Center ── roles de administración/lectura

ALB
 └─ ECS/EC2 role
     ├─ leer secreto concreto de Secrets Manager
     ├─ acceder a un prefijo concreto de S3
     └─ publicar métricas y logs

Pipeline role
 └─ desplegar solo stacks y servicios autorizados
```

Separa el rol de ejecución de la aplicación, el rol de despliegue y el acceso humano. No concedas permisos de base de datos o infraestructura a todos por comodidad.

## Laboratorio

1. Configura un perfil SSO de laboratorio.
2. Crea un bucket privado y un rol limitado a un prefijo.
3. Prueba `GetObject` en el prefijo permitido.
4. Demuestra que falla contra otro prefijo y al intentar borrar el bucket.
5. Consulta el evento en CloudTrail.
6. Usa Policy Simulator o Access Analyzer para revisar el alcance.

### Criterio de finalización

- [ ] No existen access keys de larga duración para el estudiante.
- [ ] Se conserva evidencia de una acción permitida y otra denegada.
- [ ] La policy limita acciones y recursos.
- [ ] La identidad activa se verificó con STS.
- [ ] Los recursos del laboratorio se eliminaron.

## Preguntas de repaso

1. ¿Qué diferencia hay entre permissions policy y trust policy?
2. ¿Por qué una SCP no concede permisos?
3. ¿Qué identidad usarías para una task de ECS?
4. ¿Qué gana un equipo al usar credenciales temporales?

**Siguiente:** [03 - EC2](03-ec2.md)
