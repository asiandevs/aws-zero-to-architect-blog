---
title: "Module 7 – Amazon Route 53 (DNS and Traffic Routing)"
date: 2025-11-04
categories: aws networking dns
tags: aws route53 dns failover latency weighted
image: /assets/images/route53.jpg
description: "DNS routing, domain registration, hosted zones, and routing policies with Route 53."
---

---

# Module 7 – Amazon Route 53 (DNS and Traffic Routing) 🌍

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand how **Amazon Route 53** provides DNS and traffic routing on AWS.
* Configure **domain registration, hosted zones**, and **DNS record sets**.
* Implement **routing policies** (simple, weighted, latency-based, failover).
* Integrate Route 53 with **Load Balancers, CloudFront, and S3**.
* Test end-to-end DNS resolution with **public and private hosted zones**.

---

## ☁️ What is Amazon Route 53?    <img width="50" height="60" alt="image" src="https://github.com/user-attachments/assets/7174fa2c-15d4-4e28-86ab-c4e15a61b8dc" />


**Amazon Route 53** is a **highly available and scalable Domain Name System (DNS) web service** designed to route end users to internet applications.
It performs three main functions:

| **Function**                   | **Description**                                          |
| ------------------------------ | -------------------------------------------------------- |
| **Domain Registration**        | Register new domain names directly via AWS.              |
| **DNS Resolution**             | Translate human-readable domain names into IP addresses. |
| **Health Checking & Failover** | Monitor endpoint health and route traffic accordingly.   |

> 💡 **Analogy:**
> Think of Route 53 as the “phonebook” of the internet — it translates names like `myapp.com` into IP addresses (like 54.23.118.12) so browsers can find your servers.

---

## 🧱 Key Features of Route 53

| **Feature**              | **Purpose**                                                    |
| ------------------------ | -------------------------------------------------------------- |
| **Scalable DNS**         | Handles millions of queries per second globally.               |
| **Private Hosted Zones** | DNS resolution for private VPCs.                               |
| **Health Checks**        | Monitors the health of endpoints (e.g., web servers).          |
| **DNS Failover**         | Redirects traffic to healthy resources.                        |
| **Traffic Routing**      | Directs requests based on latency, geography, or weight.       |
| **Integration**          | Works seamlessly with ELB, CloudFront, and S3 static websites. |

---

## 🌐 Route 53 Architecture Overview

```
       ┌───────────────────────────────────────┐
       │             Route 53                  │
       └───────────────┬───────────────────────┘
                       │
     ┌─────────────────┼───────────────────┐
     │                 │                   │
  DNS Records       Health Checks     Traffic Routing
     │                 │                   │
     ▼                 ▼                   ▼
  ELB / EC2         Web Servers        Global Users
```

---

## 🧩 Domain Registration

Route 53 allows you to register domains directly or use an existing registrar.

### **Steps:**

1. Go to **Route 53 → Domains → Register Domain**.
2. Search for your preferred domain (e.g., `mycloudlabdemo.com`).
3. Add to cart and proceed to checkout.
4. Configure contact details and complete registration.

> 💡 *Tip:* You can also transfer existing domains into Route 53 for centralized DNS management.

---

## 📁 Hosted Zones

| **Type**                | **Description**                                 | **Use Case**                |
| ----------------------- | ----------------------------------------------- | --------------------------- |
| **Public Hosted Zone**  | Routes internet traffic to AWS resources.       | Websites, APIs, public apps |
| **Private Hosted Zone** | Used within a VPC for internal name resolution. | Intranet, private apps      |

---

## 🧪 Hands-On Lab 1: Create a Hosted Zone and DNS Record

### 🧰 **Objective**

Map a domain name to an EC2 instance using Route 53.

---

### **Step 1: Create a Hosted Zone**

1. Navigate to **Route 53 → Hosted Zones → Create Hosted Zone**.
2. Domain name: `mycloudlabdemo.com`.
3. Type: **Public Hosted Zone**.
4. Click **Create**.

---

### **Step 2: Add DNS Record Sets**

1. Click the hosted zone you just created.
2. Click **Create Record** → Add an **A Record**:

   * Record name: `www`
   * Value: Public IP of your EC2 instance
   * Routing policy: **Simple routing**
3. Save changes.

**CLI Example:**

```bash
aws route53 change-resource-record-sets \
--hosted-zone-id ZXXXXXXXXXXXX \
--change-batch '{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {
      "Name": "www.mycloudlabdemo.com.",
      "Type": "A",
      "TTL": 300,
      "ResourceRecords": [{"Value": "54.23.118.12"}]
    }
  }]
}'
```

---

### **Step 3: Update Name Servers**

If your domain is registered outside AWS:

1. Copy the **Name Servers (NS)** from your hosted zone.
2. Update them in your domain registrar’s DNS settings.

After propagation (may take up to 30 minutes):

```bash
nslookup www.mycloudlabdemo.com
```

✅ You should see your EC2 instance IP.

---

## 🧠 Route 53 Record Types

| **Record Type**      | **Purpose**             | **Example**                                        |
| -------------------- | ----------------------- | -------------------------------------------------- |
| **A**                | IPv4 address mapping    | `myapp.com → 13.211.45.10`                         |
| **AAAA**             | IPv6 address mapping    | `myapp.com → 2406:da18::1`                         |
| **CNAME**            | Alias to another domain | `blog.mycloudlabdemo.com → www.mycloudlabdemo.com` |
| **MX**               | Mail routing            | Used for email servers                             |
| **TXT**              | Custom text data        | Used for SPF/DKIM validation                       |
| **Alias (AWS-only)** | Points to AWS resources | ELB, S3, CloudFront                                |

---

## ⚙️ Routing Policies in Route 53

| **Policy**             | **Purpose**                      | **Example Use Case**                                   |
| ---------------------- | -------------------------------- | ------------------------------------------------------ |
| **Simple**             | Single resource routing          | Direct traffic to one IP or ELB.                       |
| **Weighted**           | Distribute traffic by percentage | 70% traffic to ALB, 30% to new version.                |
| **Latency-Based**      | Route to lowest latency region   | Send EU users to `eu-west-1`, US users to `us-east-1`. |
| **Failover**           | Redirect on failure              | Primary → Secondary region failover.                   |
| **Geolocation**        | Route based on user’s country    | Users in Japan → Tokyo servers.                        |
| **Multi-Value Answer** | Return multiple healthy records  | DNS-based load balancing.                              |

---

## 🧪 Hands-On Lab 2: Weighted Routing Policy

### **Objective:**

Split traffic between two web servers to simulate canary deployment.

1. Create two EC2 instances: **Server-A** and **Server-B**.
2. Go to **Hosted Zone → Create Record → Weighted Routing**.
3. Create two records with:

   * Record name: `test.mycloudlabdemo.com`
   * Record A1: `Server-A IP` → Weight: 70
   * Record A2: `Server-B IP` → Weight: 30
4. Save records.
5. Refresh browser multiple times — you’ll see responses alternate approximately by ratio.

---

## 🧩 Hands-On Lab 3: Latency-Based Routing

### **Objective:**

Route users to the region with the best latency.

1. Create two EC2 instances in different regions (e.g., Sydney and Oregon).
2. In **Route 53 → Create Record**, choose:

   * Record Type: A
   * Routing Policy: **Latency-based**
   * Region: ap-southeast-2 (Sydney)
   * Value: Sydney EC2 IP
3. Add another record for `us-west-2 (Oregon)` region.

✅ Users from Australia will resolve to Sydney; U.S. users → Oregon.

---

## 🧩 Hands-On Lab 4: Failover Routing with Health Checks

### **Objective:**

Automatically redirect traffic to a backup server if the primary fails.

1. Launch two EC2 instances: **Primary (Sydney)** and **Backup (Oregon)**.
2. In Route 53:

   * **Record 1 (Primary)**: `A` record → `Active` failover.
   * **Record 2 (Backup)**: `A` record → `Passive` failover.
3. Create a **Health Check** for the primary:

   * Endpoint: `http://<primary-ip>/index.html`
   * Failure threshold: 3
4. If the primary instance becomes unhealthy, Route 53 automatically redirects to the backup.

---

## 🧠 DNS with AWS Integration

| **Service**             | **Integration Example**                    |
| ----------------------- | ------------------------------------------ |
| **ELB**                 | Route traffic to Load Balancer DNS name.   |
| **S3 Static Website**   | Map domain directly to S3 bucket endpoint. |
| **CloudFront**          | Connect global CDN to Route 53 alias.      |
| **Private Hosted Zone** | Resolve names for internal EC2 instances.  |

**Example Alias Record to ALB:**

```bash
aws route53 change-resource-record-sets \
--hosted-zone-id ZXXXXXXXXXXXX \
--change-batch '{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {
      "Name": "app.mycloudlabdemo.com.",
      "Type": "A",
      "AliasTarget": {
        "HostedZoneId": "Z1H1FL5HABSF5",
        "DNSName": "myappalb-123456.ap-southeast-2.elb.amazonaws.com",
        "EvaluateTargetHealth": false
      }
    }
  }]
}'
```

---

## 🧩 Optional: Private Hosted Zone (Internal DNS)

Used for internal name resolution within a VPC.

**Steps:**

1. Create a new **Private Hosted Zone** (e.g., `corp.internal`).
2. Associate it with your **Custom VPC**.
3. Create A Records mapping internal EC2 IPs to hostnames.
4. Verify with:

   ```bash
   dig webserver1.corp.internal
   ```

✅ Internal DNS name resolution now works within the VPC — no internet required.

---

## 🧾 Key Takeaways

* **Route 53** combines **domain registration, DNS resolution**, and **health-based routing**.
* Supports **public and private hosted zones**.
* Offers multiple **routing policies** (simple, weighted, latency, failover).
* Integrates with **ALB, CloudFront, S3, and Auto Scaling**.
* Provides **high availability** and **low-latency routing** globally.

---

> 🪄 **Next Module Preview:**
> In **Module 8**, we’ll explore **AWS CloudFormation (Infrastructure as Code)** — where you’ll learn how to automate VPC, EC2, and ELB deployments using YAML/JSON templates and stack management.

---
