---
title: "Worklog Tuần 10"
date: 2025-11-10
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10
* Nâng cao kiến thức bảo mật backend và quy trình xác thực trong môi trường cloud.
* Hoàn thiện tích hợp Cognito và áp dụng mô hình authentication dựa trên token.
* Áp dụng validation, access pattern và secondary index cho DynamoDB.
* Tìm hiểu về authorizer, throttling và rate-limit trên API Gateway.
* Bắt đầu kết nối các thành phần backend thành quy trình hoàn chỉnh.

---

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|-----|-----------|--------------|------------------|--------------------|
| 2 | Học về authentication/authorization (JWT, Access Token, ID Token). <br>- Ôn OWASP API Security. | 10/11/2025 | 10/11/2025 | AWS Study Group |
| 3 | Tìm hiểu sâu **Cognito User Pool** và **Identity Pool**. <br>- Triển khai signup, login, refresh token flow. | 11/11/2025 | 11/11/2025 | AWS Study Group |
| 4 | Tạo **Lambda Authorizer** và gắn vào API Gateway. <br>- Thêm IAM permission. | 12/11/2025 | 12/11/2025 | AWS Study Group |
| 5 | Nâng cấp DynamoDB: <br>- Thêm secondary index. <br>- Tăng cường validation & error handling. | 13/11/2025 | 13/11/2025 | AWS Study Group |
| 6 | Kiểm thử end-to-end: <br>- Client → API Gateway → Lambda → DynamoDB. | 14/11/2025 | 14/11/2025 | AWS Study Group |

---

### Kết quả đạt được trong tuần 10

#### **1. Hiểu rõ hơn về bảo mật và quy trình xác thực**
Tôi đã tìm hiểu sâu về cơ chế hoạt động của JWT, sự khác nhau giữa Access/ID/Refresh Token và các phương pháp phòng chống tấn công trong API security. Điều này giúp backend an toàn hơn và tuân thủ chuẩn OWASP.

#### **2. Hoàn thiện quy trình xác thực bằng Cognito**
Tôi đã cấu hình signup, login, xác thực đa yếu tố (nếu cần), và refresh token trong Cognito. Các flow này được test bằng Postman và CLI để đảm bảo backend xử lý chính xác danh tính người dùng.

#### **3. Tích hợp Lambda Authorizer để bảo vệ API**
Tôi đã tạo Authorizer tùy chỉnh để xác minh token trước khi API được gọi. Nhờ đó, hệ thống có thể phân quyền chính xác và bảo vệ các endpoint quan trọng khỏi truy cập trái phép.

#### **4. Tối ưu DynamoDB bằng Secondary Index**
Tôi thêm GSI vào bảng dữ liệu, cải thiện đáng kể tốc độ truy vấn và hỗ trợ các kiểu truy vấn phức tạp hơn. Đồng thời, tôi bổ sung logic validation để dữ liệu luôn chính xác và nhất quán.

#### **5. Hoàn thiện quy trình backend đầu–cuối**
Tôi đã kiểm thử toàn bộ luồng:  
Client gửi request → API Gateway xử lý → Lambda thực thi → DynamoDB lưu/truy xuất → trả về kết quả.  
Luồng hoạt động mượt mà và chính xác.

#### **6. Cải thiện cấu trúc code và khả năng bảo trì**
Tôi đã refactor lại code của Lambda theo mô hình rõ ràng hơn, tách riêng logic, validation và truy cập database. Điều này giúp backend phát triển dễ dàng và hạn chế lỗi trong tương lai.

---
