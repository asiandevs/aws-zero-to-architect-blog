---

# 🚀 Module 15 – AWS Migration, Backup, and Disaster Recovery

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand AWS’s **migration strategies, services, and tools**.
* Perform **server and database migration** using AWS-native tools.
* Design **backup strategies** using S3, EBS Snapshots, and AWS Backup.
* Build **Disaster Recovery (DR)** architectures with RTO/RPO objectives.
* Implement **cross-region and cross-account** recovery strategies.

---

## ☁️ Why Migration and DR Matter

Every cloud journey starts with migration — and every successful cloud environment must prepare for failure.

> 💡 **Analogy:**
> Migration is like **moving your home** — you decide what to bring, how to pack, and how to settle in.
> Disaster Recovery (DR) is like having a **backup home plan** — ready to move into if something goes wrong.

---

## 🧩 AWS Migration Phases

| **Phase**                  | **Description**                                             | **AWS Tools**                                      |
| -------------------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| **1. Assess**              | Discover current environment, evaluate readiness.           | Migration Evaluator, Application Discovery Service |
| **2. Mobilize**            | Build landing zone, migration plan, and baseline.           | Control Tower, AWS Organizations                   |
| **3. Migrate & Modernize** | Execute migration, modernize apps (containers, serverless). | SMS, DMS, CloudEndure, Snow Family                 |

---

## 🧠 The 6 “R” Migration Strategies

| **Strategy**                          | **Description**                          | **Example**                             |
| ------------------------------------- | ---------------------------------------- | --------------------------------------- |
| **Rehost (Lift & Shift)**             | Move workloads as-is to cloud.           | Migrate VMs via SMS.                    |
| **Replatform (Lift, Tinker & Shift)** | Minor optimizations during migration.    | Move to RDS instead of self-managed DB. |
| **Repurchase**                        | Move to SaaS-based solution.             | Shift CRM to Salesforce.                |
| **Refactor/Re-architect**             | Redesign apps for cloud-native services. | Move monolith to microservices.         |
| **Retire**                            | Decommission unused resources.           | Old archive servers.                    |
| **Retain**                            | Keep apps on-prem temporarily.           | Compliance-bound workloads.             |

---

## 🧪 Hands-On Lab 1: Migrate On-Prem VM to AWS EC2 Using Application Migration Service (MGN)

### 🧰 **Objective**

Migrate an on-premises virtual machine to AWS EC2 with minimal downtime.

---

### **Step 1: Set Up Replication Agent on Source Server**

Download and install AWS Replication Agent:

```bash
curl -o ./aws-replication-agent-installer-init.py https://aws-application-migration-service-ap-southeast-2.s3.amazonaws.com/latest/linux/aws-replication-agent-installer-init.py
sudo python3 aws-replication-agent-installer-init.py --region ap-southeast-2
```

---

### **Step 2: Monitor Replication**

* Open **AWS MGN Console → Source Servers**
* Verify the replication status as **“Healthy”**

---

### **Step 3: Launch Test Instance**

* Select **Launch Test Instance**
* Verify EC2 instance boots successfully in target subnet.

---

### **Step 4: Cutover**

* Once validated, select **Launch Cutover Instance**.
* Decommission the on-prem server.

✅ Migration is now complete — with minimal downtime and full replication.

---

## 🧩 AWS Database Migration Service (DMS)

**DMS** helps migrate databases quickly and securely — with minimal downtime.

| **Use Case**                | **Source**                     | **Target**                 |
| --------------------------- | ------------------------------ | -------------------------- |
| **Homogeneous Migration**   | Oracle → Oracle                | Minimal conversion         |
| **Heterogeneous Migration** | SQL Server → Aurora PostgreSQL | Schema conversion required |

**Related Tool:**

* **AWS Schema Conversion Tool (SCT)** converts DB schema between engines.

---

### **CLI Example: Create a DMS Replication Task**

```bash
aws dms create-replication-task \
  --replication-task-identifier "task1" \
  --source-endpoint-arn arn:aws:dms:src \
  --target-endpoint-arn arn:aws:dms:target \
  --migration-type full-load-and-cdc \
  --table-mappings file://table-mappings.json
```

✅ Data is migrated continuously with change data capture (CDC).

---

## 🧩 AWS Snow Family

Used for **offline data migration** when network bandwidth is limited.

| **Device**        | **Capacity** | **Use Case**                   |
| ----------------- | ------------ | ------------------------------ |
| **Snowcone**      | 8 TB         | Edge data collection           |
| **Snowball Edge** | 80 TB–100 TB | Bulk transfer & edge compute   |
| **Snowmobile**    | 100 PB       | Massive data center migrations |

> 💡 **Example:** Moving 1 PB of video archives using Snowball Edge instead of over-the-wire transfer.

---

## 🧪 Hands-On Lab 2: Back Up and Restore EC2 Using AWS Backup

### 🧰 **Objective**

Automate EC2 backups using the centralized **AWS Backup** service.

---

### **Step 1: Create Backup Plan**

1. Go to **AWS Backup → Create Backup Plan**.
2. Name: `DailyEC2Backup`.
3. Set frequency: **Daily @ 1 AM UTC**.
4. Retention: 30 days.
5. Assign Resource: Select EC2 instances.
6. Click **Create Plan**.

---

### **Step 2: Monitor Backups**

* Navigate to **AWS Backup → Jobs** to view progress.
* You’ll see daily jobs completing and recovery points created.

---

### **Step 3: Restore Instance**

1. Select **Protected Resource → Restore**.
2. Choose recovery point.
3. Select target subnet and security group.
4. Launch restore.

✅ EC2 instance restored successfully from backup snapshot.

---

## 🧩 EBS Snapshots (CLI Example)

```bash
# Create a snapshot
aws ec2 create-snapshot --volume-id vol-xxxx --description "Backup of DB volume"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Restore snapshot as new volume
aws ec2 create-volume --snapshot-id snap-xxxx --availability-zone ap-southeast-2a
```

> 💡 Snapshots are incremental and can be shared across accounts or copied across regions.

---

## 🧩 AWS Backup vs. Manual Snapshots

| **Feature**                | **AWS Backup**          | **Manual Snapshot** |
| -------------------------- | ----------------------- | ------------------- |
| **Automation**             | Yes (Scheduled)         | Manual              |
| **Cross-Region Backup**    | Supported               | Needs copy command  |
| **Resource Coverage**      | EC2, RDS, DynamoDB, EFS | Specific to service |
| **Centralized Management** | Yes                     | No                  |

---

## 🧩 Disaster Recovery Strategies

| **Strategy**                 | **Cost** | **Recovery Time Objective (RTO)** | **Recovery Point Objective (RPO)** | **Description**                                                 |
| ---------------------------- | -------- | --------------------------------- | ---------------------------------- | --------------------------------------------------------------- |
| **Backup & Restore**         | 💲       | Hours                             | Hours                              | Data backed up and restored on demand.                          |
| **Pilot Light**              | 💲💲     | Minutes–Hours                     | Minutes                            | Core components always running; rest activated during disaster. |
| **Warm Standby**             | 💲💲💲   | Minutes                           | Minutes                            | Scaled-down environment continuously running.                   |
| **Multi-Site Active/Active** | 💲💲💲💲 | Seconds                           | Near-zero                          | Full capacity in multiple regions; instant failover.            |

> 💡 **Analogy:**
>
> * **Backup & Restore:** Keep copies in cold storage.
> * **Warm Standby:** Keep a smaller spare system ready to boot up.
> * **Active/Active:** Two fully working systems sharing load.

---

## 🧪 Hands-On Lab 3: Cross-Region DR for S3 and RDS

### 🧰 **Objective**

Replicate data automatically to another AWS region for resilience.

---

### **Part 1: S3 Cross-Region Replication (CRR)**

1. Enable **Versioning** on both source and destination buckets.
2. In **Source Bucket → Management → Replication Rules → Create Rule**:

   * Destination: Another region (e.g., ap-southeast-1)
   * IAM Role: Create new replication role.

**CLI Example:**

```bash
aws s3api put-bucket-replication \
  --bucket my-source-bucket \
  --replication-configuration file://replication.json
```

✅ Files uploaded to the source bucket now automatically replicate to the destination bucket.

---

### **Part 2: RDS Cross-Region Read Replica**

1. Go to **RDS → Databases → Select DB → Actions → Create Read Replica**.
2. Destination Region: **ap-southeast-1**.
3. Choose storage type and encryption.
4. Launch replica.

✅ Promoting the replica during outage allows quick recovery in another region.

---

## 🧠 Testing Your DR Plan

> 💬 “A DR plan is only as good as your last test.”

### **Checklist:**

* Automate **failover scripts** (Route 53 health checks).
* Schedule **quarterly DR drills**.
* Verify **data integrity** post-recovery.
* Monitor RTO/RPO compliance metrics.

---

## 🧾 Key Takeaways

* AWS offers flexible migration paths: **Lift & Shift, Replatform, Refactor**.
* **Application Migration Service** and **DMS** automate server/database migration.
* Use **AWS Backup** for centralized, policy-driven protection.
* Implement DR architectures aligned to **RTO/RPO goals**.
* For maximum resilience → Use **cross-region replication** and **multi-site** models.
* Test and automate failover to ensure **business continuity**.

---

> 🪄 **Next Module Preview:**
> In **Module 16**, we’ll dive into **AWS Cost Optimization and FinOps Practices** — learning how to monitor, control, and optimize cloud costs using budgets, cost explorer, and resource tagging strategies.

---
