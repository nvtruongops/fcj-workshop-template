---
title: "Workshop Overview"
date: 2025-12-01
weight: 1
chapter: false
pre: " <b> 5.01. </b> "
---

#### EveryoneCook Architecture

**EveryoneCook** is a modern social cooking platform built entirely on AWS serverless technologies. The architecture follows best practices for scalability, security, and cost optimization, with a focus on Vietnamese ingredient support and AI-powered recipe suggestions.

#### Key Components

+ **Frontend**: Next.js 15 application with Flowbite React components hosted on AWS Amplify
+ **Backend**: Serverless API using API Gateway with API Router pattern and 6 Lambda modules
+ **Database**: DynamoDB Single Table Design with username-based PK and 5 GSI indexes
+ **Storage**: S3 buckets with Intelligent-Tiering and CloudFront CDN with OAC
+ **Authentication**: Cognito User Pool with Lambda triggers for custom flows
+ **AI Integration**: Amazon Bedrock (Claude 3 Haiku - anthropic.claude-3-haiku-20240307-v1:0) for ingredient translation, nutrition lookup, and recipe generation
+ **Security**: WAF for API Gateway, Shield Standard for CloudFront, KMS encryption
+ **Monitoring**: CloudWatch dashboards, alarms, and X-Ray distributed tracing

#### Architecture Diagram

![EveryoneCook Architecture](/images/architecture-diagram.png)

#### Workshop Flow

This workshop follows a practical application development workflow:

1. **Setup Environment** - Install tools (Node.js, AWS CLI, CDK CLI)
2. **CDK Bootstrap** - Prepare AWS account for CDK deployments
3. **Configure Stacks** - Set up infrastructure configuration (DNS, Certificate, Core, Auth, Backend, Observability)
4. **Deploy Infrastructure** - Deploy all CDK stacks to AWS
5. **Configure API & Lambda** - Set up API Gateway routes and Lambda functions
6. **Deploy Backend** - Deploy API and Lambda code
7. **Test Endpoints** - Verify all endpoints work end-to-end
8. **Push to GitLab** - Version control and CI/CD setup
9. **Deploy to Amplify** - Deploy frontend to AWS Amplify
10. **Monitor & Maintain** - Use CloudWatch and X-Ray for monitoring

#### What You'll Learn

+ Infrastructure as Code with AWS CDK (TypeScript)
+ Serverless architecture with API Router pattern
+ DynamoDB Single Table Design with PROVISIONED billing mode
+ CloudFront CDN with Origin Access Control (OAC) - PriceClass 200
+ Cognito authentication with Lambda triggers
+ Lambda function modular organization (6 modules: API Router, Auth/User, Social, Recipe/AI, Admin, Upload)
+ SQS-based async processing with 4 queues and workers
+ Bedrock AI integration (Claude 3 Haiku) with dictionary-first caching strategy
+ WAF security for API Gateway (REGIONAL scope)
+ CloudWatch monitoring with structured logging (X-Ray disabled for cost optimization)
+ Cost optimization strategies for low-traffic scenarios

#### Realistic Cost Estimate (Low Traffic Scenarios)

**Scenario 1: 100-500 Active Users/Day (2 hours usage)**
- **Monthly Active Users**: ~3,000-15,000
- **Daily Requests**: ~5,000-25,000 API calls
- **Estimated Monthly Cost**: **$12-18/month** ($0.40-0.60/day)

**Scenario 2: 1,000 Active Users/Day (2 hours usage)**
- **Monthly Active Users**: ~30,000
- **Daily Requests**: ~50,000 API calls
- **Estimated Monthly Cost**: **$25-35/month** ($0.83-1.17/day)

**Key Cost Factors**:
+ **DynamoDB**: PROVISIONED mode (2 RCU/WCU) with auto-scaling → $1.25/month base + $0.50-3/month scaling
+ **Lambda**: 6 functions (512MB-1GB memory) → $2-8/month (first 1M requests free)
+ **CloudFront**: PriceClass 200 (Asia/US/Europe) → $1-5/month (first 1TB free, then $0.085/GB)
+ **API Gateway**: Regional endpoint → $1-3/month (first 1M free, then $3.50/million)
+ **Cognito**: Standard security (no Advanced Security Mode) → $0-1.50/month (first 50K MAU free)
+ **S3**: Intelligent-Tiering → $0.50-2/month (first 50GB free)
+ **Bedrock AI**: Claude 3 Haiku ($0.25/1M input tokens) → $2-8/month with 99% dictionary cache hit rate
+ **SQS**: 4 queues (AI, Image, Analytics, Notification) → $0-0.40/month (first 1M requests free)
+ **CloudWatch Logs**: 3-day retention → $0.50-2/month
+ **WAF**: API Gateway only (no CloudFront WAF) → $5/month base + $0.60/million requests

**Cost Optimization Applied**:
+ X-Ray tracing disabled (saves $5-10/month)
+ CloudFront WAF removed, using Shield Standard (saves $6/month)
+ DynamoDB PROVISIONED mode with low baseline (2 RCU/WCU) instead of ON_DEMAND
+ CloudWatch log retention reduced to 3 days for dev (saves $2-5/month)
+ API Gateway caching disabled for dev (saves $14.60/month)
+ Bedrock dictionary-first strategy (99% cache hit rate, reduces AI costs by 70%)
+ Lambda Shared Dependencies Layer (reduces deployment size 90%, faster cold starts)

#### Key Features

+ **Vietnamese Support**: Vietnamese analyzer for ingredient search
+ **AI-Powered**: Bedrock Claude 3 Haiku for recipe generation
+ **Dictionary-First Translation**: 80% coverage target with intelligent caching
+ **Social Platform**: Posts, comments, reactions, friends, notifications
+ **Field-Level Privacy**: Granular control over profile visibility
+ **Content Moderation**: Automated and manual moderation workflows
+ **Advanced Search**: Full-text search with Vietnamese normalization
+ **Cost Optimized**: Intelligent-Tiering, caching, and resource optimization
