---
title: "Nhật ký Tuần 4"
date: 2025-09-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu Tuần 4:

* Triển khai kiến trúc **WordPress** cấp độ sản xuất đảm bảo Tính khả dụng Cao (HA) và Khả năng Mở rộng bằng cách sử dụng Nhóm Tự động Mở rộng và CloudFront.
* Làm chủ quy trình **Di chuyển Máy chủ** bằng cách nhập/xuất Máy ảo (VM) giữa môi trường tại chỗ và AWS.
* Thực hiện **Di chuyển Cơ sở dữ liệu** toàn diện cho các nguồn không đồng nhất bằng cách sử dụng Công cụ Chuyển đổi Lược đồ AWS (SCT) và Dịch vụ Di chuyển Cơ sở dữ liệu (DMS).
* Nghiên cứu các chiến lược và công cụ di chuyển đám mây nâng cao (DataSync, MGN, Outposts).

### Nhiệm vụ cần thực hiện trong tuần này:

| Ngày | Nhiệm vụ | Ngày Bắt đầu | Ngày Hoàn thành | Tài liệu Tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **WordPress trên AWS - Phần 1 (Triển khai)** <br> - Chuẩn bị Hạ tầng: VPC, Subnet, Nhóm Bảo mật cho Web & DB <br> - Khởi chạy **RDS Đa-AZ** để đảm bảo tính khả dụng cao của cơ sở dữ liệu <br> - Triển khai EC2 và cài đặt ứng dụng **WordPress** <br> - Kết nối WordPress với phiên bản RDS | 29/09/2025 | 29/09/2025 | https://000021.awsstudygroup.com/ |
| 3 | **WordPress trên AWS - Phần 2 (Mở rộng & CDN)** <br> - Tạo **AMI** từ phiên bản WordPress đã cấu hình <br> - Thiết lập **Nhóm Tự động Mở rộng (ASG)** với Bộ Cân bằng Tải Ứng dụng <br> - Tăng tốc phân phối nội dung bằng **Amazon CloudFront** <br> - Thực hiện các thao tác Sao lưu (Snapshot) & Khôi phục Cơ sở dữ liệu <br> - Dọn dẹp tài nguyên | 30/09/2025 | 30/09/2025 | https://000021.awsstudygroup.com/ |
| 4 | **Di chuyển Máy chủ (Nhập/Xuất VM)** <br> - Triển khai Máy chủ Ứng dụng cục bộ (Mô phỏng Tại chỗ) <br> - **Nhập VM vào AWS:** Xuất VM cục bộ, tải lên S3 và chuyển đổi thành AMI/EC2 <br> - **Xuất VM từ AWS:** Xuất phiên bản EC2 trở lại S3 để sử dụng tại chỗ <br> - Quản lý ACL S3 và vai trò CLI cho các tác vụ di chuyển | 01/10/2025 | 01/10/2025 | https://000014.awsstudygroup.com/ |
| 5 | **Di chuyển Cơ sở dữ liệu (DMS & SCT)** <br> - Thiết lập Môi trường Di chuyển: Nguồn (Oracle/SQL Server) & Đích (Aurora/RDS) <br> - Sử dụng **Công cụ Chuyển đổi Lược đồ (SCT)** để chuyển đổi lược đồ cơ sở dữ liệu <br> - Cấu hình **AWS DMS**: <br>&emsp; + Tạo Phiên bản Sao chép & Điểm cuối <br>&emsp; + Chạy Tác vụ Di chuyển (Tải đầy đủ + CDC) <br> - Khám phá **DMS Serverless** để tự động mở rộng sao chép | 02/10/2025 | 02/10/2025 | https://000043.awsstudygroup.com/ |
| 6 | **Giám sát & Chiến lược Di chuyển Nâng cao** <br> - **Giám sát DMS:** Phân tích chỉ số CloudWatch, Thống kê Bảng và Nhật ký Tác vụ <br> - **Khắc phục sự cố:** Chẩn đoán áp lực bộ nhớ và lỗi bảng trong quá trình di chuyển <br> - **Nghiên cứu:** Khám phá các mẫu di chuyển sắp tới: <br>&emsp; + Dịch vụ Di chuyển Ứng dụng AWS (MGN) <br>&emsp; + AWS DataSync & Migration Hub <br>&emsp; + Di chuyển Container sang EKS | 03/10/2025 | 03/10/2025 | https://000043.awsstudygroup.com/ <br><br> _Tự học về AWS Migration Hub_ |

# Thành tựu Tuần 4

## Kiến trúc Web Có thể Mở rộng
- Triển khai thành công trang web **WordPress** có tính khả dụng cao bằng cách sử dụng **RDS Đa-AZ** và **Nhóm Tự động Mở rộng**.
- Cấu hình **Bộ Cân bằng Tải Ứng dụng (ALB)** để phân phối lưu lượng một cách động trên các phiên bản khỏe mạnh.
- Tối ưu hóa tốc độ phân phối nội dung toàn cầu bằng cách tích hợp **Amazon CloudFront** như một CDN.
- Triển khai các quy trình khôi phục thảm họa bằng cách sử dụng Ảnh chụp nhanh RDS và kỹ thuật khôi phục.

## Chuyên môn Di chuyển Máy chủ
- Có được kinh nghiệm thực hành với các phương pháp **Nhập/Xuất VM**.
- Di chuyển thành công máy ảo từ môi trường mô phỏng tại chỗ (VMware) sang AWS EC2.
- Thành thạo quy trình ngược lại để xuất phiên bản AWS EC2 đang hoạt động trở lại thành ảnh VM di động được lưu trữ trong S3.
- Cấu hình các vai trò IAM và quyền S3 cần thiết để tạo điều kiện truyền ảnh an toàn.

## Di chuyển & Hiện đại hóa Cơ sở dữ liệu
- Thực hiện di chuyển cơ sở dữ liệu không đồng nhất (ví dụ: SQL Server/Oracle sang Aurora) bằng **AWS DMS**.
- Sử dụng **Công cụ Chuyển đổi Lược đồ AWS (SCT)** để tự động hóa việc chuyển đổi lược đồ giữa các công cụ cơ sở dữ liệu khác nhau.
- Cấu hình **Sao chép Dữ liệu Liên tục (CDC)** để giữ cho cơ sở dữ liệu nguồn và đích đồng bộ với thời gian ngừng hoạt động tối thiểu.
- Thử nghiệm với **DMS Serverless** để xử lý khối lượng công việc di chuyển thay đổi tự động.
- Học cách giám sát tình trạng di chuyển bằng CloudWatch và khắc phục các sự cố phổ biến như áp lực bộ nhớ hoặc không khớp kiểu dữ liệu.
