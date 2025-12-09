---
title: "Configure Infrastructure Stacks"
date: 2025-12-01
weight: 4
chapter: false
pre: " <b> 5.04. </b> "
---

# Configure Infrastructure Stacks


## EveryoneCook Project Overview

### Introduction
EveryoneCook is a social network platform for sharing cooking recipes, built entirely on AWS Cloud. The project uses a serverless architecture, leveraging AWS managed services to ensure scalability, security, and cost optimization.

### System Architecture

The project is designed with a **5-Stack Architecture** featuring clear layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    EveryoneCook Platform                     │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: DNS & Certificate (Foundation)                    │
│  ├─ DNS Stack: Route 53 Hosted Zone                         │
│  └─ Certificate Stack: ACM Certificates (us-east-1)         │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Data & Storage (Core Infrastructure)              │
│  └─ Core Stack: DynamoDB, S3, CloudFront, KMS               │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Authentication & Security                         │
│  └─ Auth Stack: Cognito, Lambda Triggers, SES               │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Application & Business Logic                      │
│  └─ Backend Stack: API Gateway, Lambda, SQS, WAF            │
├─────────────────────────────────────────────────────────────┤
│  Layer 5: Monitoring & Observability                        │
│  └─ Observability Stack: CloudWatch, Alarms, Dashboards     │
└─────────────────────────────────────────────────────────────┘
```

### Technologies Used

#### Infrastructure as Code
- **AWS CDK (TypeScript)**: Infrastructure management with code
- **CloudFormation**: Underlying template engine for CDK
- **Git**: Version control for infrastructure code

#### Backend Services
- **API Gateway REST API**: RESTful API endpoint with custom domain
- **Lambda Functions**: Serverless compute for business logic (6 main functions + 1 worker)
- **Lambda Layers**: Shared dependencies layer (reduces deployment size by 90%)
- **DynamoDB**: NoSQL database with Single Table Design + 5 GSI indexes
- **S3**: Object storage for user content (avatars, posts, recipes, backgrounds)
- **SQS**: Message queue for async processing (4 queues + 4 DLQs)

#### Security & Authentication
- **Cognito User Pool**: User authentication & management
- **Cognito User Pool Client**: Frontend authentication config
- **WAF (Web Application Firewall)**: API Gateway protection with rate limiting
- **KMS (Key Management Service)**: 2 customer-managed keys (DynamoDB, S3)
- **IAM Roles & Policies**: Fine-grained access control for all services

#### Content Delivery & Networking
- **CloudFront**: CDN with Origin Access Control (OAC) + Shield Standard
- **Route 53**: DNS management with Hosted Zone
- **ACM (Certificate Manager)**: SSL/TLS certificates (2 certificates)

#### Monitoring & Operations
- **CloudWatch Logs**: Centralized logging for Lambda functions
- **CloudWatch Metrics**: Performance metrics tracking
- **CloudWatch Dashboards**: 4 dashboards (Core, Auth, Backend, Overview)
- **CloudWatch Alarms**: Real-time monitoring (10+ alarms) + Composite Alarm
- **SNS (Simple Notification Service)**: Email notifications for alarms

### Project Directory Structure

```
everyonecook/
├── infrastructure/                    # AWS CDK Infrastructure
│   ├── bin/
│   │   └── app.ts                    # CDK app entry point - creates all stacks
│   ├── lib/
│   │   ├── base-stack.ts             # Base class for all stacks
│   │   ├── stacks/                   # Stack definitions
│   │   │   ├── dns-stack.ts          # Route 53 Hosted Zone
│   │   │   ├── certificate-stack.ts  # ACM Certificates
│   │   │   ├── core-stack.ts         # DynamoDB, S3, CloudFront
│   │   │   ├── auth-stack.ts         # Cognito, Lambda triggers
│   │   │   ├── backend-stack.ts      # API Gateway, Lambda, SQS
│   │   │   └── observability-stack.ts # CloudWatch, Alarms
│   │   └── constructs/               # Reusable CDK constructs
│   │       └── shared-layer.ts       # Lambda Layer with dependencies
│   ├── config/
│   │   └── environment.ts            # Environment configuration (dev/staging/prod)
│   ├── cdk.json                      # CDK configuration
│   ├── package.json                  # Node.js dependencies
│   └── tsconfig.json                 # TypeScript configuration
├── services/                         # Lambda function source code
│   ├── api-router/                   # API request routing
│   ├── auth-user/                    # Authentication endpoints
│   ├── social/                       # Social features (posts, comments)
│   ├── recipe-ai/                    # Recipe & AI endpoints
│   ├── admin/                        # Admin management
│   └── upload/                       # File upload handler
├── shared/                           # Shared code & utilities
│   ├── utils/                        # Common utilities
│   ├── models/                       # Data models
│   └── middleware/                   # Lambda middleware
├── frontend/                         # Next.js frontend (deployed separately)
│   └── ...
└── layers/                           # Lambda layers
  └── shared-dependencies/          # Common npm packages
```

## Stack Architecture Overview

### 1. DNS Stack (Phase 1)
**Purpose**: Create the foundation for DNS management


**Main resources**:
- Route 53 Hosted Zone for domain `everyonecook.cloud`
- Export nameservers to configure at the domain registrar

**Deployment order**: This stack must be deployed first

**Estimated cost**: ~$0.50/month

> **Note**: After deploying this stack, update the nameservers at Hostinger to point to Route 53.

---


### 2. Certificate Stack (Phase 1.5)
**Purpose**: Create SSL/TLS certificates for CloudFront and API Gateway

**Main resources**:
- ACM Certificate for CloudFront: `cdn.everyonecook.cloud`
- ACM Wildcard Certificate for API Gateway: `*.everyonecook.cloud`
- DNS validation records in Route 53

**Special region**: **MUST deploy to us-east-1** (CloudFront requirement)

**Dependencies**: DNS Stack (requires Hosted Zone for DNS validation)

**Estimated cost**: Free (ACM certificates are free)

> **Important note**: CloudFront only accepts certificates from the us-east-1 region.

---


### 3. Core Stack (Phase 2)
**Purpose**: Create the data layer and storage infrastructure

**Main resources**:
- **DynamoDB Table**: Single Table Design with 5 GSI indexes
  - PK: `USER#{username}`, SK: `PROFILE|RECIPE#{id}|POST#{id}|COMMENT#{id}`
  - Username-based primary key (immutable)
  - GSI1-GSI5 for different query patterns:
    - GSI1: User recipes by date
    - GSI2: User posts by date
    - GSI3: Post comments
    - GSI4: Recipe search by cuisine/difficulty
    - GSI5: Social interactions (likes, follows)
  - Billing mode: Pay-per-request (on-demand)
  - Encryption: Customer-managed KMS key with auto-rotation
- **S3 Buckets** (2 buckets):
  - **Content Bucket**: User uploads (avatars, posts, recipes, backgrounds)
    - Intelligent-Tiering enabled (automatically optimizes cost)
    - Versioning enabled
    - Lifecycle rules: Delete incomplete multipart uploads after 7 days
    - Encryption: Customer-managed KMS key
  - **CDN Logs Bucket**: CloudFront access logs
    - Intelligent-Tiering enabled
    - Auto-delete logs after 90 days (cost optimization)
- **CloudFront Distribution**:
  - Origin: S3 Content Bucket
  - Origin Access Control (OAC) to secure S3 access (replaces OAI)
  - Custom domain: `cdn.everyonecook.cloud`
  - SSL Certificate: ACM certificate from Certificate Stack
  - Cache behaviors: Optimized for images & static content
  - Compression: Gzip & Brotli enabled
  - Shield Standard: DDoS protection (free, auto-enabled)
  - **NOTE**: CloudFront WAF removed to save $9/month
- **KMS Keys** (2 customer-managed keys):
  - **DynamoDB Encryption Key**: 
    - Auto-rotation enabled (yearly)
    - Used by: DynamoDB table, CloudWatch Logs
  - **S3 Encryption Key**: 
    - Auto-rotation enabled (yearly)
    - Used by: S3 buckets, CloudFront signed URLs
- **IAM Roles**: 
  - CloudFront Origin Access Control role
  - Lambda execution roles with DynamoDB & S3 access

**Dependencies**: Certificate Stack (requires certificate for CloudFront)

**Estimated cost**: ~$8-15/month

> **Cost optimization**: 
> - S3 Intelligent-Tiering automatically moves objects to cheaper storage
> - CloudFront WAF removed (Shield Standard provides DDoS protection)
> - CloudWatch Logs auto-delete after retention period

---


### 4. Auth Stack (Phase 3)
**Purpose**: Authentication and user management

**Main resources**:
- **Cognito User Pool**:
  - Sign-in methods: Username OR Email + Password
  - NO MFA requirement (only email + password)
  - Password policy: 
    - Min 12 chars for prod (8 chars for dev)
    - Require uppercase, lowercase, digits, symbols
  - Standard attributes: email, given_name (fullName), birthdate, gender
  - Custom attributes: account_status (for admin ban), country (ISO 3166-1)
  - Device tracking: Enabled (NO challenge required)
  - Email verification: Required before use
  - Account recovery: Email-based
  - **NOTE**: Advanced Security Mode OFF to save ~$5/month
- **Lambda Triggers** (5 functions with CloudWatch Logs):
  - **Pre-signup**: 
    - Validate username uniqueness
    - Check profanity/banned words
    - Runtime: Node.js 20.x
  - **Post-confirmation**: 
    - Create user profile in DynamoDB
    - Initialize user stats
  - **Pre-authentication**: 
    - Check account status (active/banned)
    - Log login attempts
  - **Post-authentication**: 
    - Update last login timestamp
    - Track user activity
  - **Custom message**: 
    - Customize verification emails
    - Branded email templates
- **Cognito User Pool Client**: 
  - OAuth flows: Authorization code grant
  - Token validity: 
    - Access token: 1 hour
    - Refresh token: 30 days
  - Read/Write attributes configured
- **CloudWatch Log Groups**: 
  - Each Lambda trigger has a dedicated log group
  - Retention: 7 days
  - Encryption: KMS DynamoDB key
- **IAM Roles**: 
  - Lambda execution roles
  - Permissions: DynamoDB read/write, Cognito admin, Logs write

**Dependencies**: Core Stack (Lambda triggers need access to DynamoDB)

**Estimated cost**: ~$0-2/month (Cognito free tier: 50,000 MAU)

> **Security**: 
> - Advanced Security Mode not enabled to optimize cost (would cost an extra ~$5/month)
> - Device tracking enabled but NO MFA to improve user experience
> - All Lambda triggers have CloudWatch logging

---


### 5. Backend Stack (Phase 4)
**Purpose**: Application layer and business logic

**Main resources**:
- **API Gateway REST API**:
  - Custom domain: `api.everyonecook.cloud`
  - SSL Certificate: Wildcard ACM certificate (`*.everyonecook.cloud`)
  - Cognito Authorizer: Validate JWT tokens
  - Request validators: 
    - Body validator: Validate request body
    - Params validator: Validate query params & path params
    - Full validator: Validate both body & params
  - Deployment stage: dev/staging/prod
  - CloudWatch Logs: Access logs + execution logs
  - WAF Web ACL attached (API protection)
- **Lambda Functions** (6 main modules + CloudWatch Logs):
  - **API Router**: 
    - Route requests to appropriate handlers
    - Memory: 256 MB, Timeout: 30s
  - **Auth User**: 
    - Login, register, forgot-password, verify-email
    - Cognito integration
    - Memory: 512 MB, Timeout: 30s
  - **Social**: 
    - Create/edit/delete posts, comments, likes
    - Follow/unfollow users
    - Memory: 512 MB, Timeout: 30s
  - **Recipe AI**: 
    - CRUD operations for recipes
    - AI recipe generation (Bedrock integration)
    - Recipe search & recommendations
    - Memory: 1024 MB, Timeout: 60s
  - **Admin**: 
    - User management, ban/unban
    - Content moderation
    - Analytics dashboard
    - Memory: 512 MB, Timeout: 30s
  - **Upload**: 
    - File upload to S3
    - Generate pre-signed URLs
    - Image validation
    - Memory: 512 MB, Timeout: 30s
  - Runtime: Node.js 20.x
  - Environment variables: DynamoDB table, S3 bucket, SQS queues
- **Lambda Layer - Shared Dependencies**:
  - AWS SDK v3 clients: DynamoDB, S3, SQS, Cognito, Bedrock, Lambda
  - Utilities: uuid, jsonwebtoken, jwks-rsa
  - **Benefit**: Reduces deployment size from 8MB → 200KB per function (90% reduction)
  - Compatible runtimes: Node.js 18.x, 20.x
- **SQS Queues** (4 main queues + 4 DLQs):
  - **AI Queue**: 
    - AI recipe generation requests
    - Visibility timeout: 120s (2 minutes)
    - Message retention: 4 days
    - Max receive count: 3 (then move to DLQ)
  - **Image Processing Queue**: 
    - Image optimization, thumbnails
    - Visibility timeout: 60s
    - Message retention: 4 days
  - **Analytics Queue**: 
    - User activity tracking
    - Visibility timeout: 30s
    - Message retention: 4 days
  - **Notification Queue**: 
    - Push notifications, emails
    - Visibility timeout: 30s
    - Message retention: 4 days
  - Encryption: AWS managed KMS keys
- **Worker Lambda** (1 active):
  - **AI Worker**: 
    - Process AI Queue messages
    - Call Amazon Bedrock for recipe generation
    - Store results in DynamoDB
    - Memory: 1024 MB, Timeout: 120s
    - Event source: AI Queue (batch size: 10)
  - *Note: Other workers not yet implemented*
- **WAF Web ACL** (API Gateway only):
  - **Rate limiting**: 2000 requests / 5 minutes per IP
  - **AWS Managed Rules**:
    - AWSManagedRulesCommonRuleSet (OWASP Top 10)
    - AWSManagedRulesKnownBadInputsRuleSet
  - **Custom rules**: Block specific patterns
  - **NOTE**: CloudFront WAF removed to save $9/month
  - Shield Standard: Auto-enabled (DDoS protection - free)
- **CloudWatch Log Groups**:
  - API Gateway access logs + execution logs
  - Lambda function logs (per function)
  - Retention: 7 days (cost optimization)
  - Encryption: KMS DynamoDB key
- **IAM Roles & Policies**:
  - API Gateway execution role
  - Lambda execution roles (per function)
  - Permissions: DynamoDB, S3, SQS, Cognito, Bedrock, Logs, KMS

**Shared Lambda Layer**: 
- AWS SDK v3 clients (DynamoDB, S3, SQS, Cognito, Bedrock)
- Common utilities (uuid, jsonwebtoken, jwks-rsa)
- Reduces deployment size: 8MB → 200KB per function

**Dependencies**: Auth Stack (requires Cognito User Pool)

**Estimated cost**: ~$10-25/month

> **Cost optimization**: 
> - CloudFront WAF removed (Shield Standard provides DDoS protection)
> - WAF only enabled for API Gateway (main attack surface)
> - Lambda Layer reduces deployment size by 90%
> - CloudWatch Logs auto-delete after 7 days

---


### 6. Observability Stack (Phase 7)
**Purpose**: Monitoring, logging, and alerting

**Main resources**:
- **CloudWatch Dashboards** (4 dashboards):
  - **Core Dashboard**: 
    - DynamoDB metrics: Read/write capacity, throttling, latency
    - S3 metrics: Request count, 4xx/5xx errors, bytes downloaded
    - CloudFront metrics: Request count, cache hit rate, error rate
    - KMS metrics: Key usage, encryption/decryption operations
  - **Auth Dashboard**: 
    - Cognito metrics: Sign-ups, sign-ins, failed authentications
    - Lambda trigger metrics: Invocations, errors, duration
    - User pool analytics
  - **Backend Dashboard**: 
    - API Gateway metrics: Request count, 4xx/5xx errors, latency
    - Lambda metrics: Invocations, errors, duration, throttles, concurrent executions
    - SQS metrics: Messages sent, received, deleted, DLQ depth
    - WAF metrics: Blocked requests, allowed requests, rule matches
  - **Overview Dashboard**: 
    - System health summary
    - Cost tracking
    - Composite alarm status
    - Key metrics from all stacks
- **CloudWatch Alarms** (15+ alarms):
  - **Critical Alarms** (notify immediately):
    - API Gateway 5xx errors > 5% (system errors)
    - Lambda error rate > 10%
    - DynamoDB read/write throttling
    - SQS DLQ has messages (failed processing)
    - S3 5xx errors
    - Cost > $50/month (budget exceeded)
  - **Warning Alarms** (notify but not urgent):
    - API Gateway 4xx errors > 10% (client errors)
    - Lambda duration > 25s (approaching timeout)
    - Lambda throttling (concurrent limit reached)
    - DynamoDB latency high (> 100ms)
    - S3 4xx errors
    - SQS queue age > 15 minutes (processing lag)
    - Cost > $35/month (budget warning)
  - All alarms:
    - Evaluation periods: 1-2 periods
    - Datapoints to alarm: Configurable
    - Treatment of missing data: NOT_BREACHING
- **Composite Alarm**: 
  - Name: "SystemHealth"
  - Combines all critical alarms
  - Alarm rule: OR logic (any critical alarm triggers)
  - Actions: Send SNS notification
  - Purpose: Single alarm for overall system health
- **SNS Topic**:
  - Topic name: `EveryoneCook-{env}-Alarms`
  - Email subscription: Admin email from config
  - Delivery status logging enabled
  - Encryption: AWS managed key
- **CloudWatch Logs**:
  - Retention: 7 days (cost optimization)
  - Encryption: KMS DynamoDB key
  - Log groups: API Gateway, Lambda functions (all have logs)
- **Cost Tracking**:
  - CloudWatch metric: EstimatedCharges
  - Alarms: Warning ($35) + Critical ($50)
  - Daily cost monitoring

**Dependencies**: All other stacks (monitors the entire infrastructure)

**Estimated cost**: ~$3-8/month

> **Best practice**: 
> - Deploy this stack last for complete visibility
> - Composite alarm helps reduce alarm fatigue
> - 7-day log retention balances debugging needs and cost
> - Cost alarms prevent unexpected bills

---


### Stack Dependencies Flow

```
DNS Stack (Route 53)
  ↓
Certificate Stack (ACM in us-east-1)
  ↓
Core Stack (DynamoDB, S3, CloudFront, KMS)
  ↓
Auth Stack (Cognito, Lambda Triggers)
  ↓
Backend Stack (API Gateway, Lambda, SQS, WAF)
  ↓
Observability Stack (CloudWatch, Alarms)
```

**Dependency explanation**:
1. **Certificate Stack** needs DNS Stack to validate certificates via Route 53 DNS
2. **Core Stack** needs Certificate Stack to attach certificate to CloudFront
3. **Auth Stack** needs Core Stack so Lambda triggers can access DynamoDB
4. **Backend Stack** needs Auth Stack to integrate Cognito Authorizer into API Gateway
5. **Observability Stack** needs all stacks to monitor all resources


### Total Estimated Cost

**Development Environment**:
| Service | Cost/month | Notes |
|---------|------------|-------|
| Route 53 Hosted Zone | $0.50 | 1 hosted zone + DNS queries |
| ACM Certificates | $0 | Free for public certificates |
| DynamoDB (Pay-per-request) | $3-5 | Depends on usage, free tier available |
| S3 (Intelligent-Tiering) | $1-3 | 2 buckets, auto-tiering saves cost |
| CloudFront | $2-5 | CDN distribution + data transfer |
| Lambda | $0-3 | 7 functions + 1 worker, free tier 1M requests |
| Lambda Layer | $0 | No additional charge |
| API Gateway | $0-3 | REST API, free tier 1M requests |
| Cognito (Free tier) | $0 | Up to 50,000 MAU |
| SQS | $0-1 | 4 queues + 4 DLQs, free tier 1M requests |
| WAF (API Gateway only) | $5-8 | Web ACL + rules + requests processed |
| CloudWatch Logs | $1-2 | 7-day retention, all services |
| CloudWatch Dashboards | $0-1 | 4 dashboards, first 3 free |
| CloudWatch Alarms | $0.50-1 | 15+ alarms, $0.10/alarm |
| SNS | $0 | Email notifications, low volume |
| KMS | $2 | 2 customer-managed keys @ $1 each |
| IAM | $0 | Roles & policies are free |
| **TOTAL** | **$15-35/month** | Development with low traffic |

> **Note**: 
> - This is an estimate for the development environment with low traffic
> - Production environment with high traffic will have significantly higher costs
> - Free tier: Cognito (50K MAU), Lambda (1M requests), API Gateway (1M requests), SQS (1M requests)
> - Cost optimizations applied:
>   - CloudFront WAF removed (-$9/month)
>   - S3 Intelligent-Tiering (auto cost reduction)
>   - CloudWatch Logs 7-day retention
>   - Lambda Layer reduces deployment costs

---

authStack.addDependency(coreStack);
backendStack.addDependency(authStack);
observabilityStack.addDependency(backendStack);

## Configuration Guide

### Step 1: Prepare Infrastructure Directory

**1. Navigate to infrastructure directory**

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

**2. Install dependencies**

```powershell
npm install
```

> **TODO**: Take a screenshot of the terminal with the output of `npm install`

### Step 2: Configure Environment Settings

**1. Review environment configuration**

Open the file `config/environment.ts` to review the configuration:

```powershell
code config\environment.ts
```

This file contains configuration for environments (dev, staging, prod):

```typescript
// Example Dev environment configuration
dev: {
  environment: 'dev',
  account: '123456789012',        // Update with your AWS Account ID
  region: 'ap-southeast-1',        // Singapore region
  domains: {
    frontend: 'dev.everyonecook.cloud',
    api: 'api-dev.everyonecook.cloud',
    cdn: 'cdn-dev.everyonecook.cloud',
  },
  cognito: {
    passwordPolicy: {
      minLength: 8,  // Dev: 8 chars, Prod: 12 chars
    }
  }
}
```

**2. Verify AWS Account ID**

```powershell
# Check current AWS Account ID
aws sts get-caller-identity
```

Output will show:
```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

**3. Update account ID in config (if needed)**

If the account ID does not match, update it in `config/environment.ts`:

```typescript
dev: {
  account: 'YOUR_ACTUAL_ACCOUNT_ID',  // Update here
  // ... other configs
}
```

### Step 3: Review CDK App Structure

**1. Review main CDK app file**

```powershell
code bin\app.ts
```

The file `bin/app.ts` is the entry point of the CDK application. Main content:

```typescript
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { getConfig } from '../config/environment';

const app = new cdk.App();

// Get environment from context (default: 'dev')
const environment = app.node.tryGetContext('environment') || 'dev';
const config = getConfig(environment);

console.log(`🚀 Deploying Everyone Cook infrastructure for environment: ${environment}`);

// Create stacks in dependency order
const dnsStack = new DnsStack(app, `EveryoneCook-${environment}-DNS`, {...});
const certificateStack = new CertificateStack(app, `EveryoneCook-${environment}-Certificate`, {...});
const coreStack = new CoreStack(app, `EveryoneCook-${environment}-Core`, {...});
const authStack = new AuthStack(app, `EveryoneCook-${environment}-Auth`, {...});
const backendStack = new BackendStack(app, `EveryoneCook-${environment}-Backend`, {...});
const observabilityStack = new ObservabilityStack(app, `EveryoneCook-${environment}-Observability`, {...});

// Add dependencies
certificateStack.addDependency(dnsStack);
coreStack.addDependency(certificateStack);
authStack.addDependency(coreStack);
backendStack.addDependency(authStack);
observabilityStack.addDependency(backendStack);

// Add tags to all stacks
cdk.Tags.of(app).add('Project', 'EveryoneCook');
cdk.Tags.of(app).add('Environment', config.environment);
cdk.Tags.of(app).add('ManagedBy', 'CDK');
```

**2. Understand stack dependencies**

Stacks are created in order and have explicit dependencies:
- **Certificate Stack** depends on **DNS Stack**
- **Core Stack** depends on **Certificate Stack**
- **Auth Stack** depends on **Core Stack**
- **Backend Stack** depends on **Auth Stack**
- **Observability Stack** depends on all other stacks

### Step 4: Validate Configuration

**1. Compile TypeScript**

```powershell
# Navigate to infrastructure directory
cd D:\Project_AWS\everyonecook\infrastructure

# Compile TypeScript
npm run build
```

Successful output:
```
> everyonecook-infrastructure@1.0.0 build
> tsc

# No errors - compilation successful
```

**2. List all CDK stacks**

```powershell
# List all stacks for dev environment
npx cdk list --context environment=dev
```

Output:
```
EveryoneCook-dev-DNS
EveryoneCook-dev-Certificate
EveryoneCook-dev-Core
EveryoneCook-dev-Auth
EveryoneCook-dev-Backend
EveryoneCook-dev-Observability
```

**3. Synthesize CloudFormation templates**

```powershell
# Generate CloudFormation templates
npx cdk synth --context environment=dev
```

Output:
```
Successfully synthesized to D:\Project_AWS\everyonecook\infrastructure\cdk.out
Supply a stack id (EveryoneCook-dev-DNS, EveryoneCook-dev-Certificate, ...) to display its template.
```

The `cdk.out/` folder is created with CloudFormation templates:

```
cdk.out/
├── EveryoneCook-dev-DNS.template.json
├── EveryoneCook-dev-Certificate.template.json
├── EveryoneCook-dev-Core.template.json
├── EveryoneCook-dev-Auth.template.json
├── EveryoneCook-dev-Backend.template.json
└── EveryoneCook-dev-Observability.template.json
```

### Step 5: Review Stack Resources (Optional)

If you want to see details of resources in each stack:

**1. DNS Stack Resources**

```powershell
# View DNS stack template
Get-Content cdk.out\EveryoneCook-dev-DNS.template.json | ConvertFrom-Json | Select-Object -ExpandProperty Resources
```

Resources:
- **HostedZone**: Route 53 Hosted Zone
- **Outputs**: Nameservers, Hosted Zone ID

**2. Certificate Stack Resources**

```powershell
# View Certificate stack template
Get-Content cdk.out\EveryoneCook-dev-Certificate.template.json | ConvertFrom-Json | Select-Object -ExpandProperty Resources
```

Resources:
- **CloudFrontCertificate**: ACM Certificate for CloudFront (`cdn.everyonecook.cloud`)
- **ApiGatewayCertificate**: Wildcard ACM Certificate (`*.everyonecook.cloud`)
- **ValidationRecords**: Route 53 DNS validation records

**3. Core Stack Resources**

Resources (30+ resources):
- **DynamoDB Table** with 5 GSI indexes
- **S3 Buckets** (2 buckets: content, cdn-logs)
- **CloudFront Distribution** with OAC
- **KMS Keys** (2 keys: DynamoDB, S3)
- **IAM Roles** and Policies

**4. Auth Stack Resources**

Resources (20+ resources):
- **Cognito User Pool** with password policy
- **Cognito User Pool Client**
- **Lambda Functions** (5 triggers)
- **IAM Roles** for Lambda functions

**5. Backend Stack Resources**

Resources (50+ resources):
- **API Gateway REST API** with custom domain
- **Lambda Functions** (6 modules + 1 worker)
- **SQS Queues** (4 queues + 4 DLQs)
- **WAF Web ACL** for API Gateway
- **Lambda Layer** (shared dependencies)
- **IAM Roles** and Policies

**6. Observability Stack Resources**

Resources (15+ resources):
- **CloudWatch Dashboards** (4 dashboards)
- **CloudWatch Alarms** (10+ alarms)
- **Composite Alarm** for overall health
- **SNS Topic** for notifications

### Step 6: Detailed Configuration for Each Stack

To learn more about the configuration and resources of each stack, see the following sections:

- **[5.4.1 DNS Stack](./5.4.1-DNS-Stack/)**: Route 53 Hosted Zone configuration
- **[5.4.2 Certificate Stack](./5.4.2-Certificate-Stack/)**: ACM Certificates configuration
- **[5.4.3 Core Stack](./5.4.3-Core-Stack/)**: DynamoDB, S3, CloudFront configuration
- **[5.4.4 Auth Stack](./5.4.4-Auth-Stack/)**: Cognito, Lambda Triggers configuration
- **[5.4.5 Backend Stack](./5.4.5-Backend-Stack/)**: API Gateway, Lambda, SQS configuration
- **[5.4.7 Observability Stack](./5.4.7-Observability-Stack/)**: CloudWatch, Alarms configuration

---

## Configuration Checklist

Before deploying, check the following items:

- [ ] **Environment Configuration**
  - [ ] AWS Account ID has been verified and updated in `config/environment.ts`
  - [ ] Region is set correctly (ap-southeast-1 for dev)
  - [ ] Domain names configured correctly
  
- [ ] **Dependencies**
  - [ ] Node.js and npm have been installed
  - [ ] AWS CLI has been configured with credentials
  - [ ] AWS CDK CLI has been installed globally
  - [ ] npm dependencies have been installed (`npm install`)

- [ ] **Validation**
  - [ ] TypeScript compilation successful (`npm run build`)
  - [ ] CDK list shows all 6 stacks
  - [ ] CDK synth generates CloudFormation templates successfully
  - [ ] No syntax errors in stack code

- [ ] **Preparation for Deployment**
  - [ ] CDK bootstrap has been run (see [5.03 CDK Bootstrap](../5.03-cdk-bootstrap/))
  - [ ] AWS account has sufficient permissions to create resources
  - [ ] Domain has been registered (or ready to register)

---

## Next Steps

After completing configuration and validation, continue to:

**[5.05 Deploy Infrastructure](../5.05-deploy-infrastructure/)** - Deploy all stacks to AWS

In the next step, you will:
1. Deploy DNS Stack and configure nameservers
2. Deploy Certificate Stack and validate certificates
3. Deploy Core Stack (DynamoDB, S3, CloudFront)
4. Deploy Auth Stack (Cognito, Lambda Triggers)
5. Deploy Backend Stack (API Gateway, Lambda Functions)
6. Deploy Observability Stack (CloudWatch Dashboards)
