---
title: "Module 3 - Elastic Block Store (EBS) and Elastic File System (EFS)"
date: 2025-10-30
categories: [aws, storage, ebs, efs]
tags: [aws, ebs, efs, snapshots, encryption, lifecycle]
image: /assets/images/ebs.jpg
description: "Persistent block and shared file storage, snapshots, Multi-Attach, Data Lifecycle Manager, and EFS setup."
---


---

# 💾 Module 3 – Elastic Block Store (EBS) and Elastic File System (EFS)

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Explain the purpose and architecture of **Amazon EBS** and **EFS**.
* Differentiate between EBS volume types and their performance characteristics.
* Create, mount, and back up EBS volumes using snapshots.
* Configure **EBS Multi-Attach** and **Data Lifecycle Manager (DLM)** policies.
* Deploy a shared file system with **Amazon EFS** across multiple EC2 instances.

---

## 🧠 Concept Overview

### 🧩 What is Amazon Elastic Block Store (EBS)?    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/8066640d-41ca-4134-83b9-dfaa5ecfcb78" />


EBS provides **persistent block-level storage** for Amazon EC2 instances.
Think of it as an **external hard drive** for your virtual machine that can be detached, reattached, resized, and backed up independently.

> 💡 **Analogy:**
> Imagine your EC2 instance as a laptop and EBS as a portable SSD — you can plug it in, store data, unplug it, and connect it to another laptop without losing data.

---

## 🧱 Key Characteristics of EBS

| **Feature**               | **Description**                                                             |
| ------------------------- | --------------------------------------------------------------------------- |
| **Persistent Storage**    | Data survives instance stop/start and can be reattached to other instances. |
| **AZ Bound**              | Each volume resides within one Availability Zone.                           |
| **Automatic Replication** | Data is automatically replicated within the same AZ for durability.         |
| **Scalable**              | Volumes can be resized dynamically with minimal downtime.                   |
| **Encrypted**             | Supports encryption of data at rest and in transit.                         |
| **Snapshots**             | Incremental backups stored in Amazon S3.                                    |

---

## ⚙️ EBS Volume Types

| **Type**                             | **Description**                                      | **Performance (IOPS / Throughput)**   | **Use Case**                               |
| ------------------------------------ | ---------------------------------------------------- | ------------------------------------- | ------------------------------------------ |
| **gp2 (General Purpose SSD)**        | Default SSD storage for general workloads.           | 3 IOPS/GB (max 10,000 IOPS), 160 MB/s | Boot volumes, dev/test workloads.          |
| **gp3 (Enhanced General Purpose)**   | Latest generation with independent IOPS scaling.     | Up to 16,000 IOPS, 1,000 MB/s         | Cost-effective upgrade from gp2.           |
| **io1 / io2 (Provisioned IOPS SSD)** | High-performance storage for latency-sensitive apps. | Up to 90,000 IOPS                     | Critical databases, SAP workloads.         |
| **st1 (Throughput Optimized HDD)**   | Magnetic HDD optimized for sequential read/write.    | Up to 500 MB/s                        | Big data, log processing, data warehouses. |
| **sc1 (Cold HDD)**                   | Lowest-cost HDD for infrequently accessed data.      | Up to 250 MB/s                        | Archival, backups, and cold storage.       |

---

## 🧩 IOPS and Throughput

* **IOPS (Input/Output Operations per Second):** Measures random access speed (important for databases).
* **Throughput:** Measures sequential read/write speed (important for streaming or large files).

---

## 🧪 Hands-On: Create and Attach an EBS Volume

### **Objective:**

Create an EBS volume, attach it to an EC2 instance, and mount it for use.

---

### **Step 1: Create the Volume**

1. Go to **EC2 → Elastic Block Store → Volumes**.
2. Click **Create Volume**.
3. Choose:

   * **Type:** gp2 (8 GB)
   * **Availability Zone:** same as EC2 instance
4. Click **Create Volume**.

---

### **Step 2: Attach the Volume**

1. Select the new volume → **Actions → Attach Volume**.
2. Choose your **EC2 Instance ID** → click **Attach**.

---

### **Step 3: Mount the Volume on EC2**

Connect to your instance:

```bash
ssh -i mykey.pem ec2-user@<public-ip>
```

Then execute:

```bash
lsblk                                # List block devices
sudo file -s /dev/xvdf               # Check if volume is formatted
sudo mkfs -t ext4 /dev/xvdf          # Format the volume
sudo mkdir /data                     # Create a mount point
sudo mount /dev/xvdf /data           # Mount the volume
df -h                                # Verify the mount
```

To unmount:

```bash
sudo umount /data
```

---

### **Step 4: Take a Snapshot (Backup)**

1. Navigate to **Volumes → Select Volume → Actions → Create Snapshot**.
2. Name it (e.g., `EBS-Backup-Oct2025`) → Click **Create Snapshot**.
3. Snapshots are incremental and stored in **S3**, saving cost and space.

---

### **Step 5: Create Volume from Snapshot**

You can restore the same volume anytime:

1. Go to **Snapshots → Select Snapshot → Actions → Create Volume**.
2. Choose AZ → Click **Create Volume** → Attach to another EC2 if needed.

---

## 🔁 EBS Multi-Attach

**Feature:**
Attach a single **Provisioned IOPS (io1/io2)** volume to **up to 16 EC2 instances** within the same AZ.

**Use Case:**

* Clustered databases
* HA (high-availability) applications
* Shared read/write volumes

**Requirements:**

* Must be io1/io2 volume type
* Nitro-based EC2 instances only
* Multi-Attach enabled during creation

---

## 🧩 Data Lifecycle Manager (DLM)

**Purpose:** Automates snapshot creation, retention, and deletion.

**Key Benefits:**

* Reduces manual backup management.
* Enforces compliance through scheduled policies.
* Automatically removes outdated snapshots.

**Quotas:**

* Up to **100 policies per region**
* **45 tags per resource**
* One schedule per policy

---

### 🧪 **Hands-On: Configure DLM Policy**

1. Navigate to **EC2 → Lifecycle Manager → Create Lifecycle Policy**.
2. **Policy Type:** EBS Snapshot Policy.
3. **Target Resource Type:** Volume → Select using tags (e.g., `Backup=true`).
4. **Schedule:**

   * Frequency: Every 12 hours
   * Retain snapshots: 3
5. Click **Create Policy**.

✅ AWS will now automatically back up and rotate your snapshots.

---

## 🔒 EBS Encryption

* EBS supports **encryption for all volume types**.
* Encrypts data **at rest, in transit, and in snapshots**.
* Managed by **AWS KMS** (Key Management Service).
* Transparent to the instance — no code change required.

> **Tip:** You can re-encrypt or copy snapshots to change encryption settings.

---

## 📁 Elastic File System (EFS)  <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/6cc60856-654b-4db8-a089-b423c78b0186" />


Amazon EFS provides **scalable, fully managed shared file storage** for EC2 instances.

> 💡 **Analogy:**
> EFS is like a shared network drive accessible from multiple computers simultaneously — all reading and writing the same files.

---

### 🔑 Key Features of EFS

| **Feature**             | **Description**                                     |
| ----------------------- | --------------------------------------------------- |
| **Fully Managed**       | No provisioning; grows and shrinks automatically.   |
| **Multi-AZ Redundancy** | Data stored redundantly across multiple AZs.        |
| **Scalable**            | Petabyte-scale storage with automatic elasticity.   |
| **Concurrent Access**   | Mountable on multiple EC2 instances simultaneously. |
| **Linux-Compatible**    | Works natively with Linux (NFS protocol).           |
| **Durable and Secure**  | Supports encryption at rest and in transit.         |

---

## 🧪 Hands-On: Create and Mount EFS

### **Step 1: Create an EFS**

1. Navigate to **Services → EFS → Create File System**.
2. Choose **VPC** → Select **Availability Zones**.
3. Set **Performance Mode:** General Purpose.
4. **Throughput Mode:** Bursting.
5. Add tags → click **Create**.

---

### **Step 2: Mount EFS on EC2**

Connect to your EC2 instance and install NFS utilities:

```bash
sudo apt-get update
sudo apt-get install -y nfs-common
```

Mount the file system:

```bash
sudo mkdir /efs
sudo mount -t efs fs-<your-id>:/ /efs
cd /efs
touch testfile.txt
ls -l
```

---

### **Step 3: Mount EFS on Multiple Instances**

Repeat the above steps on another EC2 instance (same VPC/subnet).
Then verify file visibility:

```bash
ls -l /efs
# You should see testfile.txt created from the first instance
```

✅ Both instances now share the same file system — perfect for web servers, microservices, or distributed apps.

---

### **Step 4: Unmount (Optional)**

```bash
sudo umount /efs
```

---

## ⚖️ EBS vs. EFS Comparison

| **Feature**       | **EBS**                                      | **EFS**                        |
| ----------------- | -------------------------------------------- | ------------------------------ |
| **Type**          | Block storage                                | File storage                   |
| **Accessibility** | One instance (Multi-Attach for io1/io2 only) | Multiple instances             |
| **Performance**   | High IOPS for single instance                | Shared, scalable throughput    |
| **Persistence**   | AZ-scoped                                    | Region-scoped (multi-AZ)       |
| **Protocol**      | Block-level                                  | NFS (Linux only)               |
| **Best For**      | Databases, OS drives                         | Shared content, analytics, CMS |

---

## 📚 Case Study: EC2 + EBS + EFS Deployment

**Scenario:**
A company hosts a web application needing:

* EC2 compute layer
* EBS for OS and app data
* EFS for shared content between servers

### **Steps Summary:**

1. Launch EC2 instance in **us-east-1a**.
2. Attach 2 EBS volumes (8 GB each).
3. Create an AMI → replicate to **us-west-2**.
4. Mount and back up EBS volumes → snapshot one, delete the other.
5. Create EFS and mount it to both EC2 instances.
6. Test shared data consistency.

Result:
✅ Scalable, redundant architecture across Regions with shared storage and recoverable data.

---

## 🧾 Key Takeaways

* **EBS** provides block-level persistent storage for individual EC2 instances.
* **EFS** delivers shared, elastic file storage accessible by multiple instances.
* **Snapshots** enable efficient backups and cross-region replication.
* **Data Lifecycle Manager** automates EBS backup management.
* **Encryption** ensures security for data at rest and in transit.

---

> 🪄 **Next Module Preview:**
> In **Module 4**, we’ll explore **Amazon S3 – Simple Storage Service**, covering buckets, object storage, versioning, lifecycle rules, and cross-region replication with real console labs.

---
