---
title: "Module 13: Identity & Access Management (IAM)"
date: 2025-11-10
categories: aws iam security
tags: iam users roles policies mfa
image: /assets/images/iam.jpg
description: "Users, roles, policies, and MFA."
---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand **Identity and Access Management (IAM)** fundamentals.
* Create and manage **users, groups, roles, and policies**.
* Apply **least privilege** and **secure authentication** principles.
* Use **AWS KMS**, **Secrets Manager**, and **Security Hub** for encryption and compliance.
* Implement **Multi-Factor Authentication (MFA)** and **Service Control Policies (SCPs)** for enterprise accounts.

---

## ☁️ What Is AWS IAM?    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/ce472213-880d-4776-a3d9-8cefc5a4cc42" />


**AWS Identity and Access Management (IAM)** lets you control who can access AWS resources, what actions they can perform, and under what conditions.

> 💡 **Analogy:**
> IAM is like the **security system in a building** — it controls who can enter (authentication), which rooms they can access (authorization), and when (conditions).

---

## 🧩 IAM Core Components

| **Component**  | **Description**                                                 |
| -------------- | --------------------------------------------------------------- |
| **Users**      | Individual identities for people or applications.               |
| **Groups**     | Collections of users with shared permissions.                   |
| **Roles**      | Temporary credentials for AWS services or cross-account access. |
| **Policies**   | JSON documents that define permissions.                         |
| **Federation** | Integrates external identities (AD, SSO, Cognito).              |

---

## ⚙️ IAM Policy Structure

IAM policies are JSON documents consisting of:

* **Version**
* **Statement** with **Effect**, **Action**, **Resource**, and optional **Condition**

**Example Policy (EC2 Read-Only):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:Describe*"
    ],
    "Resource": "*"
  }]
}
```

---

## 🧠 Policy Evaluation Logic

```
Explicit Deny → Overrides all other rules  
|
|-- Allow → Grants access  
|
|-- Default (Implicit Deny) → Blocks unless explicitly allowed
```

---

## 🧪 Hands-On Lab 1: Create IAM Users, Groups, and Policies

### 🧰 **Objective**

Create users and assign permissions via groups and policies.

---

### **Step 1: Create a Group**

1. Go to **IAM → User Groups → Create Group**.
2. Name: `Developers`.
3. Attach policy: `AmazonEC2ReadOnlyAccess`.
4. Click **Create Group**.

---

### **Step 2: Create a User**

1. **IAM → Users → Create User**.
2. Username: `devuser1`.
3. Assign user to group `Developers`.
4. Enable **AWS Management Console access** and set password.
5. Click **Create User**.

✅ `devuser1` can now log in to the AWS console and view EC2 instances but cannot modify them.

---

### **Step 3: Test Access**

Log in as `devuser1` via IAM sign-in URL.
Try:

* Viewing EC2 → ✅ Allowed
* Terminating instance → ❌ Denied

This validates the policy enforcement.

---

### **CLI Example:**

```bash
aws iam create-group --group-name Developers
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

aws iam create-user --user-name devuser1
aws iam add-user-to-group --user-name devuser1 --group-name Developers
```

---

## 🧩 IAM Roles and Cross-Service Access

Roles provide **temporary credentials** that services can assume to perform actions.

### **Example Use Case:**

* EC2 instance accessing S3 without embedding credentials.

**Steps:**

1. Create an **IAM Role** → Use Case: **EC2**.
2. Attach policy: `AmazonS3ReadOnlyAccess`.
3. Launch EC2 → Attach this role.
4. Inside EC2, run:

```bash
aws s3 ls
```

✅ You can now access S3 without using static credentials.

---

## ⚙️ IAM Trust Policy Example

Defines which entity (service/account) can assume a role.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
```

---

## 🧪 Hands-On Lab 2: Multi-Factor Authentication (MFA)

### 🧰 **Objective**

Add MFA to protect user logins.

1. Go to **IAM → Users → devuser1 → Security Credentials**.
2. Choose **Assign MFA Device** → Select **Virtual MFA (Google Authenticator)**.
3. Scan QR code and enter 2 consecutive codes.
4. Save settings.

✅ MFA now adds a second authentication layer during login.

---

## 🧠 Best Practices for IAM Security

| **Best Practice**                           | **Description**                                    |
| ------------------------------------------- | -------------------------------------------------- |
| **Use IAM Roles instead of long-term keys** | Avoid static credentials in code.                  |
| **Enable MFA for privileged users**         | Adds extra security.                               |
| **Follow least privilege principle**        | Grant only required permissions.                   |
| **Rotate access keys every 90 days**        | Reduces risk exposure.                             |
| **Use IAM Access Analyzer**                 | Detects unintended public or cross-account access. |
| **Use Service Control Policies (SCPs)**     | Enforce governance across AWS Organizations.       |

---

## 🧩 AWS Key Management Service (KMS)

**AWS KMS** manages encryption keys used to protect data across AWS.

| **Feature**                     | **Description**                                |
| ------------------------------- | ---------------------------------------------- |
| **Customer Master Keys (CMKs)** | Used for encryption/decryption.                |
| **Automatic Key Rotation**      | Annually rotates CMKs automatically.           |
| **Integration**                 | Works with S3, EBS, RDS, DynamoDB, and Lambda. |
| **Audit**                       | All KMS API calls logged in CloudTrail.        |

**CLI Example:**

```bash
aws kms create-key --description "Key for S3 Encryption"
aws kms encrypt --key-id alias/mykey --plaintext fileb://data.txt --output text --query CiphertextBlob
```

---

## 🧪 Hands-On Lab 3: Enable S3 Server-Side Encryption (SSE-KMS)

### 🧰 **Objective**

Use KMS to encrypt data at rest in S3.

1. Create KMS Key:

   * Go to **KMS → Create Key → Symmetric Key**.
   * Alias: `s3-key`.
2. Go to **S3 → Bucket → Properties → Default Encryption**.
3. Choose **AWS KMS key → s3-key**.
4. Upload a file and confirm encryption icon.

✅ S3 objects are now encrypted automatically with your custom KMS key.

---

## 🧩 AWS Secrets Manager    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/16767f15-3c42-485c-9b64-03ff48166a81" />


**Secrets Manager** securely stores and rotates credentials, API keys, and database passwords.

### **Example: Store and Retrieve a Secret**

**CLI Commands:**

```bash
aws secretsmanager create-secret \
  --name MyDBPassword \
  --secret-string '{"username":"admin","password":"MySecurePass123"}'

aws secretsmanager get-secret-value \
  --secret-id MyDBPassword
```

✅ Secrets Manager integrates directly with **RDS**, **Lambda**, and **EC2** for automatic credential rotation.

---

## 🧩 AWS Security Hub and Inspector

| **Service**          | **Purpose**                                                                                   |
| -------------------- | --------------------------------------------------------------------------------------------- |
| **AWS Security Hub** | Aggregates and scores security findings from multiple services (GuardDuty, Inspector, Macie). |
| **Amazon Inspector** | Scans EC2 and ECR images for vulnerabilities.                                                 |
| **Amazon Macie**     | Detects sensitive data in S3 (PII, credentials, etc.).                                        |
| **GuardDuty**        | Detects threats based on behavior and VPC traffic analysis.                                   |

---

### **Example: Enable Security Hub**

```bash
aws securityhub enable-security-hub --region ap-southeast-2
aws securityhub get-findings --max-results 5
```

✅ Security Hub consolidates alerts from multiple AWS security tools into a unified dashboard.

---

## ⚙️ IAM Access Analyzer

Detects resources (S3 buckets, KMS keys, IAM roles) shared publicly or across accounts.

**Enable via Console:**

1. IAM → Access Analyzer → Create Analyzer → Region-wide.
2. Review findings for publicly accessible resources.

---

## 🧩 Enterprise Governance with AWS Organizations & SCPs

* **AWS Organizations** manages multiple AWS accounts centrally.
* **Service Control Policies (SCPs)** define global permissions boundaries.

**Example SCP (Restrict Region Usage):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {"aws:RequestedRegion": "ap-southeast-2"}
    }
  }]
}
```

✅ Prevents users from launching resources outside the Sydney region.

---

## 🧾 Key Takeaways

* **IAM** governs authentication and authorization across AWS.
* Use **roles** for temporary access and **policies** for permission control.
* Enforce **least privilege** and **MFA** to strengthen security posture.
* **KMS** provides centralized encryption key management.
* **Secrets Manager** and **Security Hub** enhance data and compliance security.
* Combine **IAM + KMS + SCPs** for enterprise-grade identity governance.

---

> 🪄 **Next Module Preview:**
> In **Module 14**, we’ll cover **AWS Networking and VPC Advanced Design** — exploring custom VPC architectures, NAT, peering, Transit Gateway, and hybrid connectivity (VPN + Direct Connect).

---
