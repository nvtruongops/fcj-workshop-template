---
title: "Week 4 Worklog"
date: 2025-09-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Learn and practice **Serverless Automation** using **AWS Lambda** to optimize EC2 operation costs.  
* Implement **Advanced Monitoring** with **Amazon CloudWatch** and **Grafana** for system observability.  
* Explore **CloudWatch Metrics, Logs, Alarms, and Dashboards** for real-time analytics.  
* Learn to manage and control access through **IAM Policies** and **Resource Tags**.  
* Understand how to use **AWS Systems Manager** for patching, remote commands, and centralized administration.  
* Implement **Infrastructure as Code (IaC)** using **CloudFormation** and **AWS CDK**.  
* Practice building advanced, multi-service architectures and nested stacks with **AWS CDK Advanced**.  

---

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ----------- | ---------------- | ------------------ |
| 2 | - **Serverless Automation with AWS Lambda** <br>&emsp; + Automate EC2 instance scheduling using Lambda functions. <br>&emsp; + Configure IAM Role and tags for automation. <br>&emsp; + Test and validate automatic start/stop of EC2 instances. <br><br> - **Advanced Monitoring with CloudWatch and Grafana** <br>&emsp; + Install Grafana on EC2 instance. <br>&emsp; + Connect Grafana to CloudWatch. <br>&emsp; + Create dashboards and configure alerting rules. | 30/09/2025 | 30/09/2025 |  <br> https://000022.awsstudygroup.com/ <br> https://000029.awsstudygroup.com/ |
| 3 | - **CloudWatch Advanced Workshop** <br>&emsp; + Explore CloudWatch Metrics, Logs, Alarms, and Dashboards. <br>&emsp; + Use Container Insights for ECS, EKS, and Fargate monitoring. <br><br> - **Resource Organization with Tags and Resource Groups** <br>&emsp; + Create and manage resource tags across EC2, S3, and IAM. <br>&emsp; + Build tag-based Resource Groups for centralized management. | 01/10/2025 | 01/10/2025 | https://000036.awsstudygroup.com/ <br> https://000027.awsstudygroup.com/ |
| 4 | - **Access Control with IAM and Resource Tags** <br>&emsp; + Define IAM policies with tag-based conditions for least privilege. <br>&emsp; + Create IAM roles for EC2 administrators with conditional access. <br>&emsp; + Test permissions to verify tag-based access control. <br><br> - **Systems Management with AWS Systems Manager** <br>&emsp; + Configure Patch Manager to automate OS updates. <br>&emsp; + Use Run Command to execute tasks on multiple EC2 instances. <br>&emsp; + Review compliance and clean up resources. | 02/10/2025 | 02/10/2025 | https://000028.awsstudygroup.com/ <br> https://000031.awsstudygroup.com/ |
| 5 | - **Infrastructure as Code with AWS CloudFormation** <br>&emsp; + Create CloudFormation templates to deploy AWS infrastructure. <br>&emsp; + Manage resources through CloudFormation Stacks. <br><br> - **Cloud Development Kit (AWS CDK) Essentials** <br>&emsp; + Write CDK templates using programming languages. <br>&emsp; + Deploy and update resources through CDK. | 03/10/2025 | 03/10/2025 | https://000037.awsstudygroup.com/ <br> https://000038.awsstudygroup.com/ |
| 6 | - **AWS CDK Advanced Workshop** <br>&emsp; + Build a multi-service architecture using API Gateway, ECS, ALB, Lambda, and S3. <br>&emsp; + Create nested stacks for modular deployment. <br><br> - **Infrastructure as Code Workshop Series** <br>&emsp; + Learn IaC concepts, frameworks, and benefits. <br>&emsp; + Practice deploying multi-tier architectures with CloudFormation and CDK. | 04/10/2025 | 04/10/2025 | https://000076.awsstudygroup.com/ <br> https://000102.awsstudygroup.com/ |

---

# Week 4 Achievements

## Day 2 – Serverless Automation & Advanced Monitoring
- Implemented **AWS Lambda** to automate EC2 instance lifecycle management using scheduled events and tags.  
- Configured **IAM Roles** and verified automation functionality for starting and stopping instances.  
- Learned to apply **Savings Plans** for continuous workload cost optimization.  
- Installed **Grafana** on EC2 and integrated it with **CloudWatch** for real-time metrics visualization.  
- Built dashboards showing CPU, memory, and network utilization.  
- Configured alerting rules to detect threshold breaches and improve incident response time.

## Day 3 – CloudWatch Advanced Workshop & Resource Organization
- Gained in-depth understanding of **Amazon CloudWatch** for proactive system monitoring.  
- Configured **Metrics, Logs, Alarms, and Dashboards** for application and infrastructure visibility.  
- Used **Container Insights** to monitor ECS and EKS workloads.  
- Built **CloudWatch Dashboards** summarizing key performance indicators.  
- Applied **resource tagging** to classify AWS resources by owner, purpose, and environment.  
- Created **Resource Groups** for centralized management and automated operations across tagged assets.  
- Improved monitoring efficiency and governance in multi-resource environments.

## Day 4 – Access Control & Systems Management
- Implemented **IAM tag-based policies** to enforce least privilege and secure EC2 access.  
- Created **custom IAM Roles** with conditional tag-based permissions for administrators.  
- Verified access control behavior through testing and enforced tag compliance.  
- Learned **AWS Systems Manager (SSM)** for centralized automation and patch management.  
- Configured **Patch Manager** for automatic OS and security updates.  
- Used **Run Command** to manage multiple instances simultaneously.  
- Strengthened security posture and operational consistency through automation.

## Day 5 – Infrastructure as Code with CloudFormation & AWS CDK
- Practiced deploying infrastructure automatically using **CloudFormation templates** in YAML.  
- Created and managed **CloudFormation Stacks** to provision VPCs, subnets, and EC2 instances.  
- Learned rollback and stack update mechanisms for safer deployments.  
- Implemented **AWS CDK** to define infrastructure using **Python/TypeScript code** instead of raw YAML.  
- Deployed, modified, and updated resources through **CDK Stacks**.  
- Gained experience in **Infrastructure as Code (IaC)** and DevOps automation workflows.

## Day 6 – AWS CDK Advanced & Infrastructure as Code Workshop
- Designed and deployed **multi-service architectures** using **AWS CDK v2.151.0**, integrating **API Gateway**, **ALB**, **ECS (Fargate)**, **Lambda**, and **S3**.  
- Implemented **nested stacks** to modularize deployments for scalability and maintainability.  
- Learned to orchestrate complex infrastructure across services via reusable CDK constructs.  
- Reviewed key **IaC principles** and compared frameworks such as **CloudFormation**, **SAM**, **CDK**, and **Terraform**.  
- Deployed a **three-tier web application** and automated infrastructure creation through code.  
- Integrated IaC principles into **DevOps pipelines** for version-controlled, repeatable, and secure deployments.  
- Strengthened proficiency in infrastructure automation, scalability, and governance.

---

### Summary
- Automated EC2 scheduling and cost optimization with **AWS Lambda**.  
- Enhanced monitoring through **CloudWatch**, **Grafana**, and **Container Insights**.  
- Secured infrastructure access with **IAM tag-based policies**.  
- Centralized management and patching via **AWS Systems Manager**.  
- Implemented **Infrastructure as Code (IaC)** using **CloudFormation** and **AWS CDK**.  
- Built advanced, multi-service architectures using **CDK Advanced** and nested stacks.  
- Strengthened overall understanding of **DevOps automation**, **cloud governance**, and **scalable infrastructure design** on AWS.

---
