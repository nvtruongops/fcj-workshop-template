---
title: "Nhật ký Tuần 5"
date: 2025-10-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---
### Mục tiêu Tuần 5:

* **Xuất sắc Vận hành:** Tự động hóa các tác vụ vận hành (khởi động/dừng máy chủ) bằng **AWS Lambda** và tích hợp với các công cụ truyền thông (Slack).
* **Giám sát & Quan sát:** Xây dựng bảng điều khiển nâng cao sử dụng **Grafana** tích hợp với **Amazon CloudWatch** để có cái nhìn sâu sắc về hệ thống.
* **Quản trị & Gắn thẻ:** Triển khai các chiến lược tổ chức tài nguyên bằng cách sử dụng **Thẻ**, **Nhóm Tài nguyên** và thực thi Kiểm soát Truy cập Dựa trên Thuộc tính (ABAC) thông qua **IAM**.
* **Quản lý Hệ thống (SSM):** Khám phá quản lý máy chủ mà không cần SSH bằng **Session Manager** và hiểu quy trình vá tự động (Patch Manager).
* **Hạ tầng dưới dạng Mã (IaC):** Bắt đầu tự động hóa triển khai hạ tầng bằng **AWS CloudFormation**.

### Nhiệm vụ cần thực hiện trong tuần này:

| Ngày | Nhiệm vụ | Ngày Bắt đầu | Ngày Hoàn thành | Tài liệu Tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Tối ưu Chi phí & Tự động hóa** <br> - Tạo **hàm Lambda** (Python boto3) để tự động Khởi động/Dừng các phiên bản EC2 dựa trên lịch trình. <br> - Tích hợp Lambda với **Slack Webhooks** để thông báo vận hành. <br> - Cấu hình trình kích hoạt **EventBridge** (CloudWatch Events). | 06/10/2025 | 06/10/2025 | https://000022.awsstudygroup.com/ |
| 3 | **Giám sát Nâng cao (Grafana)** <br> - Khởi chạy EC2 để lưu trữ **Grafana** (Mã nguồn Mở). <br> - Kết nối Grafana với nguồn dữ liệu **Amazon CloudWatch**. <br> - Tạo bảng điều khiển tùy chỉnh trực quan hóa các chỉ số CPU, Mạng và Đĩa từ EC2. | 07/10/2025 | 07/10/2025 | https://000029.awsstudygroup.com/ |
| 4 | **Quản trị & Bảo mật (Thẻ & IAM)** <br> - **Chiến lược Gắn thẻ:** Áp dụng các thẻ nhất quán cho tài nguyên (Env, Owner, CostCenter). <br> - Tạo **Nhóm Tài nguyên** để quản lý tài nguyên theo thẻ. <br> - **Triển khai ABAC:** Cấu hình Chính sách IAM để kiểm soát quyền truy cập EC2 dựa trên thẻ khớp giữa Người dùng và Tài nguyên. | 08/10/2025 | 08/10/2025 | https://000027.awsstudygroup.com/ <br><br> https://000028.awsstudygroup.com/ |
| 5 | **Quản lý Hệ thống (SSM)** <br> - **Session Manager:** Kết nối với các phiên bản riêng tư/công khai mà không cần mở cổng SSH (Chuyển tiếp Cổng, Nhật ký Kiểm tra sang S3). <br> - **Run Command:** Thực thi lệnh từ xa trên nhiều phiên bản. <br> - *Lý thuyết/Xem xét:* Quy trình **Patch Manager** (quét & tuân thủ) do giới hạn quyền Free Tier/Lab tiềm ẩn. | 09/10/2025 | 09/10/2025 | https://000058.awsstudygroup.com/ <br><br> https://000031.awsstudygroup.com/ |
| 6 | **Hạ tầng dưới dạng Mã (CloudFormation)** <br> - Viết **mẫu CloudFormation** cơ bản (YAML/JSON) để triển khai VPC và EC2. <br> - Hiểu về **Stack**, **Tham số**, **Ánh xạ** và **Đầu ra**. <br> - Khám phá **Phát hiện Lệch** để xác định các thay đổi thủ công đối với hạ tầng. | 10/10/2025 | 10/10/2025 | https://000037.awsstudygroup.com/ |

# Thành tựu Tuần 5

## Tự động hóa & Quản lý Chi phí
- Triển khai thành công **cơ chế tiết kiệm chi phí tự động** bằng cách lên lịch các phiên bản EC2 không sản xuất dừng lại trong giờ nghỉ bằng **AWS Lambda**.
- Triển khai hệ thống thông báo nơi các sự kiện vận hành kích hoạt tin nhắn đến **kênh Slack**, cải thiện khả năng hiển thị của nhóm.
- Giảm can thiệp thủ công bằng cách tận dụng **Quy tắc EventBridge** để kích hoạt quy trình tự động hóa.

## Trực quan hóa Giám sát
- Triển khai phiên bản **Grafana** tự lưu trữ trên EC2.
- Tích hợp thành công **Amazon CloudWatch** làm nguồn dữ liệu cho Grafana, cho phép trực quan hóa các chỉ số AWS trong giao diện thống nhất, có thể tùy chỉnh.
- Tạo bảng điều khiển cụ thể để giám sát tình trạng EC2, vượt qua các giới hạn của chế độ xem CloudWatch mặc định.

## Quản trị Tài nguyên & Bảo mật
- Thành thạo **chiến lược Gắn thẻ** để tổ chức tài nguyên theo môi trường (Dev/Prod) và dự án.
- Triển khai **Nhóm Tài nguyên** để quản lý hàng loạt tài nguyên chia sẻ các thẻ chung.
- Tăng cường bảo mật bằng cách sử dụng **Kiểm soát Truy cập Dựa trên Thuộc tính (ABAC)**: Cấu hình các chính sách IAM cho phép người dùng chỉ quản lý các phiên bản EC2 khớp với thẻ phòng ban cụ thể của họ, cách ly hiệu quả các môi trường.

## Vận hành Hệ thống (SysOps)
- Loại bỏ nhu cầu về Bastion Host/khóa SSH bằng cách sử dụng **AWS Systems Manager Session Manager** để truy cập phiên bản an toàn.
- Cấu hình **Ghi nhật ký Phiên** sang S3 cho mục đích kiểm tra và tuân thủ.
- Nghiên cứu và mô phỏng quy trình **Patch Manager** để tự động hóa cập nhật hệ điều hành và duy trì tuân thủ trên toàn bộ các phiên bản.
- Sử dụng **Run Command** để thực thi các tập lệnh quản trị trên nhiều máy chủ đồng thời mà không cần đăng nhập vào từng máy.

## Hạ tầng dưới dạng Mã (IaC)
- Chuyển đổi từ các thao tác thủ công trên console sang định nghĩa hạ tầng bằng mã sử dụng **AWS CloudFormation**.
- Tạo và triển khai Stack để cung cấp VPC và các phiên bản EC2, đảm bảo triển khai nhất quán và có thể lặp lại.
- Học cách phát hiện "Lệch Cấu hình" để xác định nơi trạng thái hạ tầng thực tế lệch khỏi các mẫu CloudFormation đã định nghĩa.


