---
title: "Blog 2: Các mô hình Qwen hiện đã có sẵn trong Amazon Bedrock"
date: 2025-09-18
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Tổng quan: Mở rộng lựa chọn mô hình với Qwen từ Alibaba Cloud

Amazon Bedrock tiếp tục mở rộng thư viện Generative AI của mình bằng cách bổ sung các mô hình **Qwen3** từ Alibaba Cloud. Đây là các mô hình nền tảng (Foundation Models - FMs) trọng số mở, được quản lý hoàn toàn và không yêu cầu máy chủ (serverless).

Việc ra mắt này mang đến cho các nhà phát triển nhiều lựa chọn linh hoạt hơn, từ các mô hình **Mixture-of-Experts (MoE)** hiệu suất cao đến các mô hình **Dense** (dày đặc) tối ưu chi phí, đáp ứng đa dạng nhu cầu từ lập trình, suy luận phức tạp đến triển khai trên các thiết bị biên.

---

## 1. Các biến thể Qwen3 có sẵn trên Bedrock

Bản phát hành này bao gồm 4 biến thể mô hình chính, mỗi biến thể được tối ưu hóa cho các yêu cầu cụ thể về hiệu suất và chi phí:

1.  **Qwen3-Coder-480B-A35B-Instruct:** Mô hình MoE khổng lồ với 480 tỷ tham số (35 tỷ kích hoạt). Tối ưu hóa cho các tác vụ lập trình và tác tử (agentic), xử lý tốt việc phân tích kho mã nguồn lớn.
2.  **Qwen3-Coder-30B-A3B-Instruct:** Mô hình MoE nhỏ gọn hơn, chuyên biệt cho việc tạo mã, gỡ lỗi và tuân theo hướng dẫn lập trình đa ngôn ngữ.
3.  **Qwen3-235B-A22B-Instruct-2507:** Mô hình MoE cân bằng giữa khả năng và hiệu quả, cạnh tranh mạnh mẽ trong các tác vụ suy luận chung và toán học.
4.  **Qwen3-32B (Dense):** Mô hình kiến trúc dày đặc, phù hợp cho các môi trường thời gian thực hoặc hạn chế tài nguyên (như thiết bị di động/edge) nhờ hiệu suất ổn định.

---

## 2. Các tính năng kiến trúc nổi bật

Các mô hình Qwen3 giới thiệu nhiều cải tiến kỹ thuật quan trọng giúp giải quyết các bài toán doanh nghiệp phức tạp.

* **Chế độ tư duy lai (Hybrid Thinking Modes):** Hỗ trợ hai chế độ giải quyết vấn đề:
    * *Thinking mode:* Suy luận từng bước cho các vấn đề phức tạp.  
    * *Non-thinking mode:* Phản hồi tức thì cho các tác vụ đơn giản, giúp tối ưu hóa chi phí.
* **Khả năng Agentic & Gọi công cụ (Tool Use):** Các mô hình có khả năng lập kế hoạch nhiều bước và giao tiếp chuẩn hóa với các API bên ngoài, lý tưởng để xây dựng các quy trình tự động hóa.
* **Cửa sổ ngữ cảnh siêu lớn (Long-context):** Dòng Qwen3-Coder hỗ trợ ngữ cảnh lên tới **1 triệu tokens** (thông qua phương pháp ngoại suy), cho phép xử lý toàn bộ kho tài liệu kỹ thuật hoặc lịch sử hội thoại dài trong một lần gọi.

---

## 3. Truy cập và Tích hợp

Người dùng có thể trải nghiệm ngay các mô hình Qwen thông qua **Amazon Bedrock Console** tại khu vực Chat/Text Playground hoặc tích hợp vào ứng dụng thông qua AWS SDK.

Đặc biệt, Amazon Bedrock đã đơn giản hóa quy trình truy cập mô hình. Quản trị viên tài khoản có toàn quyền kiểm soát việc bật/tắt quyền truy cập thông qua **AWS IAM policies** và **Service Control Policies (SCPs)**, đảm bảo tuân thủ bảo mật doanh nghiệp mà không cần kích hoạt thủ công từng mô hình.

---

## Kết luận

Sự bổ sung Qwen3 vào Amazon Bedrock mang lại sức mạnh to lớn cho các kỹ sư phần mềm và nhà khoa học dữ liệu. Với khả năng xử lý mã nguồn vượt trội và kiến trúc linh hoạt, doanh nghiệp có thể:
* Tự động hóa quy trình phân tích và refactor mã nguồn.
* Xây dựng các trợ lý AI thông minh với chi phí suy luận tối ưu.
* Triển khai đa dạng từ đám mây xuống thiết bị biên mà không thay đổi nền tảng quản lý.

---

## Tác giả

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/DaniloPoccia.png" alt="Danilo Poccia" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Danilo Poccia</h3>
    <p>Danilo là Chief Evangelist (EMEA) tại Amazon Web Services. Ông làm việc với các công ty khởi nghiệp và doanh nghiệp ở mọi quy mô để hỗ trợ đổi mới sáng tạo. Ông tận dụng kinh nghiệm của mình để giúp mọi người hiện thực hóa ý tưởng, tập trung vào kiến trúc serverless, lập trình hướng sự kiện (event-driven), và tác động kinh doanh của máy học (Machine Learning) cũng như điện toán biên (Edge Computing). Ông cũng là tác giả cuốn sách "AWS Lambda in Action" của nhà xuất bản Manning.</p>
  </div>
</div>