---
title: "Module 17 – AWS Backup and Disaster Recovery"
date: 2025-11-14
categories: aws backup dr business-continuity
tags: aws backup drs cross-region recoverypoint
image: /assets/images/backup.jpg
description: "Designing resilient backup and disaster recovery strategies using AWS Backup, cross-region replication, and DR planning."
---

---

# 💰 Module 16 – AWS Cost Optimization and FinOps Practices

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand AWS **pricing models and billing concepts**.
* Use **Cost Explorer**, **Budgets**, and **Cost Anomaly Detection** for visibility.
* Apply **FinOps best practices** to manage cloud spend efficiently.
* Identify cost-saving opportunities using **Right-Sizing**, **Savings Plans**, and **Spot Instances**.
* Implement **cost governance** using **tags**, **service control policies**, and **AWS Organizations**.

---

## ☁️ What Is FinOps?

**FinOps** (Cloud Financial Operations) is the practice of bringing **financial accountability** to variable cloud spending — a collaboration between **finance, engineering, and business**.

> 💡 **Analogy:**
> Think of FinOps as **“DevOps for cloud money”** — it ensures that cloud resources deliver maximum value without overspending.

---

## 🧩 AWS Pricing Fundamentals

| **Pricing Model**            | **Description**                                   | **Example**                        |
| ---------------------------- | ------------------------------------------------- | ---------------------------------- |
| **On-Demand**                | Pay per use, no long-term commitment.             | EC2 t3.micro by the hour.          |
| **Reserved Instances (RIs)** | Commit for 1–3 years for lower price.             | 40%+ savings for steady workloads. |
| **Savings Plans**            | Flexible discount model across instance families. | EC2 + Fargate + Lambda.            |
| **Spot Instances**           | Use spare capacity at up to 90% discount.         | Batch processing, CI/CD workloads. |
| **Dedicated Hosts**          | Physical isolation for compliance.                | Oracle/SQL Server BYOL scenarios.  |

---

## 🧠 Key AWS Cost Concepts

| **Term**                 | **Description**                                       |
| ------------------------ | ----------------------------------------------------- |
| **RDS Storage and IOPS** | Cost depends on allocated size and throughput.        |
| **Data Transfer**        | Inbound is free; outbound to the internet is charged. |
| **EBS Volumes**          | Charged per GB provisioned per month.                 |
| **Lambda**               | Charged per request and execution time.               |
| **S3**                   | Charged for storage class and data retrieval.         |

> 💬 Always align **costs to usage patterns** — don’t over-provision what’s idle.

---

## 🧪 Hands-On Lab 1: Analyze Your Cloud Spending with AWS Cost Explorer

### 🧰 **Objective**

Visualize and analyze your AWS costs by service, usage type, and tags.

---

### **Step 1: Enable Cost Explorer**

1. Go to **Billing → Cost Explorer → Enable**.
2. Select **View by: Service** → Group by **Linked Account / Region**.
3. Time Range: Last 3 months.
4. Export data to CSV for trend analysis.

---

### **Step 2: Identify Top Services**

Look for high-cost services (e.g., EC2, S3, Data Transfer).
Click each service to drill down into:

* Instance Type (e.g., m5.large vs. t3.medium)
* Region (e.g., Sydney vs. Singapore)
* Usage Type (e.g., ComputeHours, StorageGB)

✅ Now you can identify optimization opportunities by service and region.

---

### **CLI Example:**

```bash
aws ce get-cost-and-usage \
  --time-period Start=2025-10-01,End=2025-10-28 \
  --granularity MONTHLY \
  --metrics "AmortizedCost" \
  --group-by Type=DIMENSION,Key=SERVICE
```

---

## 🧩 AWS Budgets and Alerts

Create **spending thresholds** to receive alerts when costs exceed expected levels.

### **Step 1: Create Budget**

1. Go to **Billing → Budgets → Create Budget**.
2. Budget Type: **Cost Budget**.
3. Period: **Monthly**, Amount: `$200`.
4. Alert threshold: 80%.
5. Notification: SNS or Email.
6. Save.

✅ You’ll now be alerted when monthly spending approaches your limit.

---

### **CLI Example:**

```bash
aws budgets create-budget \
  --account-id 123456789012 \
  --budget-name "MonthlyEC2Budget" \
  --budget-type COST \
  --time-unit MONTHLY \
  --budget-limit Amount=200,Unit=USD
```

---

## 🧩 Cost Anomaly Detection

AI-powered service that learns your spending patterns and detects anomalies automatically.

**Steps:**

1. Go to **AWS Cost Anomaly Detection**.
2. Choose “Service” as the **monitoring dimension**.
3. Notification via SNS → “FinanceAlertsTopic”.

✅ Receive alerts for unusual spending (e.g., EC2 costs spike due to misconfiguration).

---

## 🧩 Cost Allocation Tags

Tagging resources is the foundation of cost governance.

| **Tag Key**   | **Purpose**                              |
| ------------- | ---------------------------------------- |
| `Environment` | Identify environments (Dev, Test, Prod). |
| `Project`     | Track project-level spending.            |
| `Owner`       | Assign accountability.                   |
| `CostCenter`  | Align with finance reporting.            |

**Steps:**

1. Go to **Billing → Cost Allocation Tags → Activate Tags**.
2. AWS updates cost reports with tag-based breakdowns.

---

## 🧪 Hands-On Lab 2: Enable and Track Costs by Tags

### 🧰 **Objective**

Group costs by department using tags.

1. Add tags to EC2 and S3 resources:

   ```bash
   aws ec2 create-tags --resources i-0abc1234 --tags Key=Project,Value=CRM_Migration
   ```
2. Enable **Cost Allocation Tags** in the Billing console.
3. In **Cost Explorer**, group by → “Project”.

✅ Now, reports show costs per project — crucial for FinOps accountability.

---

## 🧩 Rightsizing and Compute Optimization

Rightsizing identifies **underutilized instances** and recommends adjustments.

### **Tools:**

* **AWS Compute Optimizer**
* **Trusted Advisor**

**Example CLI:**

```bash
aws compute-optimizer get-ec2-instance-recommendations
```

You’ll see:

* Recommended instance type
* Projected savings percentage
* CPU and memory utilization graphs

✅ Move from **m5.large** → **t3.medium** if utilization < 20%.

---

## 🧩 Storage Cost Optimization

| **Storage Tier**            | **Use Case**                  | **Relative Cost** |
| --------------------------- | ----------------------------- | ----------------- |
| **S3 Standard**             | Active data                   | 💲💲              |
| **S3 Intelligent-Tiering**  | Unpredictable access patterns | 💲                |
| **S3 Glacier**              | Long-term archives            | 💲                |
| **S3 Glacier Deep Archive** | Rarely accessed archives      | 💲 (lowest)       |

**CLI Example: Transition Objects to Glacier**

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket mybucket \
  --lifecycle-configuration file://lifecycle.json
```

**Example `lifecycle.json`:**

```json
{
  "Rules": [{
    "ID": "ArchiveOldFiles",
    "Filter": {"Prefix": ""},
    "Status": "Enabled",
    "Transitions": [{"Days": 30, "StorageClass": "GLACIER"}]
  }]
}
```

✅ Automatically moves files to Glacier after 30 days, reducing cost significantly.

---

## 🧩 Compute Savings Plans

Savings Plans are flexible pricing models that apply discounts across EC2, Fargate, and Lambda.

| **Type**                      | **Scope**                | **Flexibility** | **Discount** |
| ----------------------------- | ------------------------ | --------------- | ------------ |
| **Compute Savings Plan**      | All compute types        | Highest         | Up to 66%    |
| **EC2 Instance Savings Plan** | Specific instance family | Medium          | Up to 72%    |

**CLI Example:**

```bash
aws savingsplans describe-savings-plans
```

✅ Use Savings Plans for predictable workloads, Spot for variable workloads.

---

## 🧩 Spot Instances for Batch Workloads

* **Best for:** Non-critical, fault-tolerant workloads.
* **Pricing:** Up to 90% cheaper than On-Demand.
* **Lifecycle:** Can be interrupted (2-minute warning).

**Example:**

```bash
aws ec2 run-instances \
  --instance-type t3.medium \
  --image-id ami-xxxx \
  --instance-market-options MarketType=spot
```

✅ Combine Spot + Auto Scaling for cost-effective compute pools.

---

## 🧠 Governance and Multi-Account Cost Control

| **Mechanism**                       | **Purpose**                               |
| ----------------------------------- | ----------------------------------------- |
| **AWS Organizations**               | Consolidated billing and cost sharing.    |
| **Service Control Policies (SCPs)** | Enforce spending or region restrictions.  |
| **Resource Tagging Policies**       | Standardize cost tagging across accounts. |
| **Budgets + Alerts per OU**         | Enable FinOps visibility per department.  |

**Example SCP (Block EC2 in Non-Approved Regions):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "ec2:*",
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {"aws:RequestedRegion": "ap-southeast-2"}
    }
  }]
}
```

✅ Prevents cost leakage due to resources created in non-approved regions.

---

## 🧾 Key Takeaways

* **FinOps** aligns cost, usage, and business value across teams.
* Use **Cost Explorer**, **Budgets**, and **Anomaly Detection** for cost visibility.
* Apply **tagging and governance** to map spend by project or owner.
* Implement **Savings Plans** and **Spot Instances** for compute optimization.
* Transition data to **S3 lifecycle tiers** to reduce storage costs.
* Manage costs across accounts via **Organizations and SCPs**.

---

> 🪄 **Next Module Preview:**
> In **Module 17**, we’ll explore **High Availability and Fault Tolerance Architecture in AWS** — learning to design self-healing, multi-AZ, and multi-region systems for mission-critical workloads.

---

Would you like me to continue with **Module 17 – High Availability and Fault Tolerance Architecture in AWS** next?


Excellent ✅ — let’s move forward with **Module 17: High Availability and Fault Tolerance Architecture in AWS**, re-organized from your *AWS Solution Architect Professional (Edureka)* notes.
It’s designed for **YouTube-style narration**, **hands-on labs**, and **reference documentation**.

---

# ⚡ Module 17 – High Availability and Fault Tolerance Architecture in AWS

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Differentiate **high availability (HA)** from **fault tolerance (FT)**.
* Design **multi-AZ and multi-region architectures**.
* Implement **load balancing, auto-scaling, and health checks**.
* Build **stateless tiers** for horizontal scalability.
* Apply **resiliency patterns** for compute, database, and storage.

---

## ☁️ Concepts: HA vs FT vs DR

| Concept               | Definition                                                              | Example                              |
| --------------------- | ----------------------------------------------------------------------- | ------------------------------------ |
| **High Availability** | Minimize downtime using redundant components.                           | Multi-AZ RDS deployment              |
| **Fault Tolerance**   | System continues operation without interruption when a component fails. | Multi-region active/active setup     |
| **Disaster Recovery** | Recover services and data after major outage.                           | Cross-region failover using Route 53 |

> 💡 **Analogy:** HA is like having two engines on an airplane; FT is having two airplanes in the air simultaneously.

---

## 🧩 AWS Building Blocks for Resilience

| Layer              | Service                                             | Role                             |
| ------------------ | --------------------------------------------------- | -------------------------------- |
| **Compute**        | Auto Scaling Groups, EC2 Health Checks              | Replace failed instances         |
| **Load Balancing** | ALB, NLB, GLB                                       | Distribute traffic               |
| **Database**       | RDS Multi-AZ, Aurora Replica, DynamoDB Global Table | Redundant data layer             |
| **Storage**        | S3 Replication, EFS Multi-AZ                        | Durable storage                  |
| **DNS Failover**   | Route 53 Health Checks & Policies                   | Direct traffic to healthy region |
| **Monitoring**     | CloudWatch & EventBridge                            | Detect & recover automatically   |

---

## 🧪 Hands-On Lab 1: Create a Highly Available Web Tier

### 🧰 Objective

Deploy an auto-scaled, load-balanced EC2 web application across multiple AZs.

---

### Step 1 – Create Launch Template

```bash
aws ec2 create-launch-template \
  --launch-template-name WebTemplate \
  --launch-template-data '{
    "ImageId":"ami-0abcd1234",
    "InstanceType":"t3.micro",
    "SecurityGroupIds":["sg-xxxx"],
    "UserData":"IyEvYmluL2Jhc2gKYXB0LWdldCB1cGRhdGUgLXkKc3lzdGVtY3RsIHN0YXJ0IG5naW54"
  }'
```

### Step 2 – Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name WebASG \
  --launch-template LaunchTemplateName=WebTemplate \
  --min-size 2 --max-size 4 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-a,subnet-b"
```

### Step 3 – Attach Application Load Balancer

1. Create ALB with **public subnets**.
2. Add target group with **health check** `/index.html`.
3. Register the ASG instances automatically.

✅ If one AZ or instance fails, traffic shifts automatically to healthy instances.

---

## 🧩 Multi-AZ vs Multi-Region Designs

| Strategy                        | Description                                               | Use Case                   |
| ------------------------------- | --------------------------------------------------------- | -------------------------- |
| **Multi-AZ**                    | Replication within one region; automatic failover.        | RDS Multi-AZ, EFS Multi-AZ |
| **Multi-Region Active-Passive** | Standby region pre-provisioned.                           | Low-latency DR             |
| **Multi-Region Active-Active**  | Both regions serve traffic; global failover via Route 53. | Mission-critical apps      |

> 🔧 Design Tip: Use **Route 53 Latency-Based Routing** for geo-distributed traffic.

---

## 🧪 Hands-On Lab 2: Set Up RDS Multi-AZ Database

### 🧰 Objective

Deploy an RDS MySQL DB Instance with automatic failover.

1. Go to **RDS → Create Database**.
2. Choose **MySQL** → Production Template.
3. Enable **Multi-AZ Deployment**.
4. Storage Auto Scaling: ✅ Enabled.
5. Launch and note the primary endpoint.

✅ If the primary AZ fails, RDS promotes the standby automatically with no data loss.

---

## 🧠 Stateless Application Design

| Best Practice                                          | Description                                         |
| ------------------------------------------------------ | --------------------------------------------------- |
| **Store session state externally**                     | Use ElastiCache or DynamoDB for session management. |
| **Immutable infrastructure**                           | Deploy new instances instead of modifying in place. |
| **Use Load Balancers for stickiness only if required** | Prefer truly stateless tiers for HA.                |

---

## 🧩 Storage and Data Resilience

### S3

* 99.999999999% durability (11 nines).
* Use **Cross-Region Replication (CRR)** for DR.

### EFS

* Multi-AZ NFS storage with regional redundancy.

### EBS

* Snapshots for backup; consider **EBS Multi-Attach** for clustered apps.

**CLI:**

```bash
aws s3api put-bucket-replication --bucket mydata --replication-configuration file://replication.json
```

---

## 🧩 Route 53 Health Checks and Failover

### **Scenario:**

Web App in Sydney Region (AP-Southeast-2)
DR App in Singapore (AP-Southeast-1)

1. Create two ALB endpoints:

   * `web-syd.example.com`
   * `web-sgp.example.com`
2. Create **Health Checks** for each ALB.
3. Configure **Failover Routing Policy** (Sydney = Primary, Singapore = Secondary).

✅ If Sydney fails, Route 53 automatically redirects users to Singapore.

---

## 🧠 Resiliency Patterns and Services

| Pattern                    | AWS Service                         | Outcome                                   |
| -------------------------- | ----------------------------------- | ----------------------------------------- |
| **Auto Healing**           | EC2 Auto Recovery, ASG Health Check | Replace failed instances                  |
| **Queue-Based Decoupling** | SQS / SNS / EventBridge             | Prevent cascading failures                |
| **Circuit Breaker**        | Lambda + Step Functions             | Stop retry loops on failure               |
| **Bulkhead Isolation**     | Separate tiers per VPC/Subnet       | Contain impact                            |
| **Graceful Degradation**   | CloudFront + Static Fallback        | Serve reduced functionality during outage |

---

## 🧪 Hands-On Lab 3: Simulate and Recover from Failure

### 🧰 Objective

Test self-healing capabilities using Auto Scaling Group and CloudWatch Alarm.

1. Create ASG with desired capacity = 2.
2. In **CloudWatch**, create alarm on `StatusCheckFailed_Instance > 0`.
3. Set alarm action → **EC2 Recover Instance**.
4. Manually stop one instance to simulate failure.

✅ ASG detects and replaces the instance automatically within minutes.

---

## 🧩 Multi-Region Data Replication Patterns

| Data Type          | Recommended Service         | Notes                      |
| ------------------ | --------------------------- | -------------------------- |
| **Relational**     | Aurora Global Database      | Sub-second replication     |
| **NoSQL**          | DynamoDB Global Tables      | Active/active multi-region |
| **Object Storage** | S3 Cross-Region Replication | Versioning required        |
| **Cache**          | Global Datastore for Redis  | Multi-region read replicas |

---

## 🧩 Monitoring & Resilience Testing

| Tool                                    | Purpose                                    |
| --------------------------------------- | ------------------------------------------ |
| **CloudWatch Alarms**                   | Detect service health changes              |
| **AWS Fault Injection Simulator (FIS)** | Perform chaos engineering experiments      |
| **AWS Resilience Hub**                  | Evaluate and score application resilience  |
| **CloudTrail + Config**                 | Audit infrastructure changes post-incident |

> 🔬 Practice “game days” — simulate failures to validate HA and DR plans.

---

## 🧾 Key Takeaways

* **HA ≠ FT** — HA minimizes downtime; FT eliminates it.
* Design with **redundancy, statelessness, and auto-recovery**.
* **Multi-AZ deployments** handle local failures; **multi-region** protects against regional outages.
* Implement **Route 53 failover** and **cross-region replication** for global resilience.
* Regularly **test failover and recovery** using automation and chaos engineering.

---

> 🪄 **Next Module Preview:**
> In **Module 18**, we’ll cover **AWS Security, Compliance and Governance Frameworks** — learning how to build secure, auditable architectures aligned with CIS, NIST and ISO standards.

---
