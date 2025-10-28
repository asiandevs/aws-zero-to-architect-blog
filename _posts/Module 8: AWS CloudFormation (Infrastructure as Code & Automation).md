---

# 🧩 Module 8 – AWS CloudFormation (Infrastructure as Code & Automation)

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Explain the concept of **Infrastructure as Code (IaC)** and CloudFormation’s role in AWS.
* Create and manage **CloudFormation stacks** using templates.
* Understand **CloudFormation template structure** (YAML/JSON).
* Deploy a multi-tier architecture automatically (VPC, EC2, ELB).
* Use **parameters, mappings, conditions, and outputs** in templates.
* Manage stack updates, rollbacks, and deletions safely.

---

## ☁️ What Is AWS CloudFormation?    <img width="50" height="55" alt="image" src="https://github.com/user-attachments/assets/e1febbf5-0ae6-4a4f-8898-231c89bf020c" />


**AWS CloudFormation** is an Infrastructure as Code (IaC) service that lets you **model, provision, and manage AWS resources** using **templates** written in **YAML or JSON**.

Instead of manually creating resources in the console, CloudFormation lets you:

* **Automate deployments**
* **Maintain consistency**
* **Version-control infrastructure**
* **Easily replicate environments**

> 💡 **Analogy:**
> Think of CloudFormation as a **blueprint** for your cloud environment.
> Once defined, AWS can repeatedly build identical “houses” (environments) anywhere, anytime.

---

## 🧱 CloudFormation Core Concepts

| **Concept**         | **Description**                                         |
| ------------------- | ------------------------------------------------------- |
| **Template**        | The blueprint file (YAML/JSON) defining AWS resources.  |
| **Stack**           | A deployed collection of resources based on a template. |
| **Change Set**      | A preview of changes before applying updates.           |
| **Drift Detection** | Identifies manual changes outside CloudFormation.       |
| **Nested Stack**    | Stack inside another stack for modularity.              |
| **StackSet**        | Deploy stacks across multiple accounts or regions.      |

---

## 🧩 Benefits of CloudFormation

| **Feature**                 | **Advantage**                                          |
| --------------------------- | ------------------------------------------------------ |
| **Automation**              | Deploy hundreds of resources with one command.         |
| **Consistency**             | Avoid human error — identical environments every time. |
| **Version Control**         | Templates can be stored in Git or S3.                  |
| **Rollback Safety**         | Automatically reverts on failure.                      |
| **Dependency Management**   | Handles resource creation order automatically.         |
| **Cross-Region Deployment** | Use StackSets to roll out infrastructure globally.     |

---

## 🧠 CloudFormation Template Structure

A CloudFormation template is divided into **six major sections**:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "My CloudFormation Template Example"

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: !Ref InstanceType

Outputs:
  InstanceID:
    Value: !Ref MyEC2Instance
    Description: "EC2 Instance ID"
```

| **Section**                | **Purpose**                              |
| -------------------------- | ---------------------------------------- |
| `AWSTemplateFormatVersion` | Template version (always `2010-09-09`).  |
| `Description`              | Brief purpose of the template.           |
| `Parameters`               | Dynamic inputs for the stack.            |
| `Mappings`                 | Static key-value pairs.                  |
| `Conditions`               | Conditional logic for resource creation. |
| `Resources`                | The main section defining AWS resources. |
| `Outputs`                  | Values exported after stack creation.    |

---

## 🧪 Hands-On Lab 1: Create EC2 with CloudFormation

### **Objective**

Use CloudFormation to automatically create an EC2 instance with a web server.

---

### **Step 1: Create a YAML Template**

Save the following as `ec2-webserver.yaml`:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Launch EC2 Instance with Web Server"

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "Name of an existing EC2 KeyPair"

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      SecurityGroups: [!Ref WebServerSG]
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum install -y httpd
          echo "Hello from CloudFormation Web Server - $(hostname)" > /var/www/html/index.html
          systemctl start httpd
          systemctl enable httpd

  WebServerSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Allow HTTP and SSH access"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceID:
    Description: "EC2 Instance ID"
    Value: !Ref MyInstance
```

---

### **Step 2: Deploy the Stack**

1. Go to **AWS Console → CloudFormation → Create Stack**.
2. Upload `ec2-webserver.yaml`.
3. Enter parameters (e.g., KeyPair name).
4. Click **Next → Next → Create Stack**.

CloudFormation will now automatically:

* Create the security group.
* Launch an EC2 instance.
* Run the user data script to install Apache.

---

### **Step 3: Verify Deployment**

1. Go to **EC2 → Instances**.
2. Copy the **Public IP** and open in your browser.

   ```
   http://<public-ip>
   ```

✅ You should see:
**“Hello from CloudFormation Web Server - ip-10-0-x-x”**

---

## 🧩 Hands-On Lab 2: Multi-Tier Web Architecture (VPC + EC2 + ALB)

### **Objective**

Deploy a **VPC**, **Subnets**, **Security Groups**, **EC2 Instances**, and an **Application Load Balancer** — all through one CloudFormation template.

---

### **Step 1: Template Overview**

The following structure automates a 2-tier setup:

```
VPC
 ├── Public Subnet
 │     └── Application Load Balancer
 └── Private Subnet
       └── EC2 Web Servers
```

### **Step 2: Key Resource Definitions (Simplified)**

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref MyVPC

  WebServerSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Allow HTTP traffic"
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: t2.micro
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds: [!Ref WebServerSG]
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum install -y httpd
          echo "WebApp via CloudFormation - $(hostname)" > /var/www/html/index.html
          systemctl start httpd
```

Save the template as `vpc-webapp.yaml`.

---

### **Step 3: Deploy Stack**

```bash
aws cloudformation create-stack \
  --stack-name WebAppStack \
  --template-body file://vpc-webapp.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=MyKeyPair
```

Monitor progress:

```bash
aws cloudformation describe-stacks --stack-name WebAppStack
```

Once complete:

* Access the **EC2 public IP or ALB DNS name**.
* You’ll see “WebApp via CloudFormation” displayed.

---

## 🧠 CloudFormation Advanced Concepts

| **Feature**     | **Description**                                           |
| --------------- | --------------------------------------------------------- |
| **Parameters**  | Customize stack inputs (instance type, CIDR range, etc.). |
| **Mappings**    | Static lookups (e.g., Region → AMI ID).                   |
| **Conditions**  | Deploy resources only under specific circumstances.       |
| **Outputs**     | Export values for cross-stack references.                 |
| **Change Sets** | Preview infrastructure updates before applying.           |

---

## ⚙️ Stack Management

| **Action**   | **Command**                          |
| ------------ | ------------------------------------ |
| Create Stack | `aws cloudformation create-stack`    |
| Update Stack | `aws cloudformation update-stack`    |
| Delete Stack | `aws cloudformation delete-stack`    |
| View Outputs | `aws cloudformation describe-stacks` |

---

## ⚠️ Rollback and Drift Detection

* If a stack creation or update fails, **CloudFormation automatically rolls back** changes.
* **Drift Detection** lets you compare stack configuration with actual resource states.

**CLI Example:**

```bash
aws cloudformation detect-stack-drift --stack-name WebAppStack
aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id abc123
```

---

## 🧾 Key Takeaways

* **CloudFormation** automates infrastructure provisioning using **templates**.
* Templates can define **entire environments**, not just single resources.
* **Stacks** are deployed and managed as single entities.
* Use **Parameters, Mappings, Conditions, Outputs** for modular and flexible templates.
* **StackSets** allow **multi-account** and **multi-region** automation.
* Always test updates with **Change Sets** to avoid accidental disruptions.

---

> 🪄 **Next Module Preview:**
> In **Module 9**, we’ll explore **AWS Lambda and API Gateway**, building your first **serverless application** with live API triggers, integrations, and monitoring.

---
