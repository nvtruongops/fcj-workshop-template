---
title: "Worklog Tuần 9"
date: 2025-11-03
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9
* Củng cố kiến thức nền tảng backend: thiết kế API, xác thực, cấu trúc dữ liệu.
* Làm quen với các dịch vụ backend quan trọng trên AWS: **API Gateway, Lambda, DynamoDB, Cognito**.
* Xây dựng phiên bản đầu tiên của kiến trúc backend dự án.
* Thực hành phát triển API an toàn và xử lý request/response đúng chuẩn.

---

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|-----|-----------|--------------|------------------|--------------------|
| 2 | Học thiết kế REST API, phương thức HTTP và quy trình backend. <br>- Thiết kế các API endpoints ban đầu. | 03/11/2025 | 03/11/2025 | AWS Study Group |
| 3 | Học **API Gateway**. <br>- Tạo routes, methods, validate request và enable logging. | 04/11/2025 | 04/11/2025 | AWS Study Group |
| 4 | Tìm hiểu **AWS Lambda**. <br>- Viết và deploy các hàm CRUD cơ bản. <br>- Test bằng event triggers. | 05/11/2025 | 05/11/2025 | AWS Study Group |
| 5 | Học **DynamoDB**. <br>- Thiết kế schema. <br>- Thực hiện CRUD operations. | 06/11/2025 | 06/11/2025 | AWS Study Group |
| 6 | Học **Cognito Authentication**. <br>- Tạo User Pool. <br>- Kết nối Cognito với API Gateway. | 07/11/2025 | 07/11/2025 | AWS Study Group |

---

### Kết quả đạt được trong tuần 9

#### 1. Nắm vững kiến trúc backend hiện đại
Tôi hiểu rõ hơn về cách tổ chức backend theo kiến trúc nhiều lớp, cách tách biệt logic xử lý và cách API giao tiếp với dịch vụ xác thực và cơ sở dữ liệu. Điều này giúp tôi tự tin hơn khi xây dựng hệ thống quy mô lớn.

#### 2. Thiết kế tài liệu API rõ ràng và đầy đủ
Tôi đã hoàn thiện bản API specification đầu tiên, bao gồm routes, body, response, error codes và mô tả logic. Đây là bước cực kỳ quan trọng giúp đồng bộ giữa backend và frontend.

#### 3. Cấu hình thành công API Gateway
Tôi đã tạo các API routes, cấu hình CORS, logging và kết nối đến Lambda. Việc này giúp tôi hiểu rõ cách request từ client được chuyển đến backend trong mô hình serverless.

#### 4. Xây dựng Lambda Functions xử lý logic
Tôi đã viết các hàm Lambda đầu tiên, học cách quản lý input/output và triển khai logic backend mà không cần server. Điều này giúp tôi làm quen với mô hình serverless hiệu quả và tiết kiệm chi phí.

#### 5. Thiết kế và vận hành DynamoDB
Tôi đã tạo bảng, chọn partition/sort key hợp lý và thực hành các thao tác CRUD. Tôi hiểu rõ hơn về mô hình dữ liệu NoSQL và cách tối ưu truy vấn.

#### 6. Tích hợp xác thực bằng Cognito
Tôi đã tạo User Pool, đăng ký người dùng và tích hợp xác thực vào API Gateway. Điều này giúp backend trở nên an toàn hơn và tuân thủ quy trình bảo mật tiêu chuẩn.

---

