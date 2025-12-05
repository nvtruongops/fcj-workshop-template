---
title: "Nhật ký công việc Tuần 1"
date: 2025-09-08
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---
### Mục tiêu Tuần 1:

* Kết nối và làm quen với các thành viên của First Cloud Journey.  
* Hiểu các dịch vụ cơ bản của AWS, cách sử dụng Console & CLI.  

### Nhiệm vụ cần thực hiện trong tuần:

| Ngày | Nhiệm vụ                                                                                                                                                                                                                          | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo                                                                 |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ---------------------------------------------------------------------------------- |
| 2   | - Làm quen với các thành viên FCJ <br> - Đọc và ghi chú lại nội quy, quy định của đơn vị thực tập                                                                                                                              | 08/09/2025   | 08/09/2025      |                                                                                    |
| 3   | - Tìm hiểu về AWS và các nhóm dịch vụ chính <br>&emsp; + Compute <br>&emsp; + Storage <br>&emsp; + Networking <br>&emsp; + Database <br> - Học cách vẽ kiến trúc AWS trên draw.io <br> - Cách thực hiện workshop AWS <br /><br> | 09/09/2025   | 09/09/2025      | https://cloudjourney.awsstudygroup.com/ <br> <br> https://www.youtube.com/watch?v=l8isyDe-GwY&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=2 <br> <br> https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3|
| 4   | - Tạo tài khoản AWS mới <br> - Quản lý chi phí sử dụng trên AWS bằng AWS Budgets <br> - Yêu cầu hỗ trợ với AWS Support <br> -**Thực hành:** <br>&emsp; + Thiết lập tài khoản AWS đầu tiên, bật MFA, tạo nhóm/người dùng Admin, cấu hình Console và quản lý Support Cases <br>&emsp; + Tạo ngân sách bằng mẫu, bao gồm Cost Budget, Usage Budget, RI Budget và Savings Plans Budget. <br>&emsp; + Thay đổi gói hỗ trợ và tạo yêu cầu hỗ trợ. <br> | 10/09/2025   | 10/09/2025      | https://000001.awsstudygroup.com/ <br> <br> https://000007.awsstudygroup.com/ <br> <br> https://000009.awsstudygroup.com/ |
| 5   | - Quản lý truy cập với AWS Identity and Access Management (AWS IAM) <br> - Triển khai hạ tầng mạng với Amazon Virtual Private Cloud (Amazon VPC) <br>&emsp; + Firewall trong VPC <br>&emsp; + Triển khai Amazon EC2 Instances <br>&emsp; + Thiết lập Site-to-Site VPN Connection trong AWS <br> **Thực hành:** <br>&emsp; + Tạo VPC với subnet, internet gateway, route table, security group, EC2 server, NAT gateway, và môi trường VPN. | 11/09/2025   | 11/09/2025      | https://000002.awsstudygroup.com/ <br><br> https://000003.awsstudygroup.com/ |
| 6   | - Bắt đầu và triển khai ứng dụng trên Amazon Compute Cloud (Amazon EC2) <br>&emsp; + Khởi chạy Microsoft Windows Server 2022 Instance <br>&emsp; + Triển khai ứng dụng Quản lý người dùng AWS trên Amazon Linux 2 <br>&emsp; + Triển khai ứng dụng Node.js trên Amazon EC2 Windows <br>&emsp; + Quản trị chi phí & sử dụng với IAM <br> - Cấp quyền ứng dụng truy cập dịch vụ AWS thông qua IAM Role (AWS IAM) <br>-**Thực hành:** <br>&emsp; + Kết nối tới Windows Server 2022 và Amazon Linux, cài đặt web server LAMP/XAMPP, và triển khai quản trị chi phí với AWS IAM. <br>&emsp; + Tạo S3 bucket | 12/09/2025   | 12/09/2025      | https://000004.awsstudygroup.com/ <br><br> https://000048.awsstudygroup.com/ |

# Thành tựu Tuần 1

## Định hướng & Làm quen
- Làm quen với các thành viên FCJ  
- Đọc và ghi chú lại nội quy, quy định của đơn vị thực tập  

## Nền tảng AWS cơ bản
- Tìm hiểu về AWS và các nhóm dịch vụ chính: **Compute, Storage, Networking, Database**  
- Thực hành vẽ kiến trúc AWS bằng **draw.io**  
- Tham khảo workshop AWS qua tài liệu và video hướng dẫn  

## Thiết lập tài khoản & Quản lý chi phí
- Tạo và cấu hình thành công tài khoản AWS Free Tier  
- Bật **Multi-Factor Authentication (MFA)** để tăng cường bảo mật  
- Tạo **nhóm Admin** và **người dùng Admin**, cấu hình truy cập qua AWS Management Console  
- Thực hành quản lý chi phí bằng **AWS Budgets**:  
  - Cost Budget  
  - Usage Budget  
  - RI Budget  
  - Savings Plans Budget  
- Thay đổi gói hỗ trợ và tạo yêu cầu hỗ trợ  
- Nắm được quy trình quản lý **Support Case** với AWS Support  

## IAM & Triển khai VPC
- Thực hành **Quản lý truy cập** với AWS Identity and Access Management (IAM)  
- Triển khai **hạ tầng mạng** bằng Amazon VPC:  
  - Tạo subnet, internet gateway, route table, và security group  
  - Khởi chạy EC2 instances trong VPC  
  - Cấu hình **NAT Gateway** và thiết lập **Site-to-Site VPN connection**  

## EC2 & Triển khai ứng dụng
- Khởi chạy và kết nối tới **Microsoft Windows Server 2022** và **Amazon Linux 2**  
- Cài đặt và cấu hình **LAMP** trên Linux và **XAMPP** trên Windows  
- Triển khai ứng dụng thử nghiệm bao gồm **Node.js** trên Amazon EC2  
- Thực hiện **quản trị chi phí & sử dụng** bằng IAM policies và roles  
- Cấp quyền cho ứng dụng truy cập dịch vụ AWS thông qua **IAM Roles**  
- Tạo và quản lý thành công **S3 buckets** để lưu trữ  
