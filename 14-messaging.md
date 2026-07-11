# 14 - Mensajería y arquitecturas event-driven

Desacoplar no es solo añadir una cola: implica definir entrega, orden, duplicados, reintentos, presión, observabilidad y propiedad del evento.

## Objetivos

- elegir entre SQS, SNS, EventBridge y Kinesis Data Streams;
- diseñar consumidores idempotentes;
- configurar visibility timeout, DLQ y redrive;
- aplicar fan-out, filtrado y backpressure;
- operar un flujo asíncrono de extremo a extremo.

## Elegir el servicio

| Necesidad | Servicio habitual |
|---|---|
| Cola de trabajo con consumidores desacoplados | SQS |
| Publicación simple a varios suscriptores | SNS |
| Bus de eventos, reglas, SaaS y routing por contenido | EventBridge |
| Stream ordenado por shard, replay y gran volumen | Kinesis Data Streams |
| Kafka administrado y ecosistema Kafka | Amazon MSK |

Una arquitectura puede combinarlos: SNS o EventBridge distribuye; una cola SQS por consumidor absorbe picos y aísla fallos.

## Amazon SQS

### Standard y FIFO

- **Standard:** escala ampliamente, entrega al menos una vez y orden best-effort.
- **FIFO:** conserva orden dentro de cada message group y ofrece deduplicación; aun así, el procesamiento de negocio debe ser idempotente.

“Exactly-once processing” no está garantizado de extremo a extremo: un consumidor puede completar el efecto y fallar antes de borrar el mensaje.

### Ciclo de un mensaje

```text
send → available → receive → invisible ── delete → finalizado
                       │
                       └─ timeout → available de nuevo
```

El visibility timeout debe superar el tiempo normal de proceso o extenderse mediante heartbeat. Usa long polling para reducir respuestas vacías. Los mensajes grandes deben guardar el payload en S3 y transportar una referencia.

### Consumidor idempotente

```python
def handle(message):
    event_id = message["event_id"]

    if idempotency_store.exists(event_id):
        return "already-processed"

    with database.transaction():
        process_invoice(message["invoice_id"])
        idempotency_store.record(event_id)
```

La comprobación y el efecto deben ser atómicos o diseñarse con una restricción única. Un `SELECT` seguido de un `INSERT` sin protección mantiene una carrera.

### DLQ y redrive

Envía a DLQ tras un número razonable de intentos. Alarma por edad y número de mensajes, conserva contexto y define quién diagnostica. No redirijas toda la DLQ sin corregir la causa: podrías repetir daños o crear un bucle.

Con Lambda, utiliza partial batch responses para reintentar solo los mensajes fallidos cuando el origen lo soporte.

## SNS y fan-out

```text
OrderCreated → SNS topic
  ├─ SQS billing
  ├─ SQS email
  └─ SQS analytics
```

Aplica filter policies para evitar tráfico innecesario. Usa una cola por consumidor cuando necesites reintentos, backpressure y aislamiento. HTTPS directo requiere que el endpoint gestione disponibilidad y reintentos.

## Amazon EventBridge

EventBridge enruta eventos según reglas y contenido. Es útil para integración entre dominios, eventos de servicios AWS, partners SaaS, Scheduler y Pipes.

Un buen evento describe un hecho pasado y contiene:

- identificador único;
- tipo y versión de esquema;
- fecha de ocurrencia;
- productor y correlation ID;
- datos mínimos necesarios, sin secretos.

Versiona esquemas de forma compatible. El productor no debe conocer la implementación de cada consumidor.

## Kinesis Data Streams

Kinesis conserva registros para que varios consumidores puedan procesarlos y reproducirlos. La partition key decide el shard y el orden; una clave demasiado popular crea un hot shard.

Diseña:

- capacidad on-demand o provisionada;
- distribución de partition keys;
- retención y replay;
- checkpointing del consumidor;
- iterator age y lag;
- tratamiento de registros defectuosos.

Para procesar streams con SQL, Java, Python o Scala, revisa Amazon Managed Service for Apache Flink. Firehose entrega streams a destinos administrados y puede transformar o convertir formatos.

## Reintentos y backpressure

- exponential backoff con jitter;
- máximo de intentos y timeout total;
- límites de concurrencia para proteger dependencias;
- circuit breaker para fallos sostenidos;
- DLQ o destino de fallo con propietario;
- métricas de edad, profundidad, errores y throughput.

## Laboratorio: pedidos asíncronos

1. Publica `OrderCreated` con un ID único.
2. Enruta a una cola de facturación y otra de notificaciones.
3. Implementa un consumidor idempotente.
4. Provoca un error para enviar un mensaje a DLQ.
5. Crea alarmas por edad de cola y DLQ.
6. Corrige el fallo, redirige un único mensaje y valida el resultado.

### Criterio de finalización

- [ ] Un duplicado no repite el efecto de negocio.
- [ ] Un consumidor lento no bloquea a los demás.
- [ ] DLQ, redrive y propietario están documentados.
- [ ] Las métricas permiten detectar backlog antes del impacto.
- [ ] Los recursos del laboratorio se eliminaron.

## Preguntas de repaso

1. ¿Por qué FIFO no elimina la necesidad de idempotencia?
2. ¿Qué ocurre si el visibility timeout es demasiado corto?
3. ¿Cuándo elegirías EventBridge frente a SNS?
4. ¿Cómo evitarías un hot shard en Kinesis?

**Siguiente:** [15 - Contenedores](15-containers.md)
