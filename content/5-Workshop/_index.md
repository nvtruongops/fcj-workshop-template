---
title: "Workshop"
date: 2025-12-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building EveryoneCook: A Full-Stack AWS Infrastructure Workshop

#### Overview

In this comprehensive workshop, you will learn how to build a production-ready, full-stack social cooking application infrastructure on AWS using Infrastructure as Code (IaC) with AWS CDK. The **EveryoneCook** platform demonstrates modern cloud architecture patterns, including serverless computing, content delivery, AI-powered features, advanced search with OpenSearch, and comprehensive observability.

You will deploy seven interconnected CDK stacks that work together to create a scalable, secure, and cost-optimized application:
+ **DNS Stack** - Route 53 hosted zone for domain management
+ **Certificate Stack** - ACM certificates for CloudFront and API Gateway (us-east-1)
+ **CoreStack** - DynamoDB Single Table, S3 buckets, CloudFront CDN, KMS encryption, and OpenSearch
+ **AuthStack** - Cognito User Pool with Lambda triggers and SES email integration
+ **BackendStack** - API Gateway, Lambda functions (6 modules), SQS queues, and WAF protection
+ **FrontendStack** - AWS Amplify hosting for Next.js 15 application (optional)
+ **ObservabilityStack** - CloudWatch dashboards, alarms, and X-Ray distributed tracing

#### Content

1. [Workshop Overview](5.1-workshop-overview)
2. [Setup Environment](5.2-setup-environment/)
3. [CDK Bootstrap](5.3-cdk-bootstrap/)
4. [Configure Infrastructure Stacks](5.4-configure-stacks/)
5. [Deploy Infrastructure](5.5-deploy-infrastructure/)
6. [Configure API & Lambda](5.6-configure-api-lambda/)
7. [Deploy Backend Services](5.7-deploy-backend/)
8. [Test Endpoints End-to-End](5.8-test-endpoints/)
9. [Push to GitLab](5.9-push-gitlab/)
10. [Deploy to Amplify](5.10-deploy-amplify/)
11. [Clean up](5.11-cleanup/)
