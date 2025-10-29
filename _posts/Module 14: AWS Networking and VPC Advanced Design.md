---
title: "Module 14 – AWS Security and Encryption Services"
date: 2025-11-11
categories: aws security encryption compliance
tags: aws kms ssm secrets-manager guardduty inspector
image: /assets/images/security.jpg
description: "Implementing encryption, key management, and continuous threat detection using AWS KMS, Secrets Manager, and GuardDuty."
---

---

# 🌐 Module 14 – AWS Networking and VPC Advanced Design

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Design and configure **custom Virtual Private Clouds (VPCs)**.
* Implement **public and private subnets**, **NAT gateways**, and **route tables**.
* Configure **VPC peering** and **Transit Gateway** for multi-VPC communication.
* Establish **hybrid connectivity** using **VPN** and **Direct Connect**.
* Apply **network security** using **NACLs**, **Security Groups**, and **Flow Logs**.

---

## ☁️ What Is Amazon VPC?

**Amazon Virtual Private Cloud (VPC)** enables you to launch AWS resources in a logically isolated virtual network that you define.

> 💡 **Analogy:**
> Think of a VPC as your **private neighborhood** in the AWS cloud — you define the streets (subnets), houses (EC2 instances), and gates (security rules).

---

## 🧩 VPC Core Components

| **Component**                     | **Description**                                    |
| --------------------------------- | -------------------------------------------------- |
| **VPC**                           | The isolated virtual network.                      |
| **Subnet**                        | A segment of the VPC (public or private).          |
| **Route Table**                   | Defines traffic direction (internal or internet).  |
| **Internet Gateway (IGW)**        | Enables internet access for public subnets.        |
| **NAT Gateway**                   | Allows private instances outbound internet access. |
| **VPC Peering / Transit Gateway** | Connects multiple VPCs.                            |
| **Security Groups (SGs)**         | Instance-level firewalls.                          |
| **Network ACLs (NACLs)**          | Subnet-level firewalls.                            |

---

## 🧠 Default vs. Custom VPC

| **Feature**     | **Default VPC**               | **Custom VPC**                   |
| --------------- | ----------------------------- | -------------------------------- |
| **Subnets**     | One per AZ, public by default | Defined manually                 |
| **Route Table** | Preconfigured for internet    | Customizable                     |
| **CIDR Range**  | 172.31.0.0/16                 | User-defined (e.g., 10.0.0.0/16) |
| **Control**     | Limited                       | Full network control             |

> 🧩 Always use **custom VPCs** for production workloads to enforce better segmentation and governance.

---

## 🧪 Hands-On Lab 1: Create a Custom VPC with Public and Private Subnets

### 🧰 **Objective**

Design a 2-tier architecture:

* **Public Subnet** for web servers
* **Private Subnet** for databases

---

### **Step 1: Create a VPC**

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

Or via **Console**:

* Name: `MyVPC`
* IPv4 CIDR: `10.0.0.0/16`

---

### **Step 2: Create Subnets**

| Subnet Name    | AZ              | CIDR        | Type    |
| -------------- | --------------- | ----------- | ------- |
| PublicSubnet1  | ap-southeast-2a | 10.0.1.0/24 | Public  |
| PrivateSubnet1 | ap-southeast-2b | 10.0.2.0/24 | Private |

```bash
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.1.0/24 --availability-zone ap-southeast-2a
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.2.0/24 --availability-zone ap-southeast-2b
```

---

### **Step 3: Create and Attach Internet Gateway**

```bash
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxxx --vpc-id vpc-xxxx
```

---

### **Step 4: Create Route Tables**

* **Public Route Table**

  * Destination: `0.0.0.0/0`
  * Target: Internet Gateway

* **Private Route Table**

  * Default local route only.

Associate:

```bash
aws ec2 associate-route-table --route-table-id rtb-public --subnet-id subnet-public
aws ec2 associate-route-table --route-table-id rtb-private --subnet-id subnet-private
```

✅ Your web tier (public subnet) now has internet access, while the private subnet remains internal.

---

### **Step 5: Launch EC2 Instances**

* **Web Server** → in **Public Subnet**

  * Assign public IP
  * Security Group: Allow HTTP (80), SSH (22)

* **Database Server** → in **Private Subnet**

  * No public IP
  * Security Group: Allow MySQL (3306) from Web Server SG only

---

## ⚙️ Network Address Translation (NAT Gateway)

Private instances need internet access for updates, but not inbound traffic.
**NAT Gateway** allows outbound-only access.

### **Steps:**

1. Create **Elastic IP**.
2. Create **NAT Gateway** in the **Public Subnet** using that EIP.
3. Update **Private Route Table**:

   * Destination: `0.0.0.0/0`
   * Target: NAT Gateway

✅ Now private EC2s can reach the internet for updates securely.

---

## 🧪 Hands-On Lab 2: Enable VPC Peering

### 🧰 **Objective**

Connect two VPCs privately across accounts or regions.

---

### **Step 1: Create Peering Connection**

```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-aaaa1111 \
  --peer-vpc-id vpc-bbbb2222
```

### **Step 2: Accept the Peering Request**

```bash
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-xxxx
```

### **Step 3: Update Route Tables**

Add routes in both VPCs:

```
Destination: 10.1.0.0/16 → Target: pcx-xxxx
Destination: 10.0.0.0/16 → Target: pcx-xxxx
```

✅ Instances in both VPCs can now communicate privately via internal IPs.

---

## 🧠 VPC Peering Limitations

* **No transitive peering** (A↔B, B↔C ≠ A↔C)
* CIDR ranges must **not overlap**.
* Security groups must be configured to allow inter-VPC traffic.

---

## ⚙️ Transit Gateway (TGW)

**AWS Transit Gateway** simplifies network design by connecting multiple VPCs and on-prem environments through a single hub.

### **Architecture Overview**

```
          ┌────────────────────┐
          │   On-Prem Network  │
          └──────────┬─────────┘
                     │
               ┌─────▼─────┐
               │  VPN / DX  │
               └─────┬─────┘
           ┌─────────▼─────────┐
           │  Transit Gateway  │
           └──────┬───────────┘
        ┌──────────┼──────────┐
        │          │          │
      VPC-A      VPC-B      VPC-C
```

> 💡 **Analogy:**
> Transit Gateway acts like a **central airport hub** — all networks connect through one point, simplifying routes and management.

---

### **CLI Example: Create Transit Gateway**

```bash
aws ec2 create-transit-gateway --description "MyTGW"
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-xxxx \
  --vpc-id vpc-aaaa1111 \
  --subnet-ids subnet-111 subnet-222
```

✅ Simplifies multi-VPC and hybrid routing management.

---

## 🧩 Hybrid Connectivity – VPN and Direct Connect

### **1️⃣ Site-to-Site VPN**

Secure IPsec tunnel between on-prem network and AWS VPC.

**Steps:**

1. Create **Customer Gateway (CGW)** with on-prem IP.
2. Create **Virtual Private Gateway (VGW)** and attach to VPC.
3. Establish **VPN Connection** between CGW and VGW.
4. Configure on-prem router with AWS tunnel details.

**CLI Example:**

```bash
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-xxxx \
  --vpn-gateway-id vgw-xxxx
```

✅ Enables hybrid access securely over the internet.

---

### **2️⃣ AWS Direct Connect**

* Dedicated, private fiber connection between your data center and AWS.
* Provides **lower latency**, **higher bandwidth**, and **consistent performance**.

**Typical Use Case:**

> Banks, healthcare, and government workloads requiring private, compliant data transfer.

---

## 🧠 Security Layers in VPC

| **Layer**           | **Control Mechanism** | **Example**                       |
| ------------------- | --------------------- | --------------------------------- |
| **Instance Level**  | Security Groups       | Allow inbound HTTP/SSH            |
| **Subnet Level**    | Network ACLs          | Deny all inbound except port 80   |
| **Network Flow**    | VPC Flow Logs         | Capture accepted/rejected traffic |
| **Data Encryption** | KMS, SSL/TLS          | Secure in-transit communication   |

---

### **VPC Flow Logs Example**

Capture network traffic for audit or troubleshooting:

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxx \
  --traffic-type ALL \
  --log-group-name VPCFlowLogs
```

View logs in **CloudWatch → Log Groups → VPCFlowLogs**.

---

## 🧠 Best Practices for VPC Design

* Use **/16 CIDR** for VPC and **/24** for subnets (plan for growth).
* Always enable **Flow Logs** and **VPC DNS resolution**.
* Separate workloads into **tiers (web, app, db)** with private subnets.
* Use **NAT Gateways** for private subnet internet access.
* Enforce **least privilege security groups** and **stateless NACLs**.
* For large networks → Use **Transit Gateway** over multiple peerings.
* For hybrid setups → Combine **Direct Connect + VPN failover**.

---

## 🧾 Key Takeaways

* **VPC** is the foundation of all AWS networking — fully customizable and isolated.
* **Subnets**, **Route Tables**, and **Gateways** control internal and external traffic.
* **NAT** enables secure outbound-only access from private instances.
* **VPC Peering** connects two VPCs; **Transit Gateway** scales across many.
* **VPN** and **Direct Connect** enable secure hybrid architectures.
* Always apply **multi-layered security** — SGs, NACLs, Flow Logs, and encryption.

---

> 🪄 **Next Module Preview:**
> In **Module 15**, we’ll explore **AWS Migration, Backup, and Disaster Recovery** — understanding data migration tools, backup strategies, and cross-region DR architectures for high availability.

---
