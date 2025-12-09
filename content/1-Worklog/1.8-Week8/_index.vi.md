---
title: "Báo cáo công việc Tuần 8"
date: 2025-10-27
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---
### Mục tiêu Tuần 8:

* **Tối ưu hóa Chi phí:** Làm chủ các mô hình định giá tiết kiệm như **Savings Plans**, **Reserved Instances (RI)** cho EC2 và RDS.
* **Tối ưu hóa Tài nguyên:** Sử dụng **AWS Compute Optimizer** và CloudWatch để xác định và điều chỉnh quy mô tài nguyên (Right Sizing).
* **Quản lý & Trực quan hóa Chi phí:** Phân tích báo cáo chi phí chi tiết bằng **AWS Cost Explorer** và hiểu rõ các chi phí truyền tải dữ liệu.
* **Đánh giá Kiến thức:** Thực hiện bài kiểm tra giữa kỳ mô phỏng kỳ thi **AWS Certified Solutions Architect**.

### Các nhiệm vụ thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | **Các Mô hình Tiết kiệm Chi phí** <br> - Tìm hiểu và phân biệt **Savings Plans** (Compute/EC2) và **Reserved Instances**. <br> - Phân tích khuyến nghị Savings Plan (Recommendations). <br> - Thực hành quy trình mua Savings Plan và Reserved DB Instances. | 27/10/2025 | 27/10/2025 | https://000042.awsstudygroup.com/ |
| 3 | **Tối ưu hóa Tài nguyên (Right Sizing)** <br> - Cài đặt **CloudWatch Agent** để thu thập metrics bộ nhớ (RAM). <br> - Cấu hình IAM Role cho CloudWatch và Compute Optimizer. <br> - Sử dụng **AWS Compute Optimizer** để nhận khuyến nghị về loại instance tối ưu. <br> - Áp dụng các Best Practices về EC2 Resource Optimization. | 28/10/2025 | 28/10/2025 | https://000032.awsstudygroup.com/ |
| 4 | **Trực quan hóa & Phân tích Chi phí cơ bản** <br> - Sử dụng **Cost Explorer** để xem chi phí theo Dịch vụ (Service) và Tài khoản (Account). <br> - Theo dõi mức độ bao phủ (Coverage) và sử dụng (Utilization) của Savings Plan/RI. <br> - Phân tích tính đàn hồi (Elasticity) của hạ tầng. | 29/10/2025 | 29/10/2025 | https://000034.awsstudygroup.com/ |
| 5 | **Phân tích Chi phí Nâng cao & Ôn tập** <br> - Tạo các **Custom Reports** cho EC2. <br> - Phân tích chi phí **Data Transfer** (truyền tải dữ liệu). <br> - *Nghiên cứu thêm:* Thiết lập AWS Cost & Usage Report (CUR) và truy vấn bằng **Amazon Athena** (Lý thuyết). <br> - Hệ thống hóa lại kiến thức 8 tuần qua để chuẩn bị thi. | 30/10/2025 | 30/10/2025 | https://000034.awsstudygroup.com/ <br><br> _Ôn tập kiến thức SAA_ |
| 6 | **Kiểm tra Giữa kỳ (Midterm Exam)** <br> - Thực hiện bài thi thử **AWS Certified Solutions Architect - Associate (SAA-C03)**. <br> - Định dạng: 65 câu hỏi trắc nghiệm / 130 phút. <br> - Nội dung bao phủ: IAM, VPC, EC2, S3, RDS, HA/Scaling, Cost Optimization. <br> - Đánh giá kết quả và rà soát các lỗ hổng kiến thức. | 31/10/2025 | 31/10/2025 | _Đề thi thử AWS SAA Practice Exam_ |

# Thành tựu Tuần 8

## Chiến lược Tối ưu hóa Chi phí
- Đã nắm vững sự khác biệt giữa **Savings Plans** (linh hoạt hơn) và **Reserved Instances** (truyền thống), từ đó đưa ra quyết định mua phù hợp cho từng loại workload.
- Biết cách sử dụng công cụ **Recommendations** để xác định mức cam kết chi phí tối ưu dựa trên lịch sử sử dụng.
- Thực hiện thành công việc áp dụng Reserved Instance cho cơ sở dữ liệu RDS để giảm chi phí vận hành lâu dài.

## Tối ưu hóa Tài nguyên Hệ thống
- Triển khai thành công **CloudWatch Agent** để thu thập các chỉ số custom (như Memory usage) mà CloudWatch mặc định không cung cấp, giúp đánh giá chính xác hiệu năng instance.
- Sử dụng **AWS Compute Optimizer** để phát hiện các tài nguyên đang bị dư thừa (Over-provisioned) hoặc thiếu hụt (Under-provisioned) và thực hiện điều chỉnh kích thước (Right Sizing) hợp lý.

## Quản trị Tài chính Đám mây (FinOps)
- Làm chủ công cụ **AWS Cost Explorer** để trực quan hóa dòng tiền và chi phí sử dụng theo ngày, tháng, hoặc theo từng dịch vụ cụ thể.
- Tạo được các báo cáo tùy chỉnh (Custom Reports) để theo dõi chi phí Data Transfer - một loại chi phí ẩn thường bị bỏ qua.
- Hiểu rõ cách theo dõi hiệu quả sử dụng của các gói Savings Plan/RI thông qua các biểu đồ Coverage và Utilization.

## Đánh giá Năng lực (Midterm)
- Hoàn thành bài kiểm tra mô phỏng **AWS Certified Solutions Architect**.
- Đã rà soát lại toàn bộ kiến thức nền tảng về 4 trụ cột chính: Compute, Storage, Networking và Database.
- Xác định được các điểm yếu cần cải thiện trong các tuần tiếp theo (ví dụ: các kịch bản về Hybrid Cloud hoặc Advanced Networking).