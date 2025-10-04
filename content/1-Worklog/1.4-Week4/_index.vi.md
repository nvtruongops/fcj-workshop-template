---
title: "Nhật ký Tuần 4"
date: 2025-09-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu Tuần 4:

* Tìm hiểu và thực hành **Serverless Automation** với **AWS Lambda** để tối ưu chi phí vận hành EC2.  
* Triển khai **giám sát nâng cao (Advanced Monitoring)** với **Amazon CloudWatch** và **Grafana** nhằm tăng khả năng quan sát hệ thống.  
* Khám phá các tính năng **Metrics, Logs, Alarms và Dashboards** trong CloudWatch.  
* Quản lý và kiểm soát truy cập thông qua **IAM Policies** và **Resource Tags**.  
* Tìm hiểu cách sử dụng **AWS Systems Manager** để quản lý bản vá (patch), chạy lệnh từ xa và quản trị tập trung.  
* Triển khai **Hạ tầng dưới dạng mã (Infrastructure as Code – IaC)** với **AWS CloudFormation** và **AWS CDK**.  
* Xây dựng kiến trúc nâng cao, đa dịch vụ và triển khai **nested stacks** bằng **AWS CDK Advanced**.  

---

### Nhiệm vụ thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ----------- | ---------------- | ------------------ |
| 2 | - **Tự động hóa Serverless với AWS Lambda** <br>&emsp; + Tự động khởi động/dừng EC2 theo lịch bằng Lambda. <br>&emsp; + Cấu hình IAM Role và tag cho Lambda. <br>&emsp; + Kiểm thử tự động hóa và xác minh kết quả. <br><br> - **Giám sát nâng cao với CloudWatch và Grafana** <br>&emsp; + Cài đặt Grafana trên EC2. <br>&emsp; + Kết nối Grafana với CloudWatch. <br>&emsp; + Tạo dashboard và thiết lập cảnh báo. | 30/09/2025 | 30/09/2025 |  <br> https://000022.awsstudygroup.com/ <br> https://000029.awsstudygroup.com/ |
| 3 | - **CloudWatch Advanced Workshop** <br>&emsp; + Khám phá các tính năng Metrics, Logs, Alarms và Dashboards. <br>&emsp; + Sử dụng Container Insights cho ECS, EKS, Fargate. <br><br> - **Tổ chức tài nguyên với Tags và Resource Groups** <br>&emsp; + Tạo và quản lý thẻ (tag) cho EC2, S3, IAM. <br>&emsp; + Xây dựng nhóm tài nguyên dựa trên tag để quản lý tập trung. | 01/10/2025 | 01/10/2025 | https://000036.awsstudygroup.com/ <br> https://000027.awsstudygroup.com/ |
| 4 | - **Kiểm soát truy cập với IAM và Resource Tags** <br>&emsp; + Định nghĩa chính sách IAM có điều kiện theo tag để giới hạn quyền. <br>&emsp; + Tạo IAM Role cho quản trị viên EC2 với quyền truy cập có điều kiện. <br>&emsp; + Kiểm thử và xác minh kết quả. <br><br> - **Quản lý hệ thống với AWS Systems Manager** <br>&emsp; + Cấu hình Patch Manager để tự động cập nhật hệ điều hành. <br>&emsp; + Sử dụng Run Command để chạy lệnh trên nhiều EC2 cùng lúc. <br>&emsp; + Xem báo cáo tuân thủ và dọn dẹp tài nguyên. | 02/10/2025 | 02/10/2025 | https://000028.awsstudygroup.com/ <br> https://000031.awsstudygroup.com/ |
| 5 | - **Triển khai Hạ tầng dưới dạng mã với AWS CloudFormation** <br>&emsp; + Tạo template CloudFormation để triển khai hạ tầng AWS. <br>&emsp; + Quản lý tài nguyên qua CloudFormation Stack. <br><br> - **AWS Cloud Development Kit (CDK) Cơ bản** <br>&emsp; + Viết mã CDK bằng ngôn ngữ lập trình (Python/TypeScript). <br>&emsp; + Triển khai và cập nhật tài nguyên qua CDK. | 03/10/2025 | 03/10/2025 | https://000037.awsstudygroup.com/ <br> https://000038.awsstudygroup.com/ |
| 6 | - **AWS CDK Nâng Cao (Advanced)** <br>&emsp; + Xây dựng kiến trúc đa dịch vụ với API Gateway, ECS, ALB, Lambda và S3. <br>&emsp; + Tạo nested stacks để triển khai mô-đun hóa. <br><br> - **Chuỗi Workshop về Hạ tầng dưới dạng mã (Infrastructure as Code Series)** <br>&emsp; + Hiểu khái niệm và lợi ích của IaC. <br>&emsp; + Thực hành triển khai kiến trúc nhiều tầng với CloudFormation và CDK. | 04/10/2025 | 04/10/2025 | https://000076.awsstudygroup.com/ <br> https://000102.awsstudygroup.com/ |

---

# Kết quả đạt được trong Tuần 4

## Ngày 2 – Tự động hóa Serverless & Giám sát nâng cao
- Xây dựng **AWS Lambda** để tự động bật/tắt EC2 theo lịch định sẵn, giúp tiết kiệm chi phí vận hành.  
- Cấu hình **IAM Role** và tag cho Lambda, kiểm thử hoạt động tự động hóa.  
- Tìm hiểu **Savings Plans** cho bài toán tối ưu chi phí vận hành liên tục.  
- Cài đặt **Grafana** trên EC2 và tích hợp với **CloudWatch** để giám sát thời gian thực.  
- Tạo **dashboard động** hiển thị các chỉ số CPU, Memory, Network.  
- Thiết lập cảnh báo và tự động thông báo khi vượt ngưỡng.  
- Nâng cao khả năng quan sát hệ thống và quản lý hạ tầng chủ động.

## Ngày 3 – CloudWatch Nâng cao & Tổ chức tài nguyên
- Hiểu sâu về **Amazon CloudWatch** trong việc giám sát hạ tầng và ứng dụng.  
- Cấu hình **Metrics, Logs, Alarms và Dashboards** để phát hiện sự cố sớm.  
- Sử dụng **Container Insights** để theo dõi ECS, EKS và Fargate.  
- Xây dựng **CloudWatch Dashboard** tổng hợp các chỉ số hệ thống.  
- Áp dụng **Tagging** để phân loại tài nguyên theo nhóm (chủ sở hữu, môi trường, dự án).  
- Tạo **Resource Groups** dựa trên tag để quản lý hàng loạt tài nguyên.  
- Cải thiện khả năng quản trị và hiệu suất giám sát hệ thống.

## Ngày 4 – Kiểm soát truy cập & Quản lý hệ thống
- Áp dụng **chính sách IAM theo tag** để thực thi nguyên tắc quyền tối thiểu (Least Privilege).  
- Tạo **IAM Role và Policy** có điều kiện (`aws:ResourceTag`, `StringEquals`) để kiểm soát truy cập EC2.  
- Kiểm thử hành vi truy cập để đảm bảo chính sách hoạt động đúng.  
- Sử dụng **AWS Systems Manager (SSM)** để quản lý và tự động hóa nhiệm vụ vận hành.  
- Cấu hình **Patch Manager** để cập nhật hệ điều hành và bản vá bảo mật tự động.  
- Thực hiện **Run Command** trên nhiều máy chủ EC2 cùng lúc.  
- Củng cố bảo mật và hiệu quả vận hành thông qua tự động hóa.

## Ngày 5 – Hạ tầng dưới dạng mã với CloudFormation & CDK
- Thực hành triển khai tài nguyên AWS tự động bằng **CloudFormation Templates** (YAML).  
- Quản lý **Stack** để triển khai VPC, Subnet và EC2 một cách thống nhất.  
- Tìm hiểu cơ chế **rollback và update stack** an toàn khi triển khai.  
- Triển khai **AWS CDK** để định nghĩa hạ tầng bằng mã Python/TypeScript.  
- Thực hiện **deploy, update và remove** hạ tầng qua CDK.  
- Làm quen với quy trình **DevOps – IaC** trong quản lý hạ tầng.

## Ngày 6 – AWS CDK Nâng cao & Hạ tầng dưới dạng mã
- Thiết kế và triển khai **kiến trúc đa dịch vụ** với **AWS CDK v2.151.0**, bao gồm **API Gateway, ALB, ECS, Lambda, S3**.  
- Áp dụng **nested stacks** để tái sử dụng và quản lý hạ tầng mô-đun.  
- Thực hành triển khai ứng dụng nhiều tầng (multi-tier) bằng IaC.  
- Hiểu rõ lợi ích và so sánh các framework IaC: **CloudFormation, SAM, CDK, Terraform, Pulumi**.  
- Tích hợp IaC vào **CI/CD pipeline** để tự động triển khai và theo dõi thay đổi hạ tầng.  
- Củng cố kỹ năng thiết kế, tự động hóa và quản lý hạ tầng AWS ở quy mô lớn.

---

### Tổng kết
- Tự động hóa quản lý EC2 bằng **AWS Lambda**, giúp giảm chi phí.  
- Nâng cao khả năng giám sát bằng **CloudWatch**, **Grafana** và **Container Insights**.  
- Thực thi **kiểm soát truy cập theo tag** với IAM để tăng bảo mật.  
- Quản lý và tự động hóa bản vá bằng **AWS Systems Manager**.  
- Triển khai **Infrastructure as Code** qua **CloudFormation** và **CDK**.  
- Xây dựng **kiến trúc CDK nâng cao** với nhiều dịch vụ tích hợp.  
- Nâng cao kỹ năng **DevOps**, **tự động hóa hạ tầng**, và **quản trị hiệu quả đám mây** trên AWS.

---
