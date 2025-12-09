---
title: "Deploy lên Amplify"
date: 2025-12-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Tổng quan

Bước cuối cùng là deploy frontend Next.js 15 lên AWS Amplify với tích hợp GitLab để tự động deploy khi có code mới.

#### Bước 1: Chuẩn bị Frontend Code

**1. Kiểm tra Frontend Structure**

```bash
cd frontend

# Kiểm tra package.json
cat package.json

# Phải hiển thị Next.js 15
```

**2. Test Frontend Locally**

```bash
# Cài đặt dependencies
npm install

# Chạy development server
npm run dev

# Mở http://localhost:3000
```

**3. Build Frontend**

```bash
# Build cho production
npm run build

# Test production build
npm start
```

#### Bước 2: Tạo Amplify App

**1. Vào AWS Amplify Console**

1. Mở AWS Console
2. Tìm kiếm "Amplify"
3. Click "Get started" hoặc "New app"

**2. Kết nối với GitLab**

1. Chọn "Host web app"
2. Chọn "GitLab"
3. Click "Connect branch"
4. Ủy quyền AWS Amplify truy cập GitLab

![Amplify GitLab Connection](/images/5-Workshop/5.10-deploy-amplify/amplify-gitlab-connect.png)
*Screenshot: Amplify hiển thị kết nối GitLab*

**3. Chọn Repository**

1. Chọn repository: `everyonecook`
2. Chọn branch: `main` (hoặc `dev` cho development)
3. Click "Next"

#### Bước 3: Cấu hình Build Settings

**1. Tên App**

```
App name: everyonecook-frontend
```

**2. Build Settings**

Amplify tự động phát hiện Next.js, nhưng hãy xác minh:

```yaml
version: 1
applications:
  - frontend:
      phases:
        preBuild:
          commands:
            - export HUSKY=0
            - npm install --legacy-peer-deps --ignore-scripts
        build:
          commands:
            - echo "=== Creating .env.production from Amplify env vars ==="
            - rm -f .env.production
            - env | grep -e NEXT_PUBLIC_ > .env.production || true
            - echo "=== .env.production content ==="
            - cat .env.production
            - echo "=== Building frontend ==="
            - npm run build
      artifacts:
        baseDirectory: .next
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
          - .next/cache/**/*
    appRoot: frontend
```


**3. Advanced Settings**

Thêm environment variables:
```
NEXT_PUBLIC_API_URL=https://api.everyonecook.cloud
NEXT_PUBLIC_CDN_URL=https://cdn.everyonecook.cloud
NEXT_PUBLIC_USER_POOL_ID=us-east-1_ABC123
NEXT_PUBLIC_USER_POOL_CLIENT_ID=abc123def456
NEXT_PUBLIC_REGION=us-east-1
```

![Amplify Environment Variables](/images/5-Workshop/5.10-deploy-amplify/amplify-env-vars.png)
*Screenshot: Cấu hình environment variables của Amplify*

#### Bước 4: Deploy Frontend

**1. Bắt đầu Deployment**

Click "Save and deploy"

Amplify sẽ:
1. Clone repository từ GitLab
2. Cài đặt dependencies
3. Build Next.js app
4. Deploy lên CDN
5. Cung cấp domain

**2. Theo dõi Deployment**

Xem tiến trình deployment:
- Provision
- Build
- Deploy
- Verify


**3. Đợi Hoàn thành**

Deployment mất 5-10 phút.

![Amplify Deployment Success](/images/5-Workshop/5.10-deploy-amplify/amplify-success.png)
*Screenshot: Amplify hiển thị deployment thành công*

#### Bước 5: Cấu hình Custom Domain

**1. Thêm Custom Domain**

1. Vào Amplify app → Domain management
2. Click "Add domain"
3. Nhập domain: `everyonecook.cloud`
4. Amplify sẽ tự động cấu hình:
   - Root domain: `everyonecook.cloud`
   - WWW subdomain: `www.everyonecook.cloud`

**2. Cấu hình DNS**

Amplify tự động tạo DNS records trong Route 53:
```
everyonecook.cloud → A record → Amplify
www.everyonecook.cloud → CNAME → Amplify
```

**3. SSL Certificate**

Amplify tự động cung cấp SSL certificate qua ACM.

**4. Đợi DNS Propagation**

Mất 5-15 phút.

![Amplify Custom Domain](/images/5-Workshop/5.10-deploy-amplify/amplify-custom-domain.png)
*Screenshot: Amplify hiển thị custom domain đã cấu hình*

#### Bước 6: Xác minh Deployment

**1. Truy cập Frontend**

```bash
# Qua Amplify domain
curl -I https://main.d1234567890.amplifyapp.com

# Qua custom domain
curl -I https://everyonecook.cloud

# Phải trả về 200 OK
```

**2. Test Chức năng Frontend**

1. Mở https://everyonecook.cloud trong trình duyệt
2. Test đăng ký user
3. Test đăng nhập
4. Test tạo posts
5. Test tạo recipes
6. Test tính năng AI

![Frontend Live](/images/5-Workshop/5.10-deploy-amplify/domain.png)
*Screenshot: Trình duyệt hiển thị EveryoneCook frontend đang chạy*


#### Best Practices

1. **Sử dụng Environment Variables**: Không bao giờ hardcode API URLs
2. **Bật Auto-Deploy**: Tự động deployments khi push
3. **Nhiều Environments**: Tách biệt dev và prod
4. **Monitor Performance**: Sử dụng Amplify metrics
5. **Thiết lập Notifications**: Nhận cảnh báo khi build fails
6. **Sử dụng Custom Domain**: Giao diện chuyên nghiệp
7. **Bật HTTPS**: Luôn sử dụng SSL
8. **Cấu hình Headers**: Security headers để bảo vệ

#### Bước tiếp theo

Chúc mừng! Ứng dụng của bạn giờ đã được deploy đầy đủ:
- ✅ Infrastructure trên AWS
- ✅ Backend APIs đang chạy
- ✅ Frontend trên Amplify
- ✅ Code trên GitLab
- ✅ CI/CD đã cấu hình

Tiếp tục đến [Cleanup](../5.11-cleanup/) khi bạn hoàn thành testing, hoặc bắt đầu sử dụng ứng dụng!

