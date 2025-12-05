---
title: "Week 3 Worklog"
date: 2025-09-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---
### Week 3 Objectives:

* Implement comprehensive monitoring and observability strategies using **Amazon CloudWatch** (Metrics, Logs, Alarms, Dashboards).
* Establish **Hybrid DNS** architectures to integrate on-premises networks with AWS using **Route 53 Resolver** and Microsoft Active Directory.
* Achieve proficiency in **AWS CLI** for advanced resource management (Infrastructure as Code basics) and automation across Storage, Networking, Identity, and Compute services.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **Monitoring & Observability (CloudWatch)** <br> - Configure **CloudWatch Metrics**: Viewing, Search expressions, Math expressions <br> - Analyze logs with **CloudWatch Logs Insights** & Metric Filters <br> - Set up **CloudWatch Alarms** for proactive monitoring <br> - Create unified **CloudWatch Dashboards** <br> - **Practice:** Visualize EC2 performance and set up alerts for high CPU usage. | 22/09/2025 | 22/09/2025 | https://000008.awsstudygroup.com/ |
| 3 | **Hybrid Connectivity & DNS** <br> - Deploy **Microsoft Active Directory (AD)** <br> - Configure **Remote Desktop Gateway (RDGW)** for secure access <br> - Setup **Hybrid DNS** with Route 53 Resolver: <br>&emsp; + Create Outbound/Inbound Endpoints <br>&emsp; + Configure Resolver Rules <br> - Test DNS resolution between simulated on-prem and AWS environments. | 23/09/2025 | 23/09/2025 | https://000010.awsstudygroup.com/ |
| 4 | **AWS CLI Mastery - Part 1 (Setup & Storage)** <br> - Install and Configure **AWS CLI v2** <br> - **S3 Automation:** Manage buckets, perform multipart uploads via CLI <br> - **Messaging:** Manage **Amazon SNS** topics and subscriptions via CLI <br> - Explore CLI output formats (json, table, text) and filtering. | 24/09/2025 | 24/09/2025 | https://000011.awsstudygroup.com/ <br><br> https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html |
| 5 | **AWS CLI Mastery - Part 2 (Compute & Network)** <br> - **IAM Security:** Manage users, roles, and assume roles via CLI <br>&emsp; + Handle MFA tokens with CLI <br> - **Network Infrastructure:** Deploy VPC, Subnets, and Internet Gateways via CLI <br> - **Compute:** Launch and configure **EC2 instances** entirely through command line. | 25/09/2025 | 25/09/2025 | https://000011.awsstudygroup.com/ <br><br> [Deep Dive: AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) |
| 6 | **AWS CLI Mastery - Part 3 (Troubleshoot & Cleanup)** <br> - **Troubleshooting:** Diagnose common CLI errors (credentials, permissions, formatting) <br> - **Advanced Auth:** Store SAML credentials <br> - Perform full resource cleanup using CLI commands to ensure no lingering costs. | 26/09/2025 | 26/09/2025 | https://000011.awsstudygroup.com/ <br><br> [AWS re:Invent 2019: Introduction to the AWS CLI v2](https://www.youtube.com/watch?v=k_l8k2A8F_k) |

# Week 3 Achievements

## Observability & Monitoring
- Successfully deployed a monitoring stack using **Amazon CloudWatch**.
- Created custom **Dashboards** to visualize critical metrics (CPU, Memory, Disk I/O) in real-time.
- Configured **CloudWatch Alarms** to trigger automated notifications via SNS when thresholds were breached.
- Mastered **Logs Insights** to query and analyze application log data effectively.

## Hybrid Networking
- Designed and implemented a **Hybrid DNS** solution bridging on-premises and cloud environments.
- Deployed **Microsoft Active Directory** on AWS and integrated it with Route 53.
- Configured **Route 53 Resolver Endpoints** (Inbound/Outbound) and **Forwarding Rules** to resolve domain names across the hybrid network.
- Verified secure connectivity using **Remote Desktop Gateway (RDGW)**.

## Advanced Infrastructure Management (CLI)
- Transitioned from Console-based management to **Command Line Interface (CLI)** operations.
- Automating resource provisioning:
    - **Storage:** Created S3 buckets and managed object lifecycles/uploads.
    - **Security:** Managed IAM users, policies, and practiced **Assuming Roles** for cross-account access.
    - **Networking:** Built a complete VPC stack (VPC, Subnets, IGW, Route Tables) using CLI commands.
    - **Compute:** Launched EC2 instances with user-data scripts via CLI.
- Gained troubleshooting skills for CLI configuration, credential management (MFA/SAML), and API error handling.