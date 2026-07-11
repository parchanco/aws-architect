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
