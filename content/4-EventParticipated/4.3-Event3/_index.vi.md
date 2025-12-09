---
title: "Event 3"
date: 2025-11-17
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo Cáo Sau Sự Kiện: AWS Cloud Mastery Series #2 — DevOps on AWS

## Mục đích Sự kiện

- Củng cố tư duy và văn hóa DevOps.
- Giới thiệu các dịch vụ AWS hỗ trợ quy trình CI/CD: CodeCommit, CodeBuild, CodeDeploy, CodePipeline.
- Đào sâu về mô hình Hạ tầng như Mã (IaC) với CloudFormation và AWS CDK.
- Tìm hiểu các dịch vụ container (ECS, EKS, App Runner) và công cụ quan sát hệ thống (CloudWatch, X-Ray).

## Nội dung Nổi bật

### Phiên sáng: CI/CD Pipeline & Infrastructure as Code (08:30 – 12:00)

- Ôn tập tư duy DevOps, tập trung vào các chỉ số DORA như Deployment Frequency, MTTR.
- Các thành phần CI/CD:
  - **CodeCommit**: quản lý mã nguồn và phân tích chiến lược Git (GitFlow vs. trunk-based).
  - **CodeBuild**: tự động hóa xây dựng và kiểm thử.
  - **CodeDeploy**: các chiến lược triển khai (Blue/Green, Canary, Rolling).
  - **CodePipeline**: tự động hóa toàn bộ pipeline.
- Demo thực hành: chạy thử một pipeline CI/CD hoàn chỉnh.
- IaC:
  - **CloudFormation**: template, stack và drift detection.
  - **AWS CDK**: constructs, reusable patterns và hỗ trợ đa ngôn ngữ.
- Demo & thảo luận: triển khai với CloudFormation và CDK; phân tích lý do lựa chọn từng công cụ.

### Phiên chiều: Containers, Observability & Best Practices (13:00 – 17:00)

- Tổng quan dịch vụ container:
  - Kiến thức nền tảng về Docker.
  - **Amazon ECR**: lưu trữ image, quét bảo mật và quản lý vòng đời.
  - **ECS & EKS**: chiến lược triển khai, scaling và orchestration.
  - **App Runner**: cách triển khai container kiểu PaaS đơn giản.
- Giám sát & quan sát hệ thống:
  - **CloudWatch**: metrics, logs, alarms, dashboards.
  - **AWS X-Ray**: phân tích truy vết trong kiến trúc microservices.
- Best practices:
  - Feature flags và A/B testing.
  - Quản lý sự cố và viết postmortem hiệu quả.
- Demo & case study: so sánh các chiến lược triển khai microservices.

## Kiến thức Rút ra

- Hiểu rõ hơn về văn hóa DevOps, các chỉ số DORA và tư duy tự động hóa.
- Nắm được cách tích hợp các dịch vụ AWS Code để tạo thành pipeline CI/CD hoàn chỉnh cùng các chiến lược triển khai nâng cao.
- Có nền tảng vững chắc về **CloudFormation** và **AWS CDK**, hỗ trợ tối ưu template và xây dựng kiến trúc đa stack.
- Nhận thức tầm quan trọng của quan sát hệ thống toàn diện với CloudWatch và X-Ray để xử lý lỗi và theo dõi hoạt động.
- Các dịch vụ container (ECS/EKS) mở ra định hướng thiết kế kiến trúc linh hoạt và mở rộng cho các dự án tương lai.
- Các demo thực tế giúp hiểu rõ hơn quy trình xây dựng CI/CD và IaC trong môi trường thực tế.

## Trải nghiệm Sự kiện

- Buổi workshop kéo dài cả ngày nhưng được tổ chức rất logic và thực tiễn, giúp việc tiếp thu kiến thức trở nên dễ dàng.
- Các phần demo và case study hỗ trợ tốt cho việc áp dụng vào dự án hiện tại.
- Sự kiện cũng là cơ hội tốt để trao đổi và kết nối với cộng đồng cũng như các chuyên gia trong lĩnh vực Cloud.
