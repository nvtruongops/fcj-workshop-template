---
title: "Nhật ký Tuần 2"
date: 2025-09-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---
### Mục tiêu Tuần 2:

* Làm chủ việc sử dụng AWS Cloud9 IDE và AWS CLI để tương tác với các dịch vụ AWS.
* Tìm hiểu sâu về các dịch vụ Lưu trữ AWS (S3) bao gồm Lưu trữ Trang web Tĩnh, Phiên bản, Sao chép và Mạng Phân phối Nội dung (CloudFront).
* Triển khai và quản lý Cơ sở dữ liệu Quan hệ với Amazon RDS, bao gồm các chiến lược sao lưu và phục hồi.
* Triển khai kiến trúc Tính khả dụng cao và Khả năng mở rộng bằng cách sử dụng Mẫu Khởi chạy, Cân bằng Tải Đàn hồi (ELB) và Nhóm Tự động Mở rộng (ASG).

### Nhiệm vụ cần thực hiện trong tuần này:

| Ngày | Nhiệm vụ | Ngày Bắt đầu | Ngày Hoàn thành | Tài liệu Tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Tương tác với Dịch vụ AWS (Cloud9 & CLI)** <br> - Tạo môi trường AWS Cloud9 <br> - Khám phá Tính năng Cơ bản: Sử dụng dòng lệnh, Làm việc với tệp văn bản <br> - Thực hành sử dụng **AWS CLI** để tương tác với các dịch vụ <br> - Dọn dẹp tài nguyên | 15/09/2025 | 15/09/2025 | https://000049.awsstudygroup.com/ |
| 3 | **Lưu trữ & Phân phối Nội dung (S3 & CloudFront)** <br> - Tạo S3 Bucket & Tải lên Dữ liệu <br> - Cấu hình **Lưu trữ Trang web Tĩnh** trên S3 <br> - Quản lý Chặn Truy cập Công khai & chính sách Đối tượng Công khai <br> - Tăng tốc trang web với tích hợp **Amazon CloudFront** <br> - Triển khai **Phiên bản S3 Bucket** để bảo vệ dữ liệu <br> - Thực hành Di chuyển Đối tượng & **Sao chép Liên Vùng (CRR)** | 16/09/2025 | 16/09/2025 | https://000057.awsstudygroup.com/ |
| 4 | **Quản lý Cơ sở dữ liệu (Amazon RDS)** <br> - Chuẩn bị Mạng: VPC, Nhóm Bảo mật cho EC2 & RDS, Nhóm Subnet DB <br> - Khởi chạy phiên bản EC2 cho ứng dụng <br> - Cung cấp phiên bản cơ sở dữ liệu **Amazon RDS** <br> - Triển khai ứng dụng và kết nối với RDS <br> - Thực hiện các thao tác **Sao lưu và Phục hồi** cho RDS | 17/09/2025 | 17/09/2025 | https://000005.awsstudygroup.com/ |
| 5 | **Kiến trúc Tính khả dụng Cao - Phần 1** <br> - Chuẩn bị Hạ tầng: Mạng, EC2, RDS, Triển khai Máy chủ Web <br> - Tạo **Mẫu Khởi chạy** cho việc khởi chạy phiên bản tiêu chuẩn <br> - Cấu hình **Bộ Cân bằng Tải Ứng dụng (ALB)** <br>&emsp; + Tạo Nhóm Mục tiêu <br>&emsp; + Thiết lập trình lắng nghe Bộ Cân bằng Tải <br> - Kiểm tra phân phối lưu lượng Cân bằng Tải | 18/09/2025 | 18/09/2025 | https://000006.awsstudygroup.com/ |
| 6 | **Kiến trúc Tính khả dụng Cao - Phần 2 (Tự động Mở rộng)** <br> - Tạo **Nhóm Tự động Mở rộng (ASG)** gắn với Bộ Cân bằng Tải <br> - Kiểm tra Giải pháp Mở rộng: <br>&emsp; + **Mở rộng Thủ công**: Điều chỉnh dung lượng thủ công <br>&emsp; + **Mở rộng Theo lịch**: Mở rộng dựa trên thời gian <br>&emsp; + **Mở rộng Động**: Mở rộng dựa trên các chỉ số (CPU, v.v.) <br>&emsp; + **Mở rộng Dự đoán**: Xem xét các chỉ số <br> - Dọn dẹp tất cả tài nguyên | 19/09/2025 | 19/09/2025 | https://000006.awsstudygroup.com/ |

# Thành tựu Tuần 2

## Công cụ Phát triển & CLI
- Thiết lập thành công **AWS Cloud9** như một IDE dựa trên đám mây.
- Thành thạo trong việc sử dụng **AWS CLI** để quản lý dịch vụ mà không cần Console.
- Hiểu được quy trình chỉnh sửa, gỡ lỗi và chạy tập lệnh trực tiếp trên Cloud9.

## Làm chủ Lưu trữ & CDN
- Triển khai **Trang web Tĩnh** sử dụng Amazon S3.
- Bảo mật và tối ưu hóa phân phối nội dung toàn cầu bằng **Amazon CloudFront**.
- Triển khai các chiến lược bảo vệ dữ liệu bằng **Phiên bản S3**.
- Cấu hình **Sao chép Liên Vùng (CRR)** để khôi phục thảm họa và định vị dữ liệu.

## Quản lý Cơ sở dữ liệu
- Triển khai cơ sở dữ liệu quan hệ được quản lý hoàn toàn bằng **Amazon RDS**.
- Kết nối thành công ứng dụng web được lưu trữ trên EC2 với phiên bản RDS.
- Thực hành các nhiệm vụ bảo trì cơ sở dữ liệu bao gồm tạo **Ảnh chụp nhanh** và thực hiện **Khôi phục Tại thời điểm**.
- Quản lý bảo mật mạng cho cơ sở dữ liệu bằng Nhóm Bảo mật và Nhóm Subnet DB.

## Tính khả dụng Cao & Khả năng Mở rộng
- Thiết kế kiến trúc chịu lỗi bằng cách sử dụng **Cân bằng Tải Đàn hồi (ALB)** và **Nhóm Tự động Mở rộng (ASG)**.
- Tạo **Mẫu Khởi chạy** để xác định cấu hình phiên bản cho tự động mở rộng.
- Triển khai và kiểm tra các chính sách mở rộng khác nhau:
    - **Mở rộng Thủ công** để kiểm soát trực tiếp.
    - **Mở rộng Theo lịch** cho các mẫu lưu lượng có thể dự đoán.
    - **Mở rộng Động** để phản ứng với các thay đổi tải thời gian thực.
- Xác minh phân phối lưu lượng và tính khả dụng của ứng dụng trong các sự kiện mở rộng.  
