---
title: "Week 7 Worklog"
date: 2025-10-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---
### Week 7 Objectives:

* **Containerization:** Master the containerization of applications using **Docker** and deployment on EC2.
* **Container Orchestration:** Deploy and manage scalable containerized applications using **Amazon ECS (Elastic Container Service)**.
* **DevOps & Automation:** Implement **CI/CD Pipelines** (AWS CodePipeline, CodeBuild) for automated deployments to EC2 and ECS.
* **Storage Solutions:** Integrate on-premises environments with cloud storage using **File Storage Gateway** and **FSx for Windows**.
* **Advanced Database Architecture:** Deep dive into **Amazon DynamoDB** design patterns, indexing strategies, and data modeling for high-performance applications.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | **Application Containerization (Docker)** <br> - Install Docker on EC2 & Local Machine. <br> - Build Docker images for a sample application (Frontend/Backend). <br> - Use **Docker Compose** for multi-container orchestration. <br> - Push images to **Amazon ECR** (Elastic Container Registry). | 20/10/2025 | 20/10/2025 | https://000015.awsstudygroup.com/ |
| 3 | **Container Orchestration (Amazon ECS)** <br> - Create **ECS Cluster** (Fargate/EC2 Launch Type). <br> - Define **Task Definitions** and Services. <br> - Configure **Application Load Balancer (ALB)** for traffic distribution. <br> - Implement Service Discovery with **AWS Cloud Map**. | 21/10/2025 | 21/10/2025 | https://000016.awsstudygroup.com/ |
| 4 | **CI/CD for Containers** <br> - Setup **AWS CodePipeline** triggering from GitHub/GitLab. <br> - Configure **AWS CodeBuild** to build and push Docker images. <br> - Automate deployment to ECS Clusters. <br> - Implement Monitoring with **Container Insights** and Logging with **FireLens**. | 22/10/2025 | 22/10/2025 | https://000017.awsstudygroup.com/ |
| 5 | **CI/CD for EC2 & Hybrid Storage** <br> - **CodePipeline for EC2:** Automate deployments using CodeDeploy agent. <br> - **Storage Gateway:** Configure File Gateway to mount S3 buckets as NFS shares. <br> - **Amazon FSx:** Deploy fully managed Windows File Server and mount via SMB. | 23/10/2025 | 23/10/2025 | https://000023.awsstudygroup.com/ <br><br> _Self-Study on FSx/Storage Gateway_ |
| 6 | **Advanced DynamoDB Architecture** <br> - **Data Modeling:** Design Single Table Design patterns. <br> - **Performance Tuning:** Work with Partition Keys, Sort Keys, and Capacity Units (RCU/WCU). <br> - **Advanced Features:** Implement Global Secondary Indexes (GSI), DynamoDB Streams, and Global Tables. | 24/10/2025 | 24/10/2025 | https://000039.awsstudygroup.com/ |

# Week 7 Achievements

## Containerization & Orchestration
- Successfully "dockerized" a monolithic application, decoupling it from the underlying infrastructure.
- Deployed a highly available container cluster using **Amazon ECS**, utilizing Task Definitions to define resource requirements.
- Configured **Service Discovery** to allow microservices within the cluster to communicate via logical names instead of IP addresses.
- Implemented **Application Load Balancer** to distribute public traffic to dynamic container targets.

## DevOps & CI/CD Automation
- Built a robust **CI/CD Pipeline** using AWS CodePipeline and CodeBuild.
- Achieved **Continuous Deployment**: Code changes pushed to the repository automatically trigger a build, image creation, and rolling update to the ECS service without downtime.
- Integrated comprehensive monitoring using **CloudWatch Container Insights** to track CPU/Memory utilization of tasks.
- Centralized container logs using **FireLens** for easier debugging and analysis.

## Hybrid Storage Implementation
- Bridged the gap between on-premises protocols and cloud storage.
- Configured **File Storage Gateway** to allow legacy applications to write to S3 using standard NFS protocols.
- Deployed **Amazon FSx for Windows File Server**, providing a fully managed, native Windows file system with Active Directory integration.

## High-Performance Database Design
- Mastered NoSQL concepts with **Amazon DynamoDB**.
- Optimized database performance by designing efficient Partition and Sort Keys.
- Implemented **Global Secondary Indexes (GSI)** to support diverse access patterns without duplicating data tables.
- Explored **DynamoDB Streams** for event-driven architectures (triggering Lambda functions on database changes).