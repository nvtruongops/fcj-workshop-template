---
title: "5.4.4 Auth Stack"
weight: 4
---
---
# Auth Stack - Xác Thực & Quản Lý Người Dùng

## Tổng Quan

Auth Stack là **tầng xác thực Phase 3** của dự án EveryoneCook. Nó quản lý xác thực người dùng, đăng ký và bảo mật tài khoản bằng AWS Cognito với các Lambda trigger tùy chỉnh để nâng cao trải nghiệm người dùng.

**Thứ Tự Triển Khai**: Stack này **BẮT BUỘC** phải được triển khai sau Core Stack và **trước** Backend Stack.

### Trách Nhiệm Chính

- Tạo Cognito User Pool với các thiết lập bảo mật cấp production
- Cấu hình Cognito User Pool Client cho ứng dụng web
- Thiết lập 5 Lambda triggers cho các quy trình xác thực tùy chỉnh
- Quản lý đăng ký người dùng, xác minh email và quy trình đăng nhập
- Xử lý tạo hồ sơ người dùng trong DynamoDB (qua PostConfirmation trigger)

### Những Gì Stack Này Bao Gồm

**Cognito User Pool**:
- Đăng nhập: Username hoặc email
- Chính sách mật khẩu: Tối thiểu 12 ký tự (8 cho dev), chữ hoa, chữ thường, chữ số, ký hiệu
- Yêu cầu xác minh email
- MFA: Tắt (chỉ email + mật khẩu)
- Device tracking: Bật (không có MFA challenge)
- Thuộc tính chuẩn: username, email, given_name (fullName)
- Thuộc tính tùy chỉnh: account_status, country

**Cognito User Pool Client**:
- Auth flows: USER_PASSWORD_AUTH, USER_SRP_AUTH
- OAuth flows: Authorization code grant (social login trong tương lai)
- Token validity: Access/ID (1h), Refresh (30 ngày)
- Callback URLs: Theo từng environment

**Lambda Triggers** (5 triggers):
1. **PreSignUp**: Dọn dẹp người dùng chưa xác minh (hết hạn 24h)
2. **CustomMessage**: Tùy chỉnh email templates
3. **PostConfirmation**: Tạo hồ sơ người dùng trong DynamoDB
4. **PreAuthentication**: Kiểm tra nếu người dùng bị cấm/tạm ngưng
5. **PostAuthentication**: Cập nhật timestamp đăng nhập cuối cùng

---

## Kiến Trúc

```
┌─────────────────────────────────────────────────────────────────┐
│                    Auth Stack (Phase 3)                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Cognito User Pool                                        │  │
│  │  ├─ Đăng nhập: Username hoặc Email                      │  │
│  │  ├─ Mật khẩu: Tối thiểu 12 ký tự, chính sách mạnh      │  │
│  │  ├─ Xác minh Email: Bắt buộc                            │  │
│  │  ├─ MFA: Tắt (chỉ email + mật khẩu)                     │  │
│  │  ├─ Device Tracking: Bật (không có MFA)                 │  │
│  │  └─ Thuộc tính tùy chỉnh: account_status, country       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Lambda Triggers (Quy trình tùy chỉnh)                    │  │
│  │                                                           │  │
│  │  1️⃣ PreSignUp                                            │  │
│  │     ├─ Kiểm tra username/email tồn tại                  │  │
│  │     ├─ Xóa người dùng chưa xác minh hết hạn (>24h)     │  │
│  │     └─ Cho phép đăng ký mới                             │  │
│  │                                                           │  │
│  │  2️⃣ CustomMessage                                         │  │
│  │     ├─ Tùy chỉnh template xác minh email                │  │
│  │     ├─ Tùy chỉnh template đặt lại mật khẩu              │  │
│  │     └─ Thêm styling và branding                         │  │
│  │                                                           │  │
│  │  3️⃣ PostConfirmation                                      │  │
│  │     ├─ Tạo DynamoDB entities:                            │  │
│  │     │  ├─ Core Profile (PK=USER#{userId}, SK=PROFILE)  │  │
│  │     │  ├─ Privacy Settings (SK=PRIVACY_SETTINGS)       │  │
│  │     │  └─ AI Preferences (SK=AI_PREFERENCES)           │  │
│  │     └─ Khởi tạo dữ liệu người dùng                      │  │
│  │                                                           │  │
│  │  4️⃣ PreAuthentication                                     │  │
│  │     ├─ Kiểm tra trạng thái tài khoản                    │  │
│  │     ├─ Từ chối nếu bị cấm/tạm ngưng                     │  │
│  │     └─ Cho phép đăng nhập nếu active                    │  │
│  │                                                           │  │
│  │  5️⃣ PostAuthentication                                    │  │
│  │     ├─ Cập nhật timestamp lastLoginAt                   │  │
│  │     └─ Theo dõi hoạt động người dùng                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          │                                      │
│                          ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Cognito User Pool Client                                │  │
│  │  ├─ Client Type: Web (không có secret)                  │  │
│  │  ├─ Auth Flows: Password, SRP                          │  │
│  │  ├─ OAuth: Authorization code grant                     │  │
│  │  ├─ Tokens: 1h access, 1h ID, 30d refresh             │  │
│  │  ├─ Callback: https://{env}.everyonecook.cloud        │  │
│  │  └─ Security: Token revocation, user enum protection   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                          │
                          │ Exports
                          ▼
                   Backend Stack
        (API Gateway Cognito Authorizer)
```

---

## Cấu Hình Stack

### Cấu Trúc File

```
infrastructure/lib/stacks/
└── auth-stack.ts                           # Auth Stack (865 dòng)

services/auth-module/triggers/
├── pre-signup.ts                          # PreSignUp trigger
├── custom-message.ts                      # CustomMessage trigger
├── post-confirmation.ts                   # PostConfirmation trigger
├── pre-authentication.ts                  # PreAuthentication trigger
└── post-authentication.ts                 # PostAuthentication trigger
```

### Điểm Nổi Bật Trong Triển Khai Code

**File**: `infrastructure/lib/stacks/auth-stack.ts`

#### 1. Tạo Cognito User Pool

```typescript
/**
 * Tạo Cognito User Pool với bảo mật cấp production
 */
private createUserPool(): cdk.aws_cognito.UserPool {
  const cognitoConfig = this.config.cognito;
  
  const userPool = new cdk.aws_cognito.UserPool(this, 'UserPool', {
    userPoolName: `EveryoneCook-${this.config.environment}`,
    
    // Cấu hình đăng nhập
    signInAliases: {
      username: true,
      email: true,
    },
    
    // Cho phép tự đăng ký
    selfSignUpEnabled: true,
    
    // Thuộc tính chuẩn
    standardAttributes: {
      email: {
        required: true,
        mutable: false,  // Email không thể thay đổi
      },
      givenName: {
        required: true,  // fullName được lưu trong given_name
        mutable: true,
      },
      birthdate: { required: false, mutable: true },
      gender: { required: false, mutable: true },
    },
    
    // Thuộc tính tùy chỉnh
    customAttributes: {
      account_status: new cdk.aws_cognito.StringAttribute({
        mutable: true,
        minLen: 1,
        maxLen: 20,
      }),
      country: new cdk.aws_cognito.StringAttribute({
        mutable: true,
        minLen: 2,
        maxLen: 2,  // ISO 3166-1 alpha-2
      }),
    },
    
    // Chính sách mật khẩu
    passwordPolicy: {
      minLength: 12,  // 8 cho dev
      requireLowercase: true,
      requireUppercase: true,
      requireDigits: true,
      requireSymbols: true,
      tempPasswordValidity: cdk.Duration.days(7),
    },
    
    // Khôi phục tài khoản
    accountRecovery: cdk.aws_cognito.AccountRecovery.EMAIL_ONLY,
    
    // Cấu hình email (Cognito mặc định)
    email: cdk.aws_cognito.UserPoolEmail.withCognito(),
    
    // Tự động xác minh email
    autoVerify: { email: true },
    
    // MFA: Tắt
    mfa: cdk.aws_cognito.Mfa.OFF,
    
    // Device tracking (không có MFA challenge)
    deviceTracking: {
      challengeRequiredOnNewDevice: false,
      deviceOnlyRememberedOnUserPrompt: true,
    },
    
    // Email templates
    userVerification: {
      emailSubject: '🍳 Xác minh tài khoản Everyone Cook của bạn',
      emailBody: 'Xin chào {username}, mã xác minh của bạn là: {####}',
      emailStyle: cdk.aws_cognito.VerificationEmailStyle.CODE,
    },
    
    // Deletion protection cho production
    deletionProtection: this.config.environment === 'prod',
  });
  
  return userPool;
}
```

#### 2. Lambda Triggers

```typescript
/**
 * Tạo PostConfirmation Lambda Trigger
 * 
 * Tạo 3 DynamoDB entities sau khi xác minh email:
 * 1. Core Profile (PK=USER#{userId}, SK=PROFILE)
 * 2. Privacy Settings (SK=PRIVACY_SETTINGS)
 * 3. AI Preferences (SK=AI_PREFERENCES)
 */
private createPostConfirmationTrigger(
  dynamoTable: cdk.aws_dynamodb.ITable
): cdk.aws_lambda.Function {
  const trigger = new cdk.aws_lambda.Function(this, 'PostConfirmationTrigger', {
    functionName: `EveryoneCook-${this.config.environment}-PostConfirmation`,
    runtime: cdk.aws_lambda.Runtime.NODEJS_20_X,
    handler: 'post-confirmation.handler',
    code: cdk.aws_lambda.Code.fromAsset(
      path.join(__dirname, '../../../services/auth-module/triggers/dist')
    ),
    memorySize: 512,
    timeout: cdk.Duration.seconds(30),
    environment: {
      DYNAMODB_TABLE_NAME: dynamoTable.tableName,
      ENVIRONMENT: this.config.environment,
    },
  });
  
  // Cấp quyền ghi DynamoDB
  dynamoTable.grantReadWriteData(trigger);
  
  return trigger;
}

/**
 * Tạo PreSignUp Lambda Trigger
 * 
 * Xử lý dọn dẹp người dùng chưa xác minh:
 * - Nếu user tồn tại và UNCONFIRMED >24h → xóa và cho phép đăng ký mới
 * - Nếu user tồn tại và UNCONFIRMED <24h → từ chối với thông báo "chờ 24h"
 * - Nếu user không tồn tại → cho phép đăng ký
 */
private createPreSignUpTrigger(): cdk.aws_lambda.Function {
  const trigger = new cdk.aws_lambda.Function(this, 'PreSignUpTrigger', {
    functionName: `EveryoneCook-${this.config.environment}-PreSignUp`,
    runtime: cdk.aws_lambda.Runtime.NODEJS_20_X,
    handler: 'pre-signup.handler',
    code: cdk.aws_lambda.Code.fromAsset(
      path.join(__dirname, '../../../services/auth-module/triggers/dist')
    ),
    memorySize: 256,
    timeout: cdk.Duration.seconds(10),
  });
  
  // Cấp quyền Cognito
  trigger.addToRolePolicy(
    new cdk.aws_iam.PolicyStatement({
      effect: cdk.aws_iam.Effect.ALLOW,
      actions: ['cognito-idp:ListUsers', 'cognito-idp:AdminDeleteUser'],
      resources: [`arn:aws:cognito-idp:${this.region}:${this.account}:userpool/*`],
    })
  );
  
  return trigger;
}
```

#### 3. User Pool Client

```typescript
/**
 * Tạo Cognito User Pool Client cho ứng dụng web
 */
private createUserPoolClient(): cdk.aws_cognito.UserPoolClient {
  const callbackUrls = this.getCallbackUrls();
  const logoutUrls = this.getLogoutUrls();
  
  const userPoolClient = new cdk.aws_cognito.UserPoolClient(
    this, 'UserPoolClient', {
      userPoolClientName: `EveryoneCook-Web-Client-${this.config.environment}`,
      userPool: this.userPool,
      
      // Auth flows
      authFlows: {
        userPassword: true,  // USER_PASSWORD_AUTH
        userSrp: true,       // USER_SRP_AUTH (Secure Remote Password)
        custom: false,
        adminUserPassword: false,
      },
      
      // Cấu hình OAuth (social login trong tương lai)
      oAuth: {
        flows: {
          authorizationCodeGrant: true,
          implicitCodeGrant: false,
          clientCredentials: false,
        },
        scopes: [
          cdk.aws_cognito.OAuthScope.EMAIL,
          cdk.aws_cognito.OAuthScope.OPENID,
          cdk.aws_cognito.OAuthScope.PROFILE,
        ],
        callbackUrls: callbackUrls,
        logoutUrls: logoutUrls,
      },
      
      // Token validity
      accessTokenValidity: cdk.Duration.hours(1),
      idTokenValidity: cdk.Duration.hours(1),
      refreshTokenValidity: cdk.Duration.days(30),
      
      // Read attributes
      readAttributes: new cdk.aws_cognito.ClientAttributes()
        .withStandardAttributes({
          email: true,
          emailVerified: true,
          givenName: true,
        })
        .withCustomAttributes('account_status', 'country'),
      
      // Thiết lập bảo mật
      preventUserExistenceErrors: true,  // Ngăn enumeration attacks
      enableTokenRevocation: true,       // Cho phép thu hồi token
      generateSecret: false,             // Không có secret cho web apps
    }
  );
  
  return userPoolClient;
}
```

---

## Chi Tiết Cấu Hình Chính

### 1. Quy Trình Đăng Ký Người Dùng

**Quy Trình Đăng Ký**:
```
1. Người dùng đăng ký → PreSignUp trigger
   ├─ Kiểm tra nếu username/email tồn tại
   ├─ Nếu UNCONFIRMED >24h: Xóa user cũ
   ├─ Nếu UNCONFIRMED <24h: Từ chối với "chờ 24h"
   └─ Cho phép đăng ký

2. Người dùng nhận email xác minh → CustomMessage trigger
   ├─ Tùy chỉnh email template
   └─ Gửi mã xác minh

3. Người dùng xác minh email → PostConfirmation trigger
   ├─ Tạo DynamoDB entities:
   │  ├─ Core Profile (username, email, fullName, v.v.)
   │  ├─ Privacy Settings (mặc định: private)
   │  └─ AI Preferences (thiết lập mặc định)
   └─ Tài khoản người dùng sẵn sàng

4. Người dùng đăng nhập → PreAuthentication trigger
   ├─ Kiểm tra account_status
   ├─ Nếu banned/suspended: Từ chối đăng nhập
   └─ Cho phép đăng nhập

5. Đăng nhập thành công → PostAuthentication trigger
   └─ Cập nhật timestamp lastLoginAt
```

### 2. Chính Sách Mật Khẩu

**Environments**:

| Environment | Độ dài tối thiểu | Yêu cầu |
|-------------|-----------------|---------|
| **Dev** | 8 ký tự | Chữ hoa, chữ thường, chữ số, ký hiệu |
| **Staging** | 12 ký tự | Chữ hoa, chữ thường, chữ số, ký hiệu |
| **Prod** | 12 ký tự | Chữ hoa, chữ thường, chữ số, ký hiệu |

**Ví dụ Mật Khẩu Hợp Lệ**:
- `MyP@ssw0rd123` (12 ký tự)
- `Str0ng!Pass` (11 ký tự, không hợp lệ cho prod/staging)

### 3. Token Validity

| Loại Token | Validity | Mục đích |
|-----------|----------|----------|
| **Access Token** | 1 giờ | API authorization |
| **ID Token** | 1 giờ | User identity claims |
| **Refresh Token** | 30 ngày | Gia hạn access/ID tokens |

**Quy Trình Refresh Token**:
```
Access token hết hạn (1h) → Dùng refresh token → Nhận access/ID tokens mới
Refresh token hết hạn (30d) → Người dùng phải đăng nhập lại
```

### 4. Chi Tiết Lambda Trigger

#### PreSignUp Trigger

**Mục đích**: Ngăn lỗi "username đã được sử dụng" cho người dùng chưa xác minh

**Logic**:
```typescript
if (userExists && userStatus === 'UNCONFIRMED') {
  const hoursSinceCreation = (now - userCreationDate) / (1000 * 60 * 60);
  
  if (hoursSinceCreation > 24) {
    // Xóa user chưa xác minh đã hết hạn
    await deleteUser(username);
    return allowSignUp();
  } else {
    // User vẫn còn thời gian để xác minh
    return rejectSignUp(`Vui lòng chờ ${24 - hoursSinceCreation}h để đăng ký lại`);
  }
} else {
  return allowSignUp();
}
```

#### PostConfirmation Trigger

**DynamoDB Entities Được Tạo**:

```javascript
// 1. Core Profile
{
  PK: "USER#{userId}",
  SK: "PROFILE",
  username: "john_doe",
  email: "john@example.com",
  fullName: "John Doe",
  account_status: "active",
  createdAt: "2025-01-01T00:00:00Z",
  // ... các field khác
}

// 2. Privacy Settings
{
  PK: "USER#{userId}",
  SK: "PRIVACY_SETTINGS",
  profileVisibility: "private",
  showEmail: false,
  allowMessages: "friends",
  // ... các settings khác
}

// 3. AI Preferences
{
  PK: "USER#{userId}",
  SK: "AI_PREFERENCES",
  aiEnabled: true,
  preferredLanguage: "en",
  dietaryRestrictions: [],
  // ... các preferences khác
}
```

---

## Stack Outputs

Sau khi triển khai, stack export các giá trị sau:

| Output Name | Value | Được sử dụng bởi |
|------------|-------|------------------|
| `UserPoolId` | `ap-southeast-1_XXXXXXXXX` | Backend Stack (Authorizer) |
| `UserPoolArn` | `arn:aws:cognito-idp:...` | Lambda IAM policies |
| `UserPoolClientId` | `1234567890abcdef` | Frontend (Amplify config) |
| `CustomMessageFunctionArn` | `arn:aws:lambda:...` | Monitoring |
| `PostConfirmationFunctionArn` | `arn:aws:lambda:...` | Monitoring |
| `PreAuthenticationFunctionArn` | `arn:aws:lambda:...` | Monitoring |
| `PostAuthenticationFunctionArn` | `arn:aws:lambda:...` | Monitoring |

---

## Các Bước Triển Khai

### Bước 1: Build Lambda Triggers

Trước khi triển khai, biên dịch Lambda triggers sang JavaScript:

```powershell
cd D:\Project_AWS\everyonecook\services\auth-module\triggers

# Cài đặt dependencies
npm install

# Build TypeScript sang JavaScript
npm run build
```

Kết quả mong đợi:
```
> auth-module-triggers@1.0.0 build
> tsc

Compiled successfully to dist/
```

### Bước 2: Xác Minh Điều Kiện Tiên Quyết

-  Core Stack đã triển khai thành công
-  DynamoDB table tồn tại
-  Lambda triggers đã build vào thư mục `dist/`

### Bước 3: Triển Khai Auth Stack

Di chuyển đến thư mục infrastructure:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Triển khai Auth Stack đến **ap-southeast-1**:

```powershell
# Triển khai Auth Stack
npx cdk deploy EveryoneCook-dev-Auth --context environment=dev
```

Kết quả mong đợi:

```
✨  Synthesis time: 7.23s

EveryoneCook-dev-Auth: deploying...
[████████████████████████████████████████] (9/9)

EveryoneCook-dev-Auth: creating CloudFormation changeset...

   EveryoneCook-dev-Auth

✨  Deployment time: 180.45s

Outputs:
EveryoneCook-dev-Auth.UserPoolId = ap-southeast-1_a1B2c3D4e
EveryoneCook-dev-Auth.UserPoolClientId = 1a2b3c4d5e6f7g8h9i0j
EveryoneCook-dev-Auth.UserPoolArn = arn:aws:cognito-idp:ap-southeast-1:616580903213:userpool/...
EveryoneCook-dev-Auth.PostConfirmationFunctionArn = arn:aws:lambda:ap-southeast-1:...
EveryoneCook-dev-Auth.PreAuthenticationFunctionArn = arn:aws:lambda:ap-southeast-1:...

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:616580903213:stack/EveryoneCook-dev-Auth/...
```

### Bước 4: Xác Minh trong AWS Console

#### Điều hướng đến Cognito User Pool

1. Mở AWS Console → region **ap-southeast-1**
2. Đi đến **Amazon Cognito** > **User pools**
3. Tìm `EveryoneCook-dev`

![Cognito User Pool Overview](/images/5-Workshop/5.4-configure-stacks/cognito-user-pool.png)
*Cognito User Pool hiển thị tùy chọn đăng nhập (username/email), MFA disabled, password policy, và deletion protection*

**Xác minh**:
-  Đăng nhập: Username hoặc Email
-  MFA: Disabled
-  Password policy: Đã cấu hình
-  Email verification: Required

#### Xác Minh Cấu Hình User Pool

Click vào User Pool để xem chi tiết:

![Cognito User Pool Details](/images/5-Workshop/5.4-configure-stacks/cognito-pool-details.png)
*Cấu hình User Pool hiển thị authentication settings, attributes, password policy, và security features*

**Kiểm tra**:
- **Sign-in experience**: Username và Email enabled
- **User attributes**: email, given_name (bắt buộc), birthdate, gender (tùy chọn)
- **Custom attributes**: account_status, country
- **Password policy**: Tối thiểu 12 ký tự, yêu cầu chữ hoa, chữ thường, chữ số, ký hiệu
- **MFA**: Off
- **Device tracking**: Enabled

#### Xác Minh Lambda Triggers

Đi đến **User pool properties** > **Lambda triggers**:

![Cognito Lambda Triggers](/images/5-Workshop/5.4-configure-stacks/cognito-triggers.png)
*Lambda triggers được cấu hình cho Pre sign-up, Custom message, Post confirmation, Pre authentication, và Post authentication*

**Xác minh 5 triggers**:
-  Pre sign-up: `EveryoneCook-dev-PreSignUp`
-  Custom message: `EveryoneCook-dev-CustomMessage`
-  Post confirmation: `EveryoneCook-dev-PostConfirmation`
-  Pre authentication: `EveryoneCook-dev-PreAuthentication`
-  Post authentication: `EveryoneCook-dev-PostAuthentication`

#### Xác Minh User Pool Client

Đi đến **App integration** > **App clients**:

![Cognito User Pool Client](/images/5-Workshop/5.4-configure-stacks/cognito-client.png)
*User Pool Client hiển thị auth flows, OAuth settings, token validity, callback URLs, và security settings*

**Xác minh**:
-  Client type: Public (không có secret)
-  Auth flows: USER_PASSWORD_AUTH, USER_SRP_AUTH
-  OAuth flows: Authorization code grant
-  Callback URLs: Theo environment
-  Token validity: 1h access, 1h ID, 30d refresh

#### Điều hướng đến Lambda Functions

Đi đến **Lambda** > **Functions**, tìm Auth triggers:

![Lambda Triggers List](/images/5-Workshop/5.4-configure-stacks/cognito-triggers.png)
*Lambda functions hiển thị tất cả 5 Cognito triggers với runtime Node.js 20.x, memory 256-512 MB, và timeout 10-30s*

**Xác minh**:
-  Tất cả 5 Lambda functions đã tạo
-  Runtime: Node.js 20.x
-  Environment variables đã cấu hình
-  CloudWatch log groups đã tạo

#### Kiểm Tra Lambda Permissions

Click vào Lambda function → **Configuration** > **Permissions**:

![Lambda IAM Permissions](/images/5-Workshop/5.4-configure-stacks/lambda-permissions.png)
*Lambda execution role hiển thị permissions cho DynamoDB (PostConfirmation), Cognito (PreSignUp), và CloudWatch Logs*

**Quyền mong đợi**:
- **PostConfirmation**: DynamoDB read/write
- **PreAuthentication**: DynamoDB read
- **PostAuthentication**: DynamoDB read/write
- **PreSignUp**: Cognito ListUsers, AdminDeleteUser
- **Tất cả triggers**: CloudWatch Logs write

---

## Chi Tiết Chi Phí

### Chi Phí Hàng Tháng (Dev Environment)

| Tài nguyên | Cấu hình | Chi phí hàng tháng | Ghi chú |
|------------|----------|-------------------|---------|
| **Cognito User Pool** | <50 MAU | $0 | 50K MAU đầu miễn phí |
| **Lambda Triggers** | 5 functions, low invocations | $0-1 | Free tier bao phủ hầu hết |
| **CloudWatch Logs** | 7-day retention, 5 log groups | $0.50 | ~1GB logs |
| **Tổng (Ước tính)** | | **~$0.50-1.50/tháng** | Rất thấp cho dev |

### Ghi Chú Chi Phí

- **Cognito**: 50,000 MAU đầu miễn phí, sau đó $0.0055/MAU
- **Lambda**: 1M requests/tháng miễn phí, sau đó $0.20 per 1M
- **CloudWatch Logs**: $0.50/GB ingested, $0.03/GB stored
- **Không có phí MFA**: MFA tắt tiết kiệm $0.05/MAU

**Ước Tính Production** (1000 MAU):
- Cognito: 1000 MAU × $0.0055 = **$5.50/tháng**
- Lambda triggers: ~5000 invocations/tháng = **$0 (free tier)**
- CloudWatch Logs: **$1-2/tháng**
- **Tổng**: **~$7-8/tháng**

---

## Cross-Stack Dependencies

### Import từ Các Stack Trước

**Từ Core Stack**:
```typescript
dynamoTable: cdk.Fn.importValue('EveryoneCook-dev-DynamoDBTableName')
```

### Export Được Sử Dụng Bởi Các Stack Khác

**Backend Stack** import:
- User Pool ID (cho Cognito Authorizer)
- User Pool ARN (cho API Gateway)
- User Pool Client ID (cho frontend config)

### Luồng Dependency

```
Core Stack → DynamoDB Table
    │
    ▼
Auth Stack (tạo Cognito + Lambda triggers)
    │
    ├─► User Pool ID → Backend Stack (API Gateway Authorizer)
    ├─► User Pool Client ID → Frontend (Amplify config)
    └─► Lambda triggers → Quy trình quản lý người dùng
```

---

## Danh Sách Kiểm Tra Xác Thực

Trước khi tiến hành triển khai Backend Stack:

- [ ] Auth Stack đã triển khai thành công đến ap-southeast-1
- [ ] Cognito User Pool tồn tại với các thiết lập chính xác
- [ ] 5 Lambda triggers đã cấu hình và gắn kết
- [ ] User Pool Client đã tạo với OAuth settings
- [ ] Lambda functions có đúng IAM permissions
- [ ] CloudWatch log groups đã tạo (7-day retention)
- [ ] Stack outputs đã export thành công
- [ ] Lambda trigger code đã build vào thư mục `dist/`

---

## Kiểm Tra

### Kiểm Tra Quy Trình Đăng Ký Người Dùng

1. **Kiểm tra sign-up với Cognito console**:
   
   Đi đến **Cognito** > **User pools** > **EveryoneCook-dev** > **Users** > **Create user**

   Tạo test user:
   ```
   Username: testuser01
   Email: your-email@example.com
   Full Name: Test User
   Temporary Password: TempP@ss123
   ```

2. **Xác minh email đã gửi**:
   
   Kiểm tra email của bạn để nhận mã xác minh.

3. **Kiểm tra Lambda logs**:
   
   ```powershell
   # Xem PostConfirmation logs
   aws logs tail /aws/lambda/EveryoneCook-dev-PostConfirmation --follow --region ap-southeast-1
   ```

4. **Xác minh DynamoDB entries**:
   
   ```powershell
   # Truy vấn user profile
   aws dynamodb query `
     --table-name EveryoneCook-dev-v2 `
     --key-condition-expression "PK = :pk" `
     --expression-attribute-values '{":pk":{"S":"USER#testuser01"}}' `
     --region ap-southeast-1
   ```

   Mong đợi: 3 items (PROFILE, PRIVACY_SETTINGS, AI_PREFERENCES)

### Kiểm Tra Quy Trình Xác Thực

1. **Đăng nhập với test user**:
   
   Sử dụng AWS CLI để xác thực:
   ```powershell
   aws cognito-idp initiate-auth `
     --auth-flow USER_PASSWORD_AUTH `
     --client-id <USER_POOL_CLIENT_ID> `
     --auth-parameters USERNAME=testuser01,PASSWORD=<password> `
     --region ap-southeast-1
   ```

2. **Kiểm tra PreAuthentication logs**:
   
   ```powershell
   aws logs tail /aws/lambda/EveryoneCook-dev-PreAuthentication --follow --region ap-southeast-1
   ```

3. **Kiểm tra PostAuthentication logs**:
   
   ```powershell
   aws logs tail /aws/lambda/EveryoneCook-dev-PostAuthentication --follow --region ap-southeast-1
   ```

### Kiểm Tra PreSignUp Cleanup

1. **Tạo unverified user**:
   
   Đăng ký user nhưng không xác minh email.

2. **Chờ 24 giờ** (hoặc sửa trigger code thành 1 phút để kiểm tra)

3. **Thử đăng ký lại với cùng username**:
   
   PreSignUp trigger nên xóa user cũ và cho phép đăng ký mới.

---


## Các Bước Tiếp Theo

Sau khi triển khai thành công Auth Stack:

➡️ **[5.4.5 Backend Stack](../5.4.5-Backend-Stack/)** - Tạo API Gateway, Lambda functions, và SQS queues

Backend Stack sẽ:
- Tạo API Gateway REST API với Cognito Authorizer
- Tạo 5 Lambda functions (auth, social, recipe, AI, admin)
- Tạo 6 SQS queues cho async processing
- Tạo 6 worker Lambda functions
- Import User Pool ID từ Auth Stack
- Cấu hình API Gateway custom domain

---

## Tài Liệu Tham Khảo

- **Mã nguồn**: `infrastructure/lib/stacks/auth-stack.ts`
- **Lambda Triggers**: `services/auth-module/triggers/`
- **Base Stack**: `infrastructure/lib/base-stack.ts`
- **Environment Config**: `infrastructure/config/environment.ts`
- **Tài liệu AWS**:
  - [Cognito User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)
  - [Cognito Lambda Triggers](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html)
  - [Cognito User Pool Client](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-client-apps.html)
  - [Password Policies](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-policies.html)
