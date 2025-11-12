---
title: "Module 11 – AWS RDS and DynamoDB (Relational vs. NoSQL Databases)"
date: 2025-11-08
categories: aws database sql nosql
tags: aws rds dynamodb mysql postgres scaling replication
image: /assets/images/rds.jpg
description: "Comparing relational and NoSQL databases with RDS and DynamoDB; replication, scaling, and automation."
---

---

#  Module 11 – AWS RDS and DynamoDB (Relational vs. NoSQL Databases) 🧮

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand the difference between **Relational (SQL)** and **NoSQL** databases.
* Deploy and manage **Amazon RDS** instances (MySQL/PostgreSQL/Oracle).
* Explore **RDS Multi-AZ**, **Read Replicas**, and **Automated Backups**.
* Work with **Amazon DynamoDB** for serverless NoSQL workloads.
* Optimize database performance and cost with scaling and caching.

---

## ☁️ Relational vs. NoSQL Overview

| **Aspect**          | **Relational (RDS)**              | **NoSQL (DynamoDB)**                 |
| ------------------- | --------------------------------- | ------------------------------------ |
| **Data Model**      | Structured tables (rows/columns)  | Key-value or document store          |
| **Schema**          | Fixed                             | Flexible                             |
| **Scaling**         | Vertical + Read Replicas          | Horizontal (automatic)               |
| **Transactions**    | ACID compliant                    | Eventually consistent (can be tuned) |
| **Best For**        | Financial systems, ERP, analytics | IoT, gaming, e-commerce catalogs     |
| **Service Example** | Amazon RDS (MySQL, Oracle, etc.)  | Amazon DynamoDB                      |

> 💡 **Analogy:**
>
> * **RDS** is like a **well-organized library** — books sorted by category, author, and ISBN.
> * **DynamoDB** is like a **warehouse** — you can quickly fetch any item by ID without searching through structured shelves.

---

## 🧩 Amazon RDS Overview    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/1641de05-08ec-439d-9624-63af652b06d4" />


**Amazon Relational Database Service (RDS)** makes it easy to set up, operate, and scale relational databases in the cloud.

| **Supported Engines** | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora           |
| --------------------- | ---------------------------------------------------------------- |
| **Features**          | Automated backups, Multi-AZ, Read replicas, Encryption, IAM auth |

---

## 🧠 RDS Architecture

```
       ┌──────────────────────┐
       │   Primary DB (AZ-A)  │
       │  Read/Write Traffic  │
       └──────────┬───────────┘
                  │
        Synchronous Replication
                  │
       ┌──────────▼──────────┐
       │ Standby DB (AZ-B)   │
       │  Failover Target    │
       └─────────────────────┘
```

---

## 🧪 Hands-On Lab 1: Launch an RDS MySQL Instance

### 🧰 **Objective**

Create and connect to an RDS database from EC2 or local MySQL client.

---

### **Step 1: Create RDS Database**

1. Go to **AWS Console → RDS → Databases → Create Database**.
2. Choose **Standard create**.
3. Engine: **MySQL**.
4. Template: **Free Tier**.
5. DB instance identifier: `myrdslab`.
6. Credentials:

   * Username: `admin`
   * Password: `<yourpassword>`
7. Storage: 20 GB (gp2).
8. Availability: Single-AZ (for lab).
9. Connectivity:

   * VPC: your default or custom VPC
   * Public access: **Yes**
   * Security group: allow inbound port **3306**.
10. Click **Create Database**.

---

### **Step 2: Connect to RDS**

Once available, copy the **endpoint** from the RDS dashboard.

```bash
mysql -h myrdslab.abcdefghijk.ap-southeast-2.rds.amazonaws.com \
      -u admin -p
```

✅ You are now connected to your managed MySQL instance.

---

### **Step 3: Create Database and Table**

```sql
CREATE DATABASE demo;
USE demo;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

INSERT INTO users (name, email)
VALUES ('Monowar', 'monowar@example.com'),
       ('CloudUser', 'clouduser@demo.com');

SELECT * FROM users;
```

---

## 🧠 RDS Backup, Scaling, and Monitoring

| **Feature**              | **Description**                                |
| ------------------------ | ---------------------------------------------- |
| **Automated Backups**    | Daily snapshots + transaction logs.            |
| **Manual Snapshots**     | User-initiated backups for retention.          |
| **Multi-AZ Deployment**  | High availability via synchronous replication. |
| **Read Replicas**        | Asynchronous replicas for scaling reads.       |
| **Performance Insights** | Visualize query latency and resource usage.    |
| **Encryption**           | KMS-based encryption at rest and in transit.   |

---

## 🧪 Hands-On Lab 2: Create a Read Replica

### **Objective**

Improve read scalability and performance.

1. Select your primary RDS → Click **Actions → Create Read Replica**.
2. Choose the same instance type, enable **Public Access**, and launch.
3. Once ready, connect using:

   ```bash
   mysql -h myrdslab-replica.ap-southeast-2.rds.amazonaws.com -u admin -p
   ```
4. Run:

   ```sql
   SHOW SLAVE STATUS\G
   ```

✅ You’ll see replication running.

**Read replicas** can be promoted to standalone instances if the primary fails.

---

## 🧩 RDS Security Best Practices

* Use **IAM authentication** instead of hardcoded passwords.
* Enable **Encryption at Rest (KMS)** and **SSL in Transit**.
* Restrict access using **Security Groups** and **Subnet Groups**.
* Rotate credentials periodically with **Secrets Manager**.
* Enable **Enhanced Monitoring** for CPU, memory, and disk usage.

---

## ⚙️ Automating with CloudFormation (RDS)

Example snippet to create RDS MySQL:

```yaml
MyRDSInstance:
  Type: AWS::RDS::DBInstance
  Properties:
    DBInstanceIdentifier: myrdslab
    Engine: MySQL
    MasterUsername: admin
    MasterUserPassword: MySecurePass123
    DBInstanceClass: db.t3.micro
    AllocatedStorage: 20
    PubliclyAccessible: true
```

Deploy using:

```bash
aws cloudformation create-stack \
  --stack-name MyRDSStack \
  --template-body file://rds-template.yaml
```

---

## 🧩 Amazon DynamoDB Overview    <img width="51" height="50" alt="image" src="https://github.com/user-attachments/assets/27b7483e-52b3-40ce-aafd-f577c4cd273f" />


**Amazon DynamoDB** is a fully managed **NoSQL** database that delivers single-digit millisecond performance at any scale.

| **Feature**     | **Description**                     |
| --------------- | ----------------------------------- |
| **Data Model**  | Key-Value and Document              |
| **Performance** | Single-digit ms latency             |
| **Scaling**     | On-demand or provisioned throughput |
| **Durability**  | Multi-AZ replication by default     |
| **Integration** | Lambda, Streams, API Gateway        |

---

## 🧪 Hands-On Lab 3: Create DynamoDB Table and Insert Data

### 🧰 **Objective**

Store and retrieve data using DynamoDB CLI.

---

### **Step 1: Create a Table**

```bash
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions AttributeName=UserID,AttributeType=S \
  --key-schema AttributeName=UserID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

---

### **Step 2: Insert Items**

```bash
aws dynamodb put-item \
  --table-name Users \
  --item '{"UserID": {"S": "1001"}, "Name": {"S": "Monowar"}, "Email": {"S": "monowar@example.com"}}'

aws dynamodb put-item \
  --table-name Users \
  --item '{"UserID": {"S": "1002"}, "Name": {"S": "CloudUser"}, "Email": {"S": "clouduser@demo.com"}}'
```

---

### **Step 3: Retrieve Items**

```bash
aws dynamodb scan --table-name Users
aws dynamodb get-item \
  --table-name Users \
  --key '{"UserID": {"S": "1001"}}'
```

✅ DynamoDB scales automatically and provides consistent low-latency responses.

---

## 🧩 DynamoDB Streams and Triggers

* **Streams** capture table-level changes (INSERT, MODIFY, REMOVE).
* Can trigger **AWS Lambda** functions for real-time processing.

Example:

> When a new record is added to “Orders” table, a Lambda function sends an email confirmation.

---

## 🧠 DynamoDB Advanced Features

| **Feature**                    | **Description**                                   |
| ------------------------------ | ------------------------------------------------- |
| **Global Tables**              | Multi-region replication for global applications. |
| **DAX (DynamoDB Accelerator)** | In-memory cache for microsecond latency.          |
| **TTL (Time to Live)**         | Automatically delete expired items.               |
| **Transactions**               | ACID-compliant multi-item operations.             |
| **On-Demand Mode**             | Automatic scaling without capacity planning.      |

---

## ⚙️ RDS vs DynamoDB – Quick Comparison

| **Feature**        | **Amazon RDS**                        | **Amazon DynamoDB**           |
| ------------------ | ------------------------------------- | ----------------------------- |
| **Data Model**     | Relational                            | NoSQL (Key-Value)             |
| **Scaling**        | Manual (Read Replicas, instance type) | Automatic                     |
| **Query Language** | SQL                                   | API-based (Query/Scan)        |
| **Latency**        | Milliseconds                          | Sub-milliseconds              |
| **Cost Model**     | Instance-hours + storage              | Read/Write capacity + storage |
| **Use Case**       | Traditional apps, ERP, BI             | IoT, gaming, serverless apps  |

---

## 🧾 Key Takeaways

* **Amazon RDS** simplifies relational database management — with built-in HA, backups, and scaling.
* **Multi-AZ** provides automatic failover; **Read Replicas** enhance performance.
* **Amazon DynamoDB** offers serverless, low-latency NoSQL storage at scale.
* Choose **RDS** for structured data and **DynamoDB** for flexible, high-speed access.
* Both integrate with **Lambda, CloudWatch, and CloudFormation** for full automation.

---

> 🪄 **Next Module Preview:**
> In **Module 12**, we’ll explore **AWS Monitoring and Logging (CloudWatch, CloudTrail, and Config)** — covering metrics, alerts, audit trails, and compliance tracking across your cloud environment.

---
