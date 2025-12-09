---
title: "Test Endpoints End-to-End"
date: 2025-12-01
weight: 8
chapter: false
pre: " <b> 5.08. </b> "
---

#### Tổng quan

Sau khi deploy backend thành công, bạn cần test tất cả API endpoints để đảm bảo hệ thống hoạt động đúng từ đầu đến cuối. Workshop này hướng dẫn chi tiết cách test từng module của hệ thống EveryoneCook.

#### Kiến trúc API

Dự án EveryoneCook sử dụng **API Router Pattern** với các thành phần:

- **API Gateway**: REST API với custom domain `api-dev.everyonecook.cloud`
- **API Router Lambda**: Routing requests tới các module Lambda
- **5 Module Lambda**:
  - `auth-user-lambda`: Authentication & User Management
  - `social-lambda`: Posts, Comments, Reactions, Friends, Notifications
  - `recipe-ai-lambda`: Recipes & AI Features
  - `admin-lambda`: Admin Dashboard & Content Moderation
  - `upload-lambda`: File Upload với S3 Presigned URLs
- **4 SQS Queues**: Async processing (AI, Image, Analytics, Notifications)
- **WAF**: Web Application Firewall bảo vệ API

#### Luồng Test

```
1. Lấy API Endpoint
2. Test Health Check (Public)
3. Test User Registration & Verification
4. Test User Login & Get JWT Token
5. Test Profile Management
6. Test Social Features (Posts, Friends, Notifications)
7. Test Recipe Management
8. Test AI Features (Recipe Generation, Translation)
9. Test File Upload (S3 + CloudFront)
10. Test Admin Features (nếu có quyền admin)
```

---

### Bước 1: Lấy API Endpoint

**1. Lấy API URL từ CloudFormation Outputs**

```powershell
# Get API endpoint từ Backend Stack
$API_ENDPOINT = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Backend `
  --query 'Stacks[0].Outputs[?OutputKey==`ApiCustomDomain`].OutputValue' `
  --output text

Write-Host "API Endpoint: $API_ENDPOINT"
# Output: https://api-dev.everyonecook.cloud
```

**2. Hoặc lấy từ file outputs.json**

```powershell
# Đọc từ infrastructure/outputs.json
cd D:\Project_AWS\everyonecook\infrastructure
$outputs = Get-Content outputs.json | ConvertFrom-Json
$API_ENDPOINT = $outputs.'EveryoneCook-dev-Backend'.ApiCustomDomain
Write-Host "API Endpoint: $API_ENDPOINT"
```

**3. Setup biến môi trường**

```powershell
# Set API endpoint cho PowerShell session
$API_ENDPOINT = "https://api-dev.everyonecook.cloud"
$HEADERS_JSON = @{"Content-Type"="application/json"}

Write-Host "Environment configured:"
Write-Host "  API Endpoint: $API_ENDPOINT"
```

---

### Bước 2: Test Health Check (Public)

Health check endpoint không cần authentication.

**1. Test với PowerShell**

```powershell
# Test health endpoint
$response = Invoke-RestMethod -Uri "$API_ENDPOINT/health" -Method Get
$response | ConvertTo-Json

# Expected Output:
# {
#   "status": "healthy",
#   "timestamp": "2025-12-09T10:30:00.000Z",
#   "service": "EveryoneCook API",
#   "environment": "dev"
# }
```

**2. Test với curl (nếu có WSL hoặc Git Bash)**

```bash
curl -X GET "$API_ENDPOINT/health" | jq
```

**3. Verify API Router hoạt động**

```powershell
# Check API Router Lambda logs
aws logs tail /aws/lambda/everyonecook-dev-api-router --follow
```

✅ **Expected Result**: Status 200, response JSON có `"status": "healthy"`

---

### Bước 3: Test User Registration

**1. Lấy User Pool Client ID**

```powershell
# Get Cognito User Pool Client ID
$CLIENT_ID = aws cloudformation describe-stacks `
  --stack-name EveryoneCook-dev-Auth `
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolClientId`].OutputValue' `
  --output text

Write-Host "User Pool Client ID: $CLIENT_ID"
```

**2. Register User mới**

```powershell
# Đăng ký user với Cognito (không qua API - trực tiếp với Cognito)
$username = "testuser_$(Get-Random -Maximum 9999)"
$email = "test_$username@example.com"
$password = "TestPassword123!"

aws cognito-idp sign-up `
  --client-id $CLIENT_ID `
  --username $username `
  --password $password `
  --user-attributes `
    Name=email,Value=$email `
    Name=given_name,Value="Test User"

Write-Host "User registered: $username"
Write-Host "Email: $email"
Write-Host "Password: $password"
```

**3. Verify Pre-Signup Trigger (Lambda Cognito Trigger)**

```powershell
# Check logs của Pre-Signup trigger
aws logs tail /aws/lambda/EveryoneCook-dev-PreSignup --since 5m
```

✅ **Expected**: 
- Cognito trả về success message
- Pre-Signup trigger logs không có error

---

### Bước 4: Verify Email & Confirm User

**1. Lấy Confirmation Code**

```powershell
# Trong môi trường dev, có thể lấy code từ email hoặc dùng admin command
# Cách 1: Check email (nếu dùng real email)
# Cách 2: Admin confirm (cho testing)

aws cognito-idp admin-confirm-sign-up `
  --user-pool-id ap-southeast-1_PKoL34PF0 `
  --username $username

Write-Host "User confirmed: $username"
```

**2. Verify Post-Confirmation Trigger**

Post-Confirmation trigger sẽ tự động tạo user profile trong DynamoDB.

```powershell
# Check DynamoDB - User profile được tạo tự động
aws dynamodb get-item `
  --table-name EveryoneCook-dev `
  --key "{\"PK\":{\"S\":\"USER#$username\"},\"SK\":{\"S\":\"PROFILE\"}}"

# Expected: User profile với các fields:
# - userId (Cognito sub ID)
# - email
# - fullName
# - createdAt
# - updatedAt
```

**3. Check Post-Confirmation Lambda logs**

```powershell
aws logs tail /aws/lambda/EveryoneCook-dev-PostConfirmation --since 5m
```

✅ **Expected**: User profile được tạo trong DynamoDB table

---

### Bước 5: Test User Login & Get JWT Token

**1. Login để lấy JWT tokens**

```powershell
# Sign in với Cognito
$authResult = aws cognito-idp initiate-auth `
  --client-id $CLIENT_ID `
  --auth-flow USER_PASSWORD_AUTH `
  --auth-parameters USERNAME=$username,PASSWORD=$password `
  | ConvertFrom-Json

# Extract tokens
$ID_TOKEN = $authResult.AuthenticationResult.IdToken
$ACCESS_TOKEN = $authResult.AuthenticationResult.AccessToken
$REFRESH_TOKEN = $authResult.AuthenticationResult.RefreshToken

Write-Host "Login successful!"
Write-Host "ID Token length: $($ID_TOKEN.Length)"
```

**2. Verify Post-Authentication Trigger**

Post-Authentication trigger update `lastLoginAt` trong DynamoDB.

```powershell
# Check lastLoginAt updated
aws dynamodb get-item `
  --table-name EveryoneCook-dev `
  --key "{\"PK\":{\"S\":\"USER#$username\"},\"SK\":{\"S\":\"PROFILE\"}}" `
  --projection-expression "lastLoginAt"
```

**3. Setup Authorization Header**

```powershell
# Create headers with JWT token
$HEADERS_AUTH = @{
  "Content-Type" = "application/json"
  "Authorization" = "Bearer $ID_TOKEN"
}
```

✅ **Expected**: Login thành công, nhận được JWT tokens

---

### Bước 6: Test Profile Management

**Endpoint**: `/users/me`, `/users/profile`

**1. Get Current User Profile**

```powershell
# GET /users/me
$response = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/me" `
  -Method Get `
  -Headers $HEADERS_AUTH

$response | ConvertTo-Json -Depth 5

# Expected Response:
# {
#   "userId": "...",
#   "username": "testuser_1234",
#   "email": "test_testuser_1234@example.com",
#   "fullName": "Test User",
#   "birthday": null,
#   "gender": null,
#   "country": null,
#   "createdAt": "2025-12-09T10:30:00.000Z",
#   "lastLoginAt": "2025-12-09T11:00:00.000Z"
# }
```

**2. Update Profile (Onboarding)**

```powershell
# PUT /users/profile - Complete onboarding
$profileUpdate = @{
  birthday = "1990-01-01"
  gender = "male"
  country = "Vietnam"
  bio = "Test user for EveryoneCook platform"
} | ConvertTo-Json

$response = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/profile" `
  -Method Put `
  -Headers $HEADERS_AUTH `
  -Body $profileUpdate

$response | ConvertTo-Json
```

**3. Get Privacy Settings**

```powershell
# GET /users/profile/privacy
$privacy = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/profile/privacy" `
  -Method Get `
  -Headers $HEADERS_AUTH

$privacy | ConvertTo-Json
```

**4. Update Privacy Settings**

```powershell
# PUT /users/profile/privacy
$privacyUpdate = @{
  profileVisibility = "public"
  showEmail = $false
  showBirthday = $false
  allowFriendRequests = $true
} | ConvertTo-Json

Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/profile/privacy" `
  -Method Put `
  -Headers $HEADERS_AUTH `
  -Body $privacyUpdate
```

✅ **Expected**: Profile được update thành công trong DynamoDB

---

### Bước 7: Test Social Features - Posts

**Endpoints**: `/posts`, `/posts/{postId}`

**1. Create Post**

```powershell
# POST /posts
$newPost = @{
  content = "My first post on EveryoneCook! Testing the platform 🍳"
  visibility = "public"
  type = "text"
} | ConvertTo-Json

$postResponse = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $newPost

$POST_ID = $postResponse.postId
Write-Host "Created Post ID: $POST_ID"
$postResponse | ConvertTo-Json
```

**2. Get All Posts (Feed)**

```powershell
# GET /posts - Get all posts
$posts = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts?limit=10" `
  -Method Get `
  -Headers $HEADERS_AUTH

Write-Host "Found $($posts.items.Count) posts"
$posts.items | ConvertTo-Json -Depth 3
```

**3. Get Specific Post**

```powershell
# GET /posts/{postId}
$post = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts/$POST_ID" `
  -Method Get `
  -Headers $HEADERS_AUTH

$post | ConvertTo-Json
```

**4. Like Post**

```powershell
# POST /posts/{postId}/like
Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts/$POST_ID/like" `
  -Method Post `
  -Headers $HEADERS_AUTH

Write-Host "Post liked successfully"
```

**5. Add Comment**

```powershell
# POST /posts/{postId}/comments
$comment = @{
  content = "Great post! 👍"
} | ConvertTo-Json

$commentResponse = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts/$POST_ID/comments" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $comment

$COMMENT_ID = $commentResponse.commentId
Write-Host "Comment ID: $COMMENT_ID"
```

**6. Get Post Comments**

```powershell
# GET /posts/{postId}/comments
$comments = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/posts/$POST_ID/comments" `
  -Method Get `
  -Headers $HEADERS_AUTH

$comments | ConvertTo-Json -Depth 3
```

✅ **Expected**: Posts, likes, comments được lưu trong DynamoDB

---

### Bước 8: Test Social Features - Friends

**Endpoints**: `/friends/*`

**1. Search Users**

```powershell
# GET /users/search?q=test
$users = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/search?q=test&limit=10" `
  -Method Get `
  -Headers $HEADERS_AUTH

$users | ConvertTo-Json
```

**2. Send Friend Request** (cần 2 users)

```powershell
# POST /friends/{userId}/request
# Giả sử có USER_ID của user khác
$TARGET_USER_ID = "another-user-id"

Invoke-RestMethod `
  -Uri "$API_ENDPOINT/friends/$TARGET_USER_ID/request" `
  -Method Post `
  -Headers $HEADERS_AUTH

Write-Host "Friend request sent"
```

**3. Get Friend Requests**

```powershell
# GET /friends/requests
$requests = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/friends/requests" `
  -Method Get `
  -Headers $HEADERS_AUTH

$requests | ConvertTo-Json
```

✅ **Expected**: Friend requests được tạo với status "pending"

---

### Bước 9: Test Recipe Management

**Endpoints**: `/recipes`, `/recipes/{recipeId}`

**1. Create Recipe**

```powershell
# POST /recipes
$newRecipe = @{
  title = "Phở Bò Hà Nội"
  description = "Traditional Vietnamese beef noodle soup"
  ingredients = @(
    @{ name = "beef bones"; amount = "2"; unit = "kg" }
    @{ name = "rice noodles"; amount = "500"; unit = "g" }
    @{ name = "ginger"; amount = "1"; unit = "piece" }
    @{ name = "star anise"; amount = "3"; unit = "pieces" }
  )
  instructions = @(
    "Boil beef bones for 2-3 hours to make broth"
    "Add spices (ginger, star anise, cinnamon) and simmer"
    "Prepare rice noodles separately"
    "Serve noodles with broth and garnish with herbs"
  )
  cuisine = "Vietnamese"
  difficulty = "medium"
  prepTime = 30
  cookTime = 180
  servings = 4
} | ConvertTo-Json -Depth 5

$recipeResponse = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/recipes" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $newRecipe

$RECIPE_ID = $recipeResponse.recipeId
Write-Host "Created Recipe ID: $RECIPE_ID"
$recipeResponse | ConvertTo-Json -Depth 5
```

**2. Get User's Recipes**

```powershell
# GET /users/{userId}/recipes
$userId = $authResult.AuthenticationResult.AccessToken | 
  ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(($_.Split('.')[1]))) } | 
  ConvertFrom-Json | 
  Select-Object -ExpandProperty sub

$recipes = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/users/$userId/recipes" `
  -Method Get `
  -Headers $HEADERS_AUTH

$recipes | ConvertTo-Json -Depth 3
```

**3. Search Recipes**

```powershell
# POST /recipes/search
$searchQuery = @{
  query = "phở"
  filters = @{
    cuisine = "Vietnamese"
    difficulty = "medium"
  }
  limit = 10
} | ConvertTo-Json

$searchResults = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/recipes/search" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $searchQuery

$searchResults | ConvertTo-Json -Depth 3
```

✅ **Expected**: Recipes được lưu trong DynamoDB với proper structure

---

### Bước 10: Test AI Features (Bedrock)

**Endpoints**: `/recipes/generate-ai`, `/ai/nutrition`, `/dictionary/{ingredient}`

**1. Generate Recipe with AI**

```powershell
# POST /recipes/generate-ai
$aiRequest = @{
  ingredients = @("chicken", "rice", "vegetables", "fish sauce")
  cuisine = "Vietnamese"
  dietaryRestrictions = @("gluten-free")
  servings = 4
  difficulty = "medium"
} | ConvertTo-Json

Write-Host "Generating recipe with AI... (this takes 5-10 seconds)"
$aiRecipe = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/recipes/generate-ai" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $aiRequest

$aiRecipe | ConvertTo-Json -Depth 5

# Expected: AI-generated recipe with Vietnamese ingredient names
# Uses Amazon Bedrock Claude model
```

**2. Get Nutrition Analysis**

```powershell
# POST /ai/nutrition
$nutritionRequest = @{
  ingredients = @(
    @{ name = "chicken breast"; amount = "200"; unit = "g" }
    @{ name = "rice"; amount = "100"; unit = "g" }
  )
} | ConvertTo-Json

$nutrition = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/ai/nutrition" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $nutritionRequest

$nutrition | ConvertTo-Json
```

**3. Translate Ingredient (Vietnamese Dictionary)**

```powershell
# GET /dictionary/{ingredient}
$translation = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/dictionary/tomato" `
  -Method Get `
  -Headers $HEADERS_AUTH

Write-Host "Translation: $($translation.vietnamese)"
# Expected: { "ingredient": "tomato", "vietnamese": "cà chua" }
```

✅ **Expected**: 
- AI recipe generation mất 5-10 giây
- Response có Vietnamese ingredient names
- Bedrock Lambda được invoke

---

### Bước 11: Test File Upload (S3 + CloudFront)

**Endpoint**: `/upload/presigned-url`

**1. Request Presigned URL**

```powershell
# POST /upload/presigned-url
$uploadRequest = @{
  fileType = "avatar"
  fileName = "test-avatar.jpg"
  contentType = "image/jpeg"
  fileSize = 1024000  # 1MB
} | ConvertTo-Json

$uploadResponse = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/upload/presigned-url" `
  -Method Post `
  -Headers $HEADERS_AUTH `
  -Body $uploadRequest

$PRESIGNED_URL = $uploadResponse.url
$UPLOAD_KEY = $uploadResponse.key

Write-Host "Presigned URL: $PRESIGNED_URL"
Write-Host "Upload Key: $UPLOAD_KEY"
```

**2. Upload File to S3**

```powershell
# Create test image file
$testImage = "D:\test-avatar.jpg"
# (Tạo file test image hoặc dùng file có sẵn)

# Upload file using presigned URL
Invoke-RestMethod `
  -Uri $PRESIGNED_URL `
  -Method Put `
  -InFile $testImage `
  -ContentType "image/jpeg"

Write-Host "File uploaded to S3 successfully"
```

**3. Access via CloudFront CDN**

```powershell
# Get CloudFront distribution domain
$CDN_DOMAIN = "d2shrpzup69rju.cloudfront.net"  # Từ outputs.json

# Access file via CloudFront
$fileUrl = "https://$CDN_DOMAIN/$UPLOAD_KEY"
Invoke-WebRequest -Uri $fileUrl -Method Head

# Check caching headers
# First request: X-Cache: Miss from cloudfront
# Second request: X-Cache: Hit from cloudfront
```

✅ **Expected**: File được upload lên S3 và serve qua CloudFront

---

### Bước 12: Test Admin Features (Requires Admin Role)

**Endpoints**: `/admin/*`

**1. Get System Stats**

```powershell
# GET /admin/stats
$stats = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/admin/stats" `
  -Method Get `
  -Headers $HEADERS_AUTH

$stats | ConvertTo-Json

# Expected (nếu có admin role):
# {
#   "totalUsers": 123,
#   "totalPosts": 456,
#   "totalRecipes": 789,
#   "activeUsers": 45
# }
```

**2. Get All Users**

```powershell
# GET /admin/users
$allUsers = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/admin/users?limit=20" `
  -Method Get `
  -Headers $HEADERS_AUTH

$allUsers | ConvertTo-Json
```

**3. Get Reported Posts**

```powershell
# GET /admin/posts/reported
$reportedPosts = Invoke-RestMethod `
  -Uri "$API_ENDPOINT/admin/posts/reported" `
  -Method Get `
  -Headers $HEADERS_AUTH

$reportedPosts | ConvertTo-Json
```

⚠️ **Note**: Admin endpoints require user có group "Admins" trong Cognito User Pool

---

### Bước 13: Verify Async Processing (SQS Queues)

**1. Check SQS Queues**

```powershell
# List all queues
aws sqs list-queues --queue-name-prefix everyonecook-dev

# Get AI Queue attributes
$AI_QUEUE_URL = "https://sqs.ap-southeast-1.amazonaws.com/616580903213/everyonecook-dev-ai-queue"

aws sqs get-queue-attributes `
  --queue-url $AI_QUEUE_URL `
  --attribute-names ApproximateNumberOfMessages,ApproximateNumberOfMessagesNotVisible
```

**2. Monitor Worker Lambdas**

```powershell
# Check Image Processing Worker logs
aws logs tail /aws/lambda/everyonecook-dev-image-worker --follow

# Check AI Worker logs (for recipe generation)
aws logs tail /aws/lambda/everyonecook-dev-ai-worker --follow
```

✅ **Expected**: Messages được process bởi worker Lambdas

---

### Testing Checklist

Sử dụng checklist này để track progress:

#### Public Endpoints
- [ ] `GET /health` - Health check responds
- [ ] `GET /status` - Status check responds

#### Authentication & User Management
- [ ] User registration với Cognito works
- [ ] Email verification/confirmation works  
- [ ] User login successful, nhận JWT tokens
- [ ] `GET /users/me` - Get current user profile
- [ ] `PUT /users/profile` - Update profile
- [ ] `GET /users/profile/privacy` - Get privacy settings
- [ ] `PUT /users/profile/privacy` - Update privacy settings

#### Social Features
- [ ] `POST /posts` - Create post
- [ ] `GET /posts` - Get posts feed
- [ ] `GET /posts/{postId}` - Get specific post
- [ ] `POST /posts/{postId}/like` - Like post
- [ ] `POST /posts/{postId}/comments` - Add comment
- [ ] `GET /posts/{postId}/comments` - Get comments
- [ ] `POST /friends/{userId}/request` - Send friend request
- [ ] `GET /friends/requests` - Get friend requests
- [ ] `GET /notifications` - Get notifications

#### Recipe Management
- [ ] `POST /recipes` - Create recipe
- [ ] `GET /recipes` - Get all recipes
- [ ] `GET /recipes/{recipeId}` - Get specific recipe
- [ ] `POST /recipes/search` - Search recipes

#### AI Features
- [ ] `POST /recipes/generate-ai` - AI recipe generation (Bedrock)
- [ ] `POST /ai/nutrition` - Nutrition analysis
- [ ] `GET /dictionary/{ingredient}` - Ingredient translation

#### File Upload
- [ ] `POST /upload/presigned-url` - Get presigned URL
- [ ] Upload file to S3 using presigned URL
- [ ] Access file via CloudFront CDN
- [ ] Verify CloudFront caching (Miss → Hit)

#### Admin Features (if admin)
- [ ] `GET /admin/stats` - Get system stats
- [ ] `GET /admin/users` - List all users
- [ ] `GET /admin/posts/reported` - Get reported content

#### Infrastructure
- [ ] All Lambda functions execute without errors
- [ ] SQS queues process messages correctly
- [ ] CloudWatch logs show proper execution
- [ ] DynamoDB items created correctly
- [ ] WAF rules protecting API Gateway

---

### Performance Benchmarks

**Expected Response Times:**

| Endpoint | Expected Time | Notes |
|----------|--------------|-------|
| `GET /health` | < 50ms | Direct response |
| `POST /auth/login` | < 200ms | Cognito validation |
| `GET /users/me` | < 100ms | DynamoDB single query |
| `POST /posts` | < 300ms | DynamoDB write + notifications |
| `GET /posts` | < 500ms | DynamoDB query with pagination |
| `POST /recipes/generate-ai` | 5-10 seconds | Bedrock AI generation |
| `POST /upload/presigned-url` | < 100ms | S3 presigned URL generation |
| `POST /recipes/search` | < 200ms | DynamoDB GSI query |

---

### Troubleshooting Common Issues

#### 1. 401 Unauthorized

```powershell
# Verify JWT token not expired
$ID_TOKEN = "your-token-here"
$parts = $ID_TOKEN.Split('.')
$payload = [System.Text.Encoding]::UTF8.GetString(
  [System.Convert]::FromBase64String($parts[1])
) | ConvertFrom-Json

$exp = [DateTimeOffset]::FromUnixTimeSeconds($payload.exp).DateTime
Write-Host "Token expires at: $exp"

# If expired, login again
```

#### 2. 403 Forbidden (WAF Block)

```powershell
# Check WAF logs
aws wafv2 get-web-acl `
  --name EveryoneCook-API-WAF-dev `
  --scope REGIONAL `
  --region ap-southeast-1

# Check if IP blocked
```

#### 3. 500 Internal Server Error

```powershell
# Check Lambda function logs
aws logs tail /aws/lambda/everyonecook-dev-api-router --follow
aws logs tail /aws/lambda/everyonecook-dev-social --follow

# Check DynamoDB table
aws dynamodb describe-table --table-name EveryoneCook-dev
```

#### 4. Slow AI Recipe Generation

```powershell
# Check Bedrock model availability
aws bedrock list-foundation-models --region us-east-1

# Check AI Queue
aws sqs get-queue-attributes `
  --queue-url $AI_QUEUE_URL `
  --attribute-names All
```

---

### Next Steps

Sau khi test thành công tất cả endpoints:

1. ✅ **Verify Infrastructure**: Tất cả AWS resources hoạt động đúng
2. 📊 **Monitor CloudWatch**: Check metrics và logs
3. 🔒 **Security Review**: Verify WAF rules và Cognito auth
4. 📝 **Document API**: Update API documentation nếu cần
5. 🚀 **Deploy Frontend**: Proceed to frontend deployment
6. 📦 **Version Control**: Commit code và push to GitLab (next step)

Proceed to: [5.09 - Push to GitLab](../5.09-push-gitlab/)

**2. Confirm User**

```bash
# Confirm user with code
aws cognito-idp confirm-sign-up \
  --client-id $CLIENT_ID \
  --username testuser \
  --confirmation-code 123456
```

**3. Verify Post-Confirmation Trigger**

```bash
# Check if user profile was created in DynamoDB
aws dynamodb get-item \
  --table-name EveryoneCook-dev \
  --key '{"PK":{"S":"USER#testuser"},"SK":{"S":"PROFILE"}}'

# Should return user profile with:
# - PK: USER#testuser
# - SK: PROFILE
# - userId: cognito-sub-id
# - email: test@example.com
# - fullName: Test User
# - birthday: null
# - gender: null
# - country: null
```

#### Step 5: Test User Login

**1. Sign In**

```bash
# Sign in to get tokens
TOKENS=$(aws cognito-idp initiate-auth \
  --client-id $CLIENT_ID \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=testuser,PASSWORD=TestPassword123!)

# Extract tokens
ACCESS_TOKEN=$(echo $TOKENS | jq -r '.AuthenticationResult.AccessToken')
ID_TOKEN=$(echo $TOKENS | jq -r '.AuthenticationResult.IdToken')
REFRESH_TOKEN=$(echo $TOKENS | jq -r '.AuthenticationResult.RefreshToken')

echo "ID Token: $ID_TOKEN"
```

**2. Verify Post-Authentication Trigger**

```bash
# Check if lastLoginAt was updated
aws dynamodb get-item \
  --table-name EveryoneCook-dev \
  --key '{"PK":{"S":"USER#testuser"},"SK":{"S":"PROFILE"}}' \
  --projection-expression "lastLoginAt"
```

#### Step 6: Test Profile Management

**1. Get Profile**

```bash
# Get user profile
curl -X GET \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/auth/profile

# Expected: User profile data
```

**2. Update Profile**

```bash
# Update profile (onboarding)
curl -X PUT \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "birthday": "1990-01-01",
    "gender": "male",
    "country": "US"
  }' \
  $API_ENDPOINT/auth/profile

# Expected: Updated profile
```

**3. Verify Update in DynamoDB**

```bash
# Check updated profile
aws dynamodb get-item \
  --table-name EveryoneCook-dev \
  --key '{"PK":{"S":"USER#testuser"},"SK":{"S":"PROFILE"}}'

# Should show birthday, gender, country updated
```

#### Step 7: Test Social Features

**1. Create Post**

```bash
# Create a post
POST_RESPONSE=$(curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "My first post on EveryoneCook!",
    "visibility": "public"
  }' \
  $API_ENDPOINT/social/posts)

POST_ID=$(echo $POST_RESPONSE | jq -r '.postId')
echo "Created post: $POST_ID"
```

**2. Get Posts Feed**

```bash
# Get posts
curl -X GET \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/social/posts

# Expected: Array of posts including the one just created
```

**3. Like Post**

```bash
# Like the post
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/social/posts/$POST_ID/like

# Expected: Success message
```

**4. Comment on Post**

```bash
# Add comment
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Great post!"}' \
  $API_ENDPOINT/social/posts/$POST_ID/comment

# Expected: Comment created
```

#### Step 8: Test Recipe Features

**1. Create Recipe**

```bash
# Create a recipe
RECIPE_RESPONSE=$(curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Pho Bo (Vietnamese Beef Noodle Soup)",
    "description": "Traditional Vietnamese beef noodle soup",
    "ingredients": [
      {"name": "beef bones", "amount": "2", "unit": "kg"},
      {"name": "rice noodles", "amount": "500", "unit": "g"},
      {"name": "ginger", "amount": "1", "unit": "piece"}
    ],
    "instructions": [
      "Boil beef bones for 2 hours",
      "Add spices and simmer",
      "Prepare noodles and serve"
    ],
    "cuisine": "Vietnamese",
    "difficulty": "medium",
    "prepTime": 30,
    "cookTime": 120
  }' \
  $API_ENDPOINT/recipes)

RECIPE_ID=$(echo $RECIPE_RESPONSE | jq -r '.recipeId')
echo "Created recipe: $RECIPE_ID"
```

**2. Get Recipes**

```bash
# Get all recipes
curl -X GET \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/recipes

# Expected: Array of recipes
```

**3. Get Recipe by ID**

```bash
# Get specific recipe
curl -X GET \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/recipes/$RECIPE_ID

# Expected: Recipe details
```

#### Step 9: Test AI Features

**1. Generate Recipe with AI**

```bash
# Generate recipe using Bedrock
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ingredients": ["chicken", "rice", "vegetables"],
    "cuisine": "Vietnamese",
    "dietaryRestrictions": ["gluten-free"],
    "servings": 4
  }' \
  $API_ENDPOINT/ai/generate-recipe

# Expected: AI-generated recipe (takes 5-10 seconds)
# Response includes Vietnamese ingredient names
```

**2. Translate Ingredient**

```bash
# Translate ingredient to Vietnamese
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ingredient": "tomato",
    "targetLanguage": "vi"
  }' \
  $API_ENDPOINT/ai/translate

# Expected: {"translation": "cà chua", "confidence": 0.99}
```

#### Step 10: Test File Upload

**1. Request Pre-signed URL**

```bash
# Get pre-signed URL for avatar upload
UPLOAD_RESPONSE=$(curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fileType": "avatar",
    "fileName": "avatar.jpg",
    "contentType": "image/jpeg",
    "fileSize": 1024000
  }' \
  $API_ENDPOINT/upload/presigned-url)

PRESIGNED_URL=$(echo $UPLOAD_RESPONSE | jq -r '.url')
UPLOAD_KEY=$(echo $UPLOAD_RESPONSE | jq -r '.key')

echo "Pre-signed URL: $PRESIGNED_URL"
echo "Upload Key: $UPLOAD_KEY"
```

**2. Upload File to S3**

```bash
# Create test image
echo "Test image content" > test-avatar.jpg

# Upload using pre-signed URL
curl -X PUT \
  -H "Content-Type: image/jpeg" \
  --upload-file test-avatar.jpg \
  "$PRESIGNED_URL"

# Expected: 200 OK
```

**3. Mark Upload Complete**

```bash
# Notify backend that upload is complete
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"key\":\"$UPLOAD_KEY\"}" \
  $API_ENDPOINT/upload/complete

# Expected: Success message
```

**4. Access via CloudFront**

```bash
# Access file via CloudFront CDN
curl -I https://cdn.everyonecook.cloud/$UPLOAD_KEY

# First request: X-Cache: Miss from cloudfront
# Second request: X-Cache: Hit from cloudfront
```

#### Step 11: Test Search (OpenSearch)

**If OpenSearch is enabled:**

```bash
# Search recipes with Vietnamese query
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "phở bò",
    "filters": {
      "cuisine": "Vietnamese",
      "difficulty": "medium"
    },
    "limit": 10
  }' \
  $API_ENDPOINT/ai/search

# Expected: Array of matching recipes
# Vietnamese analyzer handles: "phở bò" = "pho bo" = "beef noodle soup"
```

#### Step 12: Test Admin Features

**1. List Users (Admin Only)**

```bash
# Get all users (requires admin role)
curl -X GET \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/admin/users

# Expected: Array of users or 403 Forbidden if not admin
```

#### Step 13: Verify Async Processing

**1. Check SQS Queue Processing**

```bash
# Send message to SearchIndex queue
QUEUE_URL=$(aws sqs list-queues \
  --queue-name-prefix EveryoneCook-dev-SearchIndexQueue \
  | jq -r '.QueueUrls[0]')

aws sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body "{
    \"eventName\": \"INSERT\",
    \"tableName\": \"recipes\",
    \"keys\": {\"PK\": \"USER#testuser\", \"SK\": \"RECIPE#$RECIPE_ID\"},
    \"newImage\": {
      \"title\": \"Pho Bo\",
      \"ingredients\": [\"beef\", \"noodles\"],
      \"cuisine\": \"Vietnamese\"
    }
  }"

# Check worker logs
aws logs tail /aws/lambda/EveryoneCook-dev-SearchSyncWorker --follow
```

#### Testing Checklist

- [ ] Health check responds
- [ ] User registration works
- [ ] Email verification works
- [ ] User login successful
- [ ] Profile CRUD operations work
- [ ] Posts can be created
- [ ] Posts can be liked
- [ ] Comments can be added
- [ ] Recipes can be created
- [ ] AI recipe generation works
- [ ] Ingredient translation works
- [ ] File upload to S3 works
- [ ] CloudFront serves files
- [ ] Search works (if OpenSearch enabled)
- [ ] SQS queues process messages
- [ ] All Lambda functions execute without errors

#### Performance Benchmarks

**Expected Response Times:**
- Health check: < 50ms
- User login: < 200ms
- Get profile: < 100ms
- Create post: < 300ms
- Get posts feed: < 500ms
- AI recipe generation: 5-10 seconds
- File upload (pre-signed URL): < 100ms
- Search query: < 200ms

#### Next Steps

Once all tests pass, proceed to [Push to GitLab](../5.9-push-gitlab/) to version control your code and set up CI/CD.
