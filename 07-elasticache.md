# 07 - ElastiCache

In-memory cache para mejorar performance.

## Engines

### Redis (Recomendado)
```
✅ Data structures (strings, lists, sets, hashes)
✅ Persistence (RDB, AOF)
✅ High availability (Multi-AZ, replicación)
✅ Backups
✅ Pub/Sub
Precio: ~$0.017/h (cache.t3.micro)
```

### Memcached
```
✅ Simple key-value
✅ Multi-threaded
❌ No persistence
❌ No replicación
```

**Selección**: Usa Redis (más features, HA).

## Arquitectura Redis

```
Primary (Read/Write)
  ↓ Replicación
  ├─ Replica 1 (Read)
  ├─ Replica 2 (Read)
  └─ Replica 3 (Read)

Hasta 5 replicas
Multi-AZ con auto-failover
```

## ElastiCache para Odoo

### 1. Session Storage

```python
# odoo.conf
session_store = redis
redis_host = redis.xxx.cache.amazonaws.com
redis_port = 6379

Beneficios:
✅ Sessions compartidas entre instancias
✅ No necesitas sticky sessions en ALB
✅ Sessions persisten reinicio Odoo
```

### 2. Application Cache

```python
# Cache queries frecuentes
def get_partner_stats(partner_id):
    cache_key = f'partner:{partner_id}:stats'
    
    # Try cache
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Query DB
    stats = compute_stats(partner_id)
    
    # Cache 1 hora
    redis_client.setex(cache_key, 3600, json.dumps(stats))
    return stats
```

### 3. Queue de Jobs

```python
# Redis como queue
import rq
queue = rq.Queue('odoo-jobs', connection=redis_conn)

# Enqueue
queue.enqueue('process_invoice', invoice_id=123)
```

## Caching Strategies

### Lazy Loading
```python
def get_data(key):
    data = cache.get(key)
    if not data:
        data = db.query(key)
        cache.set(key, data, ttl=3600)
    return data

Pros: Solo cachea lo que se pide
Cons: Cache miss penalty
```

### Write-Through
```python
def save_data(key, data):
    cache.set(key, data)
    db.save(key, data)

Pros: Cache siempre actualizado
Cons: Write penalty
```

### Cache Invalidation
```python
# Al actualizar data
def update_partner(partner_id, data):
    db.update(partner_id, data)
    cache.delete(f'partner:{partner_id}')
```

## Security

```python
Security Group:
Inbound: Port 6379 desde SG de apps
No acceso desde Internet

Encryption:
- At-rest: KMS
- In-transit: TLS
- Auth token: Password para conexión
```

## Monitoring

```python
CloudWatch Metrics:
- CPUUtilization
- FreeableMemory
- CacheHits / CacheMisses
- Evictions

Alarms:
- CPU > 75%
- Memory < 50 MB
- Evictions > threshold
```

---

**Siguiente**: [08 - Route 53](08-route53.md)
