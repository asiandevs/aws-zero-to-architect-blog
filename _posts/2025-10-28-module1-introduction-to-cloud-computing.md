---
title: "Module 1 – Introduction to Cloud Computing"
date: 2025-10-28
categories: aws cloud-computing iaas paas saas
tags: aws azure gcp oci virtualization
image: /assets/images/cc.jpg
description: "Virtualization, Cloud service models, Deployment models, AWS overview."
---
---

# 🧭 Module 1 – Introduction to Cloud Computing

---

## 🎯 Learning Objectives

By the end of this module, you’ll be able to:

* Explain what cloud computing is and how it evolved from virtualization.
* Describe cloud deployment and service models.
* Identify key advantages of cloud adoption.
* Compare leading cloud providers — AWS, Azure, Google Cloud, and Oracle Cloud.

---

## ☁️ What Is Cloud Computing?

**Cloud computing** is the on-demand delivery of computing resources—servers, storage, databases, networking, software, and analytics—over the internet (“the cloud”).
It replaces the need for owning or maintaining physical data centers and servers.

### 🔑 Key Concept

> **Cloud Computing = Virtualization + Internet**

Virtualization allows multiple operating systems and applications to run on a single physical server, while the internet provides scalable, on-demand access.

---

## 💡 Real-World Analogy

> Think of cloud computing like renting an apartment instead of buying a house.
> You still get space and amenities, but you don’t worry about maintenance, electricity, or infrastructure costs—someone else manages that for you.

---

## ⚙️ Advantages of Cloud Computing

| **Feature**           | **Description**                                                    |
| --------------------- | ------------------------------------------------------------------ |
| **Cost Efficiency**   | Pay only for what you use — no upfront hardware investment.        |
| **High Availability** | Services remain available with minimal downtime.                   |
| **Scalability**       | Automatically scale resources up or down based on demand.          |
| **Security**          | Cloud providers implement robust physical and digital protections. |
| **Flexibility**       | Access resources from anywhere, enabling remote collaboration.     |
| **Rapid Deployment**  | Deploy applications and environments within minutes.               |

---

## 🧩 Cloud Deployment Models

| **Model**         | **Description**                                                  | **Use Case Example**                                    |
| ----------------- | ---------------------------------------------------------------- | ------------------------------------------------------- |
| **Public Cloud**  | Shared infrastructure over the internet (e.g., AWS, Azure, GCP). | Hosting web apps for global users.                      |
| **Private Cloud** | Dedicated infrastructure for one organization.                   | Banking or government systems needing isolation.        |
| **Hybrid Cloud**  | Mix of public and private environments.                          | Sensitive data on-prem, front-end apps in public cloud. |

---

## 🧱 Cloud Service Models

| **Model**                       | **Abbreviation** | **Definition**                            | **Example AWS Service**    |
| ------------------------------- | ---------------- | ----------------------------------------- | -------------------------- |
| **Infrastructure as a Service** | IaaS             | Virtualized hardware resources on demand. | EC2, EBS, VPC              |
| **Platform as a Service**       | PaaS             | Managed runtime and development tools.    | Elastic Beanstalk          |
| **Software as a Service**       | SaaS             | Ready-to-use applications over internet.  | Amazon Chime, AWS WorkMail |

---

## ☁️ Major Cloud Service Providers

| **Provider**                          | **Market Position** | **Example Customers**                 |
| ------------------------------------- | ------------------- | ------------------------------------- |
| **Amazon Web Services (AWS)**         | #1                  | Netflix, Reddit, U.S. NSA             |
| **Microsoft Azure**                   | #2                  | BMW, Samsung Electronics, PwC         |
| **Google Cloud Platform (GCP)**       | #3                  | LinkedIn, PayPal, Spotify             |
| **Oracle Cloud Infrastructure (OCI)** | #4                  | Zoom, 8×8 Inc., Australian Government |

---

## 🧠 Key AWS Concepts 

| **Domain**     | **Example Services**      |
| -------------- | ------------------------- |
| **Compute**    | EC2, Lambda               |
| **Storage**    | S3, EBS, EFS              |
| **Database**   | RDS, DynamoDB             |
| **Networking** | VPC, CloudFront, Route 53 |

---

## 🧪 Hands-On Lab – Understanding Cloud & Virtualization

### 🧰 **Objective**

See how virtualization enables multiple environments on a single host and understand how AWS uses it to power EC2.

---

### 🖥️ Step 1: Identify Hypervisor Types

| **Type**                | **Architecture**                      | **Usage in AWS**                                         |
| ----------------------- | ------------------------------------- | -------------------------------------------------------- |
| **Type 1 (Bare-Metal)** | Hardware → Hypervisor → Guest OS      | ✅ Used by AWS                                            |
| **Type 2 (Hosted)**     | Hardware → OS → Hypervisor → Guest OS | Used for local testing (VirtualBox / VMware Workstation) |

AWS primarily uses **Xen** and **KVM** hypervisors under the hood.

> **Tip:** The “Nitro System” combines custom hardware + lightweight hypervisor for near-bare-metal performance.

---

### 🧩 Step 2: Explore Virtualization Locally (Optional Pre-AWS Lab)

**On your laptop:**

```bash
# Example using VirtualBox
sudo apt install virtualbox
virtualbox
# Create a new VM and install Ubuntu to simulate isolated environments
```

Observe how each VM acts as an independent system — similar to AWS EC2 instances.

---

### ☁️ Step 3: Create an AWS Free-Tier Account

Visit: [https://aws.amazon.com/resources/create-account/](https://aws.amazon.com/resources/create-account/)

* Verify email and billing details.
* Sign in to the AWS Management Console.
* Familiarize yourself with the **Regions & Services** layout.

---

### 🧭 Step 4: Navigate the AWS Console

1. Go to **Services → Compute → EC2**.
2. Click **Launch Instance** (you’ll perform this fully in Module 2).
3. Observe available **Regions**, **Instance Types**, and **Pricing Tiers**.
4. Note that the **Region** you select determines **data residency** and **pricing**.

---

## 🧾 Key Takeaways

* Cloud computing delivers virtualized IT resources as a service.
* Deployment = Public | Private | Hybrid clouds.
* Services = IaaS | PaaS | SaaS.
* AWS dominates the market with global infrastructure (Regions + Availability Zones).
* Virtualization underpins all compute resources in AWS (EC2).
* AWS’s Nitro System ensures secure, high-performance virtual machines.

---

> 🪄 **Next Module Preview:**
> In **Module 2**, we’ll dive deep into **AWS EC2**, exploring Regions, Availability Zones, AMIs, instance types, and how to launch your first instance from the console and CLI.

---
