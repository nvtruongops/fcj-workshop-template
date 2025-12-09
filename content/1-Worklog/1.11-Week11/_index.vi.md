---
title: "Worklog Tuần 11"
date: 2025-11-17
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11
* Bắt đầu kết nối frontend với các API backend có xác thực.
* Tăng cường cơ chế xử lý lỗi cho API Gateway và Lambda.
* Triển khai lưu trữ token an toàn và logic refresh token.
* Xây dựng API client tái sử dụng (Axios/Fetch) với interceptors.
* Cải thiện logging, debugging và phân tích CloudWatch.
* Hoàn thiện luồng full-stack từ UI → Gateway → Lambda → Database.

---

### Các công việc thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|-----|-----------|---------------|------------------|---------------------|
| 2 | Kết nối frontend đến các endpoint Cognito. <br>- Xây UI đăng nhập và đăng ký. | 17/11/2025 | 17/11/2025 | AWS Study Group |
| 3 | Triển khai lưu token an toàn & cơ chế refresh token. <br>- Thêm interceptors cho API client. | 18/11/2025 | 18/11/2025 | AWS Study Group |
| 4 | Tích hợp frontend với các route API Gateway có bảo vệ. <br>- Kiểm tra luồng Lambda Authorizer. | 19/11/2025 | 19/11/2025 | AWS Study Group |
| 5 | Xây hệ thống xử lý lỗi toàn cục (frontend + backend). <br>- Chuẩn hóa response API. | 20/11/2025 | 20/11/2025 | AWS Study Group |
| 6 | Tăng cường giám sát bằng CloudWatch Logs & Metrics. <br>- Debug toàn bộ luồng end-to-end. | 21/11/2025 | 21/11/2025 | AWS Study Group |

---

### Kết quả đạt được trong tuần 11

#### **1. Hoàn tất kết nối Frontend với Cognito**
Tôi đã kết nối thành công frontend với Cognito, cho phép người dùng đăng ký, đăng nhập và đăng xuất ngay từ giao diện. Mọi thao tác lấy token và quản lý session đều hoạt động ổn định.

#### **2. Triển khai lưu token an toàn & tự động refresh**
Tôi sử dụng bộ nhớ tạm của trình duyệt để lưu token an toàn và xây dựng cơ chế refresh token tự động. Tôi cũng thêm interceptors để tự động đính kèm access token vào mỗi request.

#### **3. Kết nối frontend với API backend được bảo vệ**
Frontend hiện có thể gọi các route API Gateway có yêu cầu xác thực JWT, giúp xác nhận trọn vẹn luồng bảo mật từ UI → Client → Gateway → Lambda → DynamoDB.

#### **4. Xây dựng hệ thống xử lý lỗi thống nhất**
Tôi chuẩn hóa lỗi backend trong Lambda, config API Gateway mapping, và tạo một global error handler trên frontend để hiển thị thông báo rõ ràng hơn. Điều này cải thiện UX và giảm thời gian debug.

#### **5. Cải thiện logging & monitoring bằng CloudWatch**
Tôi phân tích log Lambda, phát hiện bottleneck, và tạo các bộ lọc CloudWatch Metrics để theo dõi tỷ lệ lỗi và độ trễ. Workflow debug nhanh hơn và hiệu quả hơn.

#### **6. Hoàn thành luồng Full-Stack Authentication + Data**
Cuối tuần, tôi đã chạy thành công trọn vẹn luồng full-stack:
UI → Xác thực → Gửi request → Lambda → DynamoDB → Trả kết quả.  
Mọi thứ hoạt động mượt mà với đầy đủ bảo mật và xử lý lỗi.

---
