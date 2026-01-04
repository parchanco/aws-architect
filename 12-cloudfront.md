# 12 - CloudFront & Global Accelerator

## CloudFront (CDN)

Content Delivery Network de AWS.

### Conceptos
```
Origin: Fuente de contenido (S3, ALB, EC2)
Edge Location: Cache cerca de usuarios (400+)
Distribution: Configuración CloudFront

User (Madrid) → Edge (Madrid) → Origin (Virginia)
Cache en Edge → Siguientes requests más rápidos
```

### Ventajas
```
✅ Reduce latency (content cerca usuario)
✅ DDoS protection (AWS Shield)
✅ HTTPS global
✅ Geo-restriction
✅ Compression
✅ Reduce load en origin
```

### Origins

**S3**:
```
CloudFront → S3 bucket
OAI (Origin Access Identity): Solo CloudFront accede a S3
Use case: Static website, downloads
```

**Custom Origin (HTTP)**:
```
CloudFront → ALB → EC2
CloudFront → EC2
CloudFront → On-premise server

Use case: Dynamic content
```

## CloudFront para Odoo

```
www.tuempresa.com
    ↓
CloudFront Distribution
    ├─ /static/* → S3 (CSS, JS, images)
    └─ /* → ALB (Odoo app)

Beneficios:
✅ Static assets cached en Edge
✅ Reduce load en Odoo
✅ Faster para usuarios globales
✅ SSL/TLS termination
```

### Invalidation

```bash
# Borrar cache manualmente
aws cloudfront create-invalidation     --distribution-id E123456     --paths "/static/*" "/*.js"

Cost: $0.005 per path (primeros 1000 gratis/mes)
```

## CloudFront vs S3 Cross-Region Replication

```
CloudFront:
- Global edge network
- TTL cache
- Great for static content
- Read-only

S3 CRR:
- Specific regions
- Real-time replication
- Great for dynamic, low-latency
- Read/Write
```

## AWS Global Accelerator

Optimiza routing global usando red privada AWS.

```
User → Edge Location → AWS private network → App

vs

User → Internet → App

Beneficios:
✅ 60% latency improvement
✅ Static IPs (2)
✅ Fast regional failover
✅ Health checks
```

### Cuándo usar
```
CloudFront:
- Content cacheable
- Static assets
- HTTP/HTTPS

Global Accelerator:
- Non-HTTP (TCP/UDP)
- Dynamic content
- Need static IPs
- Gaming, IoT
```

---

**Siguiente**: [13 - Storage Extras](13-storage-extras.md)
