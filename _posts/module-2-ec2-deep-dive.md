---
title: "AWS EC2 Masterclass: Launch, Secure & Scale Your First Instance"
date: 2025-11-04
categories: aws ec2 iam security vpc
tags: ec2 ami ebs instance-types key-pairs
image: /assets/images/ec2-launch.jpg
description: "Step-by-step guide to launching, configuring, and securing EC2 instances. Learn AMIs, instance types, EBS, key pairs, and the Nitro System."
---

![EC2 Instance](/assets/images/ec2-dashboard.png)

---

## What Is Amazon EC2?

**EC2 (Elastic Compute Cloud)** = **resizable virtual servers** in the cloud.

You launch **instances** from **AMIs** (templates), attach **EBS storage**, and manage via **SSH/RDP**.

> [Official Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

---

## AWS Global Infrastructure

| Concept | Definition |
|--------|------------|
| **Region** | Geographic location (e.g., `ap-southeast-2`) |
| **AZ** | Isolated data center within a region |
| **Edge Location** | CDN endpoint (CloudFront) |

---

## EC2 Core Concepts

| Term | Description |
|------|-----------|
| **Instance** | Virtual server |
| **AMI** | Pre-built OS + app template |
| **EBS** | Persistent block storage |
| **Instance Type** | CPU, RAM, storage config |
| **Key Pair** | `.pem` for SSH |
| **Security Group** | Virtual firewall |

---

## EC2 Instance Families (2025)

| Family | Use Case |
|-------|----------|
| **T4g/T3** | Burstable, general |
| **M7g** | Balanced (Graviton) |
| **C7g** | Compute-optimized |
| **R7g** | Memory-optimized |
| **I4i** | Storage-optimized |

> [Instance Types](https://aws.amazon.com/ec2/instance-types/)

---

## Hands-On: Launch Your First EC2

### Step 1: Create Free Tier Account
→ [https://aws.amazon.com/free/](https://aws.amazon.com/free/)

### Step 2: Launch Instance
1. **AMI**: Amazon Linux 2
2. **Type**: `t2.micro` (free tier)
3. **VPC**: Default
4. **Storage**: 8 GB EBS
5. **Tag**: `Name = MyFirstEC2`
6. **Security Group**: Allow SSH (22) from **My IP**
7. **Key Pair**: Download `.pem`

### Step 3: Connect
```bash
chmod 400 mykey.pem
ssh -i mykey.pem ec2-user@ec2-xx-xx-xx.compute.amazonaws.com
```

## Create & Copy AMI
# From Console
Actions → Image and Templates → Create Image
Name: MyWebServer-AMI
→ Copy to Oregon (us-west-2)

## EBS: Persistent Storage
Feature,Detail
AZ-bound,Must be in same AZ as instance
Snapshots,Incremental backups in S3
Encryption,KMS-managed

## Hypervisors in AWS
Type,Usage
Xen,Legacy
KVM + Nitro,"Modern, near bare-metal"

### Key Takeaways

EC2 = virtual servers
AMI = golden images
EBS = persistent disks
Nitro = performance + security

