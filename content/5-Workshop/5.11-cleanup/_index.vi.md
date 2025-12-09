---
title: "Dọn dẹp tài nguyên"
date: 2025-12-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Tổng quan

Khi bạn hoàn thành workshop và không muốn tiếp tục sử dụng, hãy xóa tất cả tài nguyên để tránh phát sinh chi phí.

**⚠️ Cảnh báo:** Quá trình này không thể hoàn tác. Tất cả dữ liệu sẽ bị xóa vĩnh viễn.

#### Thứ tự Dọn dẹp

Phải xóa theo thứ tự ngược lại với deployment:

```
1. Amplify App (Frontend)
2. Observability Stack
3. Backend Stack
4. Auth Stack
5. Core Stack
6. Certificate Stack
7. DNS Stack
8. CDK Bootstrap (tùy chọn)
```

#### Bước 1: Xóa Amplify App

**1. Xóa qua Console**

1. Vào AWS Amplify Console
2. Chọn app của bạn
3. Click "Actions" → "Delete app"
4. Nhập tên app để xác nhận
5. Click "Delete"

**2. Xóa qua CLI**

```bash
# Lấy app ID
APP_ID=$(aws amplify list-apps \
  --query 'apps[?name==`everyonecook-frontend`].appId' \
  --output text)

# Xóa app
aws amplify delete-app --app-id $APP_ID
```

#### Bước 2: Làm trống S3 Buckets

**S3 buckets phải được làm trống trước khi xóa:**

```bash
# Liệt kê tất cả EveryoneCook buckets
aws s3 ls | grep everyonecook

# Làm trống từng bucket (4 buckets)
aws s3 rm s3://everyonecook-content-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-logs-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-emails-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-cdn-logs-dev-xxxxx --recursive

# Hoặc sử dụng PowerShell script
$buckets = aws s3 ls | Select-String "everyonecook" | ForEach-Object { $_.ToString().Split()[-1] }
foreach ($bucket in $buckets) {
  Write-Host "Emptying bucket: $bucket"
  aws s3 rm "s3://$bucket" --recursive
}
```

#### Bước 3: Xóa CDK Stacks

**1. Xóa Observability Stack**

```bash
cd infrastructure

# Xóa Observability stack
npx cdk destroy EveryoneCook-dev-Observability --context environment=dev

# Nhập 'y' để xác nhận
```

**2. Xóa Backend Stack**

```bash
# Xóa Backend stack
npx cdk destroy EveryoneCook-dev-Backend --context environment=dev

# Nhập 'y' để xác nhận
```

**3. Xóa Auth Stack**

```bash
# Xóa Auth stack
npx cdk destroy EveryoneCook-dev-Auth --context environment=dev

# Nhập 'y' để xác nhận
```

**4. Xóa Core Stack**

```bash
# Xóa Core stack (mất 15-20 phút do CloudFront)
npx cdk destroy EveryoneCook-dev-Core --context environment=dev

# Nhập 'y' để xác nhận
```

**5. Xóa Certificate Stack**

```bash
# Xóa Certificate stack
npx cdk destroy EveryoneCook-dev-Certificate --context environment=dev

# Nhập 'y' để xác nhận
```

**6. Xóa DNS Stack (Tùy chọn)**

```bash
# Xóa DNS stack (chỉ nếu bạn không cần domain)
npx cdk destroy EveryoneCook-dev-DNS --context environment=dev

# Nhập 'y' để xác nhận
```

#### Bước 4: Xác minh Tất cả Stacks đã Xóa

```bash
# Liệt kê các stacks còn lại
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query 'StackSummaries[?contains(StackName, `EveryoneCook-dev`)].StackName'

# Phải trả về mảng rỗng
```

#### Bước 5: Xóa CDK Bootstrap (Tùy chọn)

**⚠️ Chỉ làm điều này nếu bạn đã xong hoàn toàn với CDK:**

```bash
# Xóa CDK bootstrap stack
aws cloudformation delete-stack --stack-name CDKToolkit

# Làm trống và xóa CDK assets bucket
BUCKET_NAME=$(aws s3 ls | grep cdk | awk '{print $3}')
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
```

#### Bước 6: Xóa GitLab Repository (Tùy chọn)

**1. Archive Repository**

1. Vào GitLab → Settings → General
2. Kéo xuống "Advanced"
3. Click "Archive project"

**2. Xóa Repository**

1. Vào GitLab → Settings → General
2. Kéo xuống "Advanced"
3. Click "Delete project"
4. Nhập tên project để xác nhận
5. Click "Yes, delete project"

#### Bước 7: Xác minh Dọn dẹp Hoàn tất

**1. Kiểm tra CloudFormation**

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  | grep EveryoneCook

# Không trả về gì
```

**2. Kiểm tra S3**

```bash
aws s3 ls | grep everyonecook

# Không trả về gì
```

**3. Kiểm tra Lambda**

```bash
aws lambda list-functions | grep EveryoneCook

# Không trả về gì
```

**4. Kiểm tra DynamoDB**

```bash
aws dynamodb list-tables | grep EveryoneCook

# Không trả về gì
```

**5. Kiểm tra API Gateway**

```bash
aws apigateway get-rest-apis | grep EveryoneCook

# Không trả về gì
```

**6. Kiểm tra Cognito**

```bash
aws cognito-idp list-user-pools --max-results 10 | grep EveryoneCook

# Không trả về gì
```

**7. Kiểm tra CloudFront**

```bash
aws cloudfront list-distributions | grep EveryoneCook

# Không trả về gì
```

**8. Kiểm tra OpenSearch**

```bash
aws opensearch list-domain-names | grep everyonecook

# Không trả về gì
```

**9. Kiểm tra Amplify**

```bash
aws amplify list-apps | grep everyonecook

# Không trả về gì
```

#### Chi phí Sau Dọn dẹp

**Ngay lập tức:**
- Hầu hết tài nguyên: $0/tháng
- Route 53 Hosted Zone: $0.50/tháng (nếu giữ lại)
- KMS keys: Đã lên lịch xóa (7-30 ngày), không tính phí trong thời gian chờ

**Sau 30 ngày:**
- Mọi thứ: $0/tháng (nếu DNS stack cũng đã xóa)

#### Khắc phục sự cố Dọn dẹp

**Vấn đề: S3 bucket xóa thất bại**

```bash
# Buộc làm trống và xóa
aws s3 rb s3://bucket-name --force
```

**Vấn đề: CloudFormation stack bị stuck**

```bash
# Kiểm tra stack events
aws cloudformation describe-stack-events \
  --stack-name EveryoneCook-dev-Core \
  --max-items 20

# Nếu stuck, bỏ qua tài nguyên failed
aws cloudformation delete-stack \
  --stack-name EveryoneCook-dev-Core \
  --retain-resources ResourceLogicalId
```

**Vấn đề: CloudFront distribution không thể xóa**

```bash
# Vô hiệu hóa distribution trước
DIST_ID=$(aws cloudfront list-distributions \
  --query "DistributionList.Items[?Comment=='EveryoneCook-dev'].Id" \
  --output text)

# Lấy ETag
ETAG=$(aws cloudfront get-distribution --id $DIST_ID \
  --query "ETag" --output text)

# Vô hiệu hóa distribution
aws cloudfront update-distribution \
  --id $DIST_ID \
  --if-match $ETAG \
  --distribution-config file://disabled-config.json

# Đợi 15-20 phút, sau đó xóa
aws cloudfront delete-distribution --id $DIST_ID --if-match $ETAG
```

**Vấn đề: DynamoDB table có deletion protection**

```bash
# Tắt deletion protection
aws dynamodb update-table \
  --table-name EveryoneCook-dev \
  --no-deletion-protection-enabled

# Sau đó xóa stack
```

#### Xác minh Cuối cùng

```bash
# Kiểm tra AWS billing
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost

# Phải hiển thị chi phí giảm dần
```

#### Checklist Dọn dẹp

- [ ] Amplify app đã xóa
- [ ] Tất cả S3 buckets đã làm trống và xóa
- [ ] Observability stack đã xóa
- [ ] Backend stack đã xóa
- [ ] Auth stack đã xóa
- [ ] Core stack đã xóa
- [ ] Certificate stack đã xóa
- [ ] DNS stack đã xóa (tùy chọn)
- [ ] CDK bootstrap đã xóa (tùy chọn)
- [ ] GitLab repository đã archive/xóa (tùy chọn)
- [ ] Không còn CloudFormation stacks
- [ ] Không còn Lambda functions
- [ ] Không còn DynamoDB tables
- [ ] Không còn S3 buckets
- [ ] Không còn CloudFront distributions
- [ ] Không còn Cognito user pools
- [ ] Billing hiển thị $0 hoặc chi phí tối thiểu

#### Kết luận

Bạn đã hoàn thành workshop EveryoneCook! Bạn đã học được:

 **Infrastructure as Code** với AWS CDK  
 **Serverless Architecture** với Lambda và API Gateway  
 **DynamoDB Single Table Design** với username-based PK  
 **OpenSearch** với Vietnamese analyzer  
 **CloudFront CDN** với Origin Access Control  
 **Cognito Authentication** với Lambda triggers  
 **Bedrock AI** integration  
 **GitLab** version control và CI/CD  
 **AWS Amplify** frontend deployment  
 **CloudWatch** monitoring và X-Ray tracing

**Tổng cộng đã deploy:**
- 7 CDK stacks
- 100+ AWS resources
- 6 Lambda modules + 1 worker
- Full-stack application
- CI/CD pipeline
- Production-ready infrastructure

Cảm ơn bạn đã hoàn thành workshop! 🎉
