---
title: "Nhật ký Tuần 2"
date: 2025-09-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu Tuần 2:

* Học và thực hành sử dụng **AWS Cloud9** cho phát triển ứng dụng trên nền tảng đám mây.  
* Tìm hiểu **lưu trữ website tĩnh với Amazon S3** và cách tăng tốc website bằng **Amazon CloudFront**.  
* Học cách mở rộng ứng dụng với **EC2 Auto Scaling**.  
* Trải nghiệm thực hành với **Amazon CloudWatch** để giám sát và cảnh báo.  
* Khám phá **Mạng lai (Hybrid Networking)** và **tích hợp Dịch vụ Thư mục (Directory Service)**.  
* Thực hành **triển khai & tự động hóa hạ tầng bằng AWS CLI**.  

---

### Nhiệm vụ trong tuần:

| Ngày | Nhiệm vụ                                                                                                                                                                                                                                                                                                                                                               | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo                                                                                                                                                                                                                                         |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2   | - **Phát triển trên nền tảng đám mây với AWS Cloud9** <br>&emsp; + Sử dụng các tính năng cơ bản của Cloud9 (dòng lệnh, tệp văn bản). <br>&emsp; + Sử dụng AWS CLI. <br><br> - **Lưu trữ website tĩnh bằng Amazon S3** <br>&emsp; + Hiểu sự khác nhau giữa Bucket và Object. <br>&emsp; + Kích hoạt tính năng lưu trữ website tĩnh. <br>&emsp; + Cấu hình chặn quyền truy cập công khai. <br>&emsp; + Cấu hình quyền truy cập đối tượng công khai. <br>&emsp; + Tăng tốc bằng CloudFront. <br>&emsp; + Thiết lập sao chép đa vùng (Multi-Region Replication). <br><br> **Thực hành:** <br>&emsp; + Tạo phiên bản Cloud9. <br>&emsp; + Tạo và cấu hình bucket S3. <br>&emsp; + Kiểm tra phân phối CloudFront. | 15/09/2025 | 15/09/2025 | https://000049.awsstudygroup.com/ <br> https://000057.awsstudygroup.com/ |
| 3   | - **Mở rộng ứng dụng với EC2 Auto Scaling** <br>&emsp; + Tìm hiểu khái niệm Auto Scaling. <br>&emsp; + AMI và Launch Template. <br>&emsp; + Thiết lập Load Balancer. <br>&emsp; + Hiểu nhu cầu kỹ thuật và kinh doanh của Auto Scaling. <br><br> **Thực hành:** <br>&emsp; + Tạo Launch Template. <br>&emsp; + Tạo Target Group. <br>&emsp; + Tạo Load Balancer. <br>&emsp; + Tạo Auto Scaling Group. | 16/09/2025 | 16/09/2025 | https://000006.awsstudygroup.com/ |
| 4   | - **Giám sát với Amazon CloudWatch** <br>&emsp; + Các bước chuẩn bị. <br>&emsp; + CloudWatch Metrics. <br>&emsp; + CloudWatch Logs. <br>&emsp; + CloudWatch Alarms. <br>&emsp; + CloudWatch Dashboards. <br><br> **Thực hành:** <br>&emsp; + Thu thập số liệu EC2. <br>&emsp; + Kích hoạt và xem nhật ký. <br>&emsp; + Tạo cảnh báo. <br>&emsp; + Xây dựng bảng điều khiển (Dashboard). | 17/09/2025 | 17/09/2025 | https://000008.awsstudygroup.com/ |
| 5   | - **Mạng lai & Tích hợp Dịch vụ Thư mục (Directory Service)** <br>&emsp; + Các yêu cầu tiên quyết cho kết nối lai. <br>&emsp; + Kết nối qua Direct Connect / VPN / Transit Gateway. <br>&emsp; + Thiết lập tích hợp Active Directory (AD). <br>&emsp; + Cấu hình DNS lai. <br><br> **Thực hành:** <br>&emsp; + Cấu hình kết nối mạng lai. <br>&emsp; + Kết nối môi trường giả lập on-premise qua Transit Gateway. <br>&emsp; + Triển khai Dịch vụ Thư mục (AD). <br>&emsp; + Thiết lập DNS lai. | 18/09/2025 | 18/09/2025 | https://000010.awsstudygroup.com/ |
| 6   | - **Triển khai & Tự động hóa hạ tầng với CLI** <br>&emsp; + Cài đặt & cấu hình AWS CLI. <br>&emsp; + Triển khai hạ tầng (VPC, Subnet, Gateway, Security Group). <br>&emsp; + Quản lý Bucket và Object trong S3. <br>&emsp; + Sử dụng SNS để gửi thông báo. <br>&emsp; + Cấu hình IAM. <br>&emsp; + Khởi tạo EC2 instance. <br>&emsp; + Xử lý sự cố. <br><br> **Thực hành:** <br>&emsp; + Cài đặt CLI & kiểm tra cấu hình. <br>&emsp; + Triển khai hạ tầng bằng CLI. <br>&emsp; + Tạo bucket S3. <br>&emsp; + Cấu hình chủ đề & đăng ký SNS. <br>&emsp; + Áp dụng chính sách IAM. <br>&emsp; + Khởi tạo EC2 & kiểm tra kết nối. <br>&emsp; + Khắc phục lỗi mạng hoặc instance. | 19/09/2025 | 19/09/2025 | https://000011.awsstudygroup.com/ |

---

# Thành tựu Tuần 2

## Ngày 2 – Cloud9 & Lưu trữ Website bằng S3
- Làm việc với **Cloud9 IDE** (dòng lệnh, tệp văn bản).  
- Cài đặt & kiểm tra **AWS CLI**.  
- Cấu hình **bucket S3** để lưu trữ website tĩnh.  
- Kích hoạt **chặn truy cập công khai** và cấu hình quyền đối tượng.  
- Tích hợp và kiểm tra **CloudFront**.  
- Thực hành **sao chép đa vùng (Multi-Region Replication)**.  

## Ngày 3 – EC2 Auto Scaling
- Học **khái niệm Auto Scaling** để tăng tính sẵn sàng & tối ưu chi phí.  
- Hiểu **AMI** và **Launch Template**.  
- Cấu hình **Application Load Balancer** và **Target Group**.  
- Thực hành: Tạo Launch Template, Target Group, Load Balancer, Auto Scaling Group.  

## Ngày 4 – Giám sát với CloudWatch
- Học **CloudWatch Metrics, Logs, Alarms, Dashboards**.  
- Thu thập số liệu EC2 và bật tính năng ghi log.  
- Tạo **Cảnh báo (Alarm)** để gửi thông báo.  
- Xây dựng **bảng điều khiển giám sát (Dashboard)** trực quan.  

## Ngày 5 – Mạng lai (Hybrid Networking)
- Tìm hiểu **Direct Connect, VPN, Transit Gateway**.  
- Triển khai **Active Directory (AD)** trên AWS.  
- Cấu hình **DNS lai** cho việc phân giải tên giữa các môi trường.  
- Thực hành: kết nối mạng lai, tích hợp AD và cấu hình DNS.  

## Ngày 6 – Tự động hóa hạ tầng
- Cài đặt và cấu hình **AWS CLI**.  
- Triển khai tài nguyên hạ tầng (VPC, Subnet, Gateway, Security Group) bằng CLI.  
- Tạo và cấu hình **bucket S3**.  
- Cấu hình **SNS** để gửi thông báo.  
- Áp dụng **chính sách và vai trò IAM**.  
- Khởi tạo **EC2 instance** và kiểm tra khả năng kết nối.  
- Thực hành xử lý lỗi thường gặp khi triển khai hạ tầng AWS.  
