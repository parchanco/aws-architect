# 31 - CI/CD para aplicaciones AWS

La entrega continua reduce el riesgo convirtiendo cada cambio en un proceso pequeño, probado, observable y reversible.

## Objetivos

- diseñar etapas de integración y entrega continua;
- separar compilación, artefacto, configuración y despliegue;
- usar identidades temporales y mínimo privilegio;
- comparar rolling, blue/green y canary;
- automatizar verificaciones y rollback.

## Pipeline de referencia

```text
commit
  └─ lint + unit tests + secret scan
       └─ build once + firma/SBOM
            └─ deploy dev + integration tests
                 └─ aprobación de riesgo
                      └─ canary prod + métricas
                           └─ promover o rollback
```

El mismo artefacto inmutable se promueve entre entornos. Las variables y secretos se inyectan en tiempo de ejecución mediante servicios adecuados, no se recompila para cada entorno.

## Servicios y decisiones

- **CodeBuild:** ejecutar builds administrados.
- **CodePipeline:** orquestar etapas y aprobaciones.
- **CodeDeploy:** automatizar despliegues en EC2, Lambda o ECS según el caso.
- **ECR:** almacenar y analizar imágenes de contenedor.
- **CloudFormation/CDK:** desplegar infraestructura junto al cambio.
- **GitHub Actions u otro CI:** válido si el equipo ya lo opera; usa OIDC para obtener credenciales temporales.

## Estrategias de despliegue

| Estrategia | Ventaja | Riesgo o coste |
|---|---|---|
| Rolling | Uso eficiente de capacidad | Conviven versiones; rollback gradual |
| Blue/green | Cambio y rollback rápidos | Capacidad duplicada temporalmente |
| Canary | Limita el impacto inicial | Requiere métricas y routing fiables |
| Todo a la vez | Sencillo | Alto radio de impacto; evitar en cargas críticas |

## Puertas de calidad

Una aprobación manual no compensa la falta de automatización. Añade controles según el riesgo:

- pruebas unitarias, integración y smoke;
- análisis de dependencias, secretos e IaC;
- política de cobertura razonable, no una cifra vacía;
- compatibilidad de migraciones de base de datos;
- alarmas de errores, latencia y saturación;
- rollback automático basado en señales útiles.

## GitHub Actions con OIDC

OIDC permite que GitHub solicite una sesión temporal de AWS sin guardar access keys como secretos. El rol de AWS debe confiar únicamente en el repositorio, rama o environment necesarios.

Trust policy simplificada:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
      },
      "StringLike": {
        "token.actions.githubusercontent.com:sub": "repo:example/orders:environment:production"
      }
    }
  }]
}
```

Workflow mínimo:

```yaml
name: deploy
on:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/orders-deploy
          aws-region: eu-west-1

      - run: ./scripts/test.sh
      - run: ./scripts/deploy.sh
      - run: ./scripts/smoke-test.sh
```

Fija actions de terceros por commit SHA cuando el riesgo lo requiera, limita permisos de `GITHUB_TOKEN`, protege environments y separa el rol que valida del que despliega.

## Artefactos y cadena de suministro

- construye una vez y asigna digest o checksum;
- genera inventario de dependencias/SBOM;
- analiza dependencias, imágenes e IaC;
- firma o verifica artefactos según el modelo de amenaza;
- conserva procedencia: commit, workflow, builder y parámetros;
- promueve por digest, nunca reconstruyas para producción;
- bloquea el despliegue de vulnerabilidades críticas según una política con excepciones auditadas.

## Migraciones compatibles

Una aplicación rolling o canary convive temporalmente con dos versiones. Usa expand/contract:

1. añade esquema compatible con ambas versiones;
2. despliega código que escriba o lea de forma compatible;
3. migra y verifica datos;
4. retira dependencias antiguas en otro despliegue.

Evita renombrar o borrar columnas en el mismo cambio que deja de usarlas. El rollback de código no revierte automáticamente una migración destructiva.

## Rollback guiado por señales

Define antes del despliegue:

- métricas y umbrales de error, latencia y negocio;
- ventana de observación;
- porcentaje de tráfico inicial;
- condiciones de pausa y rollback;
- comportamiento de migraciones y mensajes producidos;
- persona o equipo responsable.

Prueba el mecanismo periódicamente. Un botón de rollback que nunca se ha ejecutado es una hipótesis.

## Laboratorio

Construye un pipeline para una Lambda, contenedor o aplicación web:

1. Ejecuta tests en cada pull request.
2. Genera un artefacto versionado una sola vez.
3. Despliega infraestructura en `dev` mediante IaC.
4. Ejecuta una prueba de humo.
5. Promueve a otro entorno con una condición explícita.
6. Introduce un fallo controlado y demuestra el rollback.

### Criterio de finalización

- [ ] No hay claves de larga duración en CI.
- [ ] Cada ejecución puede trazarse hasta un commit.
- [ ] Un fallo detiene la promoción.
- [ ] El rollback ha sido probado, no solo documentado.
- [ ] Los logs del pipeline permiten diagnosticar el error sin exponer secretos.

## Métricas de mejora

Observa frecuencia de despliegue, tiempo desde commit hasta producción, tasa de cambios fallidos y tiempo de recuperación. Úsalas para mejorar el sistema, no para comparar personas.

## Preguntas de repaso

1. ¿Por qué se debe construir una vez y promover el mismo artefacto?
2. ¿Qué señal usarías para abortar un canary?
3. ¿Cómo harías reversible una migración de esquema?
4. ¿Qué permisos necesita realmente el rol de despliegue?

**Anterior:** [30 - Infraestructura como código](30-infraestructura-como-codigo.md)  
**Siguiente:** [32 - Operaciones](32-operaciones-systems-manager.md)
