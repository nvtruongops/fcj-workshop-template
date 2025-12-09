---
title: "5.4.5 Backend Stack"
weight: 5
---
---
# Backend Stack - Application Layer

## Tổng Quan

Backend Stack là **tầng ứng dụng Phase 4** của hạ tầng EveryoneCook. Nó tổng hợp API Gateway, Lambda functions, SQS queues và bảo mật WAF thành một kiến trúc backend thống nhất.

**Thứ Tự Triển Khai**: Stack này **BẮT BUỘC** phải được triển khai sau Core Stack và Auth Stack.

⚠️ **Lưu Ý Environment**: Hướng dẫn này tập trung vào triển khai môi trường **Development (dev)**. Để triển khai staging/production, xem phần [Sự Khác Biệt Giữa Các Environment](#sự-khác-biệt-giữa-các-environment).

### Trách Nhiệm Chính

- Tạo API Gateway REST API với custom domain
- Triển khai **6 Lambda functions** (API Router + 5 business modules)
- Cấu hình **2 SQS queues hoạt động** cho async processing (AI, Image queues với DLQs)
- Triển khai **2 worker Lambda functions** cho event processing (AI Worker, Image Worker)
- Thiết lập WAF Web ACL để bảo vệ API Gateway
- Cấu hình CloudWatch monitoring và alarms

**Những Gì Thực Sự Đang Chạy**:
-  8 Lambda Functions (6 business + 2 workers)
-  4 SQS Queues được triển khai (AI + Image queues với DLQs của chúng)
-  1 WAF Web ACL với 5 quy tắc bảo mật
-  CloudWatch Logs và Alarms

### Những Gì Stack Này Bao Gồm

**API Gateway** (Dev Environment):
- Custom domain: `api-dev.everyonecook.cloud`
- ACM certificate cho HTTPS (Regional - ap-southeast-1)
- Cognito authorizer cho JWT validation
- Caching: **Tắt** (chỉ bật trong production)
- Compression: **Bật** (gzip/deflate cho responses >1KB)
- Rate limiting: 10K req/sec, burst 5K
- Request validation: Body, parameters, headers
- Data trace logging: **Bật** (tắt trong production)

**Lambda Functions (6 functions - Dev Environment)**:
1. **API Router** (`everyonecook-dev-api-router`): Định tuyến requests đến target Lambda functions
2. **Auth & User** (`everyonecook-dev-auth-user`): Xác thực, user profiles, privacy settings
3. **Social** (`everyonecook-dev-social`): Posts, comments, reactions, friends, notifications
4. **Recipe & AI** (`everyonecook-dev-recipe-ai`): Recipe CRUD, AI generation, search, trending
5. **Admin** (`everyonecook-dev-admin`): Content moderation, user management, appeals
6. **Upload** (`everyonecook-dev-upload`): S3 presigned URLs, file uploads

**Source Code**: Tất cả functions nằm trong thư mục `services/`

**SQS Queues (2 queues hoạt động + 2 DLQs)**:
1. **AI Queue** (`everyonecook-dev-ai-queue`): Bedrock AI recipe generation (2-minute timeout) → AI Worker 
2. **Image Queue** (`everyonecook-dev-image-queue`): S3 image optimization (60-second timeout) → Image Worker 

**Lưu ý**: Analytics và Notification queues được định nghĩa trong code nhưng KHÔNG được triển khai. Notifications được xử lý trực tiếp trong Social Module.

**Worker Lambda Functions (2 workers được triển khai hoạt động)**:
1. **AI Worker** (`ai-module/workers/`): Xử lý các công việc AI generation từ AI Queue sử dụng Bedrock Claude 3 Haiku
2. **Image Worker** (`image-worker/`): Xử lý tối ưu hóa hình ảnh từ Image Queue (resize, compress, watermark)

**WAF Web ACL**:
- Rate limiting: 2000 req/5min mỗi IP
- SQL injection protection (AWS Managed)
- XSS protection (AWS Managed)
- Known bad inputs (AWS Managed)
- Request size limit: 10MB tối đa

---

## Kiến Trúc

```
┌──────────────────────────────────────────────────────────────────────┐
│              Backend Stack (Phase 4 - Dev Environment)                │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  WAF Web ACL (REGIONAL) - Bảo vệ API Gateway                    │ │
│  │  ├─ Rate Limiting: 2000 req/5min mỗi IP                        │ │
│  │  ├─ SQL Injection Protection (AWS Managed)                     │ │
│  │  ├─ XSS Protection (AWS Managed)                               │ │
│  │  ├─ Known Bad Inputs (AWS Managed)                             │ │
│  │  └─ Request Size Limit: 10MB tối đa                            │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                          │                                            │
│                          ▼                                            │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  API Gateway REST API                                            │ │
│  │  ├─ Custom Domain: api.everyonecook.cloud                      │ │
│  │  ├─ ACM Certificate (Regional - ap-southeast-1)                │ │
│  │  ├─ Cognito Authorizer (JWT validation)                        │ │
│  │  ├─ Caching: 0.5GB, 5-min TTL (chỉ prod)                      │ │
│  │  ├─ Compression: gzip/deflate (>1KB)                           │ │
│  │  ├─ Rate Limiting: 10K req/sec, burst 5K                       │ │
│  │  └─ Request Validation: Body, params, headers                  │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                          │                                            │
│                          ▼                                            │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Lambda Function: API Router (512MB, 30s timeout)               │ │
│  │  ├─ JWT Validation & User Context Extraction                   │ │
│  │  ├─ Request Routing đến Target Lambda Functions                │ │
│  │  └─ Error Handling & Response Formatting                       │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                          │                                            │
│         ┌────────────────┼────────────────┬──────────────────┐       │
│         ▼                ▼                ▼                  ▼       │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐    │
│  │ Auth &  │      │ Social  │      │ Recipe  │      │ Admin   │    │
│  │ User    │      │ Module  │      │ & AI    │      │ Module  │    │
│  │ Lambda  │      │ Lambda  │      │ Lambda  │      │ Lambda  │    │
│  │ (512MB) │      │ (512MB) │      │ (512MB) │      │ (512MB) │    │
│  └─────────┘      └─────────┘      └─────────┘      └─────────┘    │
│                          │                │                          │
│                          ▼                ▼                          │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  SQS Queues + Worker Lambdas                                    │ │
│  │                                                                  │ │
│  │  1️⃣ AI Queue → AI Worker (1024MB, 60s)                          │ │
│  │     ├─ Bedrock Claude 3 Haiku (us-east-1)                      │ │
│  │     ├─ Tạo recipe từ ingredients                               │ │
│  │     └─ Visibility timeout: 2 phút                              │ │
│  │                                                                  │ │
│  │  2️⃣ Image Queue → Image Worker (512MB, 60s)                     │ │
│  │     ├─ Tối ưu hóa hình ảnh S3                                  │ │
│  │     ├─ Resize, compress, watermark                             │ │
│  │     └─ Visibility timeout: 60 giây                             │ │
│  │                                                                  │ │
│  │  📌 Chỉ 2 queues được sử dụng tích cực (AI & Image)           │ │
│  │  📌 Analytics & Notification queues tồn tại nhưng không có workers │ │
│  │  📌 Tất cả queues có DLQ (14-day retention)                    │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  Dependencies từ Các Stack Khác:                                     │
│  ├─ DynamoDB Table (Core Stack)                                     │
│  ├─ S3 Content Bucket (Core Stack)                                  │
│  ├─ CloudFront Distribution (Core Stack)                            │
│  ├─ Cognito User Pool (Auth Stack)                                  │
│  └─ Cognito User Pool Client (Auth Stack)                           │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Cấu Hình Stack

### Cấu Trúc File

```
infrastructure/lib/stacks/
└── backend-stack.ts          # Backend Stack implementation (2965 dòng)
    ├─ API Gateway configuration
    ├─ Lambda function definitions (6 functions)
    ├─ SQS queue configuration (4 queues + 4 DLQs)
    ├─ Worker Lambda definitions (2 workers hoạt động)
    └─ WAF Web ACL configuration

services/                     # Lambda function source code
├── api-router/               # API Router Lambda
├── auth-module/              # Auth & User Lambda
├── social-module/            # Social Lambda
├── recipe-module/            # Recipe & AI Lambda
├── admin-module/             # Admin Lambda
├── upload-module/            # Upload Lambda
├── ai-module/                # AI Worker Lambda
│   └── workers/
├── image-worker/             # Image Worker Lambda
├── websocket-module/         # WebSocket (stack riêng, không trong Backend)
└── shared/                   # Shared utilities
```

### Triển Khai Code

**File**: `infrastructure/lib/stacks/backend-stack.ts`

#### 1. Stack Interface & Constructor

```typescript
export interface BackendStackProps extends BaseStackProps {
  dynamoTable: cdk.aws_dynamodb.ITable;
  contentBucket: cdk.aws_s3.IBucket;
  distribution: cdk.aws_cloudfront.IDistribution;
  userPool: cdk.aws_cognito.IUserPool;
  userPoolClient: cdk.aws_cognito.IUserPoolClient;
}

export class BackendStack extends BaseStack {
  // Lambda Layer
  public readonly sharedLayer: SharedDependenciesLayer;

  // API Gateway
  public readonly api: cdk.aws_apigateway.RestApi;
  public readonly cognitoAuthorizer: cdk.aws_apigateway.CognitoUserPoolsAuthorizer;
  public readonly apiDomainName: cdk.aws_apigateway.DomainName;

  // Request Validators
  public readonly bodyValidator: cdk.aws_apigateway.RequestValidator;
  public readonly paramsValidator: cdk.aws_apigateway.RequestValidator;
  public readonly fullValidator: cdk.aws_apigateway.RequestValidator;

  // Lambda Functions (6 business functions)
  public readonly apiRouterFunction: cdk.aws_lambda.Function;    // services/api-router
  public readonly authUserFunction: cdk.aws_lambda.Function;     // services/auth-module
  public readonly socialFunction: cdk.aws_lambda.Function;       // services/social-module
  public readonly recipeAIFunction: cdk.aws_lambda.Function;     // services/recipe-module
  public readonly adminFunction: cdk.aws_lambda.Function;        // services/admin-module
  public readonly uploadFunction: cdk.aws_lambda.Function;       // services/upload-module

  // SQS Queues (4 queues + 4 DLQs)
  public readonly aiQueue: cdk.aws_sqs.Queue;
  public readonly imageProcessingQueue: cdk.aws_sqs.Queue;
  public readonly analyticsQueue: cdk.aws_sqs.Queue;
  public readonly notificationQueue: cdk.aws_sqs.Queue;

  // Worker Lambdas (2 workers hoạt động)
  public readonly aiWorker: cdk.aws_lambda.Function;             // services/ai-module/workers
  public readonly imageWorker?: cdk.aws_lambda.Function;         // services/image-worker
  // Lưu ý: analyticsWorker & notificationWorker tồn tại trong code nhưng KHÔNG triển khai
  // public readonly analyticsWorker?: cdk.aws_lambda.Function;  // Đã comment out
  // public readonly notificationWorker?: cdk.aws_lambda.Function; // Không được tạo

  // WAF WebACL
  public readonly apiGatewayWebAcl?: cdk.aws_wafv2.CfnWebACL;

  constructor(scope: Construct, id: string, props: BackendStackProps) {
    super(scope, id, props);

    // Thêm các tag đặc thù cho stack
    cdk.Tags.of(this).add('StackType', 'Backend');
    cdk.Tags.of(this).add('Layer', 'Application');
    cdk.Tags.of(this).add('CostCenter', `Backend-${this.config.environment}`);

    // Tạo resources (xem implementation bên dưới)
    // 1. Shared Dependencies Layer
    // 2. SQS Queues và DLQs
    // 3. API Gateway
    // 4. Lambda Functions
    // 5. Worker Lambdas
    // 6. WAF Web ACL
    // 7. Export outputs
  }
}
```

#### 2. Cấu Hình API Gateway

```typescript
private createAPIGateway(): cdk.aws_apigateway.RestApi {
  const cachingEnabled = this.config.apiGateway.caching.enabled;
  const compressionEnabled = this.config.apiGateway.compression;

  const api = new cdk.aws_apigateway.RestApi(this, 'EveryoneCookAPI', {
    restApiName: `EveryoneCook-API-${this.config.environment}`,
    description: `Everyone Cook REST API - ${this.config.environment}`,

    // Compression cho responses >1KB
    minCompressionSize: compressionEnabled ? cdk.Size.kibibytes(1) : undefined,

    deployOptions: {
      stageName: 'api',

      // Cấu hình caching (chỉ production)
      cachingEnabled: cachingEnabled,
      cacheClusterEnabled: cachingEnabled,
      cacheClusterSize: cachingEnabled ? this.config.apiGateway.caching.cacheSize : undefined,
      cacheTtl: cachingEnabled ? cdk.Duration.seconds(this.config.apiGateway.caching.ttl) : undefined,
      cacheDataEncrypted: cachingEnabled,

      // Throttling settings
      throttlingRateLimit: this.config.apiGateway.throttling.rateLimit,
      throttlingBurstLimit: this.config.apiGateway.throttling.burstLimit,

      // X-Ray Tracing - Tắt (CloudWatch Logs đủ dùng)
      tracingEnabled: false,

      // Cấu hình logging
      loggingLevel: cdk.aws_apigateway.MethodLoggingLevel.INFO,
      dataTraceEnabled: this.config.environment !== 'prod',
      metricsEnabled: true,

      // Access logging
      accessLogDestination: new cdk.aws_apigateway.LogGroupLogDestination(
        new cdk.aws_logs.LogGroup(this, 'APIGatewayAccessLogs', {
          logGroupName: `/aws/apigateway/everyonecook-${this.config.environment}`,
          retention: this.config.cloudwatch.logRetentionDays,
        })
      ),
    },

    // Cấu hình CORS
    defaultCorsPreflightOptions: {
      allowOrigins: [
        `https://${this.config.domains.frontend}`,
        `https://www.${this.config.domains.frontend}`,
        ...(this.config.environment !== 'prod' ? ['http://localhost:3000'] : []),
      ],
      allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
      allowHeaders: [
        'Content-Type',
        'Authorization',
        'X-Amz-Date',
        'X-Api-Key',
        'X-Amz-Security-Token',
        'X-Correlation-Id',
        'Cache-Control',
        'Accept-Encoding',
      ],
      allowCredentials: false,
      maxAge: cdk.Duration.hours(1),
    },

    // Cấu hình endpoint
    endpointConfiguration: {
      types: [cdk.aws_apigateway.EndpointType.REGIONAL],
    },

    // Binary media types
    binaryMediaTypes: ['image/*', 'application/octet-stream'],
  });

  // Thêm Gateway Responses với CORS headers
  const corsHeaders = {
    'gatewayresponse.header.Access-Control-Allow-Origin': "'*'",
    'gatewayresponse.header.Access-Control-Allow-Headers': "'Content-Type,Authorization,...'",
    'gatewayresponse.header.Access-Control-Allow-Methods': "'GET,POST,PUT,DELETE,PATCH,OPTIONS'",
  };

  api.addGatewayResponse('Unauthorized', {
    type: cdk.aws_apigateway.ResponseType.UNAUTHORIZED,
    statusCode: '401',
    responseHeaders: corsHeaders,
  });

  return api;
}
```

#### 3. Cấu Hình SQS Queue

```typescript
private createAIQueue(): cdk.aws_sqs.Queue {
  const queue = new cdk.aws_sqs.Queue(this, 'AIQueue', {
    queueName: `everyonecook-${this.config.environment}-ai-queue`,
    visibilityTimeout: cdk.Duration.seconds(120), // 2 phút cho AI processing
    retentionPeriod: cdk.Duration.days(4),
    deadLetterQueue: {
      queue: this.aiDLQ,
      maxReceiveCount: 3,
    },
    encryption: cdk.aws_sqs.QueueEncryption.KMS_MANAGED,
  });

  // CloudWatch alarm cho queue depth
  if (this.config.cloudwatch.alarms.enabled) {
    new cdk.aws_cloudwatch.Alarm(this, 'AIQueueDepthAlarm', {
      alarmName: `EveryoneCook-${this.config.environment}-AI-Queue-Depth`,
      metric: queue.metricApproximateNumberOfMessagesVisible(),
      threshold: 100,
      evaluationPeriods: 2,
    });
  }

  return queue;
}
```

#### 4. Cấu Hình Lambda Function

```typescript
private createAuthUserLambda(props: BackendStackProps): cdk.aws_lambda.Function {
  const logGroup = new cdk.aws_logs.LogGroup(this, 'AuthUserLogGroup', {
    logGroupName: `/aws/lambda/everyonecook-${this.config.environment}-auth-user`,
    retention: this.config.cloudwatch.logRetentionDays,
  });

  const authUserFunction = new cdk.aws_lambda.Function(this, 'AuthUserFunction', {
    functionName: `everyonecook-${this.config.environment}-auth-user`,
    runtime: cdk.aws_lambda.Runtime.NODEJS_20_X,
    handler: 'services/auth-module/index.handler',
    code: this.createLambdaCode('services/auth-module/deployment'),
    layers: [this.sharedLayer.layer],
    memorySize: 512,
    timeout: cdk.Duration.seconds(30),
    tracing: cdk.aws_lambda.Tracing.DISABLED,
    environment: {
      DYNAMODB_TABLE: props.dynamoTable.tableName,
      USER_POOL_ID: props.userPool.userPoolId,
      USER_POOL_CLIENT_ID: props.userPoolClient.userPoolClientId,
      CONTENT_BUCKET: props.contentBucket.bucketName,
      LOG_LEVEL: this.config.environment === 'prod' ? 'INFO' : 'DEBUG',
    },
    logGroup: logGroup,
  });

  // Cấp permissions
  props.dynamoTable.grantReadWriteData(authUserFunction);
  props.contentBucket.grantReadWrite(authUserFunction);
  props.userPool.grant(authUserFunction, 'cognito-idp:AdminGetUser', 'cognito-idp:ListUsers');

  return authUserFunction;
}
```

#### 5. Cấu Hình WAF Web ACL

```typescript
private createApiGatewayWebAcl(): cdk.aws_wafv2.CfnWebACL {
  const webAcl = new cdk.aws_wafv2.CfnWebACL(this, 'ApiGatewayWebACL', {
    name: `EveryoneCook-API-WAF-${this.config.environment}`,
    scope: 'REGIONAL',
    defaultAction: { allow: {} },
    visibilityConfig: {
      sampledRequestsEnabled: true,
      cloudWatchMetricsEnabled: true,
      metricName: `EveryoneCook-API-WAF-${this.config.environment}`,
    },
    rules: [
      // Rule 1: Rate Limiting
      {
        name: 'RateLimitRule',
        priority: 0,
        statement: {
          rateBasedStatement: {
            limit: 2000, // 2000 requests mỗi 5 phút
            aggregateKeyType: 'IP',
          },
        },
        action: { block: {} },
        visibilityConfig: {
          sampledRequestsEnabled: true,
          cloudWatchMetricsEnabled: true,
          metricName: 'RateLimitRule',
        },
      },
      // Rule 2-5: AWS Managed Rules (SQL injection, XSS, v.v.)
      // ... xem full implementation
    ],
  });

  return webAcl;
}
```

---

## Chi Tiết Cấu Hình Chính

### 1. Shared Dependencies Layer

Backend Stack sử dụng Lambda Layer để chia sẻ common dependencies giữa tất cả Lambda functions:

```typescript
// Tạo Shared Dependencies Layer
this.sharedLayer = new SharedDependenciesLayer(this, 'SharedDependenciesLayer');

// Lợi ích:
// - Giảm 90% kích thước deployment (8MB → 200KB mỗi Lambda)
// - Triển khai nhanh hơn
// - Phiên bản dependency nhất quán
// - Chi phí lưu trữ thấp hơn

// Dependencies bao gồm:
// - AWS SDK v3 clients (DynamoDB, Lambda, Cognito, S3, SQS, Bedrock)
// - uuid, jsonwebtoken, jwks-rsa
```

### 2. API Gateway Custom Domain

Stack tạo custom domain cho API Gateway với ACM certificate:

```typescript
// Cấu hình custom domain
const domainName = this.config.domains.api; // api.everyonecook.cloud

// Tạo ACM certificate trong ap-southeast-1 (Regional endpoint)
const certificate = new acm.Certificate(this, 'ApiGatewayCertificate', {
  domainName: '*.everyonecook.cloud',
  validation: acm.CertificateValidation.fromDns(hostedZone),
});

// Tạo API Gateway DomainName
const apiDomainName = new cdk.aws_apigateway.DomainName(this, 'ApiDomain', {
  domainName: this.config.domains.api,
  certificate: certificate,
  endpointType: cdk.aws_apigateway.EndpointType.REGIONAL,
  securityPolicy: cdk.aws_apigateway.SecurityPolicy.TLS_1_2,
});

// Tạo Route 53 A record (Alias)
new cdk.aws_route53.ARecord(this, 'ApiAliasRecord', {
  zone: hostedZone,
  recordName: this.config.domains.api,
  target: cdk.aws_route53.RecordTarget.fromAlias(
    new cdk.aws_route53_targets.ApiGatewayDomain(apiDomainName)
  ),
});
```

**Environments**:

- **Dev**: `api-dev.everyonecook.cloud`
- **Staging**: `api-staging.everyonecook.cloud`
- **Prod**: `api.everyonecook.cloud`

### 3. Request Validation

Stack triển khai ba cấp độ request validation:

```typescript
// 1. Body Validator - Chỉ validate request body
this.bodyValidator = new cdk.aws_apigateway.RequestValidator(this, 'BodyValidator', {
  restApi: api,
  validateRequestBody: true,
  validateRequestParameters: false,
});

// 2. Params Validator - Chỉ validate query strings và headers
this.paramsValidator = new cdk.aws_apigateway.RequestValidator(this, 'ParamsValidator', {
  restApi: api,
  validateRequestBody: false,
  validateRequestParameters: true,
});

// 3. Full Validator - Validate cả body và parameters
this.fullValidator = new cdk.aws_apigateway.RequestValidator(this, 'FullValidator', {
  restApi: api,
  validateRequestBody: true,
  validateRequestParameters: true,
});

// Lợi ích:
// - Từ chối sớm các request không hợp lệ (trước khi invoke Lambda)
// - Tối ưu chi phí: Không tính phí Lambda cho invalid requests
// - Bảo mật: Ngăn chặn malformed requests
// - Hiệu năng: Phản hồi lỗi nhanh hơn
```

### 4. Quy Ước Đặt Tên Resource

Tất cả resources tuân theo pattern đặt tên nhất quán:

```typescript
// Lambda function format: everyonecook-{env}-{function}
functionName: `everyonecook-${this.config.environment}-auth-user`

// SQS queue format: everyonecook-{env}-{queue}
queueName: `everyonecook-${this.config.environment}-ai-queue`

// API Gateway format: EveryoneCook-API-{env}
restApiName: `EveryoneCook-API-${this.config.environment}`

// WAF format: EveryoneCook-API-WAF-{env}
name: `EveryoneCook-API-WAF-${this.config.environment}`
```

**Ví dụ**:

- Stack name: `EveryoneCook-dev-Backend`
- API: `EveryoneCook-API-dev`
- Lambda: `everyonecook-dev-auth-user`
- Queue: `everyonecook-dev-ai-queue`

### 5. Resource Tags

Mỗi resource được tag để theo dõi chi phí và quản lý:

```typescript
{
  Stack: 'EveryoneCook-dev-Backend',
  Environment: 'dev',
  StackType: 'Backend',
  Layer: 'Application',
  CostCenter: 'Backend-dev',
  Component: 'API' | 'EventProcessing' | 'Security',
  Module: 'AuthUser' | 'Social' | 'RecipeAI' | 'Admin' | 'Upload',
  Purpose: 'REST-API' | 'RequestRouting' | 'AI-Processing',
  Project: 'EveryoneCook'
}
```

---

## Stack Outputs

Sau khi triển khai, stack export các giá trị sau:

| Output Name           | Value                                    | Sử dụng                                   |
| --------------------- | ---------------------------------------- | ----------------------------------------- |
| `ApiUrl`              | `https://api.everyonecook.cloud`         | Frontend API endpoint                     |
| `ApiId`               | `abc123xyz`                              | API Gateway REST API ID                   |
| `ApiStage`            | `api`                                    | API Gateway stage name                    |
| `ApiDomainName`       | `api.everyonecook.cloud`                 | Custom domain name                        |
| `AIQueueUrl`          | `https://sqs.ap-southeast-1...`          | AI Queue URL để gửi messages             |
| `ImageQueueUrl`       | `https://sqs.ap-southeast-1...`          | Image Queue URL                           |
| `WafWebAclArn`        | `arn:aws:wafv2:...`                      | WAF Web ACL ARN                           |

---

## Các Bước Triển Khai

### Bước 1: Xem Lại Cấu Hình

Di chuyển đến thư mục infrastructure:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Xác minh cấu hình môi trường **Development** trong `config/environment.ts`:

```typescript
dev: {
  environment: 'dev',
  account: 'YOUR_AWS_ACCOUNT_ID',
  region: 'ap-southeast-1',
  domains: {
    frontend: 'dev.everyonecook.cloud',
    api: 'api-dev.everyonecook.cloud',
    cdn: 'cdn-dev.everyonecook.cloud',
  },
  apiGateway: {
    caching: {
      enabled: false, //  Tắt trong dev (bật trong prod)
      cacheSize: '0.5', // 0.5GB cache (chỉ prod)
      ttl: 300, // 5 phút (chỉ prod)
    },
    compression: true, //  Bật trong tất cả environments
    throttling: {
      rateLimit: 10000,
      burstLimit: 5000,
    },
  },
  cloudwatch: {
    logRetentionDays: 7, // Dev: 7 ngày, Staging: 30 ngày, Prod: 90 ngày
    alarms: {
      enabled: true,
    },
  },
  // ... các configs khác
}
```

**Cho Staging/Production**: Cập nhật tham số `--context environment=` trong các lệnh triển khai.

### Bước 2: Chuẩn Bị Lambda Deployment Packages

Trước khi triển khai, chuẩn bị Lambda deployment packages cho **8 Lambda functions + 1 Layer**:

```powershell
# 1. Chuẩn bị Shared Layer TRƯỚC TIÊN (bắt buộc cho tất cả Lambdas)
cd D:\Project_AWS\everyonecook\layers\shared-dependencies
.\prepare-layer.ps1

# 2. Chuẩn bị Business Lambda Functions (6 functions)
cd D:\Project_AWS\everyonecook\services\api-router
.\prepare-deployment-layer.ps1  # hoặc prepare-deployment.ps1

cd D:\Project_AWS\everyonecook\services\auth-module
.\prepare-deployment-layer.ps1

cd D:\Project_AWS\everyonecook\services\social-module
.\prepare-deployment-layer.ps1

cd D:\Project_AWS\everyonecook\services\recipe-module
.\prepare-deployment-layer.ps1

cd D:\Project_AWS\everyonecook\services\admin-module
.\prepare-deployment-layer.ps1

cd D:\Project_AWS\everyonecook\services\upload-module
.\prepare-deployment-layer.ps1

# 3. Chuẩn bị Worker Lambda Functions (2 workers)
cd D:\Project_AWS\everyonecook\services\ai-module
.\prepare-deployment.ps1  # AI Worker sử dụng prepare-deployment.ps1

cd D:\Project_AWS\everyonecook\services\image-worker
.\prepare-deployment-layer.ps1
```

**Lưu Ý Quan Trọng**:
- Mỗi service phải có thư mục `deployment/` sau khi chạy prepare script
- Shared Layer phải được build TRƯỚC TIÊN vì tất cả Lambdas phụ thuộc vào nó
- Kiểm tra `prepare-deployment.ps1` hoặc `prepare-deployment-layer.ps1` trong mỗi service folder
- Tổng kích thước deployment: ~8MB → ~200KB mỗi Lambda (nhờ Shared Layer)

### Bước 3: Synthesize CloudFormation Template

Tạo CloudFormation template để xem lại các thay đổi:

```powershell
# Quay về thư mục infrastructure
cd D:\Project_AWS\everyonecook\infrastructure

# Synthesize template
npx cdk synth EveryoneCook-dev-Backend --context environment=dev
```

Kết quả mong đợi hiển thị CloudFormation template với tất cả resources.

### Bước 4: Triển Khai Backend Stack

Triển khai Backend stack lên AWS:

```powershell
# Chỉ triển khai Backend Stack
npx cdk deploy EveryoneCook-dev-Backend --context environment=dev

# Hoặc triển khai với approval
npx cdk deploy EveryoneCook-dev-Backend --context environment=dev --require-approval never
```

Kết quả mong đợi:

```
✨  Synthesis time: 5.23s

EveryoneCook-dev-Backend: deploying...
[0%] start: Publishing abc123:current_account-current_region
[33%] success: Published abc123:current_account-current_region
[33%] start: Publishing def456:current_account-current_region
[66%] success: Published def456:current_account-current_region
[66%] start: Publishing EveryoneCook-dev-Backend
[100%] success: Published EveryoneCook-dev-Backend

EveryoneCook-dev-Backend: creating CloudFormation changeset...

   EveryoneCook-dev-Backend

✨  Deployment time: 420.15s

Outputs:
EveryoneCook-dev-Backend.ApiUrl = https://api-dev.everyonecook.cloud
EveryoneCook-dev-Backend.ApiId = abc123xyz
EveryoneCook-dev-Backend.ApiStage = api
EveryoneCook-dev-Backend.ApiDomainName = api-dev.everyonecook.cloud
EveryoneCook-dev-Backend.AIQueueUrl = https://sqs.ap-southeast-1.amazonaws.com/123456789/everyonecook-dev-ai-queue
EveryoneCook-dev-Backend.ImageQueueUrl = https://sqs.ap-southeast-1.amazonaws.com/123456789/everyonecook-dev-image-queue
EveryoneCook-dev-Backend.WafWebAclArn = arn:aws:wafv2:ap-southeast-1:123456789:regional/webacl/EveryoneCook-API-WAF-dev/...

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:123456789:stack/EveryoneCook-dev-Backend/...

✨  Total time: 425.38s
```

**Thời gian triển khai**: ~7 phút (bao gồm certificate validation)

### Bước 5: Xác Minh Triển Khai trên AWS Console

#### 5.1: Xác Minh API Gateway

1. Điều hướng đến **API Gateway Console**
2. Chọn region: **ap-southeast-1**
3. Click vào **EveryoneCook-API-dev**

![API Gateway Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.5-01-api-gateway-dashboard.png)
*Dashboard API Gateway hiển thị tên API, stage, custom domain, trạng thái caching và throttling settings*

4. Click vào **Stages** → **api**
5. Xác minh stage settings:
   - Cache Settings: Enabled (prod) / Disabled (dev)
   - Cache capacity: 0.5 GB (chỉ prod)
   - Default TTL: 300 seconds (chỉ prod)
   - Throttling: 10000 requests/sec
   - Burst: 5000 requests/sec

![API Gateway Stage Settings](/images/5-Workshop/5.4-configure-stacks/5.4.5-02-api-gateway-stage-settings.png)
*Stage settings hiển thị cấu hình caching và throttling*

6. Click vào **Custom domain names**
7. Xác minh custom domain:
   - Domain name: `api-dev.everyonecook.cloud`
   - Certificate: ACM certificate (Regional)
   - Base path mapping: `/` → `api` stage

![API Gateway Custom Domain](/images/5-Workshop/5.4-configure-stacks/5.4.5-03-api-gateway-custom-domain.png)
*Cấu hình custom domain với ACM certificate*

#### 5.2: Xác Minh Lambda Functions

1. Điều hướng đến **Lambda Console**
2. Chọn region: **ap-southeast-1**
3. Xác minh tất cả 8 Lambda functions tồn tại:

| Tên Function                       | Runtime   | Memory | Timeout | Layer                   | Source Code              |
| ---------------------------------- | --------- | ------ | ------- | ----------------------- | ------------------------ |
| `everyonecook-dev-api-router`      | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/api-router`    |
| `everyonecook-dev-auth-user`       | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/auth-module`   |
| `everyonecook-dev-social`          | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/social-module` |
| `everyonecook-dev-recipe-ai`       | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/recipe-module` |
| `everyonecook-dev-admin`           | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/admin-module`  |
| `everyonecook-dev-upload`          | Node 20.x | 512MB  | 30s     | SharedDependenciesLayer | `services/upload-module` |
| `everyonecook-dev-ai-worker`       | Node 20.x | 1024MB | 60s     | SharedDependenciesLayer | `services/ai-module/workers` |
| `everyonecook-dev-image-worker`    | Node 20.x | 512MB  | 60s     | SharedDependenciesLayer | `services/image-worker`  |

![Lambda Functions List](/images/5-Workshop/5.4-configure-stacks/cognito-triggers.png)
*Danh sách Lambda functions hiển thị tất cả 8 functions với runtime và configuration*

4. Click vào **everyonecook-dev-api-router**
5. Xác minh configuration:
   - Environment variables: `DYNAMODB_TABLE`, `USER_POOL_ID`, v.v.
   - Layers: `SharedDependenciesLayer`
   - Triggers: API Gateway (Proxy integration)

![Lambda API Router Configuration](/images/5-Workshop/5.4-configure-stacks/5.4.5-05-lambda-api-router-config.png)
*API Router function hiển thị environment variables, layers, và API Gateway trigger*

6. Click vào tab **Monitoring**
7. Xác minh CloudWatch Logs integration:
   - Log group: `/aws/lambda/everyonecook-dev-api-router`
   - Retention: Dựa trên environment config

![Lambda Monitoring Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.5-06-lambda-monitoring.png)
*Dashboard monitoring Lambda hiển thị CloudWatch Logs và metrics*

#### 5.3: Xác Minh SQS Queues

1. Điều hướng đến **SQS Console**
2. Chọn region: **ap-southeast-1**
3. Xác minh 4 queues hoạt động tồn tại (2 main + 2 DLQs):

**Lưu ý**: Chỉ AI và Image queues được sử dụng tích cực. Analytics và Notification queues tồn tại trong code nhưng không được triển khai.

| Tên Queue                                | Type | Visibility Timeout | Retention | DLQ                    | Trạng thái Worker |
| ---------------------------------------- | ---- | ------------------ | --------- | ---------------------- | ----------------- |
| `everyonecook-dev-ai-queue`              | Main | 2 phút             | 4 ngày    | everyonecook-dev-ai-dlq |  Hoạt động      |
| `everyonecook-dev-ai-dlq`                | DLQ  | 5 phút             | 14 ngày   | -                      | -                 |
| `everyonecook-dev-image-queue`           | Main | 60 giây            | 4 ngày    | everyonecook-dev-image-dlq |  Hoạt động      |
| `everyonecook-dev-image-dlq`             | DLQ  | 5 phút             | 14 ngày   | -                      | -                 |

![SQS Queues List](/images/5-Workshop/5.4-configure-stacks/5.4.5-07-sqs-queues-list.png)
*Danh sách SQS queues hiển thị 4 queues hoạt động: ai-queue, ai-dlq, image-queue, image-dlq*

4. Click vào **everyonecook-dev-ai-queue**
5. Xác minh queue configuration:
   - Visibility timeout: 2 phút
   - Message retention: 4 ngày
   - Dead-letter queue: `everyonecook-dev-ai-dlq`
   - Maximum receives: 3
   - Encryption: KMS managed

![SQS AI Queue Configuration](/images/5-Workshop/5.4-configure-stacks/5.4.5-08-sqs-ai-queue-config.png)
*Cấu hình AI Queue hiển thị visibility timeout, retention, DLQ, và encryption settings*

6. Click vào tab **Lambda triggers**
7. Xác minh Lambda trigger:
   - Function: `everyonecook-dev-ai-worker`
   - Batch size: 1
   - Batch window: 0 seconds

![SQS AI Queue Lambda Trigger](/images/5-Workshop/5.4-configure-stacks/5.4.5-09-sqs-ai-queue-lambda-trigger.png)
*AI Queue Lambda trigger hiển thị ai-worker function với batch configuration*

#### 5.4: Xác Minh WAF Web ACL

1. Điều hướng đến **WAF & Shield Console**
2. Chọn region: **ap-southeast-1** (Regional)
3. Click vào **Web ACLs**
4. Click vào **EveryoneCook-API-WAF-dev**

![WAF Web ACL Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.5-10-waf-web-acl-dashboard.png)
*Dashboard WAF Web ACL để bảo vệ EveryoneCook API*

5. Click vào tab **Rules**
6. Xác minh tất cả 5 rules tồn tại:

| Priority | Tên Rule                         | Type           | Action | Status  |
| -------- | -------------------------------- | -------------- | ------ | ------- |
| 0        | RateLimitRule                    | Rate-based     | Block  | Enabled |
| 1        | AWSManagedRulesSQLi              | Managed        | Block  | Enabled |
| 2        | AWSManagedRulesKnownBadInputs    | Managed        | Block  | Enabled |
| 3        | AWSManagedRulesCoreRuleSet       | Managed        | Block  | Enabled |
| 4        | RequestSizeLimit                 | Size constraint| Block  | Enabled |


#### 5.5: Xác Minh CloudWatch Alarms

1. Điều hướng đến **CloudWatch Console**
2. Chọn region: **ap-southeast-1**
3. Filter theo: `EveryoneCook-dev-Backend`

![CloudWatch Alarms List](/images/5-Workshop/5.4-configure-stacks/5.4.5-13-cloudwatch-alarms.png)
*CloudWatch alarms cho Backend Stack: Queue depth, Lambda errors/duration, WAF blocks*

#### 5.6: Xác Minh Route 53 DNS Record

1. Điều hướng đến **Route 53 Console**
2. Click vào **Hosted zones**
3. Click vào **everyonecook.cloud**
4. Xác minh A record tồn tại:
   - Name: `api-dev.everyonecook.cloud`
   - Type: A (Alias)
   - Target: API Gateway domain name
   - Routing policy: Simple

![Route 53 API Record](/images/5-Workshop/5.4-configure-stacks/5.4.5-14-route53-api-record.png)
*Route 53 A record (Alias) trỏ đến API Gateway custom domain*

### Bước 6: Kiểm Tra API Endpoint

Kiểm tra custom domain endpoint:

```powershell
# Kiểm tra API Gateway health endpoint (nếu tồn tại)
curl https://api-dev.everyonecook.cloud/health

# Hoặc kiểm tra với browser
start https://api-dev.everyonecook.cloud
```

Phản hồi mong đợi (nếu health endpoint tồn tại):

```json
{
  "status": "healthy",
  "environment": "dev",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## Sự Khác Biệt Giữa Các Environment

Hướng dẫn này sử dụng môi trường **Development (dev)** làm ví dụ. Dưới đây là các khác biệt chính giữa các môi trường:

### Domain Names

| Environment | API Domain                       | Frontend Domain            |
| ----------- | -------------------------------- | -------------------------- |
| **Dev**     | `api-dev.everyonecook.cloud`     | `dev.everyonecook.cloud`   |
| **Staging** | `api-staging.everyonecook.cloud` | `staging.everyonecook.cloud` |
| **Prod**    | `api.everyonecook.cloud`         | `everyonecook.cloud`       |

### Resource Naming

| Loại Resource | Dev                              | Staging                          | Prod                             |
| ------------- | -------------------------------- | -------------------------------- | -------------------------------- |
| Stack Name    | `EveryoneCook-dev-Backend`       | `EveryoneCook-staging-Backend`   | `EveryoneCook-prod-Backend`      |
| API Gateway   | `EveryoneCook-API-dev`           | `EveryoneCook-API-staging`       | `EveryoneCook-API-prod`          |
| Lambda        | `everyonecook-dev-auth-user`     | `everyonecook-staging-auth-user` | `everyonecook-prod-auth-user`    |
| SQS Queue     | `everyonecook-dev-ai-queue`      | `everyonecook-staging-ai-queue`  | `everyonecook-prod-ai-queue`     |
| WAF           | `EveryoneCook-API-WAF-dev`       | `EveryoneCook-API-WAF-staging`   | `EveryoneCook-API-WAF-prod`      |

### Sự Khác Biệt Cấu Hình

| Setting                | Dev                | Staging            | Production         |
| ---------------------- | ------------------ | ------------------ | ------------------ |
| **API Caching**        |  Tắt             |  Tắt             |  Bật (0.5GB)     |
| **Data Trace Logging** |  Bật             |  Bật             |  Tắt             |
| **Log Retention**      | 7 ngày             | 30 ngày            | 90 ngày            |
| **CloudWatch Removal** | DESTROY            | DESTROY            | RETAIN             |
| **WAF Rate Limit**     | 2000 req/5min      | 2000 req/5min      | 5000 req/5min      |
| **Lambda Log Level**   | DEBUG              | INFO               | INFO               |
| **CORS Localhost**     |  Cho phép        |  Không cho phép  |  Không cho phép  |

### Lệnh Triển Khai

```powershell
# Development
npx cdk deploy EveryoneCook-dev-Backend --context environment=dev

# Staging
npx cdk deploy EveryoneCook-staging-Backend --context environment=staging

# Production (yêu cầu approval)
npx cdk deploy EveryoneCook-prod-Backend --context environment=prod --require-approval broadening
```

### Cân Nhắc Đặc Thù Cho Từng Environment

#### Development
-  CORS cho phép `http://localhost:3000` để kiểm tra frontend local
-  Data trace logging bật để debug
-  Log retention thấp hơn (7 ngày) để giảm chi phí
-  Resources bị xóa khi xóa stack (RemovalPolicy.DESTROY)

#### Staging
-  Cấu hình giống production để kiểm tra pre-release
-  Domain riêng để tránh ảnh hưởng production
-  Các quy tắc bảo mật giống production
-  Resources bị xóa khi xóa stack

#### Production
-  API Gateway caching bật (0.5GB, 5-min TTL)
-  Data trace logging tắt (tối ưu chi phí)
-  Log retention 90 ngày để tuân thủ
-  Resources được giữ lại khi xóa stack (RemovalPolicy.RETAIN)
-  WAF rate limits nghiêm ngặt hơn
-  Không có localhost CORS

---

## Phân Tích Chi Phí

### Phân Tích Chi Phí Hàng Tháng (Dev Environment)

| Dịch vụ              | Cấu hình                             | Chi phí hàng tháng |
| -------------------- | ------------------------------------ | ------------------ |
| API Gateway          | 1M requests, caching tắt             | ~$3.50             |
| Lambda (6 functions) | 512MB, 1M invocations, 200ms avg     | ~$8.40             |
| Lambda (2 workers)   | 512-1024MB, 10K invocations          | ~$0.42             |
| SQS (2 queues)       | 10K messages/tháng                   | ~$0.03             |
| WAF Web ACL          | 1 ACL + 5 rules + 1M requests        | ~$10.60            |
| CloudWatch Logs      | 5GB ingestion, 30-day retention      | ~$2.50             |
| CloudWatch Alarms    | 10 alarms                            | ~$1.00             |
| **Tổng**             | **Backend Stack**                    | **~$26.45/tháng**  |

### Ước Tính Chi Phí Production

| Dịch vụ              | Cấu hình                             | Chi phí hàng tháng |
| -------------------- | ------------------------------------ | ------------------ |
| API Gateway          | 10M requests, caching bật (0.5GB)    | ~$50.00            |
| Lambda (6 functions) | 512MB, 10M invocations, 200ms avg    | ~$84.00            |
| Lambda (2 workers)   | 512-1024MB, 100K invocations         | ~$4.20             |
| SQS (2 queues)       | 100K messages/tháng                  | ~$0.25             |
| WAF Web ACL          | 1 ACL + 5 rules + 10M requests       | ~$16.00            |
| CloudWatch Logs      | 20GB ingestion, 90-day retention     | ~$10.00            |
| CloudWatch Alarms    | 15 alarms                            | ~$1.50             |
| **Tổng**             | **Backend Stack**                    | **~$165.95/tháng** |

---

## Các Bước Tiếp Theo

Sau khi triển khai Backend Stack:

1. **Triển khai Frontend Stack** (Phase 5) - Ứng dụng Next.js trên CloudFront
2. **Triển khai Observability Stack** (Phase 6) - CloudWatch dashboards và alarms
3. **Kiểm tra API Integration** - Xác minh tất cả endpoints hoạt động chính xác
4. **Load Testing** - Kiểm tra hiệu năng và khả năng mở rộng
5. **Security Audit** - Xem lại WAF rules và access controls

---

## Tài Liệu Tham Khảo

### Infrastructure Code
- **Backend Stack**: `infrastructure/lib/stacks/backend-stack.ts` (2965 dòng)
- **Shared Layer Construct**: `infrastructure/lib/constructs/shared-layer.ts`

### Lambda Function Source Code (6 Business Functions)
- **API Router**: `services/api-router/` - Request routing & JWT validation
- **Auth User Module**: `services/auth-module/` - Authentication & user profiles
- **Social Module**: `services/social-module/` - Posts, comments, friends, notifications
- **Recipe AI Module**: `services/recipe-module/` - Recipe CRUD, AI generation, search
- **Admin Module**: `services/admin-module/` - Content moderation, user management
- **Upload Module**: `services/upload-module/` - S3 presigned URLs, file uploads

### Worker Lambda Source Code (2 Active Workers)
- **AI Worker**: `services/ai-module/workers/` - Bedrock Claude 3 Haiku integration
- **Image Worker**: `services/image-worker/` - Image optimization (resize, compress)

### Shared Dependencies
- **Shared Layer**: `layers/shared-dependencies/` - AWS SDK v3, uuid, jsonwebtoken, jwks-rsa

### Other Services (Không trong Backend Stack)
- **WebSocket Module**: `services/websocket-module/` - Real-time communication (stack riêng)
- **Shared Utilities**: `services/shared/` - Common utilities across services

---

## Tóm Tắt

Backend Stack là **stack lớn nhất và phức tạp nhất** trong hạ tầng EveryoneCook. Nó tổng hợp:

-  API Gateway với custom domain (caching chỉ trong prod)
-  6 Lambda functions cho business logic
-  2 SQS queues hoạt động cho async processing (tổng 4 với DLQs)
-  2 worker Lambda functions (AI Worker, Image Worker)
-  WAF Web ACL cho bảo mật (5 quy tắc bảo vệ)
-  CloudWatch monitoring và alarms

**Thực Sự Được Triển Khai**: 8 Lambda functions, 4 SQS queues (2 hoạt động với workers), 1 WAF Web ACL, CloudWatch alarms

### Development Environment Checklist

-  Chuẩn bị tất cả Lambda deployment packages (8 functions + 1 layer)
-  Xem lại cấu hình môi trường **dev**
-  Xác minh caching **tắt** (chỉ prod)
-  Kiểm tra từng Lambda function độc lập
-  Theo dõi CloudWatch alarms
-  Xác minh WAF rules không chặn legitimate traffic
-  Kiểm tra localhost CORS (chỉ dev)

### Cho Staging/Production Deployment

-  Xem lại [Sự Khác Biệt Giữa Các Environment](#sự-khác-biệt-giữa-các-environment)
-  Bật API Gateway caching (chỉ prod)
-  Tắt data trace logging (chỉ prod)
-  Thiết lập log retention phù hợp (90 ngày cho prod)
-  Cập nhật WAF rate limits (cao hơn cho prod)
-  Loại bỏ localhost khỏi CORS
-  Sử dụng `--require-approval broadening` cho production

**Thứ Tự Triển Khai**: DNS → Certificate → Core → Auth → **Backend** → Frontend → Observability

**Environments**: Hướng dẫn này minh họa triển khai **dev**. Lặp lại cho **staging** và **prod** với cấu hình phù hợp.
