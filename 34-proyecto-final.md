# 34 - Proyecto final: una carga Well-Architected

El proyecto final integra arquitectura, entrega, operación, seguridad y coste. Puedes implementar una aplicación web, una API serverless, una plataforma Odoo pequeña o un flujo de datos. Reduce alcance antes que eliminar controles esenciales.

## Escenario

Una empresa necesita una aplicación con:

- usuarios externos y datos sensibles moderados;
- entornos separados de desarrollo y producción;
- disponibilidad objetivo y recuperación definidas;
- despliegues frecuentes sin interrupciones largas;
- crecimiento incierto y presupuesto limitado;
- auditoría, alertas y responsable operativo.

Define los requisitos que falten. No elijas servicios antes de documentarlos.

## Fase 1 — Contexto y decisiones

- [ ] Usuarios, casos de uso y flujo de datos.
- [ ] Requisitos funcionales y no funcionales medibles.
- [ ] Clasificación de datos y modelo de amenazas.
- [ ] SLO, RTO y RPO.
- [ ] Restricciones de equipo, plazo, región y presupuesto.
- [ ] Al menos tres ADR con alternativas y consecuencias.

## Fase 2 — Arquitectura

- [ ] Diagrama con límites de cuenta, VPC, AZ y servicios administrados.
- [ ] Identidad humana y de máquina con mínimo privilegio.
- [ ] Cifrado, secretos, logs y protección de entrada.
- [ ] Escalado, desacoplamiento y tratamiento de reintentos.
- [ ] Backup, restauración y estrategia de fallo.
- [ ] Estimación inicial y principales inductores de coste.

## Fase 3 — Construcción

- [ ] Infraestructura desplegada por código.
- [ ] Pipeline con tests, análisis y artefacto inmutable.
- [ ] Configuración separada de código y secretos.
- [ ] Datos de prueba ficticios, nunca información real.
- [ ] Estrategia de despliegue y rollback demostrada.
- [ ] README de operación reproducible por otra persona.

## Fase 4 — Operación

- [ ] Logs estructurados, métricas y trazas donde aporten valor.
- [ ] Dashboard con señales de tráfico, errores, latencia y saturación.
- [ ] Alarmas accionables enlazadas a runbooks.
- [ ] Inventario y parcheado si existen servidores administrados.
- [ ] Prueba de restauración o recuperación con tiempo medido.
- [ ] Presupuesto, etiquetas y revisión de recursos huérfanos.

## Fase 5 — Revisión Well-Architected

Revisa los seis pilares:

| Pilar | Pregunta de defensa |
|---|---|
| Excelencia operativa | ¿Cómo sabes que funciona y cómo mejoras el proceso? |
| Seguridad | ¿Qué ocurre si una credencial o componente se compromete? |
| Fiabilidad | ¿Cómo falla y cómo se recupera? |
| Rendimiento | ¿Qué métrica demuestra que cumple su objetivo? |
| Coste | ¿Qué genera gasto y cómo se ajusta a la demanda? |
| Sostenibilidad | ¿Cómo reduces recursos y trabajo innecesarios? |

Identifica al menos cinco riesgos, priorízalos por impacto y esfuerzo, y corrige los dos principales.

## Simulacros obligatorios

Realiza de forma controlada y solo en laboratorio:

1. despliegue defectuoso y rollback;
2. pérdida de una dependencia o AZ simulada;
3. restauración de datos;
4. alarma por error o latencia;
5. intento de acción no autorizada.

Guarda hora de inicio, detección, diagnóstico, recuperación y aprendizaje.

## Entrega

```text
project/
├─ README.md
├─ architecture/
│  ├─ diagram.png
│  ├─ decisions/
│  └─ threat-model.md
├─ infrastructure/
├─ application/
├─ tests/
├─ operations/
│  ├─ dashboards.md
│  ├─ runbooks/
│  └─ recovery-test.md
└─ cost/
   └─ estimate.md
```

## Rúbrica (100 puntos)

| Área | Puntos |
|---|---:|
| Requisitos y decisiones justificadas | 15 |
| Seguridad | 20 |
| Fiabilidad y recuperación | 15 |
| IaC, CI/CD y calidad | 20 |
| Observabilidad y operación | 15 |
| Coste, rendimiento y sostenibilidad | 10 |
| Claridad de la entrega | 5 |

**Aprobado:** 70 puntos y ningún control crítico de seguridad omitido.  
**Dominado:** 85 puntos, simulacros completos y defensa oral de las decisiones.

## Limpieza final

- [ ] Exportar solo las evidencias necesarias.
- [ ] Eliminar stacks y comprobar recursos retenidos.
- [ ] Revocar accesos temporales y secretos de prueba.
- [ ] Confirmar en facturación que no queda gasto inesperado.
- [ ] Registrar aprendizajes y siguiente mejora.

**Anterior:** [33 - Gobierno multi-cuenta](33-gobierno-multicuenta.md)  
**Volver:** [README y checklist](README.md)
