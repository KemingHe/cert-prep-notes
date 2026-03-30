# Study Notes - AWS Database and Analytics Services

> **Last Updated**: 2026-03-30 by Keming He

## Table of Contents

- [Study Notes - AWS Database and Analytics Services](#study-notes---aws-database-and-analytics-services)
  - [Table of Contents](#table-of-contents)
  - [Database Overview](#database-overview)
    - [Relational vs NoSQL Databases](#relational-vs-nosql-databases)
    - [Managed Database Benefits](#managed-database-benefits)
  - [Relational Databases (OLTP)](#relational-databases-oltp)
    - [Amazon RDS](#amazon-rds)
    - [Amazon Aurora](#amazon-aurora)
  - [In-Memory Databases](#in-memory-databases)
    - [Amazon ElastiCache](#amazon-elasticache)
  - [NoSQL Databases](#nosql-databases)
    - [Amazon DynamoDB](#amazon-dynamodb)
    - [DynamoDB Accelerator (DAX)](#dynamodb-accelerator-dax)
    - [DynamoDB Global Tables](#dynamodb-global-tables)
    - [Amazon DocumentDB](#amazon-documentdb)
  - [Graph Databases](#graph-databases)
    - [Amazon Neptune](#amazon-neptune)
  - [Time-Series Databases](#time-series-databases)
    - [Amazon Timestream](#amazon-timestream)
  - [Analytics Services (OLAP)](#analytics-services-olap)
    - [Amazon Redshift](#amazon-redshift)
    - [Amazon EMR](#amazon-emr)
    - [Amazon Athena](#amazon-athena)
    - [Amazon QuickSight](#amazon-quicksight)
  - [Data Integration](#data-integration)
    - [AWS Glue](#aws-glue)
    - [AWS DMS](#aws-dms)
  - [Specialized Services](#specialized-services)
    - [Amazon Managed Blockchain](#amazon-managed-blockchain)
  - [Service Summary](#service-summary)
  - [Shared Responsibility Model](#shared-responsibility-model)
    - [AWS Responsibilities](#aws-responsibilities)
    - [Customer Responsibilities](#customer-responsibilities)
  - [Best Practices](#best-practices)
  - [References](#references)

## Database Overview

### Relational vs NoSQL Databases

| Type | Characteristics | Examples |
| :--- | :--- | :--- |
| **Relational** | Schema-defined, tables with primary/foreign keys, SQL queries, normalized data | RDS, Aurora |
| **NoSQL** | Flexible/no schema, horizontally scalable, optimized for specific access patterns | DynamoDB, DocumentDB, Neptune |

**NoSQL database types**:

- Key-value (DynamoDB)
- Document (DocumentDB)
- Graph (Neptune)
- In-memory (ElastiCache)
- Time-series (Timestream)

### Managed Database Benefits

[AWS managed databases](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/database.html) provide advantages over self-managed databases on EC2:

- Quick provisioning with high availability
- Vertical and horizontal scaling
- Automated backups and point-in-time restore
- Automated patching and upgrades
- Monitoring, alerting, and performance insights
- Multi-AZ deployment for disaster recovery

> [↑ Back to Table of Contents](#table-of-contents)

---

## Relational Databases (OLTP)

### Amazon RDS

[Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) (Relational Database Service) is a managed service for relational databases.

**Supported engines** ([6 engines](https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/choosing-engine.html)):

| Engine | Notes |
| :--- | :--- |
| PostgreSQL | Open source, advanced features |
| MySQL | Popular open source |
| MariaDB | MySQL fork |
| Oracle | Enterprise, licensing options |
| Microsoft SQL Server | Windows ecosystem |
| IBM Db2 | Enterprise mainframe workloads |

**Key characteristics**:

- [Storage backed by EBS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Storage.html) (Elastic Block Store)
- [No SSH access](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html) to underlying instances (fully managed; exception: RDS Custom for Oracle/SQL Server)
- Continuous backup with [point-in-time recovery](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/AutomatedBackups.PiTR.html) using automated backups (daily snapshots + transaction logs every 5 minutes)
- Maintenance windows for upgrades

**Read Replicas**:

- [Up to 15 read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html) per primary instance
- Asynchronous replication from primary (read-write) to replicas (read-only)
- [Cross-region read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.XRgn.html) supported for MySQL, PostgreSQL, MariaDB, Oracle, SQL Server (not Db2)
- Good for read-heavy OLAP workloads and local read performance

**Multi-AZ Deployments**:

- [Synchronous replication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html) to standby in different AZ
- Automatic failover (60-120 seconds typical)
- Standby is for failover only, not for read scaling

### Amazon Aurora

[Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html) is part of Amazon RDS - a fully managed relational database engine compatible with MySQL and PostgreSQL. You select Aurora MySQL or Aurora PostgreSQL as the DB engine option when setting up databases through RDS.

**Performance** ([verified benchmarks](https://aws.amazon.com/rds/aurora/faqs/)):

| Compatibility | Performance Improvement |
| :--- | :--- |
| MySQL | [5x faster](https://aws.amazon.com/rds/aurora/faqs/) (SysBench on r3.8xlarge) |
| PostgreSQL | [3x faster](https://aws.amazon.com/rds/aurora/faqs/) (SysBench on r4.16xlarge) |

**Storage**:

- [Auto-scaling in 10 GiB increments](https://aws.amazon.com/rds/aurora/faqs/)
- Maximum: [256 TiB](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html) (Aurora PostgreSQL 15.13+, 16.9+, 17.5+ and Aurora MySQL 3.10+); 128 TiB for older versions
- Storage separated from compute

**Pricing**: No official percentage comparison to RDS; [pricing varies](https://aws.amazon.com/rds/aurora/pricing/) by configuration, I/O patterns, and workload. Aurora I/O-Optimized may be more cost-effective for I/O-intensive workloads.

**Aurora Serverless v2**:

- [Dynamic scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html) of compute capacity based on demand
- Pay per second, no capacity planning required
- Good for infrequent, intermittent, or unpredictable workloads
- Note: Does NOT use a "proxy fleet" - [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-proxy.html) is a separate optional service for connection pooling

> [↑ Back to Table of Contents](#table-of-contents)

---

## In-Memory Databases

### Amazon ElastiCache

[Amazon ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html) is a fully managed in-memory caching service.

**Supported engines**:

- Redis OSS
- Memcached
- Valkey (Redis fork)

**Deployment options**:

| Mode | Description |
| :--- | :--- |
| [Serverless](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html) | No provisioning, auto-scaling |
| Node-Based Clusters | Select node types (not EC2 instances - fully managed) |

**Use case**: Reduce database load for read-intensive workloads by caching frequently accessed data.

**Architecture pattern**: Client -> ELB -> EC2 -> ElastiCache (cache hit) or RDS (cache miss)

> [↑ Back to Table of Contents](#table-of-contents)

---

## NoSQL Databases

### Amazon DynamoDB

[Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) is a fully managed, serverless key-value and document database.

**High availability**:

- [Data replicated across 3 AZs](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) automatically
- 99.99% availability SLA

**Scale**:

- [Millions of requests per second](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- Trillions of rows
- Hundreds of TB of storage

**Performance**: [Single-digit millisecond latency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) at any scale

**Data model**:

- [Primary key](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) options:
  - Simple: Partition key only
  - Composite: Partition key + sort key (sort key is optional)
- [No native JOIN support](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) - denormalize data or use application-side joins

**Table classes**:

| Class | Use Case |
| :--- | :--- |
| [DynamoDB Standard](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CostOptimization_TableClass.html) | Default, frequently accessed data |
| [DynamoDB Standard-IA](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CostOptimization_TableClass.html) | 60% lower storage cost, infrequently accessed |

**Capacity modes**:

| Mode | Provisioning | Billing |
| :--- | :--- | :--- |
| On-demand | None required (truly serverless) | Pay per request |
| Provisioned | Set read/write capacity units | Pay for provisioned capacity |

### DynamoDB Accelerator (DAX)

[DAX](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html) is a fully managed, in-memory cache specifically for DynamoDB.

- [10x performance improvement](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html) - from milliseconds to microseconds
- Sits between application and DynamoDB
- DynamoDB-only (not a general-purpose cache like ElastiCache)

### DynamoDB Global Tables

[Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html) provide multi-region, multi-active replication.

- [Active-Active replication](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html) - read and write to any replica region
- Low-latency access for globally distributed applications
- Automatic conflict resolution

### Amazon DocumentDB

[Amazon DocumentDB](https://aws.amazon.com/documentdb/features/) is a MongoDB-compatible document database.

- Compatible with [MongoDB 4.0, 5.0, and 8.0](https://aws.amazon.com/documentdb/features/) APIs, drivers, and tools (3.6 is [end of standard support](https://docs.aws.amazon.com/documentdb/latest/developerguide/release-notes.html) as of March 2026, Extended Support until 2029)
- [Data replicated 6 ways across 3 AZs](https://docs.aws.amazon.com/documentdb/latest/developerguide/replication.html) (each 10 GiB segment)
- [Storage auto-scales in 10 GiB increments](https://aws.amazon.com/documentdb/features/) up to 4 PiB (128 TiB for standard clusters)
- [Elastic Clusters](https://aws.amazon.com/documentdb/features/) handle millions of reads/writes per second

> [↑ Back to Table of Contents](#table-of-contents)

---

## Graph Databases

### Amazon Neptune

[Amazon Neptune](https://aws.amazon.com/neptune/features/) is a fully managed graph database.

**Query languages**: Gremlin, openCypher, SPARQL

**High availability**:

- [Data replicated 6 ways across 3 AZs](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-storage.html)
- [Up to 15 read replicas](https://aws.amazon.com/neptune/features/) with [tens of milliseconds replication lag](https://aws.amazon.com/neptune/faqs/) (typically much less than 100ms)

**Performance**:

- [100,000+ queries per second](https://aws.amazon.com/neptune/features/) for graph traversals with low latency
- Neptune Analytics: Process [tens of billions of relationships within seconds](https://aws.amazon.com/neptune/features/) (not milliseconds for analytical queries at this scale)

**Use cases**: Knowledge graphs, fraud detection, recommendation engines, social networks

> [↑ Back to Table of Contents](#table-of-contents)

---

## Time-Series Databases

### Amazon Timestream

[Amazon Timestream](https://aws.amazon.com/timestream/features/) is a fully managed, serverless time-series database.

**Scale**: [Store and analyze trillions of events per day](https://aws.amazon.com/timestream/features/)

**Performance claims** (marketing benchmarks with qualifiers - actual results vary by workload):

- [Up to 1,000x faster](https://aws.amazon.com/timestream/features/) than relational databases for time-series workloads
- [As little as 1/10th the cost](https://aws.amazon.com/timestream/features/) of relational databases

**Features**:

- Built-in time-series analytics functions
- Automatic data tiering (recent data in memory, historical in cost-optimized storage)

**Use cases**: IoT applications, DevOps monitoring, industrial telemetry

> [↑ Back to Table of Contents](#table-of-contents)

---

## Analytics Services (OLAP)

### Amazon Redshift

[Amazon Redshift](https://aws.amazon.com/redshift/) is a fully managed data warehouse for OLAP workloads.

**Performance** ([current benchmarks](https://aws.amazon.com/redshift/price-performance/)):

- [Up to 6x better price-performance](https://aws.amazon.com/redshift/price-performance/) compared to other cloud data warehouses
- [Up to 7x better price-performance](https://aws.amazon.com/redshift/price-performance/) for high-concurrency, low-latency workloads

**Architecture**:

- [Columnar storage](https://docs.aws.amazon.com/redshift/latest/dg/c_columnar_storage_disk_mem_mgmnt.html) for analytics optimization
- [MPP (Massively Parallel Processing)](https://docs.aws.amazon.com/redshift/latest/dg/c_challenges_achieving_high_performance_queries.html) engine distributes queries across nodes
- SQL interface compatible with PostgreSQL

**Scale**: [Petabytes of data](https://aws.amazon.com/redshift/)

**Data loading**: Batch loading (not real-time) - data loaded periodically

**Integrations**: [Amazon QuickSight, Tableau, Microsoft PowerBI](https://aws.amazon.com/redshift/)

**Redshift Serverless**: Pay for compute and storage during analytics only, no cluster management

### Amazon EMR

[Amazon EMR](https://aws.amazon.com/emr/) (Elastic MapReduce) is a managed big data platform.

**Frameworks supported**:

- Apache Hadoop
- [Apache Spark, Trino, Apache Flink](https://aws.amazon.com/emr/) (performance-optimized runtimes)
- HBase, Presto

**Scale**: [Provision hundreds of nodes](https://aws.amazon.com/ec2/spot/use-case/emr/) for large-scale clusters

**Cost optimization**: [Spot Instance integration](https://aws.amazon.com/ec2/spot/use-case/emr/) - up to 90% discount

**Use cases**: Data processing, machine learning, web indexing, log analysis

### Amazon Athena

[Amazon Athena](https://aws.amazon.com/athena/faqs/) is a serverless SQL query service for data in S3.

**Query engine**: [Athena engine v3](https://docs.aws.amazon.com/athena/latest/ug/engine-versions-reference-0003.html) uses continuous integration with both open-source Trino and Presto projects, providing faster access to community improvements integrated and tuned within the Athena engine

**Supported formats**: [CSV, TSV, JSON, ORC, Parquet, Avro](https://aws.amazon.com/athena/faqs/)

**Pricing**: [$5 per TB of data scanned](https://aws.amazon.com/athena/pricing/); use columnar formats (Parquet, ORC) and compression to reduce costs

**Integrations**: Amazon QuickSight for dashboards and reporting

**Use cases**: Log analytics (VPC Flow Logs, ELB logs, CloudTrail trails), ad-hoc queries on S3 data lakes

### Amazon QuickSight

[Amazon QuickSight](https://aws.amazon.com/quicksight/features-ml/) is a serverless, ML-powered business intelligence service.

**Features**:

- [Serverless architecture](https://aws.amazon.com/quicksight/features-ml/) with auto-scaling
- [Built-in ML](https://aws.amazon.com/quicksight/features-ml/) for anomaly detection, forecasting, natural language queries
- Embeddable dashboards

**Data sources**: RDS, Aurora, Athena, Redshift, S3, and more

**Pricing** ([current model](https://aws.amazon.com/quicksight/pricing/)):

| Tier | Price |
| :--- | :--- |
| Reader | $3/user/month |
| Author | $24/user/month |
| Capacity (session-based) | $0.50/session (monthly) or $0.40/session (annual) |

> [↑ Back to Table of Contents](#table-of-contents)

---

## Data Integration

### AWS Glue

[AWS Glue](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html) is a serverless data integration service for ETL (Extract, Transform, Load).

**Components**:

| Component | Function |
| :--- | :--- |
| Glue ETL | Data preparation and transformation |
| [Glue Data Catalog](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html) | Central metadata repository for data assets |

**Data Catalog integration**: [Searchable by Amazon Athena, Amazon EMR, and Amazon Redshift Spectrum](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html)

**Use case**: Prepare and transform data between sources (e.g., S3) and destinations (e.g., Redshift)

### AWS DMS

[AWS DMS](https://aws.amazon.com/dms/features/) (Database Migration Service) migrates databases to AWS.

**Key features**:

- [Source database remains operational](https://aws.amazon.com/dms/features/) during migration
- Continuous replication of changes
- Self-healing and resilient

**Migration types**:

| Type | Example |
| :--- | :--- |
| Homogeneous | Oracle to Oracle |
| Heterogeneous | SQL Server to Aurora |

> [↑ Back to Table of Contents](#table-of-contents)

---

## Specialized Services

### Amazon Managed Blockchain

[Amazon Managed Blockchain](https://aws.amazon.com/managed-blockchain/features/) enables building blockchain applications.

**Supported frameworks**:

- [Hyperledger Fabric](https://aws.amazon.com/managed-blockchain/features/)
- [Ethereum](https://aws.amazon.com/managed-blockchain/features/)

**Use cases**: Applications requiring decentralized authority and multi-party transactions

> [↑ Back to Table of Contents](#table-of-contents)

---

## Service Summary

| Category | Service | Key Characteristic |
| :--- | :--- | :--- |
| Relational (OLTP) | RDS | 6 managed database engines |
| Relational (OLTP) | Aurora (RDS) | MySQL/PostgreSQL compatible, 3-5x faster |
| In-Memory | ElastiCache | Redis/Memcached/Valkey caching |
| Key-Value | DynamoDB | Serverless, single-digit ms latency |
| Key-Value Cache | DAX | DynamoDB-only, microsecond latency |
| Document | DocumentDB | MongoDB-compatible |
| Graph | Neptune | Gremlin, openCypher, SPARQL |
| Time-Series | Timestream | Trillions of events/day |
| Data Warehouse | Redshift | Columnar, MPP, petabyte-scale |
| Big Data | EMR | Hadoop, Spark, managed clusters |
| Query S3 | Athena | Serverless SQL on S3, Trino/Presto-based |
| BI/Dashboards | QuickSight | Serverless, ML-powered |
| ETL | Glue | Serverless data integration |
| Migration | DMS | Zero-downtime migration |
| Blockchain | Managed Blockchain | Hyperledger Fabric, Ethereum |

> [↑ Back to Table of Contents](#table-of-contents)

---

## Shared Responsibility Model

### AWS Responsibilities

- Infrastructure management and physical security
- Database engine patching and updates
- Automated backups and replication
- High availability infrastructure
- Monitoring infrastructure health

### Customer Responsibilities

- Database configuration and optimization
- Data content and schema design
- Access control (IAM policies, security groups)
- Encryption settings and key management
- Backup retention policies and testing
- Application-level security

> [↑ Back to Table of Contents](#table-of-contents)

---

## Best Practices

- **Choose the right database type**: Match database to access patterns - relational for complex queries and transactions, NoSQL for scale and flexibility
- **Use read replicas**: Offload read traffic from primary instances for read-heavy workloads
- **Enable Multi-AZ**: Deploy production databases across multiple AZs for high availability
- **Implement caching**: Use ElastiCache or DAX to reduce database load and improve latency
- **Right-size capacity**: Start with on-demand/serverless for unpredictable workloads, switch to provisioned for steady-state
- **Use appropriate storage classes**: DynamoDB Standard-IA for infrequently accessed data
- **Optimize analytics costs**: Use columnar formats (Parquet, ORC) with Athena to reduce data scanned
- **Centralize metadata**: Use Glue Data Catalog as a central repository for data discovery

> [↑ Back to Table of Contents](#table-of-contents)

---

## References

- [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Amazon Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Amazon Redshift Documentation](https://docs.aws.amazon.com/redshift/latest/dg/c_redshift_system_overview.html)
- [Amazon Athena User Guide](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
- [AWS Glue Developer Guide](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html)
- [AWS Database Services Overview](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/database.html)

> [↑ Back to Table of Contents](#table-of-contents)
