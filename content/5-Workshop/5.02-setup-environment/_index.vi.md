---
title: "Thiết lập Môi trường"
date: 2025-12-01
weight: 2
chapter: false
pre: " <b> 5.02. </b> "
---

#### Tổng quan

Trong bước này, bạn sẽ cài đặt tất cả các công cụ cần thiết để phát triển và deploy ứng dụng EveryoneCook.

#### Các công cụ cần thiết

**1. Node.js 20.x**

```bash
# Tải từ https://nodejs.org/
# Hoặc sử dụng nvm (khuyến nghị)
nvm install 20
nvm use 20

# Kiểm tra cài đặt
node --version  # Phải là v20.x
npm --version
```

![Cài đặt Node.js](/images/5-Workshop/5.2-setup-environment/nodejs-version.png)
*Screenshot: Terminal hiển thị Node.js 20.x đã cài đặt*

**2. AWS CLI v2**

```bash
# Windows: Tải từ https://aws.amazon.com/cli/
# macOS: brew install awscli
# Linux: 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Kiểm tra cài đặt
aws --version
```

![Cài đặt AWS CLI](/images/5-Workshop/5.2-setup-environment/aws-cli-version.png)
*Screenshot: Terminal hiển thị AWS CLI v2 đã cài đặt*

**3. AWS CDK CLI**

```bash
# Cài đặt CDK global
npm install -g aws-cdk

# Kiểm tra cài đặt
cdk --version
```

![Cài đặt CDK CLI](/images/5-Workshop/5.2-setup-environment/cdk-version.png)
*Screenshot: Terminal hiển thị CDK CLI đã cài đặt*

**4. Git**

```bash
# Tải từ https://git-scm.com/
# Hoặc sử dụng package manager

# Kiểm tra cài đặt
git --version
```
![Cài đặt Git](/images/5-Workshop/5.2-setup-environment/git-version.png)

**5. Code Editor**

Khuyến nghị: Visual Studio Code với các extensions:
- AWS Toolkit
- GitLab Workflow
- ESLint
- Prettier

#### Thiết lập AWS Account

**1. Tạo AWS Account**

Nếu chưa có AWS account:
1. Truy cập https://aws.amazon.com/
2. Click "Create an AWS Account"
3. Làm theo quy trình đăng ký
4. Thêm phương thức thanh toán

**2. Tạo IAM User**

Để bảo mật, không sử dụng root account

1. Vào IAM Console → Users → Create user
2. Tạo user và lưu credentials

![IAM User đã tạo](/images/5-Workshop/5.2-setup-environment/iam-user-created.png)
*Screenshot: IAM console hiển thị user đã tạo với AdministratorAccess*

**3. Cấu hình AWS CLI**

```bash
# Cấu hình AWS credentials
aws configure

# Nhập:
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json
```

![Cấu hình AWS CLI](/images/5-Workshop/5.2-setup-environment/aws-cli-configure.png)
*Screenshot: Terminal hiển thị aws configure hoàn tất*

**4. Xác minh AWS Access**

```bash
# Kiểm tra AWS credentials
aws sts get-caller-identity

# Sẽ trả về account ID và user ARN của bạn
```
![AWS Access đã xác minh](/images/5-Workshop/5.2-setup-environment/aws-sts-identity.png)

#### Thiết lập Domain (Tùy chọn)

Nếu muốn sử dụng custom domain:

**1. Đăng ký Domain**

Mua domain name trên hpanel.hostinger
Route 53 tạo dns record tới hostinger 
Cho workshop này, chúng tôi sử dụng: `everyonecook.cloud`

![AWS DNS](/images/5-Workshop/5.2-setup-environment/hostinger.png)
![AWS DNS](/images/5-Workshop/5.2-setup-environment/DNS_record.png)

**2. Ghi chú Domain Registrar**

Bạn sẽ cần truy cập domain registrar để cập nhật nameservers sau này.

#### Thiết lập GitLab

**Tạo GitLab Repo**

![GitLab Access Token](/images/5-Workshop/5.2-setup-environment/gitlab-repo.png)
*Screenshot: GitLab hiển thị personal access token đã tạo*

**3. Cấu hình Git**

```bash
# Đặt tên và email của bạn
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Xác minh cấu hình
git config --list
```

#### Thiết lập Project

**1. Clone hoặc Tạo Project**

Tùy chọn A: Clone project hiện có
```bash
git clone https://gitlab.com/your-username/everyonecook.git
cd everyonecook
```

Tùy chọn B: Tạo project mới
```bash
mkdir everyonecook
cd everyonecook
git init
```

**2. Cài đặt Dependencies**

```bash
# Cài đặt tất cả dependencies
npm install

# Cài đặt:
# - Infrastructure dependencies (CDK)
# - Backend dependencies (Lambda modules)
# - Shared dependencies
```

**3. Copy Environment Variables**

```bash
# Copy file env mẫu
cp .env.example .env

# Chỉnh sửa .env với các giá trị của bạn
# Các biến chính:
# - AWS_REGION=us-east-1
# - AWS_ACCOUNT_ID=your-account-id
# - DOMAIN_NAME=everyonecook.cloud
# - GITLAB_TOKEN=your-gitlab-token
```

#### Xác minh

Kiểm tra mọi thứ đã cài đặt đúng:

```bash
# Kiểm tra Node.js
node --version  # v20.x

# Kiểm tra npm
npm --version   # 10.x

# Kiểm tra AWS CLI
aws --version   # aws-cli/2.x

# Kiểm tra CDK
cdk --version   # 2.x

# Kiểm tra Git
git --version   # 2.x

# Kiểm tra AWS credentials
aws sts get-caller-identity

# Kiểm tra project dependencies
npm list --depth=0
```

#### Khắc phục sự cố

**Vấn đề: Node.js version không khớp**
```bash
# Sử dụng nvm để chuyển phiên bản
nvm install 20
nvm use 20
```

**Vấn đề: Không tìm thấy AWS CLI**
- Khởi động lại terminal sau khi cài đặt
- Kiểm tra biến môi trường PATH

**Vấn đề: Không tìm thấy lệnh CDK**
```bash
# Cài đặt lại CDK globally
npm uninstall -g aws-cdk
npm install -g aws-cdk
```

**Vấn đề: AWS credentials không hợp lệ**
```bash
# Cấu hình lại AWS CLI
aws configure
# Nhập credentials đúng
```

#### Bước tiếp theo

Sau khi thiết lập môi trường xong, tiếp tục với [CDK Bootstrap](../5.3-cdk-bootstrap/) để chuẩn bị AWS account cho CDK deployments.
