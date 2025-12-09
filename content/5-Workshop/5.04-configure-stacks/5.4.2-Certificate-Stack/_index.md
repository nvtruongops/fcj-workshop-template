---
title: "5.4.2 Certificate Stack"
weight: 2
---
---
# Certificate Stack - ACM Certificates for SSL/TLS

## Overview

The Certificate Stack is the **Phase 1.5 infrastructure layer** of the EveryoneCook project. It manages AWS Certificate Manager (ACM) certificates for CloudFront CDN and API Gateway, providing SSL/TLS encryption for all HTTPS traffic.

**Deployment Order**: This stack **MUST** be deployed after DNS Stack and **before** Core Stack and Backend Stack.

**Critical Region Requirement**: This stack **MUST** be deployed in **us-east-1** region because CloudFront is a global service that can only access ACM certificates from us-east-1.

### Key Responsibilities

- Create ACM certificate for CloudFront CDN (`cdn.everyonecook.cloud` or `cdn-dev.everyonecook.cloud`)
- Create wildcard ACM certificate for API Gateway (`*.everyonecook.cloud`)
- Automatic DNS validation via Route 53
- Export certificate ARNs for Core Stack and Backend Stack

### What This Stack Does NOT Include

- DNS records (managed by DNS Stack - Phase 1)
- CloudFront distribution (managed by Core Stack - Phase 2)
- API Gateway custom domain (managed by Backend Stack - Phase 4)
- CloudFront WAF Web ACL (removed for cost optimization)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Route 53 Hosted Zone                          │
│                    everyonecook.cloud                            │
│                    (from DNS Stack)                              │
└───────────────────┬─────────────────────────────────────────────┘
                    │ DNS Validation
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│            AWS Certificate Manager (us-east-1)                   │
│                                                                  │
│  Certificate 1: CloudFront Certificate                          │
│  ├─ Domain: cdn.everyonecook.cloud (or cdn-dev)                │
│  ├─ Validation: DNS (Route 53)                                 │
│  ├─ Status: Issued (5-10 minutes)                              │
│  └─ Export: CloudFrontCertificateArn                           │
│                                                                  │
│  Certificate 2: API Gateway Wildcard Certificate               │
│  ├─ Domain: *.everyonecook.cloud                               │
│  ├─ SAN: everyonecook.cloud                                    │
│  ├─ Covers: api.everyonecook.cloud, api-dev, api-staging      │
│  ├─ Validation: DNS (Route 53)                                 │
│  ├─ Status: Issued (5-10 minutes)                              │
│  └─ Export: ApiGatewayCertificateArn                           │
│                                                                  │
│  Cost Optimization:                                             │
│   CloudFront WAF removed (-$9/month)                         │
│   Shield Standard (free, auto-enabled)                       │
└─────────────────────────────────────────────────────────────────┘
                    │
                    │ Certificate ARN Exports
                    ▼
        ┌───────────────────────┬
        ▼                       ▼                    
  Core Stack              Backend Stack       
  (CloudFront)           (API Gateway)
```

---

## Stack Configuration

### File Structure

```
infrastructure/lib/stacks/
└── certificate-stack.ts     # Certificate Stack implementation
```

### Code Implementation

**File**: `infrastructure/lib/stacks/certificate-stack.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import * as acm from 'aws-cdk-lib/aws-certificatemanager';
import * as route53 from 'aws-cdk-lib/aws-route53';
import { Construct } from 'constructs';
import { BaseStack, BaseStackProps } from '../base-stack';

/**
 * Certificate Stack for CloudFront and API Gateway
 *
 * This stack creates ACM certificates for CloudFront and API Gateway.
 *
 * IMPORTANT REGION REQUIREMENTS:
 * - CloudFront certificate: MUST be in us-east-1 (CloudFront requirement)
 * - API Gateway certificate: Should be in same region as API Gateway (ap-southeast-1)
 *
 * This stack is deployed in us-east-1 to handle CloudFront's cross-region requirements.
 * For API Gateway, we use a wildcard certificate that covers api.everyonecook.cloud.
 *
 * Responsibilities:
 * - Create ACM certificate for CloudFront in us-east-1
 * - Create ACM wildcard certificate for API Gateway in us-east-1 (works globally)
 * - Validate certificates via Route 53 DNS
 * - Export certificate ARNs for Core Stack and Backend Stack to use
 *
 * COST OPTIMIZATION NOTE:
 * - CloudFront WAF removed to save $9/month ($108/year)
 * - CloudFront still protected by Shield Standard (free, auto-enabled)
 * - API Gateway has full WAF protection (BackendStack)
 */
export class CertificateStack extends BaseStack {
  public readonly cloudFrontCertificate: acm.ICertificate;
  public readonly apiGatewayCertificate: acm.ICertificate;

  constructor(scope: Construct, id: string, props: BaseStackProps) {
    super(scope, id, props);

    // Add stack-specific tags
    cdk.Tags.of(this).add('StackType', 'Certificate');
    cdk.Tags.of(this).add('Layer', 'Infrastructure');
    cdk.Tags.of(this).add('CostCenter', `Certificate-${this.config.environment}`);

    // Import Route 53 Hosted Zone from DNS Stack
    // Note: Cannot use Fn.importValue or SSM Parameter for cross-region references
    // Hosted Zone ID is stable and doesn't change, so we hardcode it
    // This value comes from DNS Stack output: Z018823421GWCSYG5UMHV
    const hostedZoneId = 'Z018823421GWCSYG5UMHV';

    const hostedZone = route53.HostedZone.fromHostedZoneAttributes(this, 'HostedZone', {
      hostedZoneId: hostedZoneId,
      zoneName: 'everyonecook.cloud',
    });

    // Create ACM certificate for CloudFront
    // This certificate MUST be in us-east-1 for CloudFront to use it
    this.cloudFrontCertificate = this.createCloudFrontCertificate(hostedZone);

    // Create ACM wildcard certificate for API Gateway
    // Wildcard *.everyonecook.cloud covers api.everyonecook.cloud
    // This certificate in us-east-1 can be used by API Gateway in any region
    this.apiGatewayCertificate = this.createApiGatewayCertificate(hostedZone);

    // COST OPTIMIZATION: CloudFront WAF removed
    // CloudFront is protected by Shield Standard (free, auto-enabled)
    // API Gateway has full WAF protection in BackendStack
    // Savings: $9/month ($108/year)

    // Export certificate ARNs
    this.exportOutputs();
  }

  /**
   * Create ACM certificate for CloudFront CDN
   *
   * CRITICAL: This stack MUST be deployed in us-east-1 region.
   * CloudFront is a global service but its control plane is in us-east-1,
   * so it can only access certificates from us-east-1.
   *
   * DNS validation is automatic via Route 53.
   * Validation typically takes 5-10 minutes.
   *
   * @param hostedZone - Route 53 Hosted Zone for DNS validation
   * @returns ACM Certificate for CloudFront
   */
  private createCloudFrontCertificate(hostedZone: route53.IHostedZone): acm.Certificate {
    const certificate = new acm.Certificate(this, 'CloudFrontCertificate', {
      domainName: this.config.domains.cdn,
      validation: acm.CertificateValidation.fromDns(hostedZone),
      certificateName: `EveryoneCook-CloudFront-${this.config.environment}`,
    });

    // Add tags
    cdk.Tags.of(certificate).add('Component', 'CloudFront');
    cdk.Tags.of(certificate).add('Purpose', 'CDN-SSL');

    return certificate;
  }

  /**
   * Create ACM wildcard certificate for API Gateway
   *
   * Creates a wildcard certificate (*.everyonecook.cloud) that covers:
   * - api.everyonecook.cloud (API Gateway)
   * - api-dev.everyonecook.cloud (API Gateway dev)
   * - api-staging.everyonecook.cloud (API Gateway staging)
   *
   * This certificate is created in us-east-1 but can be used by API Gateway
   * in any region via cross-region certificate reference.
   *
   * DNS validation is automatic via Route 53.
   * Validation typically takes 5-10 minutes.
   *
   * @param hostedZone - Route 53 Hosted Zone for DNS validation
   * @returns ACM Certificate for API Gateway
   */
  private createApiGatewayCertificate(hostedZone: route53.IHostedZone): acm.Certificate {
    const certificate = new acm.Certificate(this, 'ApiGatewayCertificate', {
      domainName: '*.everyonecook.cloud', // Wildcard covers api.everyonecook.cloud
      subjectAlternativeNames: ['everyonecook.cloud'], // Also covers root domain
      validation: acm.CertificateValidation.fromDns(hostedZone),
      certificateName: `EveryoneCook-API-${this.config.environment}`,
    });

    // Add tags
    cdk.Tags.of(certificate).add('Component', 'APIGateway');
    cdk.Tags.of(certificate).add('Purpose', 'API-SSL');

    return certificate;
  }

  /**
   * Export stack outputs for cross-stack references
   *
   * Exports:
   * - CloudFrontCertificateArn: ACM Certificate ARN for CloudFront (us-east-1)
   * - ApiGatewayCertificateArn: ACM Certificate ARN for API Gateway (us-east-1)
   *
   * REMOVED: CloudFrontWebAclArn (cost optimization)
   */
  private exportOutputs(): void {
    // Export CloudFront certificate ARN for Core Stack
    new cdk.CfnOutput(this, 'CloudFrontCertificateArn', {
      value: this.cloudFrontCertificate.certificateArn,
      exportName: this.exportName('CloudFrontCertificateArn'),
      description: 'ACM Certificate ARN for CloudFront (us-east-1)',
    });

    // Export CloudFront certificate domain for verification
    new cdk.CfnOutput(this, 'CloudFrontCertificateDomain', {
      value: this.config.domains.cdn,
      description: 'Domain name for CloudFront certificate',
    });

    // Export API Gateway certificate ARN for Backend Stack
    new cdk.CfnOutput(this, 'ApiGatewayCertificateArn', {
      value: this.apiGatewayCertificate.certificateArn,
      exportName: this.exportName('ApiGatewayCertificateArn'),
      description: 'ACM Wildcard Certificate ARN for API Gateway (us-east-1)',
    });

    // Export API Gateway certificate domain for verification
    new cdk.CfnOutput(this, 'ApiGatewayCertificateDomain', {
      value: '*.everyonecook.cloud',
      description: 'Domain name for API Gateway certificate (wildcard)',
    });
  }
}
```

---

## Key Configuration Details

### 1. Region Requirements

**Critical**: This stack **MUST** be deployed to **us-east-1**:

```typescript
// In infrastructure/bin/app.ts
const certificateStack = new CertificateStack(app, `${stackPrefix}-Certificate`, {
  env: {
    account: config.account,
    region: 'us-east-1', // MUST be us-east-1 for CloudFront
  },
  config,
  description: `ACM Certificate for CloudFront (${config.environment}) - us-east-1`,
});
```

**Why us-east-1?**

- CloudFront is a global service with control plane in us-east-1
- CloudFront can **only** access ACM certificates from us-east-1
- API Gateway wildcard certificate in us-east-1 works globally

### 2. Certificate Domains

The stack creates two certificates with environment-specific domains:

**CloudFront Certificate**:

```typescript
// Dev environment
domainName: 'cdn-dev.everyonecook.cloud'

// Staging environment
domainName: 'cdn-staging.everyonecook.cloud'

// Production environment
domainName: 'cdn.everyonecook.cloud'
```

**API Gateway Wildcard Certificate**:

```typescript
domainName: '*.everyonecook.cloud'           // Covers all subdomains
subjectAlternativeNames: ['everyonecook.cloud']  // Also covers root domain

// Covers:
// - api.everyonecook.cloud
// - api-dev.everyonecook.cloud
// - api-staging.everyonecook.cloud
// - Any future *.everyonecook.cloud subdomains
```

### 3. Automatic DNS Validation

ACM automatically creates DNS validation records in Route 53:

```typescript
validation: acm.CertificateValidation.fromDns(hostedZone)
```

**What Happens**:

1. ACM creates CNAME record in Route 53 for validation
2. Route 53 immediately responds with the validation record
3. ACM verifies the record and issues the certificate
4. Validation completes in 5-10 minutes (typically faster)

### 4. Cross-Region Certificate Reference

The certificate created in us-east-1 is used by other stacks in ap-southeast-1:

```typescript
// In Core Stack (ap-southeast-1) - imports CloudFront certificate from us-east-1
const certificate = acm.Certificate.fromCertificateArn(
  this,
  'ImportedCloudFrontCertificate',
  'arn:aws:acm:us-east-1:616580903213:certificate/8d53776e-0480-47d2-a6ff-4fe9b2eb6534'
);
```

### 5. Cost Optimization - WAF Removed

**Decision**: CloudFront WAF Web ACL removed to save costs:

```typescript
// REMOVED: CloudFront WAF Web ACL
// Previous monthly cost: $9/month = $108/year

// Protection Status:
//  Shield Standard: DDoS protection (free, auto-enabled)
//  CloudFront OAC: Blocks direct S3 access
//  Signed URLs: Private content protection
// ❌ WAF: Removed (cost optimization)
```

**Rationale**:

- CloudFront serves static content only (low attack surface)
- Shield Standard provides Layer 3/4 DDoS protection (free)
- API Gateway has full WAF protection for Layer 7 attacks
- **Savings**: $9/month = $108/year

---

## Stack Outputs

After deployment, the stack exports the following values:

| Output Name                    | Value                                                                       | Usage                                            |
| ------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------ |
| `CloudFrontCertificateArn`   | `arn:aws:acm:us-east-1:616580903213:certificate/8d53776e-...`            | Used by Core Stack for CloudFront distribution   |
| `CloudFrontCertificateDomain`| `cdn.everyonecook.cloud` (or `cdn-dev`)                                  | Verification only                                |
| `ApiGatewayCertificateArn`   | `arn:aws:acm:us-east-1:616580903213:certificate/a1b2c3d4-...`            | Used by Backend Stack for API Gateway domain     |
| `ApiGatewayCertificateDomain`| `*.everyonecook.cloud`                                                    | Verification only (wildcard)                     |

---

## Deployment Steps

### Step 1: Verify Prerequisites

Before deploying Certificate Stack, ensure:

-  DNS Stack successfully deployed
-  Route 53 Hosted Zone exists with correct nameservers
-  DNS delegation from Hostinger completed and propagated

Verify DNS is working:

```powershell
nslookup -type=NS everyonecook.cloud
```

### Step 2: Deploy Certificate Stack

Navigate to infrastructure directory:

```powershell
cd D:\Project_AWS\everyonecook\infrastructure
```

Deploy Certificate Stack to **us-east-1**:

```powershell
# Deploy Certificate Stack
npx cdk deploy EveryoneCook-dev-Certificate --context environment=dev
```

**Important**: Notice the region is **us-east-1**, not ap-southeast-1.

Expected output:

```
✨  Synthesis time: 6.12s

EveryoneCook-dev-Certificate: deploying...
[████████████████████████████████████████] (3/3)

EveryoneCook-dev-Certificate: creating CloudFormation changeset...

   EveryoneCook-dev-Certificate

✨  Deployment time: 125.34s

Outputs:
EveryoneCook-dev-Certificate.CloudFrontCertificateArn = 
  arn:aws:acm:us-east-1:616580903213:certificate/8d53776e-0480-47d2-a6ff-4fe9b2eb6534
EveryoneCook-dev-Certificate.CloudFrontCertificateDomain = cdn-dev.everyonecook.cloud
EveryoneCook-dev-Certificate.ApiGatewayCertificateArn = 
  arn:aws:acm:us-east-1:616580903213:certificate/a1b2c3d4-5678-90ef-ghij-klmnopqrstuv
EveryoneCook-dev-Certificate.ApiGatewayCertificateDomain = *.everyonecook.cloud

Stack ARN:
arn:aws:cloudformation:us-east-1:616580903213:stack/EveryoneCook-dev-Certificate/...
```

### Step 3: Wait for Certificate Validation

ACM certificates require DNS validation. This process takes 5-10 minutes:

1. ACM creates CNAME validation records in Route 53
2. Route 53 responds with validation data
3. ACM verifies and issues certificates
4. Status changes from "Pending validation" to "Issued"

You can monitor the validation progress in AWS Console.

### Step 4: Verify in AWS Console

#### Navigate to ACM (us-east-1 region)

1. Open AWS Console
2. **IMPORTANT**: Switch region to **N. Virginia (us-east-1)**
3. Navigate to **Certificate Manager**

![Switch to us-east-1 Region](/images/5-Workshop/5.4-configure-stacks/switch-region-us-east-1.png)
*Switch AWS Console to us-east-1 region before viewing certificates*

#### Verify CloudFront Certificate

1. Find certificate with domain `cdn-dev.everyonecook.cloud`
2. Check status is **Issued** 
3. Verify DNS validation records are present

![CloudFront ACM Certificate](/images/5-Workshop/5.4-configure-stacks/acm-cloudfront-cert.png)
*ACM Certificate for CloudFront showing "Issued" status, domain name, validation method (DNS), and CNAME validation record*

#### Verify API Gateway Wildcard Certificate

1. Find certificate with domain `*.everyonecook.cloud`
2. Check status is **Issued** 
3. Verify it covers wildcard and SAN (everyonecook.cloud)

![API Gateway Wildcard Certificate](/images/5-Workshop/5.4-configure-stacks/acm-api-wildcard-cert.png)
*ACM Wildcard Certificate for API Gateway showing domain `*.everyonecook.cloud`, SAN `everyonecook.cloud`, and validation records*

#### Verify DNS Validation Records in Route 53

1. Navigate to **Route 53** > **Hosted zones**
2. Select `everyonecook.cloud` hosted zone
3. Verify CNAME validation records are created

![Route 53 ACM Validation Records](/images/5-Workshop/5.4-configure-stacks/route53-acm-validation.png)
*Route 53 showing CNAME validation records automatically created by ACM for certificate validation*

Expected records:

```
_abc123def456.cdn-dev.everyonecook.cloud  CNAME  _xyz789.acm-validations.aws.
_ghi789jkl012.everyonecook.cloud          CNAME  _mno345.acm-validations.aws.
```

---

## Cost Breakdown

### Monthly Costs

| Resource                           | Cost              | Notes                                      |
| ---------------------------------- | ----------------- | ------------------------------------------ |
| **ACM Certificates**         | **$0/month**    | Free for certificates used with AWS services |
| **DNS Validation Records**   | **$0/month**    | Included in Route 53 hosted zone cost      |
| **Certificate Renewal**      | **$0/month**    | Automatic renewal (free)                   |
| **Total**                    | **$0/month**    | 100% free (no ongoing costs)               |

### Cost Optimization

-  ACM certificates are free when used with CloudFront, API Gateway, ALB, etc.
-  Automatic renewal (no manual intervention required)
-  CloudFront WAF removed to save $9/month ($108/year)
-  Shield Standard provides free DDoS protection

**Annual Savings from WAF Removal**: $108/year

---

## Cross-Stack Dependencies

### Imports from DNS Stack

The Certificate Stack imports from DNS Stack:

```typescript
// Hardcoded Hosted Zone ID (stable, doesn't change)
const hostedZoneId = 'Z018823421GWCSYG5UMHV';

const hostedZone = route53.HostedZone.fromHostedZoneAttributes(this, 'HostedZone', {
  hostedZoneId: hostedZoneId,
  zoneName: 'everyonecook.cloud',
});
```

**Why Hardcoded?**

- Cannot use `Fn.importValue` for cross-region references (us-east-1 ← ap-southeast-1)
- Hosted Zone ID is stable and never changes
- Alternative: Use SSM Parameter Store (more complex, same result)

### Exports Used By Other Stacks

The Certificate Stack exports are used by:

1. **Core Stack** (Phase 2)

   - Imports: `CloudFrontCertificateArn`
   - Purpose: Attach SSL certificate to CloudFront distribution
   - Region: ap-southeast-1 (cross-region import)
2. **Backend Stack** (Phase 4)

   - Imports: `ApiGatewayCertificateArn`
   - Purpose: Create API Gateway custom domain with SSL
   - Region: ap-southeast-1 (cross-region import)

### Dependency Flow

```
DNS Stack (ap-southeast-1)
    │
    ├─ Hosted Zone ID: Z018823421GWCSYG5UMHV
    │
    ▼
Certificate Stack (us-east-1) ← MUST be us-east-1
    │
    ├─ CloudFront Certificate ARN
    │  └─► Core Stack (ap-southeast-1)
    │
    └─ API Gateway Certificate ARN
       └─► Backend Stack (ap-southeast-1)
```

---

## Validation Checklist

Before proceeding to Core Stack deployment:

- [ ] Certificate Stack deployed to **us-east-1** region
- [ ] CloudFront certificate status is **Issued** 
- [ ] API Gateway wildcard certificate status is **Issued** 
- [ ] DNS validation CNAME records visible in Route 53
- [ ] Certificate ARNs exported in CloudFormation outputs
- [ ] Region confirmed as us-east-1 in AWS Console
- [ ] Tags applied correctly to all resources

## Next Steps

After successfully deploying the Certificate Stack:

➡️ **[5.4.3 Core Stack](../5.4.3-Core-Stack/)** - Create DynamoDB, S3, and CloudFront infrastructure

The Core Stack will:

- Create DynamoDB Single Table with encryption
- Create S3 buckets for content and CDN logs
- Create CloudFront distribution with SSL certificate
- Import `CloudFrontCertificateArn` from this stack
- Configure custom domain `cdn.everyonecook.cloud`

---

## References

- **Source Code**: `infrastructure/lib/stacks/certificate-stack.ts`
- **Base Stack**: `infrastructure/lib/base-stack.ts`
- **App Configuration**: `infrastructure/bin/app.ts`
- **AWS Documentation**:
  - [ACM Certificates](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
  - [DNS Validation](https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html)
  - [CloudFront with ACM](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https.html)
- **Cross-Region References**: [CloudFormation Cross-Region References](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-crossstackref.html)
