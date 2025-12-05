---
title: "Week 4 Worklog"
date: 2025-09-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---
### Week 4 Objectives:

* Deploy a production-grade **WordPress** architecture ensuring High Availability (HA) and Scalability using Auto Scaling Groups and CloudFront.
* Master **Server Migration** workflows by importing/exporting Virtual Machines (VMs) between on-premises environments and AWS.
* Perform comprehensive **Database Migration** for heterogeneous sources using AWS Schema Conversion Tool (SCT) and Database Migration Service (DMS).
* Research advanced cloud migration strategies and tools (DataSync, MGN, Outposts).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **WordPress on AWS - Part 1 (Deployment)** <br> - Prepare Infrastructure: VPC, Subnets, Security Groups for Web & DB <br> - Launch **Multi-AZ RDS** for high database availability <br> - Deploy EC2 and install **WordPress** application <br> - Connect WordPress to RDS instance | 29/09/2025 | 29/09/2025 | https://000021.awsstudygroup.com/ |
| 3 | **WordPress on AWS - Part 2 (Scaling & CDN)** <br> - Create **AMI** from the configured WordPress instance <br> - Setup **Auto Scaling Group (ASG)** with Application Load Balancer <br> - Accelerate content delivery using **Amazon CloudFront** <br> - Perform Database Backup (Snapshot) & Restore operations <br> - Cleanup resources | 30/09/2025 | 30/09/2025 | https://000021.awsstudygroup.com/ |
| 4 | **Server Migration (VM Import/Export)** <br> - Deploy local Application Server (Simulated On-Prem) <br> - **Import VM to AWS:** Export local VM, upload to S3, and convert to AMI/EC2 <br> - **Export VM from AWS:** Export EC2 instance back to S3 for on-prem use <br> - Manage S3 ACLs and CLI roles for migration tasks | 01/10/2025 | 01/10/2025 | https://000014.awsstudygroup.com/ |
| 5 | **Database Migration (DMS & SCT)** <br> - Set up Migration Environment: Source (Oracle/SQL Server) & Target (Aurora/RDS) <br> - Use **Schema Conversion Tool (SCT)** to convert database schemas <br> - Configure **AWS DMS**: <br>&emsp; + Create Replication Instance & Endpoints <br>&emsp; + Run Migration Task (Full Load + CDC) <br> - Explore **DMS Serverless** for auto-scaling replication | 02/10/2025 | 02/10/2025 | https://000043.awsstudygroup.com/ |
| 6 | **Advanced Migration Monitoring & Strategies** <br> - **Monitoring DMS:** Analyze CloudWatch metrics, Table statistics, and Task logs <br> - **Troubleshooting:** Diagnose memory pressure and table errors during migration <br> - **Research:** Explore upcoming migration patterns: <br>&emsp; + AWS Application Migration Service (MGN) <br>&emsp; + AWS DataSync & Migration Hub <br>&emsp; + Container Migration to EKS | 03/10/2025 | 03/10/2025 | https://000043.awsstudygroup.com/ <br><br> _Self-Study on AWS Migration Hub_ |

# Week 4 Achievements

## Scalable Web Architecture
- Successfully deployed a highly available **WordPress** site using **Multi-AZ RDS** and **Auto Scaling Groups**.
- Configured **Application Load Balancer (ALB)** to distribute traffic dynamically across healthy instances.
- Optimized global content delivery speeds by integrating **Amazon CloudFront** as a CDN.
- Implemented disaster recovery procedures using RDS Snapshots and restoration techniques.

## Server Migration Expertise
- Gained hands-on experience with **VM Import/Export** methodologies.
- Successfully migrated a virtual machine from a simulated on-premises environment (VMware) to AWS EC2.
- Mastered the reverse process of exporting an active AWS EC2 instance back to a portable VM image stored in S3.
- Configured necessary IAM roles and S3 permissions to facilitate secure image transfer.

## Database Migration & Modernization
- Executed a heterogeneous database migration (e.g., SQL Server/Oracle to Aurora) using **AWS DMS**.
- Utilized the **AWS Schema Conversion Tool (SCT)** to automate schema transformation between different database engines.
- Configured **Continuous Data Replication (CDC)** to keep source and target databases in sync with minimal downtime.
- Experimented with **DMS Serverless** to handle variable migration workloads automatically.
- Learned to monitor migration health using CloudWatch and troubleshoot common issues like memory pressure or data type mismatches.