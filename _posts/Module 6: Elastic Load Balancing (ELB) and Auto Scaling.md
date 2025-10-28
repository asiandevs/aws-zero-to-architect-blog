---

# ⚖️ Module 6 – Elastic Load Balancing (ELB) and Auto Scaling

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

* Understand **Elastic Load Balancing (ELB)** and its types.
* Configure and test **Application**, **Network**, and **Classic** Load Balancers.
* Integrate **Auto Scaling** with Load Balancers for fault tolerance.
* Design highly available, cost-optimized architectures using EC2 + ELB + ASG.

---

## ☁️ What Is Elastic Load Balancing?

**Elastic Load Balancing (ELB)** automatically distributes incoming traffic across multiple EC2 instances in one or more Availability Zones (AZs).
It improves **fault tolerance**, **scalability**, and **application performance** by ensuring no single instance becomes a bottleneck.

> 💡 **Analogy:**
> Think of ELB as a traffic controller at an airport — it directs incoming flights (user requests) to available runways (EC2 instances), ensuring smooth and balanced operations.

---

## 🧱 Types of Load Balancers

| **Type**                            | **Protocol Layer**    | **Use Case**                                            |
| ----------------------------------- | --------------------- | ------------------------------------------------------- |
| **Application Load Balancer (ALB)** | Layer 7 (HTTP/HTTPS)  | Web applications, microservices, container workloads.   |
| **Network Load Balancer (NLB)**     | Layer 4 (TCP/UDP/TLS) | High-performance, low-latency apps.                     |
| **Gateway Load Balancer (GWLB)**    | Layer 3 (Network)     | Security appliances, firewalls, deep packet inspection. |
| **Classic Load Balancer (CLB)**     | Layer 4 & 7 (legacy)  | Simple load balancing for EC2-Classic environments.     |

---

## 🧩 Key Benefits of ELB

| **Feature**                   | **Description**                                                           |
| ----------------------------- | ------------------------------------------------------------------------- |
| **Automatic Distribution**    | Balances load across healthy instances.                                   |
| **Health Checks**             | Detects and routes traffic only to healthy instances.                     |
| **Cross-Zone Load Balancing** | Distributes requests evenly across AZs.                                   |
| **SSL/TLS Termination**       | Offloads SSL decryption to the ELB.                                       |
| **Sticky Sessions**           | Routes users to the same backend instance.                                |
| **Integration**               | Works seamlessly with **Auto Scaling**, **Route 53**, and **CloudWatch**. |

---

## 🧪 Hands-On Lab 1: Configure Application Load Balancer (ALB)

### 🧰 **Objective**

Distribute HTTP traffic between two EC2 web servers in different AZs.

---

### **Step 1: Create Two EC2 Instances**

1. Launch two EC2 instances (Amazon Linux 2) in different subnets.
2. SSH into each and install a simple web page:

```bash
sudo su
yum install httpd -y
echo "Hello from Server 1 - $(hostname)" > /var/www/html/index.html
systemctl start httpd
systemctl enable httpd
```

Repeat for **Server 2** with a different message.

---

### **Step 2: Create Target Group**

1. Go to **EC2 → Target Groups → Create Target Group**.
2. Choose **Instances** → Name: `WebTG`.
3. Protocol: **HTTP**, Port: **80**.
4. Health Check Path: `/`.
5. Register both EC2 instances.
6. Click **Create Target Group**.

---

### **Step 3: Create Application Load Balancer**

1. Navigate to **EC2 → Load Balancers → Create Load Balancer**.
2. Choose **Application Load Balancer (ALB)**.
3. Name: `MyAppALB`.
4. Scheme: **Internet-facing**.
5. VPC: Select your custom VPC.
6. Select **2 subnets** (different AZs).
7. Add **Security Group** → Allow inbound HTTP (port 80).
8. Configure **Listener**:

   * Protocol: HTTP
   * Forward to: Target Group `WebTG`
9. Click **Create Load Balancer**.

---

### **Step 4: Test ALB**

Once created:

1. Copy the **DNS name** of your ALB (e.g., `myappalb-123456.ap-southeast-2.elb.amazonaws.com`).
2. Open it in your browser → refresh multiple times.

   * You’ll see alternating responses from Server 1 and Server 2.

✅ Congratulations! You’ve deployed a load-balanced web application across multiple Availability Zones.

---

## 🧠 Sticky Sessions (Session Affinity)

* Ensures a user always connects to the same backend instance.
* Useful for stateful applications (e.g., shopping carts).
* Enabled via **“Enable Stickiness”** under Target Group → Attributes.

**CLI Example:**

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --attributes Key=stickiness.enabled,Value=true
```

---

## 🧩 Cross-Zone Load Balancing

When enabled, ELB distributes traffic evenly across **all instances in all AZs** — even if one AZ has more instances.

> **Best Practice:** Enable cross-zone balancing to maximize fault tolerance and utilization.

---

## ⚡ Hands-On Lab 2: Configure Network Load Balancer (NLB)

### 🧰 **Objective**

Demonstrate load balancing at the **transport layer (Layer 4)**.

---

### **Step 1: Create a Target Group**

* Type: **Instances**, Protocol: **TCP**, Port: **80**.
* Register two EC2 web servers.

### **Step 2: Create a Network Load Balancer**

1. **Name:** `MyNLB`.
2. Scheme: Internet-facing.
3. Listeners: TCP on port 80 → Forward to Target Group.
4. Assign Elastic IPs for each subnet.
5. Click **Create**.

**CLI Example:**

```bash
aws elbv2 create-load-balancer --name MyNLB \
--type network --subnets subnet-1 subnet-2
```

---

### **Step 3: Test NLB**

Access the **DNS name** of the NLB and confirm traffic is distributed among your instances.
Network Load Balancers are ideal for low-latency, high-throughput workloads.

---

## 🧩 Gateway Load Balancer (GWLB) Overview

* Operates at **Layer 3 (Network Layer)**.
* Routes traffic to **security appliances** such as firewalls or intrusion detection systems.
* Common in **enterprise and hybrid** environments.

---

## 🧠 ELB Security and Monitoring

| **Feature**             | **Purpose**                                        |
| ----------------------- | -------------------------------------------------- |
| **SSL Termination**     | Decrypt HTTPS traffic at the load balancer.        |
| **Access Logs**         | Store connection logs in S3 for auditing.          |
| **CloudWatch Metrics**  | Monitor healthy hosts, request count, and latency. |
| **AWS WAF Integration** | Protects applications from common web exploits.    |

Enable these under **Load Balancer → Monitoring / Attributes**.

---

## ⚙️ Elastic Load Balancing Architecture

```
             ┌───────────────────────────┐
             │        Clients            │
             └────────────┬──────────────┘
                          │
                 ┌────────▼────────┐
                 │  Elastic Load   │
                 │   Balancer      │
                 └───────┬─────────┘
                 ┌───────┴─────────┐
                 │                 │
        ┌────────▼───────┐  ┌──────▼────────┐
        │  EC2 in AZ1    │  │  EC2 in AZ2   │
        └────────────────┘  └────────────────┘
```

---

# ⚙️ Auto Scaling

## 🧩 What Is Auto Scaling?

**Auto Scaling** automatically adjusts the number of EC2 instances based on load.
It ensures your application always has the right capacity at the right cost.

> 💡 **Analogy:**
> Auto Scaling is like cruise control in a car — it speeds up or slows down automatically based on the terrain (traffic demand).

---

## 🧱 Auto Scaling Components

| **Component**                | **Description**                                                 |
| ---------------------------- | --------------------------------------------------------------- |
| **Launch Template / Config** | Defines instance settings (AMI, type, security groups).         |
| **Auto Scaling Group (ASG)** | Manages a fleet of EC2 instances.                               |
| **Scaling Policy**           | Defines when to scale in/out (based on metrics).                |
| **Health Checks**            | Terminates unhealthy instances and replaces them automatically. |

---

## 🧪 Hands-On Lab 3: Create Auto Scaling with ALB

### **Step 1: Create a Launch Template**

1. Go to **EC2 → Launch Templates → Create Template**.
2. Name: `WebAppLT`.
3. AMI: Amazon Linux 2.
4. Instance type: `t2.micro`.
5. User Data:

   ```bash
   #!/bin/bash
   yum install -y httpd
   echo "Auto Scaling Web Server - $(hostname)" > /var/www/html/index.html
   systemctl start httpd
   systemctl enable httpd
   ```
6. Select Key Pair and Security Group → Create.

---

### **Step 2: Create Auto Scaling Group (ASG)**

1. Go to **Auto Scaling → Create Auto Scaling Group**.
2. Name: `WebAppASG`.
3. Select **Launch Template: WebAppLT**.
4. Choose VPC + Public Subnets (from Module 5).
5. Attach to **ALB Target Group (WebTG)**.
6. Set:

   * Desired capacity: 2
   * Minimum: 1
   * Maximum: 4
7. Enable health checks and notifications.
8. Click **Create Auto Scaling Group**.

---

### **Step 3: Configure Scaling Policies**

**Dynamic Scaling Example:**

* Metric: Average CPU Utilization
* Scale Out: If CPU > 70% for 5 minutes
* Scale In: If CPU < 20% for 10 minutes

**CLI Example:**

```bash
aws autoscaling put-scaling-policy \
  --policy-name cpu-scale-out \
  --auto-scaling-group-name WebAppASG \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity
```

---

### **Step 4: Test Auto Scaling**

1. Use a stress test tool to increase CPU load:

   ```bash
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   stress --cpu 2 --timeout 300
   ```
2. Observe Auto Scaling Group → Instances → scaling out.
3. After load drops → scaling in.

✅ Elastic Load Balancing + Auto Scaling ensures **fault-tolerant**, **highly available**, and **cost-efficient** applications.

---

## 🧾 Key Takeaways

* **ELB** distributes traffic intelligently across instances.
* **ALB** handles HTTP/HTTPS (Layer 7), **NLB** handles TCP/UDP (Layer 4).
* Use **Cross-Zone Load Balancing** and **Health Checks** for resilience.
* **Auto Scaling Groups (ASG)** dynamically manage EC2 capacity.
* Combined, ELB + ASG provides a **self-healing, scalable web tier**.

---

> 🪄 **Next Module Preview:**
> In **Module 7**, we’ll explore **Route 53 (DNS and Traffic Routing)** — learning how to register domains, configure routing policies (simple, weighted, latency-based), and integrate DNS with ALB and CloudFront.

---
