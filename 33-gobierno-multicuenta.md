# 33 - Gobierno multi-cuenta

Una estrategia multi-cuenta crea límites de seguridad, facturación y operación. Una cuenta no es solo una carpeta: es un límite de aislamiento con cuotas, identidades y registro propios.

## Objetivos

- diseñar una organización y unidades organizativas (OU);
- distinguir permisos IAM de políticas de control de servicio (SCP);
- centralizar identidad, logs y seguridad;
- crear cuentas con guardarraíles y configuración base;
- evitar dependencias peligrosas entre entornos.

## Estructura inicial

```text
Organization
├─ Security
│  ├─ Log archive
│  └─ Security tooling
├─ Infrastructure
│  ├─ Network
│  └─ Shared services
├─ Workloads
│  ├─ Production
│  └─ Non-production
└─ Sandbox
```

Adapta la jerarquía a riesgos y responsabilidades. Evita crear OUs solo para reflejar el organigrama si no cambia la política aplicada.

## Controles preventivos y detectivos

- **SCP:** establece el máximo de permisos disponible; no concede permisos.
- **IAM:** concede acciones a identidades y roles dentro de ese límite.
- **AWS Config:** evalúa configuración y cumplimiento.
- **CloudTrail:** registra actividad de API.
- **Security Hub/GuardDuty:** agrega hallazgos y señales de seguridad.
- **Control Tower:** ayuda a establecer y gobernar una landing zone.

Prueba políticas restrictivas en una OU de laboratorio. Una negación demasiado amplia puede bloquear operaciones de seguridad o recuperación.

## Identidad y acceso

Centraliza acceso humano con IAM Identity Center y un proveedor de identidad cuando corresponda. Usa permission sets por función, sesiones temporales y acceso de emergencia controlado. Evita usuarios IAM replicados en cada cuenta.

Para automatización entre cuentas:

- rol de destino con trust policy estrecha;
- rol de origen con permiso para asumirlo;
- condiciones de organización, cuenta o contexto cuando proceda;
- sesiones breves y trazables.

## Logging centralizado

La cuenta de archivo debe estar separada de las cargas que genera los logs. Protege integridad, cifrado, retención y acceso. Valida que todas las regiones relevantes y nuevas cuentas queden cubiertas automáticamente.

## Laboratorio de diseño

Sin necesidad de crear una organización real:

1. Dibuja OUs y cuentas para una empresa con desarrollo y producción.
2. Define propietarios y datos permitidos en cada cuenta.
3. Escribe tres guardarraíles preventivos y tres detectivos.
4. Diseña acceso de desarrollador, pipeline y emergencia.
5. Explica dónde se almacenan logs y quién puede borrarlos.
6. Revisa el diseño ante una cuenta comprometida.

### Criterio de finalización

- [ ] Producción y no producción tienen límites separados.
- [ ] La cuenta de gestión no aloja cargas.
- [ ] Los logs de seguridad no dependen del administrador de la carga.
- [ ] Existe un proceso de alta y baja de cuentas.
- [ ] Cada guardarraíl tiene propósito, alcance y procedimiento de excepción.

## Preguntas de repaso

1. ¿Por qué una SCP no da acceso por sí sola?
2. ¿Qué debe permanecer en la cuenta de gestión?
3. ¿Cómo limitarías el movimiento lateral tras comprometer una cuenta?
4. ¿Qué automatizarías al crear una cuenta nueva?

**Anterior:** [32 - Operaciones](32-operaciones-systems-manager.md)  
**Siguiente:** [34 - Proyecto final](34-proyecto-final.md)
