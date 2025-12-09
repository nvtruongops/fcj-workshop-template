---
title: "Đề xuất dự án"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS First Cloud AI Journey – Kế Hoạch Dự Án

**Hello World – Đại học FPT – EveryoneCook**

**Ngày:** 30/11/2025

📥 **[Tải Đề Xuất Đầy Đủ (DOCX)](Proposal.docx)**

---

# MỤC LỤC

**1. [BỐI CẢNH VÀ ĐỘNG LỰC](#boi-canh-va-dong-luc)**

&nbsp;&nbsp;&nbsp;&nbsp;1.1 [Tóm tắt điều hành](#tom-tat-dieu-hanh)

&nbsp;&nbsp;&nbsp;&nbsp;1.2 [TIÊU CHÍ THÀNH CÔNG DỰ ÁN](#tieu-chi-thanh-cong)

&nbsp;&nbsp;&nbsp;&nbsp;1.3 [Giả định](#gia-dinh)

**2. [KIẾN TRÚC GIẢI PHÁP / SƠ ĐỒ KIẾN TRÚC](#kien-truc-giai-phap)**

&nbsp;&nbsp;&nbsp;&nbsp;2.1 [Sơ đồ Kiến trúc Kỹ thuật](#so-do-kien-truc)

&nbsp;&nbsp;&nbsp;&nbsp;2.2 [Kế hoạch Kỹ thuật](#ke-hoach-ky-thuat)

&nbsp;&nbsp;&nbsp;&nbsp;2.3 [Kế hoạch Dự án](#ke-hoach-du-an)

&nbsp;&nbsp;&nbsp;&nbsp;2.4 [Cân nhắc Bảo mật](#can-nhac-bao-mat)

**3. [HOẠT ĐỘNG VÀ SẢN PHẨM BÀN GIAO](#hoat-dong-va-san-pham)**

&nbsp;&nbsp;&nbsp;&nbsp;3.1 [Hoạt động và sản phẩm bàn giao](#hoat-dong-chi-tiet)

&nbsp;&nbsp;&nbsp;&nbsp;3.2 [NGOÀI PHẠM VI](#ngoai-pham-vi)

&nbsp;&nbsp;&nbsp;&nbsp;3.3 [LỘ TRÌNH ĐƯA VÀO SẢN XUẤT](#lo-trinh-san-xuat)

**4. [DỰ TOÁN CHI PHÍ AWS THEO DỊCH VỤ](#du-toan-chi-phi)**

**5. [NHÓM DỰ ÁN](#nhom-du-an)**

**6. [NGUỒN LỰC & ƯỚC TÍNH CHI PHÍ](#nguon-luc-chi-phi)**

**7. [CHẤP THUẬN](#chap-thuan)**

---

# 1. BỐI CẢNH VÀ ĐỘNG LỰC

## 1.1 Tóm tắt điều hành

**Thông tin khách hàng**

Khách hàng là một startup tập trung xây dựng nền tảng mạng xã hội hiện đại, nơi người dùng có thể chia sẻ công thức nấu ăn, tải lên hình ảnh món ăn, trao đổi kinh nghiệm ẩm thực và khám phá các món ăn được AI gợi ý. Tổ chức hướng đến việc cung cấp một nền tảng tương tác cao, có khả năng phục vụ lượng người dùng lớn và ngày càng tăng.

**Mục tiêu kinh doanh và kỹ thuật -- động lực chuyển sang AWS cloud**

- Cho phép phát triển và triển khai nhanh chóng sử dụng các dịch vụ quản lý của AWS
- Đảm bảo khả năng mở rộng cao khi lượng người dùng và lưu trữ media tăng
- Cung cấp môi trường đáng tin cậy, độ trễ thấp cho tính toán AI và phân phối nội dung
- Giảm chi phí hạ tầng ban đầu và chuyển sang mô hình pay-as-you-go
- Cải thiện bảo mật dữ liệu, sao lưu và tuân thủ thông qua các khả năng native của AWS

**Các trường hợp sử dụng**

- Người dùng tải lên công thức, hình ảnh và video nấu ăn lên nền tảng
- Hệ thống gợi ý món ăn sử dụng AI dựa trên nguyên liệu có sẵn do người dùng cung cấp
- Người dùng tương tác xã hội thông qua thích, bình luận, chia sẻ và theo dõi
- AI xử lý văn bản và hình ảnh để tạo gợi ý công thức
- Quản trị viên quản lý nội dung, giám sát hoạt động nền tảng và theo dõi phân tích hiệu suất

Để đáp ứng mục tiêu của khách hàng trong việc xây dựng nền tảng nấu ăn xã hội có khả năng mở rộng với gợi ý công thức được hỗ trợ bởi AI, đối tác sẽ cung cấp triển khai cloud end-to-end đầy đủ trên AWS. Các dịch vụ cung cấp bao gồm:

- **Thiết kế Kiến trúc Cloud:** Định nghĩa kiến trúc serverless an toàn, có khả năng mở rộng cao sử dụng các phương pháp hay nhất của AWS (Route 53, API Gateway, Lambda, DynamoDB, S3, CloudFront, Cognito)
- **Tích hợp AI:** Triển khai AWS Bedrock (Claude 3.5 Sonnet) cho gợi ý công thức thông minh, phân tích hình ảnh và các tính năng xử lý ngôn ngữ tự nhiên
- **Triển khai Hạ tầng:** Xây dựng và triển khai tất cả backend, frontend, xác thực và lớp dữ liệu sử dụng Infrastructure as Code (IaC) với CI/CD pipelines tự động hoàn toàn
- **Bảo mật & Tuân thủ:** Cấu hình IAM roles, mã hóa (KMS), WAF, logging, monitoring và các guardrails tuân thủ để đảm bảo bảo mật nền tảng
- **Thiết lập Observability:** Bật CloudWatch dashboards, alarms, X-Ray tracing và tập trung log cho monitoring và insights hiệu suất thời gian thực
- **DevOps & Tự động hóa:** Triển khai quy trình build/deploy tự động qua GitLab + Amplify, operational pipelines và cấu hình auto-scaling
- **Tối ưu Hiệu suất:** Cấu hình CDN caching, DynamoDB capacity scaling, search indexing và xử lý background bất đồng bộ dựa trên SQS
- **Chuyển giao Kiến thức & Tài liệu:** Cung cấp tài liệu kỹ thuật, phương pháp hay nhất, hướng dẫn kiến trúc và đào tạo bàn giao cho đội ngũ kỹ thuật của khách hàng

## 1.2 TIÊU CHÍ THÀNH CÔNG DỰ ÁN

- Khả dụng hệ thống ≥ 99.9% uptime trên tất cả dịch vụ production (API Gateway, Lambda, DynamoDB, CloudFront)
- Thời gian tải trang < 2.5 giây cho giao diện người dùng chính được phân phối qua CloudFront và Amplify
- Thời gian phản hồi API < 300 ms cho 90% tất cả các yêu cầu API hướng người dùng trong điều kiện traffic bình thường
- Độ trễ xử lý AI < 5 giây cho gợi ý công thức được tạo bởi AWS Bedrock
- Tỷ lệ thành công xác thực người dùng ≥ 98% với Cognito xử lý đăng ký, đăng nhập và xác minh email
- Không có lỗ hổng bảo mật nghiêm trọng sau khi review bảo mật và triển khai WAF rules
- Độ bền dữ liệu 99.999999999% (11 nines) được đảm bảo thông qua S3 object storage và DynamoDB
- Khả năng mở rộng hỗ trợ 10,000+ người dùng đồng thời mà không giảm hiệu suất nhờ hạ tầng serverless
- Kiểm soát chi phí vận hành trong ngân sách mục tiêu: sử dụng AWS hàng tháng không vượt quá $200 cho production
- Tỷ lệ thành công tải lên & xử lý hình ảnh ≥ 99%, được hỗ trợ bởi S3, Lambda Workers và SQS
- Hiệu suất tìm kiếm dưới 1 giây (nếu OpenSearch được bật) cho các truy vấn tìm kiếm công thức/nội dung
- Phạm vi monitoring 100% dịch vụ quan trọng sử dụng CloudWatch dashboards, alarms và X-Ray tracing
- Thời gian triển khai CI/CD < 5 phút qua GitHub → Amplify và IaC automation
- Không có sự kiện mất dữ liệu, được đảm bảo bởi DynamoDB PITR và S3 versioning

## 1.3 Giả định

- Khách hàng sẽ cung cấp quyền truy cập đầy đủ vào domain registrar (Hostinger) để cấu hình DNS delegation sang Route 53
- Khách hàng sẽ cung cấp quyền truy cập AWS account hợp lệ với quyền Administrator cho triển khai và cấu hình
- Tất cả dịch vụ AWS cần thiết (Amplify, API Gateway, Lambda, DynamoDB, CloudFront, Cognito, Bedrock, SES) đều khả dụng và được hỗ trợ trong region đã chọn
- SES sẽ được chuyển thành công ra khỏi sandbox và được phê duyệt cho gửi email production
- Các tích hợp bên thứ ba (GitHub cho CI/CD, email clients bên ngoài, nguồn tải lên hình ảnh) sẽ vẫn khả dụng và ổn định
- Đội ngũ phát triển sẽ duy trì chất lượng source code và tuân theo các hướng dẫn kiến trúc do đối tác cung cấp
- Khách hàng sẽ cung cấp phản hồi và phê duyệt kịp thời trong các giai đoạn thiết kế, testing và triển khai

**Phụ thuộc**

- Kết nối internet đáng tin cậy được yêu cầu cho tất cả người dùng truy cập ứng dụng web và APIs
- Hệ thống phụ thuộc vào AWS Bedrock (Claude 3.5 Sonnet) cho tạo công thức AI và có thể gặp biến động hiệu suất nếu model bị rate-limited
- Quy trình tải lên và xử lý hình ảnh phụ thuộc vào độ tin cậy xử lý của S3, Lambda và SQS
- Nếu OpenSearch được bật, các tính năng tìm kiếm phụ thuộc vào khả dụng của OpenSearch domain
- GitHub Actions và Amplify phụ thuộc vào khả dụng dịch vụ GitHub

**Ràng buộc**

- Dự án sẽ được triển khai hoàn toàn trong một AWS region duy nhất, có thể ảnh hưởng đến độ trễ cho người dùng ngoài region
- Giải pháp được thiết kế sử dụng các pattern serverless; các workload dựa trên EC2 tùy chỉnh nằm ngoài phạm vi dự án
- Uy tín domain SES có thể ảnh hưởng đến khả năng gửi email trong những tuần đầu
- OpenSearch được triển khai dưới dạng cluster single-node để tiết kiệm chi phí, nghĩa là không có high availability cho search indexing
- Hệ thống phải nằm trong mục tiêu chi phí của khách hàng (< $200/tháng), hạn chế việc sử dụng tài nguyên compute lớn

**Rủi ro**

- Phê duyệt SES production có thể bị trì hoãn, ảnh hưởng đến email onboarding và thông báo người dùng
- Nếu traffic tăng bất ngờ, DynamoDB provisioned capacity có thể bị throttle mà không có điều chỉnh scaling kịp thời
- Thay đổi chi phí hoặc độ trễ AI model bởi AWS có thể ảnh hưởng đến hiệu suất ứng dụng hoặc kiểm soát chi phí
- Cấu hình CloudFront caching sai có thể dẫn đến độ trễ cao hơn hoặc tăng chi phí data transfer
- Bất kỳ cấu hình IAM không chính xác nào có thể dẫn đến rủi ro bảo mật hoặc gián đoạn dịch vụ
- Thay đổi nhân sự đội ngũ khách hàng hoặc thiếu kỹ năng DevOps có thể làm chậm bảo trì hoặc triển khai trong tương lai


---

# 2. KIẾN TRÚC GIẢI PHÁP / SƠ ĐỒ KIẾN TRÚC

## 2.1 Sơ đồ Kiến trúc Kỹ thuật

Kiến trúc giải pháp đề xuất cho nền tảng mạng xã hội nấu ăn được hỗ trợ bởi AI được thiết kế sử dụng stack cloud-native AWS hoàn toàn serverless và có khả năng mở rộng. Kiến trúc đảm bảo high availability, bảo mật và tích hợp liền mạch giữa web frontend, backend APIs, xác thực, lưu trữ dữ liệu và dịch vụ gợi ý AI.

Dưới đây là mô tả các thành phần chính và cách dữ liệu chảy qua hệ thống:

**1. Lớp Network & Edge**

- **Amazon Route 53:** Cung cấp DNS routing cho custom domain được sử dụng bởi nền tảng. Các yêu cầu HTTPS đến từ người dùng được resolve và chuyển tiếp đến CloudFront
- **Amazon CloudFront:** Hoạt động như CDN toàn cầu phân phối nội dung frontend với độ trễ thấp trong khi cache các file tĩnh
- **AWS WAF:** Bảo vệ ứng dụng khỏi các khai thác web phổ biến như SQL injection, XSS và bot attacks

**2. Frontend Hosting & Deployment**

- **AWS Amplify Hosting:** Host và triển khai ứng dụng frontend Next.js, tích hợp với GitLab CI/CD cho triển khai tự động từ quy trình phát triển

**3. Lớp Application**

- **Amazon Cognito:** Xử lý xác thực và ủy quyền người dùng, hỗ trợ email/password và social logins
- **Amazon API Gateway:** Phục vụ như điểm vào chính cho backend APIs, expose các REST endpoints được sử dụng bởi frontend
- **AWS Lambda:** Chứa business logic backend, bao gồm quản lý người dùng, thao tác bài đăng và công thức, phân tích nguyên liệu, kết nối đến Bedrock cho gợi ý AI

**4. Lớp AI Recommendation**

- **Amazon Bedrock:** Cung cấp khả năng generative AI để gợi ý công thức dựa trên nguyên liệu do người dùng cung cấp. Lambda gọi các model Bedrock (Claude, Titan) để phân tích danh sách nguyên liệu, tạo gợi ý công thức, phân loại danh mục thực phẩm và tối ưu các bước nấu ăn

**5. Lớp Data Storage**

- **Amazon DynamoDB:** Lưu trữ dữ liệu ứng dụng có cấu trúc như user profiles, posts/recipes, likes & comments, ingredient metadata
- **Amazon S3:** Lưu trữ dữ liệu phi cấu trúc như recipe images, user-uploaded food photos, static content. S3 bucket được tích hợp với CloudFront qua OAI cho truy cập an toàn

**6. Lớp Observability & Security**

- **Amazon CloudWatch:** Giám sát hiệu suất Lambda, API Gateway access logs và system metrics
- **AWS X-Ray:** Thực hiện distributed tracing cho API calls và debugging
- **IAM:** Định nghĩa permission boundaries giữa API, Lambda functions, Bedrock, DynamoDB và S3
- **Amazon SES:** Gửi verification emails, notifications và password recovery messages
- **Amazon SNS:** Xử lý system-level alerts và asynchronous messaging

**7. Deployment & Infrastructure Management**

- **AWS CDK:** Được developers sử dụng để định nghĩa và provision toàn bộ hạ tầng qua CloudFormation templates, đảm bảo triển khai nhất quán, có thể tái tạo và version-controlled

![Sơ đồ Kiến trúc](/images/2-Proposal/architecture-diagram.png)

## 2.2 Kế hoạch Kỹ thuật

Đối tác sẽ phát triển Infrastructure-as-Code (IaC) automation sử dụng AWS CDK (Cloud Development Kit) với TypeScript/Python để provision toàn bộ môi trường cloud. Cách tiếp cận này đảm bảo triển khai nhanh, nhất quán và có thể lặp lại trên nhiều AWS accounts và environments (dev, staging, production).

Tất cả resources như API Gateway, Lambda functions, DynamoDB tables, S3 buckets, Cognito user pools, Bedrock integration policies và CloudFront distributions sẽ được tự động hóa hoàn toàn qua IaC.

Quy trình build và deployment ứng dụng cho frontend (Next.js) sẽ được tự động hóa sử dụng AWS Amplify Hosting, tích hợp với GitLab pipelines. Các thành phần backend sẽ được triển khai thông qua CDK pipelines để đảm bảo releases được kiểm soát, versioned và có thể lặp lại.

## 2.3 Kế hoạch Dự án

Đối tác sẽ áp dụng Agile Scrum framework qua tám sprint 2 tuần. Các stakeholders từ team được yêu cầu tham gia Sprint Reviews và Sprint Retrospectives để đảm bảo alignment và cải tiến liên tục.

**Trách nhiệm team đề xuất:**

- **Product Owner:** Định nghĩa user stories, ưu tiên backlog và đảm bảo sản phẩm đáp ứng nhu cầu người dùng
- **Scrum Master:** Điều phối Scrum ceremonies, loại bỏ blockers và duy trì năng suất team
- **Development Team:** Triển khai features, thực hiện unit testing và cộng tác integration
- **AI/ML Specialist:** Phát triển và tinh chỉnh AI recommendation engine gợi ý công thức dựa trên nguyên liệu do người dùng cung cấp
- **UI/UX Designer:** Thiết kế giao diện trực quan và đảm bảo trải nghiệm người dùng mượt mà trên cả web và mobile platforms
- **QA/Testers:** Xác nhận chức năng feature, thực hiện regression testing và đảm bảo độ tin cậy hệ thống

**Nhịp độ giao tiếp:**

- **Daily Stand-ups:** Họp 15 phút cho cập nhật tiến độ và blockers ngay lập tức
- **Sprint Planning:** Vào đầu mỗi sprint để ưu tiên tasks
- **Sprint Review:** Vào cuối mỗi sprint để showcase các features hoàn thành cho stakeholders
- **Sprint Retrospective:** Sau mỗi sprint review để xác định cải tiến cho sprint tiếp theo

## 2.4 Cân nhắc Bảo mật

Đối tác sẽ triển khai các phương pháp bảo mật tốt nhất trên năm danh mục sau để đảm bảo tính bảo mật, toàn vẹn và khả dụng của nền tảng:

**1. Access**
- Bật Multi-Factor Authentication (MFA) cho tất cả tài khoản người dùng và quản trị
- Triển khai role-based access control (RBAC) để giới hạn quyền dựa trên vai trò người dùng
- Thực thi chính sách mật khẩu mạnh và rotation định kỳ

**2. Infrastructure**
- Triển khai ứng dụng trên các dịch vụ cloud được quản lý, an toàn theo các phương pháp bảo mật tốt nhất của AWS
- Sử dụng Virtual Private Cloud (VPC), network segmentation và security groups để cô lập resources
- Patch thường xuyên operating systems và containerized services để giảm thiểu vulnerabilities

**3. Data**
- Mã hóa tất cả dữ liệu at rest sử dụng AWS KMS-managed keys và dữ liệu in transit sử dụng TLS/HTTPS
- Triển khai data classification để bảo vệ thông tin người dùng nhạy cảm
- Áp dụng quy trình lưu trữ và backup dữ liệu an toàn

**4. Detection**
- Bật AWS CloudTrail và AWS Config để giám sát API activity và resource configurations
- Triển khai cơ chế logging và alerting để phát hiện các hoạt động bất thường hoặc đáng ngờ theo thời gian thực
- Thực hiện vulnerability scanning và penetration testing định kỳ trên nền tảng

**5. Incident Management**
- Thiết lập kế hoạch incident response chính thức bao gồm detection, containment, remediation và communication
- Duy trì audit trails và logs để hỗ trợ forensic investigation nếu xảy ra sự kiện bảo mật

---

# 3. HOẠT ĐỘNG VÀ SẢN PHẨM BÀN GIAO

## 3.1 Hoạt động và sản phẩm bàn giao

| **Giai đoạn Dự án** | **Timeline** | **Hoạt động** | **Sản phẩm/Milestones** | **Thời gian** |
|---------------------|--------------|---------------|-------------------------|---------------|
| Infrastructure Setup | Tuần 1-2 | Học tất cả dịch vụ AWS, Thực hành Lab | Worklog | 2 tuần |
| Project Foundation | Tuần 3 | Initialize monorepo, setup môi trường dev, Git/CI/CD, CDK project | Git repository, CDK project, environment configs | 1 tuần |
| DNS Infrastructure | Tuần 4-5 | Route 53 setup, ACM certificates, DNS validation | Hosted Zone, SSL certificates, DNS stack | 2 tuần |
| Core Stack | Tuần 6-7 | DynamoDB, S3, CloudFront, OpenSearch setup | Core infrastructure, storage, CDN | 2 tuần |
| Authentication Stack | Tuần 8 | Cognito User Pool, SES integration, Lambda triggers | User authentication system | 1 tuần |
| Backend Stack | Tuần 9-10 | API Gateway, Lambda functions, routing logic | REST API, backend services | 2 tuần |

## 3.2 NGOÀI PHẠM VI

**Hệ thống Messaging / Chat thời gian thực**
- Private hoặc group chat
- Hạ tầng messaging thời gian thực (WebSocket, SignalR, Firebase Realtime DB, etc.)
- Lưu trữ & mã hóa lịch sử tin nhắn

**Quản lý Friends / Social Graph**
- Friend requests, following/followers
- User-to-user connection graph
- Activity feed, notifications liên quan đến friend actions

**Voice & Video Calling thời gian thực**
- 1-to-1 hoặc group voice call
- Video call, screen sharing
- WebRTC signaling servers & TURN/STUN infrastructure

**Advanced Social Interaction**
- In-app messaging reactions
- Typing indicators, online/offline status
- Read receipts, presence tracking

## 3.3 LỘ TRÌNH ĐƯA VÀO SẢN XUẤT

**1. Project Foundation & Infrastructure**
- Initialize project structure
- Set up core infrastructure baseline
- Configure Route 53 Hosted Zone (DNS Stack)

**2. Cross-Region Certificate**
- Handle CloudFront requirement cho ACM certificate ở us-east-1
- Sync certificate usage với main stack ở ap-southeast-1

**3. Core Application Stacks**
- Core Stack: Shared resources / environment setup
- Authentication Stack: User auth, Cognito, permissions
- Backend Stack: API Gateway + Lambda functions

**4. Frontend Deployment**
- Deploy frontend (S3 + CloudFront)
- Bug fixes & QA testing

---

# 4. DỰ TOÁN CHI PHÍ AWS THEO DỊCH VỤ

**Target workload:** 100-500 Monthly Active Users (MAU)

| **Dịch vụ** | **Chi phí hàng tháng (USD)** | **Mô tả** |
|-------------|------------------------------|-----------|
| Amazon DynamoDB | $13.06 | Single-table design, 5 GSIs, provisioned capacity |
| Amazon S3 | $0.84 | 2 buckets, Intelligent-Tiering |
| Amazon CloudFront | $1.44 | CDN, Price Class 200 |
| Amazon Cognito | $0.00 | Free Tier (< 50K MAU) |
| AWS Lambda | $0.20 | 1M requests/month (Free Tier) |
| Amazon SES | $0.10 | Email sending |
| API Gateway | $14.60 | With caching (0.5GB) |
| Amazon Bedrock | $5.00 | Claude 3 Haiku (on-demand) |
| **Tổng** | **~$35.24/tháng** | Nằm trong ngân sách $200 |

📊 [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=7a8833402a63e273357ddc71071bfc2cdce4be2c)

---

# 5. NHÓM DỰ ÁN

**Partner Executive Sponsor**

| Tên | Chức danh | Mô tả | Liên hệ |
|-----|-----------|-------|---------|
| Nguyen Gia Hung | Director of FCJ Vietnam Training Program | Executive Sponsor chịu trách nhiệm giám sát tổng thể chương trình thực tập FCJ | hunggia@amazon.com |

**Project Stakeholders**

| Tên | Chức danh | Vai trò | Liên hệ |
|-----|-----------|---------|---------|
| Van Hoang Kha | Support Teams | Executive Assistant chịu trách nhiệm giám sát tổng thể chương trình thực tập FCJ | Khab9thd@gmail.com |

**Partner Project Team**

| Tên | Vai trò | Trách nhiệm | Liên hệ |
|-----|---------|-------------|---------|
| Pham Minh Hoang Viet | Leader | Project Manager | vietpmhse181851@gmail.com |
| Nguyen Van Truong | Member | DevOps | truongnvse182034@fpt.edu.vn |
| Huynh Duc Anh | Member | Cloud Engineer | anhhdse183114@fpt.edu.vn |
| Nguyen Thanh Hong | Member | Tester | hongntse183239@fpt.edu.vn |
| Nguyen Quy Duc | Member | Frontend | ducnqse182087@fpt.edu.vn |

---

# 6. NGUỒN LỰC & ƯỚC TÍNH CHI PHÍ

| **Nguồn lực** | **Trách nhiệm** | **Rate (USD)/Giờ** |
|---------------|-----------------|-------------------|
| Solution Architects [1] | Architecture design, AWS service selection, security review, cost optimization | $150 |
| Engineers [2] | Frontend (Next.js), Backend (Lambda/Node.js), Infrastructure (CDK), Testing | $100 |
| DevOps [1] | CI/CD setup, monitoring, deployment automation | $80 |

| **Giai đoạn Dự án** | **Architects** | **Engineers** | **DevOps** | **Tổng Giờ** |
|---------------------|----------------|---------------|------------|--------------|
| Discovery & Requirements | 16 | 24 | 8 | 48 |
| Architecture Design | 40 | 16 | 8 | 64 |
| Development | 16 | 200 | 40 | 256 |
| Testing & QA | 8 | 40 | 16 | 64 |
| Deployment & Go-Live | 8 | 24 | 24 | 56 |
| Documentation & Training | 8 | 16 | 8 | 32 |
| **Tổng Giờ** | **96** | **320** | **104** | **520** |
| **Tổng Chi phí** | **$14,400** | **$32,000** | **$8,320** | **$54,720** |

---

# 7. CHẤP THUẬN
Vì dự án này hiện đang ở giai đoạn trình bày và chưa được khách hàng đánh giá chính thức, nên quy trình chấp nhận sau đây được đề xuất cho các giai đoạn phân phối trong tương lai:

## 7.1 Tiêu chí Chấp nhận (Đề xuất)

Một sản phẩm sẽ được coi là chấp nhận được khi đáp ứng các tiêu chí sau:

- Các tính năng chức năng hoạt động theo đúng quy định (xác thực, quản lý công thức, tính năng xã hội, chức năng AI).

- Tất cả các API đều phản hồi chính xác và tích hợp với các dịch vụ AWS (Lambda, API Gateway, DynamoDB, S3).

- Đáp ứng các yêu cầu bảo mật (xác minh JWT, HTTPS, kiểm soát truy cập, mã hóa dữ liệu).

- Giao diện người dùng hoạt động như mong đợi trên các thiết bị được hỗ trợ.

- Không xuất hiện lỗi nghiêm trọng nào trong quá trình thực hiện kiểm thử.

## 7.2 Quy trình Chấp nhận {#acceptance-process .unnumbered}

- Thời gian xem xét: **8 ngày làm việc** để đánh giá và kiểm thử.

- Nếu được chấp nhận → Sản phẩm được ký duyệt.

- Nếu phát hiện vấn đề → Chúng tôi sẽ gửi thông báo từ chối kèm theo phản hồi.