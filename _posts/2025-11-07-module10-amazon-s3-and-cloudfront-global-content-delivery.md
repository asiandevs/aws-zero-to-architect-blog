# ──────────────────────────────────────────────────────────────
# MODULE 10
# File: _posts/2025-11-07-module10-amazon-s3-and-cloudfront-global-content-delivery.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 10: Amazon S3 & CloudFront (Global Delivery)"
date: 2025-11-07
categories: aws s3 cloudfront cdn
tags: cloudfront cdn s3-static https
image: /assets/images/cloudfront.jpg
description: "Static hosting, CDN distribution, HTTPS setup."

prev:
  title: "Module 9: AWS Lambda & API Gateway (Serverless)"
  url: "/_posts/2025-11-06-module9-aws-lambda-and-api-gateway-serverless-architecture.md"
next:
  title: "Module 11: AWS RDS & DynamoDB (Relational vs NoSQL)"
  url: "/_posts/2025-11-08-module11-aws-rds-and-dynamodb-relational-vs-nosql.md"
---

#  Module 10 – Amazon S3 and CloudFront (Global Content Delivery & Security) 🌎

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand **Amazon S3** storage architecture and classes.
* Configure **S3 buckets**, **versioning**, and **lifecycle policies**.
* Host a **static website** in S3.
* Distribute content globally using **Amazon CloudFront**.
* Secure content delivery using **HTTPS, OAC (Origin Access Control)**, and **Route 53** integration.

---

## ☁️ What Is Amazon S3?    <img width="50" height="53" alt="image" src="https://github.com/user-attachments/assets/e39e99e9-11de-4ce1-b37e-9863cc166ad6" />


**Amazon Simple Storage Service (S3)** provides **object storage** built for scalability, availability, and durability (11 nines: 99.999999999%).

> 💡 **Analogy:**
> Think of S3 as your **infinite online hard drive** — you can store, retrieve, and share any amount of data anytime, anywhere, without worrying about capacity or maintenance.

---

## 🧩 S3 Concepts

| **Term**          | **Description**                          |
| ----------------- | ---------------------------------------- |
| **Bucket**        | Container for objects (files).           |
| **Object**        | Data file + metadata stored in a bucket. |
| **Key**           | Unique identifier for each object.       |
| **Region**        | Buckets are created within AWS regions.  |
| **Versioning**    | Keeps multiple versions of an object.    |
| **Storage Class** | Determines cost and access frequency.    |

---

## 💾 S3 Storage Classes

| **Storage Class**                | **Use Case**                           | **Availability** | **Durability** |
| -------------------------------- | -------------------------------------- | ---------------- | -------------- |
| **S3 Standard**                  | Frequently accessed data               | 99.99%           | 99.999999999%  |
| **S3 Intelligent-Tiering**       | Automatically moves data between tiers | 99.9%            | 99.999999999%  |
| **S3 Standard-IA**               | Infrequent access                      | 99.9%            | 99.999999999%  |
| **S3 One Zone-IA**               | Non-critical infrequent data           | 99.5%            | 99.999999999%  |
| **S3 Glacier Instant Retrieval** | Archival, instant retrieval            | 99.9%            | 99.999999999%  |
| **S3 Glacier Deep Archive**      | Long-term archival                     | 99.9%            | 99.999999999%  |

> 🧠 **Tip:** Choose the right class based on access frequency, retrieval time, and cost optimization.

---

## 🧪 Hands-On Lab 1: Create and Upload to an S3 Bucket

### 🧰 **Objective**

Create a new bucket, upload files, and configure public access.

---

### **Step 1: Create a Bucket**

1. Go to **S3 Console → Create Bucket**.
2. Name: `mycloudlab-s3-demo`.
3. Region: `ap-southeast-2 (Sydney)`.
4. Uncheck **Block all public access** (for demo purposes).
5. Click **Create Bucket**.

---

### **Step 2: Upload an Object**

1. Open the bucket → Click **Upload → Add Files**.
2. Select `index.html` (your simple webpage).
3. Click **Upload**.

---

### **Step 3: Test Public Access**

1. Select `index.html` → Click **Copy URL**.
2. Open it in your browser:

   ```
   https://mycloudlab-s3-demo.s3.ap-southeast-2.amazonaws.com/index.html
   ```

✅ You’ll see your webpage content hosted from S3.

---

### **CLI Example**

```bash
aws s3 mb s3://mycloudlab-s3-demo
aws s3 cp index.html s3://mycloudlab-s3-demo/
aws s3 ls s3://mycloudlab-s3-demo/
```

---

## 🧠 Enabling S3 Versioning and Lifecycle Policies

1. Go to **Bucket → Properties → Versioning → Enable**.
2. Add a **Lifecycle Rule**:

   * Transition to **S3 Glacier** after 30 days.
   * Expire non-current versions after 90 days.

This automates cost optimization and data retention.

---

## 🌐 Hosting a Static Website in S3

### **Step 1: Upload Website Files**

Upload `index.html` and `error.html` to your bucket.

### **Step 2: Enable Static Website Hosting**

1. Go to **Bucket → Properties → Static Website Hosting → Enable**.
2. Index document: `index.html`
3. Error document: `error.html`

### **Step 3: Get Website Endpoint**

Copy the endpoint:

```
http://mycloudlab-s3-demo.s3-website-ap-southeast-2.amazonaws.com
```

---

## 🧪 Hands-On Lab 2: Secure Content with CloudFront

### 🧰 **Objective**

Distribute your S3 website globally via **CloudFront** for better speed, caching, and HTTPS.

---

### **Step 1: Create CloudFront Distribution**

1. Go to **CloudFront Console → Create Distribution**.
2. Origin domain: Select your S3 bucket.
3. **Origin Access**:

   * Choose **Origin Access Control (OAC)** → Create new → Attach policy automatically.
4. Viewer protocol policy: **Redirect HTTP to HTTPS**.
5. Cache policy: **CachingOptimized**.
6. Click **Create Distribution**.

---

### **Step 2: Wait for Deployment**

After ~10–15 minutes, your CloudFront distribution is ready.
Copy the **Distribution Domain Name**, e.g.:

```
https://d123abcd.cloudfront.net
```

✅ Accessing this URL serves your S3 website globally with caching and HTTPS.

---

### **Step 3: Restrict Direct S3 Access**

To ensure security:

1. Edit your S3 bucket policy to **allow access only via CloudFront OAC**.
2. Example Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "cloudfront.amazonaws.com"
    },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::mycloudlab-s3-demo/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E123ABCXYZ"
      }
    }
  }]
}
```

Now your S3 bucket is private, and only CloudFront can serve content.

---

## 🧩 Optional: Integrate Route 53 with CloudFront

### **Objective**

Use your own domain (e.g., `www.mycloudlabdemo.com`) for the CloudFront distribution.

---

### **Step 1: Add CNAME**

In CloudFront → Edit Distribution:

* Alternate Domain Name: `www.mycloudlabdemo.com`

### **Step 2: Attach SSL Certificate**

* Use **AWS Certificate Manager (ACM)** → Request a public certificate for your domain.
* Validate via **DNS record in Route 53**.
* Attach certificate to CloudFront.

### **Step 3: Create DNS Alias**

In **Route 53 Hosted Zone**, create an **A Record (Alias)** pointing to your CloudFront distribution.

✅ Now your static site is accessible securely at:

```
https://www.mycloudlabdemo.com
```

---

## ⚙️ CloudFront Architecture Overview    <img width="70" height="50" alt="image" src="https://github.com/user-attachments/assets/10212dfa-8826-4c8c-9f3c-5d7424b5e3f9" />


```
          ┌──────────────┐
          │    Users     │
          └──────┬───────┘
                 │
         ┌───────▼───────┐
         │ CloudFront CDN │
         │ (Edge Locations)│
         └───────┬────────┘
                 │
       ┌─────────▼──────────┐
       │   S3 Origin (OAC)  │
       └────────────────────┘
```

> 💡 **Analogy:**
> Think of **CloudFront** as a global network of mirrors — when a user requests a file, it’s served from the nearest mirror (edge location) for faster load times.

---

## 🧠 CloudFront Key Concepts

| **Term**                        | **Description**                                           |
| ------------------------------- | --------------------------------------------------------- |
| **Edge Location**               | Global data centers caching content near users.           |
| **Origin**                      | The source of truth (S3, ALB, EC2, etc.).                 |
| **Distribution**                | Configuration that defines how CloudFront serves content. |
| **Cache Policy**                | Controls caching behavior (TTL, headers, cookies).        |
| **Origin Access Control (OAC)** | Secures origin by allowing CloudFront-only access.        |

---

## 🧪 Hands-On Lab 3: Invalidate Cache

When you update S3 objects, the old version may still be cached in CloudFront.

**CLI Example:**

```bash
aws cloudfront create-invalidation \
  --distribution-id E123ABCXYZ \
  --paths "/*"
```

✅ This clears cached objects and forces CloudFront to fetch the latest files.

---

## 🧾 Key Takeaways

* **Amazon S3** provides scalable, durable, and cost-efficient object storage.
* **Versioning** and **Lifecycle Policies** automate retention and cost management.
* You can host **static websites** directly in S3.
* **Amazon CloudFront** delivers your content globally with caching and HTTPS.
* **OAC (Origin Access Control)** ensures private, secure S3 origins.
* Combine **S3 + CloudFront + Route 53 + ACM** for a production-grade global web presence.

---

> 🪄 **Next Module Preview:**
> In **Module 11**, we’ll dive into **AWS RDS and DynamoDB** — exploring relational vs. NoSQL database services, replication, backup, and performance tuning.

---
