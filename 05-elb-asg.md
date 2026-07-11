# 05 - Load Balancing & Auto Scaling

## Conceptos Fundamentales

### Scalability
**Vertical**: Aumentar recursos de la misma máquina (por ejemplo, pasar a una instancia con más vCPU y memoria)
**Horizontal**: Añadir más máquinas (1 servidor → 10 servidores)

### High Availability
Sistema sigue funcionando aunque fallen componentes.
**Principio**: Ejecutar en al menos 2 AZs.

## Elastic Load Balancing (ELB)

Distribuye tráfico entre múltiples servidores.

### Tipos de Load Balancers

**Application Load Balancer (ALB)** - Recomendado para Odoo
```
Layer 7 (HTTP/HTTPS)
✅ Routing por path, hostname, headers
✅ Target Groups
✅ SSL termination
✅ WebSocket support
✅ Health checks
Precio: ~$20-30/mes
```

**Network Load Balancer (NLB)**
```
Layer 4 (TCP/UDP)
✅ Ultra alto performance (millones req/seg)
✅ Static IP
❌ No entiende HTTP
Uso: Apps no-HTTP, extreme performance
```

**Gateway Load Balancer (GWLB)**
```
Para appliances de red (firewalls, IDS)
Uso: Muy específico
```

### ALB para Odoo

```
Internet
    ↓ :443 (HTTPS)
┌────────────┐
│    ALB     │ ← SSL Certificate (ACM)
└─────┬──────┘
      │ :8069 (HTTP interno)
 ┌────┼────┬────┐
 ↓    ↓    ↓    ↓
[Odoo-1][Odoo-2][Odoo-3]
```

**Features**:
- **SSL Termination**: ALB maneja HTTPS, backend HTTP
- **Health Checks**: `/web/health` cada 30 seg
- **Target Groups**: Organizar backends
- **Routing Rules**: `/longpolling` → Target Group específico

### Sticky Sessions
Mismo usuario → mismo servidor.

**Opción 1**: Cookie-based (ALB)
```
User login → ALB genera cookie
Siguiente request → Mismo server
```

**Opción 2**: Session storage (mejor)
```python
# odoo.conf + Redis
session_store = redis
redis_host = redis.xxx.cache.amazonaws.com

Todas las instancias comparten sessions
No necesitas sticky sessions
```

### Cross-Zone Load Balancing
Distribuye uniformemente entre todas las AZs.
- **ALB**: Enabled por defecto (gratis)
- **NLB**: Disabled por defecto

### Connection Draining
Espera a que conexiones terminen antes de sacar instancia.
```
Deregistration delay: 30-60 segundos (recomendado Odoo)

Durante deploy:
1. Marca instancia como "draining"
2. No acepta nuevas conexiones
3. Espera que actuales terminen
4. Saca instancia
→ Sin errores para usuarios
```

## Auto Scaling Groups (ASG)

Añade/quita servidores automáticamente según demanda.

### Componentes

**Launch Template**
```yaml
AMI: ami-odoo-prod-v1.0
Instance Type: t3.large
Security Groups: sg-odoo-app
IAM Role: OdooProductionRole
User Data: |
  #!/bin/bash
  systemctl start odoo
```

**Capacidad**
```
Min: 2    # Mínimo siempre activo
Max: 10   # Nunca más de 10
Desired: 4 # Intentar mantener 4
```

**Health Checks**
```
EC2: Instancia respondiendo
ELB: ALB marca como healthy

Si unhealthy → Termina y crea nueva
```

### Scaling Policies

**Target Tracking** (Recomendada)
```python
Métrica: Average CPU = 60%

CPU > 60% durante 5 min → Añadir instancias
CPU < 60% durante 15 min → Quitar instancias

Otras métricas:
- Request count per target
- Average network in/out
```

**Step Scaling**
```python
CPU > 80% → Add 3 instances
CPU > 60% → Add 1 instance
CPU < 30% → Remove 1 instance
```

**Scheduled Scaling**
```python
Lunes-Viernes:
  08:00 → Desired = 10
  20:00 → Desired = 2

Black Friday:
  Desired = 20
```

### Arquitectura Completa Odoo

```
Route 53 DNS
    ↓
┌────────────┐
│    ALB     │ ← SSL Certificate (ACM)
│  Multi-AZ  │
└─────┬──────┘
      │
┌─────▼──────────────┐
│ Auto Scaling Group │
│ Min:2 Max:10       │
│ Target CPU: 60%    │
│                    │
│ ┌────┐ ┌────┐ ┌────┐
│ │Odoo│ │Odoo│ │Odoo│
│ │AZ-a│ │AZ-b│ │AZ-c│
│ └────┘ └────┘ └────┘
└────────────────────┘
      │
┌─────▼──────┐
│ RDS Multi-AZ│
│ PostgreSQL │
└────────────┘
```

**Beneficios**:
- ✅ Alta disponibilidad (Multi-AZ)
- ✅ Auto-scaling según carga
- ✅ SSL centralizado (ACM gratis)
- ✅ Zero downtime deployments
- ✅ Cost optimization (scale down noches)

---

**Siguiente**: [06 - RDS & Aurora](06-rds-aurora.md)
