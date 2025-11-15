# ──────────────────────────────────────────────────────────────
# MODULE 9
# File: _posts/2025-11-06-module9-aws-lambda-and-api-gateway-serverless-architecture.md
# ──────────────────────────────────────────────────────────────
---
title: "Module 9: AWS Lambda & API Gateway (Serverless)"
date: 2025-11-06
categories: aws lambda serverless
tags: lambda api-gateway event-driven
image: /assets/images/lambda.jpg
description: "Build serverless APIs and event-driven apps."
---

#  Module 9 – AWS Lambda and API Gateway (Serverless Architecture) ⚙️

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand the **Serverless Computing Model** and its advantages.
* Create and manage **AWS Lambda** functions.
* Integrate Lambda with **API Gateway** to build RESTful APIs.
* Configure **IAM roles, permissions, and environment variables**.
* Monitor, scale, and version Lambda functions effectively.

---

## ☁️ What Is Serverless Computing?

**Serverless Computing** means you focus on your code — not on provisioning or managing servers.
AWS handles scaling, fault tolerance, and infrastructure automatically.

> 💡 **Analogy:**
> Think of **Serverless** like **ride-sharing apps** — you don’t own or manage a car (server), you just request a ride (function execution) when you need it, and you pay only for the distance traveled (execution time).

---

## 🧩 What Is AWS Lambda?    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/2b5ee651-d23f-440d-820d-9655feeecf39" />


**AWS Lambda** is a **compute service** that runs your code in response to events and automatically manages compute resources.
You upload your code, define triggers, and Lambda executes it whenever needed.

| **Feature**                | **Description**                                                 |
| -------------------------- | --------------------------------------------------------------- |
| **Event-driven**           | Runs in response to triggers (e.g., API Gateway, S3, DynamoDB). |
| **Automatic scaling**      | Handles any number of concurrent executions.                    |
| **Pay per execution**      | You’re billed per request and compute time.                     |
| **Multi-language support** | Supports Python, Node.js, Java, Go, C#, and more.               |
| **Stateless**              | Each invocation is independent and short-lived.                 |

---

## 🧠 Lambda Architecture Overview

```
          ┌───────────────┐
          │  Event Source │  (S3, API Gateway, SNS, CloudWatch)
          └───────┬───────┘
                  │
          ┌───────▼───────┐
          │   AWS Lambda  │
          │   (Your Code) │
          └───────┬───────┘
                  │
         ┌────────▼────────┐
         │   Target (e.g.  │
         │ DynamoDB, S3,   │
         │ SNS, etc.)      │
         └─────────────────┘
```

---

## 🧪 Hands-On Lab 1: Create and Test Your First Lambda Function

### 🧰 **Objective**

Create a simple Lambda function that returns a message when invoked.

---

### **Step 1: Create the Lambda Function**

1. Go to **AWS Console → Lambda → Create function**.
2. Choose **Author from scratch**.

   * Function name: `HelloLambda`
   * Runtime: **Python 3.12**
   * Role: Create a new role with basic Lambda permissions.
3. Click **Create Function**.

---

### **Step 2: Add Code**

In the code editor, replace default code with:

```python
import json

def lambda_handler(event, context):
    message = "Hello from AWS Lambda!"
    print(message)
    return {
        'statusCode': 200,
        'body': json.dumps({'message': message})
    }
```

Click **Deploy**.

---

### **Step 3: Test Function**

1. Click **Test → Create new test event → HelloEvent**.
2. Keep the default JSON:

   ```json
   {"name": "Cloud Architect"}
   ```
3. Click **Test**.

✅ Output:

```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello from AWS Lambda!\"}"
}
```

---

### **Step 4: CLI Deployment**

You can deploy the same function using AWS CLI:

```bash
zip function.zip lambda_function.py
aws lambda create-function \
  --function-name HelloLambdaCLI \
  --runtime python3.12 \
  --role arn:aws:iam::<account-id>:role/service-role/MyLambdaRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

Invoke it:

```bash
aws lambda invoke --function-name HelloLambdaCLI response.json
cat response.json
```

---

## 🧩 Lambda Permissions and Roles

Lambda functions need **execution roles** to access other AWS services.

**Example Policy (JSON):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["logs:*"],
    "Resource": "*"
  }]
}
```

Attach this to the **Lambda execution role** via IAM.

---

## ⚙️ Environment Variables and Versions

* Use **Environment Variables** to pass configuration data (e.g., DB connection strings).
* Lambda supports **versioning** — each deployment can be versioned and aliased.

**CLI Example:**

```bash
aws lambda publish-version --function-name HelloLambda
aws lambda create-alias \
  --function-name HelloLambda \
  --name PROD \
  --function-version 1
```

---

## 🧠 Common Lambda Triggers

| **Trigger Source**    | **Event Example**                    |
| --------------------- | ------------------------------------ |
| **API Gateway**       | REST API request                     |
| **S3**                | Object upload (`s3:ObjectCreated:*`) |
| **DynamoDB Streams**  | Table updates                        |
| **SNS/SQS**           | Message received                     |
| **CloudWatch Events** | Scheduled execution (cron jobs)      |
| **EventBridge**       | Complex event routing                |

---

## 🧪 Hands-On Lab 2: Build a REST API with Lambda and API Gateway

### 🧰 **Objective**

Create a **serverless REST API** using API Gateway as the front-end and Lambda as the compute backend.

---

### **Step 1: Create a New Lambda Function**

1. Function name: `APILambdaHandler`.
2. Runtime: **Python 3.12**.
3. Code:

```python
import json

def lambda_handler(event, context):
    name = event.get("queryStringParameters", {}).get("name", "Guest")
    return {
        'statusCode': 200,
        'body': json.dumps({'message': f'Hello, {name}! Welcome to Serverless API.'})
    }
```

Deploy it.

---

### **Step 2: Create API Gateway**    <img width="50" height="50" alt="image" src="https://github.com/user-attachments/assets/d287324b-b4ef-456a-aa1c-402e1c251f59" />


1. Go to **Amazon API Gateway → Create API → REST API → Build**.
2. Name: `HelloAPI`.
3. Click **Create Resource → /hello**.
4. Under `/hello`, choose **Create Method → GET**.
5. Integration type: **Lambda Function** → Select `APILambdaHandler`.
6. Click **Save → OK** to grant permission.

---

### **Step 3: Deploy API**

1. Click **Actions → Deploy API**.
2. Create a new stage: `dev`.
3. Copy the **Invoke URL**, e.g.:

   ```
   https://abcd1234.execute-api.ap-southeast-2.amazonaws.com/dev/hello?name=Monowar
   ```

✅ Response:

```json
{
  "message": "Hello, Monowar! Welcome to Serverless API."
}
```

---

### **Step 4: Test with CLI**

```bash
curl "https://abcd1234.execute-api.ap-southeast-2.amazonaws.com/dev/hello?name=CloudArchitect"
```

---

## 🧠 API Gateway Concepts

| **Component**            | **Purpose**                                          |
| ------------------------ | ---------------------------------------------------- |
| **Resource**             | Represents an endpoint (e.g., `/hello`).             |
| **Method**               | HTTP method (GET, POST, etc.).                       |
| **Integration Type**     | Backend to invoke (Lambda, HTTP, Mock, AWS Service). |
| **Stage**                | Deployment environment (dev, prod).                  |
| **API Key / Usage Plan** | Controls access and rate limits.                     |

---

## 🧩 Hands-On Lab 3: Add Security with IAM and API Keys

1. In API Gateway, go to **API Keys → Create API Key**.
2. Name: `DemoKey`.
3. Create **Usage Plan** and attach your API and key.
4. Under **Method Request**, set:

   * Authorization: **API Key Required = true**
5. Deploy API again.

Test using:

```bash
curl -H "x-api-key: <DemoKey>" \
"https://abcd1234.execute-api.ap-southeast-2.amazonaws.com/dev/hello?name=SecureUser"
```

✅ You’ve now secured your Lambda-backed API.

---

## ⚙️ Monitoring and Logging

* Lambda automatically integrates with **Amazon CloudWatch** for:

  * Execution logs
  * Errors
  * Invocation counts
  * Duration metrics

**CLI Example:**

```bash
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/
```

> 💡 Use **CloudWatch Insights** to analyze function performance and optimize cost.

---

## ⚡ Scaling Behavior

* Lambda scales **automatically per request**.
* Each function has a concurrency limit (default 1000 per region).
* You can set **reserved concurrency** to control scaling per function.

**CLI Example:**

```bash
aws lambda put-function-concurrency \
  --function-name HelloLambda \
  --reserved-concurrent-executions 50
```

---

## 🧾 Key Takeaways

* **AWS Lambda** allows you to run code without provisioning servers.
* Integrate with **API Gateway** for fully serverless APIs.
* Use **IAM roles** and **environment variables** for security and flexibility.
* **CloudWatch** provides observability and alerting.
* Lambda is ideal for **microservices, event processing, and automation**.
* Combine with **S3, DynamoDB, and SNS** for complete event-driven workflows.

---

> 🪄 **Next Module Preview:**
> In **Module 10**, we’ll explore **Amazon S3, CloudFront, and Global Content Delivery** — learning how to build a secure, distributed, and highly available static website architecture with caching and HTTPS.

---
