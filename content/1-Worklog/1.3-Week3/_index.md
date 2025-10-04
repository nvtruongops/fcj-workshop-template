---
title: "Week 3 Worklog"
date: 2025-09-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Learn and practice **NoSQL database management** with **Amazon DynamoDB**.  
* Understand **In-Memory Caching** with **Amazon ElastiCache (Redis)**.  
* Explore **Networking and Content Delivery** with **VPC**, **CloudFront**, and **Lambda@Edge**.  
* Learn **Directory Services** using **AWS Managed Microsoft AD**.  
* Deploy **Highly Available Web Applications** with **Auto Scaling**, **ALB**, **RDS Multi-AZ**, and **CloudFront**.  
* Practice **Migration & Disaster Recovery** using **VM Import/Export**, **Database Migration Service**, and **Elastic Disaster Recovery**.  

---

### Tasks to be carried out this week:

| Day | Task                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Start Date | Completion Date | Reference Material |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ------------------ |
| 2 | - **NoSQL Database Essentials with Amazon DynamoDB** <br>&emsp; + Learn DynamoDB components, keys, and capacity modes. <br>&emsp; + Practice table creation, PITR, backup/restore. <br><br> - **In-Memory Caching with Amazon ElastiCache (Redis)** <br>&emsp; + Learn Redis clusters, shards, and caching. <br>&emsp; + Practice Redis cluster creation and SDK operations. | 22/09/2025 | 22/09/2025 | https://000060.awsstudygroup.com/ <br> https://000061.awsstudygroup.com/ |
| 3 | - **Networking on AWS Workshop** <br>&emsp; + Learn VPC, Subnets, Routing, Security Groups, NACLs. <br>&emsp; + Practice VPC build and hybrid connectivity. <br><br> - **Content Delivery with Amazon CloudFront** <br>&emsp; + Practice CloudFront distribution with S3 origin. | 23/09/2025 | 23/09/2025 | https://000092.awsstudygroup.com/ <br> https://000094.awsstudygroup.com/ |
| 4 | - **Edge Computing with CloudFront & Lambda@Edge** <br>&emsp; + Create CloudFront distribution and Lambda@Edge. <br>&emsp; + Deploy Lambda@Edge for header/redirect logic. <br><br> - **Windows Workloads on AWS (WorkSpaces)** <br>&emsp; + Deploy and access Amazon WorkSpaces. | 24/09/2025 | 24/09/2025 | https://000130.awsstudygroup.com/ <br> https://000093.awsstudygroup.com/ |
| 5 | - **Directory Services with AWS Managed Microsoft AD** <br>&emsp; + Deploy AWS Managed AD across 2 AZs and join Windows Server 2022 to domain. <br>&emsp; + Manage OU, Users, GPOs. <br><br> - **Building Highly Available Web Applications (WordPress)** <br>&emsp; + Deploy WordPress on EC2 + RDS Multi-AZ with ALB & Auto Scaling. | 25/09/2025 | 25/09/2025 | https://000095.awsstudygroup.com/ <br> https://000101.awsstudygroup.com/ |
| 6 | - **Migrate to AWS – VM Import/Export** <br>&emsp; + Understand VM Import/Export service for migrating on-prem VMs to EC2. <br>&emsp; + Learn S3 integration and security requirements. <br><br> **Practice:** <br>&emsp; + Deploy application server on-prem (VMware/Hyper-V). <br>&emsp; + Export VM image to OVA/OVF. <br>&emsp; + Create S3 bucket and upload image. <br>&emsp; + Use AWS CLI to import VM into EC2. <br>&emsp; + Verify instance launch and network configuration. <br>&emsp; + Export VM back to on-premises environment. <br>&emsp; + Cleanup AMI and S3 objects. <br><br> - **Database Migration with AWS DMS & Schema Conversion Tool (SCT)** <br>&emsp; + Learn how SCT automates schema conversion between heterogeneous DB engines. <br>&emsp; + Understand AWS DMS for continuous replication and low-downtime migration. <br><br> **Practice:** <br>&emsp; + Install AWS Schema Conversion Tool on local or EC2. <br>&emsp; + Analyze source DB and convert schema to target RDS (MySQL/PostgreSQL). <br>&emsp; + Set up DMS Replication Instance and create source/target endpoints. <br>&emsp; + Migrate schema and data using DMS tasks with ongoing replication. <br>&emsp; + Monitor and troubleshoot migration progress. <br><br> - **Disaster Recovery with AWS Elastic Disaster Recovery (DRS)** <br>&emsp; + Learn how AWS DRS reduces downtime and data loss for critical apps. <br><br> **Practice:** <br>&emsp; + Connect to Bastion Host and configure DRS IAM User. <br>&emsp; + Install DRS Agent on source server and create Replication Settings. <br>&emsp; + Configure Launch Template for failover instances. <br>&emsp; + Perform test failover and validate recovery points. <br>&emsp; + Cleanup resources after simulation. | 26/09/2025 | 26/09/2025 | https://cloudjourney.awsstudygroup.com/2-migrate/ <br> https://000014.awsstudygroup.com/ <br> https://000043.awsstudygroup.com/ <br> https://000100.awsstudygroup.com/ |

---

# Week 3 Achievements

## Day 2 – NoSQL Database Essentials & In-Memory Caching
- Built and managed DynamoDB tables with PITR and On-Demand Backup.  
- Deployed Redis cluster and tested cache operations through SDK.  

## Day 3 – Networking on AWS & Content Delivery
- Built custom VPC, subnets, and gateways for network segmentation.  
- Configured CloudFront distribution for S3 origin and verified low-latency delivery.  

## Day 4 – Edge Computing & Windows Workloads
- Deployed Lambda@Edge functions for request manipulation at edge locations.  
- Deployed Amazon WorkSpaces virtual desktops and tested access methods.  

## Day 5 – Directory Services & Highly Available Web Applications
- Deployed AWS Managed Microsoft AD across two AZs and joined Windows Server to domain.  
- Managed Active Directory users, groups, and GPOs via Administrative Tools.  
- Implemented WordPress on EC2 with ALB, Auto Scaling, RDS Multi-AZ, and CloudFront for high availability.  

## Day 6 – Migration & Disaster Recovery on AWS

### VM Migration with AWS VM Import/Export
- Practiced importing on-prem VM images to Amazon EC2 and exporting back to VMware.  
- Understood use cases for backup, DR, and application migration.  
- Used S3 as temporary storage and AWS CLI for import/export commands.  

### Database Migration with AWS DMS & SCT
- Converted schemas using AWS Schema Conversion Tool (SCT).  
- Set up AWS DMS replication instance for continuous data migration with minimal downtime.  
- Migrated data from on-prem source DB to Amazon RDS target database.  
- Verified migration consistency and monitored tasks in AWS Console.  

### Disaster Recovery with AWS Elastic Disaster Recovery (DRS)
- Learned AWS DRS architecture and configuration steps.  
- Installed agents on source servers and configured replication to AWS.  
- Simulated failover and tested system recovery in target Region.  
- Verified point-in-time restoration and cleaned up replicated resources.  

---
