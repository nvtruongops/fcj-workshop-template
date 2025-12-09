---
title: "CDK Bootstrap"
date: 2025-12-01
weight: 3
chapter: false
pre: " <b> 5.03. </b> "
---

#### Tổng quan

AWS CDK Bootstrap chuẩn bị AWS account của bạn để triển khai các ứng dụng CDK bằng cách tạo các tài nguyên hạ tầng cần thiết. Đây là quy trình thiết lập một lần cho mỗi tổ hợp AWS account/region để thiết lập nền tảng cho tất cả các triển khai CDK.

#### CDK Bootstrap tạo gì

Quá trình bootstrap cung cấp các tài nguyên AWS sau trong account của bạn:

**1. S3 Bucket (CDK Assets Bucket)**
- Lưu trữ Lambda function deployment packages
- Lưu trữ CloudFormation templates
- Lưu trữ file assets được tham chiếu bởi CDK stacks
- Mẫu đặt tên: `cdk-hnb659fds-assets-{ACCOUNT-ID}-{REGION}`
- Versioning được bật để hỗ trợ rollback

**2. CloudFormation Stack (CDKToolkit)**
- Stack hạ tầng chính chứa tất cả bootstrap resources
- Tên stack: `CDKToolkit`
- Quản lý vòng đời của bootstrap resources

**3. IAM Roles**
- `cdk-hnb659fds-cfn-exec-role`: CloudFormation execution role để deploy stacks
- `cdk-hnb659fds-deploy-role`: Role được CDK CLI sử dụng trong quá trình deployment
- `cdk-hnb659fds-file-publishing-role`: Role để upload assets lên S3
- `cdk-hnb659fds-lookup-role`: Role để đọc environment context

**4. SSM Parameters**
- `/cdk-bootstrap/hnb659fds/version`: Lưu trữ số phiên bản bootstrap
- Sử dụng để kiểm tra tính tương thích

#### Tổng quan Infrastructure Stacks

Sau khi bootstrap, dự án của chúng ta sẽ triển khai nhiều CDK stacks được định nghĩa trong `D:\Project_AWS\everyonecook\infrastructure\lib\stacks`:

- **auth-stack.ts**: Hạ tầng Authentication và authorization (Cognito)
- **backend-stack.ts**: Core backend services và Lambda functions
- **certificate-stack.ts**: Quản lý SSL/TLS certificates
- **core-stack.ts**: Core infrastructure resources
- **dns-stack.ts**: Cấu hình Route 53 DNS
- **observability-stack.ts**: Monitoring, logging, và observability

Các stacks này phụ thuộc vào hạ tầng bootstrap để triển khai thành công.

---

### Hướng dẫn Thực hành: Bootstrap AWS Account của bạn

#### Yêu cầu trước

Trước khi bắt đầu thực hành này, đảm bảo bạn có:

-  AWS CLI đã cài đặt và cấu hình (xem phần 5.1)
-  AWS CDK CLI đã cài đặt (xem phần 5.2)
-  AWS credentials hợp lệ đã cấu hình
-  IAM user với quyền `AdministratorAccess` hoặc tương đương
-  Quyền truy cập Terminal/PowerShell vào thư mục dự án


### Bước 3: Xác minh trên AWS Console
Xác nhận tài nguyên trong AWS Management Console.

**3.1. Xác minh CloudFormation Stack**

1. Mở [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home?region=us-east-1)
2. Tìm stack tên `CDKToolkit`
3. Status phải là `CREATE_COMPLETE`
4. Click vào tab **Resources** để xem các tài nguyên đã tạo

** Yêu cầu Screenshot:**
![CloudFormation CDKToolkit Stack](/images/5-Workshop/5.2-setup-environment/cloudformation-cdktoolkit.png)


**3.2. Xác minh S3 Bucket**

1. Mở [S3 Console](https://s3.console.aws.amazon.com/s3/buckets?region=us-east-1)
2. Tìm bucket: `cdk-hnb659fds-assets-{ACCOUNT-ID}-us-east-1`
3. Xác minh thuộc tính bucket:
   - Versioning: Enabled

** Yêu cầu Screenshot:**
![S3 Bucket Console](/images/5-Workshop/5.2-setup-environment/s3-bucket-console.png)


**3.3. Xác minh IAM Roles**

1. Mở [IAM Roles Console](https://console.aws.amazon.com/iam/home#/roles)
2. Tìm kiếm: `cdk-hnb659fds`
3. Xác minh các roles sau tồn tại:
   - `cdk-hnb659fds-cfn-exec-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-deploy-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-file-publishing-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-lookup-role-{ACCOUNT-ID}-us-east-1`

![IAM Roles Console](/images/5-Workshop/5.2-setup-environment/iam-roles-console.png)

---

### Hiểu về Bootstrap Resources

#### Chi tiết S3 Bucket

CDK assets bucket đóng vai trò là kho lưu trữ trung tâm cho tất cả deployment artifacts:

- **Mục đích**: Lưu trữ CloudFormation templates, Lambda deployment packages, và static assets
- **Vòng đời**: Tồn tại qua các deployments, cho phép khả năng rollback
- **Bảo mật**: Mã hóa at rest, bucket policies hạn chế truy cập cho các roles được ủy quyền
- **Đặt tên**: Đặt tên xác định dựa trên account ID và region

#### Chi tiết IAM Roles

**1. CloudFormation Execution Role (`cfn-exec-role`)**
- Được CloudFormation sử dụng để create/update/delete stack resources
- Có quyền rộng để quản lý AWS resources
- Trust relationship với CloudFormation service

**2. Deploy Role (`deploy-role`)**
- Được CDK CLI sử dụng trong các thao tác `cdk deploy`
- Có thể assume CloudFormation execution role
- Có quyền upload assets và khởi tạo deployments

**3. File Publishing Role (`file-publishing-role`)**
- Upload file assets lên S3 bucket

**4. Image Publishing Role (`image-publishing-role`)**
- Publish Docker images lên ECR

**5. Lookup Role (`lookup-role`)**
- Đọc environment context (VPCs, subnets, etc.)
- Quyền read-only cho resource lookups
- Sử dụng trong quá trình synthesis cho context queries

---

### Phân tích Chi phí

#### Chi phí Bootstrap Resources

**Thiết lập Một lần:**
- Tạo CloudFormation stack: **Miễn phí**
- Tạo IAM roles: **Miễn phí**
- SSM parameters: **Miễn phí**

**Chi phí Hàng tháng Liên tục:**

| Resource | Sử dụng | Chi phí (Ước tính) |
|----------|---------|------------------|
| S3 Storage | < 1 GB (thông thường) | $0.023/GB = ~$0.02/tháng |
| S3 Requests | Tối thiểu (GET/PUT) | ~$0.01/tháng |
| IAM Roles | Không tính phí | $0.00 |

**Tổng Chi phí Ước tính:** ~$0.03/tháng (không đáng kể)

**Lưu ý:** Đối với serverless applications, chi phí vẫn tối thiểu vì chúng ta chỉ lưu trữ Lambda deployment packages trong S3.

---

### Checklist Hoàn thành Thực hành

Đảm bảo bạn đã hoàn thành tất cả các bước và chụp screenshots yêu cầu:

-  Bước 1: Thực thi lệnh bootstrap và thành công - **[IMAGE HERE: bootstrap-success.png]**
-  Bước 2: Kết quả xác minh CLI - **[IMAGE HERE: bootstrap-verification-cli.png]**
-  Bước 3.1: CloudFormation console - **[IMAGE HERE: cloudformation-cdktoolkit.png]**
-  Bước 3.2: S3 bucket console - **[IMAGE HERE: s3-bucket-console.png]**
-  Bước 3.3: IAM roles console - **[IMAGE HERE: iam-roles-console.png]**
1.  Xác minh trạng thái bootstrap AWS account
2.  Lấy AWS account ID của bạn
3.  Thực thi lệnh CDK bootstrap
4.  Tạo CDKToolkit CloudFormation stack
5.  Cung cấp CDK assets S3 bucket
6.  Tạo các IAM roles cần thiết cho CDK deployments
7.  Xác minh phiên bản bootstrap trong SSM Parameter Store
8.  Hiểu mục đích và chi phí của mỗi bootstrap resource

Trong thực hành này, bạn đã thành công:

1.  Thực thi lệnh CDK bootstrap cho AWS account của bạn
2.  Tạo CDKToolkit CloudFormation stack
3.  Cung cấp CDK assets S3 bucket để lưu trữ Lambda packages
4.  Tạo các IAM roles cần thiết cho CDK deployments
5.  Xác minh tất cả resources qua CLI và AWS Console

AWS account của bạn giờ đã sẵn sàng để triển khai các ứng dụng CDK serverless, bao gồm các infrastructure stacks cho dự án EveryoneCook.
- Chuẩn bị cho infrastructure deployment
