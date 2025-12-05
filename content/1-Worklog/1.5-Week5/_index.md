---
title: "Week 5 Worklog"
date: 2025-10-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---
### Week 5 Objectives:

* **Operational Excellence:** Automate operational tasks (server start/stop) using **AWS Lambda** and integrate with communication tools (Slack).
* **Monitoring & Observability:** Build advanced dashboards using **Grafana** integrated with **Amazon CloudWatch** for deep system insights.
* **Governance & Tagging:** Implement resource organization strategies using **Tags**, **Resource Groups**, and enforce Attribute-Based Access Control (ABAC) via **IAM**.
* **Systems Management (SSM):** Explore server management without SSH using **Session Manager** and understand automated patching workflows (Patch Manager).
* **Infrastructure as Code (IaC):** Begin automating infrastructure deployment using **AWS CloudFormation**.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **Cost Optimization & Automation** <br> - Create **Lambda functions** (Python boto3) to automatically Start/Stop EC2 instances based on schedules. <br> - Integrate Lambda with **Slack Webhooks** for operational notifications. <br> - Configure **EventBridge** (CloudWatch Events) triggers. | 06/10/2025 | 06/10/2025 | https://000022.awsstudygroup.com/ |
| 3 | **Advanced Monitoring (Grafana)** <br> - Launch EC2 to host **Grafana** (Open Source). <br> - Connect Grafana to **Amazon CloudWatch** data source. <br> - Create custom dashboards visualizing CPU, Network, and Disk metrics from EC2. | 07/10/2025 | 07/10/2025 | https://000029.awsstudygroup.com/ |
| 4 | **Governance & Security (Tags & IAM)** <br> - **Tagging Strategy:** Apply consistent tags to resources (Env, Owner, CostCenter). <br> - Create **Resource Groups** to manage resources by tags. <br> - **ABAC Implementation:** Configure IAM Policies to control EC2 access based on matching tags between User and Resource. | 08/10/2025 | 08/10/2025 | https://000027.awsstudygroup.com/ <br><br> https://000028.awsstudygroup.com/ |
| 5 | **Systems Management (SSM)** <br> - **Session Manager:** Connect to private/public instances without opening SSH ports (Port Forwarding, Audit Logs to S3). <br> - **Run Command:** Execute remote commands across multiple instances. <br> - *Theory/Review:* **Patch Manager** workflows (scanning & compliance) due to potential Free Tier/Lab permission limits. | 09/10/2025 | 09/10/2025 | https://000058.awsstudygroup.com/ <br><br> https://000031.awsstudygroup.com/ |
| 6 | **Infrastructure as Code (CloudFormation)** <br> - Write basic **CloudFormation templates** (YAML/JSON) to deploy VPC and EC2. <br> - Understand **Stacks**, **Parameters**, **Mappings**, and **Outputs**. <br> - Explore **Drift Detection** to identify manual changes to infrastructure. | 10/10/2025 | 10/10/2025 | https://000037.awsstudygroup.com/ |

# Week 5 Achievements

## Automation & Cost Management
- Successfully implemented **automated cost-saving mechanisms** by scheduling non-production EC2 instances to stop during off-hours using **AWS Lambda**.
- Deployed a notification system where operational events trigger messages to a **Slack channel**, improving team visibility.
- Reduced manual intervention by leveraging **EventBridge Rules** to trigger automation workflows.

## Monitoring Visualization
- Deployed a self-hosted **Grafana** instance on EC2.
- successfully integrated **Amazon CloudWatch** as a data source for Grafana, enabling the visualization of AWS metrics in a unified, customizable interface.
- Created specific dashboards to monitor EC2 health, overcoming the limitations of default CloudWatch views.

## Resource Governance & Security
- Mastered **Tagging strategies** to organize resources by environment (Dev/Prod) and project.
- Implemented **Resource Groups** to bulk-manage resources sharing common tags.
- Enhanced security using **Attribute-Based Access Control (ABAC)**: Configured IAM policies that strictly allow users to manage only the EC2 instances that match their specific department tags, effectively isolating environments.

## Systems Operations (SysOps)
- Eliminated the need for Bastion Hosts/SSH keys by using **AWS Systems Manager Session Manager** for secure instance access.
- Configured **Session Logging** to S3 for auditing and compliance purposes.
- Studied and simulated the **Patch Manager** process for automating OS updates and maintaining compliance across fleets of instances.
- Utilized **Run Command** to execute administrative scripts across multiple servers simultaneously without logging into each one.

## Infrastructure as Code (IaC)
- Transitioned from manual console clicks to defining infrastructure in code using **AWS CloudFormation**.
- Created and deployed Stacks to provision VPCs and EC2 instances, ensuring consistent and repeatable deployments.
- Learned to detect "Configuration Drift" to identify where actual infrastructure state deviates from the defined CloudFormation templates.