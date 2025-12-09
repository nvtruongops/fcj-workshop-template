---
title: "Push lên GitLab"
date: 2025-12-01
weight: 9
chapter: false
pre: " <b> 5.09. </b> "
---

#### Tổng quan

Sau khi test thành công, bạn sẽ push code lên GitLab để version control và chuẩn bị cho CI/CD.

#### Bước 1: Khởi tạo Git Repository

**1. Kiểm tra Git Status**

```bash
# Di chuyển đến project root
cd /path/to/everyonecook-dev

# Kiểm tra Git đã được khởi tạo chưa
git status

# Nếu chưa khởi tạo:
git init
```

**2. Cấu hình Git**

```bash
# Đặt tên và email của bạn
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Xác minh cấu hình
git config --list
```

#### Bước 2: Tạo .gitignore

**1. Tạo file .gitignore**

```bash
# Tạo .gitignore
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
package-lock.json
yarn.lock

# Build outputs
dist/
build/
*.js.map
*.d.ts.map
cdk.out/
.next/

# Environment variables
.env
.env.local
.env.*.local

# AWS
*.pem
*.key
cloudfront-private-key.pem

# Deployment packages
*.zip
deployment/

# Logs
logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Test coverage
coverage/
.nyc_output/

# Temporary files
tmp/
temp/
*.tmp

# CDK
cdk.context.json
outputs.json

# TypeScript
*.tsbuildinfo
EOF
```

#### Bước 3: Stage và Commit Files

**1. Thêm Files vào Staging**

```bash
# Thêm tất cả files
git add .

# Kiểm tra những gì sẽ được commit
git status
```

**2. Tạo Initial Commit**

```bash
# Commit với message
git commit -m "Initial commit: EveryoneCook infrastructure and backend"

# Xác minh commit
git log --oneline
```

![Initial Commit](/images/5-Workshop/5.9-push-gitlab/initial-commit.png)
*Screenshot: Terminal hiển thị initial commit*

#### Bước 4: Tạo GitLab Repository

**1. Đăng nhập vào GitLab**

Truy cập https://gitlab.com/ và đăng nhập

**2. Tạo New Project**

1. Click "New project"
2. Chọn "Create blank project"
3. Tên project: `everyonecook`
4. Visibility: Private (khuyến nghị)
5. Initialize with README: No (chúng ta đã có code)
6. Click "Create project"

![GitLab Project Created](/images/5-Workshop/5.9-push-gitlab/gitlab-project.png)
*Screenshot: GitLab hiển thị project mới được tạo*


#### Tóm tắt Git Workflow

```
1. Tạo feature branch từ dev
   git checkout -b feature/new-feature

2. Thực hiện thay đổi và commit
   git add .
   git commit -m "Description"

3. Push lên GitLab
   git push -u origin feature/new-feature

4. Tạo merge request trên GitLab
   feature/new-feature → dev

5. Review, test, và merge

6. Deploy lên dev environment
   (manual trigger trong GitLab CI/CD)

7. Test trong dev environment

8. Merge dev → main cho production

9. Deploy lên production
   (manual trigger trong GitLab CI/CD)

10. Tag release
    git tag -a v1.0.0 -m "Release"
```

#### Best Practices

**Commit Messages:**
```
feat: Thêm user authentication
fix: Sửa lỗi login
docs: Cập nhật README
style: Format code
refactor: Refactor user service
test: Thêm unit tests
chore: Cập nhật dependencies
```

**Đặt tên Branch:**
```
feature/add-notifications
bugfix/fix-login-error
hotfix/critical-security-patch
release/v1.0.0
```

#### Khắc phục sự cố

**Vấn đề: Authentication failed**

```bash
# Sử dụng personal access token thay vì password
# Tạo token: GitLab → Settings → Access Tokens
# Sử dụng token làm password khi push
```

**Vấn đề: Large files rejected**

```bash
# Kiểm tra kích thước file
git ls-files -z | xargs -0 du -h | sort -h

# Xóa large files khỏi history
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD
```

**Vấn đề: Merge conflicts**

```bash
# Cập nhật branch của bạn
git checkout feature/your-feature
git fetch origin
git merge origin/dev

# Giải quyết conflicts trong files
# Sau đó commit
git add .
git commit -m "Resolve merge conflicts"
git push
```

#### Bước tiếp theo

Sau khi push code lên GitLab, tiếp tục với [Deploy lên Amplify](../5.10-deploy-amplify/) để triển khai ứng dụng frontend.
