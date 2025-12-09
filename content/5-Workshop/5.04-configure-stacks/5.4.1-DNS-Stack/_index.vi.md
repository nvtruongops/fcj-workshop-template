---
title: "5.4.1 DNS Stack"
weight: 1
---
---
# DNS Stack - Route 53 Hosted Zone

## Tổng quan

DNS Stack là **lớp nền tảng** (Phase 1) của hạ tầng EveryoneCook. Nó quản lý Route 53 Hosted Zone cho domain `everyonecook.cloud`, cung cấp hạ tầng DNS mà tất cả các stacks khác phụ thuộc vào.

**Thứ tự Triển khai**: Stack này **BẮT BUỘC** phải được deploy đầu tiên trước bất kỳ stack nào khác.

### Trách nhiệm Chính

- Tạo và quản lý Route 53 Public Hosted Zone
- Export Hosted Zone ID và name cho cross-stack references
- Cung cấp nameservers cho domain delegation từ Hostinger

### Stack này KHÔNG bao gồm

- SES Email Identity (được quản lý bởi Auth Stack - Phase 3)
- DKIM/SPF/DMARC records (được quản lý bởi Auth Stack - Phase 3)
- ACM Certificates (được quản lý bởi Certificate Stack - Phase 1.5)
- Application DNS records (được quản lý bởi các stacks tương ứng)

---

## Kiến trúc

```
┌─────────────────────────────────────────────────────────┐
│                    Hostinger Domain                      │
│                  everyonecook.cloud                      │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │  Domain Registrar Settings                      │    │
│  │  • Cập nhật Nameservers thành Route 53 NS      │    │
│  └────────────────┬───────────────────────────────┘    │
└───────────────────┼──────────────────────────────────────┘
                    │ DNS Delegation
                    ▼
┌─────────────────────────────────────────────────────────┐
│           AWS Route 53 Hosted Zone                       │
│           everyonecook.cloud                             │
│                                                          │
│  Tài nguyên Được tạo:                                   │
│  • Public Hosted Zone                                   │
│  • 4 Nameserver (NS) Records                            │
│  • SOA Record (tự động)                                 │
│                                                          │
│  Exports:                                                │
│  • Hosted Zone ID → Sử dụng bởi Certificate Stack      │
│  • Hosted Zone Name → Sử dụng bởi các stacks khác      │
│  • Nameservers → Cấu hình tại Hostinger                │
└─────────────────────────────────────────────────────────┘
```

---

## Cấu hình Stack

### Cấu trúc File

```
infrastructure/lib/stacks/
└── dns-stack.ts          # DNS Stack implementation
```

### Code Implementation

**File**: `infrastructure/lib/stacks/dns-stack.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import { BaseStack, BaseStackProps } from '../base-stack';

export class DnsStack extends BaseStack {
  public readonly hostedZone: cdk.aws_route53.IHostedZone;

  constructor(scope: Construct, id: string, props: BaseStackProps) {
    super(scope, id, props);

    // Thêm stack-specific tags
    cdk.Tags.of(this).add('StackType', 'DNS');
    cdk.Tags.of(this).add('Layer', 'Foundation');
    cdk.Tags.of(this).add('CostCenter', `DNS-${this.config.environment}`);

    // Tạo Route 53 Hosted Zone
    this.hostedZone = this.createHostedZone();

    // Export stack outputs
    this.exportOutputs();
  }

  private createHostedZone(): cdk.aws_route53.IHostedZone {
    // Trích xuất root domain từ environment config
    const rootDomain = this.config.domains.frontend
      .replace(/^(dev\.|staging\.)/, ''); // everyonecook.cloud

    const hostedZone = new cdk.aws_route53.PublicHostedZone(
      this, 
      'HostedZone', 
      {
        zoneName: rootDomain,
        comment: `Hosted Zone cho Everyone Cook ${this.config.environment} environment`,
      }
    );

    cdk.Tags.of(hostedZone).add('Component', 'DNS');
    cdk.Tags.of(hostedZone).add('ManagedBy', 'CDK');

    return hostedZone;
  }

  private exportOutputs(): void {
    // Export Hosted Zone ID
    new cdk.CfnOutput(this, 'HostedZoneId', {
      value: this.hostedZone.hostedZoneId,
      exportName: this.exportName('HostedZoneId'),
      description: 'Route 53 Hosted Zone ID',
    });

    // Export Hosted Zone Name
    new cdk.CfnOutput(this, 'HostedZoneName', {
      value: this.hostedZone.zoneName,
      exportName: this.exportName('HostedZoneName'),
      description: 'Domain name được quản lý bởi Route 53',
    });

    // Export Nameservers (cho cấu hình Hostinger)
    new cdk.CfnOutput(this, 'NameServers', {
      value: cdk.Fn.join(', ', this.hostedZone.hostedZoneNameServers || []),
      description: '⚠️ Cập nhật các nameservers này tại Hostinger',
    });
  }
}
```

---

## Chi tiết Cấu hình Chính

### 1. Logic Trích xuất Domain

Stack tự động trích xuất root domain từ environment configuration:

```typescript
// Environment config: dev.everyonecook.cloud
// Domain được trích xuất: everyonecook.cloud
const rootDomain = this.config.domains.frontend.replace(/^(dev\.|staging\.)/, '');
```

**Các Environments**:

- **Dev**: `dev.everyonecook.cloud` → Hosted Zone: `everyonecook.cloud`
- **Staging**: `staging.everyonecook.cloud` → Hosted Zone: `everyonecook.cloud`
- **Prod**: `everyonecook.cloud` → Hosted Zone: `everyonecook.cloud`

### 2. Quy ước Đặt tên Resource

Tất cả resources tuân theo một mẫu đặt tên nhất quán:

```typescript
// Định dạng tên resource: everyonecook-{env}-{resource}
protected resourceName(name: string): string {
  return `everyonecook-${this.config.environment}-${name}`;
}

// Định dạng tên export: EveryoneCook-{Env}-{Export}
protected exportName(name: string): string {
  return `EveryoneCook-${this.config.environment}-${name}`;
}
```

**Ví dụ**:

- Tên stack: `EveryoneCook-dev-DNS`
- Export: `EveryoneCook-dev-HostedZoneId`

### 3. Resource Tags

Mỗi resource được tag để theo dõi chi phí và quản lý:

```typescript
{
  Stack: 'EveryoneCook-dev-DNS',
  Environment: 'dev',
  StackType: 'DNS',
  Layer: 'Foundation',
  CostCenter: 'DNS-dev',
  Component: 'DNS',
  ManagedBy: 'CDK',
  Project: 'EveryoneCook'
}
```

---

## Stack Outputs

Sau khi deployment, stack export các giá trị sau:

| Tên Output         | Giá trị                                         | Sử dụng                                              |
| ------------------ | ----------------------------------------------- | ---------------------------------------------------- |
| `HostedZoneId`   | `Z0123456789ABCDEFGHIJ`                       | Sử dụng bởi Certificate Stack cho DNS validation    |
| `HostedZoneName` | `everyonecook.cloud`                          | Sử dụng bởi các stacks khác để tạo DNS records      |
| `NameServers`    | `ns-1.awsdns-01.com, ns-2.awsdns-02.org, ...` | Cấu hình tại Hostinger cho DNS delegation           |

---

## Các Bước Triển khai

### Bước 1: Review Cấu hình

Di chuyển đến thư mục infrastructure:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Xác minh environment configuration trong `config/environment.ts`:

```typescript
dev: {
  environment: 'dev',
  account: 'YOUR_AWS_ACCOUNT_ID',
  region: 'ap-southeast-1',
  domains: {
    frontend: 'dev.everyonecook.cloud',
    api: 'api-dev.everyonecook.cloud',
    cdn: 'cdn-dev.everyonecook.cloud',
  },
  // ... các configs khác
}
```

### Bước 2: Deploy DNS Stack

Deploy DNS stack lên AWS:

```powershell
# Deploy chỉ DNS Stack
npx cdk deploy EveryoneCook-dev-DNS --context environment=dev
```

Kết quả mong đợi:

```
✨  Synthesis time: 5.23s

EveryoneCook-dev-DNS: deploying...
EveryoneCook-dev-DNS: creating CloudFormation changeset...

 ✅  EveryoneCook-dev-DNS

✨  Deployment time: 45.67s

Outputs:
EveryoneCook-dev-DNS.HostedZoneId = Z0123456789ABCDEFGHIJ
EveryoneCook-dev-DNS.HostedZoneName = everyonecook.cloud
EveryoneCook-dev-DNS.NameServers = ns-123.awsdns-45.com, ns-678.awsdns-90.net, 
                                    ns-1234.awsdns-56.org, ns-5678.awsdns-01.co.uk

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:123456789012:stack/EveryoneCook-dev-DNS/...
```

### Bước 3: Xác minh trong AWS Console

1. Di chuyển đến **Route 53** trong AWS Console
2. Vào **Hosted zones**
3. Xác minh hosted zone `everyonecook.cloud` đã được tạo

**Giao diện mong đợi**:

- **Domain name**: `everyonecook.cloud`
- **Type**: Public hosted zone
- **Records**: 2 (NS và SOA records - tự động được tạo)

![Route 53 Hosted Zone](/images/5-Workshop/5.4-configure-stacks/host_router53.jpg)
*Route 53 Hosted Zone hiển thị chi tiết domain, NS records (4 nameservers), và SOA record*

### Bước 4: Copy Nameservers

Từ CloudFormation Outputs hoặc Route 53 console, copy tất cả 4 nameserver records:

```
ns-123.awsdns-45.com
ns-678.awsdns-90.net
ns-1234.awsdns-56.org
ns-5678.awsdns-01.co.uk
```

![Route 53 Nameservers](/images/5-Workshop/5.2-setup-environment/DNS_record.png)
*Vị trí nameserver records trong Route 53 console*

---

## Cấu hình Hostinger

### Bước Quan trọng Sau Deployment

⚠️ **QUAN TRỌNG**: Sau khi deploy DNS Stack, bạn **BẮT BUỘC** phải cập nhật nameservers tại Hostinger để ủy quyền quản lý DNS cho Route 53.

### Cập nhật Nameservers tại Hostinger

1. **Đăng nhập vào Hostinger hPanel**

   - Truy cập [https://hpanel.hostinger.com](https://hpanel.hostinger.com)
   - Đăng nhập với thông tin đăng nhập Hostinger của bạn
2. **Truy cập Quản lý Domain**

   - Vào mục **Domains**
   - Chọn domain `everyonecook.cloud`
3. **Thay đổi Nameservers**

   - Click vào **DNS/Nameservers**
   - Chọn **Change nameservers**
   - Chọn **Custom nameservers**
4. **Nhập Route 53 Nameservers**

   - Nameserver 1: `ns-123.awsdns-45.com`
   - Nameserver 2: `ns-678.awsdns-90.net`
   - Nameserver 3: `ns-1234.awsdns-56.org`
   - Nameserver 4: `ns-5678.awsdns-01.co.uk`
5. **Lưu Cấu hình**

   - Click nút **Change nameservers**
   - Đợi xác nhận

![Hostinger Nameserver Configuration](/images/5-Workshop/5.2-setup-environment/hostinger.png)
*Hostinger hPanel hiển thị cấu hình custom nameservers với Route 53 NS records*

### Thời gian Propagation

- **Propagation ban đầu**: 15-30 phút
- **Full global propagation**: Lên đến 48 giờ (thường trong vòng 2-4 giờ)

### Xác minh DNS Delegation

Sau khi cập nhật nameservers, xác minh delegation:

```powershell
# Kiểm tra nameservers cho domain
nslookup -type=NS everyonecook.cloud

# Hoặc sử dụng dig (nếu có)
dig NS everyonecook.cloud
```

Kết quả mong đợi:

```
everyonecook.cloud      nameserver = ns-123.awsdns-45.com
everyonecook.cloud      nameserver = ns-678.awsdns-90.net
everyonecook.cloud      nameserver = ns-1234.awsdns-56.org
everyonecook.cloud      nameserver = ns-5678.awsdns-01.co.uk
```

---

## Phân tích Chi phí

### Chi phí Hàng tháng

| Resource                       | Chi phí                     | Ghi chú                             |
| ------------------------------ | --------------------------- | ----------------------------------- |
| **Route 53 Hosted Zone** | $0.50/tháng                 | Chi phí cố định cho mỗi hosted zone |
| **DNS Queries**          | $0.40 per triệu queries     | 1 tỷ queries đầu tiên/tháng         |
| **Tổng (Ước tính)**    | **~$0.50-1.00/tháng** | Traffic rất thấp trong dev environment  |

### Ghi chú Tối ưu Chi phí

- ✅ Một hosted zone duy nhất cho tất cả environments (dev, staging, prod)
- ✅ Sử dụng subdomain prefixes để phân biệt environments
- ✅ Không có chi phí bổ sung cho NS, SOA, hoặc các DNS records khác
- ✅ Giá pay-per-query rất hiệu quả về chi phí cho low-medium traffic

---

## Cross-Stack Dependencies

### Exports Được sử dụng bởi Các Stacks Khác

DNS Stack export các giá trị được import bởi:

1. **Certificate Stack** (Phase 1.5)

   - Imports: `HostedZoneId`
   - Mục đích: Tạo DNS validation records cho ACM certificates
2. **Core Stack** (Phase 2)

   - Imports: `HostedZoneId`, `HostedZoneName`
   - Mục đích: Tạo CloudFront A/AAAA alias records
3. **Backend Stack** (Phase 4)

   - Imports: `HostedZoneId`
   - Mục đích: Tạo API Gateway custom domain DNS records

### Luồng Dependencies

```
DNS Stack (Route 53)
    │
    ├─► Certificate Stack (ACM certificates)
    │
    ├─► Core Stack (CloudFront DNS records)
    │
    └─► Backend Stack (API Gateway DNS records)
```

---

## Checklist Validation

Trước khi tiếp tục deploy Certificate Stack:

- [ ] DNS Stack đã deploy thành công lên AWS
- [ ] Route 53 Hosted Zone hiển thị trong AWS Console
- [ ] 4 nameserver records đã lấy từ stack outputs
- [ ] Nameservers đã cập nhật tại Hostinger hPanel
- [ ] DNS delegation đã xác minh với `nslookup` hoặc `dig`
- [ ] Stack exports hiển thị trong CloudFormation console
- [ ] Tags đã được áp dụng đúng cho tất cả resources

---

## Bước tiếp theo

Sau khi deploy và cấu hình DNS Stack thành công:

➡️ **[5.4.2 Certificate Stack](../5.4.2-Certificate-Stack/)** - Tạo ACM certificates với DNS validation

Certificate Stack sẽ:

- Tạo ACM certificate cho CloudFront (`cdn.everyonecook.cloud`)
- Tạo wildcard ACM certificate cho API Gateway (`*.everyonecook.cloud`)
- Tự động tạo DNS validation records trong Route 53
- Phải được deploy tới region **us-east-1** (yêu cầu CloudFront)

---

## Tài liệu Tham khảo

- **Source Code**: `infrastructure/lib/stacks/dns-stack.ts`
- **Base Stack**: `infrastructure/lib/base-stack.ts`
- **Environment Config**: `infrastructure/config/environment.ts`
- **AWS Documentation**: [Route 53 Hosted Zones](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html)
- **Hostinger Guide**: [How to Change Nameservers](https://support.hostinger.com/en/articles/1583227-how-to-change-nameservers-at-hostinger)
