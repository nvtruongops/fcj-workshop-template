---
title: "Week 6 Worklog"
date: 2025-10-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---
### Week 6 Objectives:

* **Identity & Access Management (IAM):** Implement robust identity controls using **IAM Identity Center (SSO)**, restrict privileges with **Permission Boundaries**, and secure role assumptions with conditional policies.
* **Security Compliance & Defense:** Automate security checks with **AWS Security Hub** and protect web applications using **AWS WAF**.
* **Data Protection:** Master data encryption using **AWS KMS** (Key Management Service) and establish comprehensive backup strategies with **AWS Backup**.
* **Advanced Networking:** Implement scalable network architectures using **VPC Peering** for direct connections and **Transit Gateway** for hub-and-spoke models.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **Advanced Identity Management** <br> - Setup **IAM Identity Center (SSO)** for centralized access across accounts. <br> - Create **Permission Boundaries** to limit maximum permissions for IAM users/roles. <br> - Configure **IAM Role Conditions** to prevent role transfer abuse. | 13/10/2025 | 13/10/2025 | https://000012.awsstudygroup.com/ <br><br> https://000030.awsstudygroup.com/ <br><br> https://000044.awsstudygroup.com/ |
| 3 | **Security Posture & Application Defense** <br> - Enable **AWS Security Hub** to check compliance (CIS Standards). <br> - Deploy **AWS WAF** (Web Application Firewall): <br>&emsp; + Create Web ACLs with Managed Rules. <br>&emsp; + Block SQL Injection/XSS attacks on ALBs. | 14/10/2025 | 14/10/2025 | https://000018.awsstudygroup.com/ <br><br> https://000026.awsstudygroup.com/ |
| 4 | **Data Protection (Encryption & Backup)** <br> - Create Customer Managed Keys (CMK) in **AWS KMS**. <br> - Encrypt S3 buckets and audit usage via **CloudTrail** & **Athena**. <br> - Configure **AWS Backup**: <br>&emsp; + Create Backup Vault and Backup Plan. <br>&emsp; + Perform on-demand backup and restore of resources. | 15/10/2025 | 15/10/2025 | https://000033.awsstudygroup.com/ <br><br> https://000013.awsstudygroup.com/ |
| 5 | **Network Reliability (Peering & Transit)** <br> - **VPC Peering:** Connect two isolated VPCs and configure Route Tables/DNS. <br> - **Transit Gateway (TGW):** Build a central hub to connect multiple VPCs. <br> - Configure TGW Route Tables for traffic isolation. | 16/10/2025 | 16/10/2025 | https://000019.awsstudygroup.com/ <br><br> https://000020.awsstudygroup.com/ |
| 6 | **Review & Cleanup** <br> - Review all security configurations. <br> - **Important:** Delete Transit Gateway, NAT Gateways, and disable Security Hub/Config to avoid high costs. <br> - Verify deletion of KMS keys (schedule deletion). | 17/10/2025 | 17/10/2025 | _All Cleanup Sections_ |

# Week 6 Achievements

## Identity & Access Management (IAM)
- Successfully deployed **IAM Identity Center** to centralize user management and simplify login access across the organization.
- Implemented **Permission Boundaries** to prevent privilege escalation, ensuring users cannot grant themselves more permissions than authorized.
- Enhanced security by adding **Conditions** to IAM Roles, restricting how and where roles can be assumed or passed (e.g., limiting role passing to specific services).

## Security Compliance & Application Defense
- Activated **AWS Security Hub** to gain a comprehensive view of the security state and automatically check against CIS AWS Foundations Benchmark.
- Protected web applications by deploying **AWS WAF**, successfully creating rules to block common threats like SQL Injection and malicious IP addresses before they reach the application load balancer.

## Data Encryption & Backup Strategy
- Managed data security using **AWS KMS**, creating Customer Managed Keys to encrypt sensitive data in S3.
- Audited key usage and data access patterns by integrating KMS logs with **CloudTrail** and querying them via **Amazon Athena**.
- Established a robust Disaster Recovery plan using **AWS Backup**, automating the backup schedules for EBS volumes and testing the successful restoration of data from a recovery point.

## Advanced Networking & Connectivity
- Connected isolated network environments using **VPC Peering**, enabling private communication between VPCs without traversing the public internet.
- Solved the complexity of mesh networking by implementing **AWS Transit Gateway**, creating a scalable hub-and-spoke network architecture to interconnect multiple VPCs efficiently.
- Configured complex **Route Tables** for both Peering and Transit Gateway to ensure correct traffic flow and network isolation.