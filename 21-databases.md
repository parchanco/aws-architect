# 21 - Databases en AWS

## Tipos de Bases de Datos

### Relacional (SQL)
```
RDS: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server
Aurora: PostgreSQL-compatible, MySQL-compatible

Uso: Transactions, ACID, structured data
Ejemplo Odoo: PostgreSQL para app data
```

### NoSQL
```
DynamoDB: Key-Value, document
DocumentDB: MongoDB-compatible
Keyspaces: Cassandra-compatible

Uso: High throughput, flexible schema, scale horizontal
```

### In-Memory
```
ElastiCache: Redis, Memcached
MemoryDB for Redis: Durable Redis

Uso: Caching, session storage, real-time
```

### Graph
```
Neptune: Graph database

Uso: Social networks, fraud detection, recommendations
```

### Time Series
```
Timestream: Time series data

Uso: IoT, metrics, application monitoring
```

### Ledger
```
QLDB: Quantum Ledger Database

Uso: Immutable, cryptographically verifiable transactions
```

## RDS Deep Dive

### Storage

**Storage Types**:
```
gp3: General Purpose (default)
  - 3,000 - 16,000 IOPS
  - 125 - 1,000 MB/s
  
io1/io2: Provisioned IOPS
  - Up to 64,000 IOPS
  - 1,000 MB/s
```

**Storage Auto Scaling**:
```
Enable: ✓
Max storage: 1000 GB
Free space threshold: 10%

→ RDS scales automatically when < threshold
```

### Read Replicas

**Características**:
```
Async replication
Up to 5 replicas (15 para Aurora)
Cross-Region supported
Can be promoted to standalone
```

**Use Cases**:
- Reporting queries (no impact production)
- Analytics workloads
- Disaster recovery (cross-region)

**Replication Cost**:
- Same region: Free
- Cross-region: Pay network transfer

### Multi-AZ

**High Availability**:
```
Synchronous replication
Standby en different AZ
Automatic failover (1-2 min)
Same DNS endpoint
```

**Multi-AZ vs Read Replica**:
```
Multi-AZ:
- Purpose: High availability
- Standby: No read traffic
- Sync replication
- Same region

Read Replica:
- Purpose: Scaling reads
- Active: Serves read traffic
- Async replication
- Cross-region possible
```

### Backups

**Automated Backups**:
```
Daily full backup + transaction logs
Point-in-time recovery
Retention: 0-35 days (0 = disabled)
```

**Manual Snapshots**:
```
User-initiated
Retention: Indefinite
Can share with other accounts
Can copy to other regions
```

**Backup Window**:
```
Specify preferred window: 03:00-04:00 UTC
Some performance impact during backup
```

### Security

**Encryption**:
```
At-rest: KMS (enable at creation)
In-transit: SSL/TLS
Master encrypted → replicas encrypted
```

**Network**:
```
Deploy in private subnets
Security Groups: Port 5432/3306 from app only
No public access (unless required)
```

**IAM Authentication**:
```
Instead of password: IAM token (15 min TTL)
CloudTrail audit
```

## Aurora Deep Dive

### Architecture

```
Shared Storage Layer:
- 6 copies across 3 AZs
- 10 GB - 128 TB auto-scaling
- Self-healing
- < 10ms replica lag

Compute Layer:
- 1 Writer
- Up to 15 Readers
```

### Aurora Endpoints

```
Cluster Endpoint (Writer):
cluster-name.cluster-xxx.region.rds.amazonaws.com

Reader Endpoint:
cluster-name.cluster-ro-xxx.region.rds.amazonaws.com
→ Load balances across all readers

Custom Endpoints:
Define subset of instances
Example: analytics-endpoint → 2 large readers
```

### Aurora Features

**Backtrack**:
```
"Time-travel" database
Rewind to any point (up to 72 hours)
Seconds to complete
No new DB needed
```

**Aurora Cloning**:
```
Create copy of database
Uses copy-on-write protocol
Fast (minutes)
Cost-effective (shared storage until diverge)

Use: Testing, development
```

**Aurora Global Database**:
```
Primary region: Read/Write
Secondary regions (up to 5): Read-only
Replication lag: < 1 second
Disaster recovery: Promote secondary < 1 min
```

**Aurora Machine Learning**:
```
SQL queries con ML models

SELECT review, 
       aws_comprehend_detect_sentiment(review, 'en') as sentiment
FROM product_reviews;

Integration: SageMaker, Comprehend
```

**Aurora Serverless v2**:
```
Auto-scaling compute
Scale in seconds (not minutes como v1)
ACUs: 0.5 - 128
Pay per second
```

### Aurora vs RDS

```
Performance:
Aurora: 5x PostgreSQL, 3x MySQL
RDS: Standard performance

Availability:
Aurora: 6 copies, < 30s failover
RDS: 2 copies (Multi-AZ), 60-120s failover

Replicas:
Aurora: 15
RDS: 5

Features:
Aurora: Backtrack, Global, ML, Serverless
RDS: Standard features

Cost:
Aurora: ~20% more expensive
RDS: Baseline price
```

## DynamoDB Deep Dive

### Data Model

```
Table: Products
Items:
  {
    "ProductId": "123",  # Partition Key
    "Name": "Laptop",
    "Price": 999,
    "Category": "Electronics",
    "Stock": 50
  }
  
Composite Key:
  Partition Key: CustomerId
  Sort Key: OrderDate
  → Allows queries like "all orders for customer X"
```

### Indexes

**Global Secondary Index (GSI)**:
```
Different partition key
Can create anytime
Has own RCU/WCU
Max 20 per table

Example:
Base table: ProductId (PK)
GSI: Category (PK) → Query by category
```

**Local Secondary Index (LSI)**:
```
Same partition key, different sort key
Must create at table creation
Shares RCU/WCU with table
Max 5 per table
```

### Capacity Modes

**Provisioned**:
```
RCU: Read Capacity Units
WCU: Write Capacity Units

1 RCU = 1 strongly consistent read/s (4 KB)
1 WCU = 1 write/s (1 KB)

Auto-scaling: ✓
Cheaper if predictable workload
```

**On-Demand**:
```
Pay per request
No capacity planning
Auto scales
2.5x more expensive than provisioned
Good for unpredictable traffic
```

### DynamoDB Streams

```
Capture changes (insert/update/delete)
Retention: 24 hours
Trigger Lambda
Use cases:
- Replication
- Analytics
- Notifications
```

### DynamoDB Accelerator (DAX)

```
In-memory cache for DynamoDB
Microsecond latency
No application changes (compatible API)
10x performance improvement
```

### Global Tables

```
Multi-region, multi-active
Replication < 1 second
Disaster recovery
Low latency global access

Requirements:
- Streams enabled
- Same table name all regions
```

### Backup & Restore

**On-Demand Backup**:
```
Full backup
No performance impact
Retention: Until deleted
```

**Point-in-Time Recovery (PITR)**:
```
Continuous backups
Restore to any second (last 35 days)
Must be enabled
```

## Comparación Bases de Datos

### ¿Cuándo usar qué?

**RDS/Aurora**:
```
✓ ACID transactions needed
✓ Complex queries (JOINs)
✓ Structured data
✓ Existing SQL applications

Ejemplo Odoo: Application database
```

**DynamoDB**:
```
✓ Key-value access patterns
✓ Massive scale (10+ trillion requests/day)
✓ Single-digit millisecond latency
✓ Serverless

Ejemplo: Session storage, user profiles, IoT
```

**ElastiCache**:
```
✓ Sub-millisecond latency
✓ Temporary data
✓ Frequently accessed data

Ejemplo: Session store, cache queries
```

**Neptune**:
```
✓ Graph data
✓ Relationships important

Ejemplo: Social network, fraud detection
```

**DocumentDB**:
```
✓ MongoDB workloads
✓ JSON documents
✓ Flexible schema

Ejemplo: Content management, catalogs
```

## Database Migration

### AWS DMS (Database Migration Service)

```
Source → DMS → Target

Sources: Oracle, SQL Server, PostgreSQL, MySQL, MongoDB
Targets: RDS, Aurora, Redshift, DynamoDB, S3

Migration types:
- Homogeneous: PostgreSQL → PostgreSQL
- Heterogeneous: Oracle → PostgreSQL (need SCT)

CDC: Continuous Data Capture
→ Minimal downtime migration
```

### Schema Conversion Tool (SCT)

```
Convert schemas:
Oracle → PostgreSQL
SQL Server → Aurora MySQL

Converts:
- Schema
- Views
- Stored procedures
- Functions
```

---

**Certificación**: Conocer cuándo usar cada tipo de database es fundamental.

**Siguiente**: [22 - Data & Analytics](22-data-analytics.md)
