# AWS Analytics Notes (SAP-C02 Level)

Kinesis • MSK • Glue • EMR • Athena • Redshift • Lake Formation • OpenSearch

These notes summarize streaming, batch, data lake, and warehouse architectures, focusing on scaling models, governance integration, replay behavior, and workload separation patterns.

---

## Streaming Ingestion

### Kinesis Data Streams (KDS)

**AWS Documentation:**  
https://docs.aws.amazon.com/streams/latest/dev/introduction.html  

Used for high-throughput real-time ingestion.

### Core Concepts

- Shard-based scaling  
- 1 MB/sec ingress per shard  
- 2 MB/sec egress per shard  
- 1,000 records/sec write per shard  
- Replay supported via TRIM_HORIZON / AT_TIMESTAMP  
- Enhanced Fan-Out provides dedicated 2 MB/sec per consumer  

**Enhanced Fan-Out Docs:**  
https://docs.aws.amazon.com/streams/latest/dev/enhanced-consumers.html  

---

### Partition Key Design

Partition key determines shard placement.

Poor key distribution causes:

- Hot shards  
- Consumer lag  
- Uneven scaling  

Scaling approaches:

- Increase shard count  
- Redesign partition key  
- Use On-Demand mode for unpredictable workloads  

**On-Demand Mode Docs:**  
https://docs.aws.amazon.com/streams/latest/dev/how-do-i-scale.html  

---

### Kinesis Data Firehose

**AWS Documentation:**  
https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html  

Managed streaming delivery service.

- No shard management  
- Delivers to S3, Redshift, OpenSearch, HTTP endpoints  
- Supports Lambda transformation  
- Near real-time delivery  
- No replay capability  

Used for low-operational streaming ingestion into data lakes.

---

### Amazon MSK (Managed Kafka)

**AWS Documentation:**  
https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html  

Managed Apache Kafka.

- Partition-based scaling  
- Consumer groups manage offsets  
- Supports exactly-once semantics  
- Serverless option for unpredictable workloads  

**MSK Serverless Docs:**  
https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html  

---

### Kinesis vs MSK Comparison

| Dimension | Kinesis | MSK |
|------------|----------|------|
| Scaling Unit | Shard | Partition |
| Replay | Yes | Yes |
| Offset Management | Managed | Consumer-managed |
| Kafka API Compatible | No | Yes |

MSK is selected when Kafka ecosystem compatibility is required.

---

## Streaming Analytics

### Kinesis Data Analytics (Apache Flink)

**AWS Documentation:**  
https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html  

- SQL or Flink-based stream processing  
- Windowed aggregations  
- Stateful processing  
- Fully managed  

---

### EMR Streaming

**AWS Documentation:**  
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html  

- Spark Streaming  
- Custom frameworks  
- Long-running or complex pipelines  
- More operational control  

---

## Data Lake Foundation

### S3 as Data Lake Storage

**AWS Documentation:**  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html  

- Durable object storage  
- Partitioning improves performance  
- Columnar formats (Parquet, ORC) reduce cost  

---

### AWS Glue

**AWS Documentation:**  
https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html  

Serverless ETL service.

### Capabilities

- Spark-based transformations  
- Glue Crawlers for schema discovery  
- Glue Workflows  
- Central Data Catalog  

**Glue Data Catalog Docs:**  
https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html  

Glue Data Catalog acts as central metadata store for:

- Athena  
- EMR  
- Redshift Spectrum  
- Lake Formation  

---

### Lake Formation

**AWS Documentation:**  
https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html  

Adds governance layer to S3 data lakes.

- Fine-grained access control (table/column-level)  
- LF-Tags for classification  
- Cross-account sharing  
- Centralized permission management  

Used in enterprise multi-account data lakes.

---

## Batch Processing

### EMR

**AWS Documentation:**  
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-overview.html  

Managed Hadoop/Spark cluster.

Deployment models:

- EMR on EC2  
- EMR on EKS  
- EMR Serverless  

**EMR Serverless Docs:**  
https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/emr-serverless.html  

Used when:

- Persistent heavy Spark workloads  
- Custom big data frameworks  
- Spot-based cost optimization  

---

### Glue vs EMR Comparison

| Feature | Glue | EMR |
|----------|------|------|
| Serverless | Yes | Optional |
| Persistent Cluster | No | Yes |
| Operational Control | Low | High |
| Best For | Managed ETL | Large-scale custom processing |

---

## Query Layer

### Athena

**AWS Documentation:**  
https://docs.aws.amazon.com/athena/latest/ug/what-is.html  

Serverless SQL over S3.

- Charged per TB scanned  
- Optimized with partitioning and compression  
- Uses Glue Data Catalog  
- Supports federated queries  

**Athena Federation Docs:**  
https://docs.aws.amazon.com/athena/latest/ug/connectors-overview.html  

Allows querying:

- RDS  
- DynamoDB  
- External sources via connectors  

Used for SQL access without moving data.

---

### Amazon Redshift

**AWS Documentation:**  
https://docs.aws.amazon.com/redshift/latest/dg/welcome.html  

Massively parallel data warehouse.

### Architecture

- RA3 nodes separate compute and storage  
- Managed storage tiering  
- Concurrency Scaling  
- Workload Management (WLM)  
- Materialized Views  
- COPY command for fast S3 ingestion  

**RA3 & Managed Storage Docs:**  
https://docs.aws.amazon.com/redshift/latest/dg/r_RA3_nodes.html  

---

### Redshift Data Sharing

**AWS Documentation:**  
https://docs.aws.amazon.com/redshift/latest/dg/datashare-overview.html  

- Share data across clusters  
- No duplication  
- Useful in multi-team analytics environments  

---

### Redshift vs Athena

| Feature | Athena | Redshift |
|----------|---------|-----------|
| Data Location | S3 | Internal + S3 (Spectrum) |
| Best For | Ad-hoc queries | Enterprise BI |
| Concurrency | Moderate | High (with scaling) |
| Pricing | Per TB scanned | Provisioned / Serverless |

---

## OpenSearch

**AWS Documentation:**  
https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html  

Used for:

- Log analytics  
- Real-time search  
- Text indexing  
- Dashboarding  

Not a replacement for Redshift.

| Use Case | Choose |
|-----------|--------|
| BI dashboards | Redshift |
| Log search | OpenSearch |
| Ad-hoc SQL | Athena |

---

## Lakehouse Architecture Patterns

Common enterprise patterns:

- S3 + Glue + Athena → Serverless lake  
- S3 + Firehose → Streaming ingestion  
- S3 + EMR → Heavy Spark ETL  
- S3 + Redshift Spectrum → Hybrid lake + warehouse  
- Lake Formation → Governance layer  

---

## Analytics Decision Matrix

| Requirement | Recommended Service |
|-------------|---------------------|
| Real-time replayable stream | Kinesis Streams |
| Kafka compatibility | MSK |
| Minimal streaming ops | Firehose |
| Serverless ETL | Glue |
| Persistent Spark cluster | EMR |
| SQL on S3 | Athena |
| Enterprise BI warehouse | Redshift |
| Log analytics | OpenSearch |
| Fine-grained lake governance | Lake Formation |

---

## Common Pitfalls Observed in Analytics Architectures

- Poor partition key causing hot shards in Kinesis  
- Assuming Firehose supports replay  
- Using Lambda for heavy ETL instead of Glue/EMR  
- Ignoring Redshift WLM configuration  
- Choosing Athena for high-concurrency BI workloads  
- Forgetting Glue Data Catalog central role  
- Not implementing Lake Formation in multi-account lakes  
- Underestimating storage growth without RA3 nodes  
- Confusing OpenSearch with warehouse workloads  

Analytics design becomes clearer when separated into:

- Ingestion layer  
- Processing layer  
- Storage layer  
- Query layer  
- Governance layer  

Each layer should scale independently.