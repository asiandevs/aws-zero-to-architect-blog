---
title: "Module 19 – AWS Well-Architected Framework"
date: 2025-11-16
categories: aws architecture best-practices governance
tags: aws well-architected pillars reliability security cost-optimization
image: /assets/images/well-architected.jpg
description: "Understanding the AWS Well-Architected Framework, its five pillars, and best practices for secure, efficient, and reliable architectures."
---

---

# 📊 Module 19 – AWS Data Analytics and Big Data Architecture

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand AWS **data analytics ecosystem and architecture patterns**.
* Ingest, store, and transform large datasets using **Kinesis**, **S3**, and **Glue**.
* Query data using **Athena** and **Redshift Spectrum**.
* Design a **data lake architecture** with **Lake Formation**.
* Implement **ETL pipelines**, **data cataloging**, and **governance**.

---

## ☁️ The Modern Data Architecture on AWS

> 💡 **Analogy:**
> Think of AWS data analytics like a **factory** —
>
> * *Kinesis* collects raw materials (data streams).
> * *Glue* refines and organizes them.
> * *S3* is the warehouse.
> * *Athena* and *Redshift* are the analytics machines that extract insights.

---

## 🧩 Core AWS Analytics Services Overview

| **Layer**          | **Service**                         | **Purpose**                              |
| ------------------ | ----------------------------------- | ---------------------------------------- |
| **Data Ingestion** | Kinesis Data Streams, Firehose, DMS | Collect and stream data                  |
| **Storage**        | Amazon S3, Glacier, EFS             | Centralized data lake storage            |
| **ETL & Catalog**  | AWS Glue, Lake Formation            | Data discovery, cleaning, transformation |
| **Analytics**      | Athena, Redshift, EMR               | Query and analyze data                   |
| **Visualization**  | QuickSight                          | Dashboards and insights                  |

---

## 🧠 Data Lake Design Pattern

A **data lake** is a centralized repository for structured and unstructured data, stored in native format and processed as needed.

**Architecture Flow:**

```
┌───────────────┐
│ Data Sources  │  →  IoT, RDBMS, APIs, Logs
└───────┬───────┘
        ↓
┌───────────────┐
│ Kinesis / DMS │  → Streaming / Batch Ingestion
└───────┬───────┘
        ↓
┌───────────────┐
│ Amazon S3     │  → Raw & Processed Storage (Data Lake)
└───────┬───────┘
        ↓
┌───────────────┐
│ AWS Glue / ETL│  → Transform & Catalog
└───────┬───────┘
        ↓
┌───────────────┐
│ Athena / Redshift│  → Query & Analysis
└───────┬───────┘
        ↓
┌───────────────┐
│ QuickSight    │  → Visualization
└───────────────┘
```

---

## 🧪 Hands-On Lab 1: Build a Serverless Data Lake (S3 + Glue + Athena)

### 🧰 **Objective**

Set up a data lake on AWS, transform raw data, and run SQL queries using Athena.

---

### **Step 1: Create S3 Buckets**

```bash
aws s3 mb s3://data-lake-raw-123
aws s3 mb s3://data-lake-processed-123
```

Upload sample CSV or JSON files to `data-lake-raw-123`.

---

### **Step 2: Create AWS Glue Crawler**

1. Go to **AWS Glue → Crawlers → Add Crawler**.
2. Data source: `s3://data-lake-raw-123`.
3. IAM Role: Create `GlueCrawlerRole` with `AmazonS3FullAccess` + `AWSGlueServiceRole`.
4. Run the crawler.

✅ Glue automatically detects schema and stores metadata in the **Data Catalog**.

---

### **Step 3: Query with Amazon Athena**

1. Go to **Athena → Query Editor**.
2. Database: select the one created by Glue.
3. Run a query:

   ```sql
   SELECT region, COUNT(*) AS sales_count
   FROM sales_data
   GROUP BY region;
   ```

✅ You’ve built a **serverless data lake pipeline** — raw data → catalog → insights.

---

## 🧩 AWS Glue ETL Jobs

**AWS Glue** is a fully managed ETL (Extract, Transform, Load) service.

| **Component**    | **Description**                                              |
| ---------------- | ------------------------------------------------------------ |
| **Crawlers**     | Discover and catalog datasets automatically.                 |
| **Jobs**         | Python/Spark ETL scripts.                                    |
| **Triggers**     | Schedule or event-driven job runs.                           |
| **Data Catalog** | Central metadata store used by Athena and Redshift Spectrum. |

---

### **CLI Example: Run Glue Job**

```bash
aws glue start-job-run --job-name TransformRawSalesData
```

**Sample PySpark Script (simplified):**

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from awsglue.context import GlueContext
from pyspark.context import SparkContext

sc = SparkContext()
glueContext = GlueContext(sc)
datasource = glueContext.create_dynamic_frame.from_catalog(database="sales_db", table_name="raw_sales")
filtered = Filter.apply(frame=datasource, f=lambda x: x["amount"] > 100)
glueContext.write_dynamic_frame.from_options(frame=filtered, connection_type="s3", connection_options={"path": "s3://data-lake-processed-123"}, format="parquet")
```

✅ This script filters sales > $100 and stores the result as Parquet in the processed bucket.

---

## 🧩 Amazon Redshift – Data Warehouse

**Redshift** is AWS’s fully managed, petabyte-scale data warehouse.

| **Feature**                             | **Description**                              |
| --------------------------------------- | -------------------------------------------- |
| **Columnar Storage**                    | Faster analytical queries.                   |
| **Massively Parallel Processing (MPP)** | Distributes workloads across clusters.       |
| **Redshift Spectrum**                   | Query directly from S3 without loading data. |
| **Concurrency Scaling**                 | Automatically adds query capacity on demand. |

---

### **Hands-On: Query S3 Data Using Redshift Spectrum**

1. Create external schema:

   ```sql
   CREATE EXTERNAL SCHEMA s3_data
   FROM DATA CATALOG DATABASE 'sales_db'
   IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftRole'
   CREATE EXTERNAL DATABASE IF NOT EXISTS;
   ```

2. Query external data:

   ```sql
   SELECT * FROM s3_data.sales_data WHERE region = 'NSW';
   ```

✅ You can query S3 data directly without importing — ideal for mixed analytics workloads.

---

## 🧩 Amazon Kinesis – Real-Time Data Ingestion

| **Service**                | **Purpose**                                                  |
| -------------------------- | ------------------------------------------------------------ |
| **Kinesis Data Streams**   | Custom real-time stream processing (millisecond latency).    |
| **Kinesis Firehose**       | Fully managed, delivers data to S3, Redshift, or OpenSearch. |
| **Kinesis Data Analytics** | SQL queries on live streams.                                 |

---

### **CLI Example: Create Kinesis Stream**

```bash
aws kinesis create-stream --stream-name realTimeSales --shard-count 1
```

**Send data:**

```bash
aws kinesis put-record \
  --stream-name realTimeSales \
  --partition-key "sales" \
  --data "eyJpZCI6IjEyMyIsICJyZWdpb24iOiJOU1ciLCAiYW1vdW50IjogMjAwfQ=="
```

✅ Firehose can automatically deliver this data to S3 for further ETL with Glue.

---

## 🧩 AWS Lake Formation

Simplifies **data lake setup, permissions, and governance**.

| **Feature**                     | **Description**                                 |
| ------------------------------- | ----------------------------------------------- |
| **Data Catalog Integration**    | Central schema metadata.                        |
| **Fine-Grained Access Control** | Column-level access via IAM or federated users. |
| **Blueprints**                  | Predefined ETL workflows.                       |

---

### **Lab: Grant Fine-Grained Permissions**

1. Go to **Lake Formation → Permissions → Grant**.
2. Choose `Database: sales_db`.
3. Grant **SELECT** on specific columns (e.g., region, amount).
4. Assign to IAM role `DataAnalystRole`.

✅ Analysts can now query sensitive data with restricted column-level access.

---

## 🧩 Amazon QuickSight – Visualization Layer

QuickSight connects directly to Athena, Redshift, or S3 for interactive dashboards.

**Steps:**

1. Open **QuickSight → Manage Data → New Dataset**.
2. Choose **Athena** or **Redshift** as the source.
3. Create a dashboard with KPIs, heatmaps, or trend lines.

> 📈 Example: “Monthly Sales by Region” using Athena dataset.

---

## 🧠 Best Practices for Data Analytics on AWS

| **Category**           | **Recommendation**                                                   |
| ---------------------- | -------------------------------------------------------------------- |
| **Storage**            | Use S3 with lifecycle policies and Intelligent-Tiering.              |
| **ETL**                | Use Glue jobs with Parquet format for performance.                   |
| **Query Optimization** | Partition data by date or region.                                    |
| **Security**           | Enable encryption (KMS) and fine-grained Lake Formation permissions. |
| **Cost Control**       | Use Athena query cost controls and Redshift pause/resume features.   |
| **Automation**         | Use EventBridge to trigger ETL on new S3 uploads.                    |

---

## 🧾 Key Takeaways

* AWS provides a **complete data analytics ecosystem** — from ingestion to visualization.
* Build a **serverless data lake** with S3, Glue, and Athena.
* Use **Redshift Spectrum** for warehouse + lake integration.
* Stream real-time data with **Kinesis** and analyze it instantly.
* Manage permissions and governance using **Lake Formation**.
* Visualize data interactively using **QuickSight** dashboards.

---

> 🪄 **Next Module Preview:**
> In **Module 20**, we’ll conclude with **AWS Architecture Design Patterns and Exam Readiness** — consolidating design principles, best practices, and certification-focused architecture scenarios.

---
