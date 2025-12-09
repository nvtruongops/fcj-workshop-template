---
title: "Configure API & Lambda"
date: 2025-12-01
weight: 6
chapter: false
pre: " <b> 5.06. </b> "
---

#### Overview

Sau khi deploy infrastructure, bạn cần cấu hình API Gateway routes và Lambda functions để xử lý business logic.

#### API Architecture

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

#### Step 1: Review API Routes

**API Router Structure:**

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

#### Step 2: Configure Lambda Modules

**1. Auth Module**

```bash
cd services/auth-module

# Review configuration
cat package.json
cat tsconfig.json

# Check environment variables needed
cat index.ts | grep process.env
```

**Environment Variables:**
- `TABLE_NAME`: DynamoDB table name
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

# Check AI configuration
cat services/ai.service.ts
```

**Environment Variables:**
- `BEDROCK_MODEL_ID`: Claude 3.5 Sonnet v2
- `BEDROCK_REGION`: us-east-1
- `OPENSEARCH_ENDPOINT`: OpenSearch domain (if enabled)

**4. Admin Module**

```bash
cd services/admin-module

# Review admin operations
ls handlers/
```

**5. Upload Module**

```bash
cd services/upload-module

# Check S3 configuration
cat services/s3.service.ts
```

**Environment Variables:**
- `CONTENT_BUCKET`: S3 content bucket name
- `CLOUDFRONT_DOMAIN`: CloudFront domain
- `CLOUDFRONT_KEY_PAIR_ID`: For signed URLs

#### Step 3: Configure Lambda Layers

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

**Lambda Layer Structure:**
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

#### Step 4: Set Lambda Environment Variables

**1. Get Stack Outputs**

```bash
# Get DynamoDB table name
TABLE_NAME=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Core \
  --query 'Stacks[0].Outputs[?OutputKey==`DynamoDBTableName`].OutputValue' \
  --output text)

# Get User Pool ID
USER_POOL_ID=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Auth \
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' \
  --output text)

# Get Content Bucket
CONTENT_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Core \
  --query 'Stacks[0].Outputs[?OutputKey==`ContentBucketName`].OutputValue' \
  --output text)

echo "TABLE_NAME=$TABLE_NAME"
echo "USER_POOL_ID=$USER_POOL_ID"
echo "CONTENT_BUCKET=$CONTENT_BUCKET"
```

**2. Update Lambda Environment Variables**

Lambda environment variables được set tự động bởi CDK stack, nhưng bạn có thể verify:

```bash
# Check Auth Module environment variables
aws lambda get-function-configuration \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Environment.Variables'

# Should show:
# {
#   "TABLE_NAME": "EveryoneCook-dev",
#   "USER_POOL_ID": "us-east-1_ABC123",
#   "REGION": "us-east-1"
# }
```

#### Step 5: Configure API Gateway

**1. Review API Gateway Configuration**

```bash
# Get API Gateway ID
API_ID=$(aws cloudformation describe-stacks \
  --stack-name EveryoneCook-dev-Backend \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiId`].OutputValue' \
  --output text)

# Get API details
aws apigateway get-rest-api --rest-api-id $API_ID
```

**2. Check Cognito Authorizer**

```bash
# List authorizers
aws apigateway get-authorizers --rest-api-id $API_ID

# Should show Cognito User Pool authorizer
```

**3. Verify Custom Domain**

```bash
# Check custom domain mapping
aws apigateway get-domain-name --domain-name api.everyonecook.cloud

# Should show:
# - Domain name: api.everyonecook.cloud
# - Certificate ARN
# - Regional domain name
```

#### Step 6: Configure SQS Queues

**1. List All Queues**

```bash
# List SQS queues
aws sqs list-queues | grep EveryoneCook-dev

# Should show 12 queues (6 main + 6 DLQ)
```

**2. Configure Queue Permissions**

Permissions được set tự động bởi CDK, verify:

```bash
# Get AI Queue URL
AI_QUEUE_URL=$(aws sqs list-queues \
  --queue-name-prefix EveryoneCook-dev-AIQueue \
  | jq -r '.QueueUrls[0]')

# Check queue attributes
aws sqs get-queue-attributes \
  --queue-url $AI_QUEUE_URL \
  --attribute-names All
```

#### Step 7: Configure Lambda Triggers

**Cognito Lambda Triggers:**

```bash
# Check User Pool triggers
aws cognito-idp describe-user-pool \
  --user-pool-id $USER_POOL_ID \
  --query 'UserPool.LambdaConfig'

# Should show 5 triggers:
# - PreSignUp
# - PostConfirmation
# - PreAuthentication
# - PostAuthentication
# - CustomMessage
```

#### Step 8: Test Lambda Functions Locally

**1. Test Auth Module**

```bash
cd services/auth-module

# Run tests
npm test

# Test specific handler
npm run test:unit -- handlers/profile.test.ts
```

**2. Test API Router**

```bash
cd services/api-router

# Test routing logic
npm test

# Test with sample event
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

#### Step 9: Verify IAM Permissions

**1. Check Lambda Execution Roles**

```bash
# List Lambda functions
aws lambda list-functions \
  --query 'Functions[?contains(FunctionName, `EveryoneCook-dev`)].FunctionName'

# Check role for Auth Module
aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.Role'
```

**2. Verify DynamoDB Permissions**

```bash
# Get role name
ROLE_NAME=$(aws lambda get-function \
  --function-name EveryoneCook-dev-AuthModule \
  --query 'Configuration.Role' \
  --output text | cut -d'/' -f2)

# List attached policies
aws iam list-attached-role-policies --role-name $ROLE_NAME

# Should include DynamoDB access policy
```

#### Configuration Checklist

- [ ] API routes reviewed and understood
- [ ] Lambda modules structure reviewed
- [ ] Environment variables verified
- [ ] Lambda layers configured
- [ ] API Gateway custom domain working
- [ ] Cognito authorizer configured
- [ ] SQS queues created and accessible
- [ ] Lambda triggers attached to Cognito
- [ ] IAM permissions verified
- [ ] Local tests passing

#### Next Steps

Once configuration is complete, proceed to [Deploy Backend Services](../5.7-deploy-backend/) to deploy your Lambda code.
