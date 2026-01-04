# 15 - Containers en AWS

## ECS (Elastic Container Service)

Orchestration de Docker containers.

### Launch Types

**EC2 Launch Type**:
```
Tú gestionas: EC2 instances
AWS gestiona: Container orchestration
Más control, más management
```

**Fargate Launch Type**:
```
Serverless
AWS gestiona: Todo
Tú defines: Task definition (CPU, RAM, image)
Más caro, menos management
```

### Conceptos

**Task Definition**:
```json
{
  "family": "odoo-task",
  "containerDefinitions": [{
    "name": "odoo",
    "image": "odoo:14",
    "memory": 512,
    "cpu": 256,
    "portMappings": [{
      "containerPort": 8069
    }]
  }]
}
```

**Service**:
```
Define: Cuántas tasks ejecutar
Integration: ALB, Auto Scaling
```

## ECR (Elastic Container Registry)

Docker registry privado.

```bash
# Login
aws ecr get-login-password | docker login --username AWS --password-stdin xxx.dkr.ecr.eu-west-1.amazonaws.com

# Tag image
docker tag odoo:custom xxx.dkr.ecr.eu-west-1.amazonaws.com/odoo:custom

# Push
docker push xxx.dkr.ecr.eu-west-1.amazonaws.com/odoo:custom
```

## EKS (Elastic Kubernetes Service)

Managed Kubernetes.

```
Kubernetes en AWS
AWS gestiona: Control plane
Tú gestionas: Worker nodes (o Fargate)

Cuándo usar:
✅ Ya usas Kubernetes
✅ Multi-cloud strategy
✅ Necesitas features Kubernetes
❌ Simple apps (usa ECS Fargate)
```

## Odoo con Containers

### Docker Compose (Local Dev)

```yaml
version: '3'
services:
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo
  
  odoo:
    image: odoo:14
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
      HOST: db
      USER: odoo
      PASSWORD: odoo
```

### ECS Fargate (Production)

```
Task Definition:
├─ Container: Odoo
│  └─ Image: ECR odoo:custom
│  └─ Port: 8069
│  └─ Environment: DB credentials (Secrets Manager)
└─ Container: Nginx (sidecar)
   └─ Port: 80
   └─ Proxy to Odoo:8069

Service:
├─ Desired tasks: 3
├─ ALB integration
└─ Auto Scaling
```

---

**Siguiente**: [16 - Best Practices](16-best-practices.md)
