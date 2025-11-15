# ──────────────────────────────────────────────────────────────
# MODULE 12
# File: _posts/2025-11-09-module12-aws-monitoring-and-logging-cloudwatch-cloudtrail-config.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 12: Monitoring & Logging (CloudWatch, CloudTrail, Config)"
date: 2025-11-09
categories: aws cloudwatch cloudtrail
tags: cloudwatch alarms logs audit
image: /assets/images/cloudwatch.jpg
description: "Observability, audit logs, and compliance tools."
---

#  Module 12 – AWS Monitoring and Logging (CloudWatch, CloudTrail, AWS Config) 🛰️

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand AWS’s core monitoring and observability tools: **CloudWatch**, **CloudTrail**, and **Config**.
* Set up **custom metrics, dashboards, and alarms** using CloudWatch.
* Use **CloudTrail** for auditing user and API activity.
* Enforce compliance and resource governance with **AWS Config**.
* Build automated responses to operational events.

---

## ☁️ Why Monitoring and Logging Matter

Monitoring helps ensure **availability, security, and performance** across your cloud infrastructure.

> 💡 **Analogy:**
> Think of **CloudWatch** as your “car dashboard” — it shows speed, fuel, and engine health.
> **CloudTrail** is your “black box” — it records everything that happens.
> **AWS Config** is your “compliance inspector” — it checks if your system follows the rules.

---

## 🧩 AWS CloudWatch Overview    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/0a12be47-22ee-4ba8-b241-5a74f3783186" />


**Amazon CloudWatch** collects and visualizes metrics, logs, and events from AWS resources and applications.

| **Feature**        | **Purpose**                                            |
| ------------------ | ------------------------------------------------------ |
| **Metrics**        | Numeric data points (e.g., CPUUtilization, NetworkIn). |
| **Logs**           | Application/system log data.                           |
| **Alarms**         | Automatic actions based on thresholds.                 |
| **Dashboards**     | Custom visual displays of metrics.                     |
| **Events / Rules** | Respond to system events automatically.                |

---

## 🧠 CloudWatch Architecture

```
     ┌───────────────┐
     │ AWS Resources │
     │ (EC2, RDS, S3)│
     └───────┬───────┘
             │
       ┌─────▼─────┐
       │ CloudWatch │
       │ Metrics &  │
       │ Logs Agent │
       └─────┬─────┘
             │
      ┌──────▼──────┐
      │  Dashboards │
      │  Alarms     │
      │  Events     │
      └─────────────┘
```

---

## 🧪 Hands-On Lab 1: Monitor EC2 Instance Metrics

### 🧰 **Objective**

Monitor EC2 performance and create alarms for CPU utilization.

---

### **Step 1: View Default Metrics**

1. Go to **CloudWatch → Metrics → EC2 → Per-Instance Metrics**.
2. Select `CPUUtilization`, `NetworkIn`, `DiskReadOps`, etc.
3. Click **Add to Dashboard** → Create a new dashboard `EC2-Monitoring`.

---

### **Step 2: Create Alarm**

1. In **CloudWatch → Alarms → Create Alarm**.
2. Choose Metric: `CPUUtilization`.
3. Set threshold: **Greater than 80% for 5 minutes**.
4. Action: **Send notification via SNS topic**.
5. Name: `HighCPUAlarm`.
6. Create.

**CLI Example:**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPUAlarm \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-123abc456 \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:ap-southeast-2:123456789012:NotifyMe
```

✅ Now, when CPU > 80% for 5 minutes, CloudWatch will trigger a notification.

---

### **Step 3: Custom Metrics**

You can also **publish custom metrics** from your application:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyAppMetrics" \
  --metric-name "PageLoadTime" \
  --value 2.5 \
  --unit Seconds
```

---

## ⚙️ CloudWatch Logs

CloudWatch Logs captures **application logs, system logs**, and **Lambda execution logs** for real-time analysis.

### **EC2 Log Integration**

1. Install CloudWatch Agent:

   ```bash
   sudo yum install amazon-cloudwatch-agent -y
   ```
2. Configure the agent:

   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
   ```
3. Start the agent:

   ```bash
   sudo systemctl start amazon-cloudwatch-agent
   ```

✅ Logs will stream to your chosen **Log Group** in CloudWatch.

---

### **Useful CLI Commands**

| **Action**           | **Command**                                      |
| -------------------- | ------------------------------------------------ |
| List log groups      | `aws logs describe-log-groups`                   |
| Tail log stream      | `aws logs tail /aws/lambda/HelloLambda --follow` |
| Create metric filter | `aws logs put-metric-filter`                     |
| Delete logs          | `aws logs delete-log-group`                      |

---

## 🧠 CloudWatch Events & EventBridge

* **EventBridge (formerly CloudWatch Events)** reacts to AWS service events (e.g., EC2 stop, Lambda error).
* You can **trigger Lambda**, **SNS**, or **Step Functions** automatically.

**Example Rule (EC2 Stop Event):**

```bash
aws events put-rule \
  --name EC2StopRule \
  --event-pattern '{"source": ["aws.ec2"], "detail-type": ["EC2 Instance State-change Notification"], "detail": {"state": ["stopped"]}}'
```

Attach a target (e.g., Lambda to send email).

---

## 🧩 AWS CloudTrail Overview    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/1c204ceb-9634-40d4-84ce-ad08d2f8bc0f" />


**AWS CloudTrail** records **API calls and console activity** for governance, compliance, and auditing.

| **Feature**       | **Description**                             |
| ----------------- | ------------------------------------------- |
| **Event History** | View last 90 days of management events.     |
| **Trails**        | Continuous log delivery to S3.              |
| **Data Events**   | Track S3 object or Lambda-level operations. |
| **Integration**   | Works with CloudWatch Logs for alerting.    |

---

## 🧪 Hands-On Lab 2: Enable and Review CloudTrail Logs

### 🧰 **Objective**

Audit user and API activity across your AWS account.

---

### **Step 1: Create a Trail**

1. Go to **CloudTrail → Create Trail**.
2. Trail name: `MyAuditTrail`.
3. Choose **Apply to all regions**.
4. Create a new **S3 bucket** for log storage.
5. Enable **Log file validation**.
6. Click **Create Trail**.

✅ CloudTrail will now record all API and console actions.

---

### **Step 2: View Events**

1. Navigate to **CloudTrail → Event History**.
2. Filter by:

   * **Event Source:** `ec2.amazonaws.com`
   * **Event Name:** `StartInstances`
3. View **user**, **timestamp**, and **source IP**.

**CLI Example:**

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=StartInstances
```

---

### **Step 3: Integrate CloudTrail with CloudWatch Logs**

1. In **CloudTrail settings**, enable **CloudWatch Logs**.
2. Choose an existing Log Group or create a new one.
3. This allows you to **create alarms** for suspicious activity.

---

## 🧠 AWS Config Overview    <img width="50" height="52" alt="image" src="https://github.com/user-attachments/assets/79cd3d90-17e5-4f60-bf2e-0ff6e6beb8b9" />


**AWS Config** continuously records resource configurations and checks for compliance.

| **Feature**                | **Purpose**                                                |
| -------------------------- | ---------------------------------------------------------- |
| **Configuration Recorder** | Tracks changes to resource settings.                       |
| **Rules**                  | Evaluate compliance (e.g., “S3 bucket must be encrypted”). |
| **Snapshots**              | Capture point-in-time configuration states.                |
| **Integration**            | Works with CloudTrail and Security Hub.                    |

---

## 🧪 Hands-On Lab 3: Enforce S3 Encryption Using AWS Config

### 🧰 **Objective**

Ensure all S3 buckets have encryption enabled.

---

### **Step 1: Enable Config**

1. Go to **AWS Config → Get Started**.
2. Record all resource types.
3. Choose a new **S3 bucket** for snapshots.
4. Enable.

---

### **Step 2: Add Managed Rule**

1. Click **Add Rule → s3-bucket-server-side-encryption-enabled**.
2. Evaluate compliance automatically.
3. Wait a few minutes → non-compliant buckets will appear.

---

### **Step 3: Remediate Non-Compliant Resources**

You can automate remediation using **Systems Manager (SSM)**.

**Example CLI:**

```bash
aws s3api put-bucket-encryption \
  --bucket mybucket \
  --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

✅ This ensures compliance across all future S3 buckets.

---

## 🧩 CloudWatch vs CloudTrail vs Config – Comparison

| **Service**    | **Purpose**                     | **Data Type**           | **Primary Use**          |
| -------------- | ------------------------------- | ----------------------- | ------------------------ |
| **CloudWatch** | Performance monitoring          | Metrics & logs          | Alerting & visualization |
| **CloudTrail** | Governance & auditing           | API events              | User activity tracking   |
| **AWS Config** | Compliance & policy enforcement | Resource configurations | Compliance auditing      |

---

## 🧾 Key Takeaways

* **CloudWatch** monitors metrics, collects logs, and triggers automated responses.
* **CloudTrail** audits every API and console action across AWS accounts.
* **AWS Config** ensures compliance through continuous resource evaluation.
* Together, they form AWS’s **Observability and Governance** foundation.
* Integrate with **SNS, Lambda, and EventBridge** for automated incident response.

---

> 🪄 **Next Module Preview:**
> In **Module 13**, we’ll explore **AWS Identity and Access Management (IAM) and Security Services** — learning how to secure your cloud environment with roles, policies, SSO, and encryption.

---
