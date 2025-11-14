---
layout: single
title: "Search Modules"
permalink: /search/
search: true
---

<div class="search-container" style="max-width: 800px; margin: 40px auto; padding: 20px; font-family: 'Poppins', sans-serif;">
  <h1 style="text-align: center; color: #ff6600; margin-bottom: 30px;">Search AWS Training Modules</h1>
  
  <input 
    type="text" 
    id="search-input" 
    placeholder="Type to search modules (e.g. EC2, S3, Lambda)..." 
    autocomplete="off"
    style="
      width: 100%;
      padding: 16px 20px;
      font-size: 1.1em;
      border: 2px solid #ddd;
      border-radius: 12px;
      outline: none;
      transition: all 0.3s;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
    "
    onfocus="this.style.borderColor='#ff6600'; this.style.boxShadow='0 6px 20px rgba(255,102,0,0.15)';"
    onblur="this.style.borderColor='#ddd'; this.style.boxShadow='0 4px 12px rgba(0,0,0,0.05)';"
  />

  <div id="search-results" style="margin-top: 30px;"></div>
</div>

<!-- Search Data -->
<script>
window.searchData = [
  { "title": "Introduction to Cloud Computing", "url": "/_posts/2025-10-28-module1-introduction-to-cloud-computing.md", "desc": "Virtualization, Cloud models, Deployment types, AWS overview." },
  { "title": "Getting Started with AWS & EC2", "url": "/_posts/2025-10-29-module2-getting-started-with-aws-and-ec2.md", "desc": "EC2 instances, AMIs, EBS, and key pair management." },
  { "title": "Elastic Block Store (EBS) & Elastic File System (EFS)", "url": "/_posts/2025-10-30-module3-elastic-block-store-ebs-and-elastic-file-system-efs.md", "desc": "Persistent storage, snapshots, Multi-Attach, lifecycle." },
  { "title": "Amazon S3: Simple Storage Service", "url": "/_posts/2025-10-30-module4-amazon-s3-simple-storage-service.md", "desc": "Object storage, versioning, lifecycle, replication." },
  { "title": "Networking with Amazon VPC", "url": "/_posts/2025-11-02-module5-networking-with-amazon-vpc.md", "desc": "Custom VPCs, subnets, gateways, and routing." },
  { "title": "Elastic Load Balancing & Auto Scaling", "url": "/_posts/2025-11-03-module6-elastic-load-balancing-and-auto-scaling.md", "desc": "Load balancing types, scaling policies, and health checks." },
  { "title": "Amazon Route 53 (DNS & Routing)", "url": "/_posts/2025-11-04-module7-amazon-route53-dns-and-traffic-routing.md", "desc": "DNS routing, hosted zones, and failover strategies." },
  { "title": "AWS CloudFormation (Infrastructure as Code)", "url": "/_posts/2025-11-05-module8-aws-cloudformation-infrastructure-as-code.md", "desc": "IaC templates, stacks, and automation." },
  { "title": "AWS Lambda & API Gateway (Serverless)", "url": "/_posts/2025-11-06-module9-aws-lambda-and-api-gateway-serverless-architecture.md", "desc": "Build serverless APIs and event-driven apps." },
  { "title": "Amazon S3 & CloudFront (Global Delivery)", "url": "/_posts/2025-11-07-module10-amazon-s3-and-cloudfront-global-content-delivery.md", "desc": "Static hosting, CDN distribution, HTTPS setup." },
  { "title": "AWS RDS & DynamoDB (Relational vs NoSQL)", "url": "/_posts/2025-11-08-module11-aws-rds-and-dynamodb-relational-vs-nosql.md", "desc": "Database architecture, scaling, replication." },
  { "title": "Monitoring & Logging (CloudWatch, CloudTrail, Config)", "url": "/_posts/2025-11-09-module12-aws-monitoring-and-logging-cloudwatch-cloudtrail-config.md", "desc": "Observability, audit logs, and compliance tools." },
  { "title": "Identity & Access Management (IAM)", "url": "/_posts/2025-11-10-module13-identity-and-access-management-iam.md", "desc": "Users, roles, policies, and MFA." },
  { "title": "AWS Security & Encryption Services", "url": "/_posts/2025-11-11-module14-aws-security-and-encryption-services.md", "desc": "Encryption, KMS, Secrets Manager, GuardDuty." },
  { "title": "Automation & DevOps Tools on AWS", "url": "/_posts/2025-11-12-module15-automation-and-devops-tools-on-aws.md", "desc": "CI/CD pipelines, CodeCommit, CodeBuild, CodeDeploy." },
  { "title": "AWS Cost Management & Billing", "url": "/_posts/2025-11-13-module16-aws-cost-management-and-billing.md", "desc": "Budgets, Cost Explorer, and cost optimization." },
  { "title": "High Availability and Fault Tolerance Architecture in AWS", "url": "/_posts/2025-11-14-module17-high-availability-and-fault-tolerance-architecture-in-aws.md", "desc": "Backup automation, DR design, and recovery points." },
  { "title": "AWS Security Compliance and Governance Frameworks", "url": "/_posts/2025-11-15-module18-aws-security-compliance-and-governance-frameworks.md", "desc": "Security, Compliance, and Governance." },
  { "title": "AWS Well-Architected Framework", "url": "/_posts/2025-11-16-module19-aws-well-architected-framework.md", "desc": "Pillars of architecture excellence and governance." },
  { "title": "Project & Review", "url": "/_posts/2025-11-17-module20-project-and-review.md", "desc": "Final project integrating all modules." }
];
</script>

<!-- Search Logic -->
<script>
document.getElementById('search-input').addEventListener('input', function(e) {
  const query = e.target.value.toLowerCase().trim();
  const resultsDiv = document.getElementById('search-results');
  
  if (!query) {
    resultsDiv.innerHTML = '<p style="text-align:center; color:#777; font-style:italic;">Start typing to search modules...</p>';
    return;
  }

  const matches = window.searchData.filter(item => 
    item.title.toLowerCase().includes(query) || 
    item.desc.toLowerCase().includes(query)
  );

  if (matches.length === 0) {
    resultsDiv.innerHTML = '<p style="text-align:center; color:#e74c3c;">No modules found. Try "S3", "EC2", or "Lambda".</p>';
    return;
  }

  resultsDiv.innerHTML = matches.map(item => `
    <a href="${item.url}" style="
      display: block;
      padding: 16px;
      margin: 12px 0;
      background: white;
      border-radius: 12px;
      text-decoration: none;
      color: #333;
      box-shadow: 0 4px 12px rgba(0,0,0,0.08);
      transition: all 0.2s;
      border-left: 4px solid #ff6600;
    " onmouseover="this.style.transform='translateY(-2px)'; this.style.boxShadow='0 8px 20px rgba(0,0,0,0.15)';"
       onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='0 4px 12px rgba(0,0,0,0.08)';">
      <div style="font-weight: 700; color: #ff6600; margin-bottom: 6px;">${item.title.replace(
        new RegExp(query, 'gi'),
        match => `<span style="background:#ffebcc; padding:0 4px; border-radius:4px;">${match}</span>`
      )}</div>
      <div style="font-size: 0.95em; color: #777;">${item.desc}</div>
    </a>
  `).join('');
});
</script>

<style>
#search-input::placeholder { color: #aaa; }
#search-results a { font-family: 'Poppins', sans-serif; }
</style>
