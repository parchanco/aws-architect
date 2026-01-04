# 20 - Serverless Solution Architectures

## Arquitectura 1: Website Estático

### Problema
Host website estático de forma escalable y barata.

### Solución

```
Route 53 (DNS)
    ↓
CloudFront (CDN)
    ↓
S3 (Static website hosting)
    ↓
Optional: Lambda@Edge (dynamic headers)
```

**Configuración**:
```bash
# 1. S3 bucket
aws s3 mb s3://www.tuempresa.com
aws s3 website s3://www.tuempresa.com --index-document index.html

# 2. Upload files
aws s3 sync ./website s3://www.tuempresa.com

# 3. CloudFront distribution
aws cloudfront create-distribution --origin-domain-name www.tuempresa.com.s3.amazonaws.com

# 4. Route 53
www.tuempresa.com → ALIAS CloudFront
```

**Costo**: ~$1-5/mes (bajo tráfico)

## Arquitectura 2: REST API Serverless

### Problema
API RESTful escalable sin gestionar servidores.

### Solución

```
Client
    ↓
API Gateway
    ↓
Lambda Functions
    ├─ GET /users → lambda-get-users → DynamoDB
    ├─ POST /users → lambda-create-user → DynamoDB
    ├─ PUT /users/{id} → lambda-update-user → DynamoDB
    └─ DELETE /users/{id} → lambda-delete-user → DynamoDB
```

**Ventajas**:
- Auto-scaling
- Pay per request
- No server management
- Built-in auth (Cognito)

**Código Lambda**:
```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def get_users(event, context):
    response = table.scan()
    return {
        'statusCode': 200,
        'body': json.dumps(response['Items'])
    }

def create_user(event, context):
    data = json.loads(event['body'])
    table.put_item(Item=data)
    return {
        'statusCode': 201,
        'body': json.dumps({'message': 'User created'})
    }
```

## Arquitectura 3: Procesamiento de Imágenes

### Problema
Resize/optimize imágenes subidas por usuarios.

### Solución

```
User upload → S3 (originals)
    ↓ S3 Event
Lambda (resize)
    ↓
S3 (thumbnails)
    ↓
DynamoDB (metadata)
```

**Lambda function**:
```python
from PIL import Image
import boto3
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get uploaded file
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download
    response = s3.get_object(Bucket=bucket, Key=key)
    img = Image.open(response['Body'])
    
    # Resize
    img.thumbnail((200, 200))
    
    # Upload thumbnail
    buffer = io.BytesIO()
    img.save(buffer, 'JPEG')
    buffer.seek(0)
    
    s3.put_object(
        Bucket=bucket,
        Key=f'thumbnails/{key}',
        Body=buffer,
        ContentType='image/jpeg'
    )
```

## Arquitectura 4: Event-Driven Processing

### Problema
Procesar eventos de múltiples fuentes.

### Solución

```
Sources:
├─ S3 uploads
├─ DynamoDB Streams
├─ CloudWatch Events
└─ Custom apps

    ↓
EventBridge
    ↓
Rules + Filters
    ↓
Targets:
├─ Lambda (process)
├─ Step Functions (workflow)
├─ SQS (queue)
└─ SNS (notify)
```

**Example Rule**:
```json
{
  "source": ["custom.app"],
  "detail-type": ["order.created"],
  "detail": {
    "amount": [{"numeric": [">", 1000]}]
  }
}

→ Target: Step Functions (fraud detection workflow)
```

## Arquitectura 5: Serverless CRON Jobs

### Problema
Ejecutar tareas programadas.

### Solución

```
EventBridge Rule (schedule)
    ↓
Lambda Function
    ↓
Task (cleanup, reports, backups)
```

**Example**:
```bash
# EventBridge rule
Schedule: cron(0 2 * * ? *)  # Every day 2 AM UTC

# Lambda: Daily cleanup
def lambda_handler(event, context):
    s3 = boto3.client('s3')
    dynamodb = boto3.resource('dynamodb')
    
    # Delete old files
    # Archive old records
    # Send daily report
```

## Arquitectura 6: Real-time Streaming

### Problema
Procesar datos en tiempo real (logs, IoT, analytics).

### Solución

```
Data Sources (IoT, apps, logs)
    ↓
Kinesis Data Streams
    ↓
Lambda (process)
    ↓
Destinations:
├─ DynamoDB (store)
├─ S3 (archive)
└─ CloudWatch (metrics)

Alternative:
Kinesis → Kinesis Data Firehose → S3/Redshift
```

## Arquitectura 7: GraphQL API

### Problema
API flexible con queries complejas.

### Solución

```
Client (GraphQL query)
    ↓
AppSync (GraphQL server)
    ↓
Resolvers:
├─ Lambda
├─ DynamoDB
└─ RDS

Features:
- Real-time subscriptions
- Offline sync
- Conflict resolution
```

## Arquitectura 8: Serverless Odoo Integration

### Escenario
Integrar Odoo con servicios externos sin sobrecargar Odoo.

### Solución

```
Odoo Webhook → API Gateway → Lambda
    ↓
Step Functions Workflow:
    ├─ Lambda: Validate order
    ├─ Lambda: Check inventory (Odoo API)
    ├─ Lambda: Process payment (Stripe)
    ├─ Lambda: Create shipment (UPS API)
    └─ Lambda: Update Odoo

Alternative: EventBridge + Lambda
```

**Lambda Odoo API**:
```python
import requests
import os

ODOO_URL = os.environ['ODOO_URL']
ODOO_DB = os.environ['ODOO_DB']
ODOO_USER = os.environ['ODOO_USER']
ODOO_PASSWORD = os.environ['ODOO_PASSWORD']

def lambda_handler(event, context):
    # Authenticate
    auth = requests.post(f'{ODOO_URL}/web/session/authenticate', json={
        'jsonrpc': '2.0',
        'params': {
            'db': ODOO_DB,
            'login': ODOO_USER,
            'password': ODOO_PASSWORD
        }
    })
    
    session_id = auth.cookies.get('session_id')
    
    # Call Odoo
    response = requests.post(
        f'{ODOO_URL}/web/dataset/call_kw',
        json={
            'jsonrpc': '2.0',
            'params': {
                'model': 'sale.order',
                'method': 'search_read',
                'args': [[['state', '=', 'sale']]],
                'kwargs': {'fields': ['name', 'partner_id', 'amount_total']}
            }
        },
        cookies={'session_id': session_id}
    )
    
    return response.json()
```

## Patrones Serverless

### Fan-Out
```
1 Event → Multiple consumers

S3 Upload → SNS Topic
    ├─ Lambda: Thumbnail
    ├─ Lambda: Metadata extraction
    └─ Lambda: Virus scan
```

### Saga Pattern
```
Distributed transaction:

Step Functions:
├─ Reserve inventory
├─ Process payment
├─ Create shipment
└─ Confirm order

If any fails → Compensating transactions
```

### CQRS (Command Query Responsibility Segregation)
```
Writes: API Gateway → Lambda → DynamoDB
Reads: API Gateway → Lambda → DynamoDB GSI/Elasticsearch

Optimize reads/writes separately
```

## Serverless Best Practices

```
✅ Design for stateless
✅ Use managed services (DynamoDB, not RDS)
✅ Implement idempotency
✅ Use async where possible
✅ Monitor con X-Ray
✅ Set appropriate timeouts
✅ Use Dead Letter Queues (DLQ)
✅ Implement circuit breakers
✅ Use Lambda Layers para dependencies
✅ Version functions
✅ Use aliases para environments

❌ No usar para long-running (> 15 min)
❌ No usar para latency-critical (< 10ms)
❌ No ignorar cold starts
```

## Costos Serverless

```
Example API (1M requests/mes):

API Gateway: $3.50
Lambda: $0.20 (128 MB, 100ms avg)
DynamoDB: $1.25 (1M writes)

Total: ~$5/mes

vs EC2 t3.micro: ~$8/mes (pero limited capacity)
```

---

**Certificación**: Conocer estos patrones es clave para diseño de soluciones en examen.

**Siguiente**: [21 - Databases in AWS](21-databases.md)
