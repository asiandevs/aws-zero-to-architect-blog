---
title: "Module 18 – AWS Security, Compliance and Governance Frameworks"
date: 2025-11-14
categories: aws security compliance governance
tags: aws security-hub guardduty config compliance frameworks
image: /assets/images/migration.jpg
description: "AWS security frameworks, compliance standards, governance tools, and security services implementation."
---

> 💡 **Analogy:**
> AWS builds and secures the apartment building. You're responsible for locking your own door.

---

## 🧩 Security Layers in AWS

| **Layer**       | **Mechanism**                  | **Example Services**                |
| --------------- | ------------------------------ | ----------------------------------- |
| **Identity**    | Authentication & Authorization | IAM, SSO, Cognito                   |
| **Network**     | Isolation, segmentation        | VPC, Security Groups, NACLs         |
| **Data**        | Encryption & lifecycle         | KMS, S3 encryption, Secrets Manager |
| **Application** | Logging & threat detection     | WAF, Shield, Inspector              |
| **Monitoring**  | Continuous visibility          | CloudWatch, GuardDuty, Config       |

---

## 🧠 AWS Compliance Frameworks

AWS aligns with numerous global standards to simplify certification and compliance.

| **Framework**                 | **Focus Area**                       |
| ----------------------------- | ------------------------------------ |
| **ISO 27001 / 27017 / 27018** | Information security & cloud privacy |
| **SOC 1 / SOC 2 / SOC 3**     | Audit and control assurance          |
| **PCI-DSS**                   | Payment data protection              |
| **HIPAA**                     | Healthcare data privacy              |
| **CIS Benchmarks**            | Security best practices              |
| **NIST 800-53 / 800-171**     | US federal & defense security        |
| **IRAP / ASD**                | Australian Government compliance     |

> 🔐 AWS Artifact provides downloadable compliance reports and certifications for audit teams.

---

## 🧩 AWS Governance and Control Tools

| **Tool**                            | **Purpose**                              |
| ----------------------------------- | ---------------------------------------- |
| **AWS Organizations**               | Centralized account management & billing |
| **Service Control Policies (SCPs)** | Enforce permission boundaries            |
| **AWS Control Tower**               | Automated landing zone setup             |
| **AWS Config**                      | Continuous compliance & change tracking  |
| **Security Hub**                    | Aggregated compliance findings           |
| **CloudTrail**                      | API-level activity auditing              |

---

## 🧪 Hands-On Lab 1: Set Up a Secure Multi-Account Governance Model

### 🧰 Objective

Use AWS Control Tower and Organizations to enforce governance across accounts.

---

### Step 1 – Enable AWS Organizations

1. Navigate to **AWS Organizations → Create Organization**.
2. Choose **All Features** (for SCP and tag policies).
3. Create Organizational Units (OUs):

   * `Production`
   * `Development`
   * `Sandbox`

---

### Step 2 – Apply Service Control Policy (SCP)

**Example SCP (Restrict Region Use):**

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

✅ This prevents resources from being launched outside Sydney (compliance with data residency).

---

### Step 3 – Set Up AWS Control Tower

1. Go to **AWS Control Tower → Set up landing zone**.
2. Define regions, guardrails, and baseline configurations.
3. Control Tower automatically:

   * Creates **Audit** and **Log Archive** accounts.
   * Applies **guardrails** (preventive + detective).
   * Centralizes logging and security baselines.

✅ Your AWS environment now has consistent governance with best-practice controls.

---

## 🧩 AWS Security Hub Overview

**AWS Security Hub** aggregates, prioritizes, and standardizes findings from multiple AWS security tools.

| **Integrated Source**   | **Purpose**                                  |
| ----------------------- | -------------------------------------------- |
| **GuardDuty**           | Threat detection (malicious IPs, anomalies). |
| **Inspector**           | Vulnerability scanning (EC2, ECR).           |
| **Macie**               | Sensitive data detection in S3.              |
| **IAM Access Analyzer** | Unintended access findings.                  |

---

## 🧪 Hands-On Lab 2: Enable AWS Security Hub and GuardDuty

### 🧰 Objective

Detect threats and compliance gaps in real-time.

---

### Step 1 – Enable GuardDuty

```bash
aws guardduty create-detector --enable
```

GuardDuty starts analyzing CloudTrail logs, VPC Flow Logs, and DNS queries.

---

### Step 2 – Enable Security Hub

```bash
aws securityhub enable-security-hub
```

### Step 3 – Enable Standards

In the console → **Security Hub → Standards → Enable**:

* CIS AWS Foundations Benchmark
* AWS Foundational Security Best Practices

✅ Security Hub begins aggregating findings from GuardDuty, Config, and Inspector.

---

### Step 4 – View Findings

```bash
aws securityhub get-findings --max-results 10
```

You'll see prioritized findings with severity scores.

---

## 🧩 AWS WAF and Shield

| **Service**                            | **Purpose**                                             |
| -------------------------------------- | ------------------------------------------------------- |
| **AWS WAF (Web Application Firewall)** | Protects web apps from common exploits (SQLi, XSS).     |
| **AWS Shield Standard**                | Automatic DDoS protection (free for all AWS customers). |
| **AWS Shield Advanced**                | Enhanced DDoS mitigation and 24x7 AWS SOC support.      |

---

### Example WAF Setup (Web ACL)

1. Go to **WAF → Web ACLs → Create Web ACL**.
2. Add managed rule groups:

   * AWS Managed Rules for SQL Injection
   * Anonymous IP list blocking
3. Associate with an **Application Load Balancer**.

✅ Your web tier is now protected against OWASP Top 10 threats.

---

## 🧩 AWS Config – Continuous Compliance

**AWS Config** continuously records resource configuration changes and validates them against compliance rules.

### Example Managed Rule:

* `restricted-ssh`: Ensure security groups don't allow 0.0.0.0/0 on port 22.

**CLI Example:**

```bash
aws configservice put-config-rule \
  --config-rule-name restricted-ssh \
  --source "Owner=AWS,SourceIdentifier=INCOMING_SSH_DISABLED"
```

✅ Non-compliant resources trigger alerts and can automatically remediate via Systems Manager.

---

## 🧩 Data Protection and Encryption

| **Service**         | **Purpose**                                  |
| ------------------- | -------------------------------------------- |
| **KMS**             | Centralized key management & encryption.     |
| **Secrets Manager** | Secure storage of credentials and API keys.  |
| **S3 Encryption**   | SSE-S3, SSE-KMS, and client-side encryption. |
| **EBS Encryption**  | Transparent disk-level encryption.           |
| **RDS Encryption**  | Data at rest and in-transit encryption.      |

### Example: Encrypt an S3 Bucket Using KMS

```bash
aws s3api put-bucket-encryption \
  --bucket my-secure-bucket \
  --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms","KMSMasterKeyID":"alias/mykey"}}]}'
```

✅ All objects uploaded are automatically encrypted with your KMS key.

---

## 🧩 Compliance and Audit Automation

| **Goal**                   | **Service**         | **Description**                                           |
| -------------------------- | ------------------- | --------------------------------------------------------- |
| **Centralized Compliance** | Security Hub        | Aggregates findings from multiple tools.                  |
| **Policy Enforcement**     | Config + SSM        | Automated remediation.                                    |
| **Governance Reporting**   | AWS Audit Manager   | Continuous evidence collection for compliance frameworks. |
| **Change Tracking**        | CloudTrail + Config | Tracks who changed what and when.                         |

---

### Example: Generate CIS Compliance Report

```bash
aws auditmanager create-assessment \
  --name "CIS AWS Foundations Audit" \
  --framework-id <framework-id> \
  --role-arn arn:aws:iam::123456789012:role/AuditManagerRole
```

✅ Simplifies compliance audits by automatically collecting CloudTrail evidence.

---

## 🧩 Security Operations Center (SOC) Integration

Combine AWS-native tools for full lifecycle security monitoring:

```
┌──────────────────────┐
│ CloudTrail / Config  │   → Activity & Configuration
├──────────────────────┤
│ GuardDuty / WAF      │   → Threat Detection
├──────────────────────┤
│ Security Hub         │   → Aggregated Findings
├──────────────────────┤
│ EventBridge + SNS    │   → Alerts & Response
├──────────────────────┤
│ Lambda / SSM         │   → Automated Remediation
└──────────────────────┘
```

> ⚙️ Example: Use **EventBridge + Lambda** to quarantine EC2 instances flagged by GuardDuty.

---

## 🧠 Best Practices for Security & Governance

* Apply **least privilege** across IAM and roles.
* Use **multi-account strategy** for workload isolation.
* Enable **MFA and CloudTrail** across all accounts.
* Continuously monitor **compliance posture** via Config and Security Hub.
* Encrypt all data **in-transit and at rest**.
* Integrate **GuardDuty + WAF + Shield** for multi-layered protection.
* Regularly review **CIS and NIST** benchmarks.

---

## 🧾 Key Takeaways

* AWS uses a **shared responsibility model**: AWS secures infrastructure; you secure workloads.
* Use **Organizations**, **SCPs**, and **Control Tower** for multi-account governance.
* **Security Hub**, **GuardDuty**, and **Config** enable continuous compliance.
* Apply **encryption and access control** via KMS and IAM.
* Automate compliance with **Audit Manager** and **SSM Remediation**.
* Follow **defense-in-depth** — multiple layers of security ensure resilience.

---

> 🪄 **Next Module Preview:**
> In **Module 19**, we'll explore **AWS Data Analytics and Big Data Architecture** — covering Redshift, Glue, Athena, and data lake design patterns for enterprise-scale analytics.

