# 22 - Data & Analytics en AWS

## Amazon Athena

Query data en S3 usando SQL.

### Características
```
Serverless
Pay per query ($5 per TB scanned)
Standard SQL (Presto engine)
Formats: CSV, JSON, Parquet, ORC, Avro
Compressed formats supported
```

### Use Case

```
S3 Bucket (logs, data)
    ↓
Athena (SQL queries)
    ↓
Results → S3 or QuickSight

Example:
Query CloudTrail logs
Query ALB access logs
Query application logs
```

### Example Query

```sql
CREATE EXTERNAL TABLE alb_logs (
    type string,
    time string,
    elb string,
    client_ip string,
    target_ip string,
    request_processing_time double,
    target_processing_time double,
    response_processing_time double,
    elb_status_code string,
    target_status_code string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
LOCATION 's3://my-bucket/alb-logs/';

-- Query
SELECT 
    client_ip,
    COUNT(*) as requests,
    AVG(target_processing_time) as avg_response_time
FROM alb_logs
WHERE elb_status_code = '200'
GROUP BY client_ip
ORDER BY requests DESC
LIMIT 10;
```

### Performance Optimization

```
✅ Use columnar formats (Parquet, ORC)
✅ Compress data (GZIP, Snappy)
✅ Partition data (year/month/day)
✅ Use appropriate data types
✅ Limit scanned data (WHERE clauses)

Cost optimization:
Parquet + partitioning = 90% cost reduction
```

## Amazon Redshift

Data warehouse para analytics.

### Características
```
Columnar storage
Massively parallel processing (MPP)
SQL queries
Petabyte scale
Integration con S3, DynamoDB
BI tools: QuickSight, Tableau, PowerBI
```

### Arquitectura

```
Leader Node:
- Query planning
- Result aggregation

Compute Nodes:
- Execute queries
- Store data
- Node types: dc2, ra3
```

### Redshift Spectrum

Query data directamente en S3.

```
Redshift Cluster
    ↓
Redshift Spectrum
    ↓
S3 (exabytes of data)

No need to load data into Redshift
Pay per TB scanned
```

### Backup & DR

```
Automated snapshots: Every 8 hours or 5 GB
Retention: 1-35 days
Manual snapshots: Indefinite
Copy snapshots to other regions
```

### Enhanced VPC Routing

```
Force all COPY/UNLOAD traffic through VPC
More control
Monitoring with VPC Flow Logs
```

## Amazon EMR (Elastic MapReduce)

Managed Hadoop framework.

### Frameworks Supported
```
Hadoop
Spark
HBase
Presto
Flink
Hive
```

### Use Cases
```
✓ Big data processing
✓ Log analysis
✓ Machine learning
✓ ETL workloads
✓ Genomics
```

### Cluster Types

**Transient**:
```
Spin up cluster
Run job
Terminate cluster
Cost-effective
```

**Long-running**:
```
Always-on cluster
Multiple jobs
Interactive queries
```

### Node Types

```
Master Node: Coordinate
Core Nodes: Run tasks, store HDFS
Task Nodes: Run tasks only (Spot instances ideal)
```

## AWS Glue

Serverless ETL service.

### Components

**Glue Data Catalog**:
```
Metadata repository
Schema discovery
Integration con Athena, Redshift, EMR
```

**Glue Crawler**:
```
Scan data sources
Discover schemas
Populate Data Catalog
```

**Glue ETL Jobs**:
```
Python or Scala
Visual ETL editor
Auto-generated code
Serverless (charged per DPU-hour)
```

### Example ETL Job

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

# Initialize
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)

# Read from Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="raw_data"
)

# Transform
transformed = ApplyMapping.apply(
    frame=datasource,
    mappings=[
        ("id", "long", "product_id", "long"),
        ("name", "string", "product_name", "string"),
        ("price", "double", "product_price", "double")
    ]
)

# Write to S3 (Parquet)
glueContext.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={"path": "s3://output-bucket/"},
    format="parquet"
)

job.commit()
```

## Amazon Kinesis

Real-time streaming data.

### Kinesis Data Streams

```
Producers → Kinesis Stream → Consumers

Shards: Throughput units
1 shard = 1 MB/s write, 2 MB/s read

Retention: 1-365 days
```

**Producers**:
- SDK
- Kinesis Agent
- Kinesis Producer Library (KPL)

**Consumers**:
- SDK
- Lambda
- Kinesis Client Library (KCL)
- Kinesis Data Firehose
- Amazon Managed Service for Apache Flink

### Kinesis Data Firehose

```
Sources → Firehose → Destinations

Fully managed
Near real-time (60s latency min)
Auto-scaling
Data transformation (Lambda)

Destinations:
- S3
- Redshift
- Elasticsearch
- HTTP endpoints
- Datadog, Splunk, etc.
```

### Amazon Managed Service for Apache Flink

```
Procesamiento con Apache Flink usando SQL, Java, Python o Scala

Kinesis Stream/Firehose
    ↓
    Managed Service for Apache Flink
    ↓
Kinesis Stream/Firehose/Lambda

Use case: Real-time dashboards, metrics
```

### Kinesis vs SQS

```
Kinesis:
- Real-time (200ms)
- Ordering per shard
- Multiple consumers
- Replay data
- Complex routing

SQS:
- Near real-time (seconds)
- No ordering (except FIFO)
- Single consumer (per message)
- Message deleted after consumption
- Simple queuing
```

## Amazon MSK (Managed Streaming for Apache Kafka)

Managed Kafka service.

### Features
```
Fully compatible Apache Kafka
Multi-AZ deployment
Auto healing
Encryption in-transit and at-rest
Integration con Amazon Managed Service for Apache Flink
```

### MSK vs Kinesis

```
MSK:
✓ Want Kafka APIs
✓ Need Kafka ecosystem
✓ Complex routing

Kinesis:
✓ AWS-native integration
✓ Simpler
✓ No Kafka expertise needed
```

## Amazon QuickSight

BI and visualization.

### Features
```
Serverless
Auto-scaling
Embeddable dashboards
ML insights
Pay per session

Data sources:
- RDS, Aurora, Redshift
- Athena
- S3
- On-premise databases
```

### QuickSight Q

```
NLP queries:
"What were sales last quarter?"
→ Auto-generates visualization
```

## AWS Data Pipeline

Orchestrate and automate data workflows.

```
Workflow:
1. Extract from source (RDS, S3, DynamoDB)
2. Transform (EMR, custom scripts)
3. Load to destination (Redshift, S3)

Scheduled or on-demand
Retry logic
Notifications (SNS)
```

**vs Glue**:
```
Data Pipeline: More complex workflows, on-premise
Glue: Simpler ETL, serverless, AWS-native
```

## AWS Lake Formation

Build data lakes.

### Features
```
Centralized permissions
Data catalog (Glue)
Data ingestion
Security and governance
```

### Architecture

```
Data Sources
    ↓
Lake Formation
    ├─ Data Catalog
    ├─ Permissions
    └─ Blueprints (ingestion templates)
    ↓
S3 Data Lake
    ↓
Analytics: Athena, EMR, Redshift
```

## Data Analytics Architecture - Odoo

```
Example: Analyze Odoo data

1. Export PostgreSQL → S3 (daily)
   RDS snapshot → Glue job → S3 (Parquet)

2. Catalog
   Glue Crawler → Data Catalog

3. Query
   Athena → SQL queries on S3

4. Visualize
   QuickSight → Dashboards

5. Advanced Analytics
   EMR/Spark → ML models

Benefits:
✓ No impact on production DB
✓ Query historical data
✓ Cost-effective storage
✓ Scalable analytics
```

## Diseño moderno de un data lake

### Capas y contratos

```text
sources → raw/bronze → validated/silver → products/gold
            │                 │                │
         immutable       quality rules     business contract
```

- **Raw:** conserva el evento original, fecha de ingesta y procedencia.
- **Validated:** corrige tipos, duplicados y registros inválidos sin ocultar errores.
- **Products:** tablas preparadas para un caso de negocio y con propietario.

El nombre de una capa no garantiza calidad. Define esquema, SLA, clasificación, propietario y consumidores.

### Formatos de datos

- usa Parquet u otro formato columnar para analítica;
- comprime y particiona por campos con cardinalidad y filtros útiles;
- evita miles de archivos pequeños: aumentan planificación y peticiones;
- evalúa Apache Iceberg cuando necesites evolución de esquema, transacciones y snapshots;
- no particiones directamente por identificadores de alta cardinalidad.

### Gobierno con Lake Formation

Lake Formation permite administrar permisos sobre catálogos, bases, tablas y columnas. Combínalo con IAM, cifrado, clasificación y auditoría. Diseña el acceso desde el dato hacia los roles consumidores, no con permisos amplios sobre todo el bucket.

### Athena y control de costes

- separa equipos mediante workgroups;
- limita bytes escaneados y cifra resultados;
- usa Parquet, compresión, particionado y proyección cuando corresponda;
- observa consultas fallidas, datos escaneados y coste por producto;
- evita `SELECT *` en procesos recurrentes.

### Redshift y cargas de trabajo

Compara Redshift provisionado y Redshift Serverless según previsibilidad, aislamiento y operación. Distribución, sort keys y materialized views dependen del patrón de consulta; mide antes de optimizar.

## Calidad, linaje y observabilidad

Un pipeline correcto debe responder:

- ¿llegaron los datos a tiempo y completos?;
- ¿cumplen esquema, unicidad y reglas de negocio?;
- ¿qué transformación produjo una columna?;
- ¿qué consumidores se ven afectados por un cambio?;
- ¿cómo se reprocesa sin duplicar resultados?;
- ¿cuánto cuesta por dataset o ejecución?

Registra métricas de freshness, volumen, nulls, duplicados, distribución y fallos. Separa registros inválidos en una zona de cuarentena con proceso de resolución.

## Laboratorio: de ingesta a consulta

1. Genera eventos ficticios de pedidos y guárdalos en una zona raw.
2. Cataloga el esquema y transforma a Parquet particionado.
3. Añade controles de esquema, unicidad e importes válidos.
4. Consulta con Athena desde un workgroup limitado.
5. Restringe una columna sensible mediante gobierno de datos.
6. Introduce un registro inválido, envíalo a cuarentena y reprocesa.

### Criterio de finalización

- [ ] El dataset puede reconstruirse desde raw.
- [ ] El proceso es idempotente y soporta replay.
- [ ] Calidad, linaje, propietario y coste son visibles.
- [ ] Los consumidores no acceden directamente a datos sensibles sin autorización.
- [ ] Existe estrategia para cambios compatibles de esquema.

## Preguntas de repaso

1. ¿Por qué demasiados archivos pequeños perjudican la analítica?
2. ¿Qué aporta Iceberg frente a una colección de Parquet sin metadatos transaccionales?
3. ¿Cómo diseñarías un reprocesado sin duplicados?
4. ¿Qué diferencia hay entre que un job termine y que produzca datos correctos?

---

**Certificación**: Athena, Redshift, Kinesis son temas frecuentes en examen.

**Siguiente**: [23 - Machine Learning](23-machine-learning.md)
