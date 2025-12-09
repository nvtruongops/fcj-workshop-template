---
title: "Báo cáo công việc Tuần 7"
date: 2025-10-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---
### Mục tiêu Tuần 7:

* **Đóng gói ứng dụng (Containerization):** Làm chủ việc đóng gói ứng dụng sử dụng **Docker** và triển khai trên EC2.
* **Điều phối Container (Orchestration):** Triển khai và quản lý các ứng dụng container có khả năng mở rộng sử dụng **Amazon ECS (Elastic Container Service)**.
* **DevOps & Tự động hóa:** Triển khai **CI/CD Pipelines** (AWS CodePipeline, CodeBuild) để tự động hóa việc triển khai lên EC2 và ECS.
* **Giải pháp Lưu trữ:** Tích hợp môi trường on-premises với lưu trữ đám mây sử dụng **File Storage Gateway** và **FSx for Windows**.
* **Kiến trúc Cơ sở dữ liệu Nâng cao:** Tìm hiểu sâu về các mẫu thiết kế (design patterns), chiến lược đánh chỉ mục và mô hình hóa dữ liệu cho các ứng dụng hiệu năng cao với **Amazon DynamoDB**.

### Các nhiệm vụ thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Đóng gói Ứng dụng (Docker)** <br> - Cài đặt Docker trên EC2 & Máy cục bộ. <br> - Build Docker images cho ứng dụng mẫu (Frontend/Backend). <br> - Sử dụng **Docker Compose** để điều phối nhiều container. <br> - Đẩy (Push) images lên **Amazon ECR** (Elastic Container Registry). | 20/10/2025 | 20/10/2025 | https://000015.awsstudygroup.com/ |
| 3 | **Điều phối Container (Amazon ECS)** <br> - Tạo **ECS Cluster** (Launch Type: Fargate/EC2). <br> - Định nghĩa **Task Definitions** và Services. <br> - Cấu hình **Application Load Balancer (ALB)** để phân phối lưu lượng. <br> - Triển khai Service Discovery với **AWS Cloud Map**. | 21/10/2025 | 21/10/2025 | https://000016.awsstudygroup.com/ |
| 4 | **CI/CD cho Containers** <br> - Thiết lập **AWS CodePipeline** kích hoạt từ GitHub/GitLab. <br> - Cấu hình **AWS CodeBuild** để build và push Docker images. <br> - Tự động hóa deploy lên ECS Clusters. <br> - Triển khai Monitoring với **Container Insights** và Logging với **FireLens**. | 22/10/2025 | 22/10/2025 | https://000017.awsstudygroup.com/ |
| 5 | **CI/CD cho EC2 & Lưu trữ Hybrid** <br> - **CodePipeline cho EC2:** Tự động hóa deploy sử dụng CodeDeploy agent. <br> - **Storage Gateway:** Cấu hình File Gateway để mount S3 buckets dưới dạng NFS shares. <br> - **Amazon FSx:** Triển khai Windows File Server được quản lý hoàn toàn và mount qua SMB. | 23/10/2025 | 23/10/2025 | https://000023.awsstudygroup.com/ <br><br> _Tự nghiên cứu về FSx/Storage Gateway_ |
| 6 | **Kiến trúc DynamoDB Nâng cao** <br> - **Mô hình hóa dữ liệu:** Thiết kế theo mẫu Single Table Design. <br> - **Tối ưu hiệu năng:** Làm việc với Partition Keys, Sort Keys, và Capacity Units (RCU/WCU). <br> - **Tính năng nâng cao:** Triển khai Global Secondary Indexes (GSI), DynamoDB Streams, và Global Tables. | 24/10/2025 | 24/10/2025 | https://000039.awsstudygroup.com/ |

# Thành tựu Tuần 7

## Containerization & Orchestration
- Đã thực hiện "dockerize" thành công một ứng dụng nguyên khối (monolithic), tách biệt nó khỏi cơ sở hạ tầng bên dưới.
- Triển khai cụm container có tính sẵn sàng cao sử dụng **Amazon ECS**, tận dụng Task Definitions để định nghĩa yêu cầu tài nguyên.
- Cấu hình **Service Discovery** cho phép các microservices trong cụm giao tiếp qua tên logic thay vì địa chỉ IP.
- Triển khai **Application Load Balancer** để phân phối lưu lượng public tới các container target động.

## Tự động hóa DevOps & CI/CD
- Xây dựng **CI/CD Pipeline** mạnh mẽ sử dụng AWS CodePipeline và CodeBuild.
- Đạt được **Continuous Deployment (Triển khai liên tục)**: Các thay đổi code đẩy lên repository tự động kích hoạt quá trình build, tạo image và cập nhật rolling update cho dịch vụ ECS mà không gây gián đoạn (zero downtime).
- Tích hợp giám sát toàn diện sử dụng **CloudWatch Container Insights** để theo dõi mức sử dụng CPU/Memory của các task.
- Tập trung hóa log của container sử dụng **FireLens** để dễ dàng gỡ lỗi và phân tích.

## Triển khai Lưu trữ Hybrid
- Kết nối khoảng cách giữa các giao thức on-premises và lưu trữ đám mây.
- Cấu hình **File Storage Gateway** cho phép các ứng dụng cũ (legacy) ghi dữ liệu vào S3 bằng giao thức NFS tiêu chuẩn.
- Triển khai **Amazon FSx for Windows File Server**, cung cấp hệ thống tệp Windows native được quản lý hoàn toàn với khả năng tích hợp Active Directory.

## Thiết kế Cơ sở dữ liệu Hiệu năng cao
- Làm chủ các khái niệm NoSQL với **Amazon DynamoDB**.
- Tối ưu hóa hiệu năng cơ sở dữ liệu bằng cách thiết kế Partition Key và Sort Key hiệu quả.
- Triển khai **Global Secondary Indexes (GSI)** để hỗ trợ các mẫu truy cập đa dạng mà không cần nhân bản bảng dữ liệu.
- Khám phá **DynamoDB Streams** cho kiến trúc hướng sự kiện (kích hoạt Lambda function khi dữ liệu thay đổi).