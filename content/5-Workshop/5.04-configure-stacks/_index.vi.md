---
title: "Cấu hình các Infrastructure Stack"
date: 2025-12-01
weight: 4
chapter: false
pre: " <b> 5.04. </b> "
---

# Cấu hình các Infrastructure Stack


## Tổng quan dự án EveryoneCook

### Giới thiệu
EveryoneCook là một nền tảng mạng xã hội chia sẻ công thức nấu ăn, được xây dựng hoàn toàn trên AWS Cloud. Dự án sử dụng kiến trúc serverless, tận dụng các dịch vụ quản lý của AWS để đảm bảo khả năng mở rộng, bảo mật và tối ưu hóa chi phí.

### Kiến trúc hệ thống

Dự án được thiết kế với **Kiến trúc 5-Stack** với các lớp rõ ràng:

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

### Công nghệ sử dụng

#### Infrastructure as Code
- **AWS CDK (TypeScript)**: Quản lý hạ tầng bằng code
- **CloudFormation**: Engine template cơ sở cho CDK
- **Git**: Quản lý phiên bản cho infrastructure code

#### Backend Services
- **API Gateway REST API**: RESTful API endpoint với custom domain
- **Lambda Functions**: Serverless compute cho business logic (6 hàm chính + 1 worker)
- **Lambda Layers**: Lớp shared dependencies (giảm kích thước deployment 90%)
- **DynamoDB**: NoSQL database với Single Table Design + 5 GSI indexes
- **S3**: Object storage cho user content (avatars, posts, recipes, backgrounds)
- **SQS**: Message queue cho async processing (4 queues + 4 DLQs)

#### Security & Authentication
- **Cognito User Pool**: User authentication & management
- **Cognito User Pool Client**: Frontend authentication config
- **WAF (Web Application Firewall)**: API Gateway protection với rate limiting
- **KMS (Key Management Service)**: 2 customer-managed keys (DynamoDB, S3)
- **IAM Roles & Policies**: Fine-grained access control cho tất cả services

#### Content Delivery & Networking
- **CloudFront**: CDN với Origin Access Control (OAC) + Shield Standard
- **Route 53**: DNS management với Hosted Zone
- **ACM (Certificate Manager)**: SSL/TLS certificates (2 certificates)

#### Monitoring & Operations
- **CloudWatch Logs**: Centralized logging cho Lambda functions
- **CloudWatch Metrics**: Performance metrics tracking
- **CloudWatch Dashboards**: 4 dashboards (Core, Auth, Backend, Overview)
- **CloudWatch Alarms**: Real-time monitoring (10+ alarms) + Composite Alarm
- **SNS (Simple Notification Service)**: Email notifications cho alarms

### Cấu trúc thư mục dự án

```
everyonecook/
├── infrastructure/                    # AWS CDK Infrastructure
│   ├── bin/
│   │   └── app.ts                    # CDK app entry point - tạo tất cả stacks
│   ├── lib/
│   │   ├── base-stack.ts             # Base class cho tất cả stacks
│   │   ├── stacks/                   # Stack definitions
│   │   │   ├── dns-stack.ts          # Route 53 Hosted Zone
│   │   │   ├── certificate-stack.ts  # ACM Certificates
│   │   │   ├── core-stack.ts         # DynamoDB, S3, CloudFront
│   │   │   ├── auth-stack.ts         # Cognito, Lambda triggers
│   │   │   ├── backend-stack.ts      # API Gateway, Lambda, SQS
│   │   │   └── observability-stack.ts # CloudWatch, Alarms
│   │   └── constructs/               # Reusable CDK constructs
│   │       └── shared-layer.ts       # Lambda Layer với dependencies
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

## Tổng quan kiến trúc Stack

### 1. DNS Stack (Phase 1)
**Mục đích**: Tạo nền tảng cho DNS management


**Tài nguyên chính**:
- Route 53 Hosted Zone cho domain `everyonecook.cloud`
- Export nameservers để cấu hình tại domain registrar

**Thứ tự triển khai**: Stack này phải được triển khai đầu tiên

**Chi phí ước tính**: ~$0.50/tháng

> **Lưu ý**: Sau khi triển khai stack này, cập nhật nameservers tại Hostinger để trỏ về Route 53.

---


### 2. Certificate Stack (Phase 1.5)
**Mục đích**: Tạo SSL/TLS certificates cho CloudFront và API Gateway

**Tài nguyên chính**:
- ACM Certificate cho CloudFront: `cdn.everyonecook.cloud`
- ACM Wildcard Certificate cho API Gateway: `*.everyonecook.cloud`
- DNS validation records trong Route 53

**Region đặc biệt**: **BẮT BUỘC triển khai tại us-east-1** (yêu cầu của CloudFront)

**Phụ thuộc**: DNS Stack (cần Hosted Zone cho DNS validation)

**Chi phí ước tính**: Miễn phí (ACM certificates miễn phí)

> **Lưu ý quan trọng**: CloudFront chỉ chấp nhận certificates từ region us-east-1.

---


### 3. Core Stack (Phase 2)
**Mục đích**: Tạo lớp dữ liệu và storage infrastructure

**Tài nguyên chính**:
- **DynamoDB Table**: Single Table Design với 5 GSI indexes
  - PK: `USER#{username}`, SK: `PROFILE|RECIPE#{id}|POST#{id}|COMMENT#{id}`
  - Username-based primary key (immutable)
  - GSI1-GSI5 cho các query patterns khác nhau:
    - GSI1: User recipes theo date
    - GSI2: User posts theo date
    - GSI3: Post comments
    - GSI4: Recipe search theo cuisine/difficulty
    - GSI5: Social interactions (likes, follows)
  - Billing mode: Pay-per-request (on-demand)
  - Encryption: Customer-managed KMS key với auto-rotation
- **S3 Buckets** (2 buckets):
  - **Content Bucket**: User uploads (avatars, posts, recipes, backgrounds)
    - Intelligent-Tiering enabled (tự động tối ưu chi phí)
    - Versioning enabled
    - Lifecycle rules: Xóa incomplete multipart uploads sau 7 ngày
    - Encryption: Customer-managed KMS key
  - **CDN Logs Bucket**: CloudFront access logs
    - Intelligent-Tiering enabled
    - Tự động xóa logs sau 90 ngày (tối ưu chi phí)
- **CloudFront Distribution**:
  - Origin: S3 Content Bucket
  - Origin Access Control (OAC) để bảo mật S3 access (thay thế OAI)
  - Custom domain: `cdn.everyonecook.cloud`
  - SSL Certificate: ACM certificate từ Certificate Stack
  - Cache behaviors: Tối ưu cho images & static content
  - Compression: Gzip & Brotli enabled
  - Shield Standard: DDoS protection (miễn phí, tự động bật)
  - **LƯU Ý**: CloudFront WAF đã được gỡ bỏ để tiết kiệm $9/tháng
- **KMS Keys** (2 customer-managed keys):
  - **DynamoDB Encryption Key**: 
    - Auto-rotation enabled (hàng năm)
    - Sử dụng bởi: DynamoDB table, CloudWatch Logs
  - **S3 Encryption Key**: 
    - Auto-rotation enabled (hàng năm)
    - Sử dụng bởi: S3 buckets, CloudFront signed URLs
- **IAM Roles**: 
  - CloudFront Origin Access Control role
  - Lambda execution roles với DynamoDB & S3 access

**Phụ thuộc**: Certificate Stack (cần certificate cho CloudFront)

**Chi phí ước tính**: ~$8-15/tháng

> **Tối ưu chi phí**: 
> - S3 Intelligent-Tiering tự động chuyển objects sang storage rẻ hơn
> - CloudFront WAF đã gỡ bỏ (Shield Standard cung cấp DDoS protection)
> - CloudWatch Logs tự động xóa sau retention period

---


### 4. Auth Stack (Phase 3)
**Mục đích**: Authentication và user management

**Tài nguyên chính**:
- **Cognito User Pool**:
  - Phương thức đăng nhập: Username HOẶC Email + Password
  - KHÔNG yêu cầu MFA (chỉ email + password)
  - Password policy: 
    - Min 12 ký tự cho prod (8 ký tự cho dev)
    - Yêu cầu uppercase, lowercase, digits, symbols
  - Standard attributes: email, given_name (fullName), birthdate, gender
  - Custom attributes: account_status (cho admin ban), country (ISO 3166-1)
  - Device tracking: Enabled (KHÔNG yêu cầu challenge)
  - Email verification: Bắt buộc trước khi sử dụng
  - Account recovery: Dựa trên Email
  - **LƯU Ý**: Advanced Security Mode TẮT để tiết kiệm ~$5/tháng
- **Lambda Triggers** (5 functions với CloudWatch Logs):
  - **Pre-signup**: 
    - Validate username uniqueness
    - Kiểm tra profanity/banned words
    - Runtime: Node.js 20.x
  - **Post-confirmation**: 
    - Tạo user profile trong DynamoDB
    - Initialize user stats
  - **Pre-authentication**: 
    - Kiểm tra account status (active/banned)
    - Log login attempts
  - **Post-authentication**: 
    - Cập nhật last login timestamp
    - Track user activity
  - **Custom message**: 
    - Customize verification emails
    - Branded email templates
- **Cognito User Pool Client**: 
  - OAuth flows: Authorization code grant
  - Token validity: 
    - Access token: 1 giờ
    - Refresh token: 30 ngày
  - Read/Write attributes được cấu hình
- **CloudWatch Log Groups**: 
  - Mỗi Lambda trigger có dedicated log group
  - Retention: 7 ngày
  - Encryption: KMS DynamoDB key
- **IAM Roles**: 
  - Lambda execution roles
  - Permissions: DynamoDB read/write, Cognito admin, Logs write

**Phụ thuộc**: Core Stack (Lambda triggers cần access DynamoDB)

**Chi phí ước tính**: ~$0-2/tháng (Cognito free tier: 50,000 MAU)

> **Bảo mật**: 
> - Advanced Security Mode không bật để tối ưu chi phí (sẽ tốn thêm ~$5/tháng)
> - Device tracking enabled nhưng KHÔNG MFA để cải thiện user experience
> - Tất cả Lambda triggers có CloudWatch logging

---


### 5. Backend Stack (Phase 4)
**Mục đích**: Application layer và business logic

**Tài nguyên chính**:
- **API Gateway REST API**:
  - Custom domain: `api.everyonecook.cloud`
  - SSL Certificate: Wildcard ACM certificate (`*.everyonecook.cloud`)
  - Cognito Authorizer: Validate JWT tokens
  - Request validators: 
    - Body validator: Validate request body
    - Params validator: Validate query params & path params
    - Full validator: Validate cả body & params
  - Deployment stage: dev/staging/prod
  - CloudWatch Logs: Access logs + execution logs
  - WAF Web ACL attached (API protection)
- **Lambda Functions** (6 main modules + CloudWatch Logs):
  - **API Router**: 
    - Route requests tới appropriate handlers
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
    - CRUD operations cho recipes
    - AI recipe generation (Bedrock integration)
    - Recipe search & recommendations
    - Memory: 1024 MB, Timeout: 60s
  - **Admin**: 
    - User management, ban/unban
    - Content moderation
    - Analytics dashboard
    - Memory: 512 MB, Timeout: 30s
  - **Upload**: 
    - File upload lên S3
    - Generate pre-signed URLs
    - Image validation
    - Memory: 512 MB, Timeout: 30s
  - Runtime: Node.js 20.x
  - Environment variables: DynamoDB table, S3 bucket, SQS queues
- **Lambda Layer - Shared Dependencies**:
  - AWS SDK v3 clients: DynamoDB, S3, SQS, Cognito, Bedrock, Lambda
  - Utilities: uuid, jsonwebtoken, jwks-rsa
  - **Lợi ích**: Giảm kích thước deployment từ 8MB → 200KB per function (giảm 90%)
  - Compatible runtimes: Node.js 18.x, 20.x
- **SQS Queues** (4 main queues + 4 DLQs):
  - **AI Queue**: 
    - AI recipe generation requests
    - Visibility timeout: 120s (2 phút)
    - Message retention: 4 ngày
    - Max receive count: 3 (sau đó chuyển sang DLQ)
  - **Image Processing Queue**: 
    - Image optimization, thumbnails
    - Visibility timeout: 60s
    - Message retention: 4 ngày
  - **Analytics Queue**: 
    - User activity tracking
    - Visibility timeout: 30s
    - Message retention: 4 ngày
  - **Notification Queue**: 
    - Push notifications, emails
    - Visibility timeout: 30s
    - Message retention: 4 ngày
  - Encryption: AWS managed KMS keys
- **Worker Lambda** (1 active):
  - **AI Worker**: 
    - Process AI Queue messages
    - Call Amazon Bedrock cho recipe generation
    - Store results trong DynamoDB
    - Memory: 1024 MB, Timeout: 120s
    - Event source: AI Queue (batch size: 10)
  - *Lưu ý: Các workers khác chưa được implement*
- **WAF Web ACL** (chỉ API Gateway):
  - **Rate limiting**: 2000 requests / 5 phút per IP
  - **AWS Managed Rules**:
    - AWSManagedRulesCommonRuleSet (OWASP Top 10)
    - AWSManagedRulesKnownBadInputsRuleSet
  - **Custom rules**: Block specific patterns
  - **LƯU Ý**: CloudFront WAF đã gỡ bỏ để tiết kiệm $9/tháng
  - Shield Standard: Tự động bật (DDoS protection - miễn phí)
- **CloudWatch Log Groups**:
  - API Gateway access logs + execution logs
  - Lambda function logs (per function)
  - Retention: 7 ngày (tối ưu chi phí)
  - Encryption: KMS DynamoDB key
- **IAM Roles & Policies**:
  - API Gateway execution role
  - Lambda execution roles (per function)
  - Permissions: DynamoDB, S3, SQS, Cognito, Bedrock, Logs, KMS

**Shared Lambda Layer**: 
- AWS SDK v3 clients (DynamoDB, S3, SQS, Cognito, Bedrock)
- Common utilities (uuid, jsonwebtoken, jwks-rsa)
- Giảm deployment size: 8MB → 200KB per function

**Phụ thuộc**: Auth Stack (cần Cognito User Pool)

**Chi phí ước tính**: ~$10-25/tháng

> **Tối ưu chi phí**: 
> - CloudFront WAF đã gỡ bỏ (Shield Standard cung cấp DDoS protection)
> - WAF chỉ enabled cho API Gateway (main attack surface)
> - Lambda Layer giảm deployment size 90%
> - CloudWatch Logs tự động xóa sau 7 ngày

---


### 6. Observability Stack (Phase 7)
**Mục đích**: Monitoring, logging và alerting

**Tài nguyên chính**:
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
    - Key metrics từ tất cả stacks
- **CloudWatch Alarms** (15+ alarms):
  - **Critical Alarms** (notify ngay lập tức):
    - API Gateway 5xx errors > 5% (system errors)
    - Lambda error rate > 10%
    - DynamoDB read/write throttling
    - SQS DLQ có messages (failed processing)
    - S3 5xx errors
    - Cost > $50/tháng (vượt ngân sách)
  - **Warning Alarms** (notify nhưng không khẩn cấp):
    - API Gateway 4xx errors > 10% (client errors)
    - Lambda duration > 25s (sắp timeout)
    - Lambda throttling (đạt concurrent limit)
    - DynamoDB latency cao (> 100ms)
    - S3 4xx errors
    - SQS queue age > 15 phút (processing lag)
    - Cost > $35/tháng (cảnh báo ngân sách)
  - Tất cả alarms:
    - Evaluation periods: 1-2 periods
    - Datapoints to alarm: Có thể cấu hình
    - Treatment of missing data: NOT_BREACHING
- **Composite Alarm**: 
  - Name: "SystemHealth"
  - Kết hợp tất cả critical alarms
  - Alarm rule: OR logic (bất kỳ critical alarm nào trigger)
  - Actions: Gửi SNS notification
  - Mục đích: Một alarm duy nhất cho overall system health
- **SNS Topic**:
  - Topic name: `EveryoneCook-{env}-Alarms`
  - Email subscription: Admin email từ config
  - Delivery status logging enabled
  - Encryption: AWS managed key
- **CloudWatch Logs**:
  - Retention: 7 ngày (tối ưu chi phí)
  - Encryption: KMS DynamoDB key
  - Log groups: API Gateway, Lambda functions (tất cả đều có logs)
- **Cost Tracking**:
  - CloudWatch metric: EstimatedCharges
  - Alarms: Warning ($35) + Critical ($50)
  - Daily cost monitoring

**Phụ thuộc**: Tất cả stacks khác (monitor toàn bộ infrastructure)

**Chi phí ước tính**: ~$3-8/tháng

> **Best practice**: 
> - Deploy stack này cuối cùng để có visibility hoàn chỉnh
> - Composite alarm giúp giảm alarm fatigue
> - 7-day log retention cân bằng debugging needs và cost
> - Cost alarms ngăn ngừa hóa đơn bất ngờ

---


### Luồng phụ thuộc Stack

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

**Giải thích phụ thuộc**:
1. **Certificate Stack** cần DNS Stack để validate certificates qua Route 53 DNS
2. **Core Stack** cần Certificate Stack để attach certificate vào CloudFront
3. **Auth Stack** cần Core Stack để Lambda triggers có thể access DynamoDB
4. **Backend Stack** cần Auth Stack để integrate Cognito Authorizer vào API Gateway
5. **Observability Stack** cần tất cả stacks để monitor tất cả resources


### Tổng chi phí ước tính

**Development Environment**:
| Service | Chi phí/tháng | Ghi chú |
|---------|------------|-------|
| Route 53 Hosted Zone | $0.50 | 1 hosted zone + DNS queries |
| ACM Certificates | $0 | Miễn phí cho public certificates |
| DynamoDB (Pay-per-request) | $3-5 | Phụ thuộc usage, có free tier |
| S3 (Intelligent-Tiering) | $1-3 | 2 buckets, auto-tiering tiết kiệm chi phí |
| CloudFront | $2-5 | CDN distribution + data transfer |
| Lambda | $0-3 | 7 functions + 1 worker, free tier 1M requests |
| Lambda Layer | $0 | Không tính phí thêm |
| API Gateway | $0-3 | REST API, free tier 1M requests |
| Cognito (Free tier) | $0 | Lên đến 50,000 MAU |
| SQS | $0-1 | 4 queues + 4 DLQs, free tier 1M requests |
| WAF (chỉ API Gateway) | $5-8 | Web ACL + rules + requests processed |
| CloudWatch Logs | $1-2 | 7-day retention, tất cả services |
| CloudWatch Dashboards | $0-1 | 4 dashboards, 3 đầu tiên miễn phí |
| CloudWatch Alarms | $0.50-1 | 15+ alarms, $0.10/alarm |
| SNS | $0 | Email notifications, volume thấp |
| KMS | $2 | 2 customer-managed keys @ $1 mỗi cái |
| IAM | $0 | Roles & policies miễn phí |
| **TỔNG CỘNG** | **$15-35/tháng** | Development với traffic thấp |

> **Lưu ý**: 
> - Đây là ước tính cho development environment với traffic thấp
> - Production environment với high traffic sẽ có chi phí cao hơn đáng kể
> - Free tier: Cognito (50K MAU), Lambda (1M requests), API Gateway (1M requests), SQS (1M requests)
> - Các tối ưu chi phí đã áp dụng:
>   - CloudFront WAF đã gỡ bỏ (-$9/tháng)
>   - S3 Intelligent-Tiering (tự động giảm chi phí)
>   - CloudWatch Logs 7-day retention
>   - Lambda Layer giảm deployment costs

---

authStack.addDependency(coreStack);
backendStack.addDependency(authStack);
observabilityStack.addDependency(backendStack);

## Hướng dẫn cấu hình

### Bước 1: Chuẩn bị thư mục Infrastructure

**1. Di chuyển đến thư mục infrastructure**

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

**2. Cài đặt dependencies**

```powershell
npm install
```

> **TODO**: Chụp screenshot terminal với output của lệnh `npm install`

### Bước 2: Cấu hình Environment Settings

**1. Xem lại cấu hình environment**

Mở file `config/environment.ts` để xem lại cấu hình:

```powershell
code config\environment.ts
```

File này chứa cấu hình cho các environments (dev, staging, prod):

```typescript
// Ví dụ cấu hình Dev environment
dev: {
  environment: 'dev',
  account: '123456789012',        // Cập nhật với AWS Account ID của bạn
  region: 'ap-southeast-1',        // Singapore region
  domains: {
    frontend: 'dev.everyonecook.cloud',
    api: 'api-dev.everyonecook.cloud',
    cdn: 'cdn-dev.everyonecook.cloud',
  },
  cognito: {
    passwordPolicy: {
      minLength: 8,  // Dev: 8 ký tự, Prod: 12 ký tự
    }
  }
}
```

**2. Xác minh AWS Account ID**

```powershell
# Kiểm tra AWS Account ID hiện tại
aws sts get-caller-identity
```

Output sẽ hiển thị:
```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

**3. Cập nhật account ID trong config (nếu cần)**

Nếu account ID không khớp, cập nhật trong `config/environment.ts`:

```typescript
dev: {
  account: 'YOUR_ACTUAL_ACCOUNT_ID',  // Cập nhật tại đây
  // ... các configs khác
}
```

### Bước 3: Xem lại cấu trúc CDK App

**1. Xem lại file CDK app chính**

```powershell
code bin\app.ts
```

File `bin/app.ts` là entry point của CDK application. Nội dung chính:

```typescript
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { getConfig } from '../config/environment';

const app = new cdk.App();

// Lấy environment từ context (mặc định: 'dev')
const environment = app.node.tryGetContext('environment') || 'dev';
const config = getConfig(environment);

console.log(`🚀 Deploying Everyone Cook infrastructure for environment: ${environment}`);

// Tạo stacks theo thứ tự phụ thuộc
const dnsStack = new DnsStack(app, `EveryoneCook-${environment}-DNS`, {...});
const certificateStack = new CertificateStack(app, `EveryoneCook-${environment}-Certificate`, {...});
const coreStack = new CoreStack(app, `EveryoneCook-${environment}-Core`, {...});
const authStack = new AuthStack(app, `EveryoneCook-${environment}-Auth`, {...});
const backendStack = new BackendStack(app, `EveryoneCook-${environment}-Backend`, {...});
const observabilityStack = new ObservabilityStack(app, `EveryoneCook-${environment}-Observability`, {...});

// Thêm dependencies
certificateStack.addDependency(dnsStack);
coreStack.addDependency(certificateStack);
authStack.addDependency(coreStack);
backendStack.addDependency(authStack);
observabilityStack.addDependency(backendStack);

// Thêm tags cho tất cả stacks
cdk.Tags.of(app).add('Project', 'EveryoneCook');
cdk.Tags.of(app).add('Environment', config.environment);
cdk.Tags.of(app).add('ManagedBy', 'CDK');
```

**2. Hiểu về stack dependencies**

Stacks được tạo theo thứ tự và có explicit dependencies:
- **Certificate Stack** phụ thuộc vào **DNS Stack**
- **Core Stack** phụ thuộc vào **Certificate Stack**
- **Auth Stack** phụ thuộc vào **Core Stack**
- **Backend Stack** phụ thuộc vào **Auth Stack**
- **Observability Stack** phụ thuộc vào tất cả stacks khác

### Bước 4: Validate cấu hình

**1. Compile TypeScript**

```powershell
# Di chuyển đến thư mục infrastructure
cd D:\Project_AWS\everyonecook\infrastructure

# Compile TypeScript
npm run build
```

Output thành công:
```
> everyonecook-infrastructure@1.0.0 build
> tsc

# Không có lỗi - compilation thành công
```

**2. Liệt kê tất cả CDK stacks**

```powershell
# Liệt kê tất cả stacks cho dev environment
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

Thư mục `cdk.out/` được tạo với CloudFormation templates:

```
cdk.out/
├── EveryoneCook-dev-DNS.template.json
├── EveryoneCook-dev-Certificate.template.json
├── EveryoneCook-dev-Core.template.json
├── EveryoneCook-dev-Auth.template.json
├── EveryoneCook-dev-Backend.template.json
└── EveryoneCook-dev-Observability.template.json
```

### Bước 5: Xem lại tài nguyên Stack (Tùy chọn)

Nếu bạn muốn xem chi tiết resources trong mỗi stack:

**1. Tài nguyên DNS Stack**

```powershell
# Xem DNS stack template
Get-Content cdk.out\EveryoneCook-dev-DNS.template.json | ConvertFrom-Json | Select-Object -ExpandProperty Resources
```

Resources:
- **HostedZone**: Route 53 Hosted Zone
- **Outputs**: Nameservers, Hosted Zone ID

**2. Tài nguyên Certificate Stack**

```powershell
# Xem Certificate stack template
Get-Content cdk.out\EveryoneCook-dev-Certificate.template.json | ConvertFrom-Json | Select-Object -ExpandProperty Resources
```

Resources:
- **CloudFrontCertificate**: ACM Certificate cho CloudFront (`cdn.everyonecook.cloud`)
- **ApiGatewayCertificate**: Wildcard ACM Certificate (`*.everyonecook.cloud`)
- **ValidationRecords**: Route 53 DNS validation records

**3. Tài nguyên Core Stack**

Resources (30+ resources):
- **DynamoDB Table** với 5 GSI indexes
- **S3 Buckets** (2 buckets: content, cdn-logs)
- **CloudFront Distribution** với OAC
- **KMS Keys** (2 keys: DynamoDB, S3)
- **IAM Roles** và Policies

**4. Tài nguyên Auth Stack**

Resources (20+ resources):
- **Cognito User Pool** với password policy
- **Cognito User Pool Client**
- **Lambda Functions** (5 triggers)
- **IAM Roles** cho Lambda functions

**5. Tài nguyên Backend Stack**

Resources (50+ resources):
- **API Gateway REST API** với custom domain
- **Lambda Functions** (6 modules + 1 worker)
- **SQS Queues** (4 queues + 4 DLQs)
- **WAF Web ACL** cho API Gateway
- **Lambda Layer** (shared dependencies)
- **IAM Roles** và Policies

**6. Tài nguyên Observability Stack**

Resources (15+ resources):
- **CloudWatch Dashboards** (4 dashboards)
- **CloudWatch Alarms** (10+ alarms)
- **Composite Alarm** cho overall health
- **SNS Topic** cho notifications

### Bước 6: Cấu hình chi tiết cho từng Stack

Để tìm hiểu thêm về cấu hình và resources của từng stack, xem các phần sau:

- **[5.4.1 DNS Stack](./5.4.1-DNS-Stack/)**: Cấu hình Route 53 Hosted Zone
- **[5.4.2 Certificate Stack](./5.4.2-Certificate-Stack/)**: Cấu hình ACM Certificates
- **[5.4.3 Core Stack](./5.4.3-Core-Stack/)**: Cấu hình DynamoDB, S3, CloudFront
- **[5.4.4 Auth Stack](./5.4.4-Auth-Stack/)**: Cấu hình Cognito, Lambda Triggers
- **[5.4.5 Backend Stack](./5.4.5-Backend-Stack/)**: Cấu hình API Gateway, Lambda, SQS
- **[5.4.7 Observability Stack](./5.4.7-Observability-Stack/)**: Cấu hình CloudWatch, Alarms

---

## Checklist cấu hình

Trước khi triển khai, kiểm tra các mục sau:

- [ ] **Cấu hình Environment**
  - [ ] AWS Account ID đã được xác minh và cập nhật trong `config/environment.ts`
  - [ ] Region được set đúng (ap-southeast-1 cho dev)
  - [ ] Domain names được cấu hình đúng
  
- [ ] **Dependencies**
  - [ ] Node.js và npm đã được cài đặt
  - [ ] AWS CLI đã được cấu hình với credentials
  - [ ] AWS CDK CLI đã được cài đặt globally
  - [ ] npm dependencies đã được cài đặt (`npm install`)

- [ ] **Validation**
  - [ ] TypeScript compilation thành công (`npm run build`)
  - [ ] CDK list hiển thị tất cả 6 stacks
  - [ ] CDK synth tạo CloudFormation templates thành công
  - [ ] Không có lỗi syntax trong stack code

- [ ] **Chuẩn bị Deployment**
  - [ ] CDK bootstrap đã được chạy (xem [5.03 CDK Bootstrap](../5.03-cdk-bootstrap/))
  - [ ] AWS account có đủ permissions để tạo resources
  - [ ] Domain đã được đăng ký (hoặc sẵn sàng đăng ký)

---

## Bước tiếp theo

Sau khi hoàn thành cấu hình và validation, tiếp tục đến:

**[5.05 Deploy Infrastructure](../5.05-deploy-infrastructure/)** - Triển khai tất cả stacks lên AWS

Trong bước tiếp theo, bạn sẽ:
1. Deploy DNS Stack và cấu hình nameservers
2. Deploy Certificate Stack và validate certificates
3. Deploy Core Stack (DynamoDB, S3, CloudFront)
4. Deploy Auth Stack (Cognito, Lambda Triggers)
5. Deploy Backend Stack (API Gateway, Lambda Functions)
6. Deploy Observability Stack (CloudWatch Dashboards)
