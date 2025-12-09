---
title: "Nhật ký Tuần 3"
date: 2025-09-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---
### Mục tiêu Tuần 3:

* Triển khai các chiến lược giám sát và quan sát toàn diện sử dụng **Amazon CloudWatch** (Chỉ số, Nhật ký, Cảnh báo, Bảng điều khiển).
* Thiết lập kiến trúc **DNS Lai** để tích hợp mạng tại chỗ với AWS sử dụng **Route 53 Resolver** và Microsoft Active Directory.
* Đạt được sự thành thạo trong **AWS CLI** để quản lý tài nguyên nâng cao (Cơ bản về Hạ tầng dưới dạng Mã) và tự động hóa trên các dịch vụ Lưu trữ, Mạng, Danh tính và Điện toán.

### Nhiệm vụ cần thực hiện trong tuần này:

| Ngày | Nhiệm vụ | Ngày Bắt đầu | Ngày Hoàn thành | Tài liệu Tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Giám sát & Quan sát (CloudWatch)** <br> - Cấu hình **Chỉ số CloudWatch**: Xem, Biểu thức tìm kiếm, Biểu thức toán học <br> - Phân tích nhật ký với **CloudWatch Logs Insights** & Bộ lọc Chỉ số <br> - Thiết lập **Cảnh báo CloudWatch** để giám sát chủ động <br> - Tạo **Bảng điều khiển CloudWatch** thống nhất <br> - **Thực hành:** Trực quan hóa hiệu suất EC2 và thiết lập cảnh báo cho mức sử dụng CPU cao. | 22/09/2025 | 22/09/2025 | https://000008.awsstudygroup.com/ |
| 3 | **Kết nối Lai & DNS** <br> - Triển khai **Microsoft Active Directory (AD)** <br> - Cấu hình **Cổng Truy cập từ xa (RDGW)** để truy cập an toàn <br> - Thiết lập **DNS Lai** với Route 53 Resolver: <br>&emsp; + Tạo Điểm cuối Đi ra/Đi vào <br>&emsp; + Cấu hình Quy tắc Resolver <br> - Kiểm tra phân giải DNS giữa môi trường mô phỏng tại chỗ và AWS. | 23/09/2025 | 23/09/2025 | https://000010.awsstudygroup.com/ |
| 4 | **Làm chủ AWS CLI - Phần 1 (Thiết lập & Lưu trữ)** <br> - Cài đặt và Cấu hình **AWS CLI v2** <br> - **Tự động hóa S3:** Quản lý bucket, thực hiện tải lên nhiều phần qua CLI <br> - **Nhắn tin:** Quản lý chủ đề và đăng ký **Amazon SNS** qua CLI <br> - Khám phá các định dạng đầu ra CLI (json, table, text) và bộ lọc. | 24/09/2025 | 24/09/2025 | https://000011.awsstudygroup.com/ <br><br> https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html |
| 5 | **Làm chủ AWS CLI - Phần 2 (Điện toán & Mạng)** <br> - **Bảo mật IAM:** Quản lý người dùng, vai trò và giả định vai trò qua CLI <br>&emsp; + Xử lý mã thông báo MFA với CLI <br> - **Hạ tầng Mạng:** Triển khai VPC, Subnet và Internet Gateway qua CLI <br> - **Điện toán:** Khởi chạy và cấu hình **phiên bản EC2** hoàn toàn qua dòng lệnh. | 25/09/2025 | 25/09/2025 | https://000011.awsstudygroup.com/ <br><br> [Tìm hiểu sâu: Giao diện Dòng lệnh AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) |
| 6 | **Làm chủ AWS CLI - Phần 3 (Khắc phục sự cố & Dọn dẹp)** <br> - **Khắc phục sự cố:** Chẩn đoán các lỗi CLI phổ biến (thông tin xác thực, quyền, định dạng) <br> - **Xác thực Nâng cao:** Lưu trữ thông tin xác thực SAML <br> - Thực hiện dọn dẹp tài nguyên đầy đủ bằng các lệnh CLI để đảm bảo không có chi phí kéo dài. | 26/09/2025 | 26/09/2025 | https://000011.awsstudygroup.com/ <br><br> [AWS re:Invent 2019: Giới thiệu về AWS CLI v2](https://www.youtube.com/watch?v=k_l8k2A8F_k) |

# Thành tựu Tuần 3

## Quan sát & Giám sát
- Triển khai thành công ngăn xếp giám sát sử dụng **Amazon CloudWatch**.
- Tạo **Bảng điều khiển** tùy chỉnh để trực quan hóa các chỉ số quan trọng (CPU, Bộ nhớ, I/O Đĩa) theo thời gian thực.
- Cấu hình **Cảnh báo CloudWatch** để kích hoạt thông báo tự động qua SNS khi vượt ngưỡng.
- Thành thạo **Logs Insights** để truy vấn và phân tích dữ liệu nhật ký ứng dụng một cách hiệu quả.

## Mạng Lai
- Thiết kế và triển khai giải pháp **DNS Lai** kết nối môi trường tại chỗ và đám mây.
- Triển khai **Microsoft Active Directory** trên AWS và tích hợp với Route 53.
- Cấu hình **Điểm cuối Route 53 Resolver** (Đi vào/Đi ra) và **Quy tắc Chuyển tiếp** để phân giải tên miền trên mạng lai.
- Xác minh kết nối an toàn bằng **Cổng Truy cập từ xa (RDGW)**.

## Quản lý Hạ tầng Nâng cao (CLI)
- Chuyển đổi từ quản lý dựa trên Console sang các hoạt động **Giao diện Dòng lệnh (CLI)**.
- Tự động hóa việc cung cấp tài nguyên:
    - **Lưu trữ:** Tạo bucket S3 và quản lý vòng đời/tải lên đối tượng.
    - **Bảo mật:** Quản lý người dùng IAM, chính sách và thực hành **Giả định Vai trò** cho quyền truy cập giữa các tài khoản.
    - **Mạng:** Xây dựng ngăn xếp VPC đầy đủ (VPC, Subnet, IGW, Bảng Định tuyến) bằng các lệnh CLI.
    - **Điện toán:** Khởi chạy phiên bản EC2 với các tập lệnh user-data qua CLI.
- Đạt được kỹ năng khắc phục sự cố cho cấu hình CLI, quản lý thông tin xác thực (MFA/SAML) và xử lý lỗi API.
