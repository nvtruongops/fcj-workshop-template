---
title: "Blog 3: Quản lý Chính sách Tường lửa AWS Tập trung với AlgoSec"
date: 2025-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Tổng quan: Thách thức của Môi trường Hybrid

Khi các doanh nghiệp di chuyển workloads từ on-premise lên AWS, việc duy trì một chính sách bảo mật nhất quán trở thành thách thức lớn. **AWS Firewall Manager** giúp quản lý tập trung các quy tắc bảo mật trên cloud, nhưng các tổ chức thường gặp khó khăn trong việc đồng bộ hóa các quy tắc này với hạ tầng firewall truyền thống (on-premise).

Giải pháp mới từ **AlgoSec** (có sẵn trên AWS Marketplace) tích hợp trực tiếp với AWS Firewall Manager, mang lại khả năng hiển thị (visibility) và quản lý thống nhất cho toàn bộ môi trường Hybrid Cloud.

---

## 1. Giới thiệu AlgoSec Horizon và ACE

Giải pháp của AlgoSec bao gồm hai thành phần chính được tích hợp:
1.  **AlgoSec Horizon:** Tập trung vào việc hiển thị, ánh xạ ứng dụng và cung cấp ngữ cảnh kinh doanh cho các luồng mạng.
2.  **AlgoSec Cloud Enterprise (ACE):** Quản lý chính sách bảo mật, phân tích rủi ro và tự động hóa thay đổi.

Lợi ích khi triển khai qua AWS Marketplace:
* **Mua sắm đơn giản:** Tích hợp trực tiếp vào hóa đơn AWS (Unified billing).
* **Triển khai nhanh:** Onboarding được sắp xếp hợp lý để có kết quả ngay lập tức (Time-to-value).
* **Mở rộng:** Dễ dàng mở rộng quyền kiểm soát chính sách trên mọi môi trường.

---

## 2. Hiển thị Chính sách Firewall Hợp nhất

Sau khi đăng nhập vào ACE console, tab **AWS Firewall Policies** sẽ hiển thị toàn bộ các tường lửa AWS đã triển khai cùng các chính sách liên quan. Người dùng có thể xem chi tiết từng firewall, bao gồm VPC, Availability Zones, và Subnets.

![The main account page shows the firewalls, the policy sets, and the rules](/images/3-BlogsTranslated/3.3.1.png)

> [Figure 1] Trang chính hiển thị danh sách các Firewall, bộ chính sách (Policy Sets) và các quy tắc (Rules).

Khả năng hiển thị ngữ cảnh là điểm mạnh của giải pháp. Khi di chuột qua tên một firewall, hệ thống sẽ hiển thị ngay ngữ cảnh VPC và các ứng dụng liên quan.

![VPC context shown while hovering over the firewall name test-firewall-2](/images/3-BlogsTranslated/3.3.2.png)

> [Figure 2] Ngữ cảnh VPC hiển thị khi di chuột qua tên Firewall, giúp người quản trị hiểu ngay vị trí và phạm vi ảnh hưởng.

---

## 3. Phân tích Chi tiết Rule và Ứng dụng

Các chính sách AWS Firewall thường bao gồm hai nhóm quy tắc: **Stateless** (không trạng thái) và **Stateful** (có trạng thái). AlgoSec cho phép mở rộng (expand) để xem chi tiết từng nhóm quy tắc này.

![The expanded view of a policy set shows the existing rule groups](/images/3-BlogsTranslated/3.3.3.png)

> [Figure 3] Chế độ xem mở rộng hiển thị chi tiết các nhóm quy tắc Stateless và Stateful bên trong một bộ chính sách.

Quan trọng hơn, AlgoSec Horizon liên kết các thông tin kỹ thuật này với **ngữ cảnh kinh doanh**. Người dùng có thể xem sơ đồ ứng dụng (Application Diagram), chủ sở hữu ứng dụng (Application Owners), và trạng thái kết nối từ on-premise lên cloud.

![Application details include status, contact details, and diagram](/images/3-BlogsTranslated/3.3.4.png)

> [Figure 4] Chế độ xem ứng dụng (Application View) hiển thị sơ đồ kết nối, thông tin liên hệ và trạng thái tuân thủ của ứng dụng kinh doanh.

---

## Kết luận

Sự kết hợp giữa AWS Firewall Manager và AlgoSec mang lại một "tầm nhìn duy nhất" (single pane of glass) cho đội ngũ bảo mật. Thay vì quản lý rời rạc, giờ đây họ có thể:
* Phân tích cấu hình AWS giống như firewall truyền thống.
* Mô phỏng thay đổi (Change simulation) trước khi áp dụng để tránh gián đoạn kinh doanh.
* Đảm bảo tuân thủ (Compliance) liên tục trên toàn bộ môi trường Hybrid.

---

## Tác giả

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/JosephHallman.png" alt="Joseph Hallman" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Joseph Hallman</h3>
    <p>Joseph is a Product Marketing Manager at AlgoSec with over 20 years of experience in networking and data center markets. He has worked at both startups and global enterprises, bringing expertise in sales, product management, and marketing strategy.</p>
  </div>
</div>

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/AmitGaur.png" alt="Amit Gaur" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Amit Gaur</h3>
    <p>Amit is a Cloud Infrastructure Architect at AWS, specializing in network architecture design. He helps customers build highly scalable and resilient environments on AWS.</p>
  </div>
</div>

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/SrivalsanMannoorSudhagar.png" alt="Srivalsan Mannoor Sudhagar" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Srivalsan Mannoor Sudhagar</h3>
    <p>Srivalsan is a Sr. Cloud Infrastructure Architect at Amazon Web Services, Professional Services who brings expertise in Cloud Infrastructure and MLOps platforms.</p>
  </div>
</div>

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/SaleemMuhammad.png" alt="Saleem Muhammad" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Saleem Muhammad</h3>
    <p>Saleem is a Senior Manager of Product Management in AWS Network & Application Protection. He is passionate about building solutions that help customers to secure mission critical workloads.</p>
  </div>
</div>