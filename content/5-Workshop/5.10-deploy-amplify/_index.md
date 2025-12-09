---
title: "Deploy to Amplify"
date: 2025-12-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Overview

Bước cuối cùng là deploy frontend Next.js 15 lên AWS Amplify với tích hợp GitLab để tự động deploy khi có code mới.

#### Step 1: Prepare Frontend Code

**1. Check Frontend Structure**

```bash
cd frontend

# Check package.json
cat package.json

# Should show Next.js 15
```

**2. Test Frontend Locally**

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Open http://localhost:3000
```

**3. Build Frontend**

```bash
# Build for production
npm run build

# Test production build
npm start
```

#### Step 2: Create Amplify App

**1. Go to AWS Amplify Console**

1. Open AWS Console
2. Search for "Amplify"
3. Click "Get started" or "New app"

**2. Connect to GitLab**

1. Choose "Host web app"
2. Select "GitLab"
3. Click "Connect branch"
4. Authorize AWS Amplify to access GitLab

![Amplify GitLab Connection](/images/5-Workshop/5.10-deploy-amplify/amplify-gitlab-connect.png)
*Screenshot: Amplify showing GitLab connection*

**3. Select Repository**

1. Choose repository: `everyonecook`
2. Choose branch: `main` (or `dev` for development)
3. Click "Next"

#### Step 3: Configure Build Settings

**1. App Name**

```
App name: everyonecook-frontend
```

**2. Build Settings**

Amplify auto-detects Next.js, but verify:

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

Add environment variables:
```
NEXT_PUBLIC_API_URL=https://api.everyonecook.cloud
NEXT_PUBLIC_CDN_URL=https://cdn.everyonecook.cloud
NEXT_PUBLIC_USER_POOL_ID=us-east-1_ABC123
NEXT_PUBLIC_USER_POOL_CLIENT_ID=abc123def456
NEXT_PUBLIC_REGION=us-east-1
```

![Amplify Environment Variables](/images/5-Workshop/5.10-deploy-amplify/amplify-env-vars.png)
*Screenshot: Amplify environment variables configuration*

#### Step 4: Deploy Frontend

**1. Start Deployment**

Click "Save and deploy"

Amplify will:
1. Clone repository from GitLab
2. Install dependencies
3. Build Next.js app
4. Deploy to CDN
5. Provision domain

**2. Monitor Deployment**

Watch deployment progress:
- Provision
- Build
- Deploy
- Verify


**3. Wait for Completion**

Deployment takes 5-10 minutes.

![Amplify Deployment Success](/images/5-Workshop/5.10-deploy-amplify/amplify-success.png)
*Screenshot: Amplify showing successful deployment*

#### Step 5: Configure Custom Domain

**1. Add Custom Domain**

1. Go to Amplify app → Domain management
2. Click "Add domain"
3. Enter domain: `everyonecook.cloud`
4. Amplify will auto-configure:
   - Root domain: `everyonecook.cloud`
   - WWW subdomain: `www.everyonecook.cloud`

**2. DNS Configuration**

Amplify automatically creates DNS records in Route 53:
```
everyonecook.cloud → A record → Amplify
www.everyonecook.cloud → CNAME → Amplify
```

**3. SSL Certificate**

Amplify automatically provisions SSL certificate via ACM.

**4. Wait for DNS Propagation**

Takes 5-15 minutes.

![Amplify Custom Domain](/images/5-Workshop/5.10-deploy-amplify/amplify-custom-domain.png)
*Screenshot: Amplify showing custom domain configured*

#### Step 6: Verify Deployment

**1. Access Frontend**

```bash
# Via Amplify domain
curl -I https://main.d1234567890.amplifyapp.com

# Via custom domain
curl -I https://everyonecook.cloud

# Should return 200 OK
```

**2. Test Frontend Features**

1. Open https://everyonecook.cloud in browser
2. Test user registration
3. Test login
4. Test creating posts
5. Test creating recipes
6. Test AI features

![Frontend Live](/images/5-Workshop/5.10-deploy-amplify/domain.png)
*Screenshot: Browser showing EveryoneCook frontend live*


#### Best Practices

1. **Use Environment Variables**: Never hardcode API URLs
2. **Enable Auto-Deploy**: Automatic deployments on push
3. **Multiple Environments**: Separate dev and prod
4. **Monitor Performance**: Use Amplify metrics
5. **Set Up Notifications**: Get alerted on build failures
6. **Use Custom Domain**: Professional appearance
7. **Enable HTTPS**: Always use SSL
8. **Configure Headers**: Security headers for protection

#### Next Steps

Congratulations! Your application is now fully deployed:
- ✅ Infrastructure on AWS
- ✅ Backend APIs running
- ✅ Frontend on Amplify
- ✅ Code on GitLab
- ✅ CI/CD configured

Proceed to [Cleanup](../5.11-cleanup/) when you're done testing, or start using your application!
