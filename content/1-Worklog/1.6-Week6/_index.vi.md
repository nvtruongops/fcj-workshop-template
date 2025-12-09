---
title: "Báo cáo công việc Tuần 6"
date: 2025-10-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu Tuần 6:

* **Quản lý Danh tính & Truy cập (IAM):** Triển khai kiểm soát danh tính mạnh mẽ bằng **IAM Identity Center (SSO)**, giới hạn đặc quyền với **Permission Boundaries**, và bảo mật việc giả lập vai trò (role assumption) bằng các chính sách có điều kiện.
* **Tuân thủ Bảo mật & Phòng thủ:** Tự động hóa kiểm tra bảo mật với **AWS Security Hub** và bảo vệ ứng dụng web bằng **AWS WAF**.
* **Bảo vệ Dữ liệu:** Làm chủ mã hóa dữ liệu sử dụng **AWS KMS** (Key Management Service) và thiết lập chiến lược sao lưu toàn diện với **AWS Backup**.
* **Mạng Nâng cao:** Triển khai kiến trúc mạng mở rộng sử dụng **VPC Peering** cho kết nối trực tiếp và **Transit Gateway** cho mô hình hub-and-spoke.

### Các nhiệm vụ thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Quản lý Danh tính Nâng cao** <br> - Thiết lập **IAM Identity Center (SSO)** để truy cập tập trung giữa các tài khoản. <br> - Tạo **Permission Boundaries** để giới hạn quyền tối đa cho IAM User/Role. <br> - Cấu hình **IAM Role Conditions** để ngăn chặn lạm dụng chuyển giao vai trò. | 13/10/2025 | 13/10/2025 | https://000012.awsstudygroup.com/ <br><br> https://000030.awsstudygroup.com/ <br><br> https://000044.awsstudygroup.com/ |
| 3 | **Tư thế Bảo mật & Phòng thủ Ứng dụng** <br> - Bật **AWS Security Hub** để kiểm tra tuân thủ (theo chuẩn CIS). <br> - Triển khai **AWS WAF** (Tường lửa ứng dụng web): <br>&emsp; + Tạo Web ACLs với các Managed Rules. <br>&emsp; + Chặn tấn công SQL Injection/XSS trên ALB. | 14/10/2025 | 14/10/2025 | https://000018.awsstudygroup.com/ <br><br> https://000026.awsstudygroup.com/ |
| 4 | **Bảo vệ Dữ liệu (Mã hóa & Sao lưu)** <br> - Tạo Customer Managed Keys (CMK) trong **AWS KMS**. <br> - Mã hóa S3 bucket và kiểm tra việc sử dụng qua **CloudTrail** & **Athena**. <br> - Cấu hình **AWS Backup**: <br>&emsp; + Tạo Backup Vault và Backup Plan. <br>&emsp; + Thực hiện sao lưu và khôi phục tài nguyên theo yêu cầu. | 15/10/2025 | 15/10/2025 | https://000033.awsstudygroup.com/ <br><br> https://000013.awsstudygroup.com/ |
| 5 | **Độ tin cậy Mạng (Peering & Transit)** <br> - **VPC Peering:** Kết nối hai VPC cô lập và cấu hình Route Tables/DNS. <br> - **Transit Gateway (TGW):** Xây dựng hub trung tâm để kết nối nhiều VPC. <br> - Cấu hình TGW Route Tables để phân tách lưu lượng mạng. | 16/10/2025 | 16/10/2025 | https://000019.awsstudygroup.com/ <br><br> https://000020.awsstudygroup.com/ |
| 6 | **Rà soát & Dọn dẹp** <br> - Rà soát lại tất cả các cấu hình bảo mật. <br> - **Quan trọng:** Xóa Transit Gateway, NAT Gateways, và tắt Security Hub/Config để tránh chi phí cao. <br> - Xác nhận việc xóa các KMS keys (lên lịch xóa). | 17/10/2025 | 17/10/2025 | _Tất cả các phần Dọn dẹp_ |

# Thành tựu Tuần 6

## Quản lý Danh tính & Truy cập (IAM)
- Đã triển khai thành công **IAM Identity Center** để quản lý người dùng tập trung và đơn giản hóa việc đăng nhập truy cập trên toàn tổ chức.
- Áp dụng **Permission Boundaries** để ngăn chặn leo thang đặc quyền, đảm bảo người dùng không thể tự cấp quyền vượt quá mức cho phép.
- Tăng cường bảo mật bằng cách thêm **Conditions (Điều kiện)** vào IAM Roles, giới hạn cách thức và nơi vai trò có thể được đảm nhận hoặc chuyển giao (ví dụ: giới hạn việc chuyển role cho các dịch vụ cụ thể).

## Tuân thủ Bảo mật & Phòng thủ Ứng dụng
- Kích hoạt **AWS Security Hub** để có cái nhìn toàn diện về trạng thái bảo mật và tự động kiểm tra theo tiêu chuẩn CIS AWS Foundations Benchmark.
- Bảo vệ các ứng dụng web bằng cách triển khai **AWS WAF**, tạo thành công các quy tắc chặn các mối đe dọa phổ biến như SQL Injection và các địa chỉ IP độc hại trước khi chúng tiếp cận Application Load Balancer.

## Chiến lược Mã hóa & Sao lưu Dữ liệu
- Quản lý bảo mật dữ liệu sử dụng **AWS KMS**, tạo Customer Managed Keys để mã hóa dữ liệu nhạy cảm trong S3.
- Kiểm toán việc sử dụng khóa và truy cập dữ liệu bằng cách tích hợp nhật ký KMS với **CloudTrail** và truy vấn thông qua **Amazon Athena**.
- Thiết lập kế hoạch Khôi phục Thảm họa (DR) mạnh mẽ bằng **AWS Backup**, tự động hóa lịch trình sao lưu cho các EBS volume và kiểm thử thành công việc khôi phục dữ liệu từ điểm sao lưu.

## Kết nối & Mạng Nâng cao
- Kết nối các môi trường mạng cô lập sử dụng **VPC Peering**, cho phép giao tiếp riêng tư giữa các VPC mà không cần đi qua internet công cộng.
- Giải quyết sự phức tạp của mạng lưới dạng lưới (mesh) bằng cách triển khai **AWS Transit Gateway**, tạo kiến trúc mạng mô hình hub-and-spoke có khả năng mở rộng để kết nối nhiều VPC một cách hiệu quả.
- Cấu hình các **Route Table** phức tạp cho cả Peering và Transit Gateway để đảm bảo luồng lưu lượng chính xác và sự cô lập mạng.