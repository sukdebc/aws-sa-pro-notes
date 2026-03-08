# AWS Analytics Notes (SAP-C02 Level)

Kinesis • MSK • Glue • EMR • Athena • Redshift • Lake Formation • OpenSearch

These notes summarize streaming, batch analytics, data lake architectures, and warehouse workloads across AWS analytics services.

The focus is on scaling models, replay behavior, governance integration, and workload separation across ingestion, processing, storage, and query layers.

---

# Streaming Ingestion

## Kinesis Data Streams

Documentation  
https://docs.aws.amazon.com/streams/latest/dev/introduction.html

Kinesis Data Streams supports high-throughput real-time ingestion.

Core characteristics:

- Shard-based scaling
- 1 MB/sec ingress per shard
- 2 MB/sec egress per shard
- 1,000 records/sec writes per shard
- Data replay supported
- Ordered processing within a shard

Replay can be performed using:

- TRIM_HORIZON
- AT_TIMESTAMP

Enhanced fan-out provides dedicated throughput for each consumer.

Documentation  
https://docs.aws.amazon.com/streams/latest/dev/enhanced-consumers.html

Enhanced fan-out characteristics:

- Dedicated 2 MB/sec per consumer
- Lower consumer latency
- Reduced read contention

---

## Partition Key Design

Partition key determines shard placement.

Poor partition key design may cause:

- Hot shards
- Consumer lag
- Uneven scaling

Scaling approaches include:

- Increasing shard count
- Redesigning partition key
- Using on-demand capacity mode

Documentation  
https://docs.aws.amazon.com/streams/latest/dev/how-do-i-scale.html

> **EXAM TIP**  
> Kinesis scaling issues are often caused by poor partition key distribution.

---

## Kinesis Data Firehose

Documentation  
https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html

Firehose is a managed streaming delivery service.

Key characteristics:

- No shard management
- Delivers to S3, Redshift, OpenSearch, and HTTP endpoints
- Supports Lambda transformation
- Near real-time delivery
- Automatically scales

Firehose does not support replay capability.

Firehose is typically used for simplified streaming ingestion into data lakes.

> **EXAM TIP**  
> Managed streaming delivery with minimal operational overhead → Firehose.

---

## Amazon MSK

Documentation  
https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html

Amazon MSK provides managed Apache Kafka clusters.

Core characteristics:

- Partition-based scaling
- Consumer group offset management
- Persistent log storage
- Replay capability
- Kafka ecosystem compatibility

MSK Serverless supports automatic scaling for unpredictable workloads.

Documentation  
https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html

---

## Kinesis vs MSK

| Dimension | Kinesis | MSK |
|-----------|---------|-----|
| Scaling Unit | Shard | Partition |
| Replay | Yes | Yes |
| Offset Management | Managed by service | Consumer-managed |
| Kafka API Compatibility | No | Yes |

MSK is selected when Kafka protocol compatibility is required.

---

# Streaming Analytics

## Kinesis Data Analytics

Documentation  
https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html

Kinesis Data Analytics provides stream processing using Apache Flink.

Capabilities include:

- SQL-based streaming analytics
- Stateful stream processing
- Windowed aggregations
- Real-time transformation pipelines

Used for continuous analytics on streaming data.

---

## EMR Streaming

Documentation  
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html

EMR supports streaming analytics using:

- Apache Spark Streaming
- Apache Flink
- Custom frameworks

EMR streaming is used when:

- Custom frameworks are required
- Long-running analytics pipelines exist
- Operational control is necessary

---

# Data Lake Foundation

## Amazon S3 as Data Lake Storage

Documentation  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

S3 provides durable object storage for data lakes.

Key considerations:

- High durability
- Virtually unlimited storage
- Partitioning improves query performance
- Columnar formats such as Parquet and ORC reduce query cost

---

## AWS Glue

Documentation  
https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html

AWS Glue provides serverless ETL capabilities.

Capabilities include:

- Apache Spark-based data transformation
- Glue Crawlers for schema discovery
- Workflow orchestration
- Centralized metadata management

Glue Data Catalog acts as a central metadata repository.

Documentation  
https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html

The catalog is used by:

- Athena
- EMR
- Redshift Spectrum
- Lake Formation

---

## Lake Formation

Documentation  
https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html

Lake Formation adds governance controls to data lakes.

Capabilities include:

- Fine-grained access control at table and column level
- Tag-based access control using LF-Tags
- Cross-account data sharing
- Centralized permission management

Lake Formation is commonly used in enterprise multi-account data lakes.

---

# Batch Processing

## Amazon EMR

Documentation  
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-overview.html

EMR provides managed big data processing.

Supported frameworks include:

- Apache Spark
- Hadoop
- Hive
- Presto

Deployment models include:

- EMR on EC2
- EMR on EKS
- EMR Serverless

Documentation  
https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/emr-serverless.html

EMR is commonly used when:

- Persistent Spark workloads are required
- Custom frameworks must be deployed
- Spot instance cost optimization is desired

---

## Glue vs EMR

| Feature | Glue | EMR |
|--------|------|-----|
| Serverless | Yes | Optional |
| Persistent Cluster | No | Yes |
| Operational Control | Low | High |
| Best For | Managed ETL | Custom big data workloads |

Glue minimizes operational management while EMR offers full control.

---

# Query Layer

## Amazon Athena

Documentation  
https://docs.aws.amazon.com/athena/latest/ug/what-is.html

Athena provides serverless SQL querying over S3 data.

Key characteristics:

- Pay-per-query pricing
- Charged per TB scanned
- Uses Glue Data Catalog
- Supports partitioning and compression
- Supports federated queries

Documentation  
https://docs.aws.amazon.com/athena/latest/ug/connectors-overview.html

Athena federation allows querying external sources such as:

- RDS
- DynamoDB
- External databases

Athena enables analytics without moving data.

---

## Amazon Redshift

Documentation  
https://docs.aws.amazon.com/redshift/latest/dg/welcome.html

Redshift is a massively parallel data warehouse.

Architecture includes:

- RA3 nodes separating compute and storage
- Managed storage tiering
- Workload Management
- Concurrency scaling
- Materialized views
- COPY command for S3 ingestion

Documentation  
https://docs.aws.amazon.com/redshift/latest/dg/r_RA3_nodes.html

RA3 nodes allow independent scaling of compute and storage.

---

## Redshift Data Sharing

Documentation  
https://docs.aws.amazon.com/redshift/latest/dg/datashare-overview.html

Redshift supports cross-cluster data sharing.

Characteristics:

- Data sharing without duplication
- Supports multiple consumer clusters
- Useful for multi-team analytics environments

---

## Redshift vs Athena

| Feature | Athena | Redshift |
|--------|--------|----------|
| Data Location | S3 | Internal storage + S3 Spectrum |
| Best For | Ad-hoc queries | Enterprise BI workloads |
| Concurrency | Moderate | High with scaling |
| Pricing | Per TB scanned | Provisioned or serverless |

Athena is optimized for exploratory queries while Redshift supports enterprise analytics workloads.

---

# OpenSearch

Documentation  
https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html

OpenSearch provides search and analytics capabilities.

Typical use cases include:

- Log analytics
- Full-text search
- Real-time dashboards
- Index-based queries

OpenSearch is not designed as a data warehouse.

| Use Case | Recommended Service |
|----------|--------------------|
| Enterprise BI dashboards | Redshift |
| Log analytics | OpenSearch |
| Ad-hoc SQL analytics | Athena |

---

# Lakehouse Architecture Patterns

Common enterprise data lake patterns include:

S3 + Glue + Athena  
Serverless data lake.

S3 + Firehose  
Streaming ingestion pipeline.

S3 + EMR  
Large-scale Spark processing.

S3 + Redshift Spectrum  
Hybrid lake and warehouse architecture.

Lake Formation  
Central governance layer.

---

# Analytics Decision Matrix

| Requirement | Recommended Service |
|-------------|---------------------|
| Real-time replayable streaming | Kinesis Data Streams |
| Kafka ecosystem compatibility | Amazon MSK |
| Minimal operational streaming ingestion | Firehose |
| Serverless ETL | AWS Glue |
| Persistent Spark cluster | EMR |
| SQL queries on S3 | Athena |
| Enterprise data warehouse | Redshift |
| Log analytics and search | OpenSearch |
| Fine-grained data lake governance | Lake Formation |

---

# Common Pitfalls

- Poor partition key distribution causing hot shards in Kinesis  
- Assuming Firehose supports replay  
- Using Lambda for heavy ETL workloads  
- Ignoring Redshift workload management configuration  
- Choosing Athena for high-concurrency BI workloads  
- Overlooking Glue Data Catalog as central metadata layer  
- Missing Lake Formation governance in multi-account lakes  
- Underestimating storage growth without RA3 nodes  
- Confusing OpenSearch with warehouse workloads  

---

# Design Considerations

Analytics architecture typically separates concerns into independent layers.

Key architectural layers include:

- Ingestion layer
- Processing layer
- Storage layer
- Query layer
- Governance layer

Each layer should scale independently to support evolving data volume, analytics complexity, and governance requirements.
