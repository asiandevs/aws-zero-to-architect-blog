# ──────────────────────────────────────────────────────────────
# MODULE 4
# File: _posts/2025-10-31-module4-amazon-s3-simple-storage-service.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 4: Amazon S3: Simple Storage Service"
date: 2025-10-31
categories: aws storage s3
tags: s3 versioning lifecycle replication glacier
image: /assets/images/s3.jpg
description: "Object storage, versioning, lifecycle, replication."
---

# Module 4 - Amazon S3: Simple Storage Service 🪣

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Explain the purpose and key features of **Amazon S3**.
* Create, configure, and manage **S3 buckets** and **objects**.
* Implement **versioning, lifecycle management**, and **replication**.
* Understand **S3 storage classes** and **cost optimization**.
* Perform hands-on **S3 bucket creation, upload, access control, and cross-region replication**.

---

## ☁️ Introduction to Amazon S3  <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/c0318bdd-fdd4-4e3f-ba2e-385c6a3fcc42" />


**Amazon S3 (Simple Storage Service)** is an **object storage service** that offers industry-leading scalability, data availability, and security.
It allows you to **store and retrieve any amount of data** at any time, from anywhere on the web.

> 💡 **Analogy:**
> Think of S3 as an online Dropbox for your AWS environment — infinitely scalable, always available, and programmable via API.

---

## 🧱 Core Concepts

| **Term**        | **Definition**                                                  |
| --------------- | --------------------------------------------------------------- |
| **Bucket**      | A top-level container for storing objects (similar to folders). |
| **Object**      | The actual data (files) stored in S3.                           |
| **Key**         | The unique identifier for each object within a bucket.          |
| **Region**      | The physical location where your bucket and data are stored.    |
| **Versioning**  | Keeps multiple variants of an object in the same bucket.        |
| **Replication** | Automatically copies data across Regions for redundancy.        |

---

## 🗺️ S3 Global View

* **Buckets** are created in specific **Regions**, but **names are globally unique**.
* You can access S3 using:

  * AWS Management Console
  * AWS CLI
  * SDKs / REST API

---

## 🔑 Key Features of S3

| **Feature**                        | **Description**                                  |
| ---------------------------------- | ------------------------------------------------ |
| **Scalable Storage**               | Store unlimited data objects up to 5 TB each.    |
| **Durability**                     | 99.999999999% (11 nines) durability by default.  |
| **Availability**                   | 99.99% availability for Standard class.          |
| **Secure Access**                  | IAM policies, bucket policies, ACLs, encryption. |
| **Versioning**                     | Maintains previous versions of objects.          |
| **Cross-Region Replication (CRR)** | Syncs buckets across Regions.                    |
| **Lifecycle Management**           | Automatically transitions or deletes objects.    |
| **Static Website Hosting**         | Serve static content directly from S3.           |

---

## ⚙️ S3 Storage Classes

| **Storage Class**                   | **Durability** | **Availability** | **Use Case**                                       |
| ----------------------------------- | -------------- | ---------------- | -------------------------------------------------- |
| **Standard**                        | 11 nines       | 99.99%           | Frequently accessed data.                          |
| **Intelligent-Tiering**             | 11 nines       | 99.9%            | Automatically optimizes storage cost.              |
| **Standard-IA (Infrequent Access)** | 11 nines       | 99.9%            | Data that's infrequently accessed.                 |
| **One Zone-IA**                     | 11 nines       | 99.5%            | Backup data that doesn't need multi-AZ.            |
| **Glacier Instant Retrieval**       | 11 nines       | 99.9%            | Archive with immediate access.                     |
| **Glacier Flexible Retrieval**      | 11 nines       | 99.8%            | Long-term archive with minutes-to-hours retrieval. |
| **Glacier Deep Archive**            | 11 nines       | 99.8%            | Lowest-cost archival (12-hour retrieval).          |

---

## 🧪 Hands-On: Create and Configure an S3 Bucket      <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/4b6dc488-e63b-4b8c-8042-05afdc4723c7" />


### 🧰 **Objective**

Create a new S3 bucket, upload files, enable versioning, and set up lifecycle management.

---

### **Step 1: Create a Bucket**

1. Open **AWS Console → Services → S3 → Create Bucket**.
2. Enter a unique **Bucket Name** (e.g., `my-s3-lab-demo`).
3. Choose a **Region** (e.g., `ap-southeast-2`).
4. Leave **Block Public Access** enabled for now.
5. Click **Create Bucket**.

---

### **Step 2: Upload Files**

1. Open your new bucket → Click **Upload**.
2. Add files or folders (e.g., images, text files).
3. Click **Upload**.

**CLI Example:**

```bash
aws s3 cp localfile.txt s3://my-s3-lab-demo/
```

---

### **Step 3: Enable Versioning**

1. Go to **Bucket → Properties → Versioning → Edit → Enable**.
2. Upload the same file again with different content.
3. Go to **Objects → Show Versions** to view both copies.

**CLI Example:**

```bash
aws s3api put-bucket-versioning --bucket my-s3-lab-demo --versioning-configuration Status=Enabled
```

---

### **Step 4: Enable Static Website Hosting**

1. Go to **Properties → Static Website Hosting → Edit**.
2. Choose "Host a static website."
3. Set:

   * Index document: `index.html`
   * Error document: `error.html`
4. Upload both files to your bucket.
5. Note the **Endpoint URL** — your website is now live.

**CLI Example:**

```bash
aws s3 website s3://my-s3-lab-demo/ --index-document index.html --error-document error.html
```

---

## 🔐 S3 Access Control

S3 provides **three layers of access control**:

| **Mechanism**                 | **Scope**       | **Example Use**                                           |
| ----------------------------- | --------------- | --------------------------------------------------------- |
| **IAM Policy**                | User/Role level | Grant developers S3 access via IAM role.                  |
| **Bucket Policy**             | Bucket-wide     | Allow only specific IPs or accounts to access the bucket. |
| **ACL (Access Control List)** | Object-level    | Grant access to specific users for individual files.      |

> **Best Practice:** Use **IAM roles** and **bucket policies** — avoid public ACLs.

---

### **Example Bucket Policy (Read-Only Access)**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::my-s3-lab-demo/*"]
    }
  ]
}
```

Apply via **Permissions → Bucket Policy → Edit**.

---

## ♻️ S3 Lifecycle Management

Lifecycle policies automate the **transition or deletion of objects** based on their age.

| **Action**     | **Trigger**    | **Example**                          |
| -------------- | -------------- | ------------------------------------ |
| **Transition** | After 30 days  | Move from Standard → Standard-IA.    |
| **Archive**    | After 90 days  | Move from IA → Glacier Deep Archive. |
| **Expiration** | After 365 days | Permanently delete objects.          |

### **Hands-On: Create a Lifecycle Rule**

1. Navigate to **Management → Lifecycle Rules → Create Rule**.
2. Name: `Archive-After-30-Days`.
3. Filter: Prefix = `/logs/` (optional).
4. Transition:

   * Move to Standard-IA after 30 days.
   * Move to Glacier after 90 days.
5. Expire after 365 days.
6. Click **Create Rule**.

---

## 🌏 Cross-Region Replication (CRR)

**Goal:** Automatically copy objects between buckets in different Regions.

### **Requirements**

* Versioning must be enabled on both buckets.
* IAM permissions to replicate objects.

---

### **Hands-On: Enable CRR**

1. Create a **source bucket** (e.g., `my-s3-sydney`).
2. Create a **destination bucket** (e.g., `my-s3-oregon`).
3. Enable **Versioning** on both.
4. Go to **Source Bucket → Management → Replication → Add Rule**.
5. Set:

   * Source: entire bucket.
   * Destination: `my-s3-oregon`.
   * IAM Role: create a new replication role automatically.
6. Save the rule and upload files to the source bucket.
7. Verify replication in the destination bucket.

---

## 🧠 S3 Encryption Options

| **Type**                   | **Description**                                        | **Managed By** |
| -------------------------- | ------------------------------------------------------ | -------------- |
| **SSE-S3**                 | Server-side encryption with AWS-managed keys.          | AWS            |
| **SSE-KMS**                | Server-side encryption with KMS customer-managed keys. | Customer       |
| **SSE-C**                  | Customer-provided encryption keys.                     | Customer       |
| **Client-Side Encryption** | Data encrypted before upload.                          | Customer       |

**CLI Example (SSE-S3):**

```bash
aws s3 cp report.csv s3://my-s3-lab-demo/ --sse AES256
```

---

## 📊 S3 Monitoring and Logging

| **Feature**                | **Purpose**                                 |
| -------------------------- | ------------------------------------------- |
| **S3 Storage Lens**        | Provides organization-wide usage analytics. |
| **Access Logs**            | Track who accessed your bucket and when.    |
| **CloudTrail Integration** | Record API-level activity for auditing.     |
| **CloudWatch Metrics**     | Monitor storage, requests, and errors.      |

Enable these under **Bucket → Properties → Metrics / Logging**.

---

## 🧩 Real-World Use Case: Static Website Hosting with Replication

**Scenario:**
A company hosts marketing content in Sydney (primary) and replicates it to Oregon for disaster recovery.

### **Steps Summary:**

1. Create `sydney-marketing-bucket` (ap-southeast-2).
2. Enable versioning and upload website files.
3. Create destination bucket `oregon-marketing-bucket`.
4. Enable CRR between the two.
5. Configure both for **static website hosting**.
6. Access site via public endpoint:

   ```
   http://sydney-marketing-bucket.s3-website-ap-southeast-2.amazonaws.com
   ```

✅ Result: Fault-tolerant static website hosted globally via S3.

---

## 🧾 Key Takeaways

* **Amazon S3** stores objects (not files) in buckets.
* Buckets are **regionally created** but **globally unique**.
* Use **Versioning**, **Lifecycle Rules**, and **CRR** for durability and efficiency.
* **Storage Classes** balance cost vs. access frequency.
* **Encryption** and **policies** safeguard data integrity.
* **S3** can act as a **web server**, **backup store**, or **data lake**.

---

> 🪄 **Next Module Preview:**
> In **Module 5**, we'll explore **Networking with Amazon VPC**, including Subnets, Route Tables, Internet Gateways, NAT Gateways, and Security Groups — with detailed hands-on labs to build your first AWS network.

---
