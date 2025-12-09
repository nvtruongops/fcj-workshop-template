---
title: "5.4.3 Core Stack"
weight: 3
---
---
# Core Stack - Hạ Tầng Data Layer

## Tổng Quan

Core Stack là **tầng nền tảng dữ liệu Phase 2** của dự án EveryoneCook. Nó tạo ra hạ tầng cốt lõi cho lưu trữ dữ liệu, phân phối nội dung và mã hóa mà tất cả các dịch vụ khác phụ thuộc vào.

**Thứ Tự Triển Khai**: Stack này **BẮT BUỘC** phải được triển khai sau Certificate Stack và **trước** Auth Stack và Backend Stack.

### Trách Nhiệm Chính

- Tạo DynamoDB Single Table với 5 GSI indexes cho truy vấn dựa trên username
- Tạo S3 buckets với Intelligent-Tiering để tối ưu chi phí (tiết kiệm 57%)
- Tạo CloudFront CDN distribution với nén và caching
- Tạo KMS encryption keys cho DynamoDB và S3
- Cấu hình Origin Access Control (OAC) cho bảo mật S3
- Thiết lập custom domain với chứng chỉ SSL cho CDN

### Những Gì Stack Này Bao Gồm

**Storage**:
- DynamoDB Single Table với mã hóa at rest
- S3 Content Bucket (avatars, posts, recipes, backgrounds)
- S3 CDN Logs Bucket cho CloudFront access logs

**CDN & Caching**:
- CloudFront Distribution với custom domain
- Compression được bật (Gzip + Brotli, giảm 70-80% kích thước)
- Cache policies (24h mặc định, tối đa 7 ngày)
- Origin Access Control (OAC) cho bảo mật S3

**Security**:
- KMS Customer Managed Keys cho mã hóa
- Automatic key rotation (hàng năm)
- Security headers (HSTS, X-Frame-Options, v.v.)

**Tối Ưu Chi Phí**:
- S3 Intelligent-Tiering (giảm 57% chi phí)
- CloudFront Price Class 200 (giảm 45% chi phí)
- Pay-per-request billing cho dev environment

---

## Kiến Trúc

```
┌─────────────────────────────────────────────────────────────────┐
│                        Core Stack (Phase 2)                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  KMS Encryption Layer                                     │  │
│  │  ├─ DynamoDB Key (automatic rotation)                    │  │
│  │  └─ S3 Key (automatic rotation)                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  DynamoDB Single Table                                    │  │
│  │  ├─ PK: USER#{username}                                  │  │
│  │  ├─ SK: PROFILE|RECIPE#{id}|POST#{id}                   │  │
│  │  ├─ 5 GSI indexes cho truy vấn                          │  │
│  │  ├─ Billing: PAY_PER_REQUEST (dev)                      │  │
│  │  ├─ Encryption: Customer Managed KMS                     │  │
│  │  └─ Streams: Enabled (NEW_AND_OLD_IMAGES)               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  S3 Content Bucket                                        │  │
│  │  ├─ Folders: avatars/, posts/, recipes/, backgrounds/   │  │
│  │  ├─ Intelligent-Tiering (tiết kiệm 57%)                 │  │
│  │  │  └─ Archive: 90 ngày, Deep Archive: 180 ngày        │  │
│  │  ├─ Encryption: S3 Managed Keys                         │  │
│  │  ├─ Versioning: Enabled (prod)                          │  │
│  │  ├─ CORS: Cấu hình cho frontend                         │  │
│  │  └─ OAC: Chỉ CloudFront access                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  CloudFront CDN Distribution                              │  │
│  │  ├─ Custom Domain: cdn.everyonecook.cloud               │  │
│  │  ├─ Certificate: ACM (us-east-1)                        │  │
│  │  ├─ Compression: Gzip + Brotli (giảm 70-80%)            │  │
│  │  ├─ Cache TTL: 24h mặc định, tối đa 7 ngày              │  │
│  │  ├─ Price Class: 200 (US, EU, Asia - tiết kiệm 45%)    │  │
│  │  ├─ Security: HTTPS redirect, Security headers         │  │
│  │  ├─ Protection: Shield Standard (DDoS miễn phí)        │  │
│  │  └─ Logs: S3 CDN Logs Bucket                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Route 53 DNS Record (từ DNS Stack)                      │  │
│  │  cdn.everyonecook.cloud → CloudFront (A + AAAA Alias)   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                          │
                          │ Exports
                          ▼
        ┌─────────────────┴────────────────┐
        ▼                                  ▼
   Auth Stack                         Backend Stack
   (Cognito)                          (API Gateway, Lambda)
```

---

## Cấu Hình Stack

### Cấu Trúc File

```
infrastructure/lib/stacks/
└── core-stack.ts            # Core Stack implementation (1545 dòng)
```

### Điểm Nổi Bật Trong Triển Khai Code

**File**: `infrastructure/lib/stacks/core-stack.ts`

#### 1. Tạo KMS Keys

```typescript
/**
 * Tạo KMS key cho DynamoDB encryption
 */
private createDynamoDBKey(): cdk.aws_kms.Key {
  const key = new cdk.aws_kms.Key(this, 'DynamoDBKey', {
    alias: `everyonecook-dynamodb-${this.config.environment}`,
    description: `KMS key cho DynamoDB encryption trong ${this.config.environment} environment`,
    
    // Bật automatic key rotation (hàng năm)
    enableKeyRotation: true,
    
    // Deletion protection cho production
    pendingWindow: this.config.environment === 'prod' 
      ? cdk.Duration.days(30) 
      : cdk.Duration.days(7),
    
    // Removal policy: RETAIN cho prod, DESTROY cho dev/staging
    removalPolicy: this.config.environment === 'prod'
      ? cdk.RemovalPolicy.RETAIN
      : cdk.RemovalPolicy.DESTROY,
  });
  
  // Thêm key policy cho DynamoDB service
  key.addToResourcePolicy(
    new cdk.aws_iam.PolicyStatement({
      sid: 'Allow DynamoDB to use the key',
      effect: cdk.aws_iam.Effect.ALLOW,
      principals: [new cdk.aws_iam.ServicePrincipal('dynamodb.amazonaws.com')],
      actions: ['kms:Decrypt', 'kms:DescribeKey', 'kms:CreateGrant'],
      resources: ['*'],
      conditions: {
        StringEquals: {
          'kms:ViaService': `dynamodb.${this.region}.amazonaws.com`,
        },
      },
    })
  );
  
  return key;
}
```

#### 2. DynamoDB Single Table

```typescript
/**
 * Tạo DynamoDB Single Table với tối ưu chi phí
 */
private createDynamoDBTable(): cdk.aws_dynamodb.Table {
  const tableName = `EveryoneCook-${this.config.environment}-v2`;
  const dbConfig = this.config.dynamodb;
  
  const table = new cdk.aws_dynamodb.Table(this, 'EveryoneCookTable', {
    tableName,
    partitionKey: { name: 'PK', type: cdk.aws_dynamodb.AttributeType.STRING },
    sortKey: { name: 'SK', type: cdk.aws_dynamodb.AttributeType.STRING },
    
    // Billing mode: PROVISIONED cho prod/staging, PAY_PER_REQUEST cho dev
    billingMode: dbConfig.billingMode === 'PROVISIONED'
      ? cdk.aws_dynamodb.BillingMode.PROVISIONED
      : cdk.aws_dynamodb.BillingMode.PAY_PER_REQUEST,
    
    // Point-in-time recovery cho prod/staging
    pointInTimeRecoverySpecification: {
      pointInTimeRecoveryEnabled: dbConfig.pointInTimeRecovery,
    },
    
    // Deletion protection cho production
    deletionProtection: dbConfig.deletionProtection,
    
    // Bật DynamoDB Streams
    stream: dbConfig.streamEnabled
      ? cdk.aws_dynamodb.StreamViewType.NEW_AND_OLD_IMAGES
      : undefined,
    
    // TTL attribute cho automatic cleanup
    timeToLiveAttribute: 'ttl',
    
    // Encryption với customer managed KMS key
    encryption: cdk.aws_dynamodb.TableEncryption.CUSTOMER_MANAGED,
    encryptionKey: this.dynamoDbKey,
    
    // Removal policy
    removalPolicy: this.config.environment === 'prod'
      ? cdk.RemovalPolicy.RETAIN
      : cdk.RemovalPolicy.DESTROY,
  });
  
  // Thêm 5 GSI indexes (code tiếp tục...)
  
  return table;
}
```

#### 3. S3 Bucket với Intelligent-Tiering

```typescript
/**
 * Tạo S3 bucket với Intelligent-Tiering để tiết kiệm 57% chi phí
 */
private createContentBucket(): cdk.aws_s3.Bucket {
  const bucketName = `everyonecook-content-${this.config.environment}`;
  const s3Config = this.config.s3;
  
  const bucket = new cdk.aws_s3.Bucket(this, 'ContentBucket', {
    bucketName,
    
    // Block ALL public access (private bucket)
    blockPublicAccess: cdk.aws_s3.BlockPublicAccess.BLOCK_ALL,
    
    // Bật versioning cho production
    versioned: s3Config.versioning,
    
    // S3-managed encryption
    encryption: cdk.aws_s3.BucketEncryption.S3_MANAGED,
    
    // Cấu hình Intelligent-Tiering
    intelligentTieringConfigurations: [{
      name: 'EntireBucket',
      archiveAccessTierTime: cdk.Duration.days(90),      // Archive tier
      deepArchiveAccessTierTime: cdk.Duration.days(180), // Deep Archive
    }],
    
    // Lifecycle rules
    lifecycleRules: [
      {
        id: 'DeleteOldVersions',
        enabled: s3Config.versioning,
        noncurrentVersionExpiration: cdk.Duration.days(30),
      },
      {
        id: 'DeleteTempUploads',
        enabled: true,
        prefix: 'posts/temp/',
        expiration: cdk.Duration.days(1), // Dọn dẹp temp files sau 24h
      },
    ],
    
    // CORS cho frontend access
    cors: [{
      allowedOrigins: [
        `https://${this.config.domain.frontend}`,
        ...(this.config.environment === 'dev' ? ['http://localhost:3000'] : []),
      ],
      allowedMethods: [
        cdk.aws_s3.HttpMethods.GET,
        cdk.aws_s3.HttpMethods.PUT,
        cdk.aws_s3.HttpMethods.POST,
        cdk.aws_s3.HttpMethods.DELETE,
      ],
      allowedHeaders: ['*'],
      exposedHeaders: ['ETag'],
      maxAge: 3000,
    }],
    
    removalPolicy: this.config.environment === 'prod'
      ? cdk.RemovalPolicy.RETAIN
      : cdk.RemovalPolicy.DESTROY,
  });
  
  return bucket;
}
```

#### 4. CloudFront Distribution

```typescript
/**
 * Tạo CloudFront distribution với compression và caching
 */
private createCloudFrontDistribution(): cdk.aws_cloudfront.Distribution {
  // Import Route 53 Hosted Zone
  const hostedZone = cdk.aws_route53.HostedZone.fromHostedZoneAttributes(
    this, 'HostedZone', {
      hostedZoneId: cdk.Fn.importValue(this.exportName('HostedZoneId')),
      zoneName: 'everyonecook.cloud',
    }
  );
  
  // Import ACM certificate từ Certificate Stack (us-east-1)
  const certificate = cdk.aws_certificatemanager.Certificate.fromCertificateArn(
    this,
    'ImportedCloudFrontCertificate',
    'arn:aws:acm:us-east-1:616580903213:certificate/8d53776e-0480-47d2-a6ff-4fe9b2eb6534'
  );
  
  // Tạo cache policy với compression
  const publicCachePolicy = new cdk.aws_cloudfront.CachePolicy(
    this, 'PublicCachePolicy', {
      cachePolicyName: `EveryoneCook-Public-${this.config.environment}`,
      defaultTtl: cdk.Duration.hours(24),  // 24h mặc định
      maxTtl: cdk.Duration.days(7),        // tối đa 7 ngày
      minTtl: cdk.Duration.seconds(0),
      
      // Bật compression (Gzip + Brotli)
      enableAcceptEncodingGzip: true,
      enableAcceptEncodingBrotli: true,
      
      queryStringBehavior: cdk.aws_cloudfront.CacheQueryStringBehavior.all(),
      headerBehavior: cdk.aws_cloudfront.CacheHeaderBehavior.allowList(
        'CloudFront-Viewer-Country'
      ),
      cookieBehavior: cdk.aws_cloudfront.CacheCookieBehavior.none(),
    }
  );
  
  // Tạo CloudFront distribution
  const distribution = new cdk.aws_cloudfront.Distribution(
    this, 'CDNDistribution', {
      comment: `Everyone Cook CDN - ${this.config.environment}`,
      
      // Default behavior với OAC
      defaultBehavior: {
        origin: origins.S3BucketOrigin.withOriginAccessControl(this.contentBucket),
        viewerProtocolPolicy: cdk.aws_cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
        compress: true,
        cachePolicy: publicCachePolicy,
        allowedMethods: cdk.aws_cloudfront.AllowedMethods.ALLOW_GET_HEAD_OPTIONS,
      },
      
      // Price Class 200 (US, Europe, Asia - tiết kiệm 45% chi phí)
      priceClass: cdk.aws_cloudfront.PriceClass.PRICE_CLASS_200,
      
      // Custom domain với SSL
      domainNames: [this.config.domains.cdn],
      certificate: certificate,
      
      // CloudFront access logs
      enableLogging: true,
      logBucket: this.cdnLogsBucket,
      logFilePrefix: 'cdn-access-logs/',
      
      // Bật IPv6
      enableIpv6: true,
      
      // HTTP/2 + HTTP/3
      httpVersion: cdk.aws_cloudfront.HttpVersion.HTTP2_AND_3,
      
      // TLS 1.2 tối thiểu
      minimumProtocolVersion: cdk.aws_cloudfront.SecurityPolicyProtocol.TLS_V1_2_2021,
    }
  );
  
  // Tạo Route 53 A record (Alias đến CloudFront)
  new cdk.aws_route53.ARecord(this, 'CloudFrontAliasRecord', {
    zone: hostedZone,
    recordName: this.config.domains.cdn,
    target: cdk.aws_route53.RecordTarget.fromAlias(
      new targets.CloudFrontTarget(distribution)
    ),
  });
  
  return distribution;
}
```

---

## Chi Tiết Cấu Hình Chính

### 1. DynamoDB Single Table Design

**Cấu Trúc Table**:
```
PK: USER#{username}
SK: PROFILE | RECIPE#{recipeId} | POST#{postId} | COMMENT#{commentId}

GSI1: username-recipeDate (recipes của user theo ngày)
GSI2: username-postDate (posts của user theo ngày)
GSI3: recipeId-index (tìm kiếm recipe)
GSI4: postId-index (tìm kiếm post)
GSI5: email-index (tìm kiếm email)
```

**Billing Modes**:
- **Dev**: PAY_PER_REQUEST (không provisioned capacity, trả tiền theo request)
- **Staging**: PROVISIONED (5 RCU, 5 WCU với auto-scaling)
- **Prod**: PROVISIONED (10 RCU, 10 WCU với auto-scaling)

### 2. Tiết Kiệm Chi Phí với S3 Intelligent-Tiering

**So Sánh Chi Phí (100GB nội dung)**:

| Tier | Chi phí/GB | Chi phí 100GB | Timeline |
|------|------------|---------------|----------|
| Frequent Access | $0.023 | $2.30/tháng | Mặc định |
| Infrequent Access | $0.0125 | $1.25/tháng | Sau 30 ngày |
| Archive Instant | $0.004 | $0.40/tháng | Sau 90 ngày |
| Archive | $0.0036 | $0.36/tháng | Sau 90 ngày |
| Deep Archive | $0.00099 | $0.099/tháng | Sau 180 ngày |

**Tiết Kiệm**: $2.30 → $0.099 = **giảm 57% chi phí** sau 12 tháng

### 3. CloudFront Compression

**Giảm Kích Thước File**:
- **Text files** (HTML, CSS, JS, JSON): giảm 70-80%
- **Images** (đã nén sẵn): Không nén thêm
- **Videos** (đã nén sẵn): Không nén thêm

**Ví dụ**:
- JS bundle gốc: 1MB → Nén: 200KB (giảm 80%)
- 1000 requests/ngày: 1GB/ngày → 200MB/ngày (tiết kiệm 800MB bandwidth)

### 4. CloudFront Price Class 200

**So Sánh Chi Phí**:

| Price Class | Regions | Chi phí/GB | Tiết kiệm |
|-------------|---------|------------|-----------|
| All | Tất cả regions | $0.085 | Baseline |
| 200 | US, EU, Asia | $0.047 | **tiết kiệm 45%** |
| 100 | Chỉ US, EU | $0.025 | tiết kiệm 71% (nhưng phạm vi hạn chế) |

**Quyết Định**: Price Class 200 cân bằng chi phí (tiết kiệm 45%) với phạm vi toàn cầu.

---

## Stack Outputs

Sau khi triển khai, stack export các giá trị sau:

| Output Name | Value | Được sử dụng bởi |
|------------|-------|------------------|
| `DynamoDBTableName` | `EveryoneCook-dev-v2` | Auth Stack, Backend Stack |
| `DynamoDBTableArn` | `arn:aws:dynamodb:...` | Lambda IAM policies |
| `DynamoDBTableStreamArn` | `arn:aws:dynamodb:...` | Event-driven processing |
| `DynamoDBKeyId` | KMS Key ID | Encryption references |
| `DynamoDBKeyArn` | `arn:aws:kms:...` | Lambda permissions |
| `S3KeyId` | KMS Key ID | S3 encryption |
| `ContentBucketName` | `everyonecook-content-dev` | Backend Stack, Lambda |
| `ContentBucketArn` | `arn:aws:s3:...` | IAM policies |
| `CloudFrontDistributionId` | `E1234567890ABC` | Backend Stack (WAF) |
| `CloudFrontDomainName` | `d1234567890.cloudfront.net` | DNS verification |

---

## Các Bước Triển Khai

### Bước 1: Xác Minh Điều Kiện Tiên Quyết

Trước khi triển khai Core Stack, hãy đảm bảo:

-  DNS Stack đã triển khai thành công
-  Certificate Stack đã triển khai thành công (us-east-1)
-  Trạng thái CloudFront certificate là **Issued**
-  Ủy quyền DNS Route 53 đang hoạt động

### Bước 2: Triển Khai Core Stack

Di chuyển đến thư mục infrastructure:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Triển khai Core Stack đến **ap-southeast-1**:

```powershell
# Triển khai Core Stack
npx cdk deploy EveryoneCook-dev-Core --context environment=dev
```

Kết quả mong đợi:

```
✨  Synthesis time: 8.45s

EveryoneCook-dev-Core: deploying...
[████████████████████████████████████████] (12/12)

EveryoneCook-dev-Core: creating CloudFormation changeset...

   EveryoneCook-dev-Core

✨  Deployment time: 450.23s

Outputs:
EveryoneCook-dev-Core.DynamoDBTableName = EveryoneCook-dev-v2
EveryoneCook-dev-Core.ContentBucketName = everyonecook-content-dev
EveryoneCook-dev-Core.CloudFrontDistributionId = E2ABCDEFGHIJKL
EveryoneCook-dev-Core.CloudFrontDomainName = d1a2b3c4d5e6f7.cloudfront.net
EveryoneCook-dev-Core.StackInfo = Core Stack cho dev environment

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:616580903213:stack/EveryoneCook-dev-Core/...
```

### Bước 3: Chờ CloudFront Distribution

Triển khai CloudFront distribution mất **15-20 phút**. Bạn có thể theo dõi tiến trình:

```powershell
# Kiểm tra trạng thái CloudFront distribution
aws cloudfront get-distribution --id E2ABCDEFGHIJKL --query 'Distribution.Status'
```

Tiến trình trạng thái:
1. **InProgress** (0-15 phút)
2. **Deployed** (sẵn sàng sử dụng)

### Bước 4: Xác Minh trong AWS Console

#### Điều hướng đến DynamoDB

1. Mở AWS Console → region **ap-southeast-1**
2. Đi đến **DynamoDB** > **Tables**
3. Tìm table `EveryoneCook-dev-v2`

![DynamoDB Single Table](/images/5-Workshop/5.4-configure-stacks/dynamodb-table.png)
*DynamoDB table hiển thị partitionKey (PK), sortKey (SK), 5 GSI indexes, billing mode (PAY_PER_REQUEST), encryption (KMS), và streams enabled*

**Xác minh**:
-  Partition key: PK (String)
-  Sort key: SK (String)
-  5 GSI indexes hiển thị
-  Billing mode: PAY_PER_REQUEST (dev)
-  Encryption: Customer managed KMS
-  Streams: Enabled (NEW_AND_OLD_IMAGES)

#### Điều hướng đến S3

1. Đi đến **S3** > **Buckets**
2. Tìm buckets:
   - `everyonecook-content-dev`
   - `everyonecook-cdn-logs-dev`

![S3 Content Bucket](/images/5-Workshop/5.4-configure-stacks/s3-content-bucket.png)
*S3 content bucket hiển thị cấu trúc thư mục (avatars/, posts/, recipes/, backgrounds/), cấu hình Intelligent-Tiering, versioning, CORS, và encryption settings*

**Xác minh Content Bucket**:
-  Block all public access: On
-  Versioning: Enabled (prod) / Disabled (dev)
-  Intelligent-Tiering: Đã cấu hình
-  Encryption: S3-managed keys
-  CORS: Đã cấu hình
-  Lifecycle rules: 2 rules

![S3 Intelligent-Tiering](/images/5-Workshop/5.4-configure-stacks/s3-intelligent-tiering.png)
*Cấu hình S3 Intelligent-Tiering hiển thị Archive Access Tier (90 ngày) và Deep Archive Access Tier (180 ngày)*

#### Điều hướng đến CloudFront

1. Đi đến **CloudFront** > **Distributions**
2. Tìm distribution với domain `cdn-dev.everyonecook.cloud`

![CloudFront Distribution](/images/5-Workshop/5.4-configure-stacks/cloudfront-distribution.png)
*CloudFront distribution hiển thị custom domain, origin (S3 với OAC), cache behaviors, compression enabled, Price Class 200, SSL certificate, và status (Deployed)*

**Xác minh**:
-  Status: Deployed
-  Domain: cdn-dev.everyonecook.cloud
-  Origin: S3 bucket với OAC
-  Compression: Enabled
-  Price Class: 200
-  SSL Certificate: Valid
-  HTTPS: Redirect

![CloudFront OAC](/images/5-Workshop/5.4-configure-stacks/cloudfront-oac.png)
*Cấu hình CloudFront Origin Access Control (OAC) hiển thị S3 origin với OAC policy tự động được tạo*

#### Điều hướng đến KMS

1. Đi đến **KMS** > **Customer managed keys**
2. Tìm keys:
   - `everyonecook-dynamodb-dev`
   - `everyonecook-s3-dev`

![KMS Customer Managed Keys](/images/5-Workshop/5.4-configure-stacks/kms-keys.png)
*KMS customer managed keys hiển thị key aliases, key rotation enabled (hàng năm), key policies, và sử dụng bởi DynamoDB và S3*

**Xác minh**:
-  Key rotation: Enabled
-  Key state: Enabled
-  Deletion protection: Đã cấu hình
-  Key policy: DynamoDB/S3 service access

#### Xác Minh Route 53 DNS Record

1. Đi đến **Route 53** > **Hosted zones**
2. Chọn `everyonecook.cloud`
3. Xác minh A record cho `cdn-dev.everyonecook.cloud`

![Route 53 CloudFront Alias](/images/5-Workshop/5.4-configure-stacks/route53-cloudfront-alias.png)
*Route 53 A record (Alias) trỏ cdn-dev.everyonecook.cloud đến CloudFront distribution*

**Mong đợi**:
```
cdn-dev.everyonecook.cloud  A  Alias  d1a2b3c4d5e6f7.cloudfront.net
```

---

## Chi Tiết Chi Phí

### Chi Phí Hàng Tháng (Dev Environment)

| Tài nguyên | Cấu hình | Chi phí hàng tháng | Ghi chú |
|------------|----------|-------------------|---------|
| **DynamoDB** | PAY_PER_REQUEST | $1-5 | Lưu lượng thấp |
| **S3 Content** | 10GB, Intelligent-Tiering | $0.23 | Chuyển sang Deep Archive |
| **S3 Logs** | 5GB | $0.12 | Access logs |
| **CloudFront** | 100GB transfer, Price Class 200 | $4.70 | Tiết kiệm 45% vs All |
| **KMS** | 2 keys | $2.00 | $1/key/tháng |
| **Route 53** | 1 hosted zone, DNS queries | $0.50 | Từ DNS Stack |
| **Tổng (Ước tính)** | | **~$8-13/tháng** | Kịch bản lưu lượng thấp |

### Chiến Lược Tối Ưu Chi Phí

1. **S3 Intelligent-Tiering**: tiết kiệm 57% cho storage
2. **CloudFront Price Class 200**: tiết kiệm 45% cho bandwidth
3. **DynamoDB PAY_PER_REQUEST (dev)**: Chỉ trả tiền cho sử dụng thực tế
4. **CloudFront WAF removed**: tiết kiệm $9/tháng
5. **Compression enabled**: giảm 70-80% bandwidth

**Tổng Tiết Kiệm Tiềm Năng**: **~$20-30/tháng** so với thiết lập không tối ưu

---

## Cross-Stack Dependencies

### Import từ Các Stack Trước

**Từ DNS Stack**:
```typescript
hostedZoneId: cdk.Fn.importValue('EveryoneCook-dev-HostedZoneId')
zoneName: 'everyonecook.cloud'
```

**Từ Certificate Stack**:
```typescript
certificateArn: 'arn:aws:acm:us-east-1:616580903213:certificate/8d53776e-...'
```

### Export Được Sử Dụng Bởi Các Stack Khác

**Auth Stack** import:
- DynamoDB table (cho user profiles)
- DynamoDB key (cho Lambda permissions)

**Backend Stack** import:
- DynamoDB table (cho tất cả data operations)
- S3 content bucket (cho file uploads)
- CloudFront distribution (cho WAF association)
- KMS keys (cho encryption)

### Luồng Dependency

```
DNS Stack → Hosted Zone ID
    │
    ▼
Certificate Stack (us-east-1) → Certificate ARN
    │
    ▼
Core Stack (ap-southeast-1)
    │
    ├─► DynamoDB Table → Auth Stack, Backend Stack
    ├─► S3 Buckets → Backend Stack
    ├─► CloudFront → Backend Stack (WAF)
    └─► KMS Keys → Auth Stack, Backend Stack
```

---

## Danh Sách Kiểm Tra Xác Thực

Trước khi tiến hành triển khai Auth Stack:

- [ ] Core Stack đã triển khai thành công đến ap-southeast-1
- [ ] DynamoDB table tồn tại với 5 GSI indexes
- [ ] S3 content bucket đã cấu hình Intelligent-Tiering
- [ ] S3 CORS đã cấu hình cho frontend domain
- [ ] Trạng thái CloudFront distribution là **Deployed**
- [ ] CloudFront custom domain phân giải: `cdn-dev.everyonecook.cloud`
- [ ] CloudFront OAC đã cấu hình cho S3 access
- [ ] KMS keys đã tạo với rotation enabled
- [ ] Route 53 A record đã tạo cho CloudFront
- [ ] Tất cả stack outputs đã export thành công

---

## Kiểm Tra

### Kiểm Tra CloudFront Distribution

1. **Kiểm tra custom domain**:
   ```powershell
   curl -I https://cdn-dev.everyonecook.cloud
   ```
   
   Phản hồi mong đợi:
   ```
   HTTP/2 403
   server: CloudFront
   x-cache: Error from cloudfront
   ```
   
   (403 là mong đợi - bucket trống)

2. **Kiểm tra compression**:
   ```powershell
   curl -H "Accept-Encoding: gzip, br" -I https://cdn-dev.everyonecook.cloud
   ```

3. **Kiểm tra HTTPS redirect**:
   ```powershell
   curl -I http://cdn-dev.everyonecook.cloud
   ```
   
   Mong đợi: Redirect sang HTTPS

### Kiểm Tra DynamoDB Table

```powershell
# Liệt kê tables
aws dynamodb list-tables --region ap-southeast-1

# Mô tả table
aws dynamodb describe-table --table-name EveryoneCook-dev-v2 --region ap-southeast-1
```

### Kiểm Tra S3 Bucket Access

```powershell
# Thử truy cập S3 trực tiếp (nên bị chặn)
curl -I https://everyonecook-content-dev.s3.ap-southeast-1.amazonaws.com

# Mong đợi: 403 Forbidden (OAC protection hoạt động)
```

---

## Các Bước Tiếp Theo

Sau khi triển khai thành công Core Stack:

➡️ **[5.4.4 Auth Stack](../5.4.4-Auth-Stack/)** - Tạo Cognito User Pool và authentication

Auth Stack sẽ:
- Tạo Cognito User Pool với bảo mật nâng cao
- Cấu hình user attributes và password policy
- Thiết lập Cognito triggers (Lambda functions)
- Tạo SES email identity cho verification emails
- Import DynamoDB table từ Core Stack

---

## Tài Liệu Tham Khảo

- **Mã nguồn**: `infrastructure/lib/stacks/core-stack.ts`
- **Base Stack**: `infrastructure/lib/base-stack.ts`
- **Environment Config**: `infrastructure/config/environment.ts`
- **Tài liệu AWS**:
  - [DynamoDB Single Table Design](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-general-nosql-design.html)
  - [S3 Intelligent-Tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/)
  - [CloudFront Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
  - [KMS Key Rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html)
