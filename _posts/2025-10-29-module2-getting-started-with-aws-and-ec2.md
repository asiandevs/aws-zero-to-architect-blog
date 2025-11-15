# ──────────────────────────────────────────────────────────────
# MODULE 2
# File: _posts/2025-10-29-module2-getting-started-with-aws-and-ec2.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 2: Getting Started with AWS & EC2"
date: 2025-10-29
categories: aws ec2 compute
tags: ec2 ami launch-template autoscaling
image: /assets/images/ec2.jpg
description: "EC2 instances, AMIs, EBS, and key pair management."
---

#  Module 2 – Getting Started with AWS & EC2

---

##  Learning Objectives

By the end of this module, you will be able to:

* Understand AWS global infrastructure (Regions and Availability Zones).
* Launch and configure your first EC2 instance.
* Create and copy Amazon Machine Images (AMIs).
* Manage and attach Elastic Block Store (EBS) volumes.

---

##  AWS Global Infrastructure

Amazon Web Services is a global cloud provider with **data centers distributed worldwide**, organized into:

| **Concept**                | **Definition**                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| **Region**                 | A physical location in the world containing multiple isolated data centers (Availability Zones). |
| **Availability Zone (AZ)** | One or more data centers with redundant power, networking, and connectivity within a Region.     |
| **Edge Location**          | A caching endpoint in Amazon CloudFront used for low-latency content delivery.                   |

>  **Analogy:**
> Think of a *Region* as a country, an *Availability Zone* as a city within that country, and an *Edge Location* as a local post office optimizing delivery speed.

---

###  Example AWS Regions (as per your notes)

| **Continent**            | **Regions**                                                           |
| ------------------------ | --------------------------------------------------------------------- |
| **North America**        | N. Virginia, Ohio, Oregon, Canada Central, AWS GovCloud (East & West) |
| **Europe**               | Ireland, London, Frankfurt, Paris, Madrid, Milan, Stockholm, Zurich   |
| **Asia Pacific**         | Sydney, Melbourne, Singapore, Tokyo, Mumbai, Seoul, Jakarta, Osaka    |
| **Middle East & Africa** | Bahrain, UAE, Cape Town, Tel Aviv                                     |
| **South America**        | São Paulo                                                             |
| **Coming Soon**          | Malaysia, Thailand, Saudi Arabia                                      |

---

##  Understanding EC2 (Elastic Compute Cloud)<img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/550f0759-26c1-4c5e-80e8-8ae39426633a" />

Amazon EC2 provides **resizable compute capacity** in the cloud.
It enables you to **launch virtual machines (instances)**, configure their storage, and manage them like on-prem servers.

###  EC2 Core Concepts

| **Term**                       | **Description**                                                     |
| ------------------------------ | ------------------------------------------------------------------- |
| **Instance**                   | A virtual server in the AWS cloud.                                  |
| **AMI (Amazon Machine Image)** | A preconfigured OS + application template used to launch instances. |
| **EBS (Elastic Block Store)**  | Persistent block-level storage attached to EC2 instances.           |
| **Instance Type**              | Defines the hardware resources (CPU, memory, storage).              |
| **Key Pair (.pem / .ppk)**     | Used for SSH authentication.                                        |
| **Security Group**             | Acts as a virtual firewall controlling inbound/outbound traffic.    |

---

##  EC2 Instance Families

| **Category**              | **Examples**   | **Use Case**                                        |
| ------------------------- | -------------- | --------------------------------------------------- |
| **General Purpose**       | T2, T3, M4, M5 | Balanced CPU and memory for web apps.               |
| **Compute Optimized**     | C4, C5         | High CPU workloads (batch processing, analytics).   |
| **Memory Optimized**      | R5, X1         | In-memory databases and caching.                    |
| **Storage Optimized**     | H1, I3, D2     | Data warehousing, analytics with high disk I/O.     |
| **Accelerated Computing** | P3, G5, F1     | GPU workloads, ML training, and graphics rendering. |

---

##  Step-by-Step: Launching an EC2 Instance

Let’s perform the full EC2 setup using the AWS Console.

###  **Hands-On Lab: Create and Connect an EC2 Instance**

1. **Sign in** to your AWS Management Console.
2. Navigate to **Services → EC2**.
3. Click **Launch Instance**.
4. **Choose AMI** – e.g., *Amazon Linux 2* or *Ubuntu Server*.
5. **Choose Instance Type** – select *t2.micro* (free tier eligible).
6. **Configure Instance Details**:

   * Keep default VPC and subnet (auto-assigned public IP).
   * Optionally, enable *Protect from accidental termination*.
7. **Add Storage**:

   * Root volume: 8 GB (EBS default).
   * You can add more volumes later.
8. **Add Tags**:

   * Key: `Name` → Value: `MyFirstEC2`.
9. **Configure Security Group**:

   * Inbound Rule → Type: SSH → Source: My IP.
10. **Review and Launch** → Create a new **Key Pair** → Download `.pem` file.
11. Click **Launch Instance**.

---

###  **Connect to EC2 (Linux Instance)**

1. In the EC2 Console, select your running instance.
2. Copy the **Public IPv4 DNS**.
3. Open a terminal and run:

```bash
chmod 400 mykey.pem
ssh -i "mykey.pem" ec2-user@ec2-xx-xx-xx.compute.amazonaws.com
```

✅ You’re now connected to your EC2 instance.

---

###  **(Optional)** Using Windows (PuTTY)

1. Convert `.pem` → `.ppk` using PuTTYgen.
2. Launch PuTTY → paste `ec2-user@public-ip`.
3. Load your `.ppk` under SSH → Auth → Connect.

---

##  Deep Dive: Amazon Machine Image (AMI) <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/d93789f1-0323-4f90-a64a-9f9f6fcda2cf" />


An **AMI** contains:

* Operating system
* Application server
* Storage configuration
* Virtualization type (HVM / PV)

---

###  **Hands-On: Create and Copy an AMI**

**Creating an AMI from a Running Instance**

1. Select your running instance → **Actions → Image and Templates → Create Image**.
2. Name the image (e.g., `MyWebServer-AMI`).
3. Click **Create Image**.
4. Navigate to **AMIs** in the sidebar → Verify image creation.

**Copying AMI to Another Region**

1. Select your created AMI → **Actions → Copy AMI**.
2. Choose a **Destination Region** (e.g., Oregon).
3. Click **Copy AMI** → Switch to destination region → Check AMIs list.

This allows quick **disaster recovery** or **multi-region deployment**.

---

##  EBS – Elastic Block Store (Intro) <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/e630fae0-a765-4897-9f88-330640aebc58" />


EBS provides **persistent block-level storage** for EC2 instances.

| **Feature**            | **Description**                                  |
| ---------------------- | ------------------------------------------------ |
| **Persistent Storage** | Data survives instance stop/start.               |
| **AZ-Scoped**          | Each EBS volume exists within a single AZ.       |
| **Snapshots**          | Incremental backups stored in S3.                |
| **Encryption**         | Supports data encryption at rest and in transit. |

### EBS Volume Types

| **Type**                           | **Performance**         | **Use Case**                     |
| ---------------------------------- | ----------------------- | -------------------------------- |
| **gp2 (General Purpose SSD)**      | 3 IOPS per GB (max 10K) | Boot volumes, dev/test workloads |
| **io1/io2 (Provisioned IOPS)**     | Up to 90K IOPS          | High-performance databases       |
| **st1 (Throughput Optimized HDD)** | 500 MB/s                | Big data, log processing         |
| **sc1 (Cold HDD)**                 | Low cost, 250 MB/s      | Archival, infrequent access      |

---

###  **Hands-On: Create and Attach an EBS Volume**

1. Navigate to **EC2 → Elastic Block Store → Volumes**.
2. Click **Create Volume** → Size: 8 GB → Availability Zone: same as instance.
3. Select the volume → **Actions → Attach Volume**.
4. Choose instance ID → Click **Attach**.
5. Connect to the instance and run:

```bash
lsblk                        # List block devices
sudo mkfs -t ext4 /dev/xvdf  # Format volume
sudo mkdir /data
sudo mount /dev/xvdf /data   # Mount volume
df -h                        # Verify mount
```

6. To unmount:

```bash
sudo umount /dev/xvdf
```

>  EBS data persists even after instance termination (if “Delete on Termination” is disabled).

---

##  Hypervisors Used by AWS

| **Hypervisor**   | **Type**              | **Usage**                                   |
| ---------------- | --------------------- | ------------------------------------------- |
| **Xen**          | Type 1                | Historically used across AWS infrastructure |
| **KVM**          | Type 1                | Used in Nitro-based EC2 instances           |
| **Nitro System** | Custom hardware + KVM | Provides near bare-metal performance        |

---

##  Key Takeaways

* AWS operates globally via **Regions** and **Availability Zones**.
* **EC2** provides flexible compute capacity.
* **AMI** templates simplify deployment and replication.
* **EBS** offers persistent, scalable block storage.
* The **Nitro System** enhances performance and isolation.

---

>  **Next Module Preview:**
> In **Module 3**, we’ll explore **Elastic Block Store (EBS)** in detail—covering Snapshots, Multi-Attach, Encryption, and **Elastic File System (EFS)** setup with multiple EC2 instances.

---
