# 14 - SQS, SNS, Kinesis

## SQS (Simple Queue Service)

Managed message queue.

### Características
```
- Unlimited throughput
- Retention: 1 min - 14 días (default 4 días)
- Max message size: 256 KB
- At-least-once delivery
- Best-effort ordering (FIFO available)
```

### Standard vs FIFO

**Standard**:
```
✅ Unlimited throughput
❌ Best-effort ordering
❌ At-least-once delivery (puede duplicar)
Uso: No importa orden ni duplicados
```

**FIFO**:
```
✅ Order guaranteed
✅ Exactly-once processing
❌ Limited throughput (300-3000 msg/s)
Uso: Orden crítico
```

### Para Odoo

```python
# Queue de jobs asincrónicos
# Producer (Odoo)
import boto3
sqs = boto3.client('sqs')

sqs.send_message(
    QueueUrl='https://sqs.eu-west-1.amazonaws.com/123/odoo-jobs',
    MessageBody=json.dumps({
        'job': 'process_invoice',
        'invoice_id': 12345
    })
)

# Consumer (Worker)
while True:
    messages = sqs.receive_message(QueueUrl=queue_url, MaxNumberOfMessages=10)
    for msg in messages.get('Messages', []):
        process_job(json.loads(msg['Body']))
        sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=msg['ReceiptHandle'])
```

## SNS (Simple Notification Service)

Pub/Sub messaging.

```
Publisher → SNS Topic → Multiple Subscribers

Subscribers:
- Email
- SMS
- SQS
- Lambda
- HTTP/HTTPS endpoint
```

### Fan-out Pattern

```
Odoo Event → SNS Topic
    ├─ SQS Queue 1 → Email service
    ├─ SQS Queue 2 → Analytics service
    └─ Lambda → Update external API
```

## Kinesis

Real-time data streaming.

### Kinesis Data Streams
```
Real-time streaming (logs, metrics, IoT)
Throughput: MB/s
Retention: 1-365 días
```

### Kinesis Data Firehose
```
Load streaming data to destinations
No retention
Destinations: S3, Redshift, Elasticsearch
```

### Kinesis Data Analytics
```
SQL queries on streaming data
Real-time analytics
```

**Para Odoo**: Generalmente SQS/SNS suficiente. Kinesis para IoT/analytics real-time.

---

**Siguiente**: [15 - Containers](15-containers.md)
