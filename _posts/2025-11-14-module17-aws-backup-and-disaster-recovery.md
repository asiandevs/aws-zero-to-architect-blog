---
title: "Module 17 – AWS Backup and Disaster Recovery"
date: 2025-11-14
categories: ["aws", "backup", "dr", "business-continuity"]
tags: ["aws", "backup", "drs", "cross-region", "recoverypoint"]
image: /assets/images/backup.jpg
description: "Designing resilient backup and disaster recovery strategies using AWS Backup, cross-region replication, and DR planning."
---

# Module 17 – High Availability and Fault Tolerance Architecture in AWS

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

> 🔬 Practice "game days" — simulate failures to validate HA and DR plans.

---

## 🧾 Key Takeaways

* **HA ≠ FT** — HA minimizes downtime; FT eliminates it.
* Design with **redundancy, statelessness, and auto-recovery**.
* **Multi-AZ deployments** handle local failures; **multi-region** protects against regional outages.
* Implement **Route 53 failover** and **cross-region replication** for global resilience.
* Regularly **test failover and recovery** using automation and chaos engineering.

---

> 🪄 **Next Module Preview:**
> In **Module 18**, we'll cover **AWS Security, Compliance and Governance Frameworks** — learning how to build secure, auditable architectures aligned with CIS, NIST and ISO standards.


