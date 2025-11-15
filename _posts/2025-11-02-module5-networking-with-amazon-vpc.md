# ──────────────────────────────────────────────────────────────
# MODULE 5 (YOUR TEST)
# File: _posts/2025-11-02-module5-networking-with-amazon-vpc.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 5 – Networking with Amazon VPC"
date: 2025-11-02
categories: aws networking vpc security
tags: aws vpc subnet natgateway igw peering
image: /assets/images/vpc.jpg
description: "Designing custom VPCs with subnets, route tables, gateways, security groups, and endpoints."

prev:
  title: "Module 4: Amazon S3: Simple Storage Service"
  url: "/_posts/2025-10-30-module4-amazon-s3-simple-storage-service.md"
next:
  title: "Module 6: Elastic Load Balancing & Auto Scaling"
  url: "/_posts/2025-11-03-module6-elastic-load-balancing-and-auto-scaling.md"
---

# 🌐 Module 5 – Networking with Amazon VPC (Virtual Private Cloud)

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Explain what an Amazon VPC is and its role in AWS networking.
* Design a custom VPC with **subnets, route tables, gateways, and security layers**.
* Configure **public/private subnets**, **NAT gateways**, and **bastion hosts**.
* Understand **VPC peering, endpoints, and network ACLs (NACLs)**.
* Launch EC2 instances within your custom VPC environment.

---

## ☁️ What is Amazon VPC?    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/8dbdaaf8-02fc-4358-a7ac-fd7814751580" />


**Amazon VPC (Virtual Private Cloud)** lets you **provision a logically isolated section** of the AWS Cloud where you can define your own network configuration — IP address ranges, subnets, route tables, and security rules.

> 💡 **Analogy:**
> Imagine AWS as a massive city.
> A **VPC** is your private gated neighborhood where you design the streets (subnets), install security checkpoints (security groups/NACLs), and connect to the main highway (internet gateway).

---

## 🧱 Core Components of VPC

| **Component**              | **Description**                                       |
| -------------------------- | ----------------------------------------------------- |
| **VPC**                    | A virtual network dedicated to your AWS account.      |
| **Subnets**                | Subdivisions of a VPC (public or private).            |
| **Route Table**            | Defines where network traffic is directed.            |
| **Internet Gateway (IGW)** | Enables internet access for public subnets.           |
| **NAT Gateway**            | Allows private instances outbound internet access.    |
| **Security Group**         | Instance-level firewall (stateful).                   |
| **Network ACL (NACL)**     | Subnet-level firewall (stateless).                    |
| **VPC Peering**            | Connects two VPCs privately.                          |
| **Endpoints**              | Private connections to AWS services without internet. |

---

## 🗺️ Default vs. Custom VPC

| **Feature**     | **Default VPC**  | **Custom VPC**             |
| --------------- | ---------------- | -------------------------- |
| Auto-created    | Yes              | No                         |
| Internet-ready  | Yes              | No (you must add IGW)      |
| Default subnets | One per AZ       | User-defined               |
| Custom routing  | Limited          | Fully configurable         |
| Use case        | Quick deployment | Production-level isolation |

> Best Practice: Always create a **custom VPC** for enterprise or production workloads.

---

## 🧮 CIDR Blocks (IP Addressing)

CIDR (Classless Inter-Domain Routing) defines the IP range for your VPC.

| **Example**   | **Range**                  | **# of IPs** |
| ------------- | -------------------------- | ------------ |
| 10.0.0.0/16   | 10.0.0.0 – 10.0.255.255    | 65,536       |
| 10.0.1.0/24   | 10.0.1.0 – 10.0.1.255      | 256          |
| 172.16.0.0/20 | 172.16.0.0 – 172.16.15.255 | 4,096        |

> AWS reserves **5 IPs per subnet** for internal use.

---

## 🧪 Hands-On Lab: Build Your Own VPC

### **Objective**

Create a custom VPC with:

* One public and one private subnet
* Internet Gateway for public subnet
* NAT Gateway for private subnet
* Route tables for traffic control

---

### **Step 1: Create a Custom VPC**

1. Go to **VPC Dashboard → Create VPC**.
2. Choose **VPC Only**.
3. Enter:

   * Name: `MyCustomVPC`
   * IPv4 CIDR: `10.0.0.0/16`
4. Leave IPv6 blank (optional).
5. Click **Create VPC**.

**CLI Example:**

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

---

### **Step 2: Create Subnets**

| **Subnet Type** | **CIDR**    | **Availability Zone** |
| --------------- | ----------- | --------------------- |
| Public Subnet   | 10.0.1.0/24 | ap-southeast-2a       |
| Private Subnet  | 10.0.2.0/24 | ap-southeast-2b       |

**Console Steps:**

1. Navigate to **Subnets → Create Subnet**.
2. Select `MyCustomVPC`.
3. Add both subnets above.
4. Mark the first as **Public Subnet** (you’ll enable public IPs soon).

**CLI Example:**

```bash
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.2.0/24
```

---

### **Step 3: Create and Attach Internet Gateway (IGW)**

1. Go to **Internet Gateways → Create IGW**.
2. Name: `MyIGW`.
3. Attach to `MyCustomVPC`.

**CLI Example:**

```bash
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-xxxx --internet-gateway-id igw-xxxx
```

---

### **Step 4: Create Route Tables**

1. Go to **Route Tables → Create Route Table**.

   * Name: `Public-RT`
   * Associate with `MyCustomVPC`
2. Add route:

   * Destination: `0.0.0.0/0`
   * Target: `Internet Gateway`
3. Associate `Public Subnet` to this table.
4. Create another route table for the **Private Subnet** (no direct internet route).

---

### **Step 5: Create NAT Gateway for Private Subnet**

1. Go to **NAT Gateways → Create NAT Gateway**.
2. Choose the **Public Subnet** (requires Elastic IP).
3. Click **Create NAT Gateway**.
4. Update the **Private Route Table**:

   * Destination: `0.0.0.0/0`
   * Target: `NAT Gateway`

✅ Now your private instances can reach the internet for updates (via NAT) but remain inaccessible from outside.

---

## 🔒 Security Groups vs. Network ACLs

| **Feature**          | **Security Group**                   | **NACL (Network ACL)**      |
| -------------------- | ------------------------------------ | --------------------------- |
| **Scope**            | Instance-level                       | Subnet-level                |
| **Statefulness**     | Stateful                             | Stateless                   |
| **Default Behavior** | Deny all inbound, allow all outbound | Allow all inbound/outbound  |
| **Use Case**         | Control traffic to EC2 instances     | Control subnet-level access |

---

### **Example: Security Group for Web Server**

| **Type** | **Protocol** | **Port Range** | **Source** |
| -------- | ------------ | -------------- | ---------- |
| SSH      | TCP          | 22             | My IP      |
| HTTP     | TCP          | 80             | 0.0.0.0/0  |
| HTTPS    | TCP          | 443            | 0.0.0.0/0  |

---

## 🧩 Hands-On: Launch an EC2 in Your VPC

### **Step 1: Launch Instance**

1. Go to **EC2 → Launch Instance**.
2. Choose **AMI:** Amazon Linux 2.
3. Instance type: **t2.micro**.
4. Network: Select `MyCustomVPC`.
5. Subnet: `Public Subnet`.
6. Auto-assign Public IP: **Enable**.
7. Select **Key Pair**, Security Group → Launch.

### **Step 2: Connect and Test**

```bash
ssh -i mykey.pem ec2-user@<public-ip>
ping google.com        # Works (public subnet)
```

### **Step 3: Test from Private Subnet**

Launch another instance in **Private Subnet** (disable public IP).
SSH into it via the public instance (bastion host):

```bash
ssh -i mykey.pem ec2-user@<private-ip> -J ec2-user@<public-ip>
```

✅ Confirm private instance has outbound internet via NAT Gateway.

---

## 🌉 VPC Peering    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/c9aa4b84-5d0c-4651-b42a-dba636f87e78" />


**Definition:** Connects two VPCs using private AWS network paths (no internet).

**Use Cases:**

* Multi-region communication
* Hybrid applications
* Cross-team shared services

### **Hands-On Summary**

1. VPC → Peering Connections → Create Peering.
2. Choose Source and Target VPCs (same or different accounts).
3. Accept the request.
4. Update route tables on both VPCs to route traffic to each other.

---

## 🔗 VPC Endpoints

**Goal:** Access AWS services (like S3, DynamoDB) privately — without using public internet.

| **Type**                             | **Description**                             | **Example**      |
| ------------------------------------ | ------------------------------------------- | ---------------- |
| **Interface Endpoint (PrivateLink)** | Elastic network interface using private IP. | API Gateway, KMS |
| **Gateway Endpoint**                 | Uses route tables to access AWS services.   | S3, DynamoDB     |

**CLI Example (S3 Gateway Endpoint):**

```bash
aws ec2 create-vpc-endpoint --vpc-id vpc-xxxx --service-name com.amazonaws.ap-southeast-2.s3 --route-table-ids rtb-xxxx
```

---

## 🧩 Optional: Bastion Host Setup    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/339fe9d3-23f4-47a8-b260-8eced38501df" />


A **bastion host** provides secure SSH access to instances in private subnets.

**Steps Summary:**

1. Launch a small EC2 (public subnet).
2. Attach IAM role for SSH management.
3. SSH to the bastion → then jump into private subnet EC2s.
4. Limit access by IP or security group rules.

---

## 🧾 Key Takeaways

* A **VPC** provides a customizable, isolated network environment.
* **Subnets** define IP ranges for grouping resources.
* **Internet Gateways** and **NAT Gateways** manage inbound/outbound access.
* **Security Groups** are stateful firewalls; **NACLs** are stateless.
* **VPC Peering** and **Endpoints** connect networks securely.
* Building a custom VPC is foundational for deploying secure, multi-tier architectures.

---

> 🪄 **Next Module Preview:**
> In **Module 6**, we’ll dive into **Elastic Load Balancing (ELB)** — exploring Classic, Application, and Network Load Balancers, with live traffic routing, health checks, and auto-scaling integration.

---
