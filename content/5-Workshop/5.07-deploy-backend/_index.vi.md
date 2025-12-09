---
title: "Triển khai Backend Services"
date: 2025-12-01
weight: 7
chapter: false
pre: " <b> 5.07. </b> "
---

#### Tổng quan

Sau khi deploy infrastructure với CDK, bạn cần deploy Lambda code lên AWS. Dự án EveryoneCook cung cấp automated deployment script để build, package và deploy tất cả Lambda functions.

#### Kiến trúc Backend

**Lambda Modules (7 functions):**
- `api-router` - API Gateway routing với JWT validation
- `auth-user` - Authentication & User Management
- `social` - Posts, Comments, Reactions, Friends, Notifications
- `recipe-ai` - Recipes & AI Features (Bedrock)
- `ai-worker` - Async AI processing worker
- `admin` - Admin Dashboard & Content Moderation
- `upload` - File Upload với S3 Presigned URLs

**Lambda Layer:**
- `shared-dependencies` - Shared npm packages (aws-sdk, uuid, etc.)

**Cognito Triggers (Auth Stack):**
- Pre-Signup, Post-Confirmation, Post-Authentication, Pre-Authentication, Custom Message

#### Quy trình Triển khai

```
1. Build Lambda Layer (shared dependencies)
2. Build TypeScript modules → JavaScript
3. Validate dist folders (kiểm tra lỗi)
4. Chuẩn bị deployment packages
5. Deploy qua CDK hoặc Direct Lambda Update
6. Xác minh deployments
7. Kiểm tra CloudWatch logs
```

---

### Tùy chọn A: Triển khai Đầy đủ (Khuyến nghị - Lần đầu tiên)

Full deployment sử dụng CDK để deploy toàn bộ infrastructure và Lambda code.

#### Bước 1: Di chuyển đến Dự án

```powershell
# Di chuyển đến thư mục gốc everyonecook
cd D:\Project_AWS\everyonecook
```

#### Bước 2: Chạy Script Triển khai Đầy đủ

```powershell
# Full deployment cho dev environment
.\deploy\deploy-backend.ps1 -Environment dev

# Hoặc ngắn gọn (mặc định là dev)
.\deploy\deploy-backend.ps1
```

**Script sẽ thực hiện:**

**BƯỚC 0: Xây dựng Lambda Layer**
- Dọn dẹp các builds trước đó
- Cài đặt production dependencies vào `layers/shared-dependencies/nodejs/`
- Kích thước Layer: ~15-20 MB

**BƯỚC 1: Xây dựng Lambda modules**
- Build auth triggers (Pre-Signup, Post-Confirmation, etc.)
- Build tất cả 7 Lambda modules
- Dọn dẹp dist và tsconfig.tsbuildinfo để buộc compile mới
- Biên dịch TypeScript → JavaScript

**BƯỚC 2: Xác thực dist folders**
- Kiểm tra dist folder tồn tại
- Phát hiện node_modules trong dist (vấn đề phổ biến)
- Kiểm tra kích thước dist (nên < 10 MB)
- Xác minh index.js tồn tại
- Tự động sửa chữa nếu phát hiện vấn đề

**BƯỚC 3: Chuẩn bị deployment packages**
- Dọn dẹp các artifacts trước đó
- Sao chép nội dung dist → thư mục deployment
- Thêm build-info.json (timestamp, git commit)

**BƯỚC 4: Triển khai với CDK**
- Tổng hợp CloudFormation templates
- Deploy stack `EveryoneCook-dev-Backend`
- Cập nhật Lambda functions
- Cập nhật Lambda Layer
- Cập nhật API Gateway

**Kết quả mẫu:**

```
========================================
Backend Deployment - dev
With Lambda Layers
========================================

STEP 0: Building Lambda Layer...
  Installing layer dependencies...
  Layer built: 18.45 MB

STEP 1: Building Lambda modules...
  Building auth triggers... OK
  Building api-router... OK
  Building auth-module... OK
  Building social-module... OK
  Building ai-module... OK
  Building admin-module... OK
  Building upload-module... OK

STEP 2: Validating dist folders...
  api-router... OK (125 files, 1.2 MB)
  auth-module... OK (156 files, 1.5 MB)
  social-module... OK (189 files, 1.8 MB)
  ai-module... OK (142 files, 1.3 MB)
  admin-module... OK (98 files, 0.9 MB)
  upload-module... OK (45 files, 0.4 MB)

STEP 3: Preparing deployment packages...
  Preparing api-router... Done
  Preparing auth-module... Done
  Preparing social-module... Done
  Preparing ai-module... Done
  Preparing admin-module... Done
  Preparing upload-module... Done

STEP 4: Deploying with CDK...
  Synthesizing CloudFormation templates...
  
EveryoneCook-dev-Backend
  Deploying...
  ✅ Lambda Layer updated
  ✅ API Router function updated
  ✅ Auth User function updated
  ✅ Social function updated
  ✅ Recipe AI function updated
  ✅ AI Worker updated
  ✅ Admin function updated
  ✅ Upload function updated
  
Outputs:
  ApiUrl = https://xinq7xh300.execute-api.ap-southeast-1.amazonaws.com/v1/
  ApiCustomDomain = https://api-dev.everyonecook.cloud

========================================
FULL DEPLOYMENT SUCCESSFUL!
========================================
```

#### Bước 3: Xác minh Triển khai

```powershell
# Liệt kê tất cả Lambda functions
aws lambda list-functions `
  --query 'Functions[?contains(FunctionName, `everyonecook-dev`)].{Name:FunctionName,Runtime:Runtime,Updated:LastModified}' `
  --output table

# Kết quả mong đợi:
# --------------------------------------------------------
# |                   ListFunctions                      |
# +------------------------------------------------------+
# |  Name                           | Runtime | Updated  |
# +------------------------------------------------------+
# |  everyonecook-dev-api-router    | nodejs20.x | ...   |
# |  everyonecook-dev-auth-user     | nodejs20.x | ...   |
# |  everyonecook-dev-social        | nodejs20.x | ...   |
# |  everyonecook-dev-recipe-ai     | nodejs20.x | ...   |
# |  everyonecook-dev-ai-worker     | nodejs20.x | ...   |
# |  everyonecook-dev-admin         | nodejs20.x | ...   |
# |  everyonecook-dev-upload        | nodejs20.x | ...   |
# +------------------------------------------------------+
```

---

### Tùy chọn B: Triển khai Nhanh (Chỉ Lambda Code)

Nếu chỉ thay đổi Lambda code (không thay đổi infrastructure), dùng fast deployment.

#### Bước 1: Triển khai Chỉ Lambda

```powershell
# Fast deploy - chỉ update Lambda code
.\deploy\deploy-backend.ps1 -Environment dev -LambdaOnly

# Hoặc dùng script chuyên dụng
.\deploy\force-update-lambdas.ps1 -Environment dev
```

**Quy trình triển khai nhanh hơn:**

```
BƯỚC 0: Build Layer (hoặc bỏ qua)
BƯỚC 1: Build modules
BƯỚC 2: Validate dist
BƯỚC 3: Chuẩn bị packages
BƯỚC 4: Upload lên Lambda (Fast Deploy)
  - Tạo ZIP files
  - Upload trực tiếp qua aws lambda update-function-code
  - Cập nhật Layer ARN nếu thay đổi
  - Bỏ qua CDK (nhanh hơn)
```

**Thời gian:**
- Full Deploy (CDK): ~5-8 phút
- Lambda Only: ~2-3 phút

#### Bước 2: Các Tùy chọn Bỏ qua (Nâng cao)

```powershell
# Bỏ qua build layer (sử dụng layer hiện có)
.\deploy\deploy-backend.ps1 -Environment dev -SkipLayer

# Bỏ qua build module (sử dụng dist hiện có)
.\deploy\deploy-backend.ps1 -Environment dev -SkipBuild

# Kết hợp các tùy chọn
.\deploy\deploy-backend.ps1 -Environment dev -LambdaOnly -SkipLayer
```

---

### Bước 4: Xác minh Lambda Functions

#### 1. Kiểm tra Cấu hình Function

```powershell
# Lấy chi tiết API Router
aws lambda get-function `
  --function-name everyonecook-dev-api-router `
  --query 'Configuration.{Runtime:Runtime,Handler:Handler,Timeout:Timeout,Memory:MemorySize,Layer:Layers[0].Arn}'

# Kết quả mong đợi:
# {
#   "Runtime": "nodejs20.x",
#   "Handler": "services/api-router/dist/handlers/index.handler",
#   "Timeout": 30,
#   "Memory": 512,
#   "Layer": "arn:aws:lambda:ap-southeast-1:...:layer:everyonecook-shared-deps-dev:X"
# }
```

#### 2. Xác minh Code SHA256 (Code đã được cập nhật)

```powershell
# Kiểm tra code hash
aws lambda get-function `
  --function-name everyonecook-dev-api-router `
  --query 'Configuration.CodeSha256'

# SHA256 sẽ thay đổi mỗi khi code được cập nhật
```

#### 3. Kiểm tra Biến Môi trường

```powershell
# Lấy biến môi trường
aws lambda get-function-configuration `
  --function-name everyonecook-dev-auth-user `
  --query 'Environment.Variables'

# Kết quả mong đợi:
# {
#   "TABLE_NAME": "EveryoneCook-dev",
#   "REGION": "ap-southeast-1",
#   "USER_POOL_ID": "ap-southeast-1_PKoL34PF0",
#   ...
# }
```

---

### Bước 5: Kiểm thử Lambda Functions

#### 1. Kiểm thử API Router (Health Check)

```powershell
# Gọi API Router với test event
aws lambda invoke `
  --function-name everyonecook-dev-api-router `
  --payload '{"httpMethod":"GET","path":"/health","headers":{}}' `
  response.json

# Kiểm tra response
Get-Content response.json | ConvertFrom-Json

# Kết quả mong đợi:
# {
#   "statusCode": 200,
#   "body": "{\"status\":\"healthy\",\"timestamp\":\"...\"}"
# }
```

#### 2. Làm nóng Functions (Tránh Cold Start)

```powershell
# Làm nóng tất cả functions
$functions = @(
  "everyonecook-dev-api-router",
  "everyonecook-dev-auth-user",
  "everyonecook-dev-social",
  "everyonecook-dev-recipe-ai",
  "everyonecook-dev-admin",
  "everyonecook-dev-upload"
)

foreach ($func in $functions) {
  Write-Host "Warming up $func..." -ForegroundColor Cyan
  aws lambda invoke `
    --function-name $func `
    --payload '{"warmup":true}' `
    out.json | Out-Null
}

Write-Host "All functions warmed up!" -ForegroundColor Green
```

---

### Bước 6: Kiểm tra CloudWatch Logs

#### 1. Theo dõi Logs Real-time

```powershell
# Theo dõi API Router logs
aws logs tail /aws/lambda/everyonecook-dev-api-router --follow

# Xem logs 10 phút gần nhất
aws logs tail /aws/lambda/everyonecook-dev-api-router --since 10m
```

#### 2. Tìm kiếm Lỗi

```powershell
# Tìm kiếm lỗi trong giờ qua
$oneHourAgo = [DateTimeOffset]::Now.AddHours(-1).ToUnixTimeMilliseconds()

aws logs filter-log-events `
  --log-group-name /aws/lambda/everyonecook-dev-api-router `
  --start-time $oneHourAgo `
  --filter-pattern "ERROR"
```

#### 3. Kiểm tra Tất cả Lambda Logs

```powershell
# Liệt kê tất cả log groups
aws logs describe-log-groups `
  --log-group-name-prefix /aws/lambda/everyonecook-dev `
  --query 'logGroups[].logGroupName'
```

---

### Bước 7: Xác minh Tích hợp API Gateway

#### 1. Lấy API Endpoint

```powershell
# Lấy API URL từ CloudFormation
$apiUrl = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Backend `
  --query 'Stacks[0].Outputs[?OutputKey==`ApiCustomDomain`].OutputValue' `
  --output text

Write-Host "API Endpoint: $apiUrl"
# Output: https://api-dev.everyonecook.cloud
```

#### 2. Kiểm thử Health Endpoint

```powershell
# Kiểm thử qua API Gateway (endpoint thực)
$response = Invoke-RestMethod -Uri "$apiUrl/health" -Method Get
$response | ConvertTo-Json

# Kết quả mong đợi:
# {
#   "status": "healthy",
#   "timestamp": "2025-12-09T...",
#   "service": "EveryoneCook API",
#   "environment": "dev"
# }
```

#### 3. Xác minh Triển khai API Gateway

```powershell
# Lấy API ID
$apiId = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Backend `
  --query 'Stacks[0].Outputs[?OutputKey==`ApiId`].OutputValue' `
  --output text

# Liệt kê deployments
aws apigateway get-deployments --rest-api-id $apiId

# Lấy deployment mới nhất
aws apigateway get-deployment `
  --rest-api-id $apiId `
  --deployment-id (aws apigateway get-deployments `
    --rest-api-id $apiId `
    --query 'items[0].id' `
    --output text)
```

---

### Bước 8: Xác minh Cognito Triggers

Cognito triggers đã được deploy với Auth Stack. Xác minh:

```powershell
# Liệt kê Cognito User Pool triggers
$userPoolId = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Auth `
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' `
  --output text

aws cognito-idp describe-user-pool `
  --user-pool-id $userPoolId `
  --query 'UserPool.LambdaConfig'

# Kết quả mong đợi:
# {
#   "PreSignUp": "arn:aws:lambda:...:function:EveryoneCook-dev-PreSignup",
#   "PostConfirmation": "arn:aws:lambda:...:function:EveryoneCook-dev-PostConfirmation",
#   "PostAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PostAuthentication",
#   "PreAuthentication": "arn:aws:lambda:...:function:EveryoneCook-dev-PreAuthentication",
#   "CustomMessage": "arn:aws:lambda:...:function:EveryoneCook-dev-CustomMessage"
# }
```

---

### Bước 9: Xác minh SQS Workers

#### 1. Kiểm tra AI Worker

```powershell
# Lấy AI Worker function
aws lambda get-function --function-name everyonecook-dev-ai-worker

# Kiểm tra event source mapping (SQS trigger)
aws lambda list-event-source-mappings `
  --function-name everyonecook-dev-ai-worker

# Kết quả mong đợi: Event source từ AI Queue
```

#### 2. Kiểm thử Worker với SQS Message

```powershell
# Lấy AI Queue URL
$queueUrl = aws sqs list-queues `
  --queue-name-prefix everyonecook-dev-ai-queue `
  --query 'QueueUrls[0]' `
  --output text

# Gửi test message
aws sqs send-message `
  --queue-url $queueUrl `
  --message-body '{"type":"test","data":"worker-verification"}'

# Kiểm tra worker logs
aws logs tail /aws/lambda/everyonecook-dev-ai-worker --follow
```

---

### Danh sách Kiểm tra Triển khai

Sử dụng danh sách này để xác minh deployment:

#### Build & Package
- [ ] Lambda Layer được build thành công (~15-20 MB)
- [ ] Tất cả 7 modules được biên dịch (TypeScript → JavaScript)
- [ ] Dist folders được xác thực (không có node_modules, < 10 MB)
- [ ] Deployment packages được tạo với build-info.json

#### Lambda Functions
- [ ] Tất cả 7 Lambda functions được triển khai
- [ ] Runtime: nodejs20.x
- [ ] Handler paths chính xác
- [ ] Timeout: 30 giây
- [ ] Memory: 512 MB (auth, social), 1024 MB (ai)

- [ ] Lambda Layer được đính kèm vào tất cả functions
- [ ] Biến môi trường được cấu hình đúng
- [ ] Code SHA256 đã thay đổi (code đã cập nhật)

#### Cognito Triggers
- [ ] Pre-Signup trigger được triển khai
- [ ] Post-Confirmation trigger được triển khai
- [ ] Post-Authentication trigger được triển khai
- [ ] Pre-Authentication trigger được triển khai
- [ ] Custom Message trigger được triển khai

#### API Gateway
- [ ] API Gateway được triển khai
- [ ] Custom domain được cấu hình: `api-dev.everyonecook.cloud`
- [ ] Health endpoint phản hồi: `GET /health`
- [ ] CORS được cấu hình đúng

#### Workers & Queues
- [ ] AI Worker được triển khai
- [ ] Event source mapping được cấu hình (SQS → Lambda)
- [ ] SQS queues được tạo (ai-queue, image-queue, analytics-queue, notification-queue)
- [ ] Worker có thể xử lý test messages

#### Xác minh
- [ ] Tất cả functions được gọi thành công
- [ ] Không có lỗi trong CloudWatch logs
- [ ] API Gateway trả về 200 cho health check
- [ ] Functions đã được làm nóng (không có cold start)

---

### Xử lý Sự cố

#### Vấn đề 1: Build Thất bại - Lỗi Biên dịch TypeScript

```powershell
# Kiểm tra lỗi TypeScript
cd services/auth-module
npx tsc --noEmit

# Sửa lỗi và rebuild
npm run build
```

#### Vấn đề 2: Thư mục Dist Quá Lớn

```powershell
# Kiểm tra node_modules trong dist
Get-ChildItem -Path services/*/dist/node_modules -Recurse

# Script sẽ tự động xóa, hoặc thủ công:
Remove-Item services/auth-module/dist/node_modules -Recurse -Force

# Rebuild
cd services/auth-module
npm run build
```

#### Vấn đề 3: Cập nhật Lambda Thất bại

```powershell
# Kiểm tra trạng thái function
aws lambda get-function `
  --function-name everyonecook-dev-auth-user `
  --query 'Configuration.{State:State,StateReason:StateReason}'

# Nếu state là "Failed", kiểm tra logs
aws logs tail /aws/lambda/everyonecook-dev-auth-user --since 10m
```

#### Vấn đề 4: Function Timeout

```powershell
# Tăng timeout (qua CDK hoặc AWS CLI)
aws lambda update-function-configuration `
  --function-name everyonecook-dev-auth-user `
  --timeout 60

# Hoặc cập nhật trong backend-stack.ts và redeploy
```

#### Vấn đề 5: Hết Bộ nhớ

```powershell
# Tăng memory
aws lambda update-function-configuration `
  --function-name everyonecook-dev-recipe-ai `
  --memory-size 1024

# AI functions cần 1024 MB cho Bedrock calls
```

#### Vấn đề 6: Layer Không Được Đính kèm

```powershell
# Lấy layer ARN
$layerArn = aws lambda list-layer-versions `
  --layer-name everyonecook-shared-deps-dev `
  --query 'LayerVersions[0].LayerVersionArn' `
  --output text

# Đính kèm layer thủ công
aws lambda update-function-configuration `
  --function-name everyonecook-dev-auth-user `
  --layers $layerArn
```

#### Vấn đề 7: Thiếu Biến Môi trường

```powershell
# Kiểm tra biến môi trường hiện tại
aws lambda get-function-configuration `
  --function-name everyonecook-dev-auth-user `
  --query 'Environment.Variables'

# Cập nhật thủ công (hoặc qua CDK)
aws lambda update-function-configuration `
  --function-name everyonecook-dev-auth-user `
  --environment "Variables={TABLE_NAME=EveryoneCook-dev,REGION=ap-southeast-1}"
```

---

### Giám sát Hiệu suất

#### 1. Lambda Metrics (CloudWatch)

```powershell
# Lấy số lần gọi (1 giờ qua)
$startTime = (Get-Date).AddHours(-1).ToString("yyyy-MM-ddTHH:mm:ss")
$endTime = (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss")

aws cloudwatch get-metric-statistics `
  --namespace AWS/Lambda `
  --metric-name Invocations `
  --dimensions Name=FunctionName,Value=everyonecook-dev-api-router `
  --start-time $startTime `
  --end-time $endTime `
  --period 300 `
  --statistics Sum
```

#### 2. Tỷ lệ Lỗi

```powershell
# Lấy số lượng lỗi
aws cloudwatch get-metric-statistics `
  --namespace AWS/Lambda `
  --metric-name Errors `
  --dimensions Name=FunctionName,Value=everyonecook-dev-api-router `
  --start-time $startTime `
  --end-time $endTime `
  --period 300 `
  --statistics Sum
```

#### 3. Thời gian Thực thi (Trung bình & P99)

```powershell
# Lấy thời gian thực thi trung bình
aws cloudwatch get-metric-statistics `
  --namespace AWS/Lambda `
  --metric-name Duration `
  --dimensions Name=FunctionName,Value=everyonecook-dev-api-router `
  --start-time $startTime `
  --end-time $endTime `
  --period 300 `
  --statistics Average,Maximum
```

#### 4. Thực thi Đồng thời

```powershell
# Kiểm tra số lượng thực thi đồng thời
aws cloudwatch get-metric-statistics `
  --namespace AWS/Lambda `
  --metric-name ConcurrentExecutions `
  --dimensions Name=FunctionName,Value=everyonecook-dev-api-router `
  --start-time $startTime `
  --end-time $endTime `
  --period 300 `
  --statistics Maximum
```

---

### Thực hành Tốt nhất

#### 1. Chiến lược Triển khai

**Development:**
```powershell
# Lặp nhanh - Chỉ Lambda
.\deploy\deploy-backend.ps1 -LambdaOnly
```

**Staging/Production:**
```powershell
# Triển khai đầy đủ với CDK
.\deploy\deploy-backend.ps1 -Environment prod

# Xác minh kỹ lưỡng trước khi tiếp tục
```

#### 2. Chiến lược Rollback

```powershell
# Lấy phiên bản trước đó
aws lambda list-versions-by-function `
  --function-name everyonecook-dev-auth-user

# Rollback về phiên bản X
aws lambda update-alias `
  --function-name everyonecook-dev-auth-user `
  --name LIVE `
  --function-version X
```

#### 3. Triển khai Blue-Green

```powershell
# Triển khai lên phiên bản mới
.\deploy\deploy-backend.ps1 -Environment dev

# Kiểm thử phiên bản mới
# Nếu OK, chuyển traffic hoàn tất
# Nếu có vấn đề, rollback alias
```

#### 4. Cảnh báo Giám sát

```powershell
# Tạo CloudWatch alarm cho lỗi
aws cloudwatch put-metric-alarm `
  --alarm-name everyonecook-dev-api-router-errors `
  --alarm-description "Alert on Lambda errors" `
  --metric-name Errors `
  --namespace AWS/Lambda `
  --statistic Sum `
  --period 300 `
  --evaluation-periods 1 `
  --threshold 10 `
  --comparison-operator GreaterThanThreshold `
  --dimensions Name=FunctionName,Value=everyonecook-dev-api-router
```

---

### Benchmarks Hiệu suất

**Thời gian Triển khai Dự kiến:**

| Thao tác | Thời gian | Ghi chú |
|----------|-----------|---------|
| Build Layer | 30-60s | npm install production deps |
| Build Modules | 20-40s | Biên dịch TypeScript (7 modules) |
| Validate & Package | 10-20s | Tạo ZIP |
| CDK Deploy | 3-5 phút | Cập nhật CloudFormation |
| Lambda Only | 1-2 phút | Cập nhật function trực tiếp |

**Hiệu suất Lambda Dự kiến:**

| Function | Cold Start | Warm | Memory | Timeout |
|----------|-----------|------|--------|---------|
| api-router | 800-1200ms | 50-100ms | 512 MB | 30s |
| auth-user | 600-900ms | 80-150ms | 512 MB | 30s |
| social | 700-1000ms | 100-200ms | 512 MB | 30s |
| recipe-ai | 1500-2500ms | 200-500ms | 1024 MB | 60s |
| ai-worker | 2000-3000ms | 300-800ms | 1024 MB | 300s |
| admin | 600-800ms | 100-150ms | 512 MB | 30s |
| upload | 400-600ms | 50-80ms | 256 MB | 15s |

---

### Nâng cao: Tích hợp CI/CD

Để tự động hóa deployment trong GitLab CI/CD:

```yaml
# .gitlab-ci.yml
deploy-backend:
  stage: deploy
  script:
    - cd everyonecook
    - .\deploy\deploy-backend.ps1 -Environment $CI_ENVIRONMENT_NAME -LambdaOnly
  only:
    - main
  environment:
    name: production
```

---

### Tóm tắt

Trong lab này, bạn đã:

✅ **Triển khai Lambda Functions**: 7 modules + 5 Cognito triggers  
✅ **Triển khai Lambda Layer**: Shared dependencies  
✅ **Cấu hình API Gateway**: Tích hợp custom domain  
✅ **Thiết lập SQS Workers**: Xử lý bất đồng bộ  
✅ **Xác minh Triển khai**: Health checks, logs, metrics  
✅ **Tối ưu Hiệu suất**: Làm nóng functions, tối ưu cold start  

**Thành tựu Chính:**
- Triển khai tự động với PowerShell script
- Full CDK deployment hoặc cập nhật Lambda-only nhanh
- Xác thực toàn diện và xử lý lỗi
- Giám sát và cảnh báo CloudWatch
- Cấu hình Lambda production-ready

---

### Các Bước Tiếp theo

Backend đã được triển khai thành công! Tiếp theo:

1. ✅ **Kiểm thử Endpoints**: Xác minh tất cả API endpoints → [5.08 - Test Endpoints](../5.08-test-endpoints/)
2. 📝 **Quản lý Phiên bản**: Push code lên GitLab → [5.09 - Push to GitLab](../5.09-push-gitlab/)
3. 🚀 **Triển khai Frontend**: Ứng dụng Next.js
4. 🔍 **Giám sát**: CloudWatch dashboards và alerts

Tiếp tục đến: [5.08 - Test Endpoints End-to-End](../5.08-test-endpoints/)

```bash
# Kiểm thử get posts
aws lambda invoke \
  --function-name EveryoneCook-dev-SocialModule \
  --payload '{
    "httpMethod":"GET",
    "path":"/social/posts",
    "headers":{"Authorization":"Bearer test-token"},
    "requestContext":{"authorizer":{"claims":{"sub":"test-user-id","username":"testuser"}}}
  }' \
  response.json

cat response.json
```

#### Bước 5: Kiểm tra CloudWatch Logs

**1. Xem Logs Gần đây**

```bash
# Theo dõi Auth Module logs
aws logs tail /aws/lambda/EveryoneCook-dev-AuthModule --follow

# Hoặc xem 10 phút gần nhất
aws logs tail /aws/lambda/EveryoneCook-dev-AuthModule --since 10m
```

**2. Tìm kiếm Lỗi**

```bash
# Tìm kiếm lỗi trong giờ qua
aws logs filter-log-events \
  --log-group-name /aws/lambda/EveryoneCook-dev-AuthModule \
  --start-time $(date -d '1 hour ago' +%s)000 \
  --filter-pattern "ERROR"
```

**3. Kiểm tra Lambda Insights**

```bash
# Lấy Lambda metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=EveryoneCook-dev-AuthModule \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum
```

![CloudWatch Logs](/images/5-Workshop/5.7-deploy-backend/cloudwatch-logs.png)
*Screenshot: CloudWatch Logs hiển thị Lambda execution*

#### Bước 6: Triển khai Lambda Triggers

**Lambda triggers đã được triển khai với Auth Stack, xác minh:**

```bash
# Kiểm tra Post-Confirmation trigger
aws lambda get-function \
  --function-name EveryoneCook-dev-PostConfirmationTrigger

# Kiểm thử trigger (sẽ được gọi tự động khi user confirmation)
```

#### Bước 7: Triển khai Search Sync Worker

**1. Triển khai Worker**

```bash
# Worker được triển khai với Backend Stack
# Xác minh deployment
aws lambda get-function \
  --function-name EveryoneCook-dev-SearchSyncWorker
```

**2. Kiểm thử Worker với SQS**

```bash
# Lấy SearchIndex queue URL
QUEUE_URL=$(aws sqs list-queues \
  --queue-name-prefix EveryoneCook-dev-SearchIndexQueue \
  | jq -r '.QueueUrls[0]')

# Gửi test message
aws sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body '{
    "eventName": "INSERT",
    "tableName": "recipes",
    "keys": {"PK": "USER#testuser", "SK": "RECIPE#test-123"},
    "newImage": {"title": "Test Recipe", "cuisine": "Vietnamese"}
  }'

# Kiểm tra worker logs
aws logs tail /aws/lambda/EveryoneCook-dev-SearchSyncWorker --follow
```

#### Bước 8: Cập nhật API Gateway

**API Gateway tự động cập nhật khi triển khai Backend Stack, xác minh:**

```bash
# Lấy API Gateway deployment
API_ID=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Backend \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiId`].OutputValue' \
  --output text)

# Liệt kê deployments
aws apigateway get-deployments --rest-api-id $API_ID

# Lấy deployment mới nhất
aws apigateway get-deployment \
  --rest-api-id $API_ID \
  --deployment-id $(aws apigateway get-deployments \
    --rest-api-id $API_ID \
    --query 'items[0].id' \
    --output text)
```

#### Bước 9: Làm nóng Lambda Functions

**Tránh cold start cho các requests đầu tiên:**

```bash
# Gọi tất cả functions một lần
for func in APIRouter AuthModule SocialModule RecipeAIModule AdminModule UploadModule SearchSyncWorker; do
  echo "Warming up $func..."
  aws lambda invoke \
    --function-name EveryoneCook-dev-$func \
    --payload '{"warmup":true}' \
    /dev/null
done
```

#### Bước 10: Xác minh End-to-End

**1. Kiểm thử Health Endpoint**

```bash
# Kiểm thử qua API Gateway
curl https://api.everyonecook.cloud/health

# Nên trả về: {"status":"healthy","timestamp":"..."}
```

**2. Kiểm thử với Postman**

Import Postman collection:
```json
{
  "info": {
    "name": "EveryoneCook API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Health Check",
      "request": {
        "method": "GET",
        "url": "https://api.everyonecook.cloud/health"
      }
    }
  ]
}
```

#### Danh sách Kiểm tra Triển khai

- [ ] Script chuẩn bị hoàn tất thành công
- [ ] Tất cả Lambda functions được cập nhật
- [ ] Trạng thái function: Active
- [ ] Các lần gọi kiểm thử thành công
- [ ] CloudWatch logs hiển thị executions
- [ ] Không có lỗi trong logs
- [ ] Lambda triggers hoạt động
- [ ] Search Sync Worker xử lý messages
- [ ] API Gateway được cập nhật
- [ ] Health endpoint phản hồi
- [ ] Functions đã được làm nóng

#### Xử lý Sự cố

**Vấn đề: Cập nhật Lambda thất bại**

```bash
# Kiểm tra trạng thái function
aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.State'

# Nếu state là "Failed", kiểm tra StateReasonCode
aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.StateReasonCode'
```

**Vấn đề: Function timeout**

```bash
# Tăng timeout
aws lambda update-function-configuration \
  --function-name EveryoneCook-dev-AuthModule \
  --timeout 30
```

**Vấn đề: Hết bộ nhớ**

```bash
# Tăng memory
aws lambda update-function-configuration \
  --function-name EveryoneCook-dev-AuthModule \
  --memory-size 512
```

**Vấn đề: Thiếu biến môi trường**

```bash
# Cập nhật biến môi trường
aws lambda update-function-configuration \
  --function-name EveryoneCook-dev-AuthModule \
  --environment Variables={TABLE_NAME=EveryoneCook-dev,REGION=us-east-1}
```

#### Giám sát Hiệu suất

**Kiểm tra Lambda metrics:**

```bash
# Invocations
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=EveryoneCook-dev-AuthModule \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# Errors
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=EveryoneCook-dev-AuthModule \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# Duration
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=EveryoneCook-dev-AuthModule \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average,Maximum
```

#### Các Bước Tiếp theo

Sau khi backend được triển khai và xác minh, tiếp tục đến [Test Endpoints End-to-End](../5.8-test-endpoints/) để kiểm thử luồng ứng dụng hoàn chỉnh.
