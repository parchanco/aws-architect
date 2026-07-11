# 29 - FinOps y economía cloud

FinOps conecta ingeniería, finanzas y negocio para obtener valor de la nube. No consiste solo en reducir la factura: busca que cada equipo entienda el coste de sus decisiones y pueda actuar.

## Objetivos

Al terminar podrás:

- explicar facturación, precios y coste total de propiedad;
- asignar costes mediante cuentas, etiquetas y categorías;
- detectar anomalías y evitar sorpresas;
- elegir entre escalado, programación y compromisos de uso;
- presentar una decisión técnica en coste por unidad de negocio.

## Del coste total al coste unitario

Una factura menor no implica una arquitectura mejor. Relaciona gasto con una unidad útil:

```text
coste por pedido = coste mensual de la carga / pedidos completados
coste por usuario = coste mensual de la carga / usuarios activos
```

Mide también calidad: disponibilidad, latencia, errores y tiempo de entrega. Ahorrar degradando el servicio desplaza el coste.

## Ciclo FinOps

### 1. Informar

- separar entornos o productos con cuentas;
- definir etiquetas obligatorias: `Application`, `Environment`, `Owner`, `CostCenter`;
- activar etiquetas de asignación de costes;
- explorar Cost Explorer y Cost and Usage Report;
- crear paneles compartidos y responsables claros.

### 2. Optimizar

- ajustar tamaño con métricas, no con intuición;
- apagar laboratorios y entornos no productivos fuera de horario;
- eliminar volúmenes, snapshots, IP y balanceadores huérfanos;
- aplicar lifecycle en S3 y políticas de retención de logs;
- comparar arquitecturas bajo demanda, serverless y aprovisionadas;
- evaluar Savings Plans o Reserved Instances solo tras entender una demanda estable.

### 3. Operar

- fijar presupuestos y alertas por equipo;
- revisar anomalías semanalmente;
- incluir una estimación de coste en cada cambio de arquitectura;
- registrar acciones, propietario, ahorro esperado y ahorro realizado.

## Herramientas AWS

| Necesidad | Servicio o herramienta |
|---|---|
| Estimar antes de construir | AWS Pricing Calculator |
| Analizar gasto histórico | Cost Explorer |
| Alertar sobre un umbral | AWS Budgets |
| Detectar gasto atípico | Cost Anomaly Detection |
| Recomendaciones de uso | AWS Compute Optimizer |
| Detalle para analítica | Cost and Usage Report |

## Laboratorio: guardarraíles de coste

1. Define un presupuesto mensual pequeño para el laboratorio.
2. Configura alertas al 50 %, 80 % y 100 % del importe.
3. Etiqueta todos los recursos de una práctica.
4. Crea una vista de coste agrupada por servicio y etiqueta.
5. Localiza un recurso infrautilizado o huérfano.
6. Documenta la acción y comprueba días después su efecto.

### Criterio de finalización

- [ ] La alerta llega a un canal revisado.
- [ ] El coste puede atribuirse a un propietario.
- [ ] La recomendación incluye impacto técnico y económico.
- [ ] Los recursos de prueba se han eliminado.

## Preguntas de repaso

1. ¿Por qué un compromiso de uso no es el primer paso de optimización?
2. ¿Qué diferencia hay entre presupuesto, previsión y anomalía?
3. ¿Qué coste indirecto puede añadir una arquitectura multi-región?
4. ¿Qué métrica de negocio usarías para una tienda online?

**Anterior:** [28 - Recuperación y migraciones](28-disaster-recovery.md)  
**Siguiente:** [30 - Infraestructura como código](30-infraestructura-como-codigo.md)
