---
title: "Blog 3"
date: 2025-09-26
chapter: false
pre: " <b> 3.3. </b> "
---

# Mở rộng quy mô Trình quản lý Cụm và API Quản trị trong Amazon OpenSearch Service

Amazon OpenSearch Service là dịch vụ được quản lý giúp dễ dàng triển khai, bảo mật và vận hành các cụm OpenSearch ở quy mô lớn trên đám mây AWS. Một cụm OpenSearch điển hình bao gồm các nút quản lý cụm (cluster manager), nút dữ liệu (data nodes) và nút điều phối (coordinator nodes). Khuyến nghị nên có ba nút quản lý cụm, trong đó một nút được bầu làm nút lãnh đạo (leader node).

---

## Giới thiệu

Với phiên bản **OpenSearch Service 2.17**, Amazon OpenSearch Service hiện hỗ trợ các cụm lên đến **1.000 nút**, có thể xử lý **500.000 shard**.  
Đối với các cụm lớn, nhiều điểm nghẽn trong việc tương tác API quản trị với nút leader đã được xác định và tối ưu hóa trong phiên bản này.

Những cải tiến này giúp OpenSearch Service duy trì tần suất giám sát ổn định cho các cụm lớn trong khi vẫn sử dụng tài nguyên tối ưu (dưới 10% CPU và dưới 75% JVM trên nút leader có CPU 16 nhân và heap JVM 64 GB).  
Ngoài ra, các hoạt động quản lý siêu dữ liệu cũng có thể được thực hiện với độ trễ có thể dự đoán mà không làm mất ổn định nút leader.

---

## Tổng quan về Trạng thái Cụm (Cluster State)

Để hiểu rõ các điểm nghẽn liên quan đến trình quản lý cụm, trước tiên hãy xem xét **cluster state**, yếu tố cốt lõi trong hoạt động của nút leader. Trạng thái cụm chứa các siêu dữ liệu sau:

- Cấu hình cụm  
- Siêu dữ liệu của chỉ mục (index settings, mappings, aliases)  
- Bảng định tuyến và siêu dữ liệu shard (phân bổ shard)  
- Thông tin và thuộc tính của các nút  
- Thông tin snapshot và siêu dữ liệu tùy chỉnh  

Ví dụ, một cụm có 6 nút (3 nút quản lý và 3 nút dữ liệu), 1 chỉ mục và 3 shard sẽ có kích thước trạng thái cụm khoảng **15 KB**.  
Tuy nhiên, một cụm có **1.000 nút**, **10.000 chỉ mục** và **50 shard mỗi chỉ mục** có thể có kích thước trạng thái cụm lên đến **250 MB**.

---

## Điểm nghẽn 1: Giao tiếp Trạng thái Cụm

Các API quản trị như `_cat`, `_cluster` và `_nodes` dựa vào nút leader để lấy trạng thái cụm mới nhất.  
Trong các cụm rất lớn, những yêu cầu thường xuyên này có thể làm quá tải nút leader, dẫn đến:

- Tăng mức sử dụng CPU do nhiều yêu cầu API đồng thời  
- Áp lực bộ nhớ heap cao do tuần tự hóa lặp lại của trạng thái cụm lớn  
- Độ trễ và timeout do luồng truyền tải (transport threads) bị chiếm dụng  
- Bỏ lỡ các phản hồi heartbeat giữa các nút, kích hoạt quá trình khôi phục không cần thiết  

### Giải pháp: Sử dụng Trạng thái Cục bộ (Local Cluster State)

Mỗi nút hiện lưu trữ phiên bản gần nhất của cluster state.  
Bằng cách giới thiệu **phiên bản hóa (term và version numbers)**, nút điều phối có thể kiểm tra xem trạng thái cục bộ của nó có trùng với nút leader không trước khi gửi yêu cầu.  
Nếu trạng thái cục bộ đã cập nhật, nút đó sẽ phục vụ yêu cầu API ngay tại chỗ, **không cần truy vấn leader**.

→ Giải pháp này giảm tải cho nút leader và tăng độ ổn định cụm.

**Tác động:**  
Các bài kiểm thử tải cho thấy mức sử dụng CPU trên nút leader giảm tới **50%** mà không tăng độ trễ API, đối với cụm có **từ 25.000 shard trở lên**.

---

## Điểm nghẽn 2: Bản chất Phân tán (Scatter-Gather) của API Thống kê

Các API thống kê như `_cat/indices`, `_cat/shards`, `_cluster/stats` và `_nodes/stats` yêu cầu thu thập dữ liệu từ nhiều nút dữ liệu và tổng hợp trên nút điều phối.  

Trong cụm có **500.000 shard**, tổng kích thước phản hồi có thể đạt tới **2,5 GB**, gây ra:

- Lưu lượng mạng cao  
- Áp lực bộ nhớ trên các nút điều phối  
- Kích hoạt circuit breaker và lỗi “429 Too Many Requests”  
- Tăng sử dụng CPU do thu gom rác (GC) thường xuyên  

### Giải pháp: Tổng hợp & Lọc Cục bộ (Local Aggregation and Filtering)

Thay vì trả về toàn bộ thống kê ở cấp độ shard, mỗi nút dữ liệu nay **tổng hợp cục bộ** và chỉ gửi **các số liệu được yêu cầu** (ví dụ: số lượng tài liệu hoặc kích thước lưu trữ) cho nút điều phối.

→ Giải pháp này giảm đáng kể chi phí tính toán và bộ nhớ trên các nút điều phối.

**Tác động:**

| API | Độ trễ trước khi tối ưu | Độ trễ sau tối ưu |
|------|--------------------------|-------------------|
| _cluster/stats | 15s | 0.65s |
| _nodes/stats | 13.74s | 1.69s |
| _cluster/health | 0.56s | 0.15s |

---

## Điểm nghẽn 3: Các Yêu cầu Thống kê Chạy Lâu

Trước đây, ngay cả khi người dùng **hủy yêu cầu** hoặc **hết thời gian chờ**, nút điều phối vẫn tiếp tục xử lý và gửi yêu cầu nội bộ đến các nút dữ liệu. Điều này gây lãng phí tài nguyên tính toán và mạng.

### Giải pháp: Hủy ở Tầng Truyền tải (Transport Layer)

Nút điều phối hiện sẽ **hủy các yêu cầu con (downstream requests)** ngay khi hết thời gian chờ của client.  
Điều này giúp tránh tải không cần thiết trên các nút dữ liệu và giúp cụm xử lý lỗi một cách ổn định, tránh lan truyền.

---

## Điểm nghẽn 4: Kích thước Phản hồi Lớn

Trước đây, các API phổ biến như `_cat` **không hỗ trợ phân trang**, vì lượng dữ liệu nhỏ khi chúng được thiết kế.  
Trong các cụm hiện đại, các phản hồi không giới hạn này có thể làm quá tải các nút điều phối.

### Giải pháp: API Danh sách Có Phân trang (Paginated List APIs)

Các API danh sách mới như `_list/indices` và `_list/shards` được giới thiệu như các lựa chọn thay thế **ổn định và có khả năng mở rộng**.  
Chúng duy trì tính nhất quán khi phân trang bằng cách kết hợp **dấu thời gian tạo chỉ mục** và **tên chỉ mục**, ngay cả khi chỉ mục được thêm hoặc xóa.

**Tác động:**  
Các API có phân trang cung cấp phản hồi **ổn định, có giới hạn**, cho các cụm lên đến **500.000 shard**, đảm bảo thời gian phản hồi nhanh và giảm sử dụng tài nguyên.

---

## Kết luận

Các API quản trị đóng vai trò quan trọng trong việc **giám sát và quản lý siêu dữ liệu** trong Amazon OpenSearch Service.  
Nếu không được tối ưu, chúng có thể trở thành **điểm nghẽn hiệu năng** trong các cụm lớn.

Các cải tiến trong **phiên bản 2.17** mang lại **hiệu suất đáng kể** cho các cụm ở mọi quy mô — nhỏ (20 nút), trung bình (200 nút) và lớn (1.000 nút).  
Những tối ưu này giúp **nút leader luôn ổn định** ngay cả trong điều kiện tải metadata và API cao.

Các tính năng như **phân trang, hủy yêu cầu và tổng hợp cục bộ** có thể mở rộng và sẽ được áp dụng trong các cải tiến API trong tương lai.

---

Để biết thêm chi tiết, xem bài viết gốc trên AWS Big Data Blog:  
[Scaling cluster manager and admin APIs in Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/scaling-cluster-manager-and-admin-apis-in-amazon-opensearch-service/)
