---
title: "Week 2 Worklog"
date: 2025-09-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---
### Week 2 Objectives:

* Master the usage of AWS Cloud9 IDE and AWS CLI for interacting with AWS services.
* Deep dive into AWS Storage services (S3) including Static Website Hosting, Versioning, Replication, and Content Delivery Network (CloudFront).
* Deploy and manage Relational Databases with Amazon RDS, including backup and restoration strategies.
* Implement High Availability and Scalability architectures using Launch Templates, Elastic Load Balancing (ELB), and Auto Scaling Groups (ASG).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **Interaction with AWS Services (Cloud9 & CLI)** <br> - Create AWS Cloud9 environment <br> - Explore Basic Features: Command line usage, Working with text files <br> - Practice using **AWS CLI** to interact with services <br> - Resource cleanup | 15/09/2025 | 15/09/2025 | https://000049.awsstudygroup.com/ |
| 3 | **Storage & Content Delivery (S3 & CloudFront)** <br> - Create S3 Bucket & Upload Data <br> - Configure **Static Website Hosting** on S3 <br> - Manage Public Access Block & Public Object policies <br> - Accelerate website with **Amazon CloudFront** integration <br> - Implement **S3 Bucket Versioning** for data protection <br> - Practice Move Objects & **Cross-Region Replication (CRR)** | 16/09/2025 | 16/09/2025 | https://000057.awsstudygroup.com/ |
| 4 | **Database Management (Amazon RDS)** <br> - Prepare Network: VPC, Security Groups for EC2 & RDS, DB Subnet Group <br> - Launch EC2 instance for application <br> - Provision **Amazon RDS** database instance <br> - Deploy application and connect to RDS <br> - Perform **Backup and Restore** operations for RDS | 17/09/2025 | 17/09/2025 | https://000005.awsstudygroup.com/ |
| 5 | **High Availability Architecture - Part 1** <br> - Prepare Infrastructure: Network, EC2, RDS, Web Server Deployment <br> - Create **Launch Template** for standardized instance launches <br> - Configure **Application Load Balancer (ALB)** <br>&emsp; + Create Target Group <br>&emsp; + Setup Load Balancer listeners <br> - Test Load Balancing traffic distribution | 18/09/2025 | 18/09/2025 | https://000006.awsstudygroup.com/ |
| 6 | **High Availability Architecture - Part 2 (Auto Scaling)** <br> - Create **Auto Scaling Group (ASG)** attached to the Load Balancer <br> - Test Scaling Solutions: <br>&emsp; + **Manual Scaling**: Manually adjusting capacity <br>&emsp; + **Scheduled Scaling**: Scaling based on time <br>&emsp; + **Dynamic Scaling**: Scaling based on metrics (CPU, etc.) <br>&emsp; + **Predictive Scaling**: Review metrics <br> - Clean up all resources | 19/09/2025 | 19/09/2025 | https://000006.awsstudygroup.com/ |

# Week 2 Achievements

## Development Tools & CLI
- Successfully set up **AWS Cloud9** as a cloud-based IDE.
- Gained proficiency in using **AWS CLI** for service management without the Console.
- Understood the workflow of editing, debugging, and running scripts directly on Cloud9.

## Storage & CDN Mastery
- Deployed a **Static Website** using Amazon S3.
- Secured and optimized content delivery globally using **Amazon CloudFront**.
- Implemented data protection strategies using **S3 Versioning**.
- Configured **Cross-Region Replication (CRR)** for disaster recovery and data locality.

## Database Management
- Deployed a fully managed relational database using **Amazon RDS**.
- Successfully connected a web application hosted on EC2 to the RDS instance.
- Practiced database maintenance tasks including taking **Snapshots** and performing **Point-in-Time Recovery**.
- Managed network security for databases using Security Groups and DB Subnet Groups.

## High Availability & Scalability
- Designed a fault-tolerant architecture using **Elastic Load Balancing (ALB)** and **Auto Scaling Groups (ASG)**.
- Created **Launch Templates** to define instance configurations for auto-scaling.
- Implemented and tested various scaling policies:
    - **Manual Scaling** for hands-on control.
    - **Scheduled Scaling** for predictable traffic patterns.
    - **Dynamic Scaling** for reacting to real-time load changes.
- Verified traffic distribution and application availability during scaling events.