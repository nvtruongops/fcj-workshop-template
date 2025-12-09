---
title: "5.4.7 Observability Stack"
weight: 7
---
---
# Observability Stack - Monitoring & Alerting

## Tổng quan

Observability Stack là **lớp giám sát Phase 7** của hạ tầng EveryoneCook. Nó cung cấp giám sát toàn diện, cảnh báo và trực quan hóa cho tất cả các stack đã triển khai thông qua CloudWatch dashboards, alarms và SNS notifications.

**Thứ tự triển khai**: Stack này **PHẢI** được triển khai **CUỐI CÙNG**, sau khi tất cả các stack khác (DNS, Certificate, Core, Auth, Backend, Frontend) đã được triển khai.

⚠️ **Lưu ý môi trường**: Hướng dẫn này tập trung vào triển khai môi trường **Development (dev)**. Đối với triển khai staging/production, ngưỡng alarm và khoảng thời gian giám sát có thể khác.

### Trách nhiệm chính

- Tạo CloudWatch Dashboards cho tất cả các lớp stack
- Cấu hình CloudWatch Alarms cho các metric quan trọng
- Thiết lập SNS Topic cho thông báo alarm
- Tạo Composite Alarm cho trạng thái sức khỏe tổng thể hệ thống
- Giám sát API Gateway, Lambda, DynamoDB, S3, CloudFront và Cognito
- Theo dõi chi phí và billing metrics

### Stack này bao gồm

**CloudWatch Dashboards** (4 dashboards):
1. **Core Dashboard**: DynamoDB, S3, CloudFront metrics
2. **Auth Dashboard**: Cognito authentication metrics
3. **Backend Dashboard**: API Gateway, Lambda, SQS metrics
4. **Overview Dashboard**: Aggregated system health view

**CloudWatch Alarms** (15+ alarms):
- API Gateway: 5XX errors, 4XX errors, latency
- Lambda: Error rate, throttles, duration
- DynamoDB: Read/write throttles, latency
- S3: 4XX/5XX errors
- SQS: DLQ messages, queue age
- Cost: Daily spending warnings

**Hệ thống thông báo**:
- SNS Topic cho thông báo alarm
- Email subscriptions cho alerts
- Composite alarm cho trạng thái sức khỏe hệ thống

---

## Kiến trúc

```
┌──────────────────────────────────────────────────────────────────────┐
│            Observability Stack (Phase 7 - Dev Environment)            │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  SNS Topic (Alarm Notifications)                                │ │
│  │  ├─ Topic Name: EveryoneCook-dev-Alarms                        │ │
│  │  ├─ Email Subscription: team@everyonecook.cloud                │ │
│  │  └─ Protocol: Email (yêu cầu xác nhận)                         │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                          │                                            │
│                          ▼ (Alarm Actions)                            │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  CloudWatch Alarms (15+ alarms)                                 │ │
│  │                                                                  │ │
│  │  API Gateway Alarms:                                            │ │
│  │  ├─ 5XX Error Rate > 5% (Critical)                             │ │
│  │  ├─ 4XX Error Rate > 20% (Warning)                             │ │
│  │  └─ P99 Latency > 3s (Warning)                                 │ │
│  │                                                                  │ │
│  │  Lambda Alarms:                                                 │ │
│  │  ├─ Error Rate > 5% (Critical)                                 │ │
│  │  ├─ Throttles > 10 (Critical)                                  │ │
│  │  └─ P99 Duration > 10s (Warning)                               │ │
│  │                                                                  │ │
│  │  DynamoDB Alarms:                                               │ │
│  │  ├─ Read Throttles > 10 (Critical)                             │ │
│  │  ├─ Write Throttles > 10 (Critical)                            │ │
│  │  └─ P99 Latency > 100ms (Warning)                              │ │
│  │                                                                  │ │
│  │  S3 Alarms:                                                     │ │
│  │  ├─ 4XX Error Rate > 5% (Warning)                              │ │
│  │  └─ 5XX Errors > 0 (Critical)                                  │ │
│  │                                                                  │ │
│  │  SQS Alarms:                                                    │ │
│  │  ├─ DLQ Messages > 0 (Critical)                                │ │
│  │  └─ Message Age > 5 minutes (Warning)                          │ │
│  │                                                                  │ │
│  │  Cost Alarms:                                                   │ │
│  │  ├─ Daily Cost > $50 (Warning)                                 │ │
│  │  └─ Daily Cost > $100 (Critical)                               │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                          │                                            │
│                          ▼                                            │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Composite Alarm (System Health)                                │ │
│  │  ├─ Name: EveryoneCook-dev-SystemHealth                        │ │
│  │  ├─ Triggers: BẤT KỲ critical alarm nào kích hoạt              │ │
│  │  └─ Action: Gửi SNS notification                               │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  CloudWatch Dashboards (4 dashboards)                           │ │
│  │                                                                  │ │
│  │  1. Core Dashboard (EveryoneCook-dev-Core):                    │ │
│  │     ├─ DynamoDB: Read/Write capacity, throttles, latency       │ │
│  │     ├─ S3: Requests, errors, bytes transferred                 │ │
│  │     └─ CloudFront: Requests, error rates, bytes downloaded     │ │
│  │                                                                  │ │
│  │  2. Auth Dashboard (EveryoneCook-dev-Auth):                    │ │
│  │     ├─ Cognito: Sign-ups, sign-ins                             │ │
│  │     └─ Cognito: Failed authentications                         │ │
│  │                                                                  │ │
│  │  3. Backend Dashboard (EveryoneCook-dev-Backend):              │ │
│  │     ├─ API Gateway: Requests, latency (P50/P95/P99)            │ │
│  │     ├─ API Gateway: 4XX/5XX errors                             │ │
│  │     ├─ Lambda: Invocations, duration, errors, throttles        │ │
│  │     └─ SQS: Messages sent, visible, oldest age                 │ │
│  │                                                                  │ │
│  │  4. Overview Dashboard (EveryoneCook-dev-Overview):            │ │
│  │     ├─ System Health: Environment info, region                 │ │
│  │     ├─ Key Metrics: API requests, latency, Lambda stats        │ │
│  │     ├─ Error Trends: API 5XX, Lambda errors (giờ gần nhất)    │ │
│  │     ├─ Cost Tracking: Chi phí ước tính hàng ngày, xu hướng 7d │ │
│  │     └─ Alarm Status: Composite alarm widget                    │ │
│  └─────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
                          │
                          │ Giám sát
                          ▼
        ┌──────────────────┴───────────────────┐
        ▼                  ▼                    ▼
   Core Stack        Auth Stack           Backend Stack
   (DynamoDB,        (Cognito)            (API Gateway,
    S3, CDN)                               Lambda, SQS)
```

---

## Cấu hình Stack

### Cấu trúc thư mục

```
infrastructure/lib/stacks/
└── observability-stack.ts   # Observability Stack implementation (1175 dòng)
```

### Code Implementation Highlights

**File**: `infrastructure/lib/stacks/observability-stack.ts`

#### 1. SNS Topic cho Alarms

```typescript
/**
 * Tạo SNS Topic cho CloudWatch Alarms
 * Task 7.4.2 - Step 1
 */
private createAlarmTopic(): sns.Topic {
  const topic = new sns.Topic(this, 'AlarmTopic', {
    topicName: `EveryoneCook-${this.config.environment}-Alarms`,
    displayName: 'Everyone Cook CloudWatch Alarms',
  });

  // Thêm email subscription cho alarm notifications
  topic.addSubscription(
    new sns_subscriptions.EmailSubscription(this.config.contact.email)
  );

  return topic;
}
```

**Cấu hình**: Email subscription yêu cầu xác nhận qua AWS SNS.

#### 2. API Gateway Alarms

```typescript
// API Gateway: High 5XX Error Rate (Critical)
const api5xxAlarm = new cloudwatch.Alarm(this, 'API-5XX-Critical', {
  alarmName: `EveryoneCook-${this.config.environment}-API-5XX-Critical`,
  alarmDescription: 'API Gateway 5XX error rate > 5% trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/ApiGateway',
    metricName: '5XXError',
    dimensionsMap: { ApiName: apiName },
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 5,
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
api5xxAlarm.addAlarmAction(alarmAction);

// API Gateway: High Latency (Warning)
const apiLatencyAlarm = new cloudwatch.Alarm(this, 'API-Latency-High', {
  alarmName: `EveryoneCook-${this.config.environment}-API-Latency-High`,
  alarmDescription: 'API Gateway P99 latency > 3s trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/ApiGateway',
    metricName: 'Latency',
    dimensionsMap: { ApiName: apiName },
    statistic: 'p99',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 3000, // 3 giây tính bằng milliseconds
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
});
```

#### 3. Lambda Alarms

```typescript
// Lambda: High Error Rate (Critical)
const lambdaErrorAlarm = new cloudwatch.Alarm(this, 'Lambda-Error-Rate', {
  alarmName: `EveryoneCook-${this.config.environment}-Lambda-Error-Rate`,
  alarmDescription: 'Lambda error rate > 5% trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/Lambda',
    metricName: 'Errors',
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 5,
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
});

// Lambda: Throttles (Critical)
const lambdaThrottleAlarm = new cloudwatch.Alarm(this, 'Lambda-Throttle', {
  alarmName: `EveryoneCook-${this.config.environment}-Lambda-Throttle`,
  alarmDescription: 'Lambda throttles > 10 trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/Lambda',
    metricName: 'Throttles',
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 10,
  evaluationPeriods: 1,
});
```

#### 4. DynamoDB Alarms

```typescript
// DynamoDB: Read Throttles (Critical)
const dynamoReadThrottleAlarm = new cloudwatch.Alarm(this, 'DynamoDB-Read-Throttle', {
  alarmName: `EveryoneCook-${this.config.environment}-DynamoDB-Read-Throttle`,
  alarmDescription: 'DynamoDB read throttles > 10 trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/DynamoDB',
    metricName: 'ReadThrottleEvents',
    dimensionsMap: { TableName: dynamoTableName },
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 10,
  evaluationPeriods: 2,
});

// DynamoDB: High Latency (Warning)
const dynamoLatencyAlarm = new cloudwatch.Alarm(this, 'DynamoDB-Latency-High', {
  alarmName: `EveryoneCook-${this.config.environment}-DynamoDB-Latency-High`,
  alarmDescription: 'DynamoDB P99 latency > 100ms trong 5 phút',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/DynamoDB',
    metricName: 'SuccessfulRequestLatency',
    dimensionsMap: {
      TableName: dynamoTableName,
      Operation: 'Query',
    },
    statistic: 'p99',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 100, // 100ms
  evaluationPeriods: 3,
});
```

#### 5. Composite Alarm

```typescript
/**
 * Tạo Composite Alarm cho trạng thái sức khỏe tổng thể hệ thống
 * Task 7.4.2 - Step 2
 */
private createCompositeAlarm(alarms: cloudwatch.IAlarm[]): cloudwatch.CompositeAlarm {
  // Lọc chỉ các critical alarms
  const criticalAlarms = alarms.filter(
    (alarm) => alarm.alarmName.includes('Critical') || alarm.alarmName.includes('Throttle')
  );

  const compositeAlarm = new cloudwatch.CompositeAlarm(this, 'SystemHealth', {
    compositeAlarmName: `EveryoneCook-${this.config.environment}-SystemHealth`,
    alarmDescription: 'Trạng thái sức khỏe tổng thể hệ thống - kích hoạt nếu bất kỳ critical alarm nào fires',
    alarmRule: cloudwatch.AlarmRule.anyOf(
      ...criticalAlarms.map((alarm) =>
        cloudwatch.AlarmRule.fromAlarm(alarm, cloudwatch.AlarmState.ALARM)
      )
    ),
  });

  compositeAlarm.addAlarmAction(new cloudwatch_actions.SnsAction(this.alarmTopic));
  return compositeAlarm;
}
```

#### 6. Core Dashboard

```typescript
/**
 * Tạo Core Dashboard (DynamoDB, S3, CloudFront)
 */
private createCoreDashboard(props: ObservabilityStackProps): cloudwatch.Dashboard {
  const dashboard = new cloudwatch.Dashboard(this, 'CoreDashboard', {
    dashboardName: `EveryoneCook-${this.config.environment}-Core`,
  });

  // DynamoDB Metrics
  dashboard.addWidgets(
    new cloudwatch.GraphWidget({
      title: 'DynamoDB - Read/Write Capacity',
      left: [
        new cloudwatch.Metric({
          namespace: 'AWS/DynamoDB',
          metricName: 'ConsumedReadCapacityUnits',
          dimensionsMap: { TableName: dynamoTableName },
          statistic: 'Sum',
          period: cdk.Duration.minutes(5),
        }),
        new cloudwatch.Metric({
          namespace: 'AWS/DynamoDB',
          metricName: 'ConsumedWriteCapacityUnits',
          dimensionsMap: { TableName: dynamoTableName },
          statistic: 'Sum',
          period: cdk.Duration.minutes(5),
        }),
      ],
      width: 12,
    }),
    new cloudwatch.GraphWidget({
      title: 'DynamoDB - Throttles',
      left: [
        new cloudwatch.Metric({
          namespace: 'AWS/DynamoDB',
          metricName: 'ReadThrottleEvents',
          dimensionsMap: { TableName: dynamoTableName },
          statistic: 'Sum',
          period: cdk.Duration.minutes(5),
        }),
        new cloudwatch.Metric({
          namespace: 'AWS/DynamoDB',
          metricName: 'WriteThrottleEvents',
          dimensionsMap: { TableName: dynamoTableName },
          statistic: 'Sum',
          period: cdk.Duration.minutes(5),
        }),
      ],
      width: 12,
    })
  );

  // S3 và CloudFront metrics...
  return dashboard;
}
```

#### 7. Overview Dashboard

```typescript
/**
 * Tạo Overview Dashboard (Aggregated view)
 */
private createOverviewDashboard(props: ObservabilityStackProps): cloudwatch.Dashboard {
  const dashboard = new cloudwatch.Dashboard(this, 'OverviewDashboard', {
    dashboardName: `EveryoneCook-${this.config.environment}-Overview`,
  });

  // System Health Header
  dashboard.addWidgets(
    new cloudwatch.TextWidget({
      markdown: `# Everyone Cook - System Overview\n\n**Environment:** ${this.config.environment}\n\n**Region:** ${this.region}`,
      width: 24,
      height: 2,
    })
  );

  // Key Metrics: API, Lambda, DynamoDB, S3
  // Cost Tracking
  // Alarm Status Widget

  return dashboard;
}
```

---

## Hướng dẫn Triển khai

### Điều kiện tiên quyết

Trước khi triển khai Observability Stack, đảm bảo:

1. **Tất cả các stack khác đã triển khai**:
   - DNS Stack (Phase 1)
   - Certificate Stack (Phase 1.5)
   - Core Stack (Phase 2)
   - Auth Stack (Phase 3)
   - Backend Stack (Phase 4)
   - Frontend Stack (Phase 6 - Amplify)

2. **Stack exports có sẵn**:
   ```bash
   aws cloudformation list-exports --region ap-southeast-1
   ```
   
   Expected exports:
   - `EveryoneCook-dev-Core-TableName`
   - `EveryoneCook-dev-Core-ContentBucketName`
   - `EveryoneCook-dev-Core-DistributionId`
   - `EveryoneCook-dev-Auth-UserPoolId`
   - `EveryoneCook-dev-Backend-ApiName`

3. **Email đã cấu hình**:
   - Email hợp lệ trong `infrastructure/config/dev.ts`
   - Email sẽ nhận xác nhận subscription SNS

### Bước 1: Xem lại cấu hình

**File**: `infrastructure/config/dev.ts`

```typescript
export const devConfig: EnvironmentConfig = {
  environment: 'dev',
  region: 'ap-southeast-1',
  
  // Email cho alarm notifications
  contact: {
    email: 'your-email@example.com',  // ⚠️ Cập nhật cái này
    phone: '+1234567890',
  },
  
  // Monitoring settings (đã cấu hình)
  monitoring: {
    enableDetailedMonitoring: true,
    retainLogs: true,
    logRetentionDays: 7,
    enableXRay: false,  // Tắt cho dev để tiết kiệm chi phí
  },
};
```

⚠️ **Quan trọng**: Cập nhật địa chỉ email để nhận thông báo alarm.

### Bước 2: Synthesize CloudFormation Template

Di chuyển đến thư mục infrastructure:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Synthesize Observability Stack:

```powershell
npm run synth
```

**Kết quả mong đợi** (1175 dòng):
```
✨ Synthesis time: 3.5s

Resources:
[+] AWS::SNS::Topic                         AlarmTopic
[+] AWS::SNS::Subscription                  AlarmTopic/EmailSubscription
[+] AWS::CloudWatch::Alarm                  API-5XX-Critical
[+] AWS::CloudWatch::Alarm                  API-4XX-Warning
[+] AWS::CloudWatch::Alarm                  API-Latency-High
[+] AWS::CloudWatch::Alarm                  Lambda-Error-Rate
[+] AWS::CloudWatch::Alarm                  Lambda-Throttle
[+] AWS::CloudWatch::Alarm                  Lambda-Duration-High
[+] AWS::CloudWatch::Alarm                  DynamoDB-Read-Throttle
[+] AWS::CloudWatch::Alarm                  DynamoDB-Write-Throttle
[+] AWS::CloudWatch::Alarm                  DynamoDB-Latency-High
[+] AWS::CloudWatch::Alarm                  S3-4XX-Warning
[+] AWS::CloudWatch::Alarm                  S3-5XX-Critical
[+] AWS::CloudWatch::Alarm                  SQS-DLQ-Messages
[+] AWS::CloudWatch::Alarm                  SQS-Queue-Age
[+] AWS::CloudWatch::Alarm                  Cost-Warning
[+] AWS::CloudWatch::Alarm                  Cost-Critical
[+] AWS::CloudWatch::CompositeAlarm         SystemHealth
[+] AWS::CloudWatch::Dashboard              CoreDashboard
[+] AWS::CloudWatch::Dashboard              AuthDashboard
[+] AWS::CloudWatch::Dashboard              BackendDashboard
[+] AWS::CloudWatch::Dashboard              OverviewDashboard

Outputs:
- AlarmTopicArn
- CompositeAlarmName
- CoreDashboardName
- AuthDashboardName
- BackendDashboardName
- OverviewDashboardName
```

![CDK Synth Output](/images/5-Workshop/5.4-configure-stacks/5.4.7/synth-output.png)
*Screenshot: CDK synth output hiển thị tất cả tài nguyên Observability Stack*

### Bước 3: Xem lại Generated Template

Mở CloudFormation template đã tạo:

```powershell
code infrastructure/cdk.out/EveryoneCook-dev-Observability.template.json
```

![CloudFormation Template](/images/5-Workshop/5.4-configure-stacks/cloudformation-template.png)
*Screenshot: Generated CloudFormation template hiển thị SNS Topic, 15+ Alarms, Composite Alarm và 4 Dashboards*

### Bước 4: Deploy Observability Stack

Deploy sử dụng CDK:

```powershell
npx cdk deploy EveryoneCook-dev-Observability --require-approval never
```

**Thời gian triển khai dự kiến**: 2-3 phút

**Kết quả Deployment**:
```
EveryoneCook-dev-Observability: deploying...

EveryoneCook-dev-Observability: creating CloudFormation changeset...

EveryoneCook-dev-Observability

Outputs:
EveryoneCook-dev-Observability.AlarmTopicArn = arn:aws:sns:ap-southeast-1:123456789012:EveryoneCook-dev-Alarms
EveryoneCook-dev-Observability.CompositeAlarmName = EveryoneCook-dev-SystemHealth
EveryoneCook-dev-Observability.CoreDashboardName = EveryoneCook-dev-Core
EveryoneCook-dev-Observability.AuthDashboardName = EveryoneCook-dev-Auth
EveryoneCook-dev-Observability.BackendDashboardName = EveryoneCook-dev-Backend
EveryoneCook-dev-Observability.OverviewDashboardName = EveryoneCook-dev-Overview

Stack ARN:
arn:aws:cloudformation:ap-southeast-1:123456789012:stack/EveryoneCook-dev-Observability/...

✨ Deployment time: 2m 15s
```


## Xác minh

### Bước 1: Xác minh CloudFormation Stack

Điều hướng đến **CloudFormation Console**:

```
AWS Console → CloudFormation → Stacks
```

**Xác minh Stack**:
- Stack Name: `EveryoneCook-dev-Observability`
- Status: **CREATE_COMPLETE** ✅
- Resources: **24 resources** đã tạo

![CloudFormation Stack](/images/5-Workshop/5.4-configure-stacks/5.4.7/cloudformation-stack.png)
*Screenshot: CloudFormation stack với trạng thái CREATE_COMPLETE và 24 resources*

![CloudFormation Outputs](/images/5-Workshop/5.4-configure-stacks/5.4.7/cloudformation-outputs.png)
*Screenshot: CloudFormation Outputs tab hiển thị tất cả 6 outputs*

### Bước 2: Xác minh CloudWatch Alarms

Điều hướng đến **CloudWatch Console** → **Alarms**:

```
AWS Console → CloudWatch → All alarms
```

**Xác minh Alarms đã tạo** (15+ alarms):

| Tên Alarm | Loại | Metric | Ngưỡng | Trạng thái |
|------------|------|--------|-----------|--------|
| `EveryoneCook-dev-API-5XX-Critical` | Critical | API 5XX Errors | > 5 | OK |
| `EveryoneCook-dev-API-4XX-Warning` | Warning | API 4XX Errors | > 20 | OK |
| `EveryoneCook-dev-API-Latency-High` | Warning | API P99 Latency | > 3000ms | OK |
| `EveryoneCook-dev-Lambda-Error-Rate` | Critical | Lambda Errors | > 5 | OK |
| `EveryoneCook-dev-Lambda-Throttle` | Critical | Lambda Throttles | > 10 | OK |
| `EveryoneCook-dev-Lambda-Duration-High` | Warning | Lambda P99 Duration | > 10000ms | OK |
| `EveryoneCook-dev-DynamoDB-Read-Throttle` | Critical | DynamoDB Read Throttles | > 10 | OK |
| `EveryoneCook-dev-DynamoDB-Write-Throttle` | Critical | DynamoDB Write Throttles | > 10 | OK |
| `EveryoneCook-dev-DynamoDB-Latency-High` | Warning | DynamoDB P99 Latency | > 100ms | OK |
| `EveryoneCook-dev-S3-4XX-Warning` | Warning | S3 4XX Errors | > 5% | OK |
| `EveryoneCook-dev-S3-5XX-Critical` | Critical | S3 5XX Errors | > 0 | OK |
| `EveryoneCook-dev-SQS-DLQ-Messages` | Critical | SQS DLQ Messages | > 0 | OK |
| `EveryoneCook-dev-SQS-Queue-Age` | Warning | SQS Message Age | > 300s | OK |
| `EveryoneCook-dev-Cost-Warning` | Warning | Daily Cost | > $50 | OK |
| `EveryoneCook-dev-Cost-Critical` | Critical | Daily Cost | > $100 | OK |

**Xác minh Composite Alarm**:
- Name: `EveryoneCook-dev-SystemHealth`
- Type: Composite
- Rule: BẤT KỲ critical alarm nào → ALARM
- Status: **OK** ✅

![CloudWatch Alarms List](/images/5-Workshop/5.4-configure-stacks/cloudwatch-alarms.png)
*Screenshot: CloudWatch Alarms console hiển thị tất cả 15+ alarms với trạng thái OK*

![Composite Alarm](/images/5-Workshop/5.4-configure-stacks/13-cloudwatch-alarms.png)
*Screenshot: Chi tiết Composite alarm cho giám sát trạng thái sức khỏe hệ thống*

### Bước 3: Xác minh CloudWatch Dashboards

Điều hướng đến **CloudWatch Console** → **Dashboards**:

```
AWS Console → CloudWatch → Dashboards
```

**Xác minh Dashboards đã tạo** (5 dashboards):

#### 1. Core Dashboard (`EveryoneCook-dev-Core`)

Mở dashboard và xác minh các widgets:

**DynamoDB Widgets**:
- Biểu đồ Read/Write Capacity
- Biểu đồ Throttles
- Biểu đồ Latency (P99)
- Metric Table Size

**S3 Widgets**:
- Biểu đồ Requests
- Biểu đồ Errors (4XX/5XX)

**CloudFront Widgets**:
- Biểu đồ Requests
- Biểu đồ Error Rate (4XX/5XX)
- Biểu đồ Bytes Downloaded

![Core Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.7/core-dashboard.png)
*Screenshot: Core Dashboard hiển thị DynamoDB, S3 và CloudFront metrics*

#### 2. Auth Dashboard (`EveryoneCook-dev-Auth`)

**Cognito Widgets**:
- Biểu đồ Sign-ups
- Biểu đồ Sign-ins
- Biểu đồ Failed Authentications

![Auth Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.7/auth-dashboard.png)
*Screenshot: Auth Dashboard hiển thị Cognito authentication metrics*

#### 3. Backend Dashboard (`EveryoneCook-dev-Backend`)

**API Gateway Widgets**:
- Biểu đồ Requests
- Biểu đồ Latency (P50/P95/P99)
- Biểu đồ 4XX Errors
- Biểu đồ 5XX Errors

**Lambda Widgets**:
- Biểu đồ Invocations
- Biểu đồ Duration (P99)
- Biểu đồ Errors
- Biểu đồ Throttles

**SQS Widgets**:
- Biểu đồ Messages Sent
- Biểu đồ Messages Visible
- Biểu đồ Oldest Message Age

![Backend Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.7/backend-dashboard.png)
*Screenshot: Backend Dashboard hiển thị API Gateway, Lambda và SQS metrics*

#### 4. Overview Dashboard (`EveryoneCook-dev-Overview`)

**Phần System Health**:
- Header với thông tin environment
- Thông tin Region

**Key Metrics**:
- API Requests (5m) - Single value widget
- API P99 Latency - Single value widget
- Lambda Invocations (5m) - Single value widget
- Lambda Errors (5m) - Single value widget
- DynamoDB Read/Write Throttles - Single value widgets
- S3 Requests và Errors - Single value widgets

**Trends**:
- Biểu đồ Error Rates (API 5XX, Lambda Errors)

**Cost Tracking**:
- Estimated Daily Cost - Single value widget
- Cost Trend (7 ngày) - Graph widget

**Alarm Status**:
- Composite Alarm widget hiển thị trạng thái sức khỏe hệ thống

![Overview Dashboard](/images/5-Workshop/5.4-configure-stacks/5.4.7/overview-dashboard.png)
*Screenshot: Overview Dashboard với system health, key metrics, error trends và cost tracking*



## Phân tích Chi phí

### Chi phí hàng tháng (Development)

| Dịch vụ | Tài nguyên | Số lượng | Đơn giá | Tổng |
|---------|----------|----------|-----------|-------|
| CloudWatch Alarms | Standard alarms | 15 | $0.10/alarm | $1.50 |
| CloudWatch Alarms | Composite alarm | 1 | $0.50/alarm | $0.50 |
| CloudWatch Dashboards | Dashboards (>3) | 1 | $3.00/dashboard | $3.00 |
| CloudWatch Metrics | Standard resolution | Bao gồm | Miễn phí | $0.00 |
| SNS | Email notifications | <1000 | Free tier | $0.00 |
| CloudWatch Logs | 7-day retention | ~5 GB | $0.50/GB | $2.50 |
| **Tổng cộng** | | | | **$7.50/tháng** |

**Lợi ích Free Tier**:
- 3 dashboard đầu tiên: Miễn phí
- 10 alarm đầu tiên: Miễn phí (đã bao gồm)
- 1 triệu API request đầu tiên tới CloudWatch: Miễn phí
- SNS email (1000 đầu tiên): Miễn phí

---

## Các bước tiếp theo

Sau khi triển khai Observability Stack:

1. **Xác nhận Email Subscription**: Kiểm tra inbox và xác nhận SNS subscription

2. **Xem lại Dashboards**: Làm quen với tất cả 4 dashboards

3. **Kiểm tra Alarms**: Kích hoạt test alarm để xác minh thông báo

4. **Giám sát Chi phí**: Kiểm tra theo dõi chi phí hàng ngày trong Overview Dashboard

5. ⏭️ **Deploy Frontend**: Tiếp tục đến [5.10 Deploy to Amplify](../../5.10-deploy-amplify)

6. ⏭️ **Test End-to-End**: Kiểm tra luồng ứng dụng hoàn chỉnh và giám sát metrics

7. 📊 **Xem lại Metrics**: Sau 24 giờ, xem lại tất cả dashboards để có baseline metrics

---

## Tóm tắt

Bạn đã triển khai thành công **Observability Stack** với:

**1 SNS Topic** cho thông báo alarm  
**15+ CloudWatch Alarms** cho các metric quan trọng  
**1 Composite Alarm** cho trạng thái sức khỏe tổng thể hệ thống  
**4 CloudWatch Dashboards** cho giám sát  
**Email Notifications** đã cấu hình và xác nhận  

**Thành tựu chính**:
- Tầm nhìn hoàn chỉnh về trạng thái sức khỏe hệ thống
- Cảnh báo chủ động cho các vấn đề nghiêm trọng
- Theo dõi và tối ưu chi phí
- Dashboards giám sát tập trung

**Tổng tài nguyên**: 24 CloudFormation resources  
**Thời gian triển khai**: ~2-3 phút  
**Chi phí hàng tháng**: ~$7.50 (môi trường dev)

---

**🎉 Chúc mừng!** Bạn đã hoàn thành tất cả các triển khai infrastructure stack. Nền tảng EveryoneCook của bạn giờ đây có giám sát và observability toàn diện.

**Tiếp theo**: [5.10 Deploy to Amplify](../../5.10-deploy-amplify) để triển khai ứng dụng Next.js frontend.

---
