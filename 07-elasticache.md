# 07 - Amazon ElastiCache

ElastiCache es un servicio administrado de almacenamiento en memoria. Reduce latencia y descarga sistemas persistentes, pero introduce coherencia, invalidación y comportamiento ante fallos que deben diseñarse.

## Objetivos

- elegir entre Valkey, Redis OSS y Memcached;
- comparar caché serverless y clusters basados en nodos;
- aplicar cache-aside, write-through y TTL;
- diseñar alta disponibilidad, seguridad y observabilidad;
- evitar cache stampede y datos obsoletos.

## Motores

| Motor | Elegir cuando |
|---|---|
| Valkey | Necesitas estructuras avanzadas, replicación, persistencia, pub/sub o clustering; es la opción abierta moderna |
| Redis OSS | Mantienes compatibilidad con una carga Redis existente y una versión compatible |
| Memcached | Solo necesitas una caché simple, multihilo y efímera |

No uses una caché como única fuente de verdad. El sistema debe tolerar pérdida, evicción y reinicio.

## Serverless o basado en nodos

- **Serverless:** capacidad administrada y escalado automático; útil para demanda variable y menor operación.
- **Node-based:** elección de familias, réplicas, shards y parámetros; útil cuando necesitas control o capacidad predecible.

Compara coste con tráfico real: capacidad, peticiones, transferencia, backups y réplicas.

## Patrón cache-aside

```python
import json
import random

def get_partner(partner_id):
    key = f"partner:{partner_id}:v1"
    cached = cache.get(key)
    if cached is not None:
        return json.loads(cached)

    value = database.get_partner(partner_id)
    ttl = 300 + random.randint(0, 60)  # jitter evita expiración simultánea
    cache.setex(key, ttl, json.dumps(value))
    return value
```

Al escribir, actualiza primero la fuente de verdad y después invalida la clave. Diseña qué ocurre si la invalidación falla.

## Riesgos de caché

- **Cache stampede:** muchas peticiones recalculan el mismo valor; usa locking, request coalescing o refresh anticipado.
- **Hot keys:** una clave concentra tráfico; replica lecturas, particiona o rediseña.
- **Evictions:** indican presión de memoria o una política inadecuada.
- **Datos obsoletos:** TTL e invalidación deben derivarse del requisito de negocio.
- **Fallo de caché:** protege la base de datos con límites de concurrencia y circuit breakers.

## Alta disponibilidad

Para Valkey o Redis OSS basado en nodos, evalúa replicas, Multi-AZ, automatic failover y cluster mode. El cliente debe usar el endpoint adecuado, timeouts cortos, retry limitado y conexiones reutilizables. Prueba el failover: “Multi-AZ” no valida por sí solo el comportamiento de la aplicación.

## Seguridad

- despliega la caché en subnets privadas;
- permite el puerto solo desde el security group de la aplicación;
- habilita cifrado en tránsito y en reposo cuando corresponda;
- usa autenticación y RBAC compatibles con el motor;
- guarda secretos en Secrets Manager, nunca en `odoo.conf` versionado;
- registra cambios de configuración mediante CloudTrail.

La integración de sesiones o colas con Odoo depende del módulo y versión utilizados. Verifica compatibilidad antes de asumir que una directiva de configuración es nativa.

## Observabilidad

Observa latencia, conexiones, hit ratio, memoria, CPU/ECPU, evictions, replication lag y errores del cliente. Un hit ratio alto no compensa respuestas obsoletas.

## Laboratorio

1. Despliega una caché pequeña en una red privada.
2. Implementa cache-aside con TTL y jitter.
3. Mide latencia y carga de base de datos con caché fría y caliente.
4. Simula indisponibilidad y comprueba que la aplicación se degrada de forma controlada.
5. Provoca expiración concurrente y aplica una protección contra stampede.
6. Elimina la caché y comprueba el coste restante.

### Criterio de finalización

- [ ] La caché no es accesible desde Internet.
- [ ] La aplicación funciona, aunque más lentamente, sin caché.
- [ ] TTL e invalidación están justificados.
- [ ] Existen métricas y alarma ante evictions o errores.
- [ ] Se ha documentado la elección de motor y capacidad.

## Preguntas de repaso

1. ¿Cuándo elegirías Memcached frente a Valkey?
2. ¿Por qué añadir jitter al TTL?
3. ¿Cómo proteges la base de datos durante un fallo de caché?
4. ¿Qué diferencia hay entre disponibilidad de la caché y coherencia del dato?

**Siguiente:** [08 - Route 53](08-route53.md)
