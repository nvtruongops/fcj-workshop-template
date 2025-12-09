---
title: "Cấu hình API & Lambda"
date: 2025-12-01
weight: 6
chapter: false
pre: " <b> 5.06. </b> "
---

#### Tổng quan

Sau khi deploy infrastructure, bạn cần cấu hình API Gateway routes và Lambda functions để xử lý business logic.

#### Kiến trúc API

```
API Gateway (api.everyonecook.cloud)
    ↓
API Router Lambda (entry point)
    ↓
┌─────────┬─────────┬─────────┬─────────┬─────────┐
│  Auth   │ Social  │ Recipe  │ Admin   │ Upload  │
│  Module │ Module  │ AI      │ Module  │ Module  │
└─────────┴─────────┴─────────┴─────────┴─────────┘
    ↓           ↓         ↓         ↓         ↓
DynamoDB    DynamoDB  Bedrock  DynamoDB    S3
```

#### Bước 1: Review API Routes

**Cấu trúc API Router:**

```typescript
// services/api-router/routes/index.ts

export const routes = {
  // Auth routes
  'POST /auth/register': 'auth-module',
  'POST /auth/login': 'auth-module',
  'GET /auth/profile': 'auth-module',
  'PUT /auth/profile': 'auth-module',
  
  // Social routes
  'GET /social/posts': 'social-module',
  'POST /social/posts': 'social-module',
  'GET /social/posts/:id': 'social-module',
  'POST /social/posts/:id/like': 'social-module',
  
  // Recipe routes
  'GET /recipes': 'recipe-module',
  'POST /recipes': 'recipe-module',
  'POST /ai/generate-recipe': 'recipe-module',
  'POST /ai/search': 'recipe-module',
  
  // Admin routes
  'GET /admin/users': 'admin-module',
  'POST /admin/users/:id/ban': 'admin-module',
  
  // Upload routes
  'POST /upload/presigned-url': 'upload-module',
  'POST /upload/complete': 'upload-module'
};
```

#### Bước 2: Cấu hình Lambda Modules

**1. Auth Module**

```bash
cd services/auth-module

# Review cấu hình
cat package.json
cat tsconfig.json

# Kiểm tra environment variables cần thiết
cat index.ts | grep process.env
```

**Environment Variables:**
- `TABLE_NAME`: Tên DynamoDB table
- `USER_POOL_ID`: Cognito User Pool ID
- `REGION`: AWS region

**2. Social Module**

```bash
cd services/social-module

# Review handlers
ls handlers/
# - posts.handler.ts
# - comments.handler.ts
# - reactions.handler.ts
# - friends.handler.ts
```

**3. Recipe/AI Module**

```bash
cd services/recipe-module

# Kiểm tra cấu hình AI
cat services/ai.service.ts
```

**Environment Variables:**
- `BEDROCK_MODEL_ID`: Claude 3.5 Sonnet v2
- `BEDROCK_REGION`: us-east-1
- `OPENSEARCH_ENDPOINT`: OpenSearch domain (nếu enabled)

**4. Admin Module**

```bash
cd services/admin-module

# Review admin operations
ls handlers/
```

**5. Upload Module**

```bash
cd services/upload-module

# Kiểm tra cấu hình S3
cat services/s3.service.ts
```

**Environment Variables:**
- `CONTENT_BUCKET`: Tên S3 content bucket
- `CLOUDFRONT_DOMAIN`: CloudFront domain
- `CLOUDFRONT_KEY_PAIR_ID`: Cho signed URLs

#### Bước 3: Cấu hình Lambda Layers

**Shared Dependencies:**

```bash
cd services/shared

# Review shared code
ls -la
# - repositories/
# - business-logic/
# - utils/
# - types/
```

**Cấu trúc Lambda Layer:**
```
shared/
├── repositories/
│   ├── user.repository.ts
│   ├── post.repository.ts
│   └── recipe.repository.ts
├── business-logic/
│   ├── auth.logic.ts
│   └── validation.logic.ts
└── utils/
    ├── dynamodb.utils.ts
    └── response.utils.ts
```

#### Bước 4: Thiết lập Lambda Environment Variables

**1. Lấy Stack Outputs**

```bash
# Lấy DynamoDB table name
TABLE_NAME=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Core \
  --query 'Stacks[0].Outputs[?OutputKey==`DynamoDBTableName`].OutputValue' \
  --output text)

# Lấy User Pool ID
USER_POOL_ID=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Auth \
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' \
  --output text)

# Lấy Content Bucket
CONTENT_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Core \
  --query 'Stacks[0].Outputs[?OutputKey==`ContentBucketName`].OutputValue' \
  --output text)

echo "TABLE_NAME=$TABLE_NAME"
echo "USER_POOL_ID=$USER_POOL_ID"
echo "CONTENT_BUCKET=$CONTENT_BUCKET"
```

**2. Cập nhật Lambda Environment Variables**

Lambda environment variables được thiết lập tự động bởi CDK stack, nhưng bạn có thể xác minh:

```bash
# Kiểm tra Auth Module environment variables
aws lambda get-function-configuration \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Environment.Variables'

# Phải hiển thị:
# {
#   "TABLE_NAME": "EveryoneCook-dev",
#   "USER_POOL_ID": "us-east-1_ABC123",
#   "REGION": "us-east-1"
# }
```

#### Bước 5: Cấu hình API Gateway

**1. Review API Gateway Configuration**

```bash
# Lấy API Gateway ID
API_ID=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Backend \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiId`].OutputValue' \
  --output text)

# Lấy API details
aws apigateway get-rest-api --rest-api-id $API_ID
```

**2. Kiểm tra Cognito Authorizer**

```bash
# Liệt kê authorizers
aws apigateway get-authorizers --rest-api-id $API_ID

# Phải hiển thị Cognito User Pool authorizer
```

**3. Xác minh Custom Domain**

```bash
# Kiểm tra custom domain mapping
aws apigateway get-domain-name --domain-name api.everyonecook.cloud

# Phải hiển thị:
# - Domain name: api.everyonecook.cloud
# - Certificate ARN
# - Regional domain name
```

#### Bước 6: Cấu hình SQS Queues

**1. Liệt kê Tất cả Queues**

```bash
# Liệt kê SQS queues
aws sqs list-queues | grep EveryoneCook-dev

# Phải hiển thị 12 queues (6 main + 6 DLQ)
```

**2. Cấu hình Queue Permissions**

Permissions được thiết lập tự động bởi CDK, xác minh:

```bash
# Lấy AI Queue URL
AI_QUEUE_URL=$(aws sqs list-queues \
  --queue-name-prefix EveryoneCook-dev-AIQueue \
  | jq -r '.QueueUrls[0]')

# Kiểm tra queue attributes
aws sqs get-queue-attributes \
  --queue-url $AI_QUEUE_URL \
  --attribute-names All
```

#### Bước 7: Cấu hình Lambda Triggers

**Cognito Lambda Triggers:**

```bash
# Kiểm tra User Pool triggers
aws cognito-idp describe-user-pool \
  --user-pool-id $USER_POOL_ID \
  --query 'UserPool.LambdaConfig'

# Phải hiển thị 5 triggers:
# - PreSignUp
# - PostConfirmation
# - PreAuthentication
# - PostAuthentication
# - CustomMessage
```

#### Bước 8: Test Lambda Functions Locally

**1. Test Auth Module**

```bash
cd services/auth-module

# Chạy tests
npm test

# Test specific handler
npm run test:unit -- handlers/profile.test.ts
```

**2. Test API Router**

```bash
cd services/api-router

# Test routing logic
npm test

# Test với sample event
node -e "
const handler = require('./dist/index').handler;
const event = {
  httpMethod: 'GET',
  path: '/health',
  headers: {}
};
handler(event).then(console.log);
"
```

#### Bước 9: Xác minh IAM Permissions

**1. Kiểm tra Lambda Execution Roles**

```bash
# Liệt kê Lambda functions
aws lambda list-functions \
  --query 'Functions[?contains(FunctionName, `EveryoneCook-dev`)].FunctionName'

# Kiểm tra role cho Auth Module
aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.Role'
```

**2. Xác minh DynamoDB Permissions**

```bash
# Lấy role name
ROLE_NAME=$(aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.Role' \
  --output text | cut -d'/' -f2)

# Liệt kê attached policies
aws iam list-attached-role-policies --role-name $ROLE_NAME

# Phải bao gồm DynamoDB access policy
```

#### Checklist Cấu hình

- [ ] API routes đã review và hiểu rõ
- [ ] Cấu trúc Lambda modules đã review
- [ ] Environment variables đã xác minh
- [ ] Lambda layers đã cấu hình
- [ ] API Gateway custom domain hoạt động
- [ ] Cognito authorizer đã cấu hình
- [ ] SQS queues đã tạo và accessible
- [ ] Lambda triggers đã attach vào Cognito
- [ ] IAM permissions đã xác minh
- [ ] Local tests passing

#### Bước tiếp theo

Sau khi hoàn thành cấu hình, tiếp tục với [Deploy Backend Services](../5.7-deploy-backend/) để deploy Lambda code của bạn.
