---
title: "Tổng quan Workshop"
date: 2025-12-01
weight: 1
chapter: false
pre: " <b> 5.01. </b> "
---

#### Kiến trúc EveryoneCook

**EveryoneCook** là một nền tảng mạng xã hội nấu ăn hiện đại được xây dựng hoàn toàn trên công nghệ serverless của AWS. Kiến trúc tuân theo các best practices về khả năng mở rộng, bảo mật và tối ưu chi phí, với trọng tâm là hỗ trợ nguyên liệu tiếng Việt và đề xuất công thức nấu ăn bằng AI.

#### Các thành phần chính

+ **Frontend**: Ứng dụng Next.js 15 với Flowbite React components được host trên AWS Amplify
+ **Backend**: API serverless sử dụng API Gateway với mô hình API Router và 6 Lambda modules
+ **Database**: DynamoDB Single Table Design với username-based PK và 5 GSI indexes
+ **Storage**: S3 buckets với Intelligent-Tiering và CloudFront CDN với OAC
+ **Authentication**: Cognito User Pool với Lambda triggers cho custom flows
+ **AI Integration**: Amazon Bedrock (Claude 3 Haiku - anthropic.claude-3-haiku-20240307-v1:0) cho dịch nguyên liệu, tra cứu dinh dưỡng và tạo công thức
+ **Security**: WAF cho API Gateway, Shield Standard cho CloudFront, KMS encryption
+ **Monitoring**: CloudWatch dashboards, alarms, và X-Ray distributed tracing

#### Sơ đồ kiến trúc

![Kiến trúc EveryoneCook](/images/architecture-diagram.png)

#### Quy trình Workshop

Workshop này tuân theo quy trình phát triển ứng dụng thực tế:

1. **Thiết lập Môi trường** - Cài đặt công cụ (Node.js, AWS CLI, CDK CLI)
2. **CDK Bootstrap** - Chuẩn bị AWS account cho CDK deployments
3. **Cấu hình Stacks** - Thiết lập cấu hình hạ tầng (DNS, Certificate, Core, Auth, Backend, Observability)
4. **Triển khai Infrastructure** - Deploy tất cả CDK stacks lên AWS
5. **Cấu hình API & Lambda** - Thiết lập API Gateway routes và Lambda functions
6. **Triển khai Backend** - Deploy API và Lambda code
7. **Kiểm tra Endpoints** - Xác minh tất cả endpoints hoạt động end-to-end
8. **Push lên GitLab** - Version control và thiết lập CI/CD
9. **Triển khai lên Amplify** - Deploy frontend lên AWS Amplify
10. **Giám sát & Bảo trì** - Sử dụng CloudWatch và X-Ray để monitoring

#### Những gì bạn sẽ học

+ Infrastructure as Code với AWS CDK (TypeScript)
+ Kiến trúc serverless với mô hình API Router
+ DynamoDB Single Table Design với PROVISIONED billing mode
+ CloudFront CDN với Origin Access Control (OAC) - PriceClass 200
+ Cognito authentication với Lambda triggers
+ Tổ chức Lambda function theo modules (6 modules: API Router, Auth/User, Social, Recipe/AI, Admin, Upload)
+ Xử lý bất đồng bộ với SQS (4 queues và workers)
+ Tích hợp Bedrock AI (Claude 3 Haiku) với chiến lược dictionary-first caching
+ WAF security cho API Gateway (REGIONAL scope)
+ CloudWatch monitoring với structured logging (X-Ray disabled để tối ưu chi phí)
+ Chiến lược tối ưu chi phí cho low-traffic scenarios

#### Ước tính chi phí thực tế (Low Traffic Scenarios)

**Kịch bản 1: 100-500 Active Users/Day (2 giờ sử dụng)**
- **Monthly Active Users**: ~3,000-15,000
- **Daily Requests**: ~5,000-25,000 API calls
- **Ước tính Chi phí Tháng**: **$12-18/tháng** ($0.40-0.60/ngày)

**Kịch bản 2: 1,000 Active Users/Day (2 giờ sử dụng)**
- **Monthly Active Users**: ~30,000
- **Daily Requests**: ~50,000 API calls
- **Ước tính Chi phí Tháng**: **$25-35/tháng** ($0.83-1.17/ngày)

**Các yếu tố chi phí chính**:
+ **DynamoDB**: PROVISIONED mode (2 RCU/WCU) với auto-scaling → $1.25/tháng base + $0.50-3/tháng scaling
+ **Lambda**: 6 functions (512MB-1GB memory) → $2-8/tháng (1M requests đầu miễn phí)
+ **CloudFront**: PriceClass 200 (Asia/US/Europe) → $1-5/tháng (1TB đầu miễn phí, sau đó $0.085/GB)
+ **API Gateway**: Regional endpoint → $1-3/tháng (1M đầu miễn phí, sau đó $3.50/triệu)
+ **Cognito**: Standard security (không có Advanced Security Mode) → $0-1.50/tháng (50K MAU đầu miễn phí)
+ **S3**: Intelligent-Tiering → $0.50-2/tháng (50GB đầu miễn phí)
+ **Bedrock AI**: Claude 3 Haiku ($0.25/1M input tokens) → $2-8/tháng với 99% dictionary cache hit rate
+ **SQS**: 4 queues (AI, Image, Analytics, Notification) → $0-0.40/tháng (1M requests đầu miễn phí)
+ **CloudWatch Logs**: 3-day retention → $0.50-2/tháng
+ **WAF**: Chỉ API Gateway (không có CloudFront WAF) → $5/tháng base + $0.60/triệu requests

**Tối ưu Chi phí Đã áp dụng**:
+ X-Ray tracing disabled (tiết kiệm $5-10/tháng)
+ CloudFront WAF đã xóa, dùng Shield Standard (tiết kiệm $6/tháng)
+ DynamoDB PROVISIONED mode với baseline thấp (2 RCU/WCU) thay vì ON_DEMAND
+ CloudWatch log retention giảm xuống 3 ngày cho dev (tiết kiệm $2-5/tháng)
+ API Gateway caching disabled cho dev (tiết kiệm $14.60/tháng)
+ Bedrock dictionary-first strategy (99% cache hit rate, giảm chi phí AI 70%)
+ Lambda Shared Dependencies Layer (giảm deployment size 90%, cold starts nhanh hơn)

#### Tính năng chính

+ **Hỗ trợ tiếng Việt**: Vietnamese analyzer cho tìm kiếm nguyên liệu
+ **Hỗ trợ AI**: Bedrock Claude 3 Haiku cho tạo công thức
+ **Dịch Dictionary-First**: Mục tiêu coverage 80% với intelligent caching
+ **Nền tảng Mạng xã hội**: Posts, comments, reactions, friends, notifications
+ **Field-Level Privacy**: Kiểm soát chi tiết về khả năng hiển thị profile
+ **Content Moderation**: Quy trình kiểm duyệt tự động và thủ công
+ **Tìm kiếm Nâng cao**: Full-text search với Vietnamese normalization
+ **Tối ưu Chi phí**: Intelligent-Tiering, caching và tối ưu tài nguyên
