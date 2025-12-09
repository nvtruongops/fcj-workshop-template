---
title: "Triển khai Hạ tầng"
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
DNS Stack (không có dependencies)
    ↓
Certificate Stack (phụ thuộc vào DNS)
    ↓
Core Stack (phụ thuộc vào Certificate)
    ↓
Auth Stack (phụ thuộc vào Core)
    ↓
Backend Stack (phụ thuộc vào Auth, Core)
    ↓
Observability Stack (phụ thuộc vào Backend)
```

#### Thời gian Triển khai

| Stack | Thời gian | Tài nguyên Quan trọng |
|-------|-----------|----------------------|
| DNS Stack | 2-3 phút | Route 53 Hosted Zone |
| Certificate Stack | 5-10 phút | ACM Certificate (DNS validation) |
| Core Stack | 10-15 phút | CloudFront Distribution, DynamoDB |
| Auth Stack | 5-7 phút | Cognito User Pool, SES |
| Backend Stack | 8-12 phút | API Gateway, 7 Lambda Functions |
| Observability Stack | 3-5 phút | CloudWatch Dashboards |

**Tổng thời gian triển khai: 35-50 phút**

---

### Yêu cầu Tiên quyết

#### 1. Xác minh AWS Credentials

```powershell
# Kiểm tra AWS credentials
aws sts get-caller-identity

# Kết quả mong đợi:
# {
#     "UserId": "...",
#     "Account": "616580903213",
#     "Arn": "arn:aws:iam::616580903213:user/your-username"
# }
```

#### 2. Xác minh Node.js & CDK

```powershell
# Kiểm tra phiên bản Node.js (phải là 20.x+)
node -v
# v20.11.0 hoặc cao hơn

# Kiểm tra phiên bản CDK
cdk --version
# 2.114.0 hoặc cao hơn
```

#### 3. Di chuyển đến Infrastructure

```powershell
# Di chuyển đến thư mục infrastructure
cd D:\Project_AWS\everyonecook\infrastructure

# Xác minh cdk.json tồn tại
Get-Item cdk.json
```

---

### Phase 1: Deploy DNS Stack (Route 53)

DNS Stack tạo Route 53 Hosted Zone cho domain `everyonecook.cloud`. Đây là stack đầu tiên và quan trọng nhất.

#### Bước 1.1: Deploy DNS Stack

```powershell
# Deploy DNS Stack
npx cdk deploy EveryoneCook-dev-DNS --context environment=dev

# Xem lại các thay đổi
# Nhập 'y' để xác nhận
```

**Kết quả mong đợi:**

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
# Lấy nameservers
$nameservers = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-DNS `
  --query 'Stacks[0].Outputs[?OutputKey==`NameServers`].OutputValue' `
  --output text

Write-Host "Nameservers: $nameservers"
# ns-1164.awsdns-17.org, ns-825.awsdns-39.net, ns-1889.awsdns-44.co.uk, ns-453.awsdns-56.com
```

#### Bước 1.3: Cập nhật Domain Registrar

Cập nhật nameservers tại domain registrar (Hostinger, GoDaddy, Namecheap, v.v.):

1. **Đăng nhập** vào domain registrar
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
6. **Lưu thay đổi**

#### Bước 1.4: Xác minh DNS Propagation

```powershell
# Kiểm tra DNS propagation (có thể mất 5-30 phút)
nslookup -type=NS everyonecook.cloud

# Hoặc dùng dig (nếu có WSL/Git Bash)
dig NS everyonecook.cloud

# Mong đợi: Thấy 4 nameservers từ AWS
```

**Công cụ online để kiểm tra:**
- https://www.whatsmydns.net/
- https://dnschecker.org/

⏳ **Đợi DNS propagate** trước khi tiếp tục (thường 5-15 phút)

---

### Phase 1.5: Deploy Certificate Stack (ACM)

**QUAN TRỌNG:** Stack này phải deploy ở **us-east-1** region vì yêu cầu của CloudFront.

#### Bước 1.5.1: Deploy Certificate Stack

```powershell
# Deploy Certificate Stack (us-east-1)
npx cdk deploy EveryoneCook-dev-Certificate --context environment=dev

# Nhập 'y' để xác nhận
```

**Stack tạo 2 certificates:**
1. **CloudFront Certificate** (us-east-1): `cdn-dev.everyonecook.cloud`
2. **API Gateway Certificate** (us-east-1): `*.everyonecook.cloud` (wildcard)

#### Bước 1.5.2: Đợi DNS Validation

ACM sẽ tự động:
- Tạo CNAME records trong Route 53 để validate domain
- Xác thực quyền sở hữu
- Phát hành certificates

**Quá trình này mất 5-10 phút.**

```powershell
# Kiểm tra trạng thái certificate
aws acm list-certificates --region us-east-1

# Lấy certificate ARN
$certArn = aws acm list-certificates --region us-east-1 `
  --query 'CertificateSummaryList[?DomainName==`cdn-dev.everyonecook.cloud`].CertificateArn' `
  --output text

# Kiểm tra trạng thái validation
aws acm describe-certificate --certificate-arn $certArn --region us-east-1 `
  --query 'Certificate.{Status:Status,ValidationMethod:DomainValidationOptions[0].ValidationMethod}'
```

**Kết quả mong đợi:**
```json
{
  "Status": "ISSUED",
  "ValidationMethod": "DNS"
}
```

#### Bước 1.5.3: Xác minh Certificates

```powershell
# Liệt kê tất cả certificates
aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Certificate `
  --region us-east-1 `
  --query 'Stacks[0].Outputs'

# Mong đợi:
# CloudFrontCertificateArn = arn:aws:acm:us-east-1:...:certificate/...
# ApiGatewayCertificateArn = arn:aws:acm:us-east-1:...:certificate/...
```

✅ **Chờ đến khi cả 2 certificates có Status = ISSUED**

---

### Phase 2: Deploy Core Stack (Foundation)

Core Stack tạo foundation resources: DynamoDB, S3, CloudFront, KMS.

#### Bước 2.1: Deploy Core Stack

```powershell
# Deploy Core Stack (mất 10-15 phút)
npx cdk deploy EveryoneCook-dev-Core --context environment=dev

# Nhập 'y' để xác nhận
```

**Stack tạo:**

**DynamoDB:**
- Table: `EveryoneCook-dev`
- Billing: Pay-per-request
- 5 GSI indexes (GSI1-GSI5)
- Stream enabled (cho DynamoDB Streams)
- Mã hóa KMS

**S3 Buckets (4 buckets):**
- `everyonecook-content-dev` - Tải lên của người dùng (avatars, hình ảnh)
- `everyonecook-logs-dev` - S3 access logs
- `everyonecook-cdn-logs-dev` - CloudFront logs
- `everyonecook-incoming-emails-dev` - SES email receiving

**CloudFront Distribution:**
- Custom domain: `cdn-dev.everyonecook.cloud`
- Origin: S3 content bucket
- Chỉ HTTPS (certificate từ Certificate Stack)
- Bật nén
- Price Class 200 (Mỹ, Châu Âu, Châu Á)

**KMS Keys (2 keys):**
- DynamoDB encryption key
- S3 encryption key

#### Bước 2.2: Theo dõi Deployment

Deployment này mất lâu nhất (10-15 phút) do CloudFront Distribution.

```powershell
# Trong terminal khác, theo dõi CloudFormation events
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-Core `
  --max-items 10 `
  --query 'StackEvents[*].{Time:Timestamp,Status:ResourceStatus,Type:ResourceType,Resource:LogicalResourceId}' `
  --output table
```

#### Bước 2.3: Xác minh Core Resources

```powershell
# Kiểm tra DynamoDB table
aws dynamodb describe-table --table-name EveryoneCook-dev `
  --query 'Table.{Name:TableName,Status:TableStatus,Billing:BillingModeSummary.BillingMode,Stream:StreamSpecification.StreamEnabled}'

# Mong đợi:
# {
#   "Name": "EveryoneCook-dev",
#   "Status": "ACTIVE",
#   "Billing": "PAY_PER_REQUEST",
#   "Stream": true
# }

# Kiểm tra S3 buckets
aws s3 ls | Select-String "everyonecook"

# Mong đợi:
# everyonecook-cdn-logs-dev
# everyonecook-content-dev
# everyonecook-incoming-emails-dev
# everyonecook-logs-dev

# Kiểm tra CloudFront distribution
aws cloudfront list-distributions `
  --query 'DistributionList.Items[?Comment==`EveryoneCook CDN (dev)`].{Id:Id,Domain:DomainName,Status:Status}'

# Mong đợi:
# {
#   "Id": "E2INNJ4XX421Q3",
#   "Domain": "d2shrpzup69rju.cloudfront.net",
#   "Status": "Deployed"
# }
```

#### Bước 2.4: Lấy Stack Outputs

```powershell
# Lấy tất cả Core Stack outputs
aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Core `
  --query 'Stacks[0].Outputs[*].{Key:OutputKey,Value:OutputValue}' `
  --output table
```

**Outputs chính:**
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

# Nhập 'y' để xác nhận
```

**Stack tạo:**

**Cognito User Pool:**
- User Pool: `EveryoneCook-UserPool-dev`
- Password policy: tối thiểu 12 ký tự, yêu cầu ký tự đặc biệt
- MFA: Tùy chọn
- Xác minh email

**5 Lambda Triggers:**
1. **Pre-Signup** - Xác thực username/email
2. **Post-Confirmation** - Tạo user profile trong DynamoDB
3. **Post-Authentication** - Cập nhật lastLoginAt
4. **Pre-Authentication** - Kiểm tra trạng thái cấm
5. **Custom Message** - Templates email tùy chỉnh

**SES Email Identity:**
- Domain: `everyonecook.cloud`
- Xác thực DKIM
- Mail FROM domain: `mail.everyonecook.cloud`
- Chế độ Production (có thể gửi đến bất kỳ email nào)

**IAM Roles:**
- Lambda execution roles
- Cognito SMS role (cho MFA)

#### Bước 3.2: Xác minh Cognito User Pool

```powershell
# Lấy User Pool ID
$userPoolId = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Auth `
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' `
  --output text

Write-Host "User Pool ID: $userPoolId"
# ap-southeast-1_PKoL34PF0

# Mô tả User Pool
aws cognito-idp describe-user-pool --user-pool-id $userPoolId `
  --query 'UserPool.{Name:Name,Status:Status,MFA:MfaConfiguration}'
```

#### Bước 3.3: Xác minh Lambda Triggers

```powershell
# Kiểm tra Lambda triggers được gắn vào User Pool
aws cognito-idp describe-user-pool --user-pool-id $userPoolId `
  --query 'UserPool.LambdaConfig'

# Mong đợi: 5 triggers được cấu hình
# {
#   "PreSignUp": "arn:aws:lambda:...:function:EveryoneCook-dev-PreSignup",
#   "PostConfirmation": "arn:aws:lambda:...:function:EveryoneCook-dev-PostConfirmation",
#   "PostAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PostAuthentication",
#   "PreAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PreAuthentication",
#   "CustomMessage": "arn:aws:lambda:...:function:EveryoneCook-dev-CustomMessage"
# }
```

#### Bước 3.4: Xác minh SES Email Identity

```powershell
# Kiểm tra trạng thái SES identity
aws sesv2 get-email-identity --email-identity everyonecook.cloud `
  --query '{Status:VerifiedForSendingStatus,DKIM:DkimAttributes.Status}'

# Mong đợi:
# {
#   "Status": true,
#   "DKIM": "SUCCESS"
# }

# Kiểm tra DKIM records trong Route 53
aws route53 list-resource-record-sets `
  --hosted-zone-id Z018823421GWCSYG5UMHV `
  --query 'ResourceRecordSets[?Type==`CNAME` && contains(Name, `_domainkey`)]'
```

✅ **SES phải ở Production mode để gửi email đến bất kỳ địa chỉ nào**

---

### Phase 4: Deploy Backend Stack (API & Lambda)

Backend Stack tạo API Gateway và Lambda functions. **Trước khi deploy, cần build Lambda code.**

#### Bước 4.1: Build Lambda Code

```powershell
# Di chuyển đến project root
cd D:\Project_AWS\everyonecook

# Chạy deployment script để build tất cả Lambda code
.\deploy\deploy-backend.ps1 -Environment dev -SkipBuild:$false

# Hoặc build thủ công:
# Build tất cả modules
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
# Di chuyển về infrastructure
cd D:\Project_AWS\everyonecook\infrastructure

# Deploy Backend Stack
npx cdk deploy EveryoneCook-dev-Backend --context environment=dev

# Nhập 'y' để xác nhận
```

**Stack tạo:**

**API Gateway:**
- REST API: `EveryoneCook-API-dev`
- Custom domain: `api-dev.everyonecook.cloud`
- Cognito Authorizer
- Bảo vệ WAF Web ACL

**7 Lambda Functions:**
1. `everyonecook-dev-api-router` - API routing (512 MB, 30s)

2. `everyonecook-dev-auth-user` - Auth & User Management (512 MB, 30s)
3. `everyonecook-dev-social` - Tính năng mạng xã hội (512 MB, 30s)
4. `everyonecook-dev-recipe-ai` - Công thức & AI (1024 MB, 60s)
5. `everyonecook-dev-ai-worker` - AI processing worker (1024 MB, 300s)
6. `everyonecook-dev-admin` - Admin dashboard (512 MB, 30s)
7. `everyonecook-dev-upload` - Tải lên file (256 MB, 15s)

**Lambda Layer:**
- `everyonecook-shared-deps-dev` - Shared dependencies (~15-20 MB)

**SQS Queues (4 queues + 4 DLQs):**
- AI Queue + DLQ
- Image Processing Queue + DLQ
- Analytics Queue + DLQ
- Notification Queue + DLQ

**WAF Web ACL:**
- Bảo vệ API Gateway
- Giới hạn tốc độ (2000 req/5min mỗi IP)
- Hỗ trợ chặn theo vị trí địa lý

#### Bước 4.3: Xác minh API Gateway

```powershell
# Lấy API endpoint
$apiUrl = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Backend `
  --query 'Stacks[0].Outputs[?OutputKey==`ApiCustomDomain`].OutputValue' `
  --output text

Write-Host "API Endpoint: $apiUrl"
# https://api-dev.everyonecook.cloud

# Kiểm tra health endpoint
$response = Invoke-RestMethod -Uri "$apiUrl/health" -Method Get
$response | ConvertTo-Json

# Mong đợi:
# {
#   "status": "healthy",
#   "timestamp": "2025-12-09T...",
#   "service": "EveryoneCook API",
#   "environment": "dev"
# }
```

#### Bước 4.4: Xác minh Lambda Functions

```powershell
# Liệt kê tất cả Lambda functions
aws lambda list-functions `
  --query 'Functions[?contains(FunctionName, `everyonecook-dev`)].{Name:FunctionName,Runtime:Runtime,Memory:MemorySize,Timeout:Timeout}' `
  --output table

# Mong đợi: 7 functions + 5 Cognito triggers = 12 Lambda functions tổng cộng
```

#### Bước 4.5: Xác minh SQS Queues

```powershell
# Liệt kê SQS queues
aws sqs list-queues --queue-name-prefix everyonecook-dev

# Mong đợi: 8 queues (4 main + 4 DLQ)
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

# Nhập 'y' để xác nhận
```

**Stack tạo:**

**4 CloudWatch Dashboards:**
1. **Core Dashboard** - Metrics DynamoDB, S3, CloudFront
2. **Auth Dashboard** - Metrics Cognito, SES
3. **Backend Dashboard** - Metrics API Gateway, Lambda
4. **Overview Dashboard** - Sức khỏe hệ thống tổng quan

**CloudWatch Alarms:**
- DynamoDB throttling
- Lambda errors
- API Gateway 5xx errors
- CloudFront 5xx errors

**SNS Topic:**
- Thông báo alarm
- Hỗ trợ đăng ký email

**Composite Alarm:**
- Alarm sức khỏe hệ thống nghiêm trọng
- Tổng hợp nhiều alarms

#### Bước 5.2: Đăng ký SNS Topic

```powershell
# Lấy SNS topic ARN
$topicArn = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Observability `
  --query 'Stacks[0].Outputs[?OutputKey==`AlarmTopicArn`].OutputValue' `
  --output text

# Đăng ký email của bạn
aws sns subscribe `
  --topic-arn $topicArn `
  --protocol email `
  --notification-endpoint "your-email@example.com"

Write-Host "Kiểm tra email của bạn và xác nhận đăng ký!"
```

#### Bước 5.3: Xác minh Dashboards

```powershell
# Liệt kê CloudWatch dashboards
aws cloudwatch list-dashboards `
  --query 'DashboardEntries[?contains(DashboardName, `EveryoneCook-dev`)].DashboardName'

# Mong đợi:
# - EveryoneCook-dev-CoreDashboard
# - EveryoneCook-dev-AuthDashboard
# - EveryoneCook-dev-BackendDashboard
# - EveryoneCook-dev-OverviewDashboard

# Lấy URL dashboard
$region = "ap-southeast-1"
$dashboardName = "EveryoneCook-dev-OverviewDashboard"
Write-Host "Dashboard URL: https://console.aws.amazon.com/cloudwatch/home?region=$region#dashboards:name=$dashboardName"
```

---

### Xác minh Triển khai Hoàn chỉnh

#### Bước 6.1: Liệt kê Tất cả Stacks

```powershell
# Liệt kê tất cả deployed stacks
aws cloudformation list-stacks `
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE `
  --query 'StackSummaries[?contains(StackName, `EveryoneCook-dev`)].{Name:StackName,Status:StackStatus,Created:CreationTime}' `
  --output table
```

**Stacks mong đợi:**
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

#### Bước 6.2: Lấy Tất cả Stack Outputs

```powershell
# Tạo báo cáo outputs toàn diện
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

#### Bước 6.3: Lưu Outputs vào File

```powershell
# Lưu outputs vào infrastructure/outputs.json
cd D:\Project_AWS\everyonecook\infrastructure

# Lấy outputs dưới định dạng JSON
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

# Lưu vào file
$outputs | ConvertTo-Json -Depth 5 | Out-File -FilePath "outputs.json" -Encoding utf8

Write-Host "Outputs đã được lưu vào outputs.json" -ForegroundColor Green
```

#### Bước 6.4: Xác minh Tài nguyên Chính

```powershell
# Xác minh tài nguyên toàn diện
Write-Host "`n===== XÁC MINH TÀI NGUYÊN =====" -ForegroundColor Yellow

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

Write-Host "`n===== XÁC MINH HOÀN TẤT =====" -ForegroundColor Green
```

---

### Tóm tắt Triển khai

#### Số lượng Tài nguyên Đã triển khai

| Danh mục | Số lượng | Tài nguyên |
|----------|----------|------------|
| **Networking** | 5 | Route 53 Hosted Zone, 2 ACM Certificates, CloudFront Distribution, WAF Web ACL |
| **Storage** | 6 | DynamoDB Table (5 GSI), 4 S3 Buckets, 2 KMS Keys |
| **Compute** | 12 | 7 Lambda Functions, 5 Cognito Triggers |
| **Integration** | 8 | API Gateway, 4 SQS Queues, 4 DLQ |
| **Security** | 8 | Cognito User Pool, SES Identity, IAM Roles (6) |
| **Monitoring** | 15 | 4 Dashboards, 10+ Alarms, SNS Topic |

**Tổng số Tài nguyên: ~100+ tài nguyên AWS**

#### Phân tích theo từng Stack

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

### Xử lý Sự cố

#### Vấn đề 1: Stack Deployment Thất bại

```powershell
# Kiểm tra CloudFormation events để tìm lỗi
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-STACKNAME `
  --max-items 20 `
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED` || ResourceStatus==`ROLLBACK_IN_PROGRESS`].{Time:Timestamp,Status:ResourceStatus,Reason:ResourceStatusReason,Resource:LogicalResourceId}' `
  --output table

# Nguyên nhân thường gặp:
# - Thiếu dependencies (kiểm tra thứ tự dependency)
# - Quyền không đủ
# - Vượt quá giới hạn tài nguyên
# - Xung đột tên
```

#### Vấn đề 2: Certificate Validation Bị Treo

```powershell
# Kiểm tra trạng thái certificate
$certArn = "arn:aws:acm:us-east-1:...:certificate/..."
aws acm describe-certificate --certificate-arn $certArn --region us-east-1 `
  --query 'Certificate.DomainValidationOptions[0].{Domain:DomainName,Status:ValidationStatus,Method:ValidationMethod}'

# Kiểm tra DNS records
aws route53 list-resource-record-sets `
  --hosted-zone-id Z018823421GWCSYG5UMHV `
  --query 'ResourceRecordSets[?Type==`CNAME`]'

# Giải pháp:
# - Xác minh nameservers đã cập nhật tại registrar
# - Đợi DNS propagation (lên đến 48 giờ, thường là 15 phút)
# - Kiểm tra validation CNAME records tồn tại trong Route 53
```

#### Vấn đề 3: Lambda Deployment Thất bại

```powershell
# Kiểm tra nếu thư mục dist tồn tại
Get-ChildItem D:\Project_AWS\everyonecook\services\*\dist

# Nếu thiếu, rebuild
cd D:\Project_AWS\everyonecook
.\deploy\deploy-backend.ps1 -Environment dev

# Kiểm tra kích thước Lambda package
Get-ChildItem D:\Project_AWS\everyonecook\services\*\lambda.zip | 
  Select-Object Name, @{Name="Size(MB)";Expression={[math]::Round($_.Length/1MB,2)}}

# Giới hạn Lambda:
# - Deployment package: 50 MB (đã nén)
# - Giải nén: 250 MB
```

#### Vấn đề 4: CloudFront Distribution Thất bại

```powershell
# Kiểm tra region của certificate (phải là us-east-1)
aws acm list-certificates --region us-east-1 `
  --query 'CertificateSummaryList[?DomainName==`cdn-dev.everyonecook.cloud`]'

# Kiểm tra S3 bucket tồn tại
aws s3 ls everyonecook-content-dev

# Giải pháp:
# - Đảm bảo Certificate Stack được deploy trước
# - Certificate phải ở us-east-1
# - S3 bucket phải tồn tại trước CloudFront
```

#### Vấn đề 5: Quyền IAM Không đủ

```powershell
# Kiểm tra user/role hiện tại
aws sts get-caller-identity

# Quyền cần thiết:
# - cloudformation:* (tất cả CloudFormation operations)
# - iam:* (tạo roles, policies)
# - lambda:* (tạo functions)
# - apigateway:* (tạo API)
# - cognito-idp:* (tạo user pools)
# - s3:* (tạo buckets)
# - dynamodb:* (tạo tables)
# - route53:* (tạo hosted zones)
# - acm:* (tạo certificates)
# - cloudfront:* (tạo distributions)
# - wafv2:* (tạo web ACLs)
# - logs:* (tạo log groups)
# - sns:* (tạo topics)
# - sqs:* (tạo queues)

# Khuyến nghị: AdministratorAccess policy (cho lần deploy đầu tiên)
```

#### Vấn đề 6: Stack Rollback

```powershell
# Nếu stack rollback, kiểm tra lý do
aws cloudformation describe-stack-events `
  --stack-name EveryoneCook-dev-STACKNAME `
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`].[Timestamp,ResourceStatusReason]' `
  --output table

# Xóa failed stack trước khi thử lại
aws cloudformation delete-stack --stack-name EveryoneCook-dev-STACKNAME

# Đợi xóa hoàn tất
aws cloudformation wait stack-delete-complete --stack-name EveryoneCook-dev-STACKNAME

# Redeploy
npx cdk deploy EveryoneCook-dev-STACKNAME --context environment=dev
```

---

### Ước tính Chi phí

#### Phân tích Chi phí Hàng tháng (Môi trường Dev)

| Dịch vụ | Sử dụng | Chi phí |
|---------|---------|---------|
| **DynamoDB** | Pay-per-request, sử dụng thấp | $2-5 |
| **S3** | 10 GB lưu trữ + requests | $1-2 |
| **CloudFront** | 10 GB truyền tải | $1-2 |
| **Lambda** | 1M requests, 512 MB | $3-5 |
| **API Gateway** | 100K requests | $0.35 |
| **Cognito** | 100 MAU | $0-5 |
| **SES** | 100 emails | $0.01 |
| **CloudWatch** | Logs + metrics | $2-3 |
| **WAF** | Quy tắc cơ bản | $5 |
| **Route 53** | 1 hosted zone | $0.50 |
| **SQS** | 100K requests | $0.04 |
| **KMS** | 2 keys | $2 |

**Tổng Chi phí Ước tính Hàng tháng: $20-35 (không có OpenSearch)**

**Với OpenSearch (nếu được bật): $70-135/tháng**

#### Theo dõi Chi phí

```powershell
# Kiểm tra chi phí tháng hiện tại
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

### Các Bước Tiếp theo

Infrastructure deployment hoàn thành! Tiếp theo:

1. ✅ **Xác minh Tất cả Stacks**: Tất cả 6 stacks đã được deploy thành công
2. 🔧 **Cấu hình API**: Thiết lập API routes và Lambda integration → [5.06 - Cấu hình API & Lambda](../5.06-configure-api-lambda/)
3. 🚀 **Deploy Backend Code**: Build và deploy Lambda code → [5.07 - Deploy Backend](../5.07-deploy-backend/)
4. ✅ **Kiểm tra Endpoints**: Xác minh chức năng API → [5.08 - Kiểm tra Endpoints](../5.08-test-endpoints/)

**Trạng thái Hiện tại:**
- Infrastructure: ✅ Hoàn thành
- Lambda Code: ⏳ Cần deploy
- Testing: ⏳ Đang chờ

Tiếp tục đến: [5.06 - Cấu hình API & Lambda](../5.06-configure-api-lambda/)
