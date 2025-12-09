---
title: "Deploy Infrastructure"
date: 2025-12-01
weight: 5
chapter: false
pre: " <b> 5.05. </b> "
---

#### Tổng quan

Trong bước này, bạn sẽ deploy tất cả infrastructure stacks lên AWS theo đúng thứ tự dependencies. Dự án EveryoneCook sử dụng **5-Stack Architecture** với AWS CDK.

#### 5-Stack Architecture

```
Phase 1:  DNS Stack           → Route 53 Hosted Zone
Phase 1.5: Certificate Stack   → ACM Certificates (us-east-1)
Phase 2:  Core Stack          → DynamoDB, S3, CloudFront, KMS
Phase 3:  Auth Stack          → Cognito, SES, Lambda Triggers
Phase 4:  Backend Stack       → API Gateway, Lambda, SQS, WAF
Phase 5:  Observability Stack → CloudWatch Dashboards & Alarms
```

#### Stack Dependencies

```
DNS Stack (no dependencies)
    ↓
Certificate Stack (depends on DNS)
    ↓
Core Stack (depends on Certificate)
    ↓
Auth Stack (depends on Core)
    ↓
Backend Stack (depends on Auth, Core)
    ↓
Observability Stack (depends on Backend)
```

#### Deployment Timeline

| Stack | Time | Critical Resources |
|-------|------|-------------------|
| DNS Stack | 2-3 phút | Route 53 Hosted Zone |
| Certificate Stack | 5-10 phút | ACM Certificate (DNS validation) |
| Core Stack | 10-15 phút | CloudFront Distribution, DynamoDB |
| Auth Stack | 5-7 phút | Cognito User Pool, SES |
| Backend Stack | 8-12 phút | API Gateway, 7 Lambda Functions |
| Observability Stack | 3-5 phút | CloudWatch Dashboards |

**Total deployment time: 35-50 phút**

---

### Prerequisites

#### 1. Verify AWS Credentials

```powershell
# Check AWS credentials
aws sts get-caller-identity

# Expected output:
# {
#     "UserId": "...",
#     "Account": "616580903213",
#     "Arn": "arn:aws:iam::616580903213:user/your-username"
# }
```

#### 2. Verify Node.js & CDK

```powershell
# Check Node.js version (must be 20.x+)
node -v
# v20.11.0 or higher

# Check CDK version
cdk --version
# 2.114.0 or higher
```

#### 3. Navigate to Infrastructure

```powershell
# Navigate to infrastructure directory
cd D:\Project_AWS\everyonecook\infrastructure

# Verify cdk.json exists
Get-Item cdk.json
```

---

### Phase 1: Deploy DNS Stack (Route 53)

DNS Stack tạo Route 53 Hosted Zone cho domain `everyonecook.cloud`. Đây là stack đầu tiên và quan trọng nhất.

#### Bước 1.1: Deploy DNS Stack

```powershell
# Deploy DNS Stack
npx cdk deploy EveryoneCook-dev-DNS --context environment=dev

# Review changes
# Type 'y' to confirm
```

**Expected output:**

```
✅  EveryoneCook-dev-DNS

Outputs:
EveryoneCook-dev-DNS.HostedZoneId = Z018823421GWCSYG5UMHV
EveryoneCook-dev-DNS.HostedZoneName = everyonecook.cloud
EveryoneCook-dev-DNS.NameServers = ns-1164.awsdns-17.org, ns-825.awsdns-39.net, ns-1889.awsdns-44.co.uk, ns-453.awsdns-56.com

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:616580903213:stack/EveryoneCook-dev-DNS/...
```

#### Bước 1.2: Lưu Nameservers

**LƯU Ý QUAN TRỌNG:** Lưu lại 4 nameservers từ output!

```powershell
# Get nameservers
$nameservers = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-DNS `
  --query 'Stacks[0].Outputs[?OutputKey==`NameServers`].OutputValue' `
  --output text

Write-Host "Nameservers: $nameservers"
# ns-1164.awsdns-17.org, ns-825.awsdns-39.net, ns-1889.awsdns-44.co.uk, ns-453.awsdns-56.com
```

#### Bước 1.3: Update Domain Registrar

Cập nhật nameservers tại domain registrar (Hostinger, GoDaddy, Namecheap, etc.):

1. **Login** vào domain registrar
2. Tìm domain **everyonecook.cloud**
3. Chọn **"Custom Nameservers"** hoặc **"DNS Settings"**
4. **Xóa** nameservers cũ
5. **Nhập** 4 nameservers từ AWS:
   ```
   ns-1164.awsdns-17.org
   ns-825.awsdns-39.net
   ns-1889.awsdns-44.co.uk
   ns-453.awsdns-56.com
   ```
6. **Save changes**

#### Bước 1.4: Verify DNS Propagation

```powershell
# Check DNS propagation (có thể mất 5-30 phút)
nslookup -type=NS everyonecook.cloud

# Hoặc dùng dig (nếu có WSL/Git Bash)
dig NS everyonecook.cloud

# Expected: Thấy 4 nameservers từ AWS
```

**Online tools để check:**
- https://www.whatsmydns.net/
- https://dnschecker.org/

⏳ **Đợi DNS propagate** trước khi tiếp tục (thường 5-15 phút)

---

### Phase 1.5: Deploy Certificate Stack (ACM)

**QUAN TRỌNG:** Stack này phải deploy ở **us-east-1** region vì CloudFront requirement.

#### Bước 1.5.1: Deploy Certificate Stack

```powershell
# Deploy Certificate Stack (us-east-1)
npx cdk deploy EveryoneCook-dev-Certificate --context environment=dev

# Type 'y' to confirm
```

**Stack tạo 2 certificates:**
1. **CloudFront Certificate** (us-east-1): `cdn-dev.everyonecook.cloud`
2. **API Gateway Certificate** (us-east-1): `*.everyonecook.cloud` (wildcard)

#### Bước 1.5.2: Wait for DNS Validation

ACM sẽ tự động:
- Tạo CNAME records trong Route 53 để validate domain
- Validate ownership
- Issue certificates

**Quá trình này mất 5-10 phút.**

```powershell
# Check certificate status
aws acm list-certificates --region us-east-1

# Get certificate ARN
$certArn = aws acm list-certificates --region us-east-1 `
  --query 'CertificateSummaryList[?DomainName==`cdn-dev.everyonecook.cloud`].CertificateArn' `
  --output text

# Check validation status
aws acm describe-certificate --certificate-arn $certArn --region us-east-1 `
  --query 'Certificate.{Status:Status,ValidationMethod:DomainValidationOptions[0].ValidationMethod}'
```

**Expected output:**
```json
{
  "Status": "ISSUED",
  "ValidationMethod": "DNS"
}
```

#### Bước 1.5.3: Verify Certificates

```powershell
# List all certificates
aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Certificate `
  --region us-east-1 `
  --query 'Stacks[0].Outputs'

# Expected:
# CloudFrontCertificateArn = arn:aws:acm:us-east-1:...:certificate/...
# ApiGatewayCertificateArn = arn:aws:acm:us-east-1:...:certificate/...
```

✅ **Chờ đến khi cả 2 certificates có Status = ISSUED**

---

### Phase 2: Deploy Core Stack (Foundation)

Core Stack tạo foundation resources: DynamoDB, S3, CloudFront, KMS.

#### Bước 2.1: Deploy Core Stack

```powershell
# Deploy Core Stack (takes 10-15 minutes)
npx cdk deploy EveryoneCook-dev-Core --context environment=dev

# Type 'y' to confirm
```

**Stack tạo:**

**DynamoDB:**
- Table: `EveryoneCook-dev`
- Billing: Pay-per-request
- 5 GSI indexes (GSI1-GSI5)
- Stream enabled (for DynamoDB Streams)
- KMS encryption

**S3 Buckets (4 buckets):**
- `everyonecook-content-dev` - User uploads (avatars, images)
- `everyonecook-logs-dev` - S3 access logs
- `everyonecook-cdn-logs-dev` - CloudFront logs
- `everyonecook-incoming-emails-dev` - SES email receiving

**CloudFront Distribution:**
- Custom domain: `cdn-dev.everyonecook.cloud`
- Origin: S3 content bucket
- HTTPS only (certificate từ Certificate Stack)
- Compression enabled
- Price Class 200 (US, Europe, Asia)

**KMS Keys (2 keys):**
- DynamoDB encryption key
- S3 encryption key

#### Bước 2.2: Monitor Deployment

Deployment này mất lâu nhất (10-15 phút) do CloudFront Distribution.

```powershell
# In another terminal, monitor CloudFormation events
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-Core `
  --max-items 10 `
  --query 'StackEvents[*].{Time:Timestamp,Status:ResourceStatus,Type:ResourceType,Resource:LogicalResourceId}' `
  --output table
```

#### Bước 2.3: Verify Core Resources

```powershell
# Check DynamoDB table
aws dynamodb describe-table --table-name EveryoneCook-dev `
  --query 'Table.{Name:TableName,Status:TableStatus,Billing:BillingModeSummary.BillingMode,Stream:StreamSpecification.StreamEnabled}'

# Expected:
# {
#   "Name": "EveryoneCook-dev",
#   "Status": "ACTIVE",
#   "Billing": "PAY_PER_REQUEST",
#   "Stream": true
# }

# Check S3 buckets
aws s3 ls | Select-String "everyonecook"

# Expected:
# everyonecook-cdn-logs-dev
# everyonecook-content-dev
# everyonecook-incoming-emails-dev
# everyonecook-logs-dev

# Check CloudFront distribution
aws cloudfront list-distributions `
  --query 'DistributionList.Items[?Comment==`EveryoneCook CDN (dev)`].{Id:Id,Domain:DomainName,Status:Status}'

# Expected:
# {
#   "Id": "E2INNJ4XX421Q3",
#   "Domain": "d2shrpzup69rju.cloudfront.net",
#   "Status": "Deployed"
# }
```

#### Bước 2.4: Get Stack Outputs

```powershell
# Get all Core Stack outputs
aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Core `
  --query 'Stacks[0].Outputs[*].{Key:OutputKey,Value:OutputValue}' `
  --output table
```

**Key outputs:**
- `DynamoDBTableName`: EveryoneCook-dev
- `ContentBucketName`: everyonecook-content-dev
- `CloudFrontDistributionId`: E2INNJ4XX421Q3
- `CloudFrontDomainName`: d2shrpzup69rju.cloudfront.net

---

### Phase 3: Deploy Auth Stack (Cognito & SES)

Auth Stack tạo authentication infrastructure với Cognito và SES.

#### Bước 3.1: Deploy Auth Stack

```powershell
# Deploy Auth Stack
npx cdk deploy EveryoneCook-dev-Auth --context environment=dev

# Type 'y' to confirm
```

**Stack tạo:**

**Cognito User Pool:**
- User Pool: `EveryoneCook-UserPool-dev`
- Password policy: 12 chars min, requires symbols
- MFA: Optional
- Email verification

**5 Lambda Triggers:**
1. **Pre-Signup** - Validate username/email
2. **Post-Confirmation** - Tạo user profile trong DynamoDB
3. **Post-Authentication** - Update lastLoginAt
4. **Pre-Authentication** - Check ban status
5. **Custom Message** - Custom email templates

**SES Email Identity:**
- Domain: `everyonecook.cloud`
- DKIM authentication
- Mail FROM domain: `mail.everyonecook.cloud`
- Production mode (can send to any email)

**IAM Roles:**
- Lambda execution roles
- Cognito SMS role (for MFA)

#### Bước 3.2: Verify Cognito User Pool

```powershell
# Get User Pool ID
$userPoolId = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Auth `
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' `
  --output text

Write-Host "User Pool ID: $userPoolId"
# ap-southeast-1_PKoL34PF0

# Describe User Pool
aws cognito-idp describe-user-pool --user-pool-id $userPoolId `
  --query 'UserPool.{Name:Name,Status:Status,MFA:MfaConfiguration}'
```

#### Bước 3.3: Verify Lambda Triggers

```powershell
# Check Lambda triggers attached to User Pool
aws cognito-idp describe-user-pool --user-pool-id $userPoolId `
  --query 'UserPool.LambdaConfig'

# Expected: 5 triggers configured
# {
#   "PreSignUp": "arn:aws:lambda:...:function:EveryoneCook-dev-PreSignup",
#   "PostConfirmation": "arn:aws:lambda:...:function:EveryoneCook-dev-PostConfirmation",
#   "PostAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PostAuthentication",
#   "PreAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PreAuthentication",
#   "CustomMessage": "arn:aws:lambda:...:function:EveryoneCook-dev-CustomMessage"
# }
```

#### Bước 3.4: Verify SES Email Identity

```powershell
# Check SES identity status
aws sesv2 get-email-identity --email-identity everyonecook.cloud `
  --query '{Status:VerifiedForSendingStatus,DKIM:DkimAttributes.Status}'

# Expected:
# {
#   "Status": true,
#   "DKIM": "SUCCESS"
# }

# Check DKIM records in Route 53
aws route53 list-resource-record-sets `
  --hosted-zone-id Z018823421GWCSYG5UMHV `
  --query 'ResourceRecordSets[?Type==`CNAME` && contains(Name, `_domainkey`)]'
```

✅ **SES phải ở Production mode để send email đến bất kỳ địa chỉ nào**

---

### Phase 4: Deploy Backend Stack (API & Lambda)

Backend Stack tạo API Gateway và Lambda functions. **Trước khi deploy, cần build Lambda code.**

#### Bước 4.1: Build Lambda Code

```powershell
# Navigate to project root
cd D:\Project_AWS\everyonecook

# Run deployment script to build all Lambda code
.\deploy\deploy-backend.ps1 -Environment dev -SkipBuild:$false

# Or manual build:
# Build all modules
cd services/api-router; npm run build; cd ../..
cd services/auth-module; npm run build; cd ../..
cd services/social-module; npm run build; cd ../..
cd services/ai-module; npm run build; cd ../..
cd services/admin-module; npm run build; cd ../..
cd services/upload-module; npm run build; cd ../..
```

⚠️ **QUAN TRỌNG:** Lambda code phải được build TRƯỚC khi deploy Backend Stack!

#### Bước 4.2: Deploy Backend Stack

```powershell
# Navigate back to infrastructure
cd D:\Project_AWS\everyonecook\infrastructure

# Deploy Backend Stack
npx cdk deploy EveryoneCook-dev-Backend --context environment=dev

# Type 'y' to confirm
```

**Stack tạo:**

**API Gateway:**
- REST API: `EveryoneCook-API-dev`
- Custom domain: `api-dev.everyonecook.cloud`
- Cognito Authorizer
- WAF Web ACL protection

**7 Lambda Functions:**
1. `everyonecook-dev-api-router` - API routing (512 MB, 30s)

2. `everyonecook-dev-auth-user` - Auth & User Management (512 MB, 30s)
3. `everyonecook-dev-social` - Social features (512 MB, 30s)
4. `everyonecook-dev-recipe-ai` - Recipes & AI (1024 MB, 60s)
5. `everyonecook-dev-ai-worker` - AI processing worker (1024 MB, 300s)
6. `everyonecook-dev-admin` - Admin dashboard (512 MB, 30s)
7. `everyonecook-dev-upload` - File upload (256 MB, 15s)

**Lambda Layer:**
- `everyonecook-shared-deps-dev` - Shared dependencies (~15-20 MB)

**SQS Queues (4 queues + 4 DLQs):**
- AI Queue + DLQ
- Image Processing Queue + DLQ
- Analytics Queue + DLQ
- Notification Queue + DLQ

**WAF Web ACL:**
- API Gateway protection
- Rate limiting (2000 req/5min per IP)
- Geo blocking support

#### Bước 4.3: Verify API Gateway

```powershell
# Get API endpoint
$apiUrl = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Backend `
  --query 'Stacks[0].Outputs[?OutputKey==`ApiCustomDomain`].OutputValue' `
  --output text

Write-Host "API Endpoint: $apiUrl"
# https://api-dev.everyonecook.cloud

# Test health endpoint
$response = Invoke-RestMethod -Uri "$apiUrl/health" -Method Get
$response | ConvertTo-Json

# Expected:
# {
#   "status": "healthy",
#   "timestamp": "2025-12-09T...",
#   "service": "EveryoneCook API",
#   "environment": "dev"
# }
```

#### Bước 4.4: Verify Lambda Functions

```powershell
# List all Lambda functions
aws lambda list-functions `
  --query 'Functions[?contains(FunctionName, `everyonecook-dev`)].{Name:FunctionName,Runtime:Runtime,Memory:MemorySize,Timeout:Timeout}' `
  --output table

# Expected: 7 functions + 5 Cognito triggers = 12 Lambda functions total
```

#### Bước 4.5: Verify SQS Queues

```powershell
# List SQS queues
aws sqs list-queues --queue-name-prefix everyonecook-dev

# Expected: 8 queues (4 main + 4 DLQ)
# - everyonecook-dev-ai-queue
# - everyonecook-dev-ai-queue-dlq
# - everyonecook-dev-image-queue
# - everyonecook-dev-image-queue-dlq
# - everyonecook-dev-analytics-queue
# - everyonecook-dev-analytics-queue-dlq
# - everyonecook-dev-notification-queue
# - everyonecook-dev-notification-queue-dlq
```

---

### Phase 5: Deploy Observability Stack (CloudWatch)

Observability Stack tạo monitoring dashboards và alarms.

#### Bước 5.1: Deploy Observability Stack

```powershell
# Deploy Observability Stack
npx cdk deploy EveryoneCook-dev-Observability --context environment=dev

# Type 'y' to confirm
```

**Stack tạo:**

**4 CloudWatch Dashboards:**
1. **Core Dashboard** - DynamoDB, S3, CloudFront metrics
2. **Auth Dashboard** - Cognito, SES metrics
3. **Backend Dashboard** - API Gateway, Lambda metrics
4. **Overview Dashboard** - High-level system health

**CloudWatch Alarms:**
- DynamoDB throttling
- Lambda errors
- API Gateway 5xx errors
- CloudFront 5xx errors

**SNS Topic:**
- Alarm notifications
- Email subscription support

**Composite Alarm:**
- Critical system health alarm
- Aggregates multiple alarms

#### Bước 5.2: Subscribe to SNS Topic

```powershell
# Get SNS topic ARN
$topicArn = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Observability `
  --query 'Stacks[0].Outputs[?OutputKey==`AlarmTopicArn`].OutputValue' `
  --output text

# Subscribe your email
aws sns subscribe `
  --topic-arn $topicArn `
  --protocol email `
  --notification-endpoint "your-email@example.com"

Write-Host "Check your email and confirm subscription!"
```

#### Bước 5.3: Verify Dashboards

```powershell
# List CloudWatch dashboards
aws cloudwatch list-dashboards `
  --query 'DashboardEntries[?contains(DashboardName, `EveryoneCook-dev`)].DashboardName'

# Expected:
# - EveryoneCook-dev-CoreDashboard
# - EveryoneCook-dev-AuthDashboard
# - EveryoneCook-dev-BackendDashboard
# - EveryoneCook-dev-OverviewDashboard

# Get dashboard URL
$region = "ap-southeast-1"
$dashboardName = "EveryoneCook-dev-OverviewDashboard"
Write-Host "Dashboard URL: https://console.aws.amazon.com/cloudwatch/home?region=$region#dashboards:name=$dashboardName"
```

---

### Verify Complete Deployment

#### Bước 6.1: List All Stacks

```powershell
# List all deployed stacks
aws cloudformation list-stacks `
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE `
  --query 'StackSummaries[?contains(StackName, `EveryoneCook-dev`)].{Name:StackName,Status:StackStatus,Created:CreationTime}' `
  --output table
```

**Expected stacks:**
```
------------------------------------------------------------
|                      ListStacks                          |
+--------------------------------------------------+--------+
|  Name                              | Status      | Created|
+--------------------------------------------------+--------+
|  EveryoneCook-dev-DNS              | CREATE_COMPLETE     |
|  EveryoneCook-dev-Certificate      | CREATE_COMPLETE     |
|  EveryoneCook-dev-Core             | CREATE_COMPLETE     |
|  EveryoneCook-dev-Auth             | CREATE_COMPLETE     |
|  EveryoneCook-dev-Backend          | CREATE_COMPLETE     |
|  EveryoneCook-dev-Observability    | CREATE_COMPLETE     |
+--------------------------------------------------+--------+
```

#### Bước 6.2: Get All Stack Outputs

```powershell
# Create comprehensive outputs report
$stacks = @("DNS", "Certificate", "Core", "Auth", "Backend", "Observability")

foreach ($stack in $stacks) {
  Write-Host "`n========================================" -ForegroundColor Cyan
  Write-Host "EveryoneCook-dev-$stack Outputs" -ForegroundColor Cyan
  Write-Host "========================================" -ForegroundColor Cyan
  
  aws cloudformation describe-stacks `
    --stack-name "EveryoneCook-dev-$stack" `
    --query 'Stacks[0].Outputs[*].{Key:OutputKey,Value:OutputValue}' `
    --output table
}
```

#### Bước 6.3: Save Outputs to File

```powershell
# Save outputs to infrastructure/outputs.json
cd D:\Project_AWS\everyonecook\infrastructure

# Get outputs in JSON format
$outputs = @{}
foreach ($stack in $stacks) {
  $stackOutputs = aws cloudformation describe-stacks `
    --stack-name "EveryoneCook-dev-$stack" `
    --query 'Stacks[0].Outputs' | ConvertFrom-Json
  
  $stackData = @{}
  foreach ($output in $stackOutputs) {
    $stackData[$output.OutputKey] = $output.OutputValue
  }
  $outputs["EveryoneCook-dev-$stack"] = $stackData
}

# Save to file
$outputs | ConvertTo-Json -Depth 5 | Out-File -FilePath "outputs.json" -Encoding utf8

Write-Host "Outputs saved to outputs.json" -ForegroundColor Green
```

#### Bước 6.4: Verify Key Resources

```powershell
# Comprehensive resource verification
Write-Host "`n===== RESOURCE VERIFICATION =====" -ForegroundColor Yellow

# DynamoDB
Write-Host "`nDynamoDB Table:" -ForegroundColor Cyan
aws dynamodb describe-table --table-name EveryoneCook-dev `
  --query 'Table.{Name:TableName,Status:TableStatus,ItemCount:ItemCount,Size:TableSizeBytes}' `
  --output table

# S3 Buckets
Write-Host "`nS3 Buckets:" -ForegroundColor Cyan
aws s3 ls | Select-String "everyonecook"

# CloudFront
Write-Host "`nCloudFront Distribution:" -ForegroundColor Cyan
aws cloudfront list-distributions `
  --query 'DistributionList.Items[?Comment==`EveryoneCook CDN (dev)`].{Id:Id,Status:Status,Domain:DomainName}' `
  --output table

# Lambda Functions
Write-Host "`nLambda Functions:" -ForegroundColor Cyan
aws lambda list-functions `
  --query 'Functions[?contains(FunctionName, `everyonecook-dev`)].FunctionName' `
  --output table

# Cognito User Pool
Write-Host "`nCognito User Pool:" -ForegroundColor Cyan
aws cognito-idp list-user-pools --max-results 10 `
  --query 'UserPools[?Name==`EveryoneCook-UserPool-dev`].{Name:Name,Id:Id,Status:Status}' `
  --output table

# API Gateway
Write-Host "`nAPI Gateway:" -ForegroundColor Cyan
aws apigateway get-rest-apis `
  --query 'items[?name==`EveryoneCook-API-dev`].{Name:name,Id:id}' `
  --output table

# SQS Queues
Write-Host "`nSQS Queues:" -ForegroundColor Cyan
aws sqs list-queues --queue-name-prefix everyonecook-dev `
  --query 'QueueUrls' --output table

Write-Host "`n===== VERIFICATION COMPLETE =====" -ForegroundColor Green
```

---

### Deployment Summary

#### Deployed Resources Count

| Category | Count | Resources |
|----------|-------|-----------|
| **Networking** | 5 | Route 53 Hosted Zone, 2 ACM Certificates, CloudFront Distribution, WAF Web ACL |
| **Storage** | 6 | DynamoDB Table (5 GSI), 4 S3 Buckets, 2 KMS Keys |
| **Compute** | 12 | 7 Lambda Functions, 5 Cognito Triggers |
| **Integration** | 8 | API Gateway, 4 SQS Queues, 4 DLQ |
| **Security** | 8 | Cognito User Pool, SES Identity, IAM Roles (6) |
| **Monitoring** | 15 | 4 Dashboards, 10+ Alarms, SNS Topic |

**Total Resources: ~100+ AWS resources**

#### Stack-by-Stack Breakdown

**DNS Stack:**
- ✅ Route 53 Hosted Zone

**Certificate Stack:**
- ✅ 2 ACM Certificates (us-east-1)

**Core Stack:**
- ✅ DynamoDB Table + 5 GSIs + Stream
- ✅ 4 S3 Buckets
- ✅ CloudFront Distribution
- ✅ 2 KMS Keys

**Auth Stack:**
- ✅ Cognito User Pool + Client
- ✅ 5 Lambda Triggers
- ✅ SES Email Identity
- ✅ IAM Roles

**Backend Stack:**
- ✅ API Gateway REST API
- ✅ 7 Lambda Functions
- ✅ Lambda Layer
- ✅ 4 SQS Queues + 4 DLQs
- ✅ WAF Web ACL

**Observability Stack:**
- ✅ 4 CloudWatch Dashboards
- ✅ 10+ CloudWatch Alarms
- ✅ SNS Topic
- ✅ Composite Alarm

---

### Troubleshooting

#### Issue 1: Stack Deployment Failed

```powershell
# Check CloudFormation events for errors
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-STACKNAME `
  --max-items 20 `
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED` || ResourceStatus==`ROLLBACK_IN_PROGRESS`].{Time:Timestamp,Status:ResourceStatus,Reason:ResourceStatusReason,Resource:LogicalResourceId}' `
  --output table

# Common causes:
# - Missing dependencies (check dependency order)
# - Insufficient permissions
# - Resource limits exceeded
# - Naming conflicts
```

#### Issue 2: Certificate Validation Stuck

```powershell
# Check certificate status
$certArn = "arn:aws:acm:us-east-1:...:certificate/..."
aws acm describe-certificate --certificate-arn $certArn --region us-east-1 `
  --query 'Certificate.DomainValidationOptions[0].{Domain:DomainName,Status:ValidationStatus,Method:ValidationMethod}'

# Check DNS records
aws route53 list-resource-record-sets `
  --hosted-zone-id Z018823421GWCSYG5UMHV `
  --query 'ResourceRecordSets[?Type==`CNAME`]'

# Solutions:
# - Verify nameservers updated at registrar
# - Wait for DNS propagation (up to 48h, usually 15min)
# - Check validation CNAME records exist in Route 53
```

#### Issue 3: Lambda Deployment Failed

```powershell
# Check if dist folder exists
Get-ChildItem D:\Project_AWS\everyonecook\services\*\dist

# If missing, rebuild
cd D:\Project_AWS\everyonecook
.\deploy\deploy-backend.ps1 -Environment dev

# Check Lambda package size
Get-ChildItem D:\Project_AWS\everyonecook\services\*\lambda.zip | 
  Select-Object Name, @{Name="Size(MB)";Expression={[math]::Round($_.Length/1MB,2)}}

# Lambda limits:
# - Deployment package: 50 MB (zipped)
# - Unzipped: 250 MB
```

#### Issue 4: CloudFront Distribution Failed

```powershell
# Check certificate region (must be us-east-1)
aws acm list-certificates --region us-east-1 `
  --query 'CertificateSummaryList[?DomainName==`cdn-dev.everyonecook.cloud`]'

# Check S3 bucket exists
aws s3 ls everyonecook-content-dev

# Solutions:
# - Ensure Certificate Stack deployed first
# - Certificate must be in us-east-1
# - S3 bucket must exist before CloudFront
```

#### Issue 5: Insufficient IAM Permissions

```powershell
# Check current user/role
aws sts get-caller-identity

# Required permissions:
# - cloudformation:* (all CloudFormation operations)
# - iam:* (create roles, policies)
# - lambda:* (create functions)
# - apigateway:* (create API)
# - cognito-idp:* (create user pools)
# - s3:* (create buckets)
# - dynamodb:* (create tables)
# - route53:* (create hosted zones)
# - acm:* (create certificates)
# - cloudfront:* (create distributions)
# - wafv2:* (create web ACLs)
# - logs:* (create log groups)
# - sns:* (create topics)
# - sqs:* (create queues)

# Recommended: AdministratorAccess policy (for first deployment)
```

#### Issue 6: Stack Rollback

```powershell
# If stack rolls back, check reason
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-STACKNAME `
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`].[Timestamp,ResourceStatusReason]' `
  --output table

# Delete failed stack before retry
aws cloudformation delete-stack --stack-name EveryoneCook-dev-STACKNAME

# Wait for deletion
aws cloudformation wait stack-delete-complete --stack-name EveryoneCook-dev-STACKNAME

# Redeploy
npx cdk deploy EveryoneCook-dev-STACKNAME --context environment=dev
```

---

### Cost Estimation

#### Monthly Cost Breakdown (Dev Environment)

| Service | Usage | Cost |
|---------|-------|------|
| **DynamoDB** | Pay-per-request, low usage | $2-5 |
| **S3** | 10 GB storage + requests | $1-2 |
| **CloudFront** | 10 GB transfer | $1-2 |
| **Lambda** | 1M requests, 512 MB | $3-5 |
| **API Gateway** | 100K requests | $0.35 |
| **Cognito** | 100 MAU | $0-5 |
| **SES** | 100 emails | $0.01 |
| **CloudWatch** | Logs + metrics | $2-3 |
| **WAF** | Basic rules | $5 |
| **Route 53** | 1 hosted zone | $0.50 |
| **SQS** | 100K requests | $0.04 |
| **KMS** | 2 keys | $2 |

**Total Estimated Monthly Cost: $20-35 (without OpenSearch)**

**With OpenSearch (if enabled): $70-135/month**

#### Cost Tracking

```powershell
# Check current month costs
$startDate = (Get-Date -Day 1).ToString("yyyy-MM-dd")
$endDate = (Get-Date).AddDays(1).ToString("yyyy-MM-dd")

aws ce get-cost-and-usage `
  --time-period Start=$startDate,End=$endDate `
  --granularity DAILY `
  --metrics BlendedCost `
  --group-by Type=SERVICE `
  --output table
```

---

### Next Steps

Infrastructure deployment hoàn thành! Tiếp theo:

1. ✅ **Verify All Stacks**: All 6 stacks deployed successfully
2. 🔧 **Configure API**: Setup API routes và Lambda integration → [5.06 - Configure API & Lambda](../5.06-configure-api-lambda/)
3. 🚀 **Deploy Backend Code**: Build và deploy Lambda code → [5.07 - Deploy Backend](../5.07-deploy-backend/)
4. ✅ **Test Endpoints**: Verify API functionality → [5.08 - Test Endpoints](../5.08-test-endpoints/)

**Current Status:**
- Infrastructure: ✅ Complete
- Lambda Code: ⏳ Need to deploy
- Testing: ⏳ Pending

Proceed to: [5.06 - Configure API & Lambda](../5.06-configure-api-lambda/)
