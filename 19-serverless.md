# 19 - Serverless en AWS

## Lambda

Ejecuta código sin gestionar servidores.

### Características
```
Lenguajes: Python, Node.js, Java, Go, Ruby, .NET
Timeout: Max 15 minutos
Memory: 128 MB - 10 GB
Pricing: Por request + duration
Free tier: 1M requests/mes + 400,000 GB-segundos
```

### Conceptos Clave

**Function**:
```python
import json

def lambda_handler(event, context):
    # event: Input data
    # context: Runtime info
    
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

**Triggers**:
- API Gateway (HTTP requests)
- S3 (object upload)
- EventBridge (scheduled/events)
- SQS (messages)
- DynamoDB Streams
- SNS
- CloudWatch Logs

**Environment Variables**:
```python
import os

DB_HOST = os.environ['DB_HOST']
API_KEY = os.environ['API_KEY']
```

**Layers**:
```
Shared code/dependencies
Max 5 layers per function
Example: boto3 latest, custom libraries
```

### Lambda + API Gateway

Crear REST API sin servidores.

```
Client → API Gateway → Lambda → DynamoDB

GET /users → Lambda get_users()
POST /users → Lambda create_user()
```

### Lambda para Odoo

```python
# Procesar jobs asíncronos
# S3 upload → Lambda → Process → Odoo API

import boto3
import requests
import json

def lambda_handler(event, context):
    # S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download file
    s3 = boto3.client('s3')
    file_content = s3.get_object(Bucket=bucket, Key=key)['Body'].read()
    
    # Process (OCR, parse, etc.)
    data = process_file(file_content)
    
    # Call Odoo API
    response = requests.post(
        'https://odoo.tuempresa.com/api/invoices',
        json=data,
        headers={'Authorization': f'Bearer {os.environ["ODOO_TOKEN"]}'}
    )
    
    return {'statusCode': 200, 'body': json.dumps('Processed')}
```

### Lambda Limits

```
Deployment package: 50 MB (zipped), 250 MB (unzipped)
/tmp storage: 10 GB
Concurrent executions: 1000 (default, puede aumentarse)
Function timeout: 15 min max
Environment variables: 4 KB total
```

### Lambda Best Practices

```
✅ Use environment variables para config
✅ Minimize deployment package
✅ Use Layers para dependencies compartidas
✅ Set appropriate timeout (no siempre 15 min)
✅ Use async invocation para long-running
✅ Handle errors y retries
✅ Monitor con CloudWatch Logs
✅ Use X-Ray para tracing
✅ Keep functions stateless
✅ Use Parameter Store/Secrets Manager para secrets

❌ No hardcodear credentials
❌ No usar /tmp para persistent storage
❌ No hacer functions muy grandes (split)
```

## Step Functions

Orquestación de workflows serverless.

### State Machine
```json
{
  "StartAt": "ProcessOrder",
  "States": {
    "ProcessOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ProcessOrder",
      "Next": "CheckInventory"
    },
    "CheckInventory": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:CheckInventory",
      "Next": "HasStock?"
    },
    "HasStock?": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.stock",
          "NumericGreaterThan": 0,
          "Next": "ShipOrder"
        }
      ],
      "Default": "NotifyNoStock"
    },
    "ShipOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ShipOrder",
      "End": true
    },
    "NotifyNoStock": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:NotifyNoStock",
      "End": true
    }
  }
}
```

**Casos de uso**:
- Procesos multi-paso
- Workflows con decisiones
- Orchestración de microservices
- ETL pipelines
- Order processing

## API Gateway

Crear, publicar, mantener APIs.

### Tipos

**REST API**:
```
HTTP APIs RESTful
Request validation
API keys
Throttling
Caching
```

**HTTP API**:
```
Más simple
Más barato (70% cheaper)
Lower latency
JWT authorization
```

**WebSocket API**:
```
Two-way communication
Chat apps
Real-time dashboards
```

### Integraciones

```
Lambda: Invoke function
HTTP: Proxy a backend HTTP
AWS Services: DynamoDB, S3, etc.
Mock: Return hardcoded response
```

### Features

**Throttling**:
```
Limit requests por segundo
Default: 10,000 req/s
Burst: 5,000 req
Configurable por stage/method
```

**Caching**:
```
Cache responses
TTL: 300 segundos (default)
Size: 0.5 GB - 237 GB
Reduce load en backend
```

**API Keys**:
```
Control access
Usage plans
Quotas y throttles
```

**CORS**:
```
Enable cross-origin requests
Configure allowed origins
```

## DynamoDB

NoSQL database serverless.

### Características
```
Fully managed
Single-digit millisecond latency
Scales to 10+ trillion requests/day
Auto scaling
Encryption at rest
Point-in-time recovery
Global Tables (multi-region)
```

### Data Model

```
Table: Users
Items (rows):
  - UserId: "123" (Partition Key)
    Email: "user@example.com"
    Name: "Pablo"
    Created: "2024-01-03"
  
  - UserId: "456" (Partition Key)
    Email: "maria@example.com"
    Name: "Maria"

Primary Key:
- Partition Key (required)
- Sort Key (optional)
```

### Capacity Modes

**On-Demand**:
```
Pay per request
Auto scales
No capacity planning
$1.25 per million write requests
$0.25 per million read requests
```

**Provisioned**:
```
Define RCU/WCU
Auto scaling available
Cheaper if predictable
RCU: Read Capacity Unit
WCU: Write Capacity Unit
```

### Indexes

**LSI (Local Secondary Index)**:
```
Same partition key, different sort key
Max 5 per table
Must create at table creation
```

**GSI (Global Secondary Index)**:
```
Different partition key
Can create anytime
Max 20 per table
Has own RCU/WCU
```

### DynamoDB Streams
```
Capture modifications (insert/update/delete)
Trigger Lambda
Process changes
Replicate to other tables
```

### Para Odoo

```python
# Session storage en DynamoDB
# Alternativa a Redis para serverless

import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('odoo-sessions')

# Write session
table.put_item(
    Item={
        'session_id': 'abc123',
        'user_id': 42,
        'data': {'cart': [1, 2, 3]},
        'ttl': 1704326400  # Expires in 1 hour
    }
)

# Read session
response = table.get_item(Key={'session_id': 'abc123'})
session = response.get('Item')
```

## EventBridge

Event bus serverless.

### Conceptos

```
Event Bus: Canal de eventos
Rules: Filtros y routing
Targets: Destinos (Lambda, SQS, SNS, etc.)
```

### Example Rule

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}

Target: Lambda → Notify Slack
```

**Scheduled Events**:
```
Cron: cron(0 12 * * ? *)  # Daily at 12:00 UTC
Rate: rate(5 minutes)
```

## SAM (Serverless Application Model)

Framework para deploy serverless apps.

### template.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.9
      CodeUri: hello_world/
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

### Commands

```bash
# Build
sam build

# Local testing
sam local invoke HelloWorldFunction

# Local API
sam local start-api

# Deploy
sam deploy --guided
```

## Serverless Architecture Odoo

```
Eventos Odoo → EventBridge
    ↓
Step Functions Workflow
    ├─ Lambda: Validate data
    ├─ Lambda: Process invoice (OCR)
    ├─ DynamoDB: Save result
    └─ Lambda: Notify user (SES)

Alternative: API Gateway → Lambda → RDS/DynamoDB
```

---

**Certificación**: Lambda, API Gateway, DynamoDB son fundamentales para AWS Solutions Architect.

**Siguiente**: [20 - Serverless Architectures](20-serverless-architectures.md)
