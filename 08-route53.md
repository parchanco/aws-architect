# 08 - Route 53

Servicio DNS de AWS.

## DNS Básico

```
Domain Name System:
odoo.tuempresa.com → 52.123.45.67

Record Types:
- A: IPv4
- AAAA: IPv6
- CNAME: Alias (www → root)
- ALIAS: AWS (root domain → ELB)
- MX: Mail servers
- TXT: Verificación, SPF
```

## Hosted Zones

Container para DNS records.

```python
Precio:
- Public Hosted Zone: $0.50/mes
- Queries: $0.40 per million

Records:
tuempresa.com          A      52.123.45.67
www.tuempresa.com      CNAME  tuempresa.com
odoo.tuempresa.com     ALIAS  odoo-alb.eu-west-1.elb.amazonaws.com
```

## ALIAS vs CNAME

```
CNAME:
❌ NO puede usarse en root domain
✅ www.tuempresa.com CNAME tuempresa.com
❌ tuempresa.com CNAME other.com

ALIAS (AWS específico):
✅ Puede usarse en root domain
✅ Free (no query charges)
✅ Health checks integrados
✅ Auto IP updates

Targets ALIAS:
- ELB (ALB, NLB)
- CloudFront
- S3 website
- API Gateway
```

## Routing Policies

### Simple
Un record, múltiples valores. Cliente elige random.

### Weighted
Divide tráfico por peso.
```
70% → eu-west-1
30% → us-east-1

Uso: Blue/Green deployments
```

### Latency
Retorna record con menor latency al usuario.
```
Usuario Madrid → eu-west-1
Usuario New York → us-east-1
```

### Failover
Primary/Secondary para DR.
```
Primary: eu-west-1 (+ Health Check)
Secondary: us-east-1
Si Primary fails → Secondary
```

### Geolocation
Basado en ubicación geográfica.
```
Europe → eu-west-1
North America → us-east-1
Default → us-east-1
```

### Multi-Value
Múltiples valores con health checks.
Similar a Simple pero con HA.

## Health Checks

```python
Monitor endpoint:
Protocol: HTTPS
Path: /web/health
Port: 443
Interval: 30 seconds
Failure threshold: 3

Cost: $0.50/mes per check
```

### Health Check Odoo

```python
@http.route('/web/health', auth='none')
def health_check(self):
    try:
        # Check DB
        request.env.cr.execute("SELECT 1")
        # Check filestore
        test_file = '/tmp/health_test'
        with open(test_file, 'w') as f:
            f.write('ok')
        os.remove(test_file)
        return "OK", 200
    except:
        return "ERROR", 503
```

## Setup DNS Odoo

```
tuempresa.com
├─ tuempresa.com → ALIAS ALB eu-west-1
│                   Failover → ALB us-east-1
├─ www → CNAME tuempresa.com
├─ odoo → ALIAS ALB Production
├─ staging → ALIAS ALB Staging
└─ api → ALIAS API Gateway
```

## Blue/Green Deployment

```python
1. Estado inicial:
   odoo.com → Weight 100% ALB Blue

2. Deploy Green:
   Test en green-alb.elb.amazonaws.com

3. Traffic shift:
   odoo.com → Weight 10% ALB Green
              Weight 90% ALB Blue
   
4. Increase:
   50% / 50%

5. Complete:
   100% Green

6. Rollback (si problemas):
   100% Blue
```

---

**Siguiente**: [09 - S3 Introduction](09-s3-intro.md)
